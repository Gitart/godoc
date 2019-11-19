# С.16. Отправить почту ( `net/smtp` , Gomail v2)

В этой главе мы узнаем, как отправлять электронные письма из приложения Golang, используя следующие два метода.

1.  Используя пакет `net/smtp` .
2.  Использование библиотеки [Gomail](https://gopkg.in/gomail.v2) .

## C.16.1. Отправить письмо с помощью `net/smtp`

Golang предоставляет пакеты `net/smtp` , он содержит множество API для связи по протоколу SMTP. С помощью этого пакета мы можем сделать операцию, чтобы отправить электронное письмо.

Учетная запись электронной почты необходима для отправки электронной почты, пожалуйста, используйте любого поставщика электронной почты. В этой главе мы используем Google Mail (gmail), поэтому подготовьте учетную запись gmail для тестирования.

Давайте практиковаться. Создайте новую папку проекта, скопируйте следующий код.

```
package main

import (
    "fmt"
    "log"
    "net/smtp"
    "strings"
)

const CONFIG_SMTP_HOST = "smtp.gmail.com"
const CONFIG_SMTP_PORT = 587
const CONFIG_EMAIL = "emailanda@gmail.com"
const CONFIG_PASSWORD = "passwordemailanda"

func main() {
    to := []string{"recipient1@gmail.com", "emaillain@gmail.com"}
    cc := []string{"tralalala@gmail.com"}
    subject := "Test mail"
    message := "Hello"

    err := sendMail(to, cc, subject, message)
    if err != nil {
        log.Fatal(err.Error())
    }

    log.Println("Mail sent!")
}

```

В его реализации для возможности отправки электронной почты необходим почтовый сервер. Поскольку мы используем электронную почту Google, используется почтовый сервер Google.

В приведенном выше коде константа с префиксом `CONFIG_` \- это конфигурация, необходимая для подключения к почтовому серверу. Настройте `CONFIG_EMAIL` и `CONFIG_PASSWORD` с использованием учетной записи.

В основной функции вы можете видеть функцию, `sendMail()` вызываемую для отправки электронной почты, с четырьмя вставленными параметрами.

*   Параметр `to` , является целью электронной почты.
*   Параметр `cc` , это пункт назначения cc.
*   Параметр `subject` является темой письма.
*   Параметр `message` \- это тело письма.

Письма в константах `CONFIG_EMAIL` становятся отправителями электронной почты.

ОК, затем создайте `sendMail()` следующую функцию .

```
func sendMail(to []string, cc []string, subject, message string) error {
    body := "From: " + CONFIG_EMAIL + "\n" +
        "To: " + strings.Join(to, ",") + "\n" +
        "Cc: " + strings.Join(cc, ",") + "\n" +
        "Subject: " + subject + "\n\n" +
        message

    auth := smtp.PlainAuth("", CONFIG_EMAIL, CONFIG_PASSWORD, CONFIG_SMTP_HOST)
    smtpAddr := fmt.Sprintf("%s:%d", CONFIG_SMTP_HOST, CONFIG_SMTP_PORT)

    err := smtp.SendMail(smtpAddr, auth, CONFIG_EMAIL, append(to, cc...), []byte(body))
    if err != nil {
        return err
    }

    return nil
}

```

Функция `sendMail()` используется для отправки электронной почты. Четыре данные, которые вставляются в функцию, объединяются в определенном формате, а затем сохраняются в переменных `body` .

Оператор, содержащийся в, `body` создаст следующую строку (формат по умолчанию).

```
From: recipient1@gmail.com, emaillain@gmail.com
Cc: tralalala@gmail.com
Subject: Test mail

Hello

```

Отправка электронной почты осуществляется через `smtp.SendMail()` . В вызове вставляются 5 параметров, ниже приведено объяснение каждого параметра.

*   Первый параметр `smtpAddr` \- это комбинация порта хоста и почтового сервера.
*   Второй параметр `auth` ,, содержит учетные данные для проверки подлинности на почтовом сервере. Этот объект печатается через `smtp.PlainAuth()` .
*   Третий параметр ,, `CONFIG_EMAIL` \- это адрес электронной почты, используемый для отправки электронной почты.
*   Четвёртый параметр, содержимое всех адресов электронной почты, включая `Cc` .
*   5\-й параметр, содержимое `body` электронной почты.

Запустите приложение. Посмотрите на консоль, появляется ошибка.

![Ошибка SMTP в Gmail](https://dasarpemrogramangolang.novalagung.com/images/C.16_1_gmail_error.png)

Вышеуказанная ошибка появляется только при отправке электронных писем с использованием почтового аккаунта Google. По соображениям безопасности Google отключает учетную запись Gmail, которая будет использоваться для отправки электронной почты через код программы.

Активируйте **менее безопасную** функцию **приложений,** чтобы включить ее. Войдите в каждое сообщение Gmail, затем откройте ссылку [https://myaccount.google.com/lesssecureapps](https://myaccount.google.com/lesssecureapps) , затем нажмите кнопку\-переключатель, чтобы **отключить его** .

![Google менее безопасные приложения](https://dasarpemrogramangolang.novalagung.com/images/C.16_2_less_secure.png)

Перезапустите приложение, электронное письмо отправлено. Посмотрите в почтовый ящик, на который вы отправляете, чтобы проверить результаты.

![Письмо получено](https://dasarpemrogramangolang.novalagung.com/images/C.16_3_email_received.png)

## C.16.2. Отправка писем с помощью Gomail v2

С библиотекой [Gomail](https://gopkg.in/gomail.v2) отправка писем может быть легко выполнена. Некоторые операции, такие как создание электронной почты в виде html, добавление вложений, добавление скрытой копии, могут быть легко выполнены с помощью gomail.

Давайте практиковаться прямо сейчас. Сначала загрузите библиотеку.

```
$ go get -u gopkg.in/gomail.v2

```

Затем напишите следующий код.

```
package main

import (
    "gopkg.in/gomail.v2"
    "log"
)

const CONFIG_SMTP_HOST = "smtp.gmail.com"
const CONFIG_SMTP_PORT = 587
const CONFIG_EMAIL = "emailanda@gmail.com"
const CONFIG_PASSWORD = "passwordemailanda"

func main() {
    mailer := gomail.NewMessage()
    mailer.SetHeader("From", CONFIG_EMAIL)
    mailer.SetHeader("To", "recipient1@gmail.com", "emaillain@gmail.com")
    mailer.SetAddressHeader("Cc", "tralalala@gmail.com", "Tra Lala La")
    mailer.SetHeader("Subject", "Test mail")
    mailer.SetBody("text/html", "Hello, <b>have a nice day</b>")
    mailer.Attach("./sample.png")

    dialer := gomail.NewDialer(
        CONFIG_SMTP_HOST,
        CONFIG_SMTP_PORT,
        CONFIG_EMAIL,
        CONFIG_PASSWORD,
    )

    err := dialer.DialAndSend(mailer)
    if err != nil {
        log.Fatal(err.Error())
    }

    log.Println("Mail sent!")
}

```

Подготовьте изображение с именем `sample.png` , сохраните его в той же папке, что и основной файл. Чтобы прикрепить файлы к электронному письму, используйте `.Attach()` собственный метод `*gomail.Message` .

В этом примере содержимое электронной почты является HTML. Используйте HTML MIME в первом параметре, `.SetBody()` чтобы активировать режим электронной почты html.

Запустите приложение, а затем проверьте результаты письма, отправленного во входящие.

![Почта с вложениями](https://dasarpemrogramangolang.novalagung.com/images/C.16_4_mail_with_attachment.png)

---

*   [Gomail v2](https://gopkg.in/gomail.v2) , автор Alexandre [Cesaro](https://gopkg.in/gomail.v2) , лицензия MIT
