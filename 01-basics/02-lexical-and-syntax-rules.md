# 2. Лексика и базовые правила синтаксиса

## Ключевые слова Go

```text
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

Это именно ключевые слова языка. `len`, `make`, `append` и подобные — не ключевые слова.

## Предопределённые константы и идентификаторы

| Категория | Имена |
|---|---|
| Предопределённые константы | `true`, `false`, `iota` |
| Предопределённое значение | `nil` — не является константой |
| Предопределённые типы / идентификаторы | `any`, `comparable`, `error` |
| Встроенные функции | `append`, `cap`, `clear`, `close`, `complex`, `copy`, `delete`, `imag`, `len`, `make`, `max`, `min`, `new`, `panic`, `print`, `println`, `real`, `recover` |

Начиная с Go 1.18: `any` — alias для `interface{}`, `comparable` — constraint для сравнимых типов.  
Начиная с Go 1.21: встроенные `min`, `max`, `clear`.

`print` и `println` предназначены в основном для отладки; в прикладном коде обычно используют пакет `fmt`.

## Неявные `;`

Точка с запятой обычно не пишется: компилятор вставляет её в конце строки после идентификатора, литерала, `break`, `continue`, `fallthrough`, `return`, `++`, `--`, `)`, `]`, `}`.

⚠️ Открывающая `{` должна быть на той же строке:

```go
if ok {
    // ...
}
```

Не так:

```go
// if ok
// {
//     // ошибка синтаксиса
// }
```

⚠️ Нельзя переносить строку перед точкой:

```go
value.
    Method()
```

## Blank identifier `_`

```go
value, _ := readValue()
_ = value // подавляет ошибку "declared and not used"
```

`_` принимает значение, но не создаёт переменную и не может быть прочитан.

## Zero values

| Тип | Zero value |
|---|---|
| Числа | `0` |
| `bool` | `false` |
| `string` | `""` |
| pointer, map, slice, channel, function, interface | `nil` |
| struct | zero value каждого поля |
| array | zero value каждого элемента |

## Область видимости

```go
if true {
    value := 1
    _ = value
}
// value здесь недоступен
```

Область видимости определяется блоками `{ ... }`.

## Значения и указатели

Go передаёт аргументы по значению. Передача указателя позволяет менять исходное значение и избегать копирования больших структур.

---

