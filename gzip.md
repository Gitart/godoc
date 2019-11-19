# С.15. Сжатие HTTP Gzip (gziphandler)

В этой главе мы рассмотрим применение сжатия HTTP с кодировкой Gzip в веб\-приложении Golang.

## C.15.1. теория

HTTP\-сжатие \- это метод сжатия данных в HTTP\-ответе, поэтому размер / размер вывода уменьшается, а время ответа сокращается.

При обращении к конечной точке в заголовке запроса будет заголовок, `Accept-Encoding` который автоматически вставляется браузером.

```
GET /hello HTTP/1.1
Host: localhost:9000
Accept-Encoding: gzip, deflate

```

Если содержимое `gzip` или `deflate` , браузер готов и поддерживает получение ответов, сжатых из серверной части.

> Deflate \- это алгоритм сжатия данных без потерь. Gzip \- это метод сжатия данных, использующий алгоритм дефляции.

На самом конце, если выходные данные сжимаются, то заголовок ответа `Content-Encoding: gzip` должен быть вставлен.

```
Content-Encoding: gzip

```

Если в запросе нет заголовка `Accept-Encoding: gzip` , но ответная сторона ответа все еще сжата, в браузере появится ошибка `ERR_CONTENT_DECODING_FAILED` .

## C.15.2. практика

Голанг предоставляет пакеты `compress/gzip` . Используя API, доступный в пакете, можно выполнить сжатие данных в ответе HTTP.

Но в этой главе мы не использовали его, а вместо этого использовали одну из известных библиотек промежуточного программного обеспечения для сжатия gzip, [gziphandler](https://github.com/NYTimes/gziphandler) .

Давайте практиковаться. Подготовьте новую папку проекта, подготовьте маршрут `/image` . В обработчике маршрута есть процесс чтения содержимого файла изображения `sample.png` , который затем будет использоваться в качестве ответа на выходные данные. Используйте любой файл изображения для тестирования.

Цель этого приложения \- увидеть, насколько велики размер ответа и продолжительность времени ответа. Позже мы сравним его с результатами тестирования в приложениях, реализующих http gzip comporession.

```
package main

import (
    "io"
    "net/http"
    "os"
)

func main() {
    mux := new(http.ServeMux)

    mux.HandleFunc("/image", func(w http.ResponseWriter, r *http.Request) {
        f, err := os.Open("sample.png")
        if f != nil {
            defer f.Close()
        }
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }

        _, err = io.Copy(w, f)
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
        }
    })

    server := new(http.Server)
    server.Addr = ":9000"
    server.Handler = mux

    server.ListenAndServe()
}

```

Запустите приложение и затем проверьте результаты.

![Без сжатия](https://dasarpemrogramangolang.novalagung.com/images/C.15_1_without_compression.png)

Размер изображения составляет 137 КБ, а время отклика \- 18 мс.

Далее мы попытаемся применить промежуточное программное обеспечение gzip к небольшой программе выше. Сначала загрузите зависимость gziphandler `go get` .

```
$ go get -u github.com/NYTimes/gziphandler

```

Импортируйте библиотеку, которая была загружена в основной файл, затем оберните мультиплексор, `mux` используя `gziphandler.GzipHandler()` мультиплексор, добавленный к свойству `server.Handler` .

```
import (
    // ...
    "github.com/NYTimes/gziphandler"
)

func main() {
    // ...

    server.Handler = gziphandler.GzipHandler(mux)

    // ...
}

```

Перезапустите приложение, посмотрите сравнение.

![Без сжатия](https://dasarpemrogramangolang.novalagung.com/images/C.15_2_with_compression.png)

Разница в размере и времени может не показаться существенной, поскольку изображение маленькое, имеется только один ресурс и доступ к нему на локальном хосте. Для приложений, которые были опубликованы в Интернете и доступны с локального компьютера, это определенно будет выглядеть намного быстрее и легче.

## C.15.3. Gzip сжатие на эхо

Применение сжатия http gzip в инфраструктуре echo может осуществляться с использованием промежуточного программного обеспечения gziphandler, описанного выше. Или вы также можете использовать echo gzip middleware. Вот пример использования промежуточного программного обеспечения gzip echo.

```
e := echo.New()

e.Use(middleware.Gzip())

e.GET("/image", func(c echo.Context) error {
    f, err := os.Open("sample.png")
    if err != nil {
        return err
    }

    _, err = io.Copy(c.Response(), f)
    if err != nil {
        return err
    }

    return nil
})

e.Logger.Fatal(e.Start(":9000"))

```

---

*   [Gzip Handler](https://github.com/NYTimes/gziphandler) от команды New York Times, лицензия Apache\-2.0
*   [Эхо](https://github.com/labstack/echo) , Вишал Рана (лабораторный стек), лицензия MIT
