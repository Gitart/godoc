# С.29. Protobuf

В этой главе мы узнаем об использовании Protobufs (Protocol Buffers) в Golang.

Имейте в виду, мы не изучаем тему gRPC в этой главе, но в следующей главе.

## C.29.1. определение

Протокол Buffers \- это метод сериализации структурированных данных, созданный Google. Protobuf подходит для использования в приложениях, которые взаимодействуют с другими приложениями. Protobuf можно использовать на многих платформах, например: для связи между мобильными приложениями iOS и веб\-службой Golang можно использовать Protobuf.

Protobuf работает только в части сериализации данных, для связи между службами или между самими приложениями, использующими gRPC.

> gRPC \- это удаленный вызов процедуры или RPC от Google. gRPC использует HTTP / 2 для связи, а также буферы протокола в разделе интерфейса.

Возможно, до этого момента он все еще кажется абстрактным, запутанным, и возникает много вопросов о том, что и для чего этот Protobuf. Чтобы упростить понимание, представьте себе клиентское приложение, которое использует данные из (RESTful) веб\-сервисов с данными, отправляемыми в JSON.

![HTTP JSON Аналогия](https://dasarpemrogramangolang.novalagung.com/images/C.29_1_http_json_analogy.png)

В простой аналогии на рисунке показано, что HTTP используется в качестве транспорта между клиентом и сервером, а JSON используется в качестве типа данных запроса полезной нагрузки и типа тела ответа.

Приведенная выше архитектура (которая использует http и json), если она преобразована в формы gRPC и protobuf, будет более или менее похожа на изображение ниже.

![GRPC PROTOBUF Аналогия](https://dasarpemrogramangolang.novalagung.com/images/C.29_2_grpc_protobuf_analogy.png)

Вы можете это понять?

Проще говоря, прототип похож на JSON или XML, но более структурирован. Разница в том, что схема protobuf модели должна быть определена в начале ( *схема на записи* ).

Схема определяется расширением файла `.proto` . Из файла файл модели генерируется в соответствии с выбранным языком. Например, используемый язык \- java, затем сформировался pojo; например, язык php, класс будет сгенерирован; если язык Голанга структурирован, и так далее.

> gRPC и protobuf \- один из лучших вариантов для приложений, использующих концепцию микросервисов.

## C.29.2. установка

Схема написана в `.proto` скомпилированном формате в соответствии с выбранным языком. Отсюда ясно, что нужен компилятор, поэтому protobuf должен быть установлен первым в соответствующих локалях. Компилятор назван `protoc` или протокомпилятор.

Пожалуйста, обратитесь к [http://google.github.io/proto\-lens/install\-protoc.html,](http://google.github.io/proto-lens/installing-protoc.html) чтобы узнать, как установить в `protoc` соответствии с необходимой операционной системой.

Кроме того `protoc` , есть еще одна, которую нужно установить, а именно: исполняемый протобуф для golang (потому что здесь мы используем язык golang). Как установить это довольно просто:

```
$ go get -u github.com/golang/protobuf/proto
$ go get -u github.com/golang/protobuf/protoc-gen-go

```

> Среда выполнения Protobuf доступна для многих языков, кроме Golang, для полной информации, пожалуйста, проверьте [https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf) .

## C.29.3. Создание файла `.Proto`

Подготовьте папку со схемой, подобной следующей.

```
novalagung:src $ tree .
.
└── yourproject
    ├── main.go
    └── model
        ├── garage.proto
        └── user.proto

```

Папка `yourproject/model` содержит файлы `.proto` (определены два прото\-файла). Из двух файлов, указанных выше, файл модели будет создан `.go` с помощью команды `protoc` . Позже сгенерированный файл используется в `main.go` .

#### • файл `user.proto`

Хорошо, давайте перейдем к разделу написания кода. Откройте файл `user.proto` , напишите следующий код.

```
syntax = "proto3";

package model;

enum UserGender {
    UNDEFINED = 0;
    MALE = 1;
    FEMALE = 2;
}

```

Язык, используемый в файлах прототипов, очень минималистичен и довольно прост для понимания.

Заявление `syntax = "proto3";` , означающее, что использовалась прототипная версия `proto3` . Существует также версия `proto2` , но мы ее не используем.

Оператор `package model;` , используемый для установки имени пакета файла, который будет создан позже. Файл `user.proto` будет скомпилирован, в результате чего файл `user.pb.go` . Файл golang \- это пакет в соответствии с тем, что было определено в операторе в этом примере `model` .

Оператор `enum UserGender` используется для определения перечислений. Напишите имена перечислений и значения в фигурных скобках. Само ключевое слово `UserGender` станет типом перечисления. Значение перечисления в protobuf может использовать только числовые типы данных int.

После процесса компиляции в форму golang приведенный выше код будет более или менее выглядеть следующим образом.

```
package model

type UserGender int32

const (
    UserGender_UNDEFINED UserGender = 0
    UserGender_MALE      UserGender = 1
    UserGender_FEMALE    UserGender = 2
)

```

Затем добавьте оператор, определяющий следующее сообщение в файле `user.proto` .

```
message User {
    string id = 1;
    string name = 2;
    string password = 3;
    UserGender gender = 4;
}

message UserList {
    repeated User list = 1;
}

```

> Чтобы уменьшить общение, авторы используют термин **модель** для `message` кода выше.

Модели определяются с использованием ключевых слов, `message` за которыми следует название модели. В фигурных скобках свойства модели записываются с помощью схемы записи `<tipe data> <nama property> = <numbered field>` .

> Видно, что в каждом поле есть *уникальный номер* . Эта информация полезна для управления версиями модели Protobuf. Каждое поле должно иметь уникальные номера в одном `message` .

Внутри `User` , свойство объявляется `id` , `name` и `password` тип `string` ; а также свойства `gender` типа enum `UserGender` .

Кроме того `User` , модель `UserList` также определена. Он содержит только одно свойство, `list` тип которого является `User` . Повторное ключевое слово в свойстве используется для определения типа массива или среза. Оператор `repeated User` эквивалентен `[]*User` (тип элемента slice должен быть указателем).

Приведенный выше код protobuf создает следующий код golang.

```
type User struct {
    Id       string
    Name     string
    Password string
    Gender   UserGender
}

type UserList struct {
    List []*User
}

```

#### • файл `garage.proto`

Теперь перейдите ко второму файлу `garage.proto` . Пожалуйста, напишите следующий код.

```
syntax = "proto3";

package model;

message GarageCoordinate {
    float latitude = 1;
    float longitude = 2;
}

message Garage {
    string id = 1;
    string name = 2;
    GarageCoordinate coordinate = 3;
}

```

Можно увидеть, свойство `coordinate` на модели `Garage` ее типа является модель , а также, то есть `GarageCoordinate` .

Выше тип данных `float` используется при определении свойств `latitude` и `longitude` . Перейдите по следующей ссылке, чтобы узнать, какие типы примитивов можно использовать в качестве типов данных свойств модели [https://developers.google.com/protocol\-buffers/docs/proto3#scalar](https://developers.google.com/protocol-buffers/docs/proto3#scalar) .

Сделайте следующие две другие модели.

```
message GarageList {
    repeated Garage list = 1;
}

message GarageListByUser {
    map<string, GarageList> list = 1;
}

```

Помимо массива / фрагмента, тип `map` также может быть использован в protobuf. Используйте ключевые слова, `map` чтобы определить тип карты. Запись сопровождается записью ключевых типов данных и типов данных карты значений.

> Написание типа данных карты похоже на написание HashMap на Java, и универсальный тип вставляется.

Возвращаясь к теме, два сообщения выше приведут к следующему коду golang.

```
type GarageList struct {
    List []*Garage
}

type GarageListByUser struct {
    List map[string]*GarageList
}

```

## C.29.4. Компиляция файлов `.Proto`

Файл `.proto` готов, теперь пришло время для компиляции файлов `.go` . Компиляция выполняется с помощью команды `protoc` . Для вывода в виде файла golang используйте флаг `--go_out` . Для более подробной информации, пожалуйста, следуйте следующей команде.

```
novalagung:model $ protoc --go_out=. *.proto
novalagung:model $ tree .
.
├── garage.pb.go
├── garage.proto
├── user.pb.go
└── user.proto

0 directories, 4 files

```

Появятся два новых файла с расширением `.pb.go` .

## C.29.5. практика

Теперь мы узнаем о работе созданных прототипов. Откройте файл `main.go` , напишите следующий код.

```
package main

import (
    "bytes"
    "fmt"
    "os"

    // sesuaikan dengan struktuk folder projek masing2
    "dasarpemrogramangolang/chapter-B.29/model"
)

func main() {
    // more code here ...
}

```

Модели пакетов, содержимое которых генерируется прото\-файлами, импортируются. Из пакета, мы можем получить доступ к сгенерированной \- структуру , как `model.User` , `model.GarageList` и многое другое. Затем попробуйте создать несколько объектов для всех сгенерированных структур.

*   Структурный объект `model.User` :

    ```
     var user1 = &model.User{
         Id:       "u001",
         Name:     "Sylvana Windrunner",
         Password: "f0r Th3 H0rD3",
         Gender:   model.UserGender_FEMALE,
     }

    ```

    Для типа перечисления доступа, как указано выше, написано `model.UserGender_*` . Просто измените `*` желаемое значение, `UNDEFINED` , `MALE` или `FEMALE` .

*   Структурный объект `model.UserList` :

    ```
     var userList = &model.UserList{
         List: []*model.User{
             user1,
         },
     }

    ```

    Рекомендуется всегда хранить объекты protobuf struct printout в виде указателей, поскольку при этом объект будет соответствовать критериям интерфейса `proto.Message` , что значительно облегчит процесс кодирования.

*   Структурный объект `model.Garage` :

    ```
     var garage1 = &model.Garage{
         Id:   "g001",
         Name: "Kalimdor",
         Coordinate: &model.GarageCoordinate{
             Latitude:  23.2212847,
             Longitude: 53.22033123,
         },
     }

    ```

*   Структурный объект `model.GarageList` :

    ```
     var garageList = &model.GarageList{
         List: []*model.Garage{
             garage1,
         },
     }

    ```

*   Структурный объект `model.GarageListByUser` :

    ```
     var garageListByUser = &model.GarageListByUser{
         List: map[string]*model.GarageList{
             user1.Id: garageList,
         },
     }

    ```

#### • Печать прототипов

Напечатайте один из объектов, созданных выше.

```
// =========== original
fmt.Printf("# ==== Original\n       %#v \n", user1)

// =========== as string
fmt.Printf("# ==== As String\n       %v \n", user1.String())

```

Запустите приложение, чтобы увидеть результаты.

![Печать объекта protobuf](https://dasarpemrogramangolang.novalagung.com/images/C.29_3_print.png)

В первом операторе печати объект отображается как есть. Сгенерированная структура имеет несколько других свойств, помимо тех, которые уже определены в прототипе сообщения, таких как `XXX_unrecognized` и некоторые другие. Это свойство необходимо для Protobuf, но оно нам не нужно, поэтому оставьте его в покое.

Во втором операторе print `.String()` доступ к методу осуществляется с отображением всех свойств, определенных в прототипе сообщения (свойство `XXX_` не отображается).

#### • Преобразование объектов прото в строки JSON

Добавьте следующий код:

```
// =========== as json string
var buf bytes.Buffer
err1 := (&jsonpb.Marshaler{}).Marshal(&buf, garageList)
if err1 != nil {
    fmt.Println(err1.Error())
    os.Exit(0)
}
jsonString := buf.String()
fmt.Printf("# ==== As JSON String\n       %v \n", jsonString)

```

Выше мы создаем много объектов с помощью сгенерированных структур. Эти объекты можно преобразовать в строки JSON с помощью пакета [github.com/golang/protobuf/jsonpb](https://github.com/golang/protobuf/) . Сделайте `go get` пакет, если у вас его нет, и не забудьте импортировать его в создаваемый файл.

Вернемся к обсуждению, создайте новый объект\-указатель из структуры `jsonpb.Marshaler` , затем получите доступ к методу `.Marshal()` . Вставьте `io.Writer` преобразованный объект типа контейнера в качестве первого параметра (в примере выше мы используем `bytes.Buffer` ). Затем также вставьте объект прото, который вы хотите преобразовать в качестве второго параметра.

Запустите приложение, проверьте результаты.

![Печатать как строку JSON](https://dasarpemrogramangolang.novalagung.com/images/C.29_4_print_json.png)

Помимо метода `.Marshal()` , преобразование в строку JSON может быть сделано с помощью метода `.MarshalToString()` .

#### • конвертировать строки json в объекты прото

Процесс демаршинга строк json в объекты прото может быть выполнен двумя способами:

*   Создайте объект указателя из структуры `jsonpb.Unmarshaler` , затем получите доступ к методу `.Unmarshal()` . Первый параметр заполняется объектом `io.Reader` , содержимое которого является строкой json, а второй параметр является получателем.

    ```
     buf2 := strings.NewReader(jsonString)
     protoObject := new(model.GarageList)

     err2 := (&jsonpb.Unmarshaler{}).Unmarshal(buf2, protoObject)
     if err2 != nil {
         fmt.Println(err2.Error())
         os.Exit(0)
     }

     fmt.Printf("# ==== As String\n       %v \n", protoObject.String())

    ```

*   Используя `jsonpb.UnmarshalString` , с первым параметром вставлены данные строки json.

    ```
     protoObject := new(model.GarageList)

     err2 := jsonpb.UnmarshalString(jsonString, protoObject)
     if err2 != nil {
         fmt.Println(err2.Error())
         os.Exit(0)
     }

     fmt.Printf("# ==== As String\n       %v \n", protoObject.String())

    ```

Пожалуйста, выберите подходящий метод по мере необходимости. Затем запустите приложение и посмотрите результаты.

![Unmarshal из строки JSON](https://dasarpemrogramangolang.novalagung.com/images/C.29_5_unmarshal.png)

## C.29.6. gRPC + Protobuf

В следующей главе мы узнаем о применении gRPC и protobuf.

---

*   [Protobuf](https://github.com/golang/protobuf) , от Google, лицензия BSD\-3\-Clause
