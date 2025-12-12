# 5. Структуры данных

## Массивы

```fujin
// Объявление
const arr: i32[] = [1, 2, 3]
const matrix: i32[][] = [[1, 2], [3, 4]]

// Доступ
const first = arr[0]
arr[1] = 42
const len = arr.length
```

### Методы массивов (минимальный набор)

**Базовые операции:**
```fujin
push(item)      // добавить в конец
pop()           // удалить из конца
length          // размер массива
slice(start, end) // срез массива
```

**Все остальное** (`map`, `filter`, `reduce`, `forEach`, `find`, `sort`, etc.) — **реализовать в библиотеках**. Это упрощает компилятор.

## Объекты (Plain Data)

**Важно:** только plain data (POJO), нет методов, нет прототипов

```fujin
// Объявление
const point = { x: 10, y: 20 }

// Доступ
point.x
point["y"]

// Деструктуризация
const { x, y } = point
```

Spread оператор (`{ ...point, z: 30 }`) не поддерживается.

## Кортежи

```fujin
const pair: [i32, string] = [42, "answer"]
const [num, str] = pair
```

## Строки

```fujin
// Литералы
const s1 = "hello"
const s2 = 'world'
```

Template literals `` `Hello ${name}` `` отсутствуют.

### Методы строк (минимальный набор)

```fujin
length          // размер строки
slice(start, end) // срез
indexOf(sub)    // поиск подстроки
split(sep)      // разделение на массив
concat(other)   // конкатенация
```

**Все остальное** (`toLowerCase`, `toUpperCase`, `trim`, `replace`, `startsWith`, etc.) — **в библиотеках**.

`Map`, `Set`, `WeakMap`, `WeakSet`, прототипы (`Array.prototype`, `String.prototype`, `Object.prototype`) и методы на объектах отсутствуют в языке.
