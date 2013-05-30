API мобильных устройств synq.ru
=========

Выполнение запроса
---------
URL всех запросов API начинаются с префикса `https://www.synq.ru/protocol/mobile/`. **Мы поддерживаем только HTTPS.**
Для того, чтобы выполнить команду протокола нужно склеить префикс с именем команды, например, для команды providers URL будет `https://www.synq.ru/protocol/mobile/providers`.
Если вы используете утилиту curl, то команду providers можно вызвать следующим образом
```shell
curl -u user:pass https://www.synq.ru/protocol/mobile/providers
```

Аутентификация
---------

Мы используем стандартную Basic аутентификацию протокола HTTP. Это безопасно, поскольку мы работаем по протоколу HTTPS.

Команды API
---------

### Проверка правильности учетных данных
* `POST /protocol/mobile/auth`
с телом

```json
{"applicationId": "my cool app"}
```
вернет статус 200 в случае, если переданные учетные данные верны. В противном случает будет 401.

### Регистрация пользователя
* `POST /protocol/mobile/register

```json
{"phone": "9267101280", "email": "someone@example.com", "password": "p@$$w0rd"}
```
вернет
```json
{"userId": 13}
```
если пользователь успешно создан.
В случае, если пользователь с таким email или phone уже существует, в ответе будет

```json
{"exist": ["email", "phoneNumber"]}
```
### Подтвержение телефона пользователя
* `POST /protocol/mobile/confirm
с телом

```json
{"phone": "9267101280", "pin": "123456"}
```
подтвердит номер телефона пользователя и вернет статус активации.

```json
{"activated": true}
```
