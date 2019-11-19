# C.30. gRPC + Protobuf

В этой главе мы узнаем о применении gRPC и protobuf на языке Golang.

Мы создадим большую папку проекта, внутри которой есть 3 приложения. Два из них являются серверными приложениями, rpc server. И одно из них \- клиентское приложение. Клиентское приложение будет связываться с обоими серверными приложениями.

Можно сказать, что это очень простой (и небрежный) пример реализации [архитектуры микросервисов](https://en.wikipedia.org/wiki/Microservices) .

> ВНИМАНИЕ: этот урок очень длинный! И не ожидайте слишком высоких ожиданий, потому что целевые читатели \- это люди, которые только изучают Golang, gRPC и Protobuf.

## C.30.1. определение

gRPC \- это RPC\-фреймворк, созданный Google. gRPC использует RPC для транспорта и protobuf в своем интерфейсе.

> Удаленный вызов процедур (RPC) \- это метод, который позволяет нам получить доступ к процедуре, находящейся на другом компьютере. Чтобы иметь возможность сделать это, сервер должен предоставлять услуги удаленных процедур.

## C.30.2. Структура приложения

Подготовьте новый проект со следующей структурой.

```
novalagung:chapter-b.30 $ tree .
.
├── common
│   ├── config
│   │   └── config.go
│   └── model
│       ├── garage.proto
│       ├── user.proto
│       └── util.proto
│
├── client
│   └── main.go
│
└── services
    ├── service-garage
    │   └── main.go
    └── service-user
        └── main.go

7 directories, 7 files

```

Проект на этот раз довольно сложный, ниже приведено объяснение для части структуры проекта выше.

#### • папка `common`

Папка `common` , содержит 2 подпапки `config` и `model` .

*   Папка `config` содержит общую или глобальную информацию, которая используется клиентскими и серверными приложениями.
*   Папка `model` содержит файлы `.proto` . Пожалуйста, скопируйте файл `garage.proto` и `user.proto` в предыдущей главе в папку.

#### • папка `client`

Он содержит один основной файл, который позже будет запущен как клиентское приложение. Это клиентское приложение будет взаимодействовать с 2 серверными приложениями.

#### • папка `services`

Один файл прото для одного приложения сервера (службы) rpc. Поскольку есть два прототипа, это означает, что существует два серверных приложения rpc, `service-user` и `service-garage` . `services` Эта папка содержит оба приложения\-службы.

Файлы `garage.proto` и файлы `user.proto` не объединяются в соответствующей папке `service-garage` и `service-user` , поскольку эти два файла также используются в клиентских приложениях, поэтому эти файлы разделяются на папки `common/model` .

## C.30.3. Файл конфигурации в папке `common/config`

Подготовьте файл с именем `config.go` в `common/config` . В этом файле конфигурации определены две константы, а именно порт для пользовательской службы и гараж. Позже серверное приложение начнет использовать этот порт.

```
package config

const (
    SERVICE_GARAGE_PORT = ":7000"
    SERVICE_USER_PORT   = ":9000"
)

```

## C.30.4. Протомодель на папке `common/model`

После того, как два файла `user.proto` и `garage.proto` в предыдущей главе будут скопированы, нам нужно добавить определение сервиса в каждый файл прото.

Ключевые слова `service` используются для создания сервисов. Этот сервис также будет преобразован в golang позже (как интерфейс) с помощью команды `protoc` . В приложении сервера rpc интерфейс должен быть сделан позже.

Хорошо, теперь добавьте следующий код в файл proto.

#### • Сервис `Users`

Откройте файл `user.proto` , добавьте следующий код в конце строки.

```
// ...

import "google/protobuf/Empty.proto";

service Users {
    rpc Register(User) returns (google.protobuf.Empty) {}
    rpc List(google.protobuf.Empty) returns (UserList) {}
}

```

Именованная служба `Users` определена с содержанием двух методов.

*   `Register()` , принимает параметр типа модели `User` и возвращает `google.protobuf.Empty` .
*   `List()` , принимает параметр типа `google.protobuf.Empty` и возвращает тип `UserList` .

Пожалуйста, посмотрите, как сделать сервис в коде выше.

В `Register()` реальном методе нам не нужно возвращаемое значение. Но поскольку требования Protobuf требуют, чтобы все методы RPC имели возвращаемое значение, мы используем модель `Empty` Protobuf от Google.

Как использовать модель `Empty` \- это сначала импортировать файл proto `google/protobuf/Empty.proto` , а затем использовать его `google.protobuf.Empty` в качестве модели.

Кроме того, с методами `List()` аргументы на самом деле не нужны, но поскольку Protobuf требует, чтобы у метода rpc был один аргумент и один тип возвращаемого значения, то неизбежно должен быть аргумент.

После компиляции создаются два интерфейса с именем `<interfacename>Server` и схемой `<interfacename>Client` . Поскольку имя службы `Users` , затем terbuatlah `UsersServer` и `UsersClient` .

*   Интерфейс `UsersServer` .

    ```
    type UsersServer interface {
       Register(context.Context, *User) (*empty.Empty, error)
       List(context.Context, *empty.Empty) (*UserList, error)
    }

    ```

    Этот интерфейс должен быть реализован в приложении сервера rpc.

*   Интерфейс `UsersClient` .

    ```
    type UsersClient interface {
       Register(ctx context.Context, in *User, opts ...grpc.CallOption) (*empty.Empty, error)
       List(ctx context.Context, in *empty.Empty, opts ...grpc.CallOption) (*UserList, error)
    }

    ```

    Этот интерфейс должен быть реализован в клиентском приложении rpc.

#### • Сервис `Garages`

В файле `garage.proto` определите сервис `Garages` с содержимым двух методов, `Add()` и `List()` .

```
import "google/protobuf/Empty.proto";

service Garages {
    rpc List(string) returns (GarageList) {}
    rpc Add(string, Garage) returns (google.protobuf.Empty) {}
}

```

Обратите внимание, что protobuf требует, чтобы метод rpc не использовал примитивные типы в качестве типов аргументов или типов возвращаемых значений. Тип `string` on `rpc List(string)` выдаст ошибку при компиляции. Кроме того, protobuf требует, чтобы метод принимал только один параметр, поэтому `rpc Add(string, Garage)` он, очевидно , недопустим.

Тогда каково решение? Снова создайте новую модель, свойства подстраиваются под параметры, необходимые в каждом методе RPC.

```
message GarageUserId {
    string user_id = 1;
}

message GarageAndUserId {
    string user_id = 1;
    Garage garage = 2;
}

service Garages {
    rpc List(GarageUserId) returns (GarageList) {}
    rpc Add(GarageAndUserId) returns (google.protobuf.Empty) {}
}

```

Как и сервисы `Users` , сервисы `Garages` также будут скомпилированы в интерфейсы.

*   Интерфейс `GaragesServer` .

    ```
    type GaragesServer interface {
       Add(context.Context, *GarageAndUserId) (*empty.Empty, error)
       List(context.Context, *GarageUserId) (*GarageList, error)
    }

    ```

*   Интерфейс `GaragesClient` .

    ```
    type GaragesClient interface {
       Add(ctx context.Context, in *GarageAndUserId, opts ...grpc.CallOption) (*empty.Empty, error)
       List(ctx context.Context, in *GarageUserId, opts ...grpc.CallOption) (*GarageList, error)
    }

    ```

## C.30.5. Компиляция файлов `.proto`

Команда, ранее использованная для компиляции:

```
$ protoc --go_out=. *.proto

```

Команда не создает сервис в качестве интерфейса. Аргумент `--go_out` нужно немного изменить, чтобы генерировались сервис и метод rpc. Используйте следующую команду.

```
$ protoc --go_out=plugins=grpc:. *.proto

```

## C.30.6. Серверное приложение `service-user`

Откройте файл `services/service-user/main.go` , импортируйте необходимый пакет.

```
package main

import (
    "context"
    "log"
    "net"

    "github.com/golang/protobuf/ptypes/empty"
    "google.golang.org/grpc"
    "dasarpemrogramangolang/chapter-B.30/common/config"
    "dasarpemrogramangolang/chapter-B.30/common/model"
)

```

Затем создайте объект с именем `localStorage` type `*model.UserList` . Этот объект будет содержать пользовательские данные, добавленные из клиентского приложения (через rpc) через метод `Register()` . И данные этого объекта также возвращаются клиенту при `List()` доступе.

```
var localStorage *model.UserList

func init() {
    localStorage = new(model.UserList)
    localStorage.List = make([]*model.User, 0)
}

```

Создать новую структуру `UsersServer` . Эта структура будет реализацией сгенерированного интерфейса `model.UsersServer` . Подготовьте методы в соответствии со спецификациями интерфейса.

```
type UsersServer struct{}

func (UsersServer) Register(ctx context.Context, param *model.User) (*empty.Empty, error) {
    localStorage.List = append(localStorage.List, param)

  log.Println("Registering user", param.String())

    return new(empty.Empty), nil
}

func (UsersServer) List(ctx context.Context, void *empty.Empty) (*model.UserList, error) {
    return localStorage, nil
}

```

Создайте функции `main()` , создайте объекты сервера grpc и объекты реализации `UsersServer` , затем зарегистрируйте два объекта в модели с помощью операторов `model.RegisterUsersServer()` .

```
func main() {
    srv := grpc.NewServer()
    var userSrv UsersServer
    model.RegisterUsersServer(srv, userSrv)

    log.Println("Starting RPC server at", config.SERVICE_USER_PORT)

  // more code here ...
}

```

Создание объекта сервера Grpc осуществляется через `grpc.NewServer()` . Пакет [google.golang.org/grpc](https://google.golang.org/grpc) необходимо `go get` импортировать и импортировать.

> Функции `RegisterUsersServer()` автоматически создаются протоколом, потому что сервис `Users` определен. Другой пример \- служба `SomeServiceTest` подготовлена, затем создана функция `RegisterSomeServiceTestServer()` .

Затем подготовьте объект прослушивателя, который прослушивает порт `config.SERVICE_USER_PORT` , затем используйте прослушиватель в качестве метода аргумента `.Serve()` объекта сервера rpc.

```
// ...

l, err := net.Listen("tcp", config.SERVICE_USER_PORT)
if err != nil {
  log.Fatalf("could not listen to %s: %v", config.SERVICE_USER_PORT, err)
}

log.Fatal(srv.Serve(l))

```

## C.30.6. Серверное приложение `service-garage`

Создайте файл `services/service-garage/main.go` , импортируйте тот же пакет, что и в `service-user` . Затем создайте объект `localStorage` из структуры `*model.GarageListByUser` .

```
var localStorage *model.GarageListByUser

func init() {
    localStorage = new(model.GarageListByUser)
    localStorage.List = make(map[string]*model.GarageList)
}

type GaragesServer struct{}

```

Задача `localStorage` более или менее та же, что и в `service-user` , просто в этом приложении гаражные данные хранятся для каждого пользователя на карте.

Затем создайте интерфейс `model.GaragesServer` , затем подготовьте методы.

*   метод `Add()`

    ```
    func (GaragesServer) Add(ctx context.Context, param *model.GarageAndUserId) (*empty.Empty, error) {
       userId := param.UserId
       garage := param.Garage

       if _, ok := localStorage.List[userId]; !ok {
           localStorage.List[userId] = new(model.GarageList)
           localStorage.List[userId].List = make([]*model.Garage, 0)
       }
       localStorage.List[userId].List = append(localStorage.List[userId].List, garage)

     log.Println("Adding garage", garage.String(), "for user", userId)

       return new(empty.Empty), nil
    }

    ```

*   метод `List()`

    ```
    func (GaragesServer) List(ctx context.Context, param *model.GarageUserId) (*model.GarageList, error) {
       userId := param.UserId

       return localStorage.List[userId], nil
    }

    ```

Запустите сервер rpc, используйте его `config.SERVICE_GARAGE_PORT` в качестве порта приложения.

```
func main() {
    srv := grpc.NewServer()
    var garageSrv GaragesServer
    model.RegisterGaragesServer(srv, garageSrv)

    log.Println("Starting RPC server at", config.SERVICE_GARAGE_PORT)

    l, err := net.Listen("tcp", config.SERVICE_GARAGE_PORT)
    if err != nil {
        log.Fatalf("could not listen to %s: %v", config.SERVICE_GARAGE_PORT, err)
    }

    log.Fatal(srv.Serve(l))
}

```

## C.30.7. Клиент и приложение для тестирования

Создайте файлы `client/main.go` , импортируйте тот же пакет, что и в `service-user` или `service-garage` . Затем подготовьте два метода, которые возвращают клиента RPC, который подключен к двум службам, которые мы создали.

*   RPC гараж клиент:

    ```
    func serviceGarage() model.GaragesClient {
       port := config.SERVICE_GARAGE_PORT
       conn, err := grpc.Dial(port, grpc.WithInsecure())
       if err != nil {
           log.Fatal("could not connect to", port, err)
       }

       return model.NewGaragesClient(conn)
    }

    ```

*   Пользователь клиента RPC:

    ```
    func serviceUser() model.UsersClient {
        port := config.SERVICE_USER_PORT
        conn, err := grpc.Dial(port, grpc.WithInsecure())
        if err != nil {
            log.Fatal("could not connect to", port, err)
        }

        return model.NewUsersClient(conn)
    }

    ```

Создайте функции `main()` , заполните их созданием объектов из сгенерированных структур в пакете `model` .

```
func main() {
  user1 := model.User{
        Id:       "n001",
        Name:     "Noval Agung",
        Password: "kw8d hl12/3m,a",
        Gender:   model.UserGender(model.UserGender_value["MALE"]),
    }

    garage1 := model.Garage{
        Id:   "q001",
        Name: "Quel'thalas",
        Coordinate: &model.GarageCoordinate{
            Latitude:  45.123123123,
            Longitude: 54.1231313123,
        },
    }

  // ...
}

```

#### • Проверьте пользователя клиента RPC

Далее, получите доступ к функции, `serviceUser()` чтобы получить пользовательский объект клиента rpc. Оттуда выполнение метода `.Register()` .

```
user := serviceUser()

fmt.Println("\n", "===========> user test")

// register user1
user.Register(context.Background(), &user1)

// register user2
user.Register(context.Background(), &user2)

```

Запустите приложение `service-user` , `service-garage` и клиент, а затем увидеть результаты.

![Тест gRPC 1](https://dasarpemrogramangolang.novalagung.com/images/C.30_1_grpc_test1.png)

Как видите, вызов метода `Add()` в приложении сервера rpc прошел `service-user` успешно.

Теперь попробуйте вызвать метод `.List()` . Перезапустите клиентское приложение, чтобы увидеть результаты. О да, приложение `service-user` также перезапускается, поэтому данные не накапливаются.

```
// show all registered users
res1, err := user.List(context.Background(), new(empty.Empty))
if err != nil {
    log.Fatal(err.Error())
}
res1String, _ := json.Marshal(res1.List)
log.Println(string(res1String))

```

Как видно на следующем рисунке, вызов метода `.List()` также прошел успешно. Появятся два ранее зарегистрированных данных.

![Тест gRPC 2](https://dasarpemrogramangolang.novalagung.com/images/C.30_2_grpc_test2.png)

#### • Проверьте клиентский гараж RPC

Добавьте несколько операторов для вызова метода в `service-garage` .

*   Добавить гараж для пользователей `user1` :

    ```
    garage := serviceGarage()

    fmt.Println("\n", "===========> garage test A")

    // add garage1 to user1
    garage.Add(context.Background(), &model.GarageAndUserId{
       UserId: user1.Id,
       Garage: &garage1,
    })

    // add garage2 to user1
    garage.Add(context.Background(), &model.GarageAndUserId{
       UserId: user1.Id,
       Garage: &garage2,
    })

    ```

*   Отображает список всех гаражей, принадлежащих `user1` :

    ```
    // show all garages of user1
    res2, err := garage.List(context.Background(), &model.GarageUserId{UserId: user1.Id})
    if err != nil {
       log.Fatal(err.Error())
    }
    res2String, _ := json.Marshal(res2.List)
    log.Println(string(res2String))

    ```

*   Добавить гараж для пользователей `user2` :

    ```
    fmt.Println("\n", "===========> garage test B")

    // add garage3 to user2
    garage.Add(context.Background(), &model.GarageAndUserId{
       UserId: user2.Id,
       Garage: &garage3,
    })

    ```

*   Отображает список всех гаражей, принадлежащих `user2` :

    ```
    // show all garages of user2
    res3, err := garage.List(context.Background(), &model.GarageUserId{UserId: user2.Id})
    if err != nil {
       log.Fatal(err.Error())
    }
    res3String, _ := json.Marshal(res3.List)
    log.Println(string(res3String))

    ```

Перезапустите сервисы и клиентские приложения, посмотрите результаты.

![Тест gRPC 3](https://dasarpemrogramangolang.novalagung.com/images/C.30_3_grpc_test3.png)

---

Хорошо, если вы прочитали эту строку, это означает, что вы терпеливо преуспели в изучении gRPC в этом очень долгом обсуждении 🎉

---

*   [Protobuf](https://github.com/golang/protobuf) , от Google, лицензия BSD\-3\-Clause
*   [gRPC](https://github.com/grpc/grpc) , Google, лицензия Apache\-2.0
