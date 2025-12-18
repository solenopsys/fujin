# Выход / Builder / emit — коды 0x28–0x2F

| Код (hex) | Команда          | Параметры                                 | Описание                                      |
|-----------|------------------|-------------------------------------------|-----------------------------------------------|
| 0x28      | init_builder     | plan_id, dst_builder                      | Создать/сбросить builder                      |
| 0x29      | set_field_scalar | builder, field_idx, src_offset            | Поле-скаляр                                   |
| 0x2A      | set_field_slice  | builder, field_idx, src_offset            | Поле-строка/bytes/slice                       |
| 0x2B      | list_start       | builder, field_idx                        | Начать список                                 |
| 0x2C      | list_push_scalar | builder, src_offset                       | Добавить скаляр в текущий список              |
| 0x2D      | list_push_slice  | builder, src_offset                       | Добавить slice в текущий список               |
| 0x2E      | list_end         | builder                                   | Завершить список                              |
| 0x2F      | emit_plan        | builder, message_id, plan_id, target      | Финализировать и отправить Cap payload        |
