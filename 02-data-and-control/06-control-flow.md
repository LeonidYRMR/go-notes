# 6. Управляющие конструкции

## `if`

```go
if ready {
    // ...
}
```

```go
if score >= 90 {
    // ...
} else if score >= 60 {
    // ...
} else {
    // ...
}
```

```go
if value, ok := m[key]; ok {
    _ = value
}
```

Переменные из инициализации `if` доступны в `if`, `else if` и `else`, но не после конструкции.

## `for`

В Go один оператор цикла — `for`.

```go
for {
    // бесконечный цикл
}
```

```go
for condition {
    // пока condition == true
}
```

```go
for i := 0; i < n; i++ {
    _ = i
}
```

```go
for i, value := range values {
    _, _ = i, value
}
```

## `break`, `continue`, labels, `goto`

```go
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue
    }
    if i == 7 {
        break
    }
}
```

```go
Outer:
    for _, row := range rows {
        for _, cell := range row {
            if cell == "stop" {
                break Outer
            }
        }
    }
}
```

```go
Outer:
    for _, row := range rows {
        for _, cell := range row {
            if cell == "" {
                continue Outer
            }
        }
    }
}
```

```go
func example(ok bool) {
    if !ok {
        goto Done
    }
Done:
}
```

`goto` существует, но подчиняется ограничениям области видимости и используется редко.

## `switch`

```go
switch day {
case "Sat", "Sun":
    // выходной
case "Mon":
    // понедельник
default:
    // всё остальное
}
```

```go
switch value, ok := m[key]; {
case !ok:
    // ключа нет
case value > 0:
    // положительное значение
default:
    // остальные случаи
}
```

```go
switch {
case n < 0:
    // ...
case n == 0:
    // ...
default:
    // ...
}
```

В Go нет неявного `fallthrough`.

```go
switch n {
case 1:
    fallthrough
case 2:
    // выполнится для 1 и 2
}
```

`fallthrough` передаёт управление следующему `case`, игнорируя его условие; используется редко.

## `select`

```go
select {
case value := <-in:
    _ = value
case out <- value:
default:
    // ни одна операция не готова
}
```

```go
var ch chan int // nil

select {
case ch <- 1:
    // эта ветка навсегда отключена
default:
}
```

**Нюанс:** `select` выбирает одну готовую ветку. Если готовы несколько, выбор псевдослучайный. Операции с `nil` channel блокируются навсегда.

---

