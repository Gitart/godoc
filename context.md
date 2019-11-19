# С.31. Контекст: значение, время ожидания и отмена

В этой главе мы рассмотрим использование `context.Context` для вставки и чтения данных об объектах `*http.Request` , а также для обработки запросов на тайм\-аут и отмены.

Будет создано небольшое приложение веб\-службы, задача поиска данных. Также будет сделано промежуточное программное обеспечение `MiddlewareUtility` , задача вставки информации запроса исходного диспетчера в контекст запроса, прежде чем, наконец, будет достигнут доступ к обработчику конечной точки.

## C.31.1. Значение контекста

Хорошо, продолжайте, подготовьте новую папку проекта со структурой, подобной следующей.

```
novalagung:1-simple $ tree .
.
├── main.go
└── middleware.go

0 directories, 2 files

```

В файле `middleware.go` содержимого с `CustomMux` теми, что были в предыдущих главах, мы уже использовали.

```
package main

import "net/http"

type CustomMux struct {
    http.ServeMux
    middlewares []func(next http.Handler) http.Handler
}

func (c *CustomMux) RegisterMiddleware(next func(next http.Handler) http.Handler) {
    c.middlewares = append(c.middlewares, next)
}

func (c *CustomMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    var current http.Handler = &c.ServeMux

    for _, next := range c.middlewares {
        current = next(current)
    }

    current.ServeHTTP(w, r)
}

```

Затем в файле `main.go` создайте конечную точку `/api/search` , в которой содержимое обработчика отображает данные, `from` взятые из контекста запроса. `from` Эти данные устанавливаются в контекст запроса промежуточным ПО `MiddlewareUtility` .

```
package main

import (
    "context"
    "fmt"
    "net/http"
)

type M map[string]interface{}

func main() {
    mux := new(CustomMux)
    mux.RegisterMiddleware(MiddlewareUtility)

    mux.HandleFunc("/api/search", func(w http.ResponseWriter, r *http.Request) {
        from := r.Context().Value("from").(string)
        w.Write([]byte(from))
    })

    server := new(http.Server)
    server.Handler = mux
    server.Addr = ":80"

    fmt.Println("Starting server at", server.Addr)
    server.ListenAndServe()
}

```

Доступ к контекстным запросам осуществляется через метод `.Context()` объекта запроса. Затем объедините метод `Value()` для извлечения данных в соответствии с ключом, вставленным в параметр.

На данный момент назначение конечной точки `/api/search` отображает только данные, не более того.

Далее подготовьте промежуточное программное обеспечение `MiddlewareUtility` . В нем есть проверка заголовка `Referer` , если есть данные значения `from` (которые затем сохраняются в контексте); тогда как, если нет, значение происходит из свойства `.Host` объекта запроса.

```
func MiddlewareUtility(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()
        if ctx == nil {
            ctx = context.Background()
        }

        from := r.Header.Get("Referer")
        if from == "" {
            from = r.Host
        }

        ctx = context.WithValue(ctx, "from", from)

        requestWithContext := r.WithContext(ctx)
        next.ServeHTTP(w, requestWithContext)
    })
}

```

Объект запроса контекста может быть получен через метод доступа к `.Context()` свойству `*http.Request` . Этот объект типа `context.Context` с нулевым типом есть `nil` .

В приведенном выше коде, если контекст есть `nil` , то инициализируется с помощью нового контекста через `context.Background()` .

