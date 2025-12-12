# 0. Структура файлов Fujin

## Иерархия типов файлов

Fujin использует 4 типа файлов с **иерархией наследования возможностей**. Каждый последующий уровень расширяет возможности предыдущего:

```
.fjc (базовый язык: структуры, акторы, целочисленная арифметика)
  ↓ наследует всё + добавляет float и расширенную сериализацию
.fjt (типы и интерфейсы: + float, + wire format)
  ↓ наследует всё + добавляет расширенные операции
.fjs (скрипты: расширенные операции)
  ↓ наследует всё + добавляет теги и каскадные обработчики
.fjx (UI: теговые структуры + каскадные обработчики)
```

---

## .fjc — Базовый язык (Core)

**Назначение:** Базовый детерминированный язык для смартконтрактов и критичной логики

**Возможности:**
- Структуры данных (объекты, массивы)
- **Аннотации `@name()`** для сериализации
- Сериализация через message bus
- Акторы с базовым синтаксисом
- `emit` для отправки сообщений
- Условия (`if-else`, `switch`)
- Единственный цикл `for`
- Целочисленная арифметика (u8-u64, i8-i64, bool, byte, string)
- Логические операторы
- Операторы сравнения (===, !==, <, >, <=, >=)
- Базовые операции с массивами и строками

**Что НЕЛЬЗЯ:**
- **Float типы (f32, f64)** — для детерминированности
- Расширенные операции

**Пример:**

```fujin
// token.fjc
type @transfer(Transfer) = {
  from: u64
  to: u64
  amount: u64
}

type @transferResult(TransferResult) = {
  success: bool
  newBalance: u64
}

actor @transfer(msg) {
  const balance = getBalance(msg.from)
  
  if (balance >= msg.amount) {
    updateBalance(msg.from, balance - msg.amount)
    updateBalance(msg.to, getBalance(msg.to) + msg.amount)
    
    emit @transferResult({
      success: true,
      newBalance: balance - msg.amount
    })
  } else {
    emit @transferResult({
      success: false,
      newBalance: balance
    })
  }
}
```

**Почему нет float:**
- Детерминированность выполнения
- Консенсус в блокчейн сетях
- Точность финансовых операций
- Предсказуемость результатов

**Почему ЕСТЬ аннотации:**
- Смартконтракты обмениваются сообщениями через шину
- Нужна сериализация для межконтрактных вызовов
- Акторы всегда работают асинхронно

---

## .fjt — Типы и интерфейсы

**Назначение:** Описание интерфейсов и сериализуемых типов

**Наследует:** Всё из `.fjc`

**Добавляет:**
- **Float типы (f32, f64)**
- **Аннотации `@name()`** для сериализации
- Сериализация типов в wire format
- Type hash для идентификации
- Union types расширенные
- Generics расширенные
- Nullable типы

**Пример:**

```fujin
// user.fjt
type @user(User) = {
  id: u64
  name: string
  email: string
  rating: f64  // float доступен в .fjt
}

type @userCreated(UserCreated) = {
  userId: u64
  timestamp: u64
}

type @notification(Notification) = {
  text: string
}

type @scores(Scores) = { scores: Array<f64> }
type @ratingAverage(RatingAverage) = { average: f64 }

type Role = "admin" | "user" | "guest"

// Актор с аннотированным типом
actor @userCreated(msg) {
  emit @notification({ text: `User ${msg.userId} created` })
}

// Можем использовать float
actor @calculateRating(msg) {
  let sum: f64 = 0.0
  for (let i: u32 = 0; i < msg.scores.length; i++) {
    sum = sum + msg.scores[i]
  }
  const average: f64 = sum / msg.scores.length
  emit @ratingAverage({ average })
}
```

**Когда использовать:**
- Определение API между модулями
- Сериализуемые типы для передачи через шину
- Интерфейсы для внешних систем
- Типы с плавающей точкой

---

