# ERACHAIN-RPC

Remote Protocol Control

Invoices (orders). Message formats for bank

### Prerequisites
1. All field names passed in JSON structures are case sensitive
2. Interaction with node is performed by sending appropriate requests through PRC ports. For test network servers - 9068/tcp for work network - 9048/tcp
An example of a command to get the address in the wallet of a node:
http://89.235.184.251:9048/addresses?password=123456789

3. For telegram and transaction methods to work, you must use a password of more than 8 characters:

curl -d "123456789" -X POST http://127.0.0.1:9048/wallet/unlock

4. Curly braces are used to indicate passed parameters, e.g. the following syntax
parameter={parameterValue}
for parameterValue=643 will be executed as 
parameter=643
1. Obtaining a list of orders (invoices)
To search for orders at the bank you need to execute an RPC request specifying the account (address) where the messages were sent to, the date (timestamp) from which you want to read the messages and (optionally) the user's identifier (filter).
Parameter decrypt=true gives a command to decrypt encrypted messages at once (requires the password parameter).
RPC command
telegrams/address/{address}/timestamp/{timestamp}?filter={filter}&decrypt={true/false}&password=123456789 

#### Examples
Search for telegrams from date 0 and by filtering user ID 9090001011:  
http://89.235.184.251:9048/telegrams/timestamp/0?filter=9090001011  

Search for a telegram by recipient address 7Dpv5... and user ID filter 909000101011:  
http://89.235.184.251:9048/telegrams/address/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob?filter=9090001011

Telegram Search by Signature 5CmLD...  
http://89.235.184.251:9048/telegrams/get/5CmLDgMuk8MLU2nf2BYxu93bBFmxzE5cFrqqe1avw5m8WdYbPjVAv1HRR5HziZVdpSGCNUhnLaUARe2Qoiixc4XB

Search for a telegram by address 79MXsj... from date 0 by user ID filter 909000111:  
http://89.235.184.251:9048/telegrams/address/79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH/timestamp/0?filter=9090001011

Answer format.  
The response is returned as a list of JSON objects, where each item is a JSON object with a single field transaction and the corresponding JSON object with the following set of fields:  


Table 1. Presentation of the transaction fields  

| Field name | Format | Description | Example |
| ----------------- | ------ | -------- | ------ |
| timestamp | number | message creation date in ms(Unix epoch time) corresponds to GMT time: Friday, 27 July 2018, 13:15:41.321 | 1532697187321 |
| title (optional) | string, UTF8 | message title |
| creator| string, Base58 | message creator account |
| publickey | string, Base58 | public key of the message creator |
| signature | string, Base58 | message digital signature - further unique order identifier | |
| message (optional) | string, UTF8 | contains data about the order |
| isText | boolean | true - message in text format |	
| encrypted | boolean | true - the message is encrypted |

The message field contains a message with a set of fields describing the order and is an encapsulated JSON object (see the example below):

Table 2. Message field representation 

| Field name | Format | Description | Example |
| ----------------- | ------ | -------- | ------ |
| date | number Date of order creation corresponds to GMT time: Friday, 27 July 2018, 13:15:41.321 | 1532697187321 |
| order | string | order ID | DF-12/q |
| user | string | buyer ID - phone number and/or e-mail and/or blockchain account. If several identifiers are specified, they are separated by a space 7DshTw6... | 916241345 |
| curr | number | currency code payable (ISO) | 643 |
| sum (optional) | number | amount payable. If the parameter is not set, you can make any payment, for example a deposit to the store | 10354.34 |
| The period of validity of the order in minutes from the date of creation of the order (parameter date). If not specified it is set in 60 minutes. | 60 |
| title (optional) | string, UTF8 | message title. It contains the brief information that the bank will send to the client as a push notification e.g.	"payment for the purchase in Ozone" | |
| description (optional) | string, UTF8 | description of the order including indication of the VAT amount (or without VAT). This field will be substituted by the bank in "Purpose of payment" | "Without VAT" |
| Payment details (optional) | string, UTF8 | list of payment details of the store (will be further used) | "KBC 190198198981b BIK 1298371398 ACCOUNT 10293809183" |
| callback (optional) | string | callback url - link to the store's callback | https://my_shop/simplechain/callback?order= |

**Examples of query responses**  
Query from iDBank (search for a telegram by recipient address, time and user ID):  
http://127.0.0.1:9048/telegrams/address/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob/timestamp/1526981437793?filter={phone number}

