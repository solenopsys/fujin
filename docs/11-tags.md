# 11. Теги

## Что такое теги

**Теги** — это обычные акторы, записанные в JSX-подобном синтаксисе для описания UI компонентов. Теги используются только в `.fjx` файлах и компилируются в вызовы акторов.

**Важно:**
- Теги - это не специальная конструкция языка, а синтаксический сахар
- Теги компилируются в обычные вызовы акторов с `emit`
- Акторы, вызываемые тегами, используют аннотированные типы (как и все акторы)
- Теги существуют только в `.fjx` файлах для удобства написания UI

---

## FJX-синтаксис в .fjx файлах

В файлах `.fjx` можно использовать JSX-подобный синтаксис для описания UI:

```fujin
actor RedButton {
  <button
    class="px-4 py-2 bg-blue-600 text-white rounded"
    onClick=@click
  >
    Press
  </button>
}
```

Этот код компилируется в:

```fujin
actor RedButton {
  emit @button {
    class: "px-4 py-2 bg-blue-600 text-white rounded",
    onClick: @click,
    children: ["Press"]
  }
}
```

---

## Как работают теги

### 1. Определяем аннотированный тип

```fujin
// types.fjt
type @button(Button) = {
  class: string
  onClick: action
  children: Array<string>
}
```

### 2. Создаём актор-обработчик

```fujin
// button-actor.fjs
actor @button(msg: Button) {
  emit @render {
    tag: "button",
    class: msg.class,
    children: msg.children
  }
  
  // Регистрируем обработчик клика
  registerHandler(msg.onClick)
}
```

### 3. Используем тег в .fjx файле

```fujin
// component.fjx
actor MyComponent {
  <button class="btn-primary" onClick=@handleClick>
    Click me
  </button>
}
```

### 4. Компиляция

Компилятор преобразует JSX в вызов актора:

```fujin
actor MyComponent {
  emit @button {
    class: "btn-primary",
    onClick: @handleClick,
    children: ["Click me"]
  }
}
```

---

## Вложенные теги

FJX поддерживает вложенность:

```fujin
actor Panel {
  <div class="panel">
    <h1>Title</h1>
    <p>Content here</p>
    <button onClick=@save>Save</button>
  </div>
}
```

Компилируется в:

```fujin
actor Panel {
  emit @div {
    class: "panel",
    children: [
      { tag: @h1, children: ["Title"] },
      { tag: @p, children: ["Content here"] },
      { tag: @button, onClick: @save, children: ["Save"] }
    ]
  }
}
```

---

## Обработчики событий

Обработчики событий - это ссылки на акторы через `action`:

```fujin
// Тип события
type @click(ClickEvent) = {
  id: string
  x: i32
  y: i32
}

// Актор-обработчик
actor @click(msg: ClickEvent) {
  emit @log { message: `Clicked at ${msg.x}, ${msg.y}` }
}

// Использование в теге
actor Button {
  <button id="btn-1" onClick=@click>
    Click me
  </button>
}
```

---

## Динамическое содержимое

### Циклы в JSX

```fujin
actor TodoList {
  <ul>
    <for each="items">
      <li bind="items.{id}">{name}</li>
    </for>
  </ul>
}
```

Компилируется в:

```fujin
actor TodoList {
  emit @ul {
    children: state("items").map(item => ({
      tag: @li,
      bind: `items.${item.id}`,
      children: [item.name]
    }))
  }
}
```

### Условный рендеринг

```fujin
actor ConditionalContent {
  <div>
    {if (state.showMessage)} {
      <p>Message is visible</p>
    }
  </div>
}
```

---

## Определение акторов для тегов

Акторы для тегов определяются так же, как и любые другие акторы:

```fujin
// types.fjt
type @div(Div) = {
  id: string
  class: string
  children: Array<Element>
}

type @span(Span) = {
  id: string
  text: string
}

type Element = Div | Span | Button

// actors.fjs
actor @div(msg: Div) {
  emit @render {
    tag: "div",
    id: msg.id,
    class: msg.class
  }
  
  for (child in msg.children) {
    emit child.tag { ...child, parent: msg.id }
  }
}

actor @span(msg: Span) {
  emit @render {
    tag: "span",
    id: msg.id,
    text: msg.text
  }
}
```

---

## Множественные теги в одном акторе

Можно обрабатывать несколько тегов одним актором:

```fujin
type @div(Div) = {
  id: string
  class: string
  children: Array<Element>
}

type @section(Section) = {
  id: string
  class: string
  children: Array<Element>
}

type @article(Article) = {
  id: string
  class: string
  children: Array<Element>
}

// Один актор для всех трёх тегов
actor @div | @section | @article (msg) {
  emit @render {
    tag: @.name,  // имя текущего тега
    id: msg.id,
    class: msg.class
  }
  
  for (child in msg.children) {
    emit child.tag { ...child, parent: msg.id }
  }
}
```

`@.name` — специальная переменная, содержащая имя текущего тега (`"div"`, `"section"` или `"article"`).

---

## Особенности тегов

### Что важно понимать

- **Теги - это синтаксический сахар** для вызова акторов
- **Теги работают только в `.fjx` файлах**
- **Под капотом это обычные `emit` вызовы**
- **Акторы для тегов используют аннотированные типы** (как и все акторы)
- **Нет магии** — всё компилируется в обычный код

### Что можно

- Использовать JSX-синтаксис в `.fjx` файлах
- Вкладывать теги друг в друга
- Использовать `action` для обработчиков событий
- Определять один актор для нескольких тегов
- Использовать циклы и условия в JSX

### Что нельзя

- Использовать JSX-синтаксис вне `.fjx` файлов
- Создавать "специальные" теги — это всегда обычные акторы
- Смешивать бизнес-логику с UI-логикой

---

## Пример: TodoApp

```fujin
// types.fjt
type @div(Div) = {
  class: string
  children: Array<Element>
}

type @input(Input) = {
  value: string
  onChange: action
}

type @ul(Ul) = {
  children: Array<Li>
}

type @li(Li) = {
  text: string
  onDelete: action
}

type @button(Button) = {
  text: string
  onClick: action
}

type Element = Div | Input | Ul | Li | Button

// TodoApp.fjx
actor TodoApp {
  <div class="app">
    <input
      value={state.newTodo}
      onChange=@handleInput
    />
    
    <ul>
      <loop each="todos">
        <li
          text={item.text}
          onDelete=@deleteTodo
        />
      </for>
    </ul>
  </div>
}

actor @handleInput(msg: InputChange) {
  emit @updateState { field: "newTodo", value: msg.value }
}

actor @deleteTodo(msg: DeleteEvent) {
  emit @removeItem { id: msg.itemId }
}
```

Этот FJX компилируется в серию `emit` вызовов к соответствующим акторам.

---

## Резюме

- **Теги — это JSX-синтаксис для акторов** в `.fjx` файлах
- Теги компилируются в обычные `emit` вызовы
- Акторы для тегов используют аннотированные типы
- Нет разницы между "тегом" и "актором" — теги это синтаксический сахар
- FJX делает UI-код более читаемым и декларативным
- Под капотом всё работает через message bus и асинхронные вызовы
