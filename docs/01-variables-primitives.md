# 1. Переменные и примитивы

## Переменные

### ✅ Оставить

```fujin
let count: i32 = 0           // изменяемая
const PI: f64 = 3.14159      // константа

// Деструктуризация
const [a, b] = [1, 2]
const {x, y} = point
```

### ❌ Не поддерживается
- `var` — устаревшая семантика
- `using`, `await using` — избыточно

---

## Примитивные типы

### ✅ Числовые типы

| Тип | Описание |
|-----|----------|
| `u8`, `i8` | 8-бит беззнаковый/знаковый |
| `u16`, `i16` | 16-бит |
| `u32`, `i32` | 32-бит |
| `u64`, `i64` | 64-бит |
| `f32`, `f64` | float/double |

### ✅ Другие примитивы
- `bool` — true/false
- `byte` — байт
- `string` — строки (immutable)

### ❌ Не поддерживается
- `undefined` — только `null`
- `any`, `unknown` — нарушают типизацию
- `symbol`, `bigint` — не нужны

---

## Литералы

```fujin
// Числа
const dec: i32 = 42
const hex: u32 = 0xFF
const bin: u8 = 0b1010
const oct: u16 = 0o755
const float: f64 = 3.14

// Строки
const s1 = "hello"
const s2 = 'world'

// Логические и null
const flag: bool = true
const empty: string | null = null
```

Template literals `` `Hello ${name}` `` отсутствуют (нет интерполяции строк).
