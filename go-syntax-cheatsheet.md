# Go: синтаксис — краткая шпаргалка

Актуально для Go 1.26. Конструкции, добавленные в недавних версиях Go, помечены минимальной версией.

`✅` — рекомендуемый / идиоматичный вариант  
`⚠️` — важный нюанс или частая ошибка  
`Нюанс:` — правило или поведение, которое стоит запомнить.

## 1. Структура Go-файла и пакеты

### Минимальный файл

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go")
}
```

`package` должен быть первым объявлением файла (после комментариев). Исполняемая программа начинается с `func main()` в пакете `main`. Функция `main` не принимает параметров и ничего не возвращает. Перед ней выполняются инициализация пакетных переменных и функции `init`. После возврата из `main` программа завершается, не ожидая остальные goroutine.

### Импорты

```go
import "fmt"
```

```go
import (
    "fmt"
    "strings"
)
```

```go
import f "fmt"

func example() {
    f.Println("alias")
}
```

```go
import _ "example.com/project/driver"
```

`_` импортирует пакет только ради его `init()`; имя пакета недоступно.

```go
import . "fmt"

func example() {
    Println("обычно избегают")
}
```

⚠️ Dot import (`.`) переносит экспортируемые имена пакета в текущую область видимости. Обычно не рекомендуется: ухудшает читаемость и может создавать конфликты имён.

### `init`

```go
package example

func init() {
    // Выполняется до main().
}
```

`init()` не принимает параметров и ничего не возвращает. В одном пакете может быть несколько `init()`.

### Экспортируемые имена

```go
package example

var PublicValue = 1 // exported
var privateValue = 2 // package-private

type User struct{} // exported
type order struct{} // package-private

func Build() {} // exported
func parse() {} // package-private
```

| Начало идентификатора | Видимость |
|---|---|
| Заглавная буква Unicode | экспортируется из пакета |
| Строчная буква | доступен только внутри пакета |

### Комментарии

```go
// Обычный комментарий.

/*
Многострочный
комментарий.
*/
```

```go
// User представляет пользователя.
type User struct {
    Name string
}
```

Doc comment обычно начинается с имени экспортируемой сущности.

```go
// Package example предоставляет примеры.
package example
```

Комментарий пакета размещают непосредственно перед `package`.

### Имена файлов

Файлы одного каталога обычно принадлежат одному пакету. Имя файла — `snake_case.go`; специальные суффиксы: `*_test.go`, `*_linux.go`, `*_amd64.go`.

---

## 2. Лексика и базовые правила синтаксиса

### Ключевые слова Go

```text
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

Это именно ключевые слова языка. `len`, `make`, `append` и подобные — не ключевые слова.

### Предопределённые константы и идентификаторы

| Категория | Имена |
|---|---|
| Предопределённые константы | `true`, `false`, `iota` |
| Предопределённое значение | `nil` — не является константой |
| Предопределённые типы / идентификаторы | `any`, `comparable`, `error` |
| Встроенные функции | `append`, `cap`, `clear`, `close`, `complex`, `copy`, `delete`, `imag`, `len`, `make`, `max`, `min`, `new`, `panic`, `print`, `println`, `real`, `recover` |

Начиная с Go 1.18: `any` — alias для `interface{}`, `comparable` — constraint для сравнимых типов.  
Начиная с Go 1.21: встроенные `min`, `max`, `clear`.

`print` и `println` предназначены в основном для отладки; в прикладном коде обычно используют пакет `fmt`.

### Неявные `;`

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

### Blank identifier `_`

```go
value, _ := readValue()
_ = value // подавляет ошибку "declared and not used"
```

`_` принимает значение, но не создаёт переменную и не может быть прочитан.

### Zero values

| Тип | Zero value |
|---|---|
| Числа | `0` |
| `bool` | `false` |
| `string` | `""` |
| pointer, map, slice, channel, function, interface | `nil` |
| struct | zero value каждого поля |
| array | zero value каждого элемента |

### Область видимости

