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
  kind: u8
  variable: string      // для load_local/store_local
  value_int: i64        // const_i64/const_i32
  value_float: f64      // const_f64
  value_str: string     // const_string
  message_id: u32       // для emit/call: индекс в messages
  arg_count: u32        // для call/emit
}
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

Коды операций остаются прежними:

| Code | Name | Description |
|------|------|-------------|
| 0 | `const_i64` | Push i64 constant |
| 1 | `const_i32` | Push i32 constant |
| 2 | `const_f64` | Push f64 constant |
| 3 | `const_string` | Push string constant |
| 4 | `const_true` | Push boolean true |
| 5 | `const_false` | Push boolean false |
| 6 | `const_null` | Push null |
| 10 | `load_local` | Load local variable |
| 11 | `store_local` | Store to local variable |
| 20 | `add_i64` | i64 addition |
| 21 | `sub_i64` | i64 subtraction |
| 22 | `mul_i64` | i64 multiplication |
| 23 | `div_i64` | i64 division |
| 24 | `mod_i64` | i64 modulo |
| 30 | `add_f64` | f64 addition |
| 31 | `sub_f64` | f64 subtraction |
| 40 | `eq` | Equality (===) |
| 41 | `neq` | Inequality (!==) |
| 42 | `lt` | Less than (<) |
| 43 | `lte` | Less than or equal (<=) |
| 44 | `gt` | Greater than (>) |
| 45 | `gte` | Greater than or equal (>=) |
| 50 | `logical_and` | Logical AND (&&) |
| 51 | `logical_or` | Logical OR (||) |
| 52 | `logical_not` | Logical NOT (!) |
| 60 | `emit` | Emit message to bus (by message_id) |
| 61 | `call` | Function call (by message_id) |
| 62 | `assert` | Assert statement |
| 70 | `return_value` | Return with value |
| 71 | `return_void` | Return void |
| 255 | `unknown` | Unknown operation (error) |

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
      {"kind": 10, "variable": "msg"},  // load_local msg
      {"kind": 60, "message_id": 0, "arg_count": 1} // emit @ping
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
