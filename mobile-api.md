API mserver
=========

Выполнение запроса
---------
URL всех запросов API начинаются с префикса `https://www.synq.ru/mserver-dev/protocol/mobile/v1/`. **Мы поддерживаем только HTTPS.**
Для того, чтобы выполнить команду протокола нужно склеить префикс с именем команды, например, для команды providers URL будет `https://www.synq.ru/mserver-dev/protocol/mobile/v1/providers`.
Для представления запросов и ответов используется формат JSON. Используйте content-type: application/json.
Если вы используете утилиту curl, то команду providers можно вызвать следующим образом
```shell
curl -u user:pass https://www.synq.ru/mserver-dev/protocol/mobile/v1/providers
```

Аутентификация
---------

Мы используем стандартную Basic аутентификацию протокола HTTP. Это безопасно, поскольку мы работаем по протоколу HTTPS.
Все команды протокола, кроме `register`и `confirm`, требуют аутентификации. В качестве логина можно передавать 
номер телефона, email или номер любого из счетов кошелька.

Команды API
---------

### Регистрация пользователя
 
 `curl -d '{"phone": "79267101280", "password": "123456"}' -H 'Content-type:application/json' https://www.synq.ru/mserver-dev/protocol/mobile/v1/register`

вернет
```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "userId" : 181
  }
}
```
если пользователь успешно создан.
В случае, если пользователь с таким email или phone уже существует, в ответе будет

```json
{
  "meta" : {
    "status" : 400,
    "code" : 400,
    "developerMessage" : "User already exist",
    "moreInfoUrl" : "https://www.synq.ru",
    "exception" : "UserExistException"
  },
  "data" : {
    "PHONE_EXIST" : "79267101280"
  }
}

```

После успешной регистрации требуется подтвердить используемый номер телефона с помощью команды `confirm`.

### Подтвержение телефона пользователя
 `curl -d '{"phone": "79267101280", "pin": "123456"}' -H 'Content-type:application/json' http://localhost:8080/protocol/mobile/v1/confirm`
подтвердит номер телефона пользователя и вернет статус активации.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "activated" : true
  }
}
```


в случае неверного PIN кода ответ будет вида

```json
{
  "meta" : {
    "status" : 400,
    "code" : 400,
    "developerMessage" : "Failed to activate user. Wrong PIN.",
    "moreInfoUrl" : "https://www.synq.ru",
    "exception" : "ActivationException"
  },
  "data" : {
    "activated" : "false"
  }
}
```


### Проверка правильности учетных данных
* `POST /auth`
с телом

```json
{"applicationId": "my cool app"}
```

вернет статус `200` и данные пользователя 

```json
{
  "id" : 47,
  "phone" : "9267101280",
  "email" : "alexander@yanyshin.ru"
}
```

в случае, если переданные учетные данные верны. В противном случает будет `401`.

### Загрузка счетов кошелька

* `GET /protocol/mobile/v1/accounts`

вернет идентификаторы, имена и балансы счетов кошелька. Баланс счетов, как и везде, в рублях.

```json
[ {
  "id" : 27,
  "name" : "acct# 01",
  "amount" : 4201
}, {
  "id" : 62,
  "name" : "Ещё один счет",
  "amount" : 100
}, {
  "id" : 75,
  "name" : "Дополнительный",
  "amount" : 0
}, {
  "id" : 104,
  "name" : "Дополнительный 1",
  "amount" : 0
} ]

```

### Создание счета в кошельке

* `POST /protocol/mobile/v1/accounts`

вернет идентификатор нового счета

```json
{"accountId" : 27}

```

### Переименование счета

* `POST /protocol/mobile/v1/accounts/27`

с телом

```json
{"name" : "Мой новый счет"}
```

переименует счет с ID 27.



### Загрузка справочника провайдеров

* `GET /protocol/mobile/v1/providers`

вернет справочник провайдеров с разбиением по группам

```json
[ {
  "id" : 5,
  "name" : "Мобильная связь СНГ",
  "providers" : [ {
    "id" : 710881731,
    "name" : "MegaCom(АльфаТелеком) Кыргызстан (7)"
  }, {
    "id" : 518042683,
    "name" : "Билайн-Армения(АрменТел) (7)"
  } ]
}, {
  "id" : 6,
  "name" : "Мобильная связь",
  "providers" : [ {
    "id" : 750739964,
    "name" : "Tele2 без комиссии"
  }, {
    "id" : 890983449,
    "name" : "Билайн без комиссии"
  }]
}]
```

### Загрузка описания провайдера

* `GET /protocol/mobile/v1/providers/${providerId}`

вернет описание провайдера, например:

```json
{
  "id" : 890983449,
  "name" : "Билайн без комиссии",
  "minsum" : 2,
  "maxsum" : 15000,
  "parameters" : [ {
    "id" : 971,
    "code" : "phoneNumber",
    "minLength" : 10,
    "maxLength" : 10,
    "title" : "Номер телефона",
    "pattern" : null,
    "type" : null,
    "patternDescription" : "10-ти значный федеральный номер",
    "main" : true
  } ]
}
```


### Отправка платежа

* `POST /protocol/mobile/v1/pay`

с телом

```json
{
  "serviceId" : 453315258,
  "account" : 27,
  "amount" : 1
  "parameters" : {"phoneNumber" : "9267101280"}
}
```

отправит платеж на 1 рубль на номер 9267101280  со счета 27 и вернет ID платежа.

```json
{
  "transactionId" : 4654456882  
}
```

### Перевод между счетами кошельков

* `POST /protocol/mobile/v1/transfer`

с телом

```json
{
  "source" : 27,
  "destination" : 42,
  "amount": 1
}
```

отправит перевод на 1 рубль с 27 на 42 счет. Счет назначения может быть задан номером счета, номером телефона кошелька 
или email кошелька.
Результатом выполнения команды будет ID платежной транзакции.


отправит платеж на 1 рубль на номер 9267101280  со счета 27 и вернет ID платежа.

```json
{
  "transactionId" : 4654456882  
}
```

### Сохранение геометки платежа

* `POST /protocol/mobile/v1/location`

с телом

```json
{
  "transactionId" : 4654456882,
  "location": {"latitude" : 23.45, "longitude" : 45.34}
}
```

ассоциирует с платежной транзакцией геометку.


### Поиск пользователей по телефонам/адресам эл. почты

* `POST /protocol/mobile/v1/findUsers`

с телом

```json
{
"logins": ["+79267101280", "alexander@yanyshin.ru"]
}

```

вернет список найденных пользователей:

```json
[ {
  "id" : 47,
  "phone" : "9267101280",
  "email" : "alexander@yanyshin.ru"
}, 
{
  "id" : 62,
  "phone" : "+79267101280",
  "email" : null
} ]

```

### Получение истории транзакций по счету

* `GET /protocol/mobile/v1/history/27/20130623/20130704`

вернет историю платежных транзакций по 27 счету между 23 июня и 4 июля 2013.


```json
{
  "history" : [ {
    "id" : 5074,
    "date" : 1372174037178,
    "amount" : 1,
    "sourceAccountId" : 27,
    "destinationAccountId" : 3,
    "transactionType" : "PAY",
    "location" : null
  }, {
    "id" : 5082,
    "date" : 1372251562237,
    "amount" : 1,
    "sourceAccountId" : 27,
    "destinationAccountId" : 3,
    "transactionType" : "PAY",
    "location" : null
  }, {
    "id" : 5084,
    "date" : 1372251800462,
    "amount" : 1,
    "sourceAccountId" : 27,
    "destinationAccountId" : 3,
    "transactionType" : "PAY",
    "location" : null
  } ]

```
