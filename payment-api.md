Платежное API synq.ru
=========

Выполнение запроса
---------
URL всех запросов API начинаются с префикса `https://www.synq.ru/protocol/xml/`. **Мы поддерживаем только HTTPS.**
Для того, чтобы выполнить команду протокола нужно склеить префикс с именем команды, например, для команды balance URL будет `https://www.synq.ru/protocol/xml/balance`.
Все команды платежного API используют метод POST и Content-Type `application/xml`.

Если вы используете утилиту curl, то команду balance можно вызвать следующим образом
```shell
curl -u 9267101280:secret \
    -X POST \
    -H 'Content-Type: application/xml' \
    -d @balance.xml \
    https://www.synq.ru/protocol/xml/balance
```

где файл balance.xml содержит
```xml
<?xml version="1.0" encoding="utf-8"?>
<request>
    <protocol-version>1.00</protocol-version>
</request>
```

Аутентификация
---------

Мы используем стандартную Basic аутентификацию протокола HTTP. Это безопасно, поскольку мы работаем по протоколу HTTPS.
В качестве имени пользователя можно использовать номер телефона, адрес электронной почты или номер счета кошелька.

Команды API
---------

### Запрос на привязку кошелька к партнеру
* `POST /protocol/xml/rebind` с телом

```xml
<?xml version="1.0" encoding="utf-8"?>
<request>
    <protocol-version>1.00</protocol-version>
    <partner>my_partner_code</partner>
</request>
```
приведет к привязке кошелька к партнеру с кодом `my_partner_code`.

### Запрос баланса счетов кошелька
* `POST /protocol/xml/balance` с телом

```xml
<?xml version="1.0" encoding="utf-8"?>
<request>
    <protocol-version>1.00</protocol-version>
</request>
```
вернет баланс счетов кошелька в копейках:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response>
    <extra name="27">100500</extra>
    <extra name="62">362</extra>
</response>
```

### Запрос списка провайдеров
* `POST /protocol/xml/provider` с телом

```xml
<?xml version="1.0" encoding="utf-8"?>
<request>
    <protocol-version>1.00</protocol-version>
</request>
```
вернет список провайдеров доступных к оплате с информацией о полях:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response>
    <provider id="529292889" max-sum="15000" min-sum="1" name="11x11 (21)">
        <provider-params>
            <params code="phoneNumber" title="Укажите Ник в игре"/>
        </provider-params>
    </provider>
    <provider id="730949264" max-sum="15000" min-sum="10" name="2pay (29)">
        <provider-params>
            <params code="phoneNumber" comments="Введите 2pay-номер" max-length="7" min-length="7" pattern="^\d{7}$" pattern-description="7 цифр" title="Введите 2pay-номер" type="numeric"/>
        </provider-params>
    </provider>
    <provider id="68819181" max-sum="15000" min-sum="1" name="Aika Online (21)">
        <provider-params>
            <params code="phoneNumber" pattern-description="Все символы" title="Укажите свой email"/>
        </provider-params>
    </provider>
    <provider id="61425299" max-sum="15000" min-sum="10" name="Цезарь Сателлит Северо-Запад (29)">
        <provider-params>
            <params code="phoneNumber" max-length="6" min-length="1" pattern="^\d{1,6}$" pattern-description="от 1 до 6 цифр" title="Укажите ПИН" type="numeric"/>
        </provider-params>
    </provider>
    <provider id="626035626" max-sum="15000" min-sum="2" name="Яндекс Деньги">
        <provider-params>
            <params code="phoneNumber" max-length="15" min-length="10" title="Номер счета" type="numeric"/>
        </provider-params>
    </provider>
</response>
```

### Проведение платежей
* `POST /protocol/xml/payment` с телом

```xml
<?xml version="1.0" encoding="utf-8"?>
<request>
    <protocol-version>1.00</protocol-version>
        <payment>
                <client-transaction-id>1982</client-transaction-id>
                <amount>100</amount>
                <service-id>453315258</service-id>
                <extra name="phoneNumber">9267101280</extra>
        </payment>
    <payment>
                <client-transaction-id>1983</client-transaction-id>
                <amount>100</amount>
                <service-id>453315258</service-id>
                <extra name="phoneNumber">9267101280</extra>
        </payment>
</request>
```
вернет ответ, содержащий состояние платежей и номер платежной транзакции на стороне synq.ru. Его в дальнейшем можно использовать для получения статуса платежа.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response>
    <payment client-transaction-id="1982" status="PAYMENT_EXT_STARTED" status-description="Начат процесс уведомления внешнего поставщика" transaction-number="443"/>
    <payment client-transaction-id="1983" status="PAYMENT_EXT_STARTED" status-description="Начат процесс уведомления внешнего поставщика" transaction-number="444"/>
</response>
```

Во вложенном в <payment> теге <extra> указываются параметры платежа, необходимые для выбранного пользователем сервис-провайдера.
Перечень параметров сервис-провайдера можно получить командой provider.
В client-transaction-id указывается идентификатор транзакции, генерируемый клиентской стороной.
Он должен уникально идентифицировать платеж для данного клиента.
Клиентский идентификатор транзакции должен быть целым  числом в диапазоне от -9223372036854775808 до 9223372036854775807.
В теге amount указывается сумма платежа (внесенная) в копейках.
Допускается указание нескольких элементов payment в элементе request для отправки нескольких платежей в одном запросе.

### Получение статуса платежа
* `POST /protocol/xml/status` с телом

```xml
<?xml version="1.0" encoding="utf-8"?>
<request>
    <protocol-version>1.00</protocol-version>
    <status>
        <payment transaction-number="443" />
        <payment transaction-number="444" />
    </status>
</request>
```
вернет состояние платежей.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response>
    <payment status="FINISHED" status-description="Финализирована" transaction-number="443"/>
    <payment status="FINISHED" status-description="Финализирована" transaction-number="444"/>
</response>
```

### Статусы платежа
|Статус               |Описание                                     |Финальный?|
|---------------------|---------------------------------------------|----------|
|PAYMENT_NOT_CONFIRMED|Платеж в очереди, запросите статус позднее.  |Нет.      |
|PAYMENT_EXT          |Платеж проводится, запросите статус позднее. |Нет.      |
|PAYMENT_EXT_STATUS   |Платеж проводится, запросите статус позднее. |Нет.      |
|PAYMENT_EXT_FINISHED |Платеж проводится, запросите статус позднее. |Нет.      |
|FINISHED             |Платеж успешно проведен.                     |Да.       |
|ABANDONED            |Платеж не проведен.                          |Да.       |

Обработка ошибок HTTP протокола
---------
В случае получения от synq.ru HTTP статусов 5xx, например `502 Bad Gateway`, `503 Service Unavailable`, или
`504 Gateway Timeout` повторите запрос позже.
