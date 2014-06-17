API Автоплатежа
=========

- [API Автоплатежа](#user-content-api-%D0%90%D0%B2%D1%82%D0%BE%D0%BF%D0%BB%D0%B0%D1%82%D0%B5%D0%B6%D0%B0)
- [Общее](#user-content-%D0%9E%D0%B1%D1%89%D0%B5%D0%B5)
	- [Аутентификация](#user-content-%D0%90%D1%83%D1%82%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F)
	- [Ошибки](#user-content-%D0%9E%D1%88%D0%B8%D0%B1%D0%BA%D0%B8)
- [Сервисы](#user-content-%D0%A1%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D1%8B)
	- [Загрузка списка сервисов](#user-content-%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D1%81%D0%BF%D0%B8%D1%81%D0%BA%D0%B0-%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D0%BE%D0%B2)
	- [Загрузка сервиса по идентификатору](#user-content-%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D0%B0-%D0%BF%D0%BE-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80%D1%83)
- [Заявки](#user-content-%D0%97%D0%B0%D1%8F%D0%B2%D0%BA%D0%B8)
	- [Создание заявки](#user-content-%D0%A1%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B7%D0%B0%D1%8F%D0%B2%D0%BA%D0%B8)
	- [Загрузка заявки по идентификатору](#user-content-%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D0%B7%D0%B0%D1%8F%D0%B2%D0%BA%D0%B8-%D0%BF%D0%BE-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80%D1%83)
	- [Загрузка списка заявок](#user-content-%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D1%81%D0%BF%D0%B8%D1%81%D0%BA%D0%B0-%D0%B7%D0%B0%D1%8F%D0%B2%D0%BE%D0%BA)
	- [Временная блокировка заявки](#user-content-%D0%92%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%BD%D0%B0%D1%8F-%D0%B1%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0-%D0%B7%D0%B0%D1%8F%D0%B2%D0%BA%D0%B8)
	- [Снятие временной блокировки](#user-content-%D0%A1%D0%BD%D1%8F%D1%82%D0%B8%D0%B5-%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%BD%D0%BE%D0%B9-%D0%B1%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B8)
	- [Отключение автоплатежа для заявки](#user-content-%D0%9E%D1%82%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B0%D0%B2%D1%82%D0%BE%D0%BF%D0%BB%D0%B0%D1%82%D0%B5%D0%B6%D0%B0-%D0%B4%D0%BB%D1%8F-%D0%B7%D0%B0%D1%8F%D0%B2%D0%BA%D0%B8)
	- [Типы заявок](#user-content-%D0%A2%D0%B8%D0%BF%D1%8B-%D0%B7%D0%B0%D1%8F%D0%B2%D0%BE%D0%BA)
	- [Статусы заявки](#user-content-%D0%A1%D1%82%D0%B0%D1%82%D1%83%D1%81%D1%8B-%D0%B7%D0%B0%D1%8F%D0%B2%D0%BA%D0%B8)
- [Счета](#user-content-%D0%A1%D1%87%D0%B5%D1%82%D0%B0)
	- [Загрузка счета по идентификатору](#user-content-%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D1%81%D1%87%D0%B5%D1%82%D0%B0-%D0%BF%D0%BE-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80%D1%83)
	- [Загрузка списка счетов](#user-content-%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D1%81%D0%BF%D0%B8%D1%81%D0%BA%D0%B0-%D1%81%D1%87%D0%B5%D1%82%D0%BE%D0%B2)
	- [Подтверждение счета для оплаты](#user-content-%D0%9F%D0%BE%D0%B4%D1%82%D0%B2%D0%B5%D1%80%D0%B6%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5-%D1%81%D1%87%D0%B5%D1%82%D0%B0-%D0%B4%D0%BB%D1%8F-%D0%BE%D0%BF%D0%BB%D0%B0%D1%82%D1%8B)
	- [Отклонение счета для оплаты](#user-content-%D0%9E%D1%82%D0%BA%D0%BB%D0%BE%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D1%81%D1%87%D0%B5%D1%82%D0%B0-%D0%B4%D0%BB%D1%8F-%D0%BE%D0%BF%D0%BB%D0%B0%D1%82%D1%8B)
	- [Статусы счета](#user-content-%D0%A1%D1%82%D0%B0%D1%82%D1%83%D1%81%D1%8B-%D1%81%D1%87%D0%B5%D1%82%D0%B0)


# Общее

## Аутентификация

Используется BASIC аутентификация со включенными сессиями. Можно слать логин/пароль каждый раз, можно прислать 1 раз за сессию, и в дальнейшем слать токен сессии в куке `JSESSONID`. В качестве логина используется код проекта.

Сервер автоплатежа, запущенный с --profile=dev при старте создает тестовый проект `mbank` с паролем `password`, и тестовый сервис `МТС` с одним параметром с разрешенными типами заявок `SCHEDULE` и `THRESHOLD` при условии, что этих объектов в базе нет.

## Ошибки

API использует HTTP статусы для возврата ошибок.

| Статус                | Описание                               |
| :-------------------: |-----------------------------------------------------------------------------------------------------------------|
| 2XX                   |  Запрос выполнен успешно                                                                                        |
| 4XX                   |  Проблема с запросом:                                                                                           |
| 400                   |  - плохо сформированный запрос (например невалидный JSON)                                                       |
| 401                   |  - неверные учетные данные                                                                                      |
| 404                   |  - запрошенный ресурс не существует (для запрашивающего)                                                        |
| 405                   |  - не тот HTTP метод, например GET вместо POST                                                                  |
| 415                   |  - не тот content-type, мы поддерживаем только `application/json`                                               |
| 422                   |  - запрос сформирован нормально, но не прошел валидацию, например не хватает полей или поле не в нужном формате |
| 5XX                   |  Проблема с сервером автоплатежа, повторите запрос позже или обратитесь в поддержку                             |

HTTP статус дублируется в поле `meta.status` ответа сервера. В определенных случаях вместе со статусом возвращается уточненный код ошибки в поле `meta.error`.

# Сервисы

## Загрузка списка сервисов

```shell
$ curl -u mbank:password http://localhost:8080/v1/services
```

```json
 {
  "meta" : {
    "code" : 200
  },
  "data" : [ {
    "id" : 1,
    "name" : "МТС",
    "parameters" : [ {
      "code" : "phoneNumber",
      "min_length" : 10,
      "max_length" : 10,
      "pattern" : "^\\d{10}$",
      "pattern_description" : "Номер телефона в 10 значном формате",
      "name" : "Номер телефона",
      "type" : "numeric"
    } ],
    "order_types" : [ "schedule", "threshold" ]
  } ]
}
```

## Загрузка сервиса по идентификатору

```shell
$ curl -u mbank:password http://localhost:8080/v1/services/1
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 1,
    "name" : "МТС",
    "parameters" : [ {
      "code" : "phoneNumber",
      "min_length" : 10,
      "max_length" : 10,
      "pattern" : "^\\d{10}$",
      "pattern_description" : "Номер телефона в 10 значном формате",
      "name" : "Номер телефона",
      "type" : "numeric"
    } ],
    "order_types" : [ "schedule", "threshold" ]
  }
}
```

# Заявки

## Создание заявки

```shell
$ curl -u mbank:password -H "Content-type:application/json" -d '{"type": "threshold", "parameters": {"phoneNumber": "91011122233"}, "service": 1, "amount": 50, "threshold": 100}' http://localhost:8080/v1/orders
```

Создана заявка на автоплатеж для сервиса МТС для номера 91011122233 по порогу. Порог срабатывания и сумма пополнения установлены в 50 рублей.

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 7,
    "type" : "threshold",
    "status" : "activating",
    "service" : {
      "id" : 1,
      "name" : "МТС"
    },
    "amount" : 50,
    "threshold" : 100,
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "Номер телефона",
      "value" : "91011122233"
    } ]
  }
}
```

## Загрузка заявки по идентификатору

```shell
$ curl -u mbank:password http://localhost:8080/v1/orders/7
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 7,
    "type" : "threshold",
    "status" : "activating",
    "service" : {
      "id" : 1,
      "name" : "МТС"
    },
    "amount" : 50,
    "threshold" : 100,
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "Номер телефона",
      "value" : "91011122233"
    } ]
  }
}
```

## Загрузка списка заявок

```shell
$ curl -u mbank:password http://localhost:8080/v1/orders
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : [ {
    "id" : 7,
    "type" : "threshold",
    "status" : "activating",
    "service" : {
      "id" : 1,
      "name" : "МТС"
    },
    "amount" : 50,
    "threshold" : 100,
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "Номер телефона",
      "value" : "91011122233"
    } ]
  }, {
    "id" : 4,
    "type" : "threshold",
    "status" : "active",
    "service" : {
      "id" : 1,
      "name" : "МТС"
    },
    "amount" : 90,
    "threshold" : 150,
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "Номер телефона",
      "value" : "91011122234"
    } ]
  } ]
}
```

## Временная блокировка заявки

```shell
$ curl -u mbank:password -H "Content-type:application/json" -X POST http://localhost:8080/v1/orders/4/suspend
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 4,
    "type" : "threshold",
    "status" : "suspended",
    "service" : {
      "id" : 1,
      "name" : "МТС"
    },
    "amount" : 50,
    "threshold" : 100,
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "Номер телефона",
      "value" : "91011122233"
    } ]
  }
}
```

## Снятие временной блокировки

```shell
$ curl -u mbank:password -H "Content-type:application/json" -X POST http://localhost:8080/v1/orders/4/resume
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 4,
    "type" : "threshold",
    "status" : "active",
    "service" : {
      "id" : 1,
      "name" : "МТС"
    },
    "amount" : 50,
    "threshold" : 100,
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "Номер телефона",
      "value" : "91011122233"
    } ]
  }
}
```

## Отключение автоплатежа для заявки

```shell
$ curl -u mbank:password -H "Content-type:application/json" -X DELETE http://localhost:8080/v1/orders/4
```

```json
{
  "meta" : {
    "code" : 200
  }
}
```

## Типы заявок

| Код типа              | Описание                                                                                                      |
| :-------------------  |---------------------------------------------------------------------------------------------------------------|
| SCHEDULE              | Автоплатеж по расписанию, в поле `schedule` следует передавать выражение cron                                 |
| THRESHOLD             | Автоплатеж по порогу, в поле `threshold` передается порог срабатывания, в поле `amount` - сумма пополнения    |
| INVOICE               | Автоплатеж по счету на оплату                                                                                 |

## Статусы заявки

| Код статуса           | Описание                                                                                                      |
| :-------------------  |---------------------------------------------------------------------------------------------------------------|
| ACTIVATING            | Автоплатеж подключается                                                                                       |
| ACTIVE                | Автоплатеж подключен                                                                                          |
| SUSPENDED             | Автоплатеж временно заблокирован                                                                              |
| DEACTIVATED           | Автоплатеж отключен                                                                                           |
| CANCELED              | Пользователь отказался от подключения Автоплатежа                                                             |

# Счета

## Загрузка счета по идентификатору

```shell
$ curl -u mbank:password http://localhost:8080/v1/invoices/2
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 2,
    "order_id" : 42,
    "status" : "created",
    "created_at" : "2014-06-17T13:36:33.661Z"
  }
}
```

## Загрузка списка счетов

```shell
$ curl -u mbank:password http://localhost:8080/v1/invoices
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : [ {
    "id" : 3,
    "order_id" : 43,
    "status" : "confirmed",
    "created_at" : "2014-06-17T13:46:05.730Z"
  }, {
    "id" : 2,
    "order_id" : 42,
    "status" : "created",
    "created_at" : "2014-06-17T13:36:33.661Z"
  }, {
    "id" : 1,
    "order_id" : 41,
    "status" : "rejected",
    "created_at" : "2014-06-17T13:12:14.207Z"
  } ]
}
```

## Подтверждение счета для оплаты

```shell
$ curl -u mbank:password -X POST http://localhost:8080/v1/invoices/3/confirm
```

```json
{
 "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 3,
    "order_id" : 43,
    "status" : "confirmed",
    "created_at" : "2014-06-17T13:46:05.730Z"
  }
}
```

## Отклонение счета для оплаты

```shell
$  curl -u mbank:password -X POST http://localhost:8080/v1/invoices/1/reject
```

```json
{
"meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 1,
    "order_id" : 41,
    "status" : "rejected",
    "created_at" : "2014-06-17T13:12:14.207Z"
  }
}
```

## Статусы счета

| Код статуса           | Описание                   |
| :---------------------|----------------------------|
| CREATED               | Счет создан, ожидает подтверждения или отклонения |
| CONFIRMED             | Счет подтвержден и будет оплачен  |
| REJECTED              | Счет отклонен и НЕ будет оплачен  |
