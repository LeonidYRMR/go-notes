# 4. Базовые типы, литералы и преобразования

## Основные типы

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

## Числовые литералы

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

## Строки

```go
escaped := "line 1\nline 2"
raw := `line 1
line 2`

_, _ = escaped, raw
```

Raw string в backticks не интерпретирует escape-последовательности.

## Unicode, `rune`, обход строки

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

## Комплексные числа

```go
z := complex(1.0, 2.0)
re := real(z)
im := imag(z)

_, _ = re, im
```

## Явные преобразования

```go
n := int64(42)
f := float64(n)
s := string(rune(65)) // "A"

_, _, _ = n, f, s
```

⚠️ `string(65)` не является преобразованием числа в `"65"`: это строка с rune U+0041 (`"A"`).

---