```go
if true {
    value := 1
    _ = value
}
// value здесь недоступен
```

Область видимости определяется блоками `{ ... }`.

### Значения и указатели

Go передаёт аргументы по значению. Передача указателя позволяет менять исходное значение и избегать копирования больших структур.

---

## 3. Переменные и константы

### `var`

**Синтаксис**

```go
var count int
var name = "Go"

var (
    host string = "localhost"
    port        = 8080
)

var x, y int
var left, right = 1, 2
```

`var` допустим на уровне пакета и внутри функций.

### Короткое объявление `:=`

```go
func example() {
    name := "Ada"
    count := 3
    _, _ = name, count
}
```

**Нюанс:** `:=` работает только внутри функций. Хотя бы одна переменная слева должна быть новой в текущей области видимости.

```go
func example() {
    x := 1
    x, y := 2, 3 // y — новая переменная; допустимо
    _, _ = x, y

    // x := 4 // ошибка: нет новых переменных
}
```

### Множественное присваивание

```go
a, b := 1, 2
a, b = b, a
```

```go
value, ok := map[string]int{"a": 1}["a"]
_, _ = value, ok
```

### Операторы

| Категория | Операторы |
|---|---|
| Арифметика | `+`, `-`, `*`, `/`, `%` |
| Сравнение | `==`, `!=`, `<`, `<=`, `>`, `>=` |
| Логика | `&&`, `||`, `!` |
| Побитовые | `&`, `|`, `^`, `&^`, `<<`, `>>` |
| Составное присваивание | `+=`, `-=`, `*=`, `/=`, `%=` и побитовые варианты |

`++` и `--` — statements, а не выражения: `x++` допустимо, а `y := x++` — нет.

### Преобразование типов

```go
var n int = 10
var f float64 = float64(n)

var b byte = 65
var r rune = rune(b)

_, _ = f, r
```

⚠️ Числовых неявных преобразований нет.

### `const`

```go
const Pi = 3.14159
const Limit int = 100
```

```go
const (
    StatusNew = iota
    StatusRunning
    StatusDone
)
```

### `iota` и enum-подобные значения

```go
type Status int

const (
    StatusNew Status = iota
    StatusRunning
    StatusDone
)
```

```go
const (
    _ = iota
    KB = 1 << (10 * iota)
    MB
    GB
)
```

`iota` начинается с `0` в каждом объявлении `const` и увеличивается для каждого следующего `ConstSpec`, а не буквально для каждой строки.

| `var` | `const` |
|---|---|
| Изменяемая переменная | Неизменяемая константа |
| Может иметь runtime-значение | Значение должно быть вычислимо на этапе компиляции |
| Может быть любого типа | Не может быть slice, map, function и т. п. |

---

## 4. Базовые типы, литералы и преобразования

### Основные типы

| Категория | Типы |
|---|---|
| Логический | `bool` |
| Знаковые целые | `int`, `int8`, `int16`, `int32`, `int64` |
| Беззнаковые целые | `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr` |
| Вещественные | `float32`, `float64` |
| Комплексные | `complex64`, `complex128` |
| Текст | `string` |
| Байт | `byte` — alias для `uint8` |
| Unicode code point | `rune` — alias для `int32` |

Размеры `int` и `uint` зависят от архитектуры. Для данных с фиксированным форматом обычно выбирают `int32`, `int64` и т. п.

### Числовые литералы

```go
decimal := 123
binary := 0b1010
octal := 0o755
hex := 0xFF
separated := 1_000_000

float := 3.14
scientific := 1.5e3

_, _, _, _, _, _, _ = decimal, binary, octal, hex, separated, float, scientific
```

### Строки

```go
escaped := "line 1\nline 2"
raw := `line 1
line 2`

_, _ = escaped, raw
```

Raw string в backticks не интерпретирует escape-последовательности.

### Unicode, `rune`, обход строки

```go
text := "Go语言"

for i, r := range text {
    // i — индекс байта, r — rune
    _, _ = i, r
}
```

