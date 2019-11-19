# С.32. Веб\-токен JSON (JWT)

В этой главе мы изучим JSON Web Token (JWT) и как его применять в Golang.

## C.31.1. определение

JWT является одним из стандартов JSON ( [RFC 7519](https://tools.ietf.org/html/rfc7519) ) для доступа к токену. Токен формируется из комбинации некоторой закодированной и зашифрованной информации. Упомянутая информация \- это заголовок, полезная нагрузка и подпись.

Пример JWT:

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV\_adQssw5c

Схема JWT:

![Схема JWT](https://dasarpemrogramangolang.novalagung.com/images/C.32_1_jwt_scheme.png)

*   **Заголовки** , их содержимое \- это тип алгоритма, используемого для генерации подписей.
*   **Полезная нагрузка** , содержимое \- важные данные для целей аутентификации, такие как *разрешения* , *группы* , когда происходит вход в систему и / или другие. Эти данные в контексте JWT обычно называют **ПРЕТЕНЗИЯМИ** .
*   **Подпись** , содержимое является результатом шифрования данных с использованием криптографических алгоритмов. Данные, о которых идет речь, представляют собой комбинацию (закодированного) заголовка, (закодированной) полезной нагрузки и секретного ключа.

Как правило, в приложениях, в которых применяются методы авторизации с использованием токенов, данные токена генерируются в произвольной серверной части (с использованием определенного алгоритма) и затем сохраняются в базе данных вместе с пользовательскими данными. Токен не имеет содержимого, только случайная строка (а может и нет). Позже, когда есть запрос, токен, включенный в запрос, сопоставляется с существующим токеном в базе данных, после чего проверяется предоставление / группа / тому подобное, чтобы определить, является ли запрос подтвержденным запросом, который имеет право на доступ к конечным точкам.

В приложениях, которые реализуют JWT, что происходит по\-другому. Токены являются результатом комбинации процессов, кодирования и шифрования некоторой информации. Позже в запросе проверка токенов не выполняется путем сравнения токенов, которые есть в запросе, с токенами, хранящимися в базе данных, потому что токены в JWT действительно не хранятся в базе данных. Проверка токена выполняется путем проверки результатов декодирования и дешифрования, которые связаны в запросе.

Возможно, на первый взгляд это выглядит ужасно, выглядит очень легко взломанным, доказательством тому является то, что данные авторизации можно легко получить из токенов JWT. И авторы говорят, что это ложное восприятие.

Информация, содержащаяся в токене, помимо того, что она закодирована, также зашифрована. Для шифрования требуется закрытый ключ или секретный ключ, и только разработчик знает. Поэтому не забывайте тщательно хранить ключ.

Пока хакер не знает используемый секретный ключ, результаты декодирования и дешифрования данных **НЕ ДОЛЖНЫ БЫТЬ ДЕЙСТВИТЕЛЬНЫ** .

## C.32.2. Подготовка к практике

Мы создадим простое приложение веб\-службы RESTful с двумя конечными точками `/index` и содержимым `/login` . Ниже приведены технические характеристики приложения:

*   Для доступа `/index` требуется токен JWT.
*   Токен получается из процесса аутентификации для конечной точки `/login` путем вставки имени пользователя и пароля.
*   В созданном нами приложении уже есть данные списка пользователей, хранящиеся в базе данных (фактически не в базе данных, а в файле json).

Хорошо, теперь подготовьте папку проекта со схемой, подобной следующей:

```
$ tree .
.
├── main.go
├── middleware.go
└── users.json

0 directories, 3 files

```

#### файл `middleware.go`

Заполните файл `middleware.go` кодом промежуточного программного обеспечения, к которому мы привыкли.

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

#### файл `users.json`

Также заполните файлы, `users.json` которые являются базами данных приложения. Пожалуйста, добавьте следующие данные JSON.

```
[{
    "username": "noval",
    "password": "kaliparejaya123",
    "email": "terpalmurah@gmail.com",
    "group": "admin"
}, {
    "username": "farhan",
    "password": "masokpakeko",
    "email": "cikrakbaja@gmail.com",
    "group": "publisher"
}]

```

#### файл `main.go`

Теперь мы сосредоточимся на файле `main.go` . Нужны импортные пакеты. Одним из таких пакетов является [jwt\-go](https://github.com/dgrijalva/jwt-go) , который используется для операций JWT.

```
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "os"
    "path/filepath"
    "strings"

    jwt "github.com/dgrijalva/jwt-go"
    "github.com/novalagung/gubrak"
)

```

В том же файле подготовьте 4 константы, а именно: имя приложения, продолжительность входа в систему, метод шифрования токена и секретный ключ.

```
type M map[string]interface{}

var APPLICATION_NAME = "My Simple JWT App"
var LOGIN_EXPIRATION_DURATION = time.Duration(1) * time.Hour
var JWT_SIGNING_METHOD = jwt.SigningMethodHS256
var JWT_SIGNATURE_KEY = []byte("the secret of kalimdor")

```

Затем создайте функцию `main()` , подготовьте `mux` новую версию и зарегистрируйте промежуточное ПО `MiddlewareJWTAuthorization` и два маршрута `/index` и `/login` .

```
func main() {
    mux := new(CustomMux)
    mux.RegisterMiddleware(MiddlewareJWTAuthorization)

    mux.HandleFunc("/login", HandlerLogin)
    mux.HandleFunc("/index", HandlerIndex)

    server := new(http.Server)
    server.Handler = mux
    server.Addr = ":8080"

    fmt.Println("Starting server at", server.Addr)
    server.ListenAndServe()
}

```

Промежуточное программное обеспечение `MiddlewareJWTAuthorization` будет создано позже, задача состоит в том, чтобы проверять каждый входящий запрос, проверяя включенный токен JWT. Это промежуточное программное обеспечение полезно только для запросов, отличных от конечных точек `/login` , потому что в этой конечной точке происходит процесс аутентификации.

## C.32.3. Аутентификация и генерация токенов

Подготовить обработчик `HandlerLogin` . Напишите следующий код.

```
func HandlerLogin(w http.ResponseWriter, r *http.Request) {
    if r.Method != "POST" {
        http.Error(w, "Unsupported http method", http.StatusBadRequest)
        return
    }

    username, password, ok := r.BasicAuth()
    if !ok {
        http.Error(w, "Invalid username or password", http.StatusBadRequest)
        return
    }

    // ...
}

```

Этот обработчик отвечает за аутентификацию клиента / потребителя. Имя пользователя и пароль передаются на конечную точку в форме [Basic Auth](https://dasarpemrogramangolang.novalagung.com/B-18-http-basic-auth.html) . Затем данные вставляются в вызов функции аутентификации `authenticateUser()` , который мы также создадим позже.

```
ok, userInfo := authenticateUser(username, password)
if !ok {
    http.Error(w, "Invalid username or password", http.StatusBadRequest)
    return
}

```

Функция `authenticateUser()` имеет два возвращаемых значения, которые содержатся в следующих переменных:

*   Переменная `ok` \- маркер успешной аутентификации.
*   Переменные `userInfo` , содержимое информации, в которую в данный момент вошел пользователь, данные получены `data.json` (но без пароля).

Далее мы создадим объектные претензии. Этот объект должен соответствовать требованиям интерфейса `jwt.Claims` . Объекты заявок могут быть сделаны из типа `map` , обернув их в функции `jwt.MapClaims()` ; или встраивая интерфейс `jwt.StandardClaims` в новую структуру, и этот метод мы будем использовать.

Как мы уже говорили ранее, утверждает, что содержимое является данными для целей аутентификации. На практике заявки \- это объекты, которые имеют много свойств или полей. Хорошо, объект заявки **должен** иметь поля, включенные в список [стандартных полей JWT](https://en.wikipedia.org/wiki/JSON_Web_Token#Standard_fields) . Интерфейс `jwt.StandardClaims` представляет собой представление предполагаемого поля.

В разрабатываемом приложении заявки, помимо размещения стандартных полей, также содержат некоторую другую информацию, поэтому нам необходимо создать `struct` новую вставку `jwt.StandardClaims` .

```
type MyClaims struct {
    jwt.StandardClaims
    Username string `json:"Username"`
    Email    string `json:"Email"`
    Group    string `json:"Group"`
}

```

Хорошо, структура `MyClaims` готова, теперь создайте новый объект из структуры.

```
claims := MyClaims{
    StandardClaims: jwt.StandardClaims{
        Issuer:    APPLICATION_NAME,
        ExpiresAt: time.Now().Add(LOGIN_EXPIRATION_DURATION).Unix(),
    },
    Username: userInfo["username"].(string),
    Email:    userInfo["email"].(string),
    Group:    userInfo["group"].(string),
}

```

Существует несколько стандартных утверждений, в приведенном выше примере только два заполнены в стоимости, `Issuer` а `ExpiresAt` остальные мы оставляем пустыми. Затем 3 дополнительных поля, которые мы создали (имя пользователя, адрес электронной почты и группа), были заполнены с использованием данных, полученных из `userInfo` .

*   `Issuer` (код `iss` ), является издателем JWT, в этом контексте `APPLICATION_NAME` .
*   `ExpiresAt` (код `exp` ), это когда токен JWT считается устаревшим.

Хорошо, объект заявки готов, теперь создайте новый токен. Сделать это можно с помощью функций, `jwt.NewWithClaims()` которые выдают возвращаемые значения типа `jwt.Token` . Первый параметр \- это используемый метод шифрования `JWT_SIGNING_METHOD` , а второй \- `claims` .

```
token := jwt.NewWithClaims(
    JWT_SIGNING_METHOD,
    claims,
)

```

Затем подпишите токен с помощью секретного ключа, определенного в `JWT_SIGNATURE_KEY` методе, вызвав `SignedString()` метод объекта `jwt.Token` . Этот вызов метода возвращает данные строкового токена, которые затем используются в качестве возвращаемого значения обработчика. Эти строковые токены необходимы клиенту / потребителю для доступа к существующим конечным точкам.

```
signedToken, err := token.SignedString(JWT_SIGNATURE_KEY)
if err != nil {
    http.Error(w, err.Error(), http.StatusBadRequest)
    return
}

tokenString, _ := json.Marshal(M{ "token": signedToken })
w.Write([]byte(tokenString))

```

Раздел аутентификации и генерации токенов фактически заканчивается здесь. Но на самом деле чего\-то не хватает, а именно функции `authenticateUser()` . Пожалуйста, сделайте функцию.

```
func authenticateUser(username, password string) (bool, M) {
    basePath, _ := os.Getwd()
    dbPath := filepath.Join(basePath, "users.json")
    buf, _ := ioutil.ReadFile(dbPath)

    data := make([]M, 0)
    err := json.Unmarshal(buf, &data)
    if err != nil {
        return false, nil
    }

    res, _ := gubrak.Find(data, func(each M) bool {
        return each["username"] == username && each["password"] == password
    })

    if res != nil {
        resM := res.(M)
        delete(resM, "password")
        return true, resM
    }

    return false, nil
}

```

Содержимое функции `authenticateUser()` , как следует из названия, довольно ясно, что соответствует имени пользователя и паролю с данными в файле `users.json` .

## C.32.4. Промежуточное ПО авторизации JWT

Теперь нам нужно подготовиться `MiddlewareJWTAuthorization` , задача состоит в том, чтобы проверять каждый запрос, который приходит к конечной точке `/login` , кроме того , есть ли введенный токен доступа или нет. И если есть, будет проверено, является ли токен действительным.

```
func MiddlewareJWTAuthorization(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {

        if r.URL.Path == "/login" {
            next.ServeHTTP(w, r)
            return
        }

        authorizationHeader := r.Header.Get("Authorization")
        if !strings.Contains(authorizationHeader, "Bearer") {
            http.Error(w, "Invalid token", http.StatusBadRequest)
            return
        }

        tokenString := strings.Replace(authorizationHeader, "Bearer ", "", -1)

        // ...
    })
}

```

Мы используем схему заголовка в `Authorization: Bearer <token>` соответствии со спецификациями [RFC 6750](https://tools.ietf.org/html/rfc6750) для целей размещения и извлечения токенов.

Токен извлекается из заголовка, затем анализируется и проверяется с помощью функции `jwt.Parse()` .

```
token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
    if method, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
        return nil, fmt.Errorf("Signing method invalid")
    } else if method != JWT_SIGNING_METHOD {
        return nil, fmt.Errorf("Signing method invalid")
    }

    return JWT_SIGNATURE_KEY, nil
})
if err != nil {
    http.Error(w, err.Error(), http.StatusBadRequest)
    return
}

```

Второй параметр функции `jwt.Parse()` содержит обратный вызов, чтобы проверить, является ли метод подписи действительным или нет, если он действителен, то возвращается секретный ключ. `jwt.Parse()` Эта функция одна возвращает объект токена.

Из объекта токена берется информация о претензии, затем проверяется действительность претензии.

```
claims, ok := token.Claims.(jwt.MapClaims)
if !ok || !token.Valid {
    http.Error(w, err.Error(), http.StatusBadRequest)
    return
}

```

Ах да, может возникнуть вопрос, почему утверждения объекта, сгенерированные `jwt.Parse()` типом, не являются `MyClaims` . Это связано с тем, что после того, как объект утверждений введен в процессе формирования токенов, через функции `jwt.NewWithClaims()` объект будет закодирован в тип `jwt.MapClaims` .

Полученные заявки на данные вставляются в контекст, так что позже в конечной точке информация `userInfo` может быть легко извлечена из запроса контекста.

```
ctx := context.WithValue(context.Background(), "userInfo", claims)
r = r.WithContext(ctx)

next.ServeHTTP(w, r)

```

#### Индекс обработчик

Наконец, нам нужно подготовить обработчик для маршрута `/index` .

```
func HandlerIndex(w http.ResponseWriter, r *http.Request) {
    userInfo := r.Context().Value("userInfo").(jwt.MapClaims)
    message := fmt.Sprintf("hello %s (%s)", userInfo["Username"], userInfo["Group"])
    w.Write([]byte(message))
}

```

Информация `userInfo` берется из контекста, а затем отображается как конечная точка ответа.

## C.32.5. тестирование

Запустите приложение, затем протестируйте его с помощью curl.

#### идентификация

```
curl -X POST --user noval:kaliparejaya123 http://localhost:8080/login

```

выход:

![Аутентификация JWT](https://dasarpemrogramangolang.novalagung.com/images/C.32_2_jwt_authentication.png)

#### Доступ к конечным точкам

Тест конечной точки `/index` . Вставьте маркер, возвращенный во время аутентификации, в качестве значения заголовка авторизации со схемой `Authorization: Bearer <token>` .

```
curl -X GET \
    --header "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzg2NTI0NzksImlzcyI6Ik15IFNpbXBsZSBKV1QgQXBwIiwiVXNlcm5hbWUiOiJub3ZhbCIsIkVtYWlsIjoidGVycGFsbXVyYWhAZ21haWwuY29tIiwiR3JvdXAiOiJhZG1pbiJ9.JREJgUAYs2R5zuquqyX8tk23QlajVVe19susm6JsZq8" \
    http://localhost:8080/index

```

выход:

![Авторизация JWT](https://dasarpemrogramangolang.novalagung.com/images/C.32_3_jwt_authorization.png)

Все прошло как положено. Чтобы быть более убедительным, попробуйте выполнить несколько тестов с неправильным сценарием, например:

![Ошибка JWT](https://dasarpemrogramangolang.novalagung.com/images/C.32_4_invalid_jwt_token.png)

# соответствие результатов ""

# Результаты не соответствуют ""
