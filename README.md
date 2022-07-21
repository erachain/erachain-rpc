# Erachain-RPC

Remote Protocol Control

Инвойсы (заказы). Форматы сообщений для банка

### Предпосылки
1. Все наименования полей передаваемые в JSON структурах чувствительны к регистру
2. Взаимодействие с нодой осуществляется путем отправки соответствующих запросов через PRC порты. Для серверов тестовой сети - 9068/tcp рабочей сети  - 9048/tcp
Пример команды получения адреса в кошельке ноды:
http://89.235.184.251:9068/addresses?password=123456789

3. Для работы методов отправки телеграмм и транзакций необходимо использовать пароль более 8-ми символов:

curl -d "123456789" -X POST http://127.0.0.1:9068/wallet/unlock

4. Для обозначения передаваемые параметров используются фигурные скобки, например следующий синтаксис
parameter={parameterValue}
для parameterValue=643 будет исполнен как 
parameter=643
1. Получение списка заказов (инвойсов)
Для поиска заказов на стороне банка, нужно выполнить RPC запрос с указанием счета (address) на который посланы сообщения, даты (timestamp) с которой нужно прочитать сообщения и (опционально) идентификатора пользователя (filter).
Параметр decrypt=true дает команду расшифровывать сразу зашифрованные сообщения (требует указание параметра password).
Команда RPC
telegrams/address/{address}/timestamp/{timestamp}?filter={filter}&decrypt={true/false}&password=123456789 

#### Примеры
Поиск телеграммы с даты 0 и по фильтру идентификатора пользователя 9090001011:  
http://89.235.184.251:9068/telegrams/timestamp/0?filter=9090001011  

Поиск телеграммы по адресу получателя 7Dpv5... и фильтру идентификатора пользователя 9090001011:  
http://89.235.184.251:9068/telegrams/address/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob?filter=9090001011

Поиск телеграммы по сигнатуре 5CmLD...  
http://89.235.184.251:9068/telegrams/get/5CmLDgMuk8MLU2nf2BYxu93bBFmxzE5cFrqqe1avw5m8WdYbPjVAv1HRR5HziZVdpSGCNUhnLaUARe2Qoiixc4XB

Поиск телеграммы по адресу 79MXsj... с даты 0 по фильтру идентификатора пользователя 909000111:  
http://89.235.184.251:9068/telegrams/address/79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH/timestamp/0?filter=9090001011

Формат Ответа  
Ответ выдается в виде списка объектов JSON, где каждый элемент списка есть JSON объект c единственным полем transaction и соответствующим ему вложенным объектом JSON имеющим следующим набор полей:  


Таблица 1. Представление полей транзакции  

| Наименование поля | Формат | Описание	| Пример |
| ----------------- | ------ | -------- | ------ |
| timestamp	| number | Дата создания сообщения в ms(Unix epoch time) соответствует времени GMT: Friday, 27 July 2018 г., 13:15:41.321  | 1532697187321 |
| title (optional) | string, UTF8 | Заголовок сообщения | |
| creator| 	string, Base58	| Счет создателя сообщения	 | |
| publickey | string, Base58 | Публичный ключ создателя сообщения | |
| signature | string, Base58 | Цифровая подпись сообщения - далее уникальный идентификатор заказа |  |
| message (optional) | string, UTF8 | Содержит данные об заказе |  |
| isText | boolean | true - сообщение в текстовом формате |  |	
| encrypted | boolean | true - сообщение зашифровано |  |

Поле message содержит Сообщение с набором полей, описывающих заказ и представляет собой инкапсулированный JSON объект (см. пример ниже):

Таблица 2. Представление полей Сообщения 

