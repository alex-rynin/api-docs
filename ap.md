API Автоплатежа
=========

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
| :-------------------: |---------------------------------------------------------------------------------------------------------------|
| SCHEDULE              | Автоплатеж по расписанию, в поле `schedule` следует передавать выражение cron                                 |
| THRESHOLD             | Автоплатеж по порогу, в поле `threshold` передается порог срабатывания, в поле `amount` - сумма пополнения    |
| INVOICE               | Автоплатеж по счету на оплату                                                                                 |

