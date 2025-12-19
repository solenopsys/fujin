# Стек, срезы, буферы — коды 0x18–0x1F

Смотрите также `memory.md` для общих правил памяти/стека (страницы 4 KiB, выделение при запуске актора, сброс после выхода).

| Код (hex) | Команда       | Параметры                      | Описание                                   |
|-----------|---------------|--------------------------------|--------------------------------------------|
| 0x18      | mov           | src_offset, dst_offset         | Копия скаляра/среза                        |
| 0x19      | alloc_bytes   | len_offset, dst_offset         | Выделить буфер (стек/арена) → SliceRef     |
| 0x1A      | copy_slice    | dst_slice, src_slice           | memcpy                                     |
| 0x1B      | load_byte     | slice_offset, index_offset, dst| Прочитать байт                             |
| 0x1C      | store_byte    | slice_offset, index_offset, src| Записать байт (для своих буферов)          |
| 0x1D–0x1F | —             | —                              | (резерв)                                   |

## Конкатенация срезов (рекомендация без отдельного опкода)

Отдельного `concat` опкода нет. Конкатенировать строки/байты/массивы нужно вручную:
```
len_a = slice_len(slice_a)
len_b = slice_len(slice_b)
total = len_a + len_b
alloc_bytes(total, result)         // создаём новый буфер
copy_slice(result[0..len_a], slice_a)
copy_slice(result[len_a..total], slice_b)
```
Это гарантирует отсутствие скрытых копий и явное управление выделениями. Для массивов примитивов `elem_size` берётся из SliceRef; `copy_slice` копирует ровно `len * elem_size` байт.

## Семантика SliceRef (строки/байты/массивы)

- SliceRef — это троица `(ptr, len, elem_size)`; `mov` копирует только метаданные, данные не дублируются.
- Для входных сообщений `ptr` указывает в payload Cap, работы ведутся zero-copy.
- `slice_len` (0x12) читает `len` из SliceRef; в циклах это единственный источник границ.
- `load_elem` (0x13) опирается на `elem_size` внутри SliceRef и адресует `ptr + idx * elem_size`; компилятор обязан гарантировать `idx < len`.
- Собственные буферы появляются только через `alloc_bytes` → SliceRef; после этого разрешены `store_byte`/`copy_slice`.
