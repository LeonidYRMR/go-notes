# 13. Конкурентность

## Goroutine

```go
go work()
```

```go
go func(value int) {
    use(value)
}(42)

func work()      {}
func use(int)    {}
```

Goroutine — лёгкая единица конкурентного выполнения, не обязательно системный поток.

## Безопасная передача переменной цикла

```go
func processAll(items []string) {
    for _, item := range items {
        go func(item string) {
            use(item)
        }(item)
    }
}

func use(string) {}
```

Начиная с Go 1.25 запуск и учёт goroutine можно объединить через `WaitGroup.Go`:

```go
var wg sync.WaitGroup

for _, item := range items {
    wg.Go(func() {
        use(item)
    })
}

wg.Wait()
```

Передача параметром явно фиксирует значение для goroutine и совместима со старыми версиями Go. Начиная с Go 1.22 переменные, объявленные циклом через `:=`, создаются заново для каждой итерации, поэтому захват `item` замыканием также безопасен.

## Channel

```go
ch := make(chan int)    // unbuffered
buffered := make(chan int, 10) // buffered

_, _ = ch, buffered
```

```go
ch := make(chan string)

go func() {
    ch <- "result"
}()

value := <-ch
_ = value
```

```go
var sendOnly chan<- int
var receiveOnly <-chan int

_, _ = sendOnly, receiveOnly
```

| Тип | Разрешённая операция |
|---|---|
| `chan T` | send и receive |
| `chan<- T` | только send |
| `<-chan T` | только receive |

## Закрытие канала

```go
ch := make(chan int)
close(ch)

value, ok := <-ch
_ = value
_ = ok // false: канал закрыт и буфер пуст
```

Чтение из закрытого канала возвращает zero value и `ok == false`, когда буфер опустел.

```go
ch := make(chan int)

go func() {
    defer close(ch) // обычно закрывает отправитель
    for _, value := range []int{1, 2, 3} {
        ch <- value
    }
}()

for value := range ch {
    _ = value
}
```

✅ Обычно канал закрывает отправитель, а не получатель. Закрытие закрытого канала вызывает panic.

## `select` для каналов

```go
func receiveEither(left, right <-chan string) string {
    select {
    case value := <-left:
        return value
    case value := <-right:
        return value
    }
}
```

```go
select {
case value := <-ch:
    _ = value
case ch <- 42:
default:
    // нет готовых операций
}
```

```go
var optional <-chan int = nil

select {
case value := <-optional:
    _ = value // ветка отключена
default:
}
```

## `sync.WaitGroup`

```go
package main

import "sync"

func main() {
    var wg sync.WaitGroup
    items := []string{"a", "b", "c"}

    wg.Add(len(items))

    for _, item := range items {
        go func(item string) {
            defer wg.Done()
            use(item)
        }(item)
    }

    wg.Wait()
}

func use(string) {}
```

---

