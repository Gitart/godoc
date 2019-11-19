# C.33. Аутентификация LDAP

В этой главе мы узнаем, как пройти аутентификацию с использованием протокола LDAP для службы каталогов. Урок мы начнем с небольшого обсуждения определения LDAP и некоторых других соответствующих терминов.

## C.33.1. определение

#### LDAP

LDAP (облегченный протокол доступа к каталогам) \- это протокол, используемый для доступа к **службам каталогов** при обмене данными между клиентом и сервером.

#### Службы каталогов

Службы каталогов \- это система, которая хранит, управляет и предоставляет доступ к информации для подключения сетевых ресурсов (или сетевых ресурсов). Сетевые ресурсы, о которых идет речь, например:

*   Электронная почта
*   Периферийные устройства
*   компьютер
*   Папка / Файлы
*   Принтеры
*   Номер телефона
*   ...

Охват сетевых ресурсов, начиная от аппаратного обеспечения, такого как компьютеры (или выше), до относительно небольших аспектов, таких как файлы.

С этими ресурсами, нам будет легко управлять многими вещами, связанными с сетью. Например, создание приложения, которое может войти в систему с использованием служебных учетных данных электронной почты или многих других.

Кроме того, LDAP часто используется в реализации единого входа (SSO).

#### Операция связывания

Операция связывания используется для аутентификации клиента в каталоге сервера, а также для изменения состояния авторизации клиента. Операции привязки выполняются путем отправки информации и паролей bind dn.

#### Сервер каталогов для тестирования

