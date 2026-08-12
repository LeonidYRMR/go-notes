# 1. Структура Go-файла и пакеты

## Минимальный файл

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go")
}
```

`package` должен быть первым объявлением файла (после комментариев). Исполняемая программа начинается с `func main()` в пакете `main`. Функция `main` не принимает параметров и ничего не возвращает. Перед ней выполняются инициализация пакетных переменных и функции `init`. После возврата из `main` программа завершается, не ожидая остальные goroutine.

## Импорты

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

## `init`

```go
package example

func init() {
    // Выполняется до main().
}
```

`init()` не принимает параметров и ничего не возвращает. В одном пакете может быть несколько `init()`.

## Экспортируемые имена

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

## Комментарии

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

## Имена файлов

Файлы одного каталога обычно принадлежат одному пакету. Имя файла — `snake_case.go`; специальные суффиксы: `*_test.go`, `*_linux.go`, `*_amd64.go`.

---