Response:  
```
{
{ "transaction":
  { "type_name": "SEND", 
      { "creator": "7Dpv5Gi8HjCBgtDN1PniuPJQCBQ5H8Zob", 
      "message": "{\"date\":1526981249588,\"order\":\"HSA-1234\",\"user\":\”79101230110\”,\"curr\":643,\"sum\":100. 5,\"expire\":300,\"title\":\"Order at store\",\"details\":\”BIK:40800123000120800034 IBAN:40800123000120800033\”,\"description\": \{"no VAT"}", "signature": "5mHi6cFKbN36U3FaPydTE2GwXhD6ZGqNvJRcBCT455MpK6Tr2Wkyf1mV51DLzu93RGZikJowZV9fPauAB6ko8Y6", "title": "9101230110",
      "encrypted": false, 
      { "recipient": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",
      "isText": true,
      "timestamp": 1526981249588,
    ... }
},
```

```
{ 
    }, { "transaction": {
            "type_name": "SEND", 
            { "creator": "7Dpv5Gi8HjCBgtDN1PniuPJQCBQ5H8Zob", 
            "message": "{\"date\":1526981337793,\"order\":\"order123\",\"user\":\”127\”,\"curr\":2,\"sum\":10.1,\"expire\":153,\"title\":\"Order at Lamoda\",\"paymentDetails\":\"--\" ,{"description\":\"VAT not included\"}", "signature": "2dn6iiRuvNDeeLT61rXT7nw56ZThNwH5ndGbRmjT6UYJWa4wT7iXHDFWnp8dSSrNmBBK13b2dRv2VRjJQmGwrTBt", "title": "127", "encrypted": false, 
            { "recipient": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",
            "isText": true,
            "timestamp": 1526981337793,
            ...        }
    }
```

### 2. Sending a message to the store
Payment notification format  
The operator's message is produced by creating a transaction in the blockchain with all the necessary information in JSON format in the message field. Data can be encrypted

Table 3: Invoice payment notification representation

Name Format Description
orderSignature
	string Signature of the paid (earlier sent) invoice in Base58 encoding, 64 bytes
curr number Code of the currency to be paid (ISO). E.g. for GBP 643, for USD 840
Sum(optional) number If the amount paid is not the same as the invoice amount then the payment is partial (the surcharge may come from another bank or it may be paid in cash to the courier upon delivery or covered by bonus points). 
If the invoice amount hasn't been set, the amount indicates how much of the debit has been credited to the store. The order amount may not be specified if the store accepts an unlimited amount of money to the user's personal account without specifying each purchase. 
callback (optional) string Callback url - a link to the bank's or operator's callback, e.g. for the fast connection next time
 

The bank sends a message to the store by sending a transaction to the blockchain using the RPC r_send method.
Formats for message transfer commands with a write to the blockchain:

**GET method**.  
r_send/{creator}/{recipient}?title={title}&message={message}&encoding={encoding}&encrypt=true&password={password}

creator - creator account, sends a message or funds by sending a transaction to the blockchain, this account is debited with the transaction fee. The account must exist in the wallet of the node on which the command is executed.
recipient - the recipient account, receives the message or funds.
title - message header in UTF8 encoding, length should not exceed 256 bytes when converting into an array of bytes.
encoding - message encoding base (number), default = 0, UTF8 encoding is used.  Valid values: 0, 16, 58, 64. For example value 58 indicates Base58 encoding.
message - message in UTF8 encoding or in encoding defined by encoding parameter.
encrypt - encrypt message, by default - false.
password - password to access the wallet.
Notes to r_send command
If the account of transaction's creator is not authenticated and the title and message are not encrypted, then such transaction will be recognized as incorrect and rejected by the node. To avoid this, it is necessary to verify the account from which the transaction is created, or encrypt the message and not to set the message title. It is acceptable to set the message encoded digitally using the encoding parameter. Sending a message digitally reduces the size of the transaction and lowers its cost. If there is a transfer of settlement assets (having value to third parties) from a certified account to an anonymous account, for example with an asset number greater than 1000, the transaction will also be rejected.

Example
http://89.235.184.251:9048/r_send/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob/79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH?message={"orderSignature":"4Jyw8hqDHr4H7RuKhjJkKZRhTZ8D2w1ggKTuEPTmmJyKbRf81HDYhWkdmjseQsgnXCBmBarz16FeyLA5c1M13tn","sum":"1598.0000"}&password=123456789

POST methodIn the body of the request you need to form a JSON object specifying all the necessary fields in it:
```
{
"creator":"7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob","recipient":"79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH",
"title":",
"message":",
"encoding":0,
"encrypt":false,
"password":"123456789"
}
```


In the message field you must pass Notification of payment for the store as a JSON object, see table 3

