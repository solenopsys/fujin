# Арифметика и логика — коды 0x30–0x4F

| Код (hex) | Команда    | Параметры        | Описание            |
|-----------|------------|------------------|---------------------|
| 0x30      | const_i64  | imm, dst         | Константа i64       |
| 0x31      | const_i32  | imm, dst         | Константа i32       |
| 0x32      | const_f64  | imm, dst         | Константа f64       |
| 0x33      | const_bool | imm, dst         | Константа bool      |
| 0x34–0x37 | —          | —                | (резерв)            |
| 0x38      | add        | left, right, dst | Сложение            |
| 0x39      | sub        | left, right, dst | Вычитание           |
| 0x3A      | mul        | left, right, dst | Умножение           |
| 0x3B      | div        | left, right, dst | Деление             |
| 0x3C      | mod        | left, right, dst | Остаток             |
| 0x3D–0x3F | —          | —                | (резерв)            |
| 0x40      | eq         | left, right, dst | Равно               |
| 0x41      | neq        | left, right, dst | Не равно            |
| 0x42      | lt         | left, right, dst | Меньше              |
| 0x43      | lte        | left, right, dst | Меньше либо равно   |
| 0x44      | gt         | left, right, dst | Больше              |
| 0x45      | gte        | left, right, dst | Больше либо равно   |
| 0x46      | and        | left, right, dst | Логическое И        |
| 0x47      | or         | left, right, dst | Логическое ИЛИ      |
| 0x48      | not        | left, dst        | Логическое НЕ       |
| 0x49      | neg        | left, dst        | Унарный минус       |
| 0x4A–0x4F | —          | —                | (резерв)            |

## Удалённые операции и их реализация в компиляторе

### Унарный плюс (`+x`)
**Удалён:** 0x4A `plus`  
**Реализация:** `mov src, dst` - простое копирование значения

### Инкремент/декремент
**Удалёны:** 0x4B `inc`, 0x4C `dec`, 0x4D `post_inc`, 0x4E `post_dec`  
**Реализация:**
- `++x` → `const_i64(1, tmp); add(x, tmp, x)`
- `x++` → `mov(x, result); const_i64(1, tmp); add(x, tmp, x)`
- `--x` → `const_i64(1, tmp); sub(x, tmp, x)`
- `x--` → `mov(x, result); const_i64(1, tmp); sub(x, tmp, x)`

### Степень
**Удалён:** 0x3D `pow`  
**Реализация:** Через вызов библиотечной функции `pow(base, exp)` или цикл умножения для целых степеней