```go
text := "Go"
firstByte := text[0] // byte
size := len(text)    // число байтов

_, _ = firstByte, size
```

**Нюанс:** `string` — неизменяемая последовательность байтов; `len(s)` возвращает байты, не количество rune.

```go
text := "Go语言"

bytes := []byte(text)
again := string(bytes)

runes := []rune(text)
_ = runes
_ = again
```

### Комплексные числа

```go
z := complex(1.0, 2.0)
re := real(z)
im := imag(z)

_, _ = re, im
```

### Явные преобразования

```go
n := int64(42)
f := float64(n)
s := string(rune(65)) // "A"

_, _, _ = n, f, s
```

⚠️ `string(65)` не является преобразованием числа в `"65"`: это строка с rune U+0041 (`"A"`).

---

## 5. Array, slice, string и map

### Array

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

### Slice

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

### `len`, `cap`, `clear`

```go
s := make([]int, 2, 8)

length := len(s)
capacity := cap(s)

clear(s) // элементы становятся zero values; len не меняется

_, _ = length, capacity
```

Начиная с Go 1.21: `clear(slice)` обнуляет элементы slice.

### `nil slice` и пустой slice

```go
var nilSlice []int
emptySlice := []int{}

fmtLike := len(nilSlice) == 0 && len(emptySlice) == 0
_ = fmtLike
```

Оба имеют длину `0`, по ним можно делать `range`, к ним можно применять `append`. `nilSlice == nil`, `emptySlice != nil`.

### Удаление элемента slice

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

### String

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

### Map

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

### `range`

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

### Адреса элементов при `range`

```go
items := []string{"a", "b"}

for i := range items {
    ptr := &items[i]
    _ = ptr
}
```

⚠️ Переменная `range` не обязана быть адресом элемента коллекции. Если нужен указатель именно на элемент slice, используйте индекс и `&items[i]`.

---

## 6. Управляющие конструкции

### `if`

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

### `for`

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

### `break`, `continue`, labels, `goto`

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

### `switch`

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

### `select`

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

## 7. Функции

### Базовые формы

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

### Именованные результаты

```go
func split(sum int) (left, right int) {
    left = sum / 2
    right = sum - left
    return
}
```

`return` без аргументов возвращает именованные результаты.

⚠️ Naked return допустим только в коротких и очевидных функциях.

### Variadic

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

### Функции как значения, анонимные функции, closure

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

### Рекурсия

```go
func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}
```

### `defer`

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

### `panic` и `recover`

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

## 8. Ошибки: только языковой минимум

### `error`

```go
type error interface {
    Error() string
}
```

`error` — предопределённый интерфейс.

### Возврат ошибки

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

## 9. Указатели

### Основной синтаксис

```go
value := 10
ptr := &value

*ptr = 20

println(value) // 20
```

| Синтаксис | Значение |
|---|---|
| `*T` | указатель на `T` |
| `&value` | адрес переменной |
| `*ptr` | разыменование |

### Указатель на struct

```go
type User struct {
    Name string
}

func rename(user *User, name string) {
    user.Name = name // эквивалентно (*user).Name = name
}

func example() {
    user := User{Name: "Ann"}
    rename(&user, "Ada")
}
```

Go автоматически разыменовывает указатель при доступе к полям и вызове подходящих методов.

### Ограничения

```go
m := map[string]User{
    "u1": {Name: "Ann"},
}

// p := &m["u1"] // ошибка: нельзя взять адрес элемента map
_ = m
```

Pointer arithmetic в безопасном Go отсутствует.

### Практическое правило

| Передавать | Когда обычно подходит |
|---|---|
| `T` | Маленький неизменяемый value; значение не нужно менять |
| `*T` | Нужно изменить объект; структура большая; нужен `nil` как отсутствие значения |

---

## 10. Struct, type, методы и embedding

### Struct

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

### Struct tags

```go
type User struct {
    Name string `json:"name"`
}
```

Struct tag — строка с метаданными поля; язык сам по себе не интерпретирует содержимое тега.