Example:
"message":"{\"orderSignature\":\"3oBXNhNh5ihNMFSYYgWVKQPYXoqjJNFmsvWdMCTEFeBBRu9bcVBHrtvmcLMX1BZMJHUiTJYupEecvuvDckotQ8QLS\",\"sum\":10000.55,\"curr\":643}"

Example command with POST method:
```
{"creator": "7Dpv5Gi8HjCBgtDN1PniuPJQCBQ5H8Zob", "recipient": "79MXsjo9DEaxzu6kSvJUauLhmQrB4WogsH",
"message":",
"encoding":0,
"encrypt":false,
"password":"123456789"
}
```



Note: escaping double quotes with a backslash inside the message field string is necessary if your programming language uses double quotes (Java, C, C++, etc.).

Note about checking if transactions are entered into the blockchain.
As it's not absolutely sure that transaction will be put in the blockchain (that's why transactions have different states - unconfirmed and confirmed), then after transaction is sent in the blockchain, it must be saved in a local table. Then organize transaction check against this table - whether transaction is entered into the blockchain And if it is entered, then delete it from the table. Otherwise send it again. Period of review of transactions must be equal to at least two duration of the block (eg 600 seconds).
Check if the transaction is validated (added) to the blockchain:
GET transactions/signature/[signature]
where signature is the signature of the transaction in Base58 encoding.
The response will return information from the transaction and if it is included in the blockchain, then the confirmations field will show the confirmation value - if it is greater than 0, then the transaction is included in the blockchain

Other commands for transactions

Verify unconfirmed transactions

For example:
http://89.235.184.251:9048/transactions/network

Notes on using the callback parameter
After confirming the payment for the order and sending a message (by the bank) to the blockchain about the payment made, the bank may use this parameter (callback) to notify the store about the payment and to redirect the user to its site. In this case, the parameter in the command must be a signature from the transaction that the bank sent to the blockchain with the payment message.

Callback as store notification option
If you set a URL specified in the callback parameter and add a signature from a transaction in which the bank has sent the payment message, the store will run a check to find that transaction in the blockchain and process the message from it. In this case the bank only needs to get a response code 200 - that the store has been notified.

Example callback links:  
https://my_shop/simplechain/callback/  
https://my_shop/simplechain/callback?signature=  

The bank has to add a signature of its transaction in which, and call this link, for example like this:  
https://my_shop/simplechain/callback/R56aSdwto9i123dhg....  
https://my_shop/simplechain/callback?signature=R56aSdwto9i123dhg....  

Callback as an option to redirect the client to the site of the store  
As a rule, after the payment is made and the store is notified about it, the "Go back to the store" button is displayed in the payer's personal account to redirect the user to the store's site. In this case you need to add the parameter source=user to the link on the button, for example like this:  
https://my_shop/simplechain/callback/R56aSdwto9i123dhg....?source=user  
https://my_shop/simplechain/callback?signature=R56aSdwto9i123dhg....&source=user  

Upon receiving a request at this URL, the store will check for changes in the order status and redirect the client to a page of this order where the status of check is reflected; e.g., "Checking payment", "Paid", "Cancelled", etc.  
### 3. Deletion of telegrams from a node
3.1 Deletion by itself on arrival of the message from another bank  
Bank can independently monitor coming of newsletters to specified accounts of stores where payment data are specified. And if the signature of the order coincides with a list of orders on the bank side, the bank may put a tick in that order Paid or the amount Paid. Thus, when the client enters the cabinet, he will see the bills paid even if they are paid by him in another bank - which is also very convenient. Or will see "Partially Paid" and the balance of how much more need to pay this bill.  
Also if you recognize bank accounts you can even see a list of which banks what amounts are paid.  
However, it is necessary to check the trusted bank accounts - only from them to receive messages about the payment. The list of trusted accounts can be supplied by the service provider "Secure payment" or you can get it by yourself from the list of account balances for a given token with trust rights.  

3.2 Forced deletion with a command  
Command:  
telegrams/delete  

Command can be executed only by POST method.   
It takes a JSON object with an array of telegram signatures to be deleted as a parameter.  

Here is an example of POST command:  
curl -d "{\"list\": [\"5HUqfaaY2uFgdmDM7XNky31rkdcUCPTzhHXeanBviSvyDfhgYnH4a64Aje3L53Jxmyb3CouRiBeUF4HZNc7yySy\"]}" -X POST http://127.0.0.1:9048/telegrams/delete  

In the response comes JSON an array of telegrams not deleted, e.g. were deleted by another node.  
Example response:  
["5HUqfaaY2uFgdmDM7XNky31rkdcUCPTzhHXeanBviSviSvyDfhgYnH4a64Aje3L53Jxmyb3CcouRiBeUF4HZNc7yySy",...]  

