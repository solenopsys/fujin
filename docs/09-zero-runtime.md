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
actor @processTask(msg: Task) {
  msg.status = "processing"
  const output = msg.payload + "_done"
  
  emit {
    type: "result",
    id: msg.id,
    status: msg.status,
    payload: output
  }
}
```

**Концептуальный IR:**
```
actor @processTask(ptr_msg):
  load msg.status
  store "processing"
  
  load msg.payload
  append "_done"
  
  alloc message
  store "result"
  store msg.id
  store msg.status
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