**Нюанс:** struct сравним через `==`, только если сравнимы все его поля.

### Новый именованный тип и alias

```go
type UserID int   // новый именованный тип
type AccountID = int // alias: то же самое, что int
```

`UserID` несовместим с `int` без явного преобразования. `AccountID` — другое имя того же типа `int`.

### Методы

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

### Embedding

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

## 11. Интерфейсы

### Объявление и неявная реализация

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

### `any`

```go
var value any = 42
var oldStyle interface{} = "text"

_, _ = value, oldStyle
```

Начиная с Go 1.18: `any` — alias для `interface{}`.

### Type assertion

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

### Type switch

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

### `nil interface` и typed nil

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

## 12. Generics

Начиная с Go 1.18: type parameters и generics.

### Generic-функция

```go
func First[T any](values []T) T {
    return values[0]
}

func example() {
    value := First([]string{"a", "b"}) // T выводится как string
    _ = value
}
```

### Constraint `comparable`

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

### Generic-тип

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

### Constraints через interface

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

## 13. Конкурентность

### Goroutine

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

### Безопасная передача переменной цикла

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

### Channel

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

### Закрытие канала

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

### `select` для каналов

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

### `sync.WaitGroup`

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

## 14. Дополнительные важные конструкции

### `nil`

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

### `new` и `make`

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

### `delete`, `clear`, `close`

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

### `any` и `comparable`

```go
func Print[T any](value T) {
    _ = value
}

func Equal[T comparable](a, b T) bool {
    return a == b
}
```

### `unsafe`

Пакет `unsafe` предназначен для низкоуровневых операций и намеренно не разбирается в этой шпаргалке.

---

## 15. Финальная сводка

### Таблица 1: Ключевые слова Go

| Ключевое слово | Роль |
|---|---|
| `break` | выход из `for`, `switch`, `select` |
| `default` | ветка по умолчанию |
| `func` | функция или метод |
| `interface` | интерфейс |
| `select` | выбор операции с channel |
| `case` | ветка `switch` / `select` |
| `defer` | отложенный вызов |
| `go` | запуск goroutine |
| `map` | тип map |
| `struct` | структура |
| `chan` | тип channel |
| `else` | альтернативная ветка `if` |
| `goto` | переход к label |
| `package` | пакет файла |
| `switch` | множественное ветвление |
| `const` | константа |
| `fallthrough` | переход к следующему `case` |
| `if` | условие |
| `range` | обход коллекции / channel |
| `type` | объявление типа / alias |
| `continue` | следующая итерация цикла |
| `for` | цикл |
| `import` | импорт пакета |
| `return` | возврат из функции |
| `var` | переменная |

### Таблица 2: Часто забываемый синтаксис

| Синтаксис | Значение |
|---|---|
| `x := value` | короткое объявление; только внутри функции |
| `values...` | раскрыть slice в variadic-вызове |
| `func f(x ...T)` | variadic-параметр |
| `defer f()` | выполнить при выходе из функции |
| `v, ok := m[key]` | значение map и наличие ключа |
| `v, ok := <-ch` | значение channel и факт его незакрытости |
| `switch {}` | switch по условиям |
| `select {}` | выбор готовой channel-операции |
| `type T = U` | alias типа |
| `type T U` | новый именованный тип |
| `any` | alias для `interface{}` |
| `comparable` | constraint для `==` / `!=` |
| `~int` | типы с базовым типом `int` в constraint |
| `chan<- T` | send-only channel |
| `<-chan T` | receive-only channel |
| `_` | игнорируемое значение / blank identifier |
| `nil` | zero value ссылочных типов |
| `make([]T, n)` | создать slice |
| `make(map[K]V)` | создать map |
| `make(chan T)` | создать channel |
| `new(T)` | вернуть `*T` на zero value |
| `append(s, v)` | добавить элементы в slice |
| `clear(s)` | обнулить элементы slice |
| `clear(m)` | удалить все ключи map |
| `delete(m, key)` | удалить ключ map |
| `close(ch)` | закрыть channel |