| Наименование поля | Формат | Описание	| Пример |
| ----------------- | ------ | -------- | ------ |
| date | number | Дата создания заказа  соответствует времени GMT: Friday, 27 July 2018 г., 13:15:41.321 | 1532697187321 |
| order | string | ID заказа | DF-12/q |
| user | string | ID покупателя - номер телефона и/или e-mail и/или счет в блокчейн. Если указывается несколько идентификаторов, то они разделяются пробелом 7DshTw6... | 916241345 |
| curr | number | Код валюты к оплате (ISO) | 643 |
| sum (optional) | number | Сумма к оплате. Если параметр не задан, то можно делать любую оплату, например пополнение депозита в магазине | 10354.34 |
| expire (optional) | number | Срок действия заказа в минутах от даты создания заказа (параметр date). Если не задан то устанавливается в 60 минут. | 60 |
| title (optional) | string, UTF8 | Заголовок сообщения. В нем указывается краткая информация, которую банк будет посылать клиенту в качестве пуш уведомления например.	"оплата покупки в Озон" |  |
| description (optional) | string, UTF8 | Описание заказа, включая указание об сумме НДС (или без НДС). Это поле будет подтавляться банком в "Назначение платежа" | "Без НДС" |
| details (optional) | string, UTF8 | Список платежных реквизитов магазина (будет использоваться в дальнейшем) | "КБК 190198198981б БИК 1298371398 СЧЕТ 10293809183" |
| callback (optional) | string | Callback url - ссылка на обратный отклик магазина | https://my_shop/simplechain/callback?order= |

**Примеры ответов на запросы**  
Запрос от iDБанк (поиск телеграммы по адресу получателя, времени и идентификатору пользователя):  
http://127.0.0.1:9068/telegrams/address/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob/timestamp/1526981437793?filter={номер телефона}

Ответ:  
```
{
"transaction":
  {            "type_name": "SEND", 
      "creator": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob", 
      "message": "{\"date\":1526981249588,\"order\":\"HSA-1234\",\"user\":\”79101230110\”,\"curr\":643,\"sum\":100.5,\"expire\":300,\"title\":\"Покупка в Магазине\",\"details\":\”BIK:40800123000120800034 IBAN:40800123000120800033\”,\"description\":\"Без НДС"}",            "signature": "5mHi6cFKbN36U3FaPydTE2GwXhD6ZGqNvJRcBCT455MpK6TLr2Wkyf1mV51DLzu93RGZikJowZV9fPauAB6ko8Y6",            "title": "9101230110",
      "encrypted": false, 
      "recipient": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",
      "isText": true,
      "timestamp": 1526981249588,
    ... }
},
```

```
{ 
    "transaction": {
            "type_name": "SEND", 
            "creator": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob", 
            "message": "{\"date\":1526981337793,\"order\":\"order123\",\"user\":\”127\”,\"curr\":2,\"sum\":10.1,\"expire\":153,\"title\":\"Заказ в магазине Ламода\",\"paymentDetails\":\"--\" ,\"description\":\"НДС не включает\"}",            "signature": "2dn6iiRuvNDeeLT61rXT7nw56ZThNwH5ndGbRmjT6UYJWa4wT7iXHDFWnp8dSSrNmBBK13b2dRv2VRjJQmGwrTBt",            "title": "127",            "encrypted": false, 
            "recipient": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",
            "isText": true,
            "timestamp": 1526981337793,
            ...        }
    }
```

### 2. Отправка сообщения в магазин
Формат уведомления об оплате  
Сообщение оператора производится путем создания транзакции в блокчейне, в поле message которой указана вся необходимая информация в формате JSON. Данные могут быть зашифрованы

Таблица 3. Представление Уведомления об оплате инвойса

Наименование	Формат	Описание
orderSignature
	string	Подпись оплаченного (ранее посланного) инвойса в кодировке Base58, 64 байта
curr	number	Код валюты к оплате (ISO). Например для публя 643, для доллара 840
sum(optional)	number	Если оплаченная сумма не совпадает с суммой инвойса, то оплата частичная (возможно доплата придет с другого банка или будет произведена наличными курьеру при доставке или покрыта бонусными баллами). 
Если сумма инвойса не была задана — то данная сумма указывает на сколько было произведено пополнение дебета в магазине. Сумма заказа может не задаваться если магазин принимает неограниченное количество денег на личный счет пользователя без конкретизации каждой покупки. 
callback (optional)	string	Callback url - ссылка на обратный отклик банка или оператора, например для быстрой связи в следующий раз
 

Банк передает сообщение в магазин, посылая транзакцию в блокчейн с использованием метода RPC r_send.
Форматы команд передачи сообщения с записью в блокчейн:

**Метод GET**  
r_send/{creator}/{recipient}?title={title}&message={message}&encoding={encoding}&encrypt=true&password={password}

