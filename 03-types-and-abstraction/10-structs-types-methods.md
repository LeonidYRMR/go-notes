# 10. Struct, type, методы и embedding

## Struct

```go
type User struct {
    ID   int
    Name string
}
```

```go
u1 := User{ID: 1, Name: "Ada"}
u2 := User{1, "Ada"}

_, _ = u1, u2
```

✅ Именованный literal предпочтительнее: устойчив к изменению порядка полей и понятнее.

```go
point := struct {
    X int
    Y int
}{
    X: 10,
    Y: 20,
}

_ = point
```

```go
type Address struct {
    City string
}

type User struct {
    Name    string
    Address Address
}
```

## Struct tags

```go
type User struct {
    Name string `json:"name"`
}
```

Struct tag — строка с метаданными поля; язык сам по себе не интерпретирует содержимое тега.

**Нюанс:** struct сравним через `==`, только если сравнимы все его поля.

## Новый именованный тип и alias

```go
type UserID int   // новый именованный тип
type AccountID = int // alias: то же самое, что int
```

`UserID` несовместим с `int` без явного преобразования. `AccountID` — другое имя того же типа `int`.

## Методы

```go
type Counter struct {
    Value int
}

func (c Counter) String() string {
    return "counter"
}

func (c *Counter) Inc() {
    c.Value++
}
```

```go
func example() {
    c := Counter{}
    c.Inc() // компилятор возьмёт &c автоматически
    _ = c.String()
}
```

| Receiver | Обычно выбирают, когда |
|---|---|
| `func (t T) Method()` | Тип маленький и не меняется |
| `func (t *T) Method()` | Метод меняет receiver; копирование нежелательно; нужен `nil` receiver |

✅ Не смешивайте value и pointer receivers без причины.

Method set кратко:

| Значение | Доступные методы |
|---|---|
| `T` | методы с receiver `T` |
| `*T` | методы с receiver `T` и `*T` |

Это именно method sets. При обычном вызове для addressable-значения компилятор может автоматически взять адрес: `c.Inc()` преобразуется в `(&c).Inc()`. При проверке реализации интерфейса method set применяется строго.

## Embedding

```go
type Logger struct{}

func (Logger) Log(message string) {
    println(message)
}

type Service struct {
    Logger
    Name string
}

func example() {
    s := Service{Name: "api"}
    s.Log("started") // promoted method
}
```

```go
type Config struct {
    Port int
}

type Server struct {
    Config // embedded field
}
```

Embedded поля и методы могут быть promoted. Embedding — не классическое наследование.

---

