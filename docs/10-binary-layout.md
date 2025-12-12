# 10. Binary Layout и Сериализация

## Основная идея

Типы в Fujin — это **упаковочный формат** для обмена сообщениями (как Cap'n Proto).

**Ключевые принципы:**
1. Порядок полей в интерфейсах **имеет значение**
2. Каждый тип имеет фиксированный binary layout
3. При компиляции тип → binary hash
4. Hash используется для идентификации типа в сообщениях

## Примитивные типы и их размеры

| Тип | Размер | Alignment |
|-----|--------|-----------|
| `u8`, `i8`, `bool`, `byte` | 1 byte | 1 |
| `u16`, `i16` | 2 bytes | 2 |
| `u32`, `i32` | 4 bytes | 4 |
| `u64`, `i64` | 8 bytes | 8 |
| `f32` | 4 bytes | 4 |
| `f64` | 8 bytes | 8 |

## Layout типов (type)

### Фиксированные структуры

```fujin
type Point = {
  x: f64  // offset 0, size 8, align 8
  y: f64  // offset 8, size 8, align 8
}
// Total size: 16 bytes
// Alignment: 8 bytes
// Hash: 0x3a4f... (computed at compile time)
```

### Порядок полей критичен

```fujin
// Эти два типа имеют РАЗНЫЕ хеши!
type A = {
  x: i32
  y: i32
}

type B = {
  y: i32  // другой порядок!
  x: i32
}
```

### Padding и alignment

```fujin
type Header = {
  magic: u32    // offset 0, size 4
  // padding: 4 bytes для alignment
  timestamp: u64 // offset 8, size 8
  flags: u16    // offset 16, size 2
  version: u16  // offset 18, size 2
}
// Total: 20 bytes, align 8
```

## Строки и массивы (variable length)

### Строки

```fujin
// Wire format: [length: u32][data: bytes]
type Message = {
  id: u64      // offset 0, fixed
  text: string // offset 8, pointer to length-prefixed data
}
```

### Массивы

```fujin
// Wire format: [length: u32][items...]
type Batch = {
  id: u64       // offset 0, fixed
  items: i32[]  // offset 8, pointer to length-prefixed array
}
```

### Фиксированные массивы

```fujin
// Inline data, no length prefix
type Vec3 = [f64, f64, f64]  // 24 bytes inline

type Transform = {
  position: Vec3  // offset 0, 24 bytes inline
  rotation: Vec3  // offset 24, 24 bytes inline
}
// Total: 48 bytes, align 8
```

## Tagged Unions (для сообщений)

```fujin
type Command = 
  | { type: "create", id: u64, name: string }
  | { type: "update", id: u64, data: string }
  | { type: "delete", id: u64 }

// Wire format:
// [tag: u8][payload according to tag]
```

## Nullable и Optional

### Nullable (union with null)

```fujin
type User = {
  id: u64
  name: string
  email: string | null  // tag byte + data
}

// Wire format for nullable:
// [is_null: u8][data if not null]
```

### Optional поля

```fujin
type Profile = {
  id: u64
  name: string
  bio?: string  // presence flag + data
}

// Wire format:
// [has_bio: u8][bio data if present]
```

## Вложенные структуры

```fujin
type Address = {
  street: string
  city: string
}

type User = {
  id: u64
  name: string
  address: Address  // embedded inline
}

// Address данные идут inline после name
```

## Аннотации для layout

### @packed — без padding

```fujin
@packed
type CompactHeader = {
  magic: u32    // offset 0
  version: u16  // offset 4 (no padding!)
  flags: u8     // offset 6
}
// Total: 7 bytes (no padding)
```

### @align(N) — выравнивание

```fujin
@align(16)
type AlignedBlock = {
  data: u64
}
// Alignment: 16 bytes (cache line)
```

## Type Hash при компиляции

```fujin
type User = {
  id: u64
  name: string
  email: string
}

// Компилятор генерирует:
// 1. Binary layout описание
// 2. Hash типа: hash(type_definition) -> u64
// 3. В сообщениях используется hash вместо имени типа
```

### Формат сообщения

```
[type_hash: u64][payload_length: u32][payload: bytes]
```

## Ограничения для wire format

### ✅ Можно передавать

- Примитивы (u8-u64, i8-i64, f32, f64, bool)
- Строки (length-prefixed)
- Массивы примитивов и структур
- Вложенные структуры
- Tagged unions
- Nullable типы

### ❌ Нельзя передавать

- Функции (no closures)
- Promises (нельзя сериализовать)
- Циклические ссылки

## Пример полного типа

```fujin
type TaskMessage = {
  // Fixed header
  id: u64           // offset 0
  timestamp: u64    // offset 8
  priority: u8      // offset 16
  // padding: 7 bytes
  
  // Variable data
  payload: string   // offset 24, pointer
  tags: string[]    // offset 32, pointer
}

// Type hash: 0xABCD1234...
// Wire format:
// [hash: u64][length: u32][fixed_data: 40 bytes][var_data...]
```
