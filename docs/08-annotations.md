# 8. Аннотации

## Что такое аннотация

**Аннотация `@name()`** — это механизм присвоения имени типу для сериализации и передачи через message bus. Аннотация делает тип **сериализуемым** и позволяет использовать его для асинхронных вызовов между акторами.

**Важно:** В акторах всегда используются аннотированные типы для параметров, потому что акторы работают асинхронно и сообщения должны передаваться через шину.

Аннотации используются для:
- Присвоения имени типу для идентификации при сериализации
- Генерации wire format и type hash
- Передачи сообщений между акторами через шину

---

## Синтаксис аннотаций

### Формат объявления

```fujin
type @name(TypeName) = {
  field1: Type1
  field2: Type2
}
```

Где:
- `@name` — имя для сериализации (используется в wire format)
- `TypeName` — имя типа для использования в коде

### Примеры аннотированных типов

```fujin
// Сообщение для обработчика клика
type @click(ButtonClick) = {
  id: string
  x: i32
  y: i32
}

// Актор всегда принимает аннотированный тип
actor @click(msg) {
  emit @log { message: `Clicked at ${msg.x}, ${msg.y}` }
}

// Сообщение с данными пользователя
type @userCreated(UserCreated) = {
  id: u64
  name: string
  email: string
}

actor @userCreated(msg) {
  emit @notification { text: `User ${msg.name} created` }
}
```

---

## Зачем нужны аннотации

### 1. Сериализация через шину

Типы с аннотациями могут быть переданы через message bus между акторами:

```fujin
type @userCreated(UserCreated) = {
  id: u64
  name: string
  timestamp: u64
}

actor @userCreated(msg) {
  // msg пришло через шину в сериализованном виде
  emit @log { message: `User ${msg.name} created` }
}
```

### 2. Асинхронные вызовы

Аннотированные типы поддерживают асинхронную передачу между акторами:

```fujin
type @taskComplete(TaskComplete) = {
  taskId: u64
  result: string
  status: string
}

actor @taskComplete(msg) {
  if (msg.status === "success") {
    emit @notification { text: `Task ${msg.taskId} completed` }
  }
}
```

### 3. Генерация Wire Format

При компиляции для каждого аннотированного типа генерируется:
- Схема сериализации (binary layout)
- Type hash для идентификации
- Код для упаковки/распаковки

```fujin
type @button(Button) = {
  class: string
  click: action  // ссылка на актор
}

// Компилятор создаст схему:
// - offset 0: string pointer (class)
// - offset 8: action reference (click)
// Type hash: 0x4F2A...
```

---

## Ключевое слово action

`action` — специальный тип для передачи **ссылки на актор**.

### Использование action

```fujin
type @button(Button) = {
  class: string
  click: action  // ссылка на актор-обработчик
}

// Актор-обработчик клика
actor @click(msg) {
  emit @log { message: `Button clicked: ${msg.id}` }
}

// Использование
actor @createButton(msg) {
  const btn: Button = {
    class: "btn-primary",
    click: @click  // передаём ссылку на актор
  }
  emit @render { ...btn }
}
```

### Как работает action

- `action` — это указатель на актор
- Позволяет передавать callback-подобную логику
- Не является функцией — это ссылка на актор в системе
- При сериализации передаётся ID актора

```fujin
type @input(Input) = {
  value: string
  onChange: action  // будет вызван при изменении
  onSubmit: action  // будет вызван при отправке
}

actor @onChange(msg) {
  emit @updateState { field: "input", value: msg.value }
}

actor @onSubmit(msg) {
  emit @sendData { data: msg.formData }
}
```

---

## Аннотации в binary layout

Аннотированные типы имеют специальный формат упаковки:

```fujin
type @message(Message) = {
  type: string    // offset 0, 8 bytes (pointer)
  id: u64         // offset 8, 8 bytes
  payload: string // offset 16, 8 bytes (pointer)
}

// При сериализации:
// Header: type hash (8 bytes)
// Fields: по порядку объявления
// Padding: выравнивание по 8 байт
```

---

## Почему акторы всегда используют аннотированные типы

Акторы работают асинхронно и общаются через message bus. Каждое сообщение должно быть:
- **Сериализовано** для передачи через шину
- **Идентифицировано** по type hash
- **Десериализовано** на стороне получателя

Поэтому в акторах всегда используются аннотированные типы:

```fujin
// Правильно - аннотированный тип
type @request(Request) = {
  userId: u64
  action: string
}

actor @request(msg) {
  // msg пришло через шину в сериализованном виде
  emit @response { userId: msg.userId, status: "ok" }
}

// Неправильно - нельзя использовать не-аннотированный тип
type PlainData = {
  value: string
}

actor @handleData(msg) {  // ошибка: нет типа @handleData - PlainData не сериализуем
  // ...
}
```

---

## Особенности аннотаций

### Что можно

- Объявлять сериализуемые типы с `@name(TypeName)`
- Передавать `action` как ссылки на акторы
- Вкладывать аннотированные типы друг в друга
- Использовать в акторах для асинхронных вызовов

### Что нельзя

- Использовать аннотации на примитивах
- Использовать не-аннотированные типы в акторах
- Изменять порядок полей после публикации (breaking change)
- Создавать циклические ссылки между аннотированными типами

---

## Примеры использования

### Бизнес-события

```fujin
type @orderCreated(OrderCreated) = {
  orderId: u64
  userId: u64
  amount: f64
  timestamp: u64
}

type @paymentProcessed(PaymentProcessed) = {
  orderId: u64
  status: string
  transactionId: string
}
```

### Системные сообщения

```fujin
type @log(LogMessage) = {
  level: string
  message: string
  timestamp: u64
}

type @error(ErrorMessage) = {
  code: u32
  message: string
  stack?: string
}
```

---

## Резюме

- **Аннотация `@name(TypeName)`** — присваивает имя типу для сериализации
- **Акторы всегда используют аннотированные типы** для параметров
- **`action`** — тип для ссылок на акторы
- Аннотированные типы передаются через message bus
- Генерируется binary layout и type hash при компиляции
- Невозможно использовать не-аннотированные типы в акторах
