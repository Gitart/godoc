# С.20. HTML Scraping & Parsing (goquery)

У Golang есть пакет `net/html` , его содержимое используется для анализа HTML.

В этой главе мы научимся [разбирать](https://github.com/PuerkitoBio/goquery) HTML проще, не используя пакеты `net/html` , а используя [goquery](https://github.com/PuerkitoBio/goquery) . Эта библиотека использует аналогичный jQuery.

Перед началом загрузите пакет, сначала используя `go get` .

```
$ go get -u github.com/PuerkitoBio/goquery

```

> Для самого процесса очистки содержимого html просто используйте функции `.Get()` пакета `net/http` .

## C.20.1. Сценарии практики

Мы будем практиковать применение goquery для получения некоторых данных с веб\-сайта [https://novalagung.com](https://novalagung.com) ; на сайте, на целевой странице, есть несколько блоков статей. Информация в каждой статье будет извлечена, сохранена в одном объекте слайса, а затем отображена в виде строки JSON.

![изображений](https://dasarpemrogramangolang.novalagung.com/images/C.20_1_novalagung.png)

## C.20.2. HTML Scraping и Парсинг Практики

Подготовьте новую папку проекта. В играх создана файл структура с именем `Article` , содержанием 3 является представлением метаданных каждой статьи, а именно `Title` , `URL` , и `Category` .

```
package main

import (
    "encoding/json"
    "github.com/PuerkitoBio/goquery"
    "log"
    "net/http"
)

type Article struct {
    Title    string
    URL      string
    Category string
}

func main() {
    // code here ...
}

```

В функции `main()` , отправьте запрос клиента GET на URL [https://novalagung.com,](https://novalagung.com) чтобы удалить HTML.

```
res, err := http.Get("https://novalagung.com")
if err != nil {
    log.Fatal(err)
}
defer res.Body.Close()

if res.StatusCode != 200 {
    log.Fatalf("status code error: %d %s", res.StatusCode, res.Status)
}

```

Получать доступ к свойствам `.Body` объекта ответа (тип которого читатель), вводить в качестве параметров в вызове `goquery.NewDocumentFromReader()` .

```
doc, err := goquery.NewDocumentFromReader(res.Body)
if err != nil {
    log.Fatal(err)
}

```

Вышеуказанное утверждение возвращает один объект `goquery.Document` , который содержится в `doc` . Если аналогичен jQuery, объекты `doc` являются экземплярами результирующих объектов `$(selector)` .

Из объекта `goquery.Document` используйте доступные API для выполнения операций по мере необходимости. Мы возьмем все элементы статьи в соответствии с существующей HTML\-схемой на целевом веб\-сайте.

![Структура сайта](https://dasarpemrogramangolang.novalagung.com/images/C.20_2_structure.png)

Хорошо, давайте потренируемся сразу, возьмем объект `.post-feed` и затем возьмем дочерние элементы.

```
rows := make([]Article, 0)

doc.Find(".post-feed").Children().Each(func(i int, sel *goquery.Selection) {
    row := new(Article)
    row.Title = sel.Find(".post-card-title").Text()
    row.URL, _ = sel.Find(".post-card-content-link").Attr("href")
    row.Category = sel.Find(".post-card-tags").Text()
    rows = append(rows, *row)
})

bts, err := json.MarshalIndent(rows, "", "  ")
if err != nil {
    log.Fatal(err)
}

log.Println(string(bts))

```

Используйте `.Find()` для поиска элементов, заполните параметры с помощью селектора поиска. Используйте `.Each()` для зацикливания всех полученных элементов.

В приведенном выше примере, после получения элемента `.post-feed` , дочерние элементы доступны с помощью `.Children()` .

Метод `.Text()` используется для извлечения текста в элементе. Между тем, чтобы получить значение атрибута элемента, используйте `.Attr()` .

В повторении 3 информация извлекается из каждого значащего элемента, затем размещается в объекте `row` , а затем добавляется к слайсу.

В конце слайса объект преобразуется в строку формы JSON, а затем отображается.

Запустите приложение, посмотрите результаты.

![Выход](https://dasarpemrogramangolang.novalagung.com/images/C.20_3_output.png)

## C.20.3. Модификация HTML

API\-интерфейсы, доступные в goquery, предназначены не только для извлечения информации, но также могут использоваться для модификации / манипулирования HTML и других операций.

Давайте просто попрактикуемся. Создайте новый файл воспроизведения, подготовьте строку html. На этот раз источником данных является не исходный веб\-сайт, а строка html.

```
package main

import (
    "github.com/PuerkitoBio/goquery"
    "github.com/yosssi/gohtml"
    "log"
    "strings"
)

const sampleHTML = `<!DOCTYPE html>
    <html>
        <head>
            <title>Sample HTML</title>
        </head>
        <body>
            <h1>Header</h1>
            <footer class="footer-1">Footer One</footer>
            <footer class="footer-2">Footer Two</footer>
            <footer class="footer-3">Footer Three</footer>
        </body>
    </html>
`

```

HTML\-строка выше превращается в форму с `goquery.Document` помощью функции `goquery.NewDocumentFromReader()` . Поскольку обязательные параметры относятся к типу читателя, выполняется преобразование строки html в тип читателя `strings.NewReader()` .

```
doc, err := goquery.NewDocumentFromReader(strings.NewReader(sampleHTML))
if err != nil {
    log.Fatal(err)
}

```

Далее внесите несколько изменений.

```
doc.Find("h1").AfterHtml("<p>Lorem Ipsum Dolor Sit Amet Gedhang Goreng</p>")
doc.Find("p").AppendHtml(" <b>Tournesol</b>")
doc.Find("h1").SetAttr("class", "header")
doc.Find("footer").First().Remove()
doc.Find("body > *:nth-child(4)").Remove()

```

Ниже приведено объяснение некоторых API, используемых в приведенном выше коде.

*   Метод `.AfterHtml()` , используемый для добавления нового элемента, его позиция размещается после вызывающего объекта объекта метода.
*   Метод `.AppendHtml()` , используемый для добавления новых дочерних элементов.
*   Метод `.SetAttr()` , используемый для добавления / изменения атрибутов элемента.
*   Метод `.First()` , используемый для доступа к первому элементу. В приведенном выше коде `doc.Find("footer")` возвращает все существующие нижние колонтитулы, в соответствии с селектором. При доступе к методу `.First()` извлекается только первый элемент нижнего колонтитула.
*   Метод `.Remove()` , используемый для удаления текущего элемента.

Примите форму строки html от `doc` измененного объекта . Не используйте, `doc.Html()` потому что возвращается [внутренний HTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) . Используйте, `goquery.OuterHtml(doc.Selection)` чтобы [внешний HTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/outerHTML) вернулся .

```
modifiedHTML, err := goquery.OuterHtml(doc.Selection)
if err != nil {
    log.Fatal(err)
}

log.Println(gohtml.Format(modifiedHTML))

```

В приведенном выше коде мы используем еще одну библиотеку, [gohtml](https://github.com/yosssi/gohtml) . Функция `.Format()` в библиотеке используется для украшения кода.

Запустите приложение, посмотрите результаты.

![Украшенный HTML](https://dasarpemrogramangolang.novalagung.com/images/C.20_4_beautified_html.png)

---

*   [goquery](https://github.com/PuerkitoBio/goquery) , автор Martin Angers, лицензия BSD\-3\-Clause
*   [gohtml](https://github.com/yosssi/gohtml) , Keiji Yoshida, лицензия MIT

# соответствие результатов ""

# Результаты не соответствуют ""
