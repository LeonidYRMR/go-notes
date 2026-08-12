# 7. Функции

## Базовые формы

```go
func greet(name string) string {
    return "Hello, " + name
}
```

```go
func add(a, b int) int {
    return a + b
}
```

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errDivisionByZero{}
    }
    return a / b, nil
}

type errDivisionByZero struct{}

func (errDivisionByZero) Error() string {
    return "division by zero"
}
```

## Именованные результаты

```go
func split(sum int) (left, right int) {
    left = sum / 2
    right = sum - left
    return
}
```

`return` без аргументов возвращает именованные результаты.

⚠️ Naked return допустим только в коротких и очевидных функциях.

## Variadic

```go
func sum(values ...int) int {
    total := 0
    for _, value := range values {
        total += value
    }
    return total
}

func example() {
    _ = sum(1, 2, 3)

    values := []int{1, 2, 3}
    _ = sum(values...)
}
```

Внутри функции variadic-параметр имеет тип `[]T`.

## Функции как значения, анонимные функции, closure

```go
func apply(value int, fn func(int) int) int {
    return fn(value)
}

func example() {
    double := func(n int) int {
        return n * 2
    }

    result := apply(3, double)
    _ = result
}
```

```go
func counter() func() int {
    n := 0
    return func() int {
        n++
        return n
    }
}
```

## Рекурсия

```go
func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}
```

## `defer`

```go
func example() {
    defer println("third")
    defer println("second")
    defer println("first")
}
```

Вывод: `first`, `second`, `third` — LIFO.

```go
func example() {
    value := 1
    defer println(value) // аргумент вычисляется сейчас
    value = 2
}
```

```go
func useResource() error {
    resource, err := openResource()
    if err != nil {
        return err
    }
    defer resource.Close()

    return resource.Use()
}

type resource struct{}

func openResource() (*resource, error) { return &resource{}, nil }
func (r *resource) Close()             {}
func (r *resource) Use() error         { return nil }
```

## `panic` и `recover`

```go
func mustPositive(n int) {
    if n < 0 {
        panic("negative value")
    }
}
```

```go
func safeCall() {
    defer func() {
        if recovered := recover(); recovered != nil {
            // обработка panic
        }
    }()

    panic("unexpected")
}
```

**Нюанс:** `recover` работает только внутри deferred-функции той же goroutine. Обычные ошибки обычно возвращают как `error`, а не обрабатывают через `panic`.

---

