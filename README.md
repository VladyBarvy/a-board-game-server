# Hexlet Chess Server

Серверная часть работы на летнюю практику Hexlet колледжа

# установка и запуск

Скопировать репозиторий

```
git clone https://github.com/Felarn/a-board-game-server.git
```

для локального запуска:

```
make dev-install
make dev-start
```

сервер запустится через nodemon и будет слушать порт 4444. Порт можно изменить в сгенерированном файле c настройками <em>developement.config</em>

для развертывания на сервере:

```
make deploy
make start
```

сервер будет слушать порт 80

# Подключение к серверу

Для подключения к вебсокет-серверу и начала работы с ним достаточно следующего скрипта, исполненного в браузере:

```js
// const serverURL = "ws://localhost:4444"; // для подключения к локальному серверу
const serverURL = "wss://felarn.ru"; // для подключения к online-серверу
const connection = new WebSocket(serverURL);

connection.onopen = () => {  // событие при установке соединения с сервером он должен отправить сообщение: '{"action":"debug","payload":{"message":"You've connected to Hexlet Chess server"}}'
  console.log("WebSocket connection established");
  connection.send("hello"); // отправка сообщения. Сервер должен будет ответить:
// 'Hi from Hexlet Chess
// You shold use {acton: %command name%, payload: {%data object%}} format 
// to communicate with this server'
};

connection.onmessage = (message) => { // слушатель событий, все ответы сервера проходят через него
  console.log(message.data); // в консоль должны будут вывестись ответы на полключение и send("hello")
};
```

# общение с сервером
Общение происходит путем отправки сообщений на сервер и ожидания сообщений от сервера\
Сообщения от сервера могут приходить как в ответ на запрос клиента, так и по инициативе сервера без запроса.\
Например когда в ходе партии опонент закончит свой ход - сервер пришлет клиенту сообдение с информацией о новом состоянии доски.\
\
Формат сообщений для общения с сервером - JSON.\
Структура объекта, который ожидает и отправляет сервер следующая:
```js
{
action:'%имя команды%',
payload:
  {<объект с полями зависящими от %имени команды%>}
}
```
если payload командой не подразумевается, то можно его опустить или задать payload:null.

## Команды серверу

### Идентификация и регистрация
Для того, чтобы начать полноценое общение с сервером необходимо каждое новое подключение идентифицировать.
Если клиент подключается к серверу впервые - необходимо отправить команду:
```js
{action:"registration"}
либо
{action:"registration",payload:{userName:"%имя пользователя%"}}
```
ответ сервера содержит ID клиента, который следует использовать в дальнейшем при подключении к серверу:
```js
{action:"registered",payload:{userID:"63c8e399-7314-46ce-bdbb-f4244d9cd99d"}}
```
При повторном подключении используется
``` js
{action:"identification",payload:{userID:"63c8e399-7314-46ce-bdbb-f4244d9cd99d"}}
```
ответ если такой пользователь существует содержит:\
* имя пользователя
* состояние пользователя
+ другие поля будут добавлены при необходимости, и будут включать, в том числе и всю информацию о текущей игре
```js
{action:"identified",payload:{userName:"Anon",userCondition:"outOfGame"}}
```
если пользователь с таким ID не найден, то сервер произведет регистрацию нового пользователя и ответит той же структурой, что и при регистрации
```js
{action:"registered",payload:{userID:"4347cfb0-d952-45c5-ab09-186c0a7a64c2"}}
```
### Переименование
доступно в состоянии игрока 'outOFGame' и 'inLobby'
```js
{action:"rename",payload:{userName:"%новое имя%"}}
```
в случае 'outOfGame' ответа от сервера не предполагается
в случае 'inGame' сервер оповестит всех игроков в комнате, разослав новый список имен
```js
WIP
```

### Чат
возможна отправка сообщений при состоянии игрока "inLobby" и "inGame"
```js
{"action":"chat","payload":{"message":"Hello"}}
```
все участники текущей игры получают сообщение
```js
{action:"chat",payload:{message:"Hello",from:"%имя отправителя%"}}
```


### Вход в комнату 
```js
{action: "join", payload: { gameID: gameID }}
```
от сервера прийдет 
```js
{action: "newUserCondition", payload: { userCondition: "inLobby" }}
```
Позже добавлю, что вместе с этим прилетит второе сообщение с содержимым комнаты (игроки, ID, название комнаты)

### Выход из комнаты 
```js
{action: "leave", payload: null} или просто {action: "leave"}
```
в ответ сервер пришлет 
```js
{action: "newUserCondition", payload: { userCondition: "outOfGame" }}
```

## сообщения от сервера

### Изменение "страницы"
Оповещение игрока о том, что он "вышел в главное меню"/"зашел в комнату"/"начал матч"/"перешел на финальный экран"
```js
{action: "newUserCondition", payload: { userCondition: userCondition }}
```
userCondition может принимать следующие значения:
* 'outOfGame' - главное меню
* 'inLobby' - в лобби, фаза подготовки игры
* 'inGame' - находится в матче
+ ('resultScreen' - находится на экране с результатом матча) стоит обсудить

### сообщения для дебага
```js
{action:"debug",payload:{message:"You've connected to Hexlet Chess server"}}
```
