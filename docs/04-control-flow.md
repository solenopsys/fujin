# 4. Управляющие конструкции

## Условия

### if-else
```fujin
if (condition) {
  // ...
} else if (other) {
  // ...
} else {
  // ...
}
```

### switch
```fujin
switch (value) {
  case 1:
    // ...
    break
  case 2:
  case 3:
    // fall-through
    break
  default:
    // ...
}
```

## Циклы — ТОЛЬКО for

**Важно:** единственный цикл в Fujin — классический `for`

```fujin
// Базовый
for (let i: i32 = 0; i < 10; i++) {
  // ...
}

// По массиву
for (let i: i32 = 0; i < arr.length; i++) {
  const item = arr[i]
}

// С шагом
for (let i: i32 = 0; i < 100; i += 10) {
  // ...
}
```

Других циклов нет: `while`, `do-while`, `for-of`, `for-in` исключены, чтобы упростить анализ и сделать поведение предсказуемым.

## Управление потоком

```fujin
break      // выход из цикла
continue   // следующая итерация
emit       // отправка сообщения
throw      // выброс исключения
```

## emit — отправка сообщений

Вместо `return` акторы используют `emit` для отправки результатов:

```fujin
actor @processData(msg: Data) {
  if (!msg.valid) {
    emit { type: "error", message: "Invalid data" }
  }
  
  const result = transform(msg)
  emit { type: "success", result }
}
```

## Обработка ошибок

```fujin
try {
  const data = fetchData()
  processData(data)
} catch (e) {
  emit { type: "error", message: e.message }
} finally {
  cleanup()
}
```

Из управляющих конструкций исключены `return`, `with`, `debugger`, метки и `async/await`.
