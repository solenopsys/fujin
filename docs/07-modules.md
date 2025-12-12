# 7. Модульная система

## Import/Export

### Именованный экспорт/импорт

```fujin
// math.fjs
type AddMsg = { a: i32, b: i32 }
type SumOut = { type: "sum", value: i32 }

export const PI = 3.14159

export actor @add(msg) {
  const out: SumOut = { type: "sum", value: msg.a + msg.b }
  emit out
}

// main.fjs
import { add, PI } from "./math"
import { add as sum } from "./math"  // с переименованием
import * as Math from "./math"       // все
```

### Default экспорт/импорт

```fujin
// logger.fjs
type LogMsg = { text: string }
type LogOut = { type: "log", text: string }

export default actor @log(msg) {
  const out: LogOut = { type: "log", text: msg.text }
  emit out
}

// main.fjs
import log from "./logger"
```

### Экспорт типов

```fujin
// types.fjt
export type User = {
  id: u64
  name: string
}

export type Status = "active" | "inactive"

// main.fjs
import { User, Status } from "./types"
```

### Реэкспорт

```fujin
// index.fjs
export { add, multiply } from "./math"
export * from "./utils"
export { default as logger } from "./logger"
```

## Структура модулей

```
project/
├── types/           # .fjt файлы
│   ├── user.fjt
│   └── index.fjt
├── services/        # .fjs файлы
│   ├── user-service.fjs
│   └── index.fjs
└── main.fjs
```

### ❌ Не поддерживается
- `require()` — CommonJS
- `import()` — динамический импорт
- `export =` — legacy
- `namespace`, `module`
