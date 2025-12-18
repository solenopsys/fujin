# Команды IR (план) — 1 байт, блоки по 8

Цель: актор читает Cap payload zero-copy, работает в плоском стеке (скаляры + SliceRef), отправляет Cap сообщения через builder с явным `target`.

Категории:
- 0x10–0x17: Ввод / MessageView — см. `input.md`
- 0x18–0x1F: Стек, срезы, буферы — см. `stack.md`
- 0x20–0x27: Контроль потока — см. `control.md`
- 0x28–0x2F: Выход / Builder / emit — см. `output.md`
- 0x30–0x4F: Арифметика и логика — см. `arith.md`
- 0x50–0x57: Локальные структуры — см. `struct.md`
