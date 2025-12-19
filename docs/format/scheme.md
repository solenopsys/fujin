# Fujin Binary Module Format (динамический Cap I/O)

Бинарный модуль нужен, чтобы рантайм без генерации кода мог принимать/отдавать cap-сообщения и мапить их на IR. Все входы/выходы — динамический Cap (Plan/Registry), внутреннее исполнение — стековое.

## Обзор

```
BinaryModule {
  module_path: string           // путь к исходнику
  messages: Message[]           // сообщения + ссылка на payload plan
  schema: MessageSchema         // описание типов для Plan
  plans: PlanDesc[]             // layout полей (offset/slot_size/kind)
  code_blocks: CodeBlock[]      // байткод
  dispatch: DispatchEntry[]     // message_id -> code_id (+ payload_plan)
  exports: u32[]                // индексы экспортируемых messages
}
```

## 1) Messages

```
Message {
  name: string      // "@ping"
  payload_plan_id: u32 // индекс в plans для payload структуры
}
```

`message_id` = индекс в массиве messages. В emit/call использовать `message_id`, не строку.

## 2) MessageSchema и Plan

Schema описывает типы, Plan даёт готовый layout для динамического сериализатора.

```
MessageSchema {
  structs: SchemaStruct[]
}

SchemaStruct {
  name: string
  fields: SchemaField[]
}

SchemaField {
  name: string
  field_type: string   // u8/u16/u32/u64/i8/i16/i32/i64/f32/f64/bool/string/bytes/array/struct
  type_name: string    // для struct_ref или array elem_type, иначе пусто
}
```

Plan описывает layout по Cap (data_section + pointer_slots + heap):

```
PlanDesc {
  name: string
  data_section_size: u32
  fields: PlanField[]
}

PlanField {
  name: string
  field_type: string   // те же примитивы + string/bytes/array/struct_ref
  type_name: string    // для struct_ref или array elem_type, иначе пусто
  offset: u32          // байтовое смещение в data_section
  slot_size: u16       // размер слота в data_section
  elem_kind: u8        // для array: 0=primitive,1=boolean,2=string,3=bytes,4=struct,5=opaque
}
```

Enum в языке нет, поэтому `field_type`/`elem_kind` не имеют enum-варианта.

## 3) CodeBlock

```
CodeBlock {
  frame_size: i64
  params: Param[]
  locals: Param[]
  operations: Operation[]
}

Param {
  name: string
  type_kind: u8        // 0=void,1=primitive,2=string,3=bytes,4=struct_ref
  type_name: string    // для struct_ref, иначе пусто
  offset: i64
}
```

## 4) Operation