creator - cчет-отправитель,  отправляет сообщение или средства, посылая транзакцию в  блокчейн, с этого счета списывается комиссия за осуществление транзакции. Счет  должен существовать в кошельке  ноды,  на которой исполняется команда.
recipient - cчет-получатель, получает сообщение или средства.
title - заголовок сообщения в кодировке UTF8, длина при преобразовании в массив байт,  не должна превышать 256 байт.
encoding - база кодировки сообщения (number), по умолчанию = 0, используется  кодировка UTF8.  Допустимые значения: 0, 16, 58, 64. Например, значение 58  указывает на кодировку Base58.
message - сообщение в кодировке UTF8 или в кодировке, задаваемой параметром  encoding.
encrypt - приказ зашифровать сообщение, по умолчанию - false.
password - пароль доступа к кошельку.
Замечания к команде r_send
Если счет создателя транзакции не удостоверен и задан заголовок (title) и сообщение (message) не зашифровано, то такая транзакция будет признана некорректной и отвергнута нодой. Для того чтобы этого не происходило, необходимо удостоверить счет с которого создается транзакция, либо зашифровать сообщение и не задавать заголовок сообщения. Допустимо задать сообщение с кодировкой в цифровом виде, используя параметр encoding. Пересылка сообщения в цифровом виде уменьшает размер транзакции и снижает ее стоимость. Если происходит пересылка расчетных активов (имеющих стоимость для третьих лиц) с удостоверенного счета на анонимный, например с номером актива более 1000, то транзакция также будет отвергнута.

Пример
http://89.235.184.251:9068/r_send/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob/79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH?message={"orderSignature":"4Jyw8hqDHr4H7RuKhhjJkKZRhTZ8D2w1ggKTuEPTmmJyKbRf81HDYhWkdmjseQsgnXCBmBarz16FeyLA5c1M13tn","sum":"1598.0000"}&password=123456789


Метод POSTВ теле запроса необходимо сформировать объект JSON указав в нем все необходимые поля:
```
{
"creator":"7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",  "recipient":"79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH",
“title”:”...”,
“message”:”...”,
"encoding ":0,
"encrypt":false,
"password":"123456789"
}
```


В поле message необходимо передать Уведомление об оплате для магазина в виде объекта JSON, см. таблицу 3

Пример:
"message":"{\"orderSignature\":\"3oBXNhN5ihNMFSYgWVKQPYXoqjJNFmsvWdMCTEFeBBRuu9bcVBHrtvmcLMX1BZMJHUiTJYupEecvuvDckotQ8QLS\",\"sum\":10000.55,\"curr\":643}",

Пример команды с методом POST:
```
{"creator":"7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob","recipient":"79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH",
“message”:”...”,
"encoding ":0,
"encrypt":false,
"password":"123456789"
}
```



Примечание: экранирование двойных кавычек обратной наклонной чертой внутри строки поля message необходимо в случае, если в используемом языке программирования строка обозначается двойными кавычками (Java, C, C++ и т.п.)

Замечание по проверке внесения транзакций в блокчейн.
Так как полной уверенности что транзакция будет внесена в блокчейн нет (именно поэтому у транзакций есть разное состояние - неподтвержденная и подтвержденная), то после того как транзакция послана в блокчейн, необходимо ее сохранить в локальной таблице. Затем организовать проверку транзакций по этой таблице - внесена ли транзакция в блокчейн И если внесена, то удаляем ее из таблицы. Иначе шлем еще раз. Период просмотра транзакций должен быть равен, по крайней мере, двум длительностям блока (например 600 секунд).
Проверка подтверждения (внесения) транзакции в блокчейн:
GET transactions/signature/[signature]
где signature - подпись транзакции в кодировке Base58.
В ответ будет выдана информация от транзакции и если она включена в блокчейн, то в поле confirmations будет выдано значение подтверждений - если оно больше 0, то транзакция включена в блокчейн

Другие команды для транзакций

Проверка неподтвержденных транзакцийtransactions/network

Например:
http://89.235.184.251:9068/transactions/network

Замечания по использованию параметра callback (03.сентября)
После подтверждения оплаты заказа и передачи сообщения (банком) в блокчейн о совершенной оплате, банк может использовать данный параметр (callback) для уведомления магазина об оплате и для перенаправления пользователя на его сайт. При этом параметром в команде должна быть задана подпись от транзакции, которую банк послал в блокчейн с сообщением об оплате.

