# Контроль потока — коды 0x20–0x27

| Код (hex) | Команда         | Параметры          | Описание                      |
|-----------|-----------------|--------------------|-------------------------------|
| 0x20      | branch_if_true  | cond_offset, label | Условный переход (если true)  |
| 0x21      | branch_if_false | cond_offset, label | Условный переход (если false) |
| 0x22      | jump            | label              | Безусловный переход           |
| 0x23      | return_value    | src_offset         | Возврат значения              |
| 0x24      | return_void     | —                  | Возврат void                  |
| 0x25      | assert          | cond_offset        | Ошибка, если false            |
| 0x26–0x27 | —               | —                  | (резерв)                      |