## .fjs — Скрипты общего назначения

**Назначение:** Бизнес-логика, обработка данных, общие скрипты

**Наследует:** Всё из `.fjt` (и `.fjc`)

**Добавляет:**
- Расширенные математические операции
- Расширенные методы массивов
- Расширенные методы строк
- Обработка ошибок (try-catch-finally)

**Пример:**

```fujin
// analytics.fjs
type @calculateMetrics(MetricsInput) = {
  values: Array<f64>
  weights: Array<f64>
}

type @metricsResult(MetricsResult) = {
  average: f64
  weightedAverage: f64
  variance: f64
}

actor @calculateMetrics(msg) {
  let sum: f64 = 0.0
  let weightedSum: f64 = 0.0
  let totalWeight: f64 = 0.0
  
  for (let i: u32 = 0; i < msg.values.length; i++) {
    const value = msg.values[i]
    sum = sum + value
    weightedSum = weightedSum + (value * msg.weights[i])
    totalWeight = totalWeight + msg.weights[i]
  }
  
  const avg = sum / msg.values.length
  const weightedAvg = weightedSum / totalWeight
  
  let variance: f64 = 0.0
  for (let i: u32 = 0; i < msg.values.length; i++) {
    const diff = msg.values[i] - avg
    variance = variance + (diff * diff)
  }
  variance = variance / msg.values.length
  
  emit @metricsResult({
    average: avg,
    weightedAverage: weightedAvg,
    variance: variance
  })
}
```

---

## .fjx — UI компоненты

**Назначение:** Пользовательский интерфейс и визуальные компоненты

**Наследует:** Всё из `.fjs` (и `.fjt`, и `.fjc`)

**Добавляет:**
- **JSX-синтаксис для тегов**
- **Каскадные обработчики событий**
- UI-специфичные конструкции
- Реактивные связи

**Пример:**

```fujin
// TodoApp.fjx
type @div(Div) = {
  class: string
  children: Array<Element>
}

type @input(Input) = {
  value: string
  onChange: action
}

type @button(Button) = {
  text: string
  onClick: action
}

type @todoItem(TodoItem) = { id: u64, text: string }
type @todoAppProps(TodoAppProps) = { todos: Array<TodoItem>, newTodo: string }
type @inputChange(InputChange) = { value: string }
type @buttonClick(ButtonClick) = { itemId?: u64, newTodo?: string }
type @updateState(UpdateState) = { field: string, value: string }
type @addItem(AddItem) = { text: string }
type @removeItem(RemoveItem) = { id: u64 }

actor @TodoApp(msg) {
  <div class="todo-app">
    <input
      value={msg.newTodo}
      onChange=@handleInput
    />
    
    <button onClick=@addTodo>
      Add Todo
    </button>
    
    <ul>
      <for each="todos">
        <li>
          {item.text}
          <button onClick=@deleteTodo>Delete</button>
        </li>
      </for>
    </ul>
  </div>
}

actor @handleInput(msg) {
  emit @updateState({ field: "newTodo", value: msg.value })
}

actor @addTodo(msg) {
  emit @addItem({ text: msg.newTodo })
  emit @updateState({ field: "newTodo", value: "" })
}

actor @deleteTodo(msg) {
  emit @removeItem({ id: msg.itemId })
}
```

---

## Таблица возможностей

| Возможность | .fjc | .fjt | .fjs | .fjx |
|-------------|------|------|------|------|
| Структуры данных | ✓ | ✓ | ✓ | ✓ |
| Акторы | ✓ | ✓ | ✓ | ✓ |
| Целочисленная арифметика | ✓ | ✓ | ✓ | ✓ |
| Float (f32, f64) | ✗ | ✓ | ✓ | ✓ |
| Аннотации @name() | ✓ | ✓ | ✓ | ✓ |
| Сериализация | ✓ | ✓ | ✓ | ✓ |
| Базовые операции | ✓ | ✓ | ✓ | ✓ |
| Расширенные операции | ✗ | ✗ | ✓ | ✓ |
| JSX-теги | ✗ | ✗ | ✗ | ✓ |
| Каскадные обработчики | ✗ | ✗ | ✗ | ✓ |

