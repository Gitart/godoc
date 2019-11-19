# С.19. Конвертировать HTML в PDF (go\-wkhtmltopdf)

Библиотека gofpdf может использоваться только для создания PDF. Обычно в приложении PDF\-отчет, загруженный с источником данных, является самой веб\-страницей отчета. Итак, в этой главе мы научимся конвертировать HTML\-файлы в формат PDF с помощью библиотеки [golang wkhtmltopdf](https://github.com/wkhtmltopdf/wkhtmltopdf) .

> На самом деле, в наши дни экспорт HTML в PDF уже может быть выполнен на внешнем уровне, используя только javascript. Особенно, если вы используете хорошо известную среду, такую ​​как Kendo UI, отчеты, созданные с помощью KendoGrid, можно легко экспортировать в Excel или PDF.
>
> Но в этой главе мы по\-прежнему будем обсуждать этот традиционный способ \- преобразование HTML в PDF на серверной части (golang). Надеюсь, это полезно.

## C.19.1. Конвертировать HTML в PDF файл

Создайте html\-файл с именем `input.html` , заполните его чем\-нибудь, если это возможно, там тоже есть картинка. Например, следующим образом.

```
<!DOCTYPE html>
<html>
    <head>
        <title>Testing</title>
    </head>
    <body>
        <p>
            Lorem ipsum dolor sit amet, consectetur adipiscing elit.
            Proin tempor ut ipsum et feugiat. Phasellus porttitor,
            felis et gravida aliquam,
            eros orci dignissim magna, at tristique elit massa at magna.
            Etiam et dignissim mi. Phasellus laoreet nulla non aliquam imperdiet.
            Aenean varius turpis at orci posuere, ut bibendum lorem porta.
            Maecenas ullamcorper posuere ante quis ultricies. Aliquam erat volutpat.
            Vestibulum ante ipsum primis in faucibus orci luctus et
            ultrices posuere cubilia Curae;
            Pellentesque eu tellus ante. Vivamus varius nisi non nulla imperdiet,
            vitae pellentesque nibh varius. Fusce diam magna, iaculis eget felis id,
            accumsan convallis elit.
            Phasellus in magna placerat, aliquet ante sed, luctus massa.
            Sed fringilla bibendum feugiat. Suspendisse tempus, purus sit amet
            accumsan consectetur, ipsum odio commodo nisi,
            vel dignissim massa mi ac turpis. Ut fringilla leo ut risus facilisis,
            nec malesuada nunc ornare. Nulla a dictum augue.
        </p>

        <img src="https://i.stack.imgur.com/440u9.png">

        <!-- other code here -->
    </body>
</html>

```

HTML\-файл выше будет преобразован в новый тип PDF\-файла. Преобразование выполняется с использованием библиотеки wkhtmltopdf. Эта библиотека на самом деле является приложением CLI, созданным с использованием **C ++** . Чтобы использовать его, мы должны сначала загрузить и установить его.

Пожалуйста, скачайте установщик wkhtmltopdf по адресу [https://wkhtmltopdf.org/downloads.html](https://wkhtmltopdf.org/downloads.html) , выберите его в соответствии с используемой операционной системой.

Поскольку wkhtmltopdf является приложением CLI, его можно использовать двумя способами.

*   Метод 1: Использование `exec.Command()` для выполнения двоичных файлов. Путь к целевому html\-файлу вставляется в качестве аргумента команды. Пожалуйста, обратитесь к [Основному справочнику](https://dasarpemrogramangolang.novalagung.com/46-exec.html) по [программированию Голанга Глава 46 \- Exec,](https://dasarpemrogramangolang.novalagung.com/46-exec.html) чтобы узнать, как использовать exec.
*   Способ 2: Использование [оболочки](https://dasarpemrogramangolang.novalagung.com/github.com/SebastiaanKlippert/go-wkhtmltopdf) goang [go\-wkhtmltopdf](https://dasarpemrogramangolang.novalagung.com/github.com/SebastiaanKlippert/go-wkhtmltopdf) . Этот метод \- то, что мы выбираем.

Технически go\-wkhtmltopdf делает то же самое в первом случае, а именно для выполнения двоичного файла wkhtmltopdf `exec.Command()` .

Давайте сразу потренируемся, создадим новую папку проекта. Подготовьте основной файл. Введите следующий код.

```
package main

import (
    "github.com/SebastiaanKlippert/go-wkhtmltopdf"
    "log"
    "os"
)

func main() {
    pdfg, err := wkhtmltopdf.NewPDFGenerator()
    if err != nil {
        log.Fatal(err)
    }

    // ...
}

```

Создание PDF объекта осуществляется через `wkhtmltopdf.NewPDFGenerator()` . Функция возвращает два объекта, объект документа и ошибку (если есть).

Один объект документа представляет 1 файл PDF.

Затем добавьте код для чтения файла `input.html` . Используйте `os.Open` , чтобы файл не читался напрямую, а вместо этого создавался как `io.Reader` . Вставьте объект чтения в функцию `wkhtmltopdf.NewPageReader()` , затем вставьте изменение как новую страницу в объект PDF.

Более или менее код выглядит следующим образом.

```
f, err := os.Open("./input.html")
if f != nil {
    defer f.Close()
}
if err != nil {
    log.Fatal(err)
}

pdfg.AddPage(wkhtmltopdf.NewPageReader(f))

pdfg.Orientation.Set(wkhtmltopdf.OrientationPortrait)
pdfg.Dpi.Set(300)

```

Метод `.AddPage()` принадлежит объекту PDF, используется для добавления новых страниц. Сами новые страницы создаются через `wkhtmltopdf.NewPageReader()` . Если окажется, что содержимое на странице слишком длинное, чтобы быть страницей, оно автоматически создаст страницу и затем настроит содержимое.

Выписка `pdfg.Orientation.Set()` используется для определения ориентации документа, будь то портрет или пейзаж. Выписка `pdfg.Dpi.Set()` используется для установки документов DPI.

При необходимости используйте доступные API go\-wkhtmltopdf. Пожалуйста, обратитесь к [https://godoc.org/github.com/SebastiaanKlippert/go\-wkhtmltopdf,](https://godoc.org/github.com/SebastiaanKlippert/go-wkhtmltopdf) чтобы увидеть, какие API доступны.

Чтобы сохранить объекты документа в физических файлах PDF, необходимо выполнить несколько шагов. Сначала создайте документ в виде буфера, используя метод `.Create()` .

```
err = pdfg.Create()
if err != nil {
    log.Fatal(err)
}

```

После этого выведите буфер как физический файл, используя `.WriteFile()` . Вставьте путь к файлу назначения в качестве параметра метода.

```
err = pdfg.WriteFile("./output.pdf")
if err != nil {
    log.Fatal(err)
}

log.Println("Done")

```

Протестируйте приложение, которое мы сделали, посмотрите полученный PDF.

![HTML в PDF](https://dasarpemrogramangolang.novalagung.com/images/C.19_1_convert_html_to_pdf.png)

Как видно, в одном PDF\-файле появляются две страницы, потому что содержимое `input.html` слишком длинное, чтобы использовать его как одну страницу.

Способ, который мы узнали, подходит для использования в HTML\-файлах, содержание которых определено в момент загрузки файла.

## C.19.2. Конвертировать HTML из URL в PDF

Как насчет HTML, источник которого не из физического файла, а из URL? все еще можно сделать. Вы делаете это путем регистрации URL\-адреса как объекта страницы `wkhtmltopdf.NewPage()` , а затем помещаете его в документ как страницу. Примеры его применения можно увидеть в коде ниже.

```
pdfg, err := wkhtmltopdf.NewPDFGenerator()
if err != nil {
    log.Fatal(err)
}

pdfg.Orientation.Set(wkhtmltopdf.OrientationLandscape)

page := wkhtmltopdf.NewPage("http://localhost:9000")
page.FooterRight.Set("[page]")
page.FooterFontSize.Set(10)
pdfg.AddPage(page)

err = pdfg.Create()
if err != nil {
    log.Fatal(err)
}

err = pdfg.WriteFile("./output.pdf")
if err != nil {
    log.Fatal(err)
}

log.Println("Done")

```

Этот метод подходит для преобразования данных HTML, содержимое которых отображается при загрузке страницы. Для контента, который `documen.onload` выглядит асинхронным, например, в случае, когда существует AJAX, и после того, как этот новый контент записан, он не может использовать этот метод. Решение может использовать технику экспорта в PDF из внешнего интерфейса.

---

*   [gofpdf](https://github.com/jung-kurt/gofpdf) , Курт Юнг, лицензия MIT
*   [wkhtmltopdf](https://github.com/wkhtmltopdf/wkhtmltopdf) , Ашиш [Кулькарни](https://github.com/wkhtmltopdf/wkhtmltopdf) , лицензия LGPL\-3.0
*   [go\-wkhtmltopdf](https://github.com/SebastiaanKlippert/go-wkhtmltopdf) , Себастьян Клипперт, лицензия MIT
