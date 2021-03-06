# Список API-методов модуля

### Содержание
* [Регистрация пользователя — `getCryptKeyWithUserReg`](#Регистрация-пользователя)
* [Авторизация пользователя — `cryptLogin`](#Авторизация)
* [Пополнение баланса пользователя — `doPayment`](#Пополнение-баланса-пользователя)
* [Получение данных пользователей партнёра — `getUserData`](#Получение-данных-пользователей-партнёра)
* [Получение списка непрочитанных сообщений пользователя — `getUserMessages`](#Получение-списка-сообщений-пользователя)
* [Получение списка непрочитанных сообщений всех пользователей партнёра — `getMessages`](#Получение-сообщений-всех-пользователей-партнёра)
* [Получение шаблонов сообщений — `getMessageTemplates`](#Получение-шаблонов-сообщений)
* [Отметка о прочтении списка сообщений — `readMessages`](#Отметка-о-прочтении-списка-сообщений)
* [Изменение URL-а проекта — `changeUrl`](#Изменение-urla-проекта)
* [Авторесайз фрейма](#Авторесайз-фрейма)
* [Deeplinks](#Deeplinks)

### Формирование пути `url` запросов к API
```
https://<HOST>/<PARTNER_PATH>/<METHOD>
```
где
> `HOST` — хост сервиса. Хост песочницы: `sandbox.promopult.org`;  
> `PARTNER_PATH` — уникальный путь к апи для каждого партнера.  
> `METHOD` — название апи-метода.
  
*Пример:*  

`https://sandbox.promopult.org/partners/acme`

<a name="Регистрация-пользователя"></a>
### Регистрация пользователя — `getCryptKeyWithUserReg`
#### Синтаксис запроса
```
GET https://<HOST>/<PARTNER_PATH>/getCryptKeyWithUserReg
 ?login=<string>
 &email=<string>
 &phone=<string>
 &hash=<string>
 &partner=<string>
 &suggestedDomain=<string>
```

Параметры:  
> `login` — уникальный логин в системе. Для поддержания уникальности желательно в него добавлять префикс, содержащий ид пользователя партнера. Например: `<PARTNER>_<DOMAIN>`

> `email` — email пользователя в системе.

> `phone` — мобильный телефон пользователя в интернациональном формате. Например: `+79551234567` (необязятельный параметр).  

> `hash` — уникальный 32-символьный случайный хеш пользователя. При этом его необходимо сохранить в БД для дальнейшего использования.

> `partner` — ваш уникальный 32-символьный идентификатор партнера.

> `suggestedDomain` — домен, который должен продвигаться в системе. Без протокола. Например: `site.ru`, `lenta.ru`, `cnn.com` (Необязательный параметр)

#### Формат ответа ####
`SUCCESS`

```json
{
  "status": {
    "code": 0,
    "message": "ok"
  },
  "error": false,
  "data": {
    "cryptKey": "<CRYPT_KEY>"
  }
}
```
где

> `CRYPT_KEY` — Ключ для шифрования-дешифрования данных пользователя. Необходимо сохранить в БД для дальнейшего использования. Формат: строка в 52 символа. Пример: `ko808duypw6hxos6q4vihukgy4wpuzwwbwnerus676krgwcoi7u`

`FAIL`

```json
{
  "status": {
    "code": <ERROR_CODE>,
    "message": "<ERROR_MESSAGE>"
  },
  "error": true
}
```

*Коды ошибок:*
> `0` — Нет ошибки  
> `1` — Ошибка валидации данных  
> `2` — Неверный ключ партнера  
> `3` — Данный ключ пользователя уже занят - сгенерируйте новый  
> `4` — Длина хеша должна быть 32 символа  

*Пример:*  

`https://sandbox.promopult.org/partners/acme/getCryptKeyWithUserReg?login=user&email=user@yandex.ru&phone=79551234567&hash=ed9d0bba9c9a56033b3b943742ef51aa&partner=c44e340a29f2e3b4e63412bf929d7fc8&suggestedDomain=example.com`

<a name="Авторизация"></a>
### Авторизация пользователя — `cryptLogin`
#### Синтаксис запроса
```
GET https://<HOST>/<PARTNER_PATH>/cryptLogin ?
  k=<PREFIX><USER_HASH><ENCRYPTED_DATA>&r=<PAGE>
```

где
> `PREFIX` — `'zaa'`;  
> `ENCRYPTED_DATA` — закодированные данные запроса.  
> `PAGE` — экран, который откроется после логина, см. [deeplinks](#Deeplinks) (необязательный параметр).

Для авторизации создадим адрес GET-запроса:

```php
//
$data = [
  'login' => '<LOGIN>',
  'hash'  => '<USER_HASH>',
  'createdOn' => date('Y-m-d h:i:s')
];

// генерируем URL
$k    = json_encode($data);
$code = SimpleCrypt::encrypt($k, '<CRYPT_KEY>');
$url  = 'https://sandbox.promopult.org/partners/acme/cryptLogin?k=zaa' . '<USER_HASH>' . urlencode($code) . '&r=<PAGE>';
```

В результате переменная `$url` будет содержать ссылку, которую можно подставлять в параметр `src` тега iframe.

*Пример:*  

`https://sandbox.promopult.org/partners/acme/cryptLogin?k=zaaf3102ac1d0588bea1db5411f662826636ZTQo%2BDP4JOoltfdzJ7T0tTXoanh0Wba5s%2BkZKaYa2uqk6OS1dfV05RvV9uimaZn2dOm0qKZbgHaqaef2Metn2OX15elpWymnm6bk5yVluPNlqve1cbej7CEnaJmaaKfoaNlqpCmn6yWZ7OYppPr`

<a name="Пополнение-баланса-пользователя"></a>
### Пополнение баланса пользователя — `doPayment`
#### Синтаксис запроса ####
```
GET https://<HOST>/<PARTNER_PATH>/doPayment ?
  k=<PREFIX><USER_HASH><ENCRYPTED_DATA>
```

Создадим GET-запрос для проводки платежа:

```php
$paymentData = array(
  'paymentSum'   => '<PAYMENT_COST>',
  'paymentHash'  => '<PAYMENT_HASH>',
  'userLogin'    => '<LOGIN>',
);

$k = json_encode($paymentData);
$code = SimpleCrypt::encrypt($k, '<CRYPT_KEY>');
$url = 'https://<HOST>/<PARTNER_PATH>/doPayment?k=zaa' .  '<USER_HASH>' . urlencode($code);

```
где
> `PAYMENT_COST` — Сумма платежа  
> `PAYMENT_HASH` — Хеш платежа. Уникальное случайное 32-символьное значение. Формат: md5-код.  
> `LOGIN` — Логин пользователя в системе.  

#### Формат ответа ####
`SUCCESS`
```json
{
  "status": {
    "code": 0,
    "message": "ok"
  },
  "error": false,
  "data": {
    "paymentId": "<PAYMENT_ID>",
    "paymentBonuses": <PAYMENT_BONUSES>,
    "paymentMonthsDiscount": <PAYMENT_MONTHS_DISCOUNT>
  }
}
```
где
> `PAYMENT_ID` — Ид платежа в системе. +Обязательно нужно его привязать (сохранить) к платежу в партнерской системе. Пример: `114163`.  
> `PAYMENT_BONUSES` — Актуально только при включенной системе скидок, иначе поле отсутствует! Сумма бонусов, полученная дополнительно к сумме оплаты.  
> `PAYMENT_MONTHS_DISCOUNT` — Актуально только при включенной системе скидок, иначе поле отсутствует!   

`FAIL`
```json
{
  "status" : {
    "code" : <ERROR_CODE>,
    "message" : "<ERROR_MESSAGE>"
  },
  "error" : true
}
```


<a name="Получение-данных-пользователей-партнёра"></a>
### Получение данных всех пользователей партнера — `getUserData`
При необходимости можно получить данные всех пользователей партнера: баланс, статус проекта и ключевых слов.
Для этого необходимо запросить ключ шифрования партнера `PARTNER_CRYPT_KEY`, т.к. метод не подразумевает использования ключа шифрования какого-либо пользователя системы.

#### Синтаксис запроса
```
GET https://<HOST>/<PARTNER_PATH>/getUserData ?
  k=<PREFIX><PARTNER_HASH><ENCRYPTED_DATA>
```
где
> `PARTNER_HASH` — Хеш партнера.

Создадим адрес GET-запроса:

```php
$queryData = array(
  'partner'   => '<PARTNER_HASH>',
  'createdOn' => date('Y-m-d h:i:s'),
);

$k = json_encode($queryData);
$code = SimpleCrypt::encrypt($k, '<PARTHER_CRYPT_KEY>');
$url = 'https://sandbox.promopult.org/partners/acme/getUsersData?k=zaa' . '<PARTNER_HASH>' . urlencode($code) . '&type=userInvite'
```

где
> `PARTHER_CRYPT_KEY` — Ключ который необходимо запросить. Уникален для партнера системы.

<a name="Получение-списка-сообщений-пользователя"></a>
## Получение списка непрочитанных сообщений пользователя — `getUserMessages`
#### Синтаксис запроса
```
GET https://<HOST>/<PARTNER_PATH>/getUserMessages ?
  k=<PREFIX><USER_HASH><ENCRYPTED_DATA>
```

Создадим урл GET-запроса для получения сообщений

```php
$data = array(
    'partner'  => <PARTNER_HASH>,
    'markRead' => <MARK_READ>,
);

$k = json_encode($data);
$code = SimpleCrypt::encrypt($k, '<CRYPT_KEY>');
$url = 'http://sandbox.promopult.org/partners/acme/getUserMessages?k=zaa' . '<USER_HASH>' . urlencode($code);
```
где
> `MARK_READ` — — Необязятельный параметр, если передан TRUE, сообщения буду как прочитанное.
 
#### Формат ответа
`SUCCESS`
```json
{
  "status": {
    "code": 0,
    "message": "ok"
  },
  "error": false,
  "data": {
    "messages": <MESSAGES>
  }
}

```
где
> `MESSAGES` — Массив сообщений в формате:
> ```json
> [
>   ...
>   {
>     "id": "<MESSAGE_ID>",
>     "userLogin": "<LOGIN>",
>     "type": "<TYPE>",
>     "title": "<TITLE>",
>     "text": "<TEXT>",
>     "params": "<PARAMS>",
>     "templateId": "<TEMPLATE_ID>",
>     "textTemplate": "<TEXT_TEMPLATE>",
>     "titleTemplate": "<TITLE_TEMPLATE>"
>   },
>   ...
> ]
> ```
> > где,  
> > `MESSAGE_ID` — Идентификатор сообщения в системе.   
> > `LOGIN` — Логин пользователя.   
> > `TYPE` — тип сообщения
> > `TITLE` — Заголовок (тема) сообщения.    
> > `TEXT` — Текст сообщения (**Важно!** в тексте могу содержаться [Шаблоны ссылок на внутренние экраны iframe](#Deeplinks-в-тексте-сообщений)).   
> > `PARAMS` — Массив динамических свойств сообщения (используется для подстановки в шаблон сообщения партнера).     
> > `TEMPLATE_ID` — Целочисленный уникальный идентификатор шаблона сообщения.  
> > `TEXT_TEMPLATE` — Шаблон текста сообщения в формате [mustache](http://mustache.github.io/mustache.5.html).  
> > `TITLE_TEMPLATE` — Шаблон заголовка (subject) сообщения в формате [mustache](http://mustache.github.io/mustache.5.html).  

`FAIL`
```json
{
  "status" : {
    "code" : <ERROR_CODE>,
    "message" : "<ERROR_MESSAGE>"
  },
  "error" : true
}
```  
<a name="Получение-сообщений-всех-пользователей-партнёра"></a>
## Получение списка непрочитанных сообщений всех пользователей партнёра — `getMessages`
#### Синтаксис запроса
```
GET https://<HOST>/<PARTNER_PATH>/getMessages ?
  k=<PREFIX><PARTNER_HASH><ENCRYPTED_DATA>
```
Создадим урл GET-запроса для получения сообщений

```php
$data = array(
    'partner'  => <PARTNER_HASH>,
    'markRead' => <MARK_READ>,
);

$k = json_encode($data);
$code = SimpleCrypt::encrypt($k, '<CRYPT_KEY>');
$url = 'http://sandbox.promopult.org/parnters/acme/getMessages?k=zaa' . '<PARTNER_HASH>' . urlencode($code);
```
где
> `MARK_READ` — Необязятельный параметр, если передан TRUE, сообщения буду как прочитанное.
 
#### Формат ответа
`SUCCESS`
```json
{
  "status": {
    "code": 0,
    "message": "ok"
  },
  "error": false,
  "data": {
    "messages": <MESSAGES>
  }
}

```
где
> `MESSAGES` — Массив сообщений в формате:
> ```json
> [
>   ...
>   {
>     "id": "<MESSAGE_ID>",
>     "userLogin": "<LOGIN>",
>     "type": "<TYPE>",
>     "title": "<TITLE>",
>     "text": "<TEXT>",
>     "params": "<PARAMS>",
>     "templateId": "<TEMPLATE_ID>",
>     "textTemplate": "<TEXT_TEMPLATE>",
>     "titleTemplate": "<TITLE_TEMPLATE>"
>   },
>   ...
> ]
> ```
> > где,  
> > `MESSAGE_ID` — Идентификатор сообщения в системе.   
> > `LOGIN` — Логин пользователя.   
> > `TYPE` — тип сообщения
> > `TITLE` — Заголовок (тема) сообщения.    
> > `TEXT` — Текст сообщения (**Важно!** в тексте могу содержаться [Шаблоны ссылок на внутренние экраны iframe](#Deeplinks-в-тексте-сообщений)).   
> > `PARAMS` — Массив динамических свойств сообщения (используется для подстановки в шаблон сообщения партнера).    
> > `TEMPLATE_ID` — Целочисленный уникальный идентификатор шаблона сообщения.  
> > `TEXT_TEMPLATE` — Шаблон текста сообщения в формате [mustache](http://mustache.github.io/mustache.5.html).  
> > `TITLE_TEMPLATE` — Шаблон заголовка (subject) сообщения в формате [mustache](http://mustache.github.io/mustache.5.html).  
`FAIL`
```json
{
  "status" : {
    "code" : <ERROR_CODE>,
    "message" : "<ERROR_MESSAGE>"
  },
  "error" : true
}
```  
<a name="Получение-шаблонов-сообщений"></a>
## Получение шаблонов сообщений — `getMessageTemplates`
#### Синтаксис запроса
```
GET https://<HOST>/<PARTNER_PATH>/getMessageTemplates ?
  k=<PREFIX><PARTNER_HASH><ENCRYPTED_DATA>
```
Создадим урл GET-запроса для получения сообщений

```php
$data = array(
    'partner'  => <PARTNER_HASH>
);

$k = json_encode($data);
$code = SimpleCrypt::encrypt($k, '<CRYPT_KEY>');
$url = 'http://sandbox.promopult.org/parnters/acme/getMessageTemplates?k=zaa' . '<PARTNER_HASH>' . urlencode($code);
```
 
#### Формат ответа
`SUCCESS`
```json
{
  "status": {
    "code": 0,
    "message": "ok"
  },
  "error": false,
  "data": {
    "messages": <MESSAGE_TEMPLATES>
  }
}

```
где
> `MESSAGE_TEMPLATES` — Массив шаблонов сообщений в формате:
> ```json
> [
>   ...
>   {
>     "templateId": "<TEMPLATE_ID>",
>     "textTemplate": "<TEXT_TEMPLATE>",
>     "titleTemplate": "<TITLE_TEMPLATE>"
>   },
>   ...
> ]
> ```
> > где,  
> > `TEMPLATE_ID` — Целочисленный уникальный идентификатор шаблона сообщения.  
> > `TEXT_TEMPLATE` — Шаблон текста сообщения в формате [mustache](http://mustache.github.io/mustache.5.html).  
> > `TITLE_TEMPLATE` — Шаблон заголовка (subject) сообщения в формате [mustache](http://mustache.github.io/mustache.5.html).      

`FAIL`
```json
{
  "status" : {
    "code" : <ERROR_CODE>,
    "message" : "<ERROR_MESSAGE>"
  },
  "error" : true
}
```

<a name="Отметка-о-прочтении-списка-сообщений"></a>
## Отметка о прочтении списка сообщений — `readMessages`
#### Синтаксис запроса
```
POST https://<HOST>/<PARTNER_PATH>/readMessages ?
  k=<PREFIX><PARTNER_HASH><ENCRYPTED_DATA>
```
`POST BODY`
```
ids=<IDS_COMMA_SEPARATED>
```
где
> `IDS_COMMA_SEPARATED` — строка чисел-идентификаторов сообщений, разделенных запятыми

#### Формат ответа
`SUCCESS`
```json
{
  "status": {
    "code": 0,
    "message": "ok"
  },
  "error": false,
  "data": {
    "success": <SUCCESS_COUNT>
  }
}
```
где
> `SUCCESS_COUNT` — количество успешно обработанных сообщений

<a name="Изменение-urla-проекта"></a>
##  Изменение URL-а проекта — `changeUrl`
#### Синтаксис запроса
```
GET https://<HOST>/<PARTNER_PATH>/changeUrl ?
  k=<PREFIX><USER_HASH><ENCRYPTED_DATA>
```
Создадим урл GET-запроса для получения сообщений

```php
$data = array(
    'url' => <NEW_URL>
);

$k = json_encode($data);
$code = SimpleCrypt::encrypt($k, '<CRYPT_KEY>');
$url = 'http://sandbox.promopult.org/partners/acme/changeUrl?k=zaa' . '<USER_HASH>' . urlencode($code);
```
где
> `NEW_URL` — Новый URL проекта, например `http://my-new-domain.com`.
 
#### Формат ответа
`SUCCESS`
```json
{
  "status": {
    "code": 0,
    "message": "ok"
  },
  "error": false,
  "data": {
    "newUrl": <NEW_URL>,
    "oldUrl": <OLD_URL>
  }
}

```
где
> `NEW_URL` — Новый URL  
> `OLD_URL` — URL проекта до вызова метода    

`FAIL`
```json
{
  "status" : {
    "code" : <ERROR_CODE>,
    "message" : "<ERROR_MESSAGE>"
  },
  "error" : true
}
```

<a name="Авторесайз-фрейма"></a>
## Авторесайз фрейма

Для установки автоматической высоты встройки внутри страницы партнёра по высоте её внутреннего содержимого, необходимо на странице подключения iframe добавить загрузку скрипта ресайзера.
```html
<script type="text/javascript"
        src="https://sandbox.iframe.promopult.org/integration/resizer.js"></script>
```

Далее на событие `onload` `html`-элемента iframe добавить вызов: `iFrameResize({ checkOrigin: false }, this)`

```html
<iframe src="https://sandbox.promopult.org
        scrolling="no"
        style="min-height: 600px"
        onload="iFrameResize({ checkOrigin: false }, this)"></iframe>
```

Также, можно ознакомиться с [репозиторием] (https://github.com/davidjbradshaw/iframe-resizer) либы на github на предмет других возможностей.


<a name="Deeplinks"></a>
## Deeplinks
Для организации _deeplinks_ в методе `cryptLogin` служит GET-параметр `r`, который указывает на какой экран перейти после логина в iframe-е. Например, если необходимо открыть страницу пополнения для `r` нужно передать значение `go/payment` в URL-кодированном виде.

_Пример:_  

`https://sandbox.promopult.org/partners/acme/cryptLogin?k=...&r=go%2Fpayment`

Для параметра `r` доступны следующие значения:
```
/go/payment
/go/support
/go/help

```

<a name="Deeplinks-в-тексте-сообщений"></a>
#### Deeplinks в тексте сообщений
В тексте сообщений может содержаться **шаблон ссылки** на один из внутренних экранов iframe-а, например, в сообщении с типом `low_balance` есть ссылка на страницу пополнения счета.

Часть сообщения, содержащая ссылку, выглядит так:
```
...
Чтобы пополнить баланс перейдите по <a target="_blank" href="{{go/payment}}">ссылке</a>. 
```
Атрибут `href` содержит шаблон адреса экрана пополнения счета в iframe `{{go/payment}}`, который, в зависимости от контекста вывода сообщения, необходимо заменить на *deeplink* через интерфейс Партнера в модуль продвижения. 

После преобразования, для пользователя ссылка в сообщении может выглядеть так: 
```
...
Чтобы пополнить баланс перейдите по <a target="_blank" href="http://my-cool-service.com/promotion?open=go/payment">ссылке</a>.
```

где `http://my-cool-service.com/promotion?open=go/payment` адрес страницы модуля на сайте партнера, а GET-параметр `open` пробросится в атрибут `src` тега iframe:
```
...
<iframe src='https://sandbox.promopult.org/iframe/cryptLogin?k=...&r=go/payment'></iframe>
...
``` 