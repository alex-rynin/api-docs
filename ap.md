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
