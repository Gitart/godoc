# A.41. Строковая функция

Go предоставляет пакет `strings` , который содержит много функций для нужд обработки строковых данных. В этой главе рассматриваются некоторые функции, содержащиеся в пакете.

## A.41.1. функция `strings.Contains()`

Используется для определения того, является ли строка (второй параметр) частью другой строки (первый параметр). Возвращает `bool` .

```
package main

import "fmt"
import "strings"

func main() {
    var isExists = strings.Contains("john wick", "wick")
    fmt.Println(isExists)
}

```

Переменные `isExists` будут оценены `true` , потому что строки `"wick"` являются частью `"john wick"` .

## A.41.2. функция `strings.HasPrefix()`

Используется для определения, начинается ли строка (первый параметр) с определенной строки (второй параметр).

```
var isPrefix1 = strings.HasPrefix("john wick", "jo")
fmt.Println(isPrefix1) // true

var isPrefix2 = strings.HasPrefix("john wick", "wi")
fmt.Println(isPrefix2) // false

```

## A.41.3. функция `strings.HasSuffix()`

Используется для определения того, заканчивается ли строка (первый параметр) конкретной строкой (второй параметр).

```
var isSuffix1 = strings.HasSuffix("john wick", "ic")
fmt.Println(isSuffix1) // false

var isSuffix2 = strings.HasSuffix("john wick", "ck")
fmt.Println(isSuffix2) // true

```

## A.41.4. функция `strings.Count()`

У него есть утилита для подсчета определенного количества символов (второй параметр) строки (первый параметр). Возвращаемое значение этой функции \- количество символов.

```
var howMany = strings.Count("ethan hunt", "t")
fmt.Println(howMany) // 2

```

Возвращаемое значение связано с `2` тем, `"ethan hunt"` что в строке два символа `"t"` .

## A.41.5. функция `strings.Index()`

Используется для поиска позиции индекса строки (второй параметр) в строке (первый параметр).

```
var index1 = strings.Index("ethan hunt", "ha")
fmt.Println(index1) // 2

```

Строка `"ha"` помещается в `2` строку `"ethan hunt"` (индекс начинается с 0). Если найдены две подстроки, берется первая, например:

```
var index2 = strings.Index("ethan hunt", "n")
fmt.Println(index2) // 4

```

Строка `"n"` находится в индексе `4` и `8` . То, что возвращается, является левым (самым маленьким), то есть `4` .

## A.41.6. функция `strings.Replace()`

Эта функция используется для замены или замены частей строки определенными строками. Количество замененных подстрок может быть определено, будь то только 1 строка, 2 строки или все они.

```
var text = "banana"
var find = "a"
var replaceWith = "o"

var newText1 = strings.Replace(text, find, replaceWith, 1)
fmt.Println(newText1) // "bonana"

var newText2 = strings.Replace(text, find, replaceWith, 2)
fmt.Println(newText2) // "bonona"

var newText3 = strings.Replace(text, find, replaceWith, -1)
fmt.Println(newText3) // "bonono"

```

объяснение:

1.  В приведенном выше примере подстроки `"a"` в строке `"banana"` будут заменены на строки `"o"` .
2.  При замене `newText1` только 1 буква, `o` поскольку максимальная подстрока, которую вы хотите заменить, определяется как 1.
3.  Число `-1` сделает процесс замены применимым ко всем подстрокам. Пример можно увидеть в `newText3` .

## A.41.7. функция `strings.Repeat()`

Используется для повторения строки (первый параметр) до заданных данных (второй параметр).

```
var str = strings.Repeat("na", 4)
fmt.Println(str) // "nananana"

```

В приведенном выше примере строка `"na"` повторяется 4 раза. Результат: `"nananana"`

## A.41.8. функция `strings.Split()`

Используемый для разделения строки (первый параметр) с разделителем может быть определен один (второй параметр). Результатом является строковый массив.

```
var string1 = strings.Split("the dark knight", " ")
fmt.Println(string1) // ["the", "dark", "knight"]

var string2 = strings.Split("batman", "")
fmt.Println(string2) // ["b", "a", "t", "m", "a", "n"]

```

Строка `"the dark knight"` отделяется пробелом `" "` , результат затем обрабатывается `string1` .

Чтобы разбить строку на массивы на 1 строку, используйте пустой разделитель строк `""` . Можно увидеть, например, в переменных `string2` .

## A.41.9. функция `strings.Join()`

Имеет противоположное применение `strings.Split()` . Используется для объединения строкового массива (первый параметр) в строку с определенным разделителем (второй параметр).

```
var data = []string{"banana", "papaya", "tomato"}
var str = strings.Join(data, "-")
fmt.Println(str) // "banana-papaya-tomato"

```

Массивы `data` объединяются с помощью разделителей *dash* ( `-` ).

## A.41.10. функция `strings.ToLower()`

Преобразует строчные буквы в строчные.

```
var str = strings.ToLower("aLAy")
fmt.Println(str) // "alay"

```

## A.41.11. функция `strings.ToUpper()`

Преобразует строчные буквы в заглавные.

```
var str = strings.ToUpper("eat!")
fmt.Println(str) // "EAT!"
```
