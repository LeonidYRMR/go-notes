# 12. Generics

Начиная с Go 1.18: type parameters и generics.

## Generic-функция

```go
func First[T any](values []T) T {
    return values[0]
}

func example() {
    value := First([]string{"a", "b"}) // T выводится как string
    _ = value
}
```

## Constraint `comparable`

```go
func Contains[T comparable](values []T, target T) bool {
    for _, value := range values {
        if value == target {
            return true
        }
    }
    return false
}
```

`comparable` разрешает `==` и `!=`.

## Generic-тип

```go
type Pair[T any] struct {
    First  T
    Second T
}

func example() {
    pair := Pair[int]{First: 1, Second: 2}
    _ = pair
}
```

## Constraints через interface

```go
type Number interface {
    ~int | ~int64 | ~float64
}

func Sum[T Number](values []T) T {
    var total T
    for _, value := range values {
        total += value
    }
    return total
}
```

`~int` означает типы, базовый тип которых — `int`, включая пользовательские именованные типы на основе `int`.

Интерфейсы с type elements (`~int`, объединения `|` и `comparable`) используются как constraints. Их нельзя использовать как тип обычной переменной вне constraint-контекста.

```go
type UserID int

func example() {
    total := Sum([]UserID{1, 2, 3})
    _ = total
}
```

---