```
Operation {
  kind: u8              // opcode, см. docs/ir/commands_future.md
  op_a: i64             // универсальный параметр A (offset/id/imm)
  op_b: i64             // универсальный параметр B
  op_c: i64             // универсальный параметр C
  op_str: string        // строковый литерал (для const_string)
}

Интерпретация `op_a`/`op_b`/`op_c` зависит от opcode и совпадает с таблицами в `docs/ir/*.md`.
Примеры:
- `load_field_scalar` (0x10): op_a=plan_id, op_b=field_idx, op_c=dst_offset
- `set_field_scalar` (0x29): op_a=builder_slot, op_b=field_idx, op_c=src_offset
- `emit_plan` (0x2F): op_a=builder_slot, op_b=message_id, op_c=plan_id
- `const_i64` (0x30): op_a=imm, op_b=dst_offset
- `branch_if_true` (0x20): op_a=cond_offset, op_b=label
```

## 5) DispatchEntry

```
DispatchEntry {
  message_id: u32       // индекс в messages
  code_id: u32          // индекс в code_blocks
  payload_plan_id: u32  // дублирует messages[message_id].payload_plan_id для удобства
}
```

## 6) Exports

Массив `message_id`, которые доступны извне.

## OpCodes (Operation.kind)

Opcode диапазоны и параметры описаны в `docs/ir/commands_future.md` и разбиты по файлам `docs/ir/*.md`:
- 0x10–0x17: ввод/MessageView (`input.md`) — plan_id/field_idx/offset
- 0x18–0x1F: стек/срезы/буферы (`stack.md`)
- 0x20–0x27: контроль потока (`control.md`)
- 0x28–0x2F: Builder/emit (`output.md`)
- 0x30–0x4F: арифметика/логика (`arith.md`)
- 0x50–0x57: локальные структуры (`struct.md`)

Каждый opcode использует `op_a`/`op_b`/`op_c` как позиционные параметры (offset/plan_id/field_idx/label/imm), см. таблицы.

## Модель выполнения

1. **Вход:** runtime получает `message_id` из транспорта и cap-пакет payload (id внутри payload нет).
2. **Dispatch:** по `message_id` ищет `DispatchEntry`, берёт `code_id` и `payload_plan_id`.
3. **Access:** строит view по `plans[payload_plan_id]` и читает поля напрямую из буфера (zero-copy строки/bytes, без материализации StructValue).
4. **Execute:** выполняет `code_blocks[code_id]`, мапит msg на стек по Param.type_kind/type_name, работает с view.
5. **Output:** `emit`/`call` используют `message_id`; payload формируется по связанному Plan (Builder/serializePlan копирует данные в выходной буфер).

## Пример модуля (логическая структура)

```json
{
  "module_path": "test-data/scripts/ping_echo.fjs",
  "messages": [
    {"name": "@ping", "payload_plan_id": 0}
  ],
  "schema": {
    "structs": [{
      "name": "Ping",
      "fields": [
        {"name": "runId", "field_type": "u64", "type_name": ""},
        {"name": "payload", "field_type": "bytes", "type_name": ""}
      ]
    }]
  },
  "plans": [{
    "name": "Ping",
    "data_section_size": 12, // runId (u64) + ptr slot (u32)
    "fields": [
      {"name": "runId", "field_type": "u64", "type_name": "", "offset": 0, "slot_size": 8, "elem_kind": 0},
      {"name": "payload", "field_type": "bytes", "type_name": "", "offset": 8, "slot_size": 4, "elem_kind": 0}
    ]
  }],
  "code_blocks": [{
    "frame_size": 0,
    "params": [{"name": "msg", "type_kind": 4, "type_name": "Ping", "offset": 0}],
    "locals": [],
    "operations": [
      {"kind": 0x10, "op_a": 0, "op_b": 0, "op_c": 0},   // load_field_scalar(plan=0, field=0 runId) -> slot0
      {"kind": 0x11, "op_a": 0, "op_b": 1, "op_c": 8},   // load_field_slice(plan=0, field=1 payload) -> slot8
      {"kind": 0x28, "op_a": 0, "op_b": 0},              // init_builder(plan=0) -> builder slot0
      {"kind": 0x29, "op_a": 0, "op_b": 0, "op_c": 0},   // set_field_scalar(builder0, field=0, src=slot0)
      {"kind": 0x2A, "op_a": 0, "op_b": 1, "op_c": 8},   // set_field_slice(builder0, field=1, src=slot8)
      {"kind": 0x2F, "op_a": 0, "op_b": 0, "op_c": 0}    // emit_plan(builder0, message_id=0, plan_id=0)
    ]
  }],
  "dispatch": [{
    "message_id": 0,
    "code_id": 0,
    "payload_plan_id": 0
  }],
  "exports": [0]
}
```

## Инструменты анализа

- `binary-to-json` — конвертация бинарника в JSON для отладки.
- План/Registry берутся напрямую из `schema`+`plans`; генерация кода не требуется.

### dump-binary  
Выводит структуру модуля в человекочитаемом виде:
```bash
./zig-out/bin/dump-binary module.bin
```

## Ссылки

- Схема формата: `module.fjt`
- Генерированный код: `src/gen/binary_format.zig`
- Runtime десериализация: `../cap/src/dynamic.zig`
- Interpreter: `../interpreter/src/binary_loader.zig`
