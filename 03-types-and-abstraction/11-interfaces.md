# 11. Интерфейсы

## Объявление и неявная реализация

```go
type Writer interface {
    Write([]byte) error
}

type Buffer struct{}

func (Buffer) Write(data []byte) error {
    _ = data
    return nil
}

func useWriter(w Writer) error {
    return w.Write([]byte("data"))
}
```

Компиляционная проверка реализации интерфейса:

```go
var _ Writer = (*Buffer)(nil)
```

Тип реализует интерфейс неявно: достаточно иметь нужный method set.

✅ Предпочитайте маленькие интерфейсы, определённые ближе к месту использования.

## `any`

```go
var value any = 42
var oldStyle interface{} = "text"

_, _ = value, oldStyle
```

Начиная с Go 1.18: `any` — alias для `interface{}`.

## Type assertion

```go
var x any = "Go"

s := x.(string)
_ = s
```

⚠️ Assertion без проверки вызывает panic, если динамический тип не совпадает.

```go
var x any = "Go"

s, ok := x.(string)
if ok {
    _ = s
}
```

## Type switch

```go
func describe(value any) string {
    switch v := value.(type) {
    case string:
        return v
    case int:
        return "integer"
    case nil:
        return "nil"
    default:
        return "other"
    }
}
```

## `nil interface` и typed nil

```go
package main

import "fmt"

type MyType struct{}

func main() {
    var p *MyType = nil
    var x any = p

    fmt.Println(p == nil) // true
    fmt.Println(x == nil) // false
}
```

Интерфейс содержит динамический тип и динамическое значение. У `x` есть динамический тип `*MyType`, даже если его значение `nil`.

---

