# C.26. FTP

В этой главе мы узнаем, как обмениваться данными через FTP (File Transfer Protocol), используя Golang.

> Само определение FTP \- это стандартный сетевой протокол, используемый для обмена или передачи данных между клиентом и сервером.

Перед началом работы необходимо подготовить одну важную вещь, а именно сервер с установленным FTP\-сервером. Если нет, вы можете использовать библиотеку [ftpd](https://github.com/goftp/ftpd) для локальной установки сервера ftp (для обучения).

На сервере подготовьте несколько файлов и папок со следующей структурой.

![Структура папок FTP](https://dasarpemrogramangolang.novalagung.com/images/C.26_1_ftp_folder_structure.png)

*   Файл `test1.txt` , заполните чем\-нибудь.
*   Файл `test2.txt` , заполните чем\-нибудь.
*   Файл `somefolder/test3.txt` , заполните чем\-нибудь.
*   Файл `movie.mp4` , используйте временные файлы.

Мы используем клиентскую библиотеку FTP [github.com/jlaffaye/ftp](https://github.com/jlaffaye/ftp) .

## C.26.1. Подключение к серверу

Создайте новую папку проекта с содержимым `main.go` . Внутри основного файла мы заполним его несколькими операциями FTP, такими как загрузка, загрузка, доступ к каталогу и другие.

ОК, продолжайте, пожалуйста, напишите следующий код.

```
package main

import (
    "fmt"
    "github.com/jlaffaye/ftp"
    "log"
)

func main() {
    const FTP_ADDR = "0.0.0.0:2121"
    const FTP_USERNAME = "admin"
    const FTP_PASSWORD = "123456"

    // ...
}

```

`FTP_` Подготовлены три константы с префиксом , содержимое которых является учетными данными FTP, чтобы иметь возможность подключить FTP к серверу.

Внутри `main()` добавьте код для подключения к серверу.

```
connt, err := ftp.Dial(FTP_ADDR)
if err != nil {
    log.Fatal(err.Error())
}

err = conn.Login(FTP_USERNAME, FTP_PASSWORD)
if err != nil {
    log.Fatal(err.Error())
}

```

Как подключиться к серверу через FTP разбит на два этапа. Сначала необходимо использовать `ftp.Dial()` аргумент адреса сервера (вместе с портом). Оператор возвращает объект типа, `*ftp.ServerConn` который содержится в переменной `conn` .

Второй шаг, передавая объект `conn` , вызывает метод `.Login()` со вставленным аргументом имени пользователя и пароля FTP.

## C.26.2. Отображение всех файлов с помощью `.List()`

По типу `*ftp.ServerConn` доступны все методы для операций FTP. Один из этих методов `.List()` полезен для перечисления всех файлов на сервере. Эта операция такая же как `ls` .

```
fmt.Println("======= PATH ./")

entries, err := conn.List(".")
if err != nil {
    log.Fatal(err.Error())
}
for _, entry := range entries {
    fmt.Println(" ->", entry.Name, getStringEntryType(entry.Type))
}

```

`.List()` Вышеприведенный метод вызывается с аргументом string `"."` , означающим, что все активы в `"."` или **текущий путь** возбуждаются.

Метод возвращает фрагмент, где каждый элемент является представлением каждого файла. В приведенном выше коде все файлы отображаются на консоли, их имена и типы.

Свойство `.Type` принадлежности к `entry` типу есть `ftp.EntryType` , на самом деле `int` . Функция, которую `getStringEntryType()` мы делаем для отображения информации в строках в соответствии с типом.

```
func getStringEntryType(t ftp.EntryType) string {
    switch t {
    case ftp.EntryTypeFile:
        return "(file)"
    case ftp.EntryTypeFolder:
        return "(folder)"
    case ftp.EntryTypeLink:
        return "(link)"
    default:
        return ""
    }
}

```

Запустите приложение, посмотрите результаты.

![Список файлов](https://dasarpemrogramangolang.novalagung.com/images/C.26_2_list_assets.png)

По сравнению с файлами на сервере, есть такой, который не появляется, то есть `somefolder/test3.txt` . Это происходит потому , что файл , который содержится в списке `"."` или **текущий путь** . Файл `test3.txt` находится в подпапке `somefolder` .

## C.26.3. Переместить в указанные папки с помощью `.ChangeDir()`

Далее мы попытаемся войти в папку `somefolder` , затем отобразить содержимое. Используя метод `.ChangeDir()` , введите путь к папке назначения в качестве аргумента.

```
fmt.Println("======= PATH ./somefolder")

err = conn.ChangeDir("./somefolder")
if err != nil {
    log.Fatal(err.Error())
}

```

После этого перечислите все файлы. Продолжайте использовать его `"."` в качестве аргумента вызова метода `.List()` для текущего списка путей. Поскольку мы ввели папку `./somefolder` , путь является текущим путем.

```
entries, err = conn.List(".")
if err != nil {
    log.Fatal(err.Error())
}
for _, entry := range entries {
    fmt.Println(" ->", entry.Name, getStringEntryType(entry.Type))
}

```

Запустите приложение, посмотрите результаты снова.

![Список файлов](https://dasarpemrogramangolang.novalagung.com/images/C.26_3_list_assets_subfolder.png)

## C.26.4. Перейти к использованию родительской папки `.ChangeDirToParent()`

Используйте метод, `.ChangeDirToParent()` чтобы изменить активный путь на родительский путь. Добавьте следующий код, чтобы текущий путь `./somefolder` вернулся к `.` .

```
err = conn.ChangeDirToParent()
if err != nil {
    log.Fatal(err.Error())
}

```

## C.26.5. Извлекайте файлы, используя, `.Retr()` затем читая их содержимое

Способ получения файлов по методу `.Retr()` . Просто напишите путь к файлу, который вы хотите использовать в качестве аргумента. Возвращаемые значения \- это типы объектов `*ftp.Response` и ошибки (если есть).

```
fmt.Println("======= SHOW CONTENT OF FILES")

fileTest1Path := "test1.txt"
fileTest1, err := conn.Retr(fileTest1Path)
if err != nil {
    log.Fatal(err.Error())
}

test1ContentInBytes, err := ioutil.ReadAll(fileTest1)
fileTest1.Close()
if err != nil {
    log.Fatal(err.Error())
}

fmt.Println(" ->", fileTest1Path, "->", string(test1ContentInBytes))

```

Прочитайте содержимое объекта ответа, используя `.Read()` его собственный метод , или также можете использовать `ioutil.ReadAll()` более практичный (возвращаемое значение \- тип, `[]byte` затем приводится к типу `string` сначала, чтобы отобразить содержимое).

> Не забудьте импортировать пакет `io/ioutil` .

В приведенном выше коде файл `test1.txt` читается. Выполните ту же операцию над файлом `somefolder/test3.txt` .

```
fileTest2Path := "somefolder/test3.txt"
fileTest2, err := conn.Retr(fileTest2Path)
if err != nil {
    log.Fatal(err.Error())
}

test2ContentInBytes, err := ioutil.ReadAll(fileTest2)
fileTest2.Close()
if err != nil {
    log.Fatal(err.Error())
}

fmt.Println(" ->", fileTest2Path, "->", string(test2ContentInBytes))

```

> Не забудьте закрыть тип объекта `*ftp.Response` после прочтения содержимого.

Запустите приложение, проверьте результаты.

![Список файлов](https://dasarpemrogramangolang.novalagung.com/images/C.26_4_read_file.png)

Как видно, содержимое прочитанного файла является оригиналом.

## C.26.6. Скачать файл

Технический процесс загрузки заключается в перемещении **содержимого файла** с удаленного сервера на локальный сервер. На локальном компьютере создается файл, который будет содержать содержимое файла с удаленного сервера, который передается. Скорость загрузки зависит от размера файла, который передается (и некоторых других факторов).

В Golang приложение для загрузки через FTP более или менее одинаково. Файлы должны быть созданы сначала локально, чтобы вместить содержимое файлов, взятых с удаленного сервера.

Хорошо, давайте потренируемся. `movie.mp4` Мы будем скачивать файл с сервера на локальный.

```
fmt.Println("======= DOWNLOAD FILE TO LOCAL")

fileMoviePath := "movie.mp4"
fileMovie, err := conn.Retr(fileMoviePath)
if err != nil {
    log.Fatal(err.Error())
}

destinationMoviePath := "/Users/novalagung/Desktop/downloaded-movie.mp4"
f, err := os.Create(destinationMoviePath)
if err != nil {
    log.Fatal(err.Error())
}

_, err = io.Copy(f, fileMovie)
if err != nil {
    log.Fatal(err.Error())
}
fileMovie.Close()
f.Close()

fmt.Println("File downloaded to", destinationMoviePath)

```

Сначала возьмите файл назначения, используя `.Retr()` . Затем создайте файл на локальном компьютере. В приведенном выше примере в локальном имени файла отличается от имени исходного файла `downloaded-movie.mp4` . Создайте файл, используя `os.Create()` .

После этого скопируйте содержимое файла, который был взят с сервера, для `downloaded-movie.mp4` использования `io.Copy()` . После завершения процесса передачи не забудьте закрыть файл с пульта, а также файл локально.

> Операции выше требуют двух других пакетов, а именно, `io` а `os` затем импортировать два пакета.

Запустите приложение, попробуйте проверить сумму md5 из обоих файлов, она должна совпадать.

![Скачать файл](https://dasarpemrogramangolang.novalagung.com/images/C.26_5_download_file.png)

Попробуйте открыть `downloaded-movie.mp4` , если процесс переноса успешен, то его обязательно можно открыть.

![Предварительный просмотр фильма](https://dasarpemrogramangolang.novalagung.com/images/C.26_6_movie_preview.png)

## C.26.7. Загрузка файла

Загрузка файлов противоположна загрузке файлов. Файлы из локального передаются на сервер. Давайте практиковаться прямо сейчас.

Сначала откройте файл назначения, используя `os.Open()` . Затем вызовите метод `.Store()` принадлежит `conn` , вставить файл назначения путь удаленного сервера в качестве первого параметра, и объект в локальный файл в качестве второго параметра.

```
fmt.Println("======= UPLOAD FILE TO FTP SERVER")

sourceFile := "/Users/novalagung/Desktop/Go-Logo_Aqua.png"
f, err = os.Open(sourceFile)
if err != nil {
    log.Fatal(err.Error())
}

destinationFile := "./somefolder/logo.png"
err = conn.Stor(destinationFile, f)
if err != nil {
    log.Fatal(err.Error())
}
f.Close()

fmt.Println("File", sourceFile, "uploaded to", destinationFile)

```

Файл `Go-Logo_Aqua.png` загружается на сервер по пути `./somefolder/logo.png` . Попробуйте список, чтобы проверить, был ли загружен файл.

```
entries, err = conn.List("./somefolder")
if err != nil {
    log.Fatal(err.Error())
}

for _, entry := range entries {
    fmt.Println(" ->", entry.Name, getStringEntryType(entry.Type))
}

```

Запустите приложение, проверьте результаты. Чтобы убедиться, что файлы на клиенте и сервере совпадают, сравните сумму md5 для двух файлов.

![Загрузить файл](https://dasarpemrogramangolang.novalagung.com/images/C.26_7_upload_file.png)

---

*   [ftpd](https://github.com/goftp/ftpd) , Ланни Сяо
*   [ftp](https://github.com/jlaffaye/ftp) , Julien Laffaye, лицензия ISC