---

## Когда использовать какой тип

### .fjc — Базовый язык
- Блокчейн смартконтракты
- Финансовые операции
- Консенсус-критичная логика
- Детерминированные вычисления
- Системы с требованием точности
- Минимальная логика без внешних зависимостей

### .fjt — Типы и интерфейсы
- Определение общих типов данных с float
- Описание API между модулями
- Сериализационные схемы
- Интерфейсы для внешних систем
- Типы с плавающей точкой

### .fjs — Скрипты
- Бизнес-логика приложений
- Обработка данных
- Аналитика и вычисления
- Общие скрипты

### .fjx — UI
- Пользовательский интерфейс
- Визуальные компоненты
- Интерактивные элементы
- Декларативное описание UI

---

## Важные правила

1. **Наследование возможностей:**
   - `.fjt` может импортировать структуры из `.fjc`
   - `.fjs` может импортировать из `.fjc` и `.fjt`
   - `.fjx` может импортировать из всех типов

2. **Ограничения базового языка (.fjc):**
   - `.fjc` НЕ может использовать float
   - `.fjc` НЕ может использовать расширенные операции
   - Это гарантирует детерминированность и минимализм

3. **Разделение ответственности:**
   - Детерминированная логика — в `.fjc`
   - Типы с сериализацией — в `.fjt`
   - Бизнес-логика — в `.fjs`
   - UI — в `.fjx`

4. **Компиляция:**
   - `.fjc` компилируется без float
   - `.fjt` добавляет wire format и type hash
   - `.fjs` компилируется с полным набором операций
   - `.fjx` компилируется с JSX и UI-логикой

---

## Примеры миграции между уровнями

### Из .fjc в .fjt (добавление float)

```fujin
// Было в .fjc (без float)
type @metrics(Metrics) = {
  count: u64
  total: u64
}

type @result(Result) = { average: f64 }

actor @metrics(msg) {
  const average = msg.total / msg.count  // целочисленное деление
  emit @result({ average })
}

// Стало в .fjt (с float)
type @metrics(Metrics) = {
  count: u64
  total: f64  // теперь можем использовать float
}

actor @metrics(msg) {
  const average: f64 = msg.total / msg.count  // float деление
  emit @result({ average })
}
```

### Из .fjt в .fjs (добавление расширенных операций)

```fujin
// Было в .fjt (базовые операции)
type @calculateAverage(CalculateAverage) = {
  values: Array<f64>
}

type @result(Result) = { average: f64 }
type @statistics(Statistics) = { average: f64, median: f64 }

actor @calculateAverage(msg) {
  let sum: f64 = 0.0
  for (let i: u32 = 0; i < msg.values.length; i++) {
    sum = sum + msg.values[i]
  }
  emit @result({ average: sum / msg.values.length })
}

// Стало в .fjs (расширенные операции)
type @calculateStatistics(CalculateStatistics) = {
  values: Array<f64>
}

actor @calculateStatistics(msg) {
  const avg = msg.values.reduce((a, b) => a + b) / msg.values.length
  const sorted = msg.values.sort()
  const median = sorted[sorted.length / 2]
  
  emit @statistics({ average: avg, median: median })
}
```

---

## Резюме

- **4 типа файлов** с иерархией наследования
- **`.fjc`** — базовый язык без float (детерминированность)
- **`.fjt`** — типы с float и аннотациями (сериализация)
- **`.fjs`** — скрипты с расширенными операциями (общее назначение)
- **`.fjx`** — UI с JSX (интерфейсы)
- Каждый уровень расширяет предыдущий
- Ограничения `.fjc` обеспечивают надёжность и детерминизм
