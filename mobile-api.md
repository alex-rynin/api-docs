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
 
```shell

curl -d '{"phone": "79267101280", "password": "123456"}' \
 -H 'Content-type:application/json' \
 https://www.synq.ru/mserver-dev/protocol/mobile/v1/register

```

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
```shell
curl -d '{"phone": "79267101280", "pin": "123456"}' \ 
-H 'Content-type:application/json' \ 
https://www.synq.ru/mserver-dev/protocol/mobile/v1/confirm
```

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
    "activated" : false
  }
}
```


### Проверка правильности учетных данных
```shell
curl -d '{"login":"79267101280", "password": "123456"}' \
-H  'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/auth
```

вернет статус `200` и данные пользователя 

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 181,
    "phone" : "79267101280",
    "email" : null,
    "active" : true
  }
}
```

в случае, если переданные учетные данные верны. В противном случает будет 
```json
{
  "meta" : {
    "status" : 400,
    "code" : 400,
    "developerMessage" : "Unable to find user by login 979267101280",
    "moreInfoUrl" : "https://www.synq.ru",
    "exception" : "UserNotFoundException"
  },
  "data" : {
    "login" : "979267101280"
  }
}
```

### Загрузка профиля пользователя

```shell
curl -u 79267101280:123456  https://www.synq.ru/mserver-dev/protocol/mobile/v1/profile
```

вернет профиль аутентифицированного пользователя.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 244,
    "phone" : "79267101280",
    "email" : "alexander@yanyshin.ru",
    "active" : true,
    "smsAuth" : false
  }
}
```

### Обновление настроек профиля пользователя

```shell
curl -u 79267101280:123456 \
-H 'Content-type:application/json' \
-d '{"smsAuth": true}'  \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/profile
```

обновит настройки профиля и вернет новые значения.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 244,
    "phone" : "79267101280",
    "email" : "alexander@yanyshin.ru",
    "active" : true,
    "smsAuth" : true
  }
}
```

### Загрузка счетов кошелька

```shell
curl -u 79267101280:123456 https://www.synq.ru/mserver-dev/protocol/mobile/v1/accounts
```

вернет идентификаторы, имена и балансы открытых счетов кошелька. Баланс счетов, как и везде, в рублях.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "main" : {
      "id" : 161,
      "name" : "Основной",
      "amount" : 0
    },
    "all" : [ {
      "id" : 161,
      "name" : "Основной",
      "amount" : 0
    }, {
      "id" : 162,
      "name" : "test",
      "amount" : 10
    }, {
      "id" : 163,
      "name" : "Дополнительный",
      "amount" : 0
    } ]
  }
}

```

### Создание счета в кошельке

```shell
curl -u 79267101280:123456 -X POST https://www.synq.ru/mserver-dev/protocol/mobile/v1/accounts
```

вернет идентификатор нового счета

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "accountId" : 162
  }
}

```

### Переименование счета

```shell
curl -u 79267101280:123456 -d '{"name": "test"}' \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/accounts/162
```

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 162,
    "name" : "test",
    "amount" : 0
  }
}
```

переименует счет с ID 162.

### Закрытие счета

```shell
curl -u 79267101280:123456 -X POST \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/accounts/closed/162
```

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 162,
    "name" : "test",
    "amount" : 0
  }
}
```

закроет счет 162.

### Загрузка справочника сервисов

```shell
curl -u 79267101280:123456 https://www.synq.ru/mserver-dev/protocol/mobile/v1/services
```

вернет справочник доступных сервисов с разбиением по группам

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : [ {
    "id" : 3,
    "name" : "Мобильная связь",
    "services" : [ {
      "id" : 1000,
      "name" : "Теле2"
    }, {
      "id" : 827,
      "name" : "МТС"
    }, {
      "id" : 770,
      "name" : "Билайн"
    }, {
      "id" : 834,
      "name" : "Мегафон"
    } ]
  }, {
    "id" : 12,
    "name" : "Игры",
    "services" : [ {
      "id" : 1066,
      "name" : "Одноклассники"
    }, {
      "id" : 1068,
      "name" : "Вконтакте"
    }, {
      "id" : 1065,
      "name" : "World of Tanks"
    } ]
  }, {
    "id" : 5,
    "name" : "Интернет",
    "services" : [ {
      "id" : 1081,
      "name" : "OnLime"
    } ]
  }, {
    "id" : 4,
    "name" : "Телевидение",
    "services" : [ {
      "id" : 1069,
      "name" : "Триколор ТВ"
    } ]
  }, {
    "id" : 6,
    "name" : "Телефония",
    "services" : [ {
      "id" : 777,
      "name" : "МГТС"
    } ]
  }, {
    "id" : 8,
    "name" : "Платежные системы",
    "services" : [ {
      "id" : 1021,
      "name" : "WebMoney"
    }, {
      "id" : 1002,
      "name" : "Яндекс.Деньги"
    } ]
  }, {
    "id" : 9,
    "name" : "Банковские услуги",
    "services" : [ {
      "id" : 1064,
      "name" : "Хоум кредит энд финанс банк"
    }, {
      "id" : 1063,
      "name" : "Банк Русский Стандарт"
    }, {
      "id" : 1102,
      "name" : "Пополнение VISA/MASTERCARD/MAESTRO Украина"
    }, {
      "id" : 1061,
      "name" : "Альфа Банк"
    }, {
      "id" : 1041,
      "name" : "Тинькофф Кредитные Системы"
    }, {
      "id" : 1062,
      "name" : "ОТП Банк"
    }, {
      "id" : 1101,
      "name" : "Пополнение VISA/MASTERCARD/MAESTRO Россия"
    } ]
  }, {
    "id" : 2,
    "name" : "Мобильная связь СНГ",
    "services" : [ {
      "id" : 1067,
      "name" : "МТС Украина"
    } ]
  } ]
}

