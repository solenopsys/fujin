# 6. Типы

## Type — для данных и Wire Format

**Type** = структура данных (поля, binary layout)

```fujin
// Структура данных (binary layout)
type User = {
  id: u64        // offset 0, 8 bytes
  name: string   // offset 8, pointer to length-prefixed string
  email: string  // offset 16, pointer
}
// Type hash: 0xABCD... (computed at compile time)

// Optional поля
type Profile = {
  id: u64
  avatar?: string
  bio?: string
}
```

**Важно для wire format:**
- Порядок полей имеет значение!
- Type → binary layout → hash
- Используется для сериализации

## Type Aliases

```fujin
type ID = u64
type Handler = (msg: Message) => void
type Point = { x: i32, y: i32 }
```

## Union Types

```fujin
type Status = "pending" | "running" | "done"
type Value = i32 | f64 | string | u8[] | null

// Message types for emit
type ResultMessage = SuccessMessage | ErrorMessage
type SuccessMessage = { type: "success", data: string }
type ErrorMessage = { type: "error", message: string }
```

## Generics

```fujin
type Box<T> = {
  value: T
}

type Option<T> = T | null

type Message<T> = {
  type: string
  payload: T
}
```

В языке нет функций с возвращаемыми значениями (только акторы с `void`), intersection types (`&`), `readonly`, tuple types как отдельных типов, классов/наследования/декораторов/enum/namespace, модификаторов доступа/`static`, `async/await` и `Promise<T>`, `return` (используется `emit`).