Поскольку связь является клиент\-сервером, нам нужно использовать один из серверов каталогов для тестирования. К счастью, [Forum Systems](https://dasarpemrogramangolang.novalagung.com/www.forumsys.com) любезно предоставила сервер каталогов, доступ к которому можно получить бесплатно, и в этой главе мы будем его использовать.

Ниже приведена информация о сервере каталогов учетных данных:

```
Server: ldap.forumsys.com
Port: 389
Bind DN: cn=read-only-admin,dc=example,dc=com
Bind Password: password

```

Есть некоторые пользовательские данные там, все из которых имеют один и тот же пароль, а именно: `password` .

Для получения более подробной информации, пожалуйста, проверьте на [http://www.forumsys.com/tutorials/integration\-how\-to/ldap/online\-ldap\-test\-server/](http://www.forumsys.com/tutorials/integration-how-to/ldap/online-ldap-test-server/)

## C.33.2. LDAP\-браузер и клиент каталогов

Пожалуйста, используйте ваш предпочитаемый LDAP\-браузер или вы можете использовать [Apache Directory Studio](https://directory.apache.org/studio/) .

Создайте новое соединение, используя вышеуказанные учетные данные. Делайте это до тех пор, пока он не вызовет список пользовательских данных, как показано ниже.

![Просмотр LDAP](https://dasarpemrogramangolang.novalagung.com/images/C.33_1_ldap_browse.png)

Видно, что есть несколько пользовательских данных. Дальнейшее тестирование будет выполнено путем входа в систему с использованием одного из этих пользователей.

Далее войдите в раздел кодирования.

## C.33.3. Вход в веб\-приложение

Сначала мы делаем веб\-приложение в первую очередь. Есть два маршрута, которые должны быть подготовлены:

*   Целевая страница, откройте форму входа
*   Действие входа в конечную точку, для обработки процесса входа

Давайте сразу потренируемся, создайте файл `main.go` , затем подготовьте HTML\-строку для формы входа.

```
package main

import "net/http"
import "fmt"
import "html/template"

// the port where web server will run
const webServerPort = 9000

// the login form html
const view = `<html>
    <head>
        <title>Template</title>
    </head>
    <body>
        <form method="post" action="/login">
            <div>
                <label>username</label>
                <input type="text" name="username" required/>
            </div>
            <div>
                <label>password</label>
                <input type="password" name="password" required/>
            </div>
            <button type="submit">Login</button>
        </form>
    </body>
</html>`

```

Затем для функции `main()` подготовьте целевую страницу обработчика маршрута, проанализируйте приведенную выше строку html.

```
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    var tmpl = template.Must(template.New("main-template").Parse(view))
    if err := tmpl.Execute(w, nil); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
})

```

Затем подготовьте обработчик маршрута для действия входа в систему.

```
http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()

    username := r.PostFormValue("username")
    password := r.PostFormValue("password")

    // authenticate via ldap
    ok, data, err := AuthUsingLDAP(username, password)
    if !ok {
        http.Error(w, "invalid username/password", http.StatusUnauthorized)
        return
    }
    if err != nil {
        http.Error(w, err.Error(), http.StatusUnauthorized)
        return
    }

    // greet user on success
    message := fmt.Sprintf("welcome %s", data.FullName)
    w.Write([]byte(message))
})

```

В приведенном выше коде процесс аутентификации неявно обрабатывается функцией `AuthUsingLDAP()` , что мы, конечно, удостоверимся. При успешном входе пользователя в систему будет отображаться приветственное сообщение, включающее данные о полном имени пользователя (данные об имени позже получаются из данных пользователя в службе каталогов).

Хорошо, два обработчика готовы, теперь запустите веб\-сервер.

```
portString := fmt.Sprintf(":%d", webServerPort)
fmt.Println("server started at", portString)
http.ListenAndServe(portString, nil)

```

## C.33.4. Обработка аутентификации LDAP

Вход в код LDAP, создайте новый файл в той же папке, `ldap.go` . Импортируйте библиотеку ldap и подготовьте несколько констант.

```
package main

import (
    "fmt"
    "github.com/go-ldap/ldap"
)

const (
    ldapServer   = "ldap.forumsys.com"
    ldapPort     = 389
    ldapBindDN   = "cn=read-only-admin,dc=example,dc=com"
    ldapPassword = "password"
    ldapSearchDN = "dc=example,dc=com"
)

```

Сторонняя библиотека [https://github.com/go\-ldap/ldap, которую](https://dasarpemrogramangolang.novalagung.com/github.com/go-ldap/ldap) мы используем для выполнения операций клиент\-сервер с каталогом сервера. Пожалуйста, `go get` сначала, если нет.

Некоторые из приведенных выше констант соответствуют указанным выше учетным данным каталога тестового сервера. Помимо этого есть еще одна постоянная, то есть `ldapSearchDN` позже нам нужно выполнить операцию поиска.

Затем подготовьте структуру для хранения пользовательских данных, возвращаемых каталогом сервера.

```
type UserLDAPData struct {
    ID       string
    Email    string
    Name     string
    FullName string
}

```

Создайте функцию `AuthUsingLDAP()` , затем инициализируйте соединение ldap. Эта функция принимает два параметра, имя пользователя и пароль, оба из которых вводятся пользователем позже через браузер.

```
func AuthUsingLDAP(username, password string) (bool, *UserLDAPData, error) {
    l, err := ldap.Dial("tcp", fmt.Sprintf("%s:%d", ldapServer, ldapPort))
    if err != nil {
        return false, nil, err
    }
    defer l.Close()

    // ...
}

```

Эта функция `ldap.Dial()` используется для установления связи с сервером каталогов.

После этого выполните операцию связывания, используя Bind DN и пароль в соответствии с подготовленными константами. `.Bind()` Для привязки используйте метод, принадлежащий объекту набора ldap.

```
err = l.Bind(ldapBindDN, ldapPassword)
if err != nil {
    return false, nil, err
}

```

После успешной привязки выполните операцию поиска. Подготовьте поисковый запрос объекта заранее. Используемое условие поиска is `uid` или username с base и is `ldapSearchDN` .

```
searchRequest := ldap.NewSearchRequest(
    ldapSearchDN,
    ldap.ScopeWholeSubtree,
    ldap.NeverDerefAliases,
    0,
    0,
    false,
    fmt.Sprintf("(&(objectClass=organizationalPerson)(uid=%s))", username),
    []string{"dn", "cn", "sn", "mail"},
    nil,
)

```

Предложение написано в форме **синтаксиса фильтра LDT** . Можно увидеть выше пример фильтра uid / username. Более подробную информацию об этом синтаксисе можно найти по [адресу http://www.ldapexplorer.com/en/manual/109010000\-ldap\-filter\-syntax.htm.](http://www.ldapexplorer.com/en/manual/109010000-ldap-filter-syntax.htm)

Параметр после фильтра \- это информация об атрибутах, которую мы хотим получить из результатов поискового запроса.

Затем объект триггера поискового запроса передает метод `.Search()` из объекта ldap. Проверьте возвращенные данные (доступны через `.Entries` свойство), если нет данных, то мы предполагаем, что поиск не удался.

```
sr, err := l.Search(searchRequest)
if err != nil {
    return false, nil, err
}

if len(sr.Entries) == 0 {
    return false, nil, fmt.Errorf("User not found")
}
entry := sr.Entries[0]

```

Если все пойдет хорошо, то мы получим как минимум 1 пользовательские данные. Выполните операцию связывания, используя DN пользователя с паролем, введенным пользователем. Это необходимо для проверки правильности введенного пользователем пароля.

```
err = l.Bind(entry.DN, password)
if err != nil {
    return false, nil, err
}

```

Если все по\-прежнему идет хорошо, это означает, что процесс аутентификации может быть уверен в успехе. Но есть еще одна вещь, которую необходимо сделать, а именно, сериализация атрибутов данных, возвращаемых с сервера, чтобы вписаться в подготовленный объект `UserLDAPData` .

```
data := new(UserLDAPData)
data.ID = username

for _, attr := range entry.Attributes {
    switch attr.Name {
    case "sn":
        data.Name = attr.Values[0]
    case "mail":
        data.Email = attr.Values[0]
    case "cn":
        data.FullName = attr.Values[0]
    }
}

return true, data, nil

```

## C.33.5. тестирование

Хорошо, давайте проверим это. Запустите программу, затем перейдите по [адресу http: // localhost: 9000 /](http://localhost:9000/) ; Войдите, используя одного из ваших существующих пользователей (пожалуйста, проверьте в вашем браузере LDAP). Здесь я выбираю использовать `boyle` пароль `password` пользователя (все пользователи имеют одинаковый пароль).

![Ldap Войти](https://dasarpemrogramangolang.novalagung.com/images/C.33_2_ldap_login.png)

Авторизация прошла успешно, о чем свидетельствует появление полного имени **Роберта Бойла** . Попробуйте также использовать неправильный пароль, чтобы сделать его более убедительным.

![Ldap Login Success](https://dasarpemrogramangolang.novalagung.com/images/C.33_3_ldap_login_success.png)

## C.33.6. LDAP TLS

Для соединений LDS с поддержкой TLS, просто вызовите метод, `.StartTLS()` принадлежащий объекту набора ldap, и заполните параметры конфигурацией tls. Этот вызов функции должен выполняться сразу после процесса набора на сервер каталогов.

```
l, err := ldap.Dial("tcp", fmt.Sprintf("%s:%d", ldapServer, ldapPort))
if err != nil {
    return false, nil, err
}
defer l.Close()

// reconnect with TLS
err = l.StartTLS(tlsConfig)
if err != nil {
    return false, nil, err
}

```

---

*   [go\-ldap / ldap](https://github.com/go-ldap/ldap) , команда go\-ldap, лицензия MIT
