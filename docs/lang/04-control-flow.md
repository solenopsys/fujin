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

Вместо `return` акторы используют `emit` для отправки результатов с аннотированными типами:

```fujin
type @processDataIn(ProcessDataIn) = { valid: bool, payload: Payload }
type @processOk(ProcessOk) = { type: "success", result: Result }
type @processError(ProcessError) = { type: "error", message: string }

actor @processData(msg) {
  if (!msg.valid) {
    emit @processError({ type: "error", message: "Invalid data" })
  } else {
    const result = transform(msg.payload)
    emit @processOk({ type: "success", result })
  }
}
```

## Обработка ошибок

```fujin
try {
  const data = fetchData()
  processData(data)
} catch (e) {
  emit @processError({ type: "error", message: e.message })
} finally {
  cleanup()
}
```

Из управляющих конструкций исключены `return`, `with`, `debugger`, метки и `async/await`.