Callback как вариант уведомления магазина
Если обратиться по URL заданному в параметре callback с добавлением подписи от транзакции в которой банк послал сообщение об оплате, то на стороне магазина будет запущена проверка на поиск этой транзакции в блокчейн и обработку сообщения из нее. В этом случае банку достаточно получить код ответа 200 - что магазин был уведомлен.

Пример callback ссылок:  
https://my_shop/simplechain/callback/  
https://my_shop/simplechain/callback?signature=  

Банк должен добавить подпись своей транзакции в которой, и вызвать эту ссылку, например так:  
https://my_shop/simplechain/callback/R56aSdwto9i123dhg….  
https://my_shop/simplechain/callback?signature=R56aSdwto9i123dhg….  

Callback как вариант перенаправления клиента на сайт магазина  
Как правило, после произведения оплаты и уведомления магазина об этом, на стороне банка в личном кабинете плательщика высвечивают кнопку "Перейти обратно в магазин" - для того чтобы пользователя перенаправить на сайт магазина. В этом случае необходимо к ссылке на кнопке добавить параметр source=user, например так:  
https://my_shop/simplechain/callback/R56aSdwto9i123dhg….?source=user  
https://my_shop/simplechain/callback?signature=R56aSdwto9i123dhg….&source=user  

Магазин, получив запрос по такому URL, запускает проверку на изменения статуса заказа и перенаправляет клиента  на страницу этого заказа, где отражает состояние проверки, например "Проверка оплаты”, "Оплачен", "Отказан" и пр.  
### 3. Удаление телеграмм с ноды
3.1 Удаление самостоятельное по приходу весточки от другого банка  
Банк может самостоятельно мониторить приход весточек на заданные счета магазинов в которых указаны данные об оплате. И если подпись заказа совпадает со списком заказов на стороне банка, то банк может поставить в этот заказ галочку Оплачено или сумму Оплаты. Таким образом при заходе клиента в кабинет, он будет видеть оплаченные счета даже если они оплачены им в другом банке - что так же очень удобно. Или будет видеть "Частично Оплачен" и Остаток сколько еще нужно оплатить данный счет.  
Так же если распознавать счета банков то можно даже видеть список в каких банках какие суммы оплачены.  
Однако нужно проверять доверенные счета банков - только с них принимать сообщения об оплате. Список доверенных счетов может поставлять поставивших услуги "Безопасный платеж" или его можно получить потом самостоятельно по списку балансов счетов для заданного токена с правами доверия.  

3.2 Принудительное удаление с помощью команды  
Команда:  
telegrams/delete  

Команда может быть выполнена только с помощью метода POST.   
В качестве параметра задается объект JSON c массивом подписей удаляемых телеграмм.  

Пример команды удаления методом POST:  
curl -d "{\"list\": [\"5HUqfaaY2uFgdmDM7XNky31rkdcUCPTzhHXeanBviSvyDfhgYnH4a64Aje3L53Jxmyb3CcouRiBeUF4HZNc7yySy\"]}" -X POST http://127.0.0.1:9068/telegrams/delete  

В ответе приходит JSON массив не удаленных телеграмм, например, были удалены другой нодой.  
Пример ответа:  
["5HUqfaaY2uFgdmDM7XNky31rkdcUCPTzhHXeanBviSvyDfhgYnH4a64Aje3L53Jxmyb3CcouRiBeUF4HZNc7yySy",...]  

Удаление до заданного времени  
GET deleteTelegramToTimestamp/{timestamp}?address=[address]&filter=[filter]  
timestamp - по время, address - счет получателя, filter - выборка по заголовку  

Удаление для заданного получателя  
GET deleteTelegramForRecipient/{recipent}?timestamp=[timestamp]&filter=[filter]  
timestamp - по время, address - счет получателя, filter - выборка по заголовку  

## Приложения
### Приложение А. Полный список полей описания телеграммы (получение данных о телеграмме)

