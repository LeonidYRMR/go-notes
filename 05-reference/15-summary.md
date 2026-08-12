# 15. Финальная сводка

## Таблица 1: Ключевые слова Go

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

## Таблица 2: Часто забываемый синтаксис

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

