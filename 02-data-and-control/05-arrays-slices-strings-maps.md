# 5. Array, slice, string и map

## Array

**Синтаксис**

```go
var a [3]int
b := [3]int{1, 2, 3}
c := [...]string{"a", "b"}

_, _, _ = a, b, c
```

Array — значение: присваивание копирует все элементы.

```go
source := [2]int{1, 2}
copyOfSource := source
copyOfSource[0] = 99

_ = source // [1 2]
```

Array имеет тип `[N]T`; slice — представление части массива с типом `[]T`.

## Slice

**Синтаксис**

```go
var s []int
numbers := []int{1, 2, 3}

a := make([]int, 3)
b := make([]int, 2, 8)

_, _, _, _ = s, numbers, a, b
```

```go
s := []int{0, 1, 2, 3, 4}

part := s[1:4]
fromStart := s[:3]
toEnd := s[2:]
limited := s[1:3:4] // full slice expression; cap(limited) == 3

_, _, _, _ = part, fromStart, toEnd, limited
```

```go
s := []int{1, 2}
s = append(s, 3, 4)

dst := make([]int, len(s))
copied := copy(dst, s)

_, _ = dst, copied
```

`append` может выделить новый backing array; всегда сохраняйте возвращённый slice.

## `len`, `cap`, `clear`

```go
s := make([]int, 2, 8)

length := len(s)
capacity := cap(s)

clear(s) // элементы становятся zero values; len не меняется

_, _ = length, capacity
```

Начиная с Go 1.21: `clear(slice)` обнуляет элементы slice.

## `nil slice` и пустой slice

```go
var nilSlice []int
emptySlice := []int{}

fmtLike := len(nilSlice) == 0 && len(emptySlice) == 0
_ = fmtLike
```

Оба имеют длину `0`, по ним можно делать `range`, к ним можно применять `append`. `nilSlice == nil`, `emptySlice != nil`.

## Удаление элемента slice

✅ Начиная с Go 1.21: `slices.Delete` — функция стандартной библиотеки, не синтаксис языка.

```go
import "slices"

func withoutIndex(s []int, i int) []int {
    return slices.Delete(s, i, i+1)
}
```

Вариант без дополнительного импорта:

```go
func withoutIndex(s []int, i int) []int {
    return append(s[:i], s[i+1:]...)
}
```

**Нюанс:** этот вариант сохраняет порядок, но может переиспользовать backing array и удерживать ссылки в его хвосте. Универсальный вариант с очисткой хвоста:

```go
func deleteAt[T any](s []T, i int) []T {
    copy(s[i:], s[i+1:])
    clear(s[len(s)-1:])
    return s[:len(s)-1]
}
```

`slices.Delete` в современных версиях Go очищает ставшие ненужными элементы.

## String

```go
s := "hello"
b := s[1]      // byte
size := len(s) // байты

for i, r := range s {
    _, _ = i, r
}

bytes := []byte(s)
again := string(bytes)

_, _, _, _ = b, size, bytes, again
```

## Map

**Синтаксис**

```go
var m map[string]int
scores := map[string]int{
    "Ann": 10,
    "Bob": 20,
}

users := make(map[string]string)
users["u1"] = "Ada"

_, _ = m, scores
```

```go
value := scores["Ann"] // zero value V, если ключа нет

value, ok := scores["Ann"]
if ok {
    _ = value
}
```

```go
delete(scores, "Ann")
clear(scores)
```

Начиная с Go 1.21: `clear(map)` удаляет все ключи.

**Нюанс:** читать из `nil map` можно; записывать нельзя.

```go
var m map[string]int

value := m["missing"] // 0
_ = value

// m["x"] = 1 // panic
```

Порядок обхода `map` не гарантирован.

## `range`

| Коллекция | Значения |
|---|---|
| array / slice | индекс, элемент |
| string | индекс байта, `rune` |
| map | ключ, значение |
| channel | значения до закрытия |
| integer (Go 1.22+) | числа от `0` до `n-1` |
| iterator function (Go 1.23+) | значения, переданные в `yield` |

```go
array := [2]string{"a", "b"}
slice := []string{"x", "y"}
text := "Go"
m := map[string]int{"a": 1}
ch := make(chan int)
close(ch)

for i, v := range array {
    _, _ = i, v
}

for i, v := range slice {
    _, _ = i, v
}

for byteIndex, r := range text {
    _, _ = byteIndex, r
}

for key, value := range m {
    _, _ = key, value
}

for value := range ch {
    _ = value
}

for i := range 5 { // Go 1.22+: 0, 1, 2, 3, 4
    _ = i
}
```

Начиная с Go 1.23 `range` также работает с iterator functions типов `func(yield func() bool)`, `func(yield func(V) bool)` и `func(yield func(K, V) bool)`. Стандартные типы `iter.Seq[V]` и `iter.Seq2[K, V]` используют две последние формы.

Если нужен только индекс или ключ:

```go
for i := range slice {
    _ = i
}
```

## Адреса элементов при `range`

```go
items := []string{"a", "b"}

for i := range items {
    ptr := &items[i]
    _ = ptr
}
```

⚠️ Переменная `range` не обязана быть адресом элемента коллекции. Если нужен указатель именно на элемент slice, используйте индекс и `&items[i]`.

---

