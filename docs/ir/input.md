# Ввод / MessageView — коды 0x10–0x17

| Код (hex) | Команда            | Параметры                          | Описание                                      |
|-----------|--------------------|------------------------------------|-----------------------------------------------|
| 0x10      | load_field_scalar  | plan_id, field_idx, dst_offset     | Скаляр из входного payload → слот             |
| 0x11      | load_field_slice   | plan_id, field_idx, dst_offset     | Строка/bytes/массив как SliceRef (zero-copy)  |
| 0x12      | slice_len          | slice_offset, dst_offset           | Длина среза/массива                           |
| 0x13      | load_elem          | slice_offset, index_offset, dst, elem_kind | Элемент массива примитивов (elem_kind imm)    |
| 0x14      | slice_sub          | slice_offset, off, len, dst_offset | Подсрез (байты/строки)                        |
| 0x15–0x17 | —                  | —                                  | (резерв)                                      |