```

### Загрузка описания сервиса

```shell
curl -u 79267101280:123456 https://www.synq.ru/mserver-dev/protocol/mobile/v1/services/1102
```

вернет описание провайдера, например:

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 1102,
    "name" : "Пополнение VISA/MASTERCARD/MAESTRO Украина,СНГ (29)",
    "minsum" : 100,
    "maxsum" : 15000,
    "parameters" : [ {
      "code" : "126",
      "minLength" : null,
      "maxLength" : null,
      "title" : "Адрес регистрации Плательщика",
      "pattern" : "^.{3,70}$",
      "type" : "text",
      "patternDescription" : "Адрес регистрации плательщика",
      "main" : false
    }, {
      "code" : "18",
      "minLength" : null,
      "maxLength" : null,
      "title" : "ФИО Плательщика",
      "pattern" : "^.{3,70}$",
      "type" : "text",
      "patternDescription" : "ФИО плательщика",
      "main" : false
    }, {
      "code" : "2",
      "minLength" : null,
      "maxLength" : null,
      "title" : "№ телефона Плательщика (10 цифр) ",
      "pattern" : "^\\d{10}$",
      "type" : "text",
      "patternDescription" : "№ телефона плательщика (10 цифр)",
      "main" : false
    }, {
      "code" : "4",
      "minLength" : null,
      "maxLength" : null,
      "title" : "Город регистрации Плательщика",
      "pattern" : "^.{2,70}$",
      "type" : "text",
      "patternDescription" : "Город регистрации плательщика",
      "main" : false
    }, {
      "code" : "phoneNumber",
      "minLength" : 16,
      "maxLength" : 19,
      "title" : "№ карты (16/18/19 цифр)",
      "pattern" : "^(\\d{16}|\\d{18}|\\d{19})$",
      "type" : "numeric",
      "patternDescription" : "№ карты (16/18/19 цифр)",
      "main" : true
    } ]
  }
}
```


### Отправка платежа

```shell
curl -u 79267101280:123456 \
-d '{"serviceId": "834", "account": 342, "amount": 1, "parameters": {"phoneNumber": "9267101280"}}' \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/pay
```


отправит платеж на 1 рубль на номер 9267101280 сервиса Мегафон со счета 162 и вернет ID платежа, а также флаг, указывающий на то, нужно ли подтверждать платеж.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "transactionId" : 1867,
    "confirmationRequired" : true
  }
}
```

Счет списания можно не указывать, тогда деньги будут списаны с главного счета кошелька.

### Подтверждение платежа кодом

Если в результате отправки платежа получили `"confirmationRequired" : true`, то нужно подтвердить платеж, передав код из SMS сообщения.

```shell
curl -u 79267101280:123456 \
-d '{"transactionId": 1867, "code": "925578"}' \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/confirmPayment
```

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "transactionId" : 1867
  }
}
```

после чего mserver сделает попытку провести платеж.

### Перевод между счетами кошельков

```shell
curl -u 79267101280:123456 \
-d '{"source": 162, "destination": "161", "amount": 1}' \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/transfer
```

отправит перевод на 1 рубль с 162 на 161 счет. Счет назначения может быть задан номером счета, номером телефона кошелька 
или email кошелька.
Результатом выполнения команды будет ID платежной транзакции.


```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "transactionId" : 1868
  }
}
```

### Однократное пополнение счета кошелька с пластиковой карты

```shell
curl -u 79267101280:123456 \
-H 'Content-type:application/json' \
-d '{"destination": "79267101280", "amount": 1.0}' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/cardPayment
```

