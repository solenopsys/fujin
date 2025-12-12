# 9. Zero-Runtime модель

## Принципы

1. **Никаких скрытых аллокаций** — все выделения памяти явные
2. **Никаких виртуальных таблиц** — нет динамической диспетчеризации
3. **Никакого GC** — управление памятью контролируется
4. **Детерминированность** — предсказуемое поведение

## Lowering цепочка

```
Fujin Source (.fjs/.fjt)
  ↓
Fujin AST
  ↓
Fujin IR
  ↓
WASM
  ↓
Actor Runtime
```

## Пример IR

**Исходный код:**
```fujin
actor processTask(task: Task) {
  task.status = "processing"
  const output = task.payload + "_done"
  
  emit {
    type: "result",
    id: task.id,
    status: task.status,
    payload: output
  }
}
```

**Концептуальный IR:**
```
actor processTask(ptr_task):
  load task.status
  store "processing"
  
  load task.payload
  append "_done"
  
  alloc message
  store "result"
  store task.id
  store task.status
  store payload
  emit message
```

## Строгая память

- Контроль аллокаций
- Предсказуемое управление ресурсами
- Статический анализ времени жизни
- Нет неявных конверсий

## WASM runtime

- Минимальный рантайм
- Без JS зависимостей
- Actor-based execution
- Контролируемый планировщик