> В дополнение к передаче `context.Background()` , создание контекста также может быть сделано через `context.TODO()` . Что касается разницы, пожалуйста, проверьте [страницу документации `context`](https://golang.org/pkg/context/#Background) .

Объект, к `ctx` которому `context.Context` мы прикрепляем данные `from` . Как это сделать, вызвав оператор `context.WithValue()` с 3 вставленными параметрами.

1.  Первый параметр, содержимое \- это объекты контекста.
2.  Второй параметр, он содержит ключ данных для хранения.
3.  Третий параметр, это значение данных, которые будут сохранены.

`.WithValue()` Вышеприведенная функция возвращает объект контекста, его содержимое является объектом контекста, который вставляется в первый параметр вызова функции, но данные были вставлены с ключом 2\-го параметра и значением 3\-го параметра. Таким образом, емкость контекста объекта только изменить это заявление на тот же объект, то есть `ctx` .

Хорошо, теперь, когда объект ctx был изменен, этот объект необходимо снова вставить в объект запроса. Вы делаете это, получая доступ к методу, `.WithContext()` принадлежащему объекту запроса, и затем используете возвращаемое значение `next.ServeHTTP()` .

Запустите приложение, результаты более или менее похожи на следующую картину.

![Пример контекста](https://dasarpemrogramangolang.novalagung.com/images/C.31_1_context_example.png)

О да, автор не использует [http: // localhost](http://localhost) для доступа к приложению, а использует . Я направил этот домен на 127.0.0.1. `[http://mysampletestapp.com](http://mysampletestapp.com)`

![Host Etc.](https://dasarpemrogramangolang.novalagung.com/images/C.31_2_etc_hosts.png)

Хорошо, сейчас кажется довольно ясным использование контекста в объекте http\-запроса. Просто развивайте это по мере необходимости. Еще один пример, может использовать контекст для хранения данных сеанса (взятых из базы данных в соответствии с идентификатором сеанса).

## C.31.2. Тайм\-аут и отмена контекста

Мы завершим программу, которая была сделана. Позже конечная точка `/api/search` выполнит поиск в Google по нужным ключевым словам. Поиск осуществляется с помощью [JSON API пользовательского поиска](https://developers.google.com/custom-search/json-api/v1/overview) .

Измените содержимое обработчика конечной точки следующим образом.

```
mux.HandleFunc("/api/search", func(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    keyword := r.URL.Query().Get("keyword")
    chanRes := make(chan []byte)
    chanErr := make(chan error)

    go doSearch(ctx, keyword, chanRes, chanErr)

    select {
    case res := <-chanRes:
        w.Header().Set("Content-type", "application/json")
        w.Write(res)
    case err := <-chanErr:
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
})

```

Процесс поиска выполняется асинхронно с помощью функции, `doSearch()` которую мы создадим позже. При вызове используются ключевые слова `go` и вставляются несколько параметров, два из которых являются типом канала.

*   Канал `chanRes` , используемый в случае успешного поиска. Данные результатов поиска передаются в основную подпрограмму через этот канал, а затем пересылаются как конечная точка ответа
*   Канал `chanErr` , используемый для передачи объекта ошибки, если происходит ошибка.

Хорошо, продолжайте, подготовьте две новые константы.

*   Константы `SEARCH_MAX_DURATION` , используемые для установки максимального времени отклика. Если оно превысит, запрос будет снова отменен.

    ```
    var SEARCH_MAX_DURATION = 4 * time.Second

    ```

*   Константы `GOOGLE_SEARCH_API_KEY` , это ключ API, необходимый для использования API пользовательского поиска. Пожалуйста, обратитесь к странице [Google Cloud Platform,](https://console.cloud.google.com/apis/dashboard) чтобы получить ключ API.

    ```
    var GOOGLE_SEARCH_API_KEY = "ASSVnHfjD_ltXXXXSyB6WWWWWWWWveMFgE"

    ```

Теперь мы начинаем входить в раздел создания функции поиска `doSearch()` . Пожалуйста, напишите следующий код.

```
func doSearch(
    ctx context.Context,
    keyword string,
    chanRes chan []byte,
    chanErr chan error,
) {
    innerChanRes := make(chan []byte)
    innerChanErr := make(chan error)

    url := "https://www.googleapis.com/customsearch/v1"
    url = fmt.Sprintf("%s?key=%s", url, GOOGLE_SEARCH_API_KEY)
    url = fmt.Sprintf("%s&cx=017576662512468239146:omuauf_lfve", url)
    url = fmt.Sprintf("%s&callback=hndlr", url)
    url = fmt.Sprintf("%s&q=%s", url, keyword)

    from := ctx.Value("from").(string)

    // ...
}

```

В рамках этой функции формируется `url` поиск, вставляются данные для ключа API и поискового ключевого слова. Кроме того, он также подготовлен, `innerChanRes` и `innerChanErr` его использование аналогично объекту канала, который вставляется в вызов функции, только эти два новых канала используются только в этой функции.

Продолжайте писать следующий код.

```
ctx, cancel := context.WithTimeout(ctx, SEARCH_MAX_DURATION)
defer cancel()

req, err := http.NewRequest("GET", url, nil)
if err != nil {
    innerChanErr <- err
    return
}

req = req.WithContext(ctx)
req.Header.Set("Referer", from)

transport := new(http.Transport)
client := new(http.Client)
client.Transport = transport

```

Объекты `ctx` , которые передаются извне, оборачиваются с помощью `context.WithTimeout()` . Эта функция возвращает объект контекста, который добавил данные крайнего срока. Сразу после выполнения этого оператора продолжительность `SEARCH_MAX_DURATION` контекста будет отменена.

> Установка контекста крайнего срока может быть выполнена `context.WithTimeout()` или может быть выполнена `context.WithDeadline()` . Разница заключается в том, `.WithDeadline()` что вставленные параметры функции имеют тип `time.Time` .

Также подготовленные объекты `req` являются объектами запроса. Этот объект переносится с использованием `req.WithContext()` содержимого параметров объекта `ctx` , что делает объект контекста, который мы создали, прикрепленным к этому запросу. Его полезность будет обсуждаться позже.

Кроме того, данные, `from` полученные из контекстного запроса, вставляются в качестве заголовка запроса на объект `req` .

Также подготовлены объекты `transport` типа `*http.Transport` и объекты `client` типа `*http.Client` .

После этого напишите код для отправленного запроса и обработайте ответ. Запустите процесс как goroutine.

```
go func() {
    resp, err := client.Do(req)
    if err != nil {
        innerChanErr <- err
        return
    }

    if resp != nil {
        defer resp.Body.Close()
        resData, err := ioutil.ReadAll(resp.Body)
        if err != nil {
            innerChanErr <- err
            return
        }
        innerChanRes <- resData
    } else {
        innerChanErr <- errors.New("No response")
    }
}()

```

Запрос запускается через оператор `client.Do(req)` . Если это приводит к ошибке, отправьте информацию об ошибке на канал `innerChanErr` . Также проверьте объект ответа в результате запроса, если он пуст, тогда выведите ошибку в тот же канал.

Прочитайте тело ответа, если проблем нет, выкиньте результат в канал `innerChanRes` .

Далее мы проверяем, используя технику, `select case` чтобы выяснить, какой канал принимает данные.

```
select {
case res := <-innerChanRes:
    chanRes <- res
    return
case err := <-innerChanErr:
    transport.CancelRequest(req)
    chanErr <- err
    return
case <-ctx.Done():
    transport.CancelRequest(req)
    chanErr <- errors.New("Search proccess exceed timeout")
    return
}

```

Пожалуйста, обратите внимание на код выше, мы обсудим `case` первые 2 .

*   Если канал `innerChanRes` имеет передачу данных, он немедленно передается на канал `chanRes` .
*   Если канал `innerChanErr` имеет передачу данных, он немедленно передается на канал `chanErr` . Не забудьте отменить запрос с помощью метода `transport.CancelRequest(req)` , это необходимо, поскольку запрос не выполнен.

В последнем случае `case <-ctx.Done()` объяснение немного длинное. У объектов контекста есть методы, `ctx.Done()` которые возвращают каналы. Этот канал будет передавать данные, если соблюден крайний срок ожидания контекста. И если это произойдет, приведенный выше код возвращает ошибку непосредственно в канал `chanErr` с содержимым ошибки `Search proccess exceed timeout` ; Не забудьте, что запрос также был отменен.

Запросы должны быть отменены, потому что, если контекст крайнего срока достиг (то есть `SEARCH_MAX_DURATION` ), в то время запрос, возможно, не был выполнен, поэтому запрос должен быть отменен.

Если оно `doSearch` суммировано , то функция выполняется более или менее следующим образом.

*   Если поисковый запрос не более чем `SEARCH_MAX_DURATION` :
    *   Если результат успешен, верните тело ответа через канал `chanRes` .
    *   Если результат не удался, верните ошибку через канал `chanErr` .
*   Если поисковый запрос больше чем `SEARCH_MAX_DURATION` , то он считается *тайм\-аутом* , немедленно выбрасывает тайм\-аут ошибки на канал `chanErr` .

Запустите, посмотрите результаты.

![Запросить ответ ОК](https://dasarpemrogramangolang.novalagung.com/images/C.31_3_result_ok.png)

Дополнительная информация: лучшая практика в отношении контекста отмены \- всегда добавлять `defer cancel()` после создания контекста отмены. Для получения более подробной информации, пожалуйста, прочитайте [https://blog.golang.org/context](https://blog.golang.org/context) .

## C.31.3. Google Search API Ограничения Referer

В начале этой главы мы узнаем о значении контекста. Почему автор решил использовать контекст для вставки данных реферера, а не более общие примеры, такие как сессии и другие? на самом деле есть причина.

Пожалуйста, попробуйте получить доступ к следующим двум URL\-адресам.

*   [http://mysampletestapp.com/api/search?keyword=golang](http://mysampletestapp.com/api/search?keyword=golang)
*   [http: // localhost / api / search? keyword = golang](http://localhost/api/search?keyword=golang)

Должен быть возвращен так же, но на самом деле доступ через url `localhost` выдает ошибку.

![Ошибка ответа на запрос](https://dasarpemrogramangolang.novalagung.com/images/C.31_4_result_fail.png)

Сообщение об ошибке:

Реферрер localhost не соответствует ограничениям реферера, настроенным для вашего ключа API. Пожалуйста, используйте API Console для обновления ваших ключевых ограничений.

Вышеуказанная ошибка появляется потому, что хост `localhost` не был зарегистрирован в API консоли. В отличие от того, что было зарегистрировано, этот хост имеет право доступа с помощью ключа API, который мы используем. `[mysampletestapp.com](http://mysampletestapp.com)`

![Конфигурация API](https://dasarpemrogramangolang.novalagung.com/images/C.31_5_api_hostname.png)

# соответствие результатов ""

# Результаты не соответствуют ""
