API mserver 2
=========

# 1. Общее

## 1.1 Аутентификация.

Используется BASIC аутентификация со включенными сессиями. Можно слать логин/пароль каждый раз, можно прислать 1 раз за сессию, и в дальнейшем слать токен сессии в куке JSESSONID. В качестве логина можно присылать номер телефона или email.

## 1.2 Ошибки.

API использует HTTP статусы для возврата ошибок.

2XX - запрос выполнен успешно, сейчас используется только 200 ровно
4XX - проблема с запросом
 400 - плохо сформированный запрос (например невалидный JSON)
 401 - неверные учетные данные
 404 - запрошенный ресурс не существует (например при запросе статуса платежа, которого нет в системе или его провели с другого кошелька)
 405 - не тот HTTP метод, например GET вместо POST
 415 - не тот content-type, мы поддерживаем только application/json
 422 - запрос сформирован нормально, но не прошел валидацию, например не хватает полей или поле не в нужном формате
5XX - проблема с mserver, повторите запрос позже

HTTP статус дублируется в поле meta.status ответа mserver. В определенных случаях вместе со статусом возвращается уточненный код ошибки в поле meta.error. Например для невалидного номера телефона при создании кошелка возвращается

```json
{
  "meta" : {
    "code" : 422,
    "error" : "invalid_phone"
  }
}
```

Справочники возможных значений error для каждого вызова API описаны ниже.

# 2. Кошелек.

## 2.1 Создание кошелька.

phone - номер телефона в международном формате
password - пароль, не короче 6 символов

$ curl -H 'Content-type:application/json' -d '{"phone": "+79261111111", "password": "p@ssw0rD"}' https://www.synq.ru/mserver2-dev/v1/wallet

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "phone" : "+79261111111",    
    "amount" : 0
  }
}
```

## 2.2 Загрузка кошелька.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/wallet

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {    
    "phone" : "+79261111111",    
    "amount" : 0
  }
}
```

# 3. Сервисы.

## 3.1 Загрузка списка сервисов.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/services

```json
 {
  "meta" : {
    "code" : 200
  },
  "data" : [ {
    "id" : 9,
    "name" : "Альфа Банк",
    "min" : 10,
    "max" : 15000,
    "parameters" : [ {
      "code" : "phoneNumber",
      "min_length" : 16,
      "max_length" : 16,
      "name" : "№ карты (16 цифр)",
      "pattern" : "^\\d{16}$",
      "type" : "numeric",
      "pattern_description" : "№ карты (16 цифр)",
      "items" : [ ]
    }, {
      "code" : "valid",
      "min_length" : null,
      "max_length" : null,
      "name" : "Срок действия (вид: ГГММ)",
      "pattern" : "^\\d{4}$",
      "type" : "text",
      "pattern_description" : "Срок действия (вид: ГГММ)",
      "items" : [ ]
    } ]
  }, 
/* остальная простыня опущена*/
  ...]
}
```

## 3.2 Загрузка сервиса по идентификатору.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/services/1

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 1,
    "name" : "Мегафон",
    "min" : 1,
    "max" : 15000,
    "parameters" : [ {
      "code" : "phoneNumber",
      "min_length" : 10,
      "max_length" : 10,
      "name" : "№ телефона (10 цифр)",
      "pattern" : "^\\d{10}$",
      "type" : "numeric",
      "pattern_description" : "№ телефона (10 цифр)",
      "items" : [ ]
    } ]
  }
}
```

## 3.3 Сервисы по группам.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/services/groups

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : [ {
    "code" : "bank",
    "name" : "Банковские услуги",
    "services" : [ {
      "id" : 9,
      "name" : "Альфа Банк",
      "min" : 10,
      "max" : 15000,
      "parameters" : [ {
        "code" : "phoneNumber",
        "min_length" : 16,
        "max_length" : 16,
        "name" : "№ карты (16 цифр)",
        "pattern" : "^\\d{16}$",
        "type" : "numeric",
        "pattern_description" : "№ карты (16 цифр)",
        "items" : [ ]
      }, {
        "code" : "valid",
        "min_length" : null,
        "max_length" : null,
        "name" : "Срок действия (вид: ГГММ)",
        "pattern" : "^\\d{4}$",
        "type" : "text",
        "pattern_description" : "Срок действия (вид: ГГММ)",
        "items" : [ ]
      } ]
    }, {
      "id" : 11,
      "name" : "Банк Русский Стандарт",
/* ... */
```

# 4. Платежи.

В качестве клиентского идентификатора платежа (client_payment_id) используется UUID. Если в процессе создания платежа клиент получил ошибку i/o, он должен повторять запрос создания платежа с тем же самым client_payment_id до получения ответа. Гарантируется, что платеж с данным client_payment_id будет создан не более 1 раза.

