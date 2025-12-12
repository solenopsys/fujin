# 3. Акторы

## Объявление актора

Акторы — основной строительный блок Fujin. В отличие от функций, акторы:
- Не возвращают значение
- Общаются через `emit` (отправка сообщений)
- Работают в actor-модели

```fujin
// Типы для сообщений
type ProcessOut = { type: "result", data: string }
type HandleOut  = { type: "success", id: u64 } | { type: "error", message: string }

// Простой актор
actor @processMessage(msg) {
  const result = transform(msg.data)
  const out: ProcessOut = { type: "result", data: result }
  emit out
}

// Актор с обработкой
actor @handleRequest(msg) {
  let out: HandleOut
  if (msg.valid) {
    out = { type: "success", id: msg.id }
  } else {
    out = { type: "error", message: "Invalid request" }
  }
  emit out
}
```

## emit — отправка сообщений

Ключевое слово `emit` отправляет сообщение в систему:

```fujin
// types.fjt
type NotifyIn = { userId: u64, text: string }
type Notification = { type: "notification", userId: u64, text: string, timestamp: u64 }
type TaskStatus = { type: "status", taskId: u64, status: string }
type TaskResult = { type: "result", taskId: u64, output: Payload }

// main.fjs
import type { NotifyIn, Notification, Task, TaskStatus, TaskResult } from "./types"

actor @notifyUser(msg) {
  const notification: Notification = {
    type: "notification",
    userId: msg.userId,
    text: msg.text,
    timestamp: now()
  }
  emit notification
}

actor @processTask(msg) {
  msg.status = "processing"
  
  // Отправляем статус
  const status: TaskStatus = { type: "status", taskId: msg.id, status: "started" }
  emit status
  
  // Обработка...
  const output = compute(msg.payload)
  
  // Отправляем результат
  const result: TaskResult = { type: "result", taskId: msg.id, output }
  emit result
}
```

## Параметры

**Важно:**
- У актора только один параметр — `msg`.
- `msg` всегда имеет **именованный тип**, объявленный отдельно (обычно в `.fjt`), чтобы его можно было сериализовать (Cap'n Proto) и генерировать интерфейсы/`*.h`.
- `emit` отправляет только значения **именованных типов**, объявленных отдельно. Inline-объекты в `emit` запрещены, описывайте payload в типах.
- Внутри актора можно деструктурировать `msg`, но сам тип не описывается inline.

```fujin
// types.fjt
type CalcMsg = { a: i32, b: i32 }
type CalcOut = { type: "sum", value: i32 }
type Point = { x: i32, y: i32 }
type PointOut = { type: "distance", value: f64 }

// calc.fjs
import type { CalcMsg, CalcOut, Point, PointOut } from "./types"

actor @calculate(msg) {
  const out: CalcOut = { type: "sum", value: msg.a + msg.b }
  emit out
}

actor @processPoint(msg) {
  const {x, y} = msg
  const out: PointOut = { type: "distance", value: sqrt(x * x + y * y) }
  emit out
}
```

Поддерживаются только акторы: нет `function`, `return`, стрелочных функций, `this/arguments` и генераторов.