```
            "type_name": "SEND",
            "creator": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",
            "message": "{\"date\":1526981249588,\"order\":\"\",\"user\":\"126\",\"curr\":2,\"sum\":1,\"expire\":9588,\"title\":\"-\",\"paymentDetails\":null,\"description\":\"-\"}", 
            "signature": "5mHi6cFKbN36U3FaPydTE2GwXhD6ZGqNvJRcBCT455MpK6TLr2Wkyf1mV51DLzu93RGZikJowZV9fPauAB6ko8Y6", 
            "fee": "0.00019584",
            "type": 31, 
            "confirmations": 0,
            "version": 0,
            "record_type": "SEND",
            "title": "126",
            "encrypted": false,
            "recipient": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",
            "isText": true,
            "timestamp": 1526981249588,
            "height": -1
```

Полный формат команды передачи сообщения с записью в блокчейн:  
r_send/{creator}/{recipient}?feePow={feePow}&assetKey={assetKey}&amount={amount}&title={title}&message={message}&encoding={encoding}&encrypt=true&password={password}  

creator -  счет-отправитель,  отправляет сообщение или средства,  с этого счета     списывается комиссия за осуществление транзакции;  
recipient - cчет-получатель, получает сообщение или переводимые отправителем  средства;  
feePow - уровень комиссии (number), допустимые значения 0 ... 6, по умолчанию 0;  
assetKey  - номер учетной единицы в блокчейн (number), по умолчанию 2 (COMPU). Для  использования бухгалтерских учетных единиц, не являющихся расчетными между  участниками Эрачейн, используются ISO коды валют, например для рублей РФ -  643, для доллара США - 840.   
При передаче значения кода валюты внутри сообщения транзакции (message)  параметр не используется.  
amount  - количество передаваемых средств (number), по умолчанию 0.  
При передаче значения внутри сообщения транзакции (message) параметр не  используется.  
title - заголовок сообщения в кодировке UTF8, длина при преобразовании в массив байт,  не должна превышать 256 байт.  
encoding  - база кодировки сообщения (number), по умолчанию = 0, используется  кодировка UTF8.  Допустимые значения: 0, 16, 58, 64. Например, значение 58  указывает на кодировку Base58.  
message - сообщение в кодировке UTF8 или в кодировке, задаваемой параметром  encoding.  
encrypt - признак зашифрованного сообщения, по умолчанию - false.  
password - пароль доступа к кошельку.  

## Приложение Б. Создание весточек через RPC
Команда:  
telegrams/send   

