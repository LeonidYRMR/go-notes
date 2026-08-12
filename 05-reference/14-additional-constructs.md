# 14. Дополнительные важные конструкции

## `nil`

`nil` применим к следующим типам:

| Тип | Можно сравнивать с `nil` |
|---|---|
| pointer | да |
| slice | да |
| map | да |
| channel | да |
| function | да |
| interface | да |

```go
var p *int
var s []int
var m map[string]int
var ch chan int
var fn func()
var x any

_, _, _, _, _, _ = p == nil, s == nil, m == nil, ch == nil, fn == nil, x == nil
```

## `new` и `make`

```go
p := new(int) // *int; значение int равно 0
initialized := new(42) // *int со значением 42; Go 1.26+
*p = 42

s := make([]int, 3)
m := make(map[string]int)
ch := make(chan int)

_, _, _, _, _ = p, initialized, s, m, ch
```

| Конструкция | Назначение |
|---|---|
| `new(T)` | выделяет zero value `T`, возвращает `*T` |
| `new(expr)` | создаёт переменную со значением выражения, возвращает указатель; Go 1.26+ |
| `make(T, ...)` | инициализирует slice, map или channel; возвращает сам `T` |

## `delete`, `clear`, `close`

```go
m := map[string]int{"a": 1}
delete(m, "a")
clear(m)

ch := make(chan int)
close(ch)
```

| Функция | Применяется к |
|---|---|
| `delete(m, key)` | map |
| `clear(x)` | map или slice |
| `close(ch)` | channel |

## `any` и `comparable`

```go
func Print[T any](value T) {
    _ = value
}

func Equal[T comparable](a, b T) bool {
    return a == b
}
```

## `unsafe`

Пакет `unsafe` предназначен для низкоуровневых операций и намеренно не разбирается в этой шпаргалке.

---