вернет идентификатор платежной транзации на пополнение счета и URL платежной страницы, на которую следует перенаправить пользователя.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "transactionId" : 2146,
    "paymentPageUrl" : "https://test1.ipsp.com/frontend/endpoint?amount=1.0&cf=2146&currency=RUB&hash=139db86f33003dc2aeacc0ab29748587d310a55e&payment_type=S&perspayee_expiry=0150&product_id=1713"
  }
}
```

### Привязка пластиковой карты

```shell
curl -u 79267101280:123456 \
-X POST -H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/card
```

вернет идентификатор карты и URL платежной страницы, на которую следует перенаправить пользователя.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "cardId" : 1,
    "paymentPageUrl" : "https://test1.ipsp.com/frontend/endpoint?amount=1&biller_client_id=1&cf=2184&currency=RUB&hash=3f043e74873bf29d03bfb7d3f957fc589ea33297&payment_type=S&perspayee_expiry=0150&product_id=1713"
  }
}
```

NOTE: В результате успешной привязки с карты будет списан 1 рубль.


### Получение статуса привязанной карты

```shell
curl -u 79267101280:123456 https://www.synq.ru/mserver-dev/protocol/mobile/v1/card/1
```

вернет статус привязанной карты с идентификатором 1.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 1,
    "title" : "431422******9913",
    "state" : "ACTIVE",
    "cardType": "Visa"
  }
}
```

Статусы карты

| Статус                | Описание                                               |
| --------------------- |:------------------------------------------------------:|
| CREATED               | Ожидает успешного завершения первого recurring платежа | 
| ACTIVE                | Первый платеж успешно выполнен                         |
| FAILED                | Отказано в первом платеже                              |
| DELETED               | Карта удалена                                          |


### Получение списка привязанных карт

```shell
curl -u 79267101280:123456 https://www.synq.ru/mserver-dev/protocol/mobile/v1/card
```

вернет список привязанных карт

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : [ {
    "id" : 1,
    "title" : "431422******9913",
    "state" : "ACTIVE",
    "cardType": "Visa"
  } ]
}
```

### Удаление привязанной карты

```shell
curl -u 79267101280:123456 -X POST https://www.synq.ru/mserver-dev/protocol/mobile/v1/card/dropped/85
```

вернет только что удаленную карту

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 85,
    "title" : "541715******9149",
    "state" : "DELETED",
    "cardType" : "MasterCard"
  }
}
```

### Пополнение счета кошелька с привязанной карты

Запрос на пополнение основного счета кошелька 79267101280 с привзанной карты с ID 1 на 1 рубль.

```shell
curl -u 79267101280:123456 \
-H 'Content-type:application/json' \
-d '{"destination": "9267101280", "amount": 1}' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/cardPayment/1
```

вернет идентификатор платежной транзакции и статус операции IPSP = {OK|KO}.


```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "transactionId" : 2193,
    "resultCode" : "OK"
  }
}
```

### Оплата в пользу провайдера с привязанной карты (транзитом через основной счет кошелька)

Оплата на 1 рубль в пользу сервиса 834 по номеру 9267101280 с привязанной карты с ID 1.

```shell
curl -u 79267101280:123456 \
-d '{"serviceId": "834", "amount": 1, "parameters": {"phoneNumber": "9267101280"}}' \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/cardProviderPayment/1
```

Вернет ID транзакций пополнения и списания и статус карточной транзакции от IPSP = {OK|KO}.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "card" : {
      "transactionId" : 2195,
      "resultCode" : "OK"
    },
    "provider" : {
      "transactionId" : 2196
    }
  }
}
```

### Получение статуса платежной транзакции

```shell
curl -u 79267101280:123456 https://www.synq.ru/mserver-dev/protocol/mobile/v1/payment/2363
```

вернет информацию о транзакции

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "id" : 2363,
    "date" : 1393541057237,
    "amount" : 1,
    "sourceAccountId" : 342,
    "destinationAccountId" : 3,
    "transactionType" : "OUT",
    "params" : [ {
      "title" : "№ телефона (10 цифр)",
      "value" : "9267101280",
      "name" : "phoneNumber"
    } ],
    "service" : {
      "id" : 834,
      "name" : "Мегафон"
    },
    "status" : "FINISHED"
  }
}

