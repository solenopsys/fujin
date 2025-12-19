# Ввод / MessageView — коды 0x10–0x17

| Код (hex) | Команда            | Параметры                          | Описание                                      |
|-----------|--------------------|------------------------------------|-----------------------------------------------|
| 0x10      | load_field_scalar  | plan_id, field_idx, dst_offset     | Скаляр из входного payload → слот             |
| 0x11      | load_field_slice   | plan_id, field_idx, dst_offset     | Строка/bytes/массив как SliceRef (zero-copy)  |
| 0x12      | slice_len          | slice_offset, dst_offset           | Длина среза/массива                           |
| 0x13      | load_elem          | slice_offset, index_offset, dst, elem_kind | Элемент массива примитивов (elem_kind imm)    |
| 0x14      | slice_sub          | slice_offset, off, len, dst_offset | Подсрез (байты/строки)                        |
| 0x15–0x17 | —                  | —                                  | (резерв)                                      |

## Zero-copy семантика входных срезов

- `load_field_slice` возвращает SliceRef, указывающий прямо в payload Cap: `(ptr, len, elem_size)`, без копирования.
- Граничные проверки выполняются в коде, использующем SliceRef (`slice_len` для длины, сравнение перед `load_elem`/`load_byte`).
- При передаче SliceRef в другие слоты/акторы данные по адресу `ptr` не копируются; копия нужна только если явно вызывается `copy_slice` с предварительным `alloc_bytes`.