## 4.1 Простые сервисы (одношаговые).

### 4.1.1. Создание платежа в пользу провайдера.

$ curl -u -u+79261111111:p@ssw0rD -H 'Content-type:application/json' -d '{"type": "out", "client_payment_id": "071c6d23-7508-4e35-ad92-852308a47677", "amount": 100, "service": 1, "parameters": {"phoneNumbe": "9267101280"}}' https://www.synq.ru/mserver2-dev/v1/payments

```json
{
  "meta" : {
    "code" : 200,
    "next_action" : "pay" /* намек клиенту, что делать дальше, может быть pay (запустить на оплату), update (нужно предоставить доп. параметры), get (опрашивать до появления update или pay)*/
  },
  "data" : {
    "id" : 1401089234881, /* идентификатор платежа на стороне mserver, используется для всех действий с платежом */
    "client_payment_id" : "071c6d23-7508-4e35-ad92-852308a47677",
    "amount" : 100, /* сумма, которую запросил клиент */
    "total" : 100,  /* полная сумма платежа, которая будет списана со счета кошелька (может быть больше amount за счет комиссий) */
    "created_at" : "2014-05-26T13:22:40.837Z",
    "status" : "processing", /* статус платежа, может быть processing (проводится), finished (проведен), canceled (отклонен) */
    "type" : "out",
    "service" : {
      "id" : 1,
      "name" : "Мегафон"
    },
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "№ телефона (10 цифр)",
      "value" : "9267101280"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.1.2. Пользователь согласен с условиями, запускаем платеж в обработку.

$ curl -u+79261111111:p@ssw0rD -H 'Content-type:application/json' -X POST https://www.synq.ru/mserver2-dev/v1/payments/1401089234881/pay

```json
{
  "meta" : {
    "code" : 200,
    "next_action" : "get"
  },
  "data" : {
    "id" : 1401089234881,
    "client_payment_id" : "071c6d23-7508-4e35-ad92-852308a47677",
    "amount" : 100,
    "total" : 100,
    "created_at" : "2014-05-26T13:22:40.837Z",
    "status" : "processing",
    "type" : "out",
    "service" : {
      "id" : 1,
      "name" : "Мегафон"
    },
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "№ телефона (10 цифр)",
      "value" : "9267101280"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.1.3. Загрузка платежа.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/payments/1401089234881

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 1401089234881,
    "client_payment_id" : "071c6d23-7508-4e35-ad92-852308a47677",
    "amount" : 100,
    "total" : 100,
    "created_at" : "2014-05-26T13:22:40.837Z",
    "processed_at" : "2014-05-26T13:24:20.962Z",
    "status" : "finished",
    "type" : "out",
    "service" : {
      "id" : 1,
      "name" : "Мегафон"
    },
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "№ телефона (10 цифр)",
      "value" : "9267101280"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

Платеж проведен.

## 4.2 Многошаговые платежи.

### 4.2.1 Создание платежа в пользу провайдера.

$ curl -u+79261111111:p@ssw0rD -H 'Content-type:application/json' -d '{"type": "out", "client_payment_id": "e731a7e2-c553-4295-867e-1023359bee28", "amount": 100, "service": 61, "parameters": {"phoneNumber": "9267101283", "BIK": "044583151", "Name": "name", "SName": "sname", "Fam": "fam"}}' https://www.synq.ru/mserver2-dev/v1/payments

```json
{
  "meta" : {
    "code" : 200,
    "next_action" : "get"
  },
  "data" : {
    "id" : 1401089234883,
    "client_payment_id" : "e731a7e2-c553-4295-867e-1023359bee28",
    "amount" : 100,
    "created_at" : "2014-05-26T15:15:07.389Z",
    "status" : "processing",
    "type" : "out",
    "service" : {
      "id" : 61,
      "name" : "Мультибанк"
    },
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "№ Телефона (10 цифр)",
      "value" : "9267101283"
    }, {
      "code" : "BIK",
      "name" : "БИК",
      "value" : "044583151"
    }, {
      "code" : "Name",
      "name" : "Имя Отправителя",
      "value" : "name"
    }, {
      "code" : "SName",
      "name" : "Отчество Отправителя",
      "value" : "sname"
    }, {
      "code" : "Fam",
      "name" : "Фамилия Отправителя",
      "value" : "fam"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.2.2 Следующий шаг, запрос состояния платежа.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/payments/1401089234883

```json
{
  "meta" : {
    "code" : 200,
    "next_action" : "update",
    "current_parameters" : [ {  /* current_parameters служит для показа клиенту / запроса дополнительной информации */
      "code" : "bank",          /* параметры с editable = true нужно дать заполнить пользователю и расширить этими полями платеж вызом update */
      "name" : "Банк",
      "value" : "ЗАО \"БАНК РУССКИЙ СТАНДАРТ\"",
      "editable" : false
    }, {
      "code" : "dest",
      "name" : "Назначение платежа",
      "value" : "Зачисление на счет/карту/N договора",
      "editable" : false
    }, {
      "code" : "Pname1",
      "name" : "Номер счета/карты/договора",
      "value" : "",
      "editable" : true
    }, {
      "code" : "Pname2",
      "name" : "ФИО владельца счета/карты/договора",
      "value" : "",
      "editable" : true
    }, {
      "code" : "whatToDo",
      "name" : "Что делать?",
      "value" : "",
      "editable" : true
    } ]
  },
  "data" : {
    "id" : 1401089234883,
    "client_payment_id" : "e731a7e2-c553-4295-867e-1023359bee28",
    "amount" : 100,
    "created_at" : "2014-05-26T15:15:07.389Z",
    "status" : "processing",
    "type" : "out",
    "service" : {
      "id" : 61,
      "name" : "Мультибанк"
    },
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "№ Телефона (10 цифр)",
      "value" : "9267101283"
    }, {
      "code" : "BIK",
      "name" : "БИК",
      "value" : "044583151"
    }, {
      "code" : "Name",
      "name" : "Имя Отправителя",
      "value" : "name"
    }, {
      "code" : "SName",
      "name" : "Отчество Отправителя",
      "value" : "sname"
    }, {
      "code" : "Fam",
      "name" : "Фамилия Отправителя",
      "value" : "fam"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.2.3 Следующий шаг, обновление платежа, передача запрошенных на предыдущем шаге дополнительных параметров (и всех остальных параметров тоже).

$ curl -u+79261111111:p@ssw0rD -H 'Content-type:application/json' -d '{"parameters": {"phoneNumber": "9267101283", "BIK": "044583151", "Name": "name", "SName": "sname", "Fam": "fam", "Pname1": "5136913818331704", "Pname2": "fio"}}' https://www.synq.ru/mserver2-dev/v1/payments/1401089234883

```json
{
  "meta" : {
    "code" : 200,
    "next_action" : "get"
  },
  "data" : {
    "id" : 1401089234883,
    "client_payment_id" : "e731a7e2-c553-4295-867e-1023359bee28",
    "amount" : 100,
    "created_at" : "2014-05-26T15:15:07.389Z",
    "status" : "processing",
    "type" : "out",
    "service" : {
      "id" : 61,
      "name" : "Мультибанк"
    },
    "parameters" : [ {
      "code" : "phoneNumber",
      "name" : "№ Телефона (10 цифр)",
      "value" : "9267101283"
    }, {
      "code" : "BIK",
      "name" : "БИК",
      "value" : "044583151"
    }, {
      "code" : "Name",
      "name" : "Имя Отправителя",
      "value" : "name"
    }, {
      "code" : "SName",
      "name" : "Отчество Отправителя",
      "value" : "sname"
    }, {
      "code" : "Fam",
      "name" : "Фамилия Отправителя",
      "value" : "fam"
    }, {
      "code" : "Pname1",
      "name" : "Номер счета/карты/договора",
      "value" : "5136913818331704"
    }, {
      "code" : "Pname2",
      "name" : "ФИО владельца счета/карты/договора",
      "value" : "fio"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.2.4 Следующий шаг, запрос состояния платежа.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/payments/1401089234883

```json
{
  "meta" : {
    "code" : 200,
    "next_action" : "pay", /* подготовка платежа к проведению закончена, можно платить */
    "current_parameters" : [ {
      "code" : "knp",
      "name" : "Номер чека на стороне провайдера",
      "value" : "486221871621136659",
      "editable" : false
    } ]
  },
  "data" : {
    "id" : 1401089234883,
    "client_payment_id" : "e731a7e2-c553-4295-867e-1023359bee28",
    "amount" : 100,
    "total" : 135, /* рассчитана полная сумма платежа, с учетом комиссий, следует показать ее клиенту перед проведением платежа */
    "created_at" : "2014-05-26T15:15:07.389Z",
    "status" : "processing",
    "type" : "out",
    "service" : {
      "id" : 61,
      "name" : "Мультибанк"
    },
    "parameters" : [ {
      "code" : "Pname1",
      "name" : "Номер счета/карты/договора",
      "value" : "5136913818331704"
    }, {
      "code" : "Pname2",
      "name" : "ФИО владельца счета/карты/договора",
      "value" : "fio"
    }, {
      "code" : "phoneNumber",
      "name" : "№ Телефона (10 цифр)",
      "value" : "9267101283"
    }, {
      "code" : "BIK",
      "name" : "БИК",
      "value" : "044583151"
    }, {
      "code" : "Name",
      "name" : "Имя Отправителя",
      "value" : "name"
    }, {
      "code" : "SName",
      "name" : "Отчество Отправителя",
      "value" : "sname"
    }, {
      "code" : "Fam",
      "name" : "Фамилия Отправителя",
      "value" : "fam"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.2.5 Пользователь согласен с условиями, запускаем платеж в обработку.

$ curl -u+79261111111:p@ssw0rD -X POST https://www.synq.ru/mserver2-dev/v1/payments/1401089234883/pay

```json
{
  "meta" : {
    "code" : 200,
    "next_action" : "get" /* можно/нужно проверять состояние проводимого платежа */
  },
  "data" : {
    "id" : 1401089234883,
    "client_payment_id" : "e731a7e2-c553-4295-867e-1023359bee28",
    "amount" : 100,
    "total" : 135,
    "created_at" : "2014-05-26T15:15:07.389Z",
    "status" : "processing",
    "type" : "out",
    "service" : {
      "id" : 61,
      "name" : "Мультибанк"
    },
    "parameters" : [ {
      "code" : "Pname1",
      "name" : "Номер счета/карты/договора",
      "value" : "5136913818331704"
    }, {
      "code" : "Pname2",
      "name" : "ФИО владельца счета/карты/договора",
      "value" : "fio"
    }, {
      "code" : "phoneNumber",
      "name" : "№ Телефона (10 цифр)",
      "value" : "9267101283"
    }, {
      "code" : "BIK",
      "name" : "БИК",
      "value" : "044583151"
    }, {
      "code" : "Name",
      "name" : "Имя Отправителя",
      "value" : "name"
    }, {
      "code" : "SName",
      "name" : "Отчество Отправителя",
      "value" : "sname"
    }, {
      "code" : "Fam",
      "name" : "Фамилия Отправителя",
      "value" : "fam"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.2.6 Последний шаг, запрос состояния платежа.

$ curl -u+79261111111:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/payments/1401089234883y

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 1401089234883,
    "client_payment_id" : "e731a7e2-c553-4295-867e-1023359bee28",
    "amount" : 100,
    "total" : 135,
    "created_at" : "2014-05-26T15:15:07.389Z",
    "processed_at" : "2014-05-26T15:37:11.589Z",
    "status" : "finished",  /* Платеж успешно проведен */
    "type" : "out",
    "service" : {
      "id" : 61,
      "name" : "Мультибанк"
    },
    "parameters" : [ {
      "code" : "Pname1",
      "name" : "Номер счета/карты/договора",
      "value" : "5136913818331704"
    }, {
      "code" : "Pname2",
      "name" : "ФИО владельца счета/карты/договора",
      "value" : "fio"
    }, {
      "code" : "phoneNumber",
      "name" : "№ Телефона (10 цифр)",
      "value" : "9267101283"
    }, {
      "code" : "BIK",
      "name" : "БИК",
      "value" : "044583151"
    }, {
      "code" : "Name",
      "name" : "Имя Отправителя",
      "value" : "name"
    }, {
      "code" : "SName",
      "name" : "Отчество Отправителя",
      "value" : "sname"
    }, {
      "code" : "Fam",
      "name" : "Фамилия Отправителя",
      "value" : "fam"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  }
}
```

### 4.3 Получение истории платежей.

$ curl -u+79099560571:p@ssw0rD https://www.synq.ru/mserver2-dev/v1/payments

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : [ {
    "id" : 1401089234883,
    "client_payment_id" : "e731a7e2-c553-4295-867e-1023359bee28",
    "amount" : 100,
    "total" : 135,
    "created_at" : "2014-05-26T15:15:07.389Z",
    "processed_at" : "2014-05-26T15:37:11.589Z",
    "status" : "finished",
    "type" : "out",
    "service" : {
      "id" : 61,
      "name" : "Мультибанк"
    },
    "parameters" : [ {
      "code" : "Pname1",
      "name" : "Номер счета/карты/договора",
      "value" : "5136913818331704"
    }, {
      "code" : "Pname2",
      "name" : "ФИО владельца счета/карты/договора",
      "value" : "fio"
    }, {
      "code" : "phoneNumber",
      "name" : "№ Телефона (10 цифр)",
      "value" : "9267101283"
    }, {
      "code" : "BIK",
      "name" : "БИК",
      "value" : "044583151"
    }, {
      "code" : "Name",
      "name" : "Имя Отправителя",
      "value" : "name"
    }, {
      "code" : "SName",
      "name" : "Отчество Отправителя",
      "value" : "sname"
    }, {
      "code" : "Fam",
      "name" : "Фамилия Отправителя",
      "value" : "fam"
    } ],
    "outbound" : {
      "id" : 35,
      "code" : "tpr_out",
      "name" : "ООО ТПР (провайдер)"
    }
  } ]
}
```
