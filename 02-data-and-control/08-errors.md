# 8. Ошибки: только языковой минимум

## `error`

```go
type error interface {
    Error() string
}
```

`error` — предопределённый интерфейс.

## Возврат ошибки

```go
func operation() (string, error) {
    return "result", nil
}
```

```go
func process() error {
    value, err := operation()
    if err != nil {
        return err
    }

    _ = value
    return nil
}
```

```go
func load() (int, error) {
    value, err := readNumber()
    if err != nil {
        return 0, err
    }
    return value, nil
}

func readNumber() (int, error) {
    return 42, nil
}
```

✅ Ошибка проверяется явно и сразу после вызова. Go не использует exceptions как основной механизм ошибок.

---

