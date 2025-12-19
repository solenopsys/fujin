# Стек, срезы, буферы — коды 0x18–0x1F

| Код (hex) | Команда       | Параметры                      | Описание                                   |
|-----------|---------------|--------------------------------|--------------------------------------------|
| 0x18      | mov           | src_offset, dst_offset         | Копия скаляра/среза                        |
| 0x19      | alloc_bytes   | len_offset, dst_offset         | Выделить буфер (стек/арена) → SliceRef     |
| 0x1A      | copy_slice    | dst_slice, src_slice           | memcpy                                     |
| 0x1B      | load_byte     | slice_offset, index_offset, dst| Прочитать байт                             |
| 0x1C      | store_byte    | slice_offset, index_offset, src| Записать байт (для своих буферов)          |
| 0x1D–0x1F | —             | —                              | (резерв)                                   |

## Удалённые операции и их реализация

### Конкатенация срезов
**Удалён:** 0x1B `concat_slices`  
**Реализация:**
```
// a + b → result
len_a = get_slice_len(slice_a)
len_b = get_slice_len(slice_b)
total_len = len_a + len_b
alloc_bytes(total_len, result)
copy_slice(result[0..len_a], slice_a)
copy_slice(result[len_a..total_len], slice_b)
```