```

Статусы платежной транзакции

| Статус                | Описание                               | Окончательный? |
| --------------------- |:--------------------------------------:| --------------:|
| FINISHED              | Проведен                               | Да             |
| CANCELED              | Отклонен                               | Да             |
| CANCELED_INVALID_DATA | Отклонен, неверные данные платежа      | Да             |
| <любой другой статус> | Проводится                             | Нет            |

### Поиск пользователей по телефонам/адресам эл. почты

```shell
curl -u 79267101280:123456 \
-d '{"logins": ["79267101280", "alexander@yanyshin.ru"]}' \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/mobile/v1/findUsers
```


вернет список найденных пользователей:

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : [ {
    "id" : 47,
    "phone" : "9267101280",
    "email" : "alexander@yanyshin.ru",
    "active" : true
  }, {
    "id" : 181,
    "phone" : "79267101280",
    "email" : null,
    "active" : true
  } ]
}

```

### Получение истории платежей

```shell
curl -u 79267101280:123456 'https://www.synq.ru/mserver-dev/protocol/mobile/v1/payment?limit=1&offset=1'
```

вернет историю платежных транзакций для кошелька 79267101280 отсортированную по дате создания платежа по убыванию. 
* limit - ограничение на количество строк истории (размер страницы)
* offset - количество строк, которое будет пропущено.

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "history" : [ {
    "id" : 1844,
    "createDate" : 1391183325868,
    "finishDate" : 1391183350293,
    "status" : "FINISHED",
    "amount" : 1,
    "commission" : 0,
    "serviceId" : 834,
    "sourceAccountId" : 25,
    "destinationAccountId" : 3,
    "type" : "OUT",
    "sourceId" : 47,
    "sourceName" : null,
    "sourceEmail" : "alexander@yanyshin.ru",
    "sourcePhone" : "79267101280",
    "sourceType" : "USER",
    "destinationId" : 1,
    "destinationName" : "Кредит Пилот",
    "destinationEmail" : null,
    "destinationPhone" : null,
    "destinationType" : "PROVIDER",
    "params" : [ {
      "title" : "Номер телефона",
      "value" : "9267101280",
      "name" : "phoneNumber"
    } ],
    "service" : {
      "code" : "843",
      "name" : "Мегафон"
    }
  } ]
  }
}
```

Также можно использовать навигацию в стиле Facebook Graph API, передавая limit и параметр before или after.
* after - ID последней транзакции в текущей выборке (передавать для следующей страницы).
* before - ID первой транзакции в текущей выборке (передавать для предыдущей страницы).

Если не указать значение параметра limit, будет использовано значение по умолчанию = 35.

API для доверенных систем
---------

Доступно только пользователям со включенной ролью ROLE_TRUSTED.

### Активация пользователя

```shell
curl -u trusted:ffuuuu \
-X POST  \
https://www.synq.ru/mserver-dev/protocol/trusted/user/active/310
```

вернет

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

и активирует пользователя с ID 310. Если пользователь уже активирован, команда не делает ничего и возвращает false.

### Смена пароля

```shell
curl -u trusted:ffuuuu \
-d '{"password": "qwe123"}' \
-H 'Content-type:application/json' \
https://www.synq.ru/mserver-dev/protocol/trusted/user/password/310
```

вернет

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : {
    "changed" : true
  }
}
```

и изменит пароль пользователя с ID 310 на qwe123.

API для разработчиков
---------

Доступно только на dev сервере при указании специального User-agent (кому положено тот знает).

### Получение списка всех пользователей
 
```shell
curl --user-agent <UA> https://www.synq.ru/mserver-dev/protocol/dev/user
```

вернет

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : [
   {
   "id" : 0,
   "phoneNumber" : "79267101280",
   "email" : null,
   "login" : null,
   "enabled" : true,
   "renewalCode" : null,
   "activationCode" : null,
   "active" : true,
   "dateCreate" : 1390486296155,
   "smsAuth" : false,
   "notifyOnTx" : false,
   "roles" : [ ],
   "userClass" : null,
   "new" : false,
   "roleNames" : [ ]
   }
  ]
}
```

### Получение пользователя по телефону или email

```shell
curl --user-agent <UA> https://www.synq.ru/mserver-dev/protocol/dev/user/79267101280
```

вернет

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" :
   {
   "id" : 0,
   "phoneNumber" : 79267101280,
   "email" : null,
   "login" : null,
   "enabled" : true,
   "renewalCode" : null,
   "activationCode" : null,
   "active" : true,
   "dateCreate" : 1390486296155,
   "smsAuth" : false,
   "notifyOnTx" : false,
   "roles" : [ ],
   "userClass" : null,
   "new" : false,
   "roleNames" : [ ]
   }
}
```

### Удаление пользователя по его телефону или email

```shell
curl --user-agent <UA> -X POST https://www.synq.ru/mserver-dev/protocol/dev/user/dropped/79267101280
```

удалит пользователя со всеми зависимыми объектами и вернет

```json
{
  "meta" : {
    "code" : "200"
  },
  "data" : null
}
```