Deleting before a given time  
GET deleteTelegramToTimestamp/{timestamp}?address=[address]&filter=[filter]  
timestamp - time, address - recipient account, filter - header sampling  

Deletion for a given recipient  
GET deleteTelegramForRecipient/{recipient}?timestamp=[timestamp]&filter=[filter]  
timestamp - time, address - recipient's account, filter - header sampling  

### Appendices
### Appendix A. Full list of telegram description fields (getting telegram data)

```
            "type_name": "SEND",
            "creator": "7Dpv5Gi8HjCBgtDN1PniuPJQCBQ5H8Zob",
            "message": "{\"date\":1526981249588,\"order\":\"\",\"user\":\"126\",\"curr\":2,\"sum\":1,\"expire\":9588,\"title\":\"-\",\"paymentDetails\":null,\"description\":\"-\"}", 
            "signature": "5mHi6cFKbN36U3FaPydTE2GwXhD6ZGqNvJRcBCT455MpK6Tr2Wkyf1mV51DLzu93RGZikJowZV9fPauAB6ko8Y6" 
            "fee": "0.00019584",
            "type": 31, 
            "confirmations": 0,
            "version": 0,
            "record_type": "SEND",
            { "title": "126",
            "encrypted": false,
            { "recipient": "7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob",
            "isText": true,
            "timestamp": "1526981249588",
            "height": -1
```

Full format of the message transmission command with blockchain entry:  
r_send/{creator}/{recipient}?feePow={feePow}&assetKey={assetKey}&amount={amount}&title={title}&message={message}&encoding={encoding}&encrypt=true&password={password}  

creator - the sending account, sends the message or funds, this account is debited with the transaction fee;  
recipient - the receiving account, that receives the message or funds sent by the sender;  
feePow - commission level (number), allowed values 0 ... 6, by default 0;  
assetKey - blockchain accounting unit number (number), the default value is 2 (COMPU). To use accounting units that are not settlement units between Erachain participants, ISO currency codes are used, e.g. for Russian rubles - 643, for US dollars - 840.   
When transmitting a currency code value inside a transaction message, the parameter is not used.  
amount - amount of transferred funds (number), by default 0.  
Parameter is not used when transmitting value inside transaction message.  
title - message header in UTF8 encoding, length should not exceed 256 bytes when converting into byte array.  
encoding - message encoding base (number), default value is 0, UTF8 encoding is used.  Valid values: 0, 16, 58, 64. For example value 58 indicates Base58 encoding.  
message - message in UTF8 encoding or in encoding, set by encoding parameter.  
encrypt - encrypted message sign, by default - false.  
password - wallet access password.  

## Appendix B. Creating messages via RPC
Command:  
telegrams/send   

An example of a command to create newsletters via POST:  
curl -d {\"sender\":\"7CzxxwH7u9aQtx5iNHskLQjyJvybyKg8rF\",\"recipient\":\"7Dpv5Gi8HjCBgtDN1P1niuPJCBQ5H8Zob\",\"title\" \"title\",\"message\":\"message\",\"encrypt\":true,\"password\":\"123456789\"} -X POST http://127. 0.0.1:9048/telegrams/send   

The response comes with JSON with the message signature  
{"signature": "2WpN9J7b4VqZhGR8X3mZ4BSMEjR2MqUWB1iKQ4s8txdPwaL3nCWUW5wAPANKiwU9yRsndBxbuG9fuY6sy4kXzK5q"}  
As the value of the message parameter, specify the message from the store according to the above described format. As sender - your account from which you send message, recipient - recipient account, and this account must be "known to blockchain" - i.e. it must have passed some transactions or be in your wallet - then the system can find the corresponding public key and encrypt it. Otherwise it will give error - unknown public key  

## Appendix B. Decrypting a message via telegrams
If you have encrypted the message in your telegram, you can decrypt it with the command datadecrypt  
datadecrypt/{signature}?password={password}  
Answer format  
Text string if the message is text or Base58 encoding if the message is byte  
An example of the GET method  
http://127.0.0.1:9088/telegrams/datadecrypt/GerrwwEJ9Ja8gZnzLrx8zdU53b7jhQjeUfVKoUAp1StCDSFP9wuyyqYSkoUhXNa8ysoTdUuFHvwiCbwarKhhBg5?password=123456789  
You can also make all telegram search requests with parameter decrypt=true and specify in it password to access the wallet, like this:  
http://127.0.0.1:9088/telegrams/address/7Dpv5Gi8HjCBgtDN1P1niuPJQCBQ5H8Zob/timestamp/1526981437793?filter=791517654423&decrypt=true&password=123456789

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
