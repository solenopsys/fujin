# Стек, срезы, буферы — коды 0x18–0x1F

| Код (hex) | Команда       | Параметры                      | Описание                                   |
|-----------|---------------|--------------------------------|--------------------------------------------|
| 0x18      | mov           | src_offset, dst_offset         | Копия скаляра/среза                        |
| 0x19      | alloc_bytes   | len_offset, dst_offset         | Выделить буфер (стек/арена) → SliceRef     |
| 0x1A      | copy_slice    | dst_slice, src_slice           | memcpy                                     |
| 0x1B      | concat_slices | slice_a, slice_b, dst_offset   | Новый буфер a+b                            |
| 0x1C      | load_byte     | slice_offset, index_offset, dst| Прочитать байт                             |
| 0x1D      | store_byte    | slice_offset, index_offset, src| Записать байт (для своих буферов)          |
| 0x1E–0x1F | —             | —                              | (резерв)                                   |