Пример команды создания весточки методом POST:  
curl -d {\"sender\":\"7CzxxwH7u9aQtx5iNHskLQjyJvybyKg8rF\",\"recipient\":\"7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob\",\"title\":\"title\",\"message\":\"message\",\"encrypt\":true,\"password\":\"123456789\"} -X POST http://127.0.0.1:9068/telegrams/send   

В ответе приходит JSON с подписью весточки  
{""signature"": "2WpN9J7b4VqZhGR8X3mZ4BSMEjR2MqUWB1iKQ4s8txdPwaL3nCWUW5wAPANKiwU9yRsndBxbuG9fuY6sy4kXzK5q"}  
В качестве значения параметра message указать сообщение от магазина в соответствии с выше описанным форматом. В качестве sender - свой счет с которого шлете сообщениие, recipient - счет получателя, причем этот счет должен быть "известен для блокчейн" - тоесть с него должны были пройти какие-то транзакции или он должен быть в вашем кошельке - тогда система сможет найти соотвествующий ему публичный ключ и зашифровать его. Иначе выдаст ошибку - неизвестный публичный ключ  

## Приложение В. Расшифровка сообщения через телеграммы
Если в телеграмм сообщение зашифровано то расшифровать его можно командой datadecrypt  
datadecrypt/{signature}?password={password}  
Формат Ответа  
Строка текста если сообщение текстовое или кодировка Base58 если сообщение байтовое  
Пример метода GET  
http://127.0.0.1:9068/telegrams/datadecrypt/GerrwwEJ9Ja8gZnzLrx8zdU53b7jhQjeUfVKoUAp1StCDSFP9wuyyqYSkoUhXNa8ysoTdUuFHvwiCbwarKhhBg5?password=123456789  
Так же все запросы на поиск телеграмм можно делать с параметром decrypt=true и указывать при это пароль доступа к кошельку, например так:  
http://127.0.0.1:9068/telegrams/address/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob/timestamp/1526981437793?filter=791517654423&decrypt=true&passwor  d=123456789

## Multi Send
Multi send scrip for send asset for many addresses or persons filtered by some parameters.
This command will run as test for calculate FEE and total AMOUNT by default. For run real send set parameter `test=false`.  
r_send/multisend/{fromAddress}/{assetKey}/{forAssetKey}?position=1&amount=0&test=true&feePow=0&activeafter=[date]&activebefore=[date]&greatequal=[amount]&koeff=1&title=&onlyperson=false&selfpay=false&password=

In console type:
`GET r_send/multisend/...`
In web-browser type:
`http://127.0.0.1:9048/r_send/multisend/...`

**Parameters:**  


     * @param fromAddress     my address in Wallet
     * @param assetKey        asset Key that send
     * @param forAssetKey     asset key of holders test
     * @param amount          absolute amount to send
     * @param onlyperson      Default: false. Use only person accounts
     * @param gender          Filter by gender. -1 = all, 0 - man, 1 - woman. Default: -1.
     * @param position        test balance position. 1 - Own, 2 - Credit, 3 - Hold, 4 - Spend, 5 - Other
     * @param greatequal      test balance is great or equal
     * @param selfpay         if set - pay to self address too. Default = true
     * @param test            default - true. test=false - real send
     * @param feePow
     * @param activeafterStr  timestamp after that is filter - yyyy-MM-dd hh:mm or timestamp(sec)
     * @param activebeforeStr timestamp before that is filter - yyyy-MM-dd hh:mm or timestamp(sec) activetypetx
     * @param activetypetx    if set - test only that type transactions
     * @param koeff           koefficient for amount in balance position of forAssetKey
     * @param title
     * @param password


Example:

```
GET r_send/multisend/7LSN788zgesVYwvMhaUbaJ11oRGjWYagNA/1036/2?amount=0.001&title=probe-multi&onlyperson=true&activeafter=1577712486&password=123
GET r_send/multisend/7LSN788zgesVYwvMhaUbaJ11oRGjWYagNA/1069/1036?amount=0.001&title=probe-multi&onlyperson=true&activeafter=2018-01-01 00:00&activebefore=2019-01-01 00:00&greatequal=0&activetypetx=24&password=1
GET r_send/multisend/7A94JWgdnNPZtbmbphhpMQdseHpKCxbrZ1/1/2?amount=0.001&title=probe-multi&onlyperson=true&gender=0&password=1
GET r_send/multisend/7LSN788zgesVYwvMhaUbaJ11oRGjWYagNA/1/2?amount=100&title=probe-multi&onlyperson=true&activeafter=2019-09-11 00:00&greatequal=0&password=1
```

**Transaction types:**

    // ISSUE ITEMS
    ISSUE_ASSET_TRANSACTION = 21;
    ISSUE_IMPRINT_TRANSACTION = 22;
    ISSUE_TEMPLATE_TRANSACTION = 23;
    ISSUE_PERSON_TRANSACTION = 24;
    ISSUE_STATUS_TRANSACTION = 25;
    ISSUE_UNION_TRANSACTION = 26;
    ISSUE_STATEMENT_TRANSACTION = 27;
    ISSUE_POLL_TRANSACTION = 28;
    // SEND ASSET
    SEND_ASSET_TRANSACTION = 31;
    // OTHER
    SIGN_NOTE_TRANSACTION = 35;
    CERTIFY_PUB_KEYS_TRANSACTION = 36;
    SET_STATUS_TO_ITEM_TRANSACTION = 37;
    SET_UNION_TO_ITEM_TRANSACTION = 38;
    SET_UNION_STATUS_TO_ITEM_TRANSACTION = 39;
    // confirm other transactions
    VOUCH_TRANSACTION = 40;
    // HASHES
    HASHES_RECORD = 41;
    // exchange of assets
    CREATE_ORDER_TRANSACTION = 50;
    CANCEL_ORDER_TRANSACTION = 51;
    // voting
    CREATE_POLL_TRANSACTION = 61;
    VOTE_ON_POLL_TRANSACTION = 62;
    VOTE_ON_ITEM_POLL_TRANSACTION = 63;
    RELEASE_PACK = 70;

    CALCULATED_TRANSACTION = 100;



