# REST API V3

**TEST** 

* `https://stgapi.opnx.com`

**LIVE** site

* `https://api.opnx.com`

For clients who do not wish to take advantage of Opnx's native WebSocket API, Opnx offers a RESTful API that implements much of the same functionality.

## Error Code

Code | Description |
---- | ----------- |
429 | Rate limit reached |
10001 | General networking failure |
20001 | Invalid parameter |
30001 | Missing parameter |
40001 | Alert from the server |
50001 | Unknown server error |
20031 | The marketCode is closed for trading temporarily |

## Rate Limits

Each IP is limited to:

* 100 requests per second
* 20 POST v3/orders requests per second
* 2500 requests over 5 minutes

Certain endpoints have extra IP restrictions:

* `s` denotes a second
* Requests limited to `1/s` & `2/10s` & `4/10s`
  * Only 1 request is permitted per second and only 2 requests are permitted within 10 seconds
* Request limit `1/10s`
  * The endpoint will block for 10 seconds after an incorrect 2FA code is provided (if the endpoint requires a 2FA code)

Affected APIs:

* [POST /v3/withdrawal](?json#post-v3-withdrawal)
* [POST /v3/transfer](?json#post-v3-transfer)

## Rest Api Authentication

> **Request**

```json
{
    "Content-Type": "application/json",
    "AccessKey": "<string>",
    "Timestamp": "<string>",
    "Signature": "<string>",
    "Nonce": "<string>"
}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json


# rest_url = 'https://api.opnx.com'
# rest_path = 'v2api.opnx.com'

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = "API-KEY"
api_secret = "API-SECRET"

ts = datetime.datetime.utcnow().isoformat()
nonce = 123
method = "API-METHOD"

# Optional and can be omitted depending on the REST method being called 
body = json.dumps({'key1': 'value1', 'key2': 'value2'})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
# When calling an endpoint that uses body
# resp = requests.post(rest_url + method, data=body, headers=header)
print(resp.json())
```

Public market data methods do not require authentication, however private methods require a *Signature* to be sent in the header of the request.  These private REST methods  use HMAC SHA256 signatures. 

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a client's API Secret as the key and a message string as the value for the HMAC operation.

The message string is constructed as follows:-

`msgString = f'{Timestamp}\n{Nonce}\n{Verb}\n{URL}\n{Path}\n{Body}'`

Component | Required | Example | Description| 
-------------------------- |--------- |------- |------- | 
Timestamp | Yes | 2020-04-30T15:20:30 | YYYY-MM-DDThh:mm:ss
Nonce | Yes | 123 | User generated
Verb | Yes| GET | Uppercase
Path | Yes | v2stgapi.opnx.com |
Method | Yes | /v3/positions | Available REST methods
Body | No | marketCode=BTC-USDT-SWAP-LIN | Optional and dependent on the REST method being called

The constructed message string should look like:-

  `2020-04-30T15:20:30\n
  123\n
  GET\n
  v2stgapi.opnx.com\n
  /v3/positions\n
  marketCode=BTC-USDT-SWAP-LIN`

Note the newline characters after each component in the message string. 
If *Body* is omitted it's treated as an empty string.

Finally, you must use the HMAC SHA256 operation to get the hash value using the API Secret as the key, and the constructed message string as the value for the HMAC operation. Then encode this hash value with BASE-64.  This output becomes the signature for the specified authenticated REST API method. 

The signature must then be included in the header of the REST API call like so:

`header = {'Content-Type': 'application/json', 'AccessKey': API-KEY, 'Timestamp': TIME-STAMP, 'Signature': SIGNATURE, 'Nonce': NONCE}`

## Account & Wallet - Private


### GET `/v3/account `

Get account information

<aside class="notice">
Calling this endpoint using an API key pair linked to the parent account with the parameter "subAcc" allows the caller to include additional sub-accounts in the response. This feature does not work when using API key pairs linked to a sub-account.
</aside>

> **Request**

```
GET v3/account?subAcc={subAcc},{subAcc}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "accountId": "21213",
            "name": "main",
            "accountType": "LINEAR",
            "balances": [
                {
                    "asset": "BTC",
                    "total": "2.823",
                    "available": "2.823",
                    "reserved": "0",
                    "lastUpdatedAt": "1593627415234"
                },
                {
                    "asset": "FLEX",
                    "total": "1585.890",
                    "available": "325.890",
                    "reserved": "1260.21",
                    "lastUpdatedAt": "1593627415123"
                }
            ],
            "positions": [
                {
                    "marketCode": "FLEX-USDT-SWAP-LIN", 
                    "baseAsset": "FLEX", 
                    "counterAsset": "USDT", 
                    "position": "11411.1", 
                    "entryPrice": "3.590", 
                    "markPrice": "6.360", 
                    "positionPnl": "31608.7470", 
                    "estLiquidationPrice": "2.59", 
                    "lastUpdatedAt": "1637876701404",
                    "marginBalance": "45264.03",
                    "maintenanceMargin": "10886.1894",
                    "marginRatio": "0.24",
                    "leverage": "3"
	            }
            ],
            "collateral": "1231231",
            "notionalPositionSize": "50000.0",
            "portfolioVarMargin": "500",
            "riskRatio": "20000.0000",
            "maintenanceMargin": "1231",
            "marginRatio": "12.3179",
            "liquidating": false,
            "feeTier": "6",
            "createdAt": "1611665624601"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- |----- | -------- | ----------- |
subAcc | STRING | NO | Name of sub account. If no subAcc is given, then the response contains only the account linked to the API-Key. Multiple subAccs can be separated with a comma, maximum of 10 subAccs, e.g. `subone,subtwo` |

Response Field | Type | Description |
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
name | STRING | Account name |
accountType | STRING | Account type |
balances | LIST of dictionaries | |
asset | STRING | Asset name |
total | STRING | Total balance|
available | STRING | Available balance |
reserved | STRING | Reserved balance |
lastUpdatedAt | STRING | Last balance update timestamp |
positions | LIST of dictionaries | Positions if applicable|
marketCode | STRING | Market code |
baseAsset | STRING | Base asset |
counterAsset | STRING | Counter asset |
position | STRING | Position size |
entryPrice | STRING | Entry price |
markPrice | STRING | Mark price |
positionPnl | STRING | Position PNL |
estLiquidationPrice | STRING | Estimated liquidation price |
lastUpdatedAt | STRING | Last position update timestamp |
marginBalance | STRING |Appears in the position section only for positions using isolated margin. Isolated margin + Unrealized position PnL|
maintenanceMargin | STRING |Appears in the position section only for positions using isolated margin|
marginRatio | STRING | Appears in the position section only for positions using isolated margin|
leverage | STRING | Appears in the position section only for positions using isolated margin|
notionalPositionSize | STRING | Notional position size |
portfolioVarMargin | STRING | Portfolio margin |
riskRatio | STRING | collateralBalance / portfolioVarMargin. Orders are rejected/cancelled if the risk ratio drops below 1, and liquidation occurs if the risk ratio drops below 0.5 |
maintenanceMargin | STRING | Maintenance margin. The minimum amount of collateral required to avoid liquidation |
marginRatio | STRING | Margin ratio. Orders are rejected/cancelled if the margin ratio reaches 50, and liquidation occurs if the margin ratio reaches 100  |
liquidating | BOOL | Available values: `true` and `false` |
feeTier | STRING | Fee tier |
createdAt | STRING | Timestamp indicating when the account was created |


### GET `/v3/account/names`

Get sub account information

<aside class="notice">
This endpoint can only be called using API keys paired with the parent account! Returns all active subaccounts.
</aside>

> **Request**

```
GET v3/account/names
```

> **Successful response format**

```json
{
    "success": true,
    "data":  [  {
                    "accountId": "21213",
                    "name": "Test 1"
                }, 
                {
                    "accountId": "21214",
	                "name": "Test 2"
              }
          ] 
}
```


Response Field | Type | Description |
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
name | STRING | Account name |


### GET `/v3/wallet`

Get account or sub-account wallet

> **Request**

```
GET v3/wallet?subAcc={name1},{name2}&type={type}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
             "accountId": "21213",
             "name": "main",
             "walletHistory": [
                  {
                    "id": "810583329159217160",
                    "asset": "USDT",
                    "type": "DEPOSIT", 
                    "amount": "10",
                    "createdAt": "162131535213"  
                    }  	
                    	]
        }
    ]
}
```

<aside class="notice">
Calling this endpoint using an API key pair linked to the parent account with the parameter "subAcc" allows the caller to include additional sub-accounts in the response. This feature does not work when using API key pairs linked to a sub-account.
</aside>

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
subAcc | STRING | NO | Max 5 |
type | STRING | NO | DEPOSIT, WITHDRAWAL, etc, default return all, most recent first |
limit | LONG | NO | Default 200, max 500 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
name | STRING | Account name |
walletHistory | LIST of dictionaries | |
id | STRING | A unique ID |
amount | STRING | Amount |
asset | STRING | Asset name |
type | STRING | |
createdAt/lastUpdatedAt | STRING | Millisecond timestamp `created time or updated time` |


### POST `/v3/transfer`

Sub-account balance transfer

<aside class="notice">
Transferring funds between sub-accounts is restricted to API keys linked to the parent account.
</aside>

> **Request**

```
POST /v3/transfer
```
```json
{
    "asset": "flexUSD",
    "quantity": "1000",
    "fromAccount": "14320",
    "toAccount": "15343"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD", 
        "quantity": "1000",
        "fromAccount": "14320",
        "toAccount": "15343",
        "transferredAt": "1635038730480"
    }
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | |
quantity | STRING | YES | |
fromAccount | STRING | YES | |
toAccount | STRING | YES | |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
fromAccount | STRING | |
toAccount | STRING | |
transferredAt | STRING | Millisecond timestamp |


### GET `/v3/transfer`

Sub-account balance transfer history

> **Request**

```url
GET /v3/transfer?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD", 
            "quantity": "1000",
            "fromAccount": "14320",
            "toAccount": "15343",
            "id": "703557273590071299",
            "status": "COMPLETED",
            "transferredAt": "1634779040611"
        }
    ]
}
```

<aside class="notice">
API keys linked to the parent account can get all account transfers, 
while API keys linked to a sub-account can only see transfers where the sub-account is either the "fromAccount" or "toAccount".
</aside>

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
fromAccount | STRING | |
toAccount | STRING | |
id | STRING | |
status | STRING | |
transferredAt | STRING | Millisecond timestamp |


### GET `/v3/balances`

> **Request**

```
GET /v3/balances?subAcc={name1},{name2}&asset={asset}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "accountId": "21213",
            "name": "main",
            "balances": [
               {
                   "asset": "BTC",
                   "total": "4468.823",              
                   "available": "4468.823",        
                   "reserved": "0",
                   "lastUpdatedAt": "1593627415234"
               },
               {
                   "asset": "FLEX",
                   "total": "1585.890",              
                   "available": "325.890",         
                   "reserved": "1260",
                   "lastUpdatedAt": "1593627415123"
               }
                       ]
        }
    ]
}
```

<aside class="notice">
Calling this endpoint using an API key pair linked to the parent account with the parameter "subAcc" allows the caller to include additional sub-accounts in the response. This feature does not work when using API key pairs linked to a sub-account.
</aside>

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
subAcc | STRING | NO | Name of sub account. If no subAcc is given, then the response contains only the account linked to the API-Key. Multiple subAccs can be separated with a comma, maximum of 10 subAccs, e.g. subone,subtwo |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
name | STRING | Parent account with the name “main” and take the first place|
balances | LIST of dictionaries | |
asset | STRING | Asset name |
available | STRING | 	Available balance |
reserved | STRING | Reserved balance |
lastUpdatedAt | STRING | Timestamp of updated at |

### GET  `/v3/positions`

> **Request**

```
GET /v3/positions?subAcc={name1},{name2}&marketCode={marketCode}
```

> **Successful response format**

```json
{
  "success": True,
  "data": [
      {
        "accountId": "1234",
        "name": "PERMISSIONLESS_1234",
        "positions": [
            {
              "marketCode": "XYZ-USDT-SWAP-LIN",
              "baseAsset": "XYZ",
              "counterAsset": "USDT",
              "position": "566.0",
              "entryPrice": "7.3400",
              "markPrice": "9.94984016",
              "positionPnl": "1477.169530560",
              "estLiquidationPrice": "0",
              "lastUpdatedAt": "1673231134601",
              "marginBalance": "61350.82873932",
              "maintenanceMargin": "988.61",
              "marginRatio": "0.01611209",
              "leverage": "2"
             }
        ]
      }
  ]
}
```

<aside class="notice">
Calling this endpoint using an API key pair linked to the parent account with the parameter "subAcc" allows the caller to include additional sub-accounts in the response. This feature does not work when using API key pairs linked to a sub-account.
  
Returns an empty array `[]` when no positions were found 
</aside>

Returns position data

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Default all markets |
subAcc | STRING | NO | Name of sub account. If no subAcc is given, then the response contains only the account linked to the API-Key. Multiple subAccs can be separated with a comma, maximum of 10 subAccs, e.g. subone,subtwo |

Response Fields | Type | Description |
--------------- | ---- | ----------- |
accountId | STRING | Account ID |
name | STRING | Parent account with the name “main” and take the first place|
positions | LIST of dictionaries | |
marketCode | STRING | Contract symbol, e.g. 'BTC-USDT-SWAP-LIN' |
baseAsset | STRING |
counterAsset | STRING |
position | STRING | Position size, e.g. '0.94' |
entryPrice | STRING | Average entry price |
markPrice | STRING |
positionPnl | STRING | Postion profit and lost |
estLiquidationPrice | STRING | Estimated liquidation price, return 0 if it is negative(<0) |
lastUpdated | STRING| Timestamp when position was last updated |
marginBalance | STRING |Appears in the position section only for positions using isolated margin. Isolated margin + Unrealized position PnL|
maintenanceMargin | STRING |Appears in the position section only for positions using isolated margin|
marginRatio | STRING | Appears in the position section only for positions using isolated margin|
leverage | STRING | Appears in the position section only for positions using isolated margin|


### GET `/v3/funding`

Get funding payments by marketCode and sorted by time in descending order.

> **Request**

```json
GET v3/funding?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | NO | e.g. `BTC-USDT-REPO-LIN` |
limit | LONG | NO | default is `200`, max is `500` |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

> **SUCCESSFUL RESPONSE**

```json
{
    "success": true,
    "data": [
        {
            "id": "810583329213284361",
            "marketCode": "BTC-USDT-SWAP-LIN",
            "payment": "-122.17530872",
            "fundingRate": "-0.00005",
            "position": "-61.093",
            "indexPrice": "39996.5",
            "createdAt": "1627617632190"
        }
    ]
}
```

Response Fields | Type | Description |
------------------- | ---- | ----------- |
id | STRING | A unique ID |
marketCode | STRING | Market code |
payment | STRING | Funding payment |
fundingRate | STRING | Funding rate |
position | STRING | Position |
indexPrice | STRING | index price |
createdAt | STRING | Timestamp of this response |



## Deposits & Withdrawals - Private

### GET `/v3/deposit-addresses`

Deposit addresses

> **Request**

```
GET /v3/deposit-addresses?asset={asset}&network={network}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "address":"0xD25bCD2DBb6114d3BB29CE946a6356B49911358e"
    }
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES |
network | STRING | YES |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
address | STRING | Deposit address |
memo | STRING | Memo (tag) if applicable |


### GET `/v3/deposit`

Deposit history

> **Request**

```
GET /v3/deposit?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD",
            "network": "SLP",
            "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
            "quantity": "1000.0",
            "id": "651573911056351237",
            "status": "COMPLETED",
            "txId": "38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d",
            "creditedAt": "1617940800000"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | | 
network | STRING | |
address | STRING | Deposit address |
memo | STRING | Memo (tag) if applicable |
quantity | STRING | |
id | STRING | |
status | STRING | |
txId | STRING | |
creditedAt | STRING | Millisecond timestamp |


### GET `/v3/withdrawal-addresses`

Withdrawal addresses

> **Request**

```
GET /v3/withdrawal-addresses?asset={asset}?network={network}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "FLEX",
            "network": "ERC20",
            "address": "0x047a13c759D9c3254B4548Fc7e784fBeB1B273g39",
            "label": "farming",
            "whitelisted": true
        }
    ]
}
```

Provides a list of all saved withdrawal addresses along with their respected labels, network, and whitelist status

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
network | STRING | NO | Default all networks |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable |
label | STRING | Withdrawal address label |
whitelisted | BOOL | |


### GET `/v3/withdrawal`

Withdrawal history

> **Request**

```
GET /v3/withdrawal?id={id}&asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "id": "651573911056351237",
            "asset": "flexUSD",
            "network": "SLP",
            "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
            "quantity": "1000.0",
            "fee": "0.000000000",
            "status": "COMPLETED",
            "txId": "38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d",
            "requestedAt": "1617940800000",
            "completedAt": "16003243243242"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
id | STRING | NO | |
asset | STRING | NO | Default all assets |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. This filter applies to "requestedAt" |
endTime | LONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. This filter applies to "requestedAt" |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
id | STRING | |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable |
quantity | STRING | |
fee | STRING | |
status | STRING | COMPLETED, PROCESSING, PENDING, ON HOLD, CANCELED, or FAILED| 
txId | STRING | |
requestedAt | STRING | Millisecond timestamp |
completedAt | STRING | Millisecond timestamp |


### POST `/v3/withdrawal`

Withdrawal request

> **Request**

```
POST /v3/withdrawal
```
```json
{
    "asset": "flexUSD",
    "network": "SLP",
    "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
    "quantity": "100",
    "externalFee": true,
    "tfaType": "GOOGLE",
    "code": "743249"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "id": "752907053614432259",
        "asset": "flexUSD",
        "network": "SLP",
        "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
        "quantity": "1000.0",
        "externalFee": true,
        "fee": "0",
        "status": "PENDING",
        "requestedAt": "1617940800000"
    }
}
```
Withdrawals may only be initiated by API keys that are linked to the parent account and have withdrawals enabled. If the wrong 2fa code is provided the endpoint will block for 10 seconds.

Request Parameter | Type | Required | Description |
----------------- | ---- |--------- | ----------- |
asset | STRING | YES |
network | STRING | YES |
address | STRING | YES |
memo | STRING | NO |Memo is required for chains that support memo tags |
quantity | STRING | YES |
externalFee | BOOL | YES | If false, then the fee is taken from the quantity, also with the burn fee for asset SOLO |
tfaType | STRING | NO | GOOGLE, or AUTHY_SECRET, or YUBIKEY |
code | STRING | NO | 2fa code if required by the account |

Response Field | Type | Description |
-------------- | ---- | ----------- |
id | STRING | |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | | 
quantity | STRING | |
externalFee | BOOL | If false, then the fee is taken from the quantity |
fee | STRING | |
status | STRING | |
requestedAt | STRING | Millisecond timestamp |


### GET `/v3/withdrawal-fee`

Withdrawal fee estimate

> **Request**

```
GET /v3/withdrawal-fee?asset={asset}&network={network}&address={address}&memo={memo}&quantity={quantity}&externalFee={externalFee}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "network": "SLP",
        "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
        "quantity": "1000.0",
        "externalFee": true,
        "estimatedFee": "0"
    }
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | |
network | STRING | YES | |
address | STRING | YES | |
memo | STRING | NO | Required only for 2 part addresses (tag or memo)|
quantity | STRING | YES | |
externalFee | BOOL | NO | Default false. If false, then the fee is taken from the quantity|

Response Field | Type | Description | 
---------------| ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable|
quantity | STRING | |
externalFee | BOOL | If false, then the fee is taken from the quantity|
estimatedFee | STRING | |


## Orders - Private

### GET `/v3/orders/status`

Get latest order status

> **Request**

```
GET /v3/orders/status?orderId={orderId}&clientOrderId={clientOrderId}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "orderId": "1000387920513",
        "clientOrderId": "1612249737434",
        "marketCode": "FLEX-USDT",
        "status": "FILLED",
        "side": "BUY",
        "price": "5.200",
        "isTriggered": false,
        "remainQuantity": "0",
        "totalQuantity": "12",
        "cumulativeMatchedQuantity": "12",
        "avgFillPrice": "5.200",
        "orderType": "LIMIT",
        "timeInForce": "GTC",
        "source": "11",
        "createdAt": "1655980336520",
        "lastModifiedAt": "1655980393780",
        "lastMatchedAt": "1655980622848"
    }
}
```
<aside class="notice">
CANCELED orders are not in the orderbook, the remainQuantity in the CANCELED orders is historical remaining quantity
</aside>

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
orderId | LONG | YES if no clientOrderId | Order ID |
clientOrderId | LONG | YES if no orderId | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |

Response Field | Type | Description |
-------------- | ---- | ----------- |
orderId | STRING | Order ID |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Market code |
status | STRING | Available values: `CANCELED`, `OPEN`, `PARTIAL_FILL`, `FILLED` |
side | STRING | Side of the order, `BUY` or `SELL` |
price | STRING | Price or limit price in the case of a STOP order |
stopPrice | STRING | Trigger price for a STOP order |
isTriggered | BOOL | `true` for a STOP order |
remainQuantity | STRING | Remaining quantity |
totalQuantity | STRING | Total quantity |
cumulativeMatchedQuantity | STRING | Cumulative quantity of the matches |
avgFillPrice | STRING | Average of filled price |
avgLeg1Price | STRING | Average of leg1 price |
avgLeg2Price | STRING | Average of leg2 price |
fees | LIST of dictionaries | Overall fees with instrument ID, if FLEX is no enough to pay the fee then USDT will be paid |
orderType | STRING | Type of the order, availabe values: `MARKET`, `LIMIT`, `STOP_LIMIT`,`STOP_MARKET` |
timeInForce | STRING | Client submitted time in force. <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade) |
source | STRING | Source of the request, available values: `0`, `2`, `10`, `11`, `13`, `22`, `101`, `102`, `103`, `111`. <p>Enumeration: `0: GUI`, `2: Borrow`, `11: REST`, `13: Websocket`, `22: Delivery`, `101: Automatic borrow`, `102: Borrow position liquidation`, `103: Contract liquidation`, `111: Automatic repayment`</p> |
createdAt | STRING | Millisecond timestamp of the order created time |
lastModifiedAt | STRING | Millisecond timestamp of the order last modified time |
lastMatchedAt | STRING | Millisecond timestamp of the order last matched time |
canceledAt | STRING | Millisecond timestamp of the order canceled time |


### GET `/v3/orders/working`

Returns all the open orders of the account connected to the API key initiating the request.

> **Request**

```
GET /v3/orders/working?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}
```

> **Successful response format**

```json
{
    "success": true,
    "data":  [ {		
                "orderId": "160067484555913076",
                "clientOrderId": "123",
                "marketCode": "BTC-USDT-SWAP-LIN",
                "status": "LIMIT"|"STOP",
                "side": "SELL",
                "price": "1.0",
                "stopPrice": "0.9",
                "isTriggered": true,
                "quantity": "0.001",
                "remainQuantity": "0.001",
                "matchedQuantity": "0",
                "orderType": "LIMIT", 
                "timeInForce": "GTC",
                "source": "0",
                "createdAt": "1613089383656", 
                "lastModifiedAt": null,
                "lastMatchedAt": null
               }]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO |default most recent orders first |
orderId | LONG | NO | Client assigned ID to help manage and identify orders |
clientOrderId | LONG | NO | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |


Response Field | Type | Description |
-------------- | ---- | ----------- |
orderId | STRING | Order ID |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Market code |
status | STRING | Available values: `OPEN`, `PARTIALLY_FILLED` |
side | STRING | Side of the order, `BUY` or `SELL` |
price | STRING | Price or limit price in the case of a STOP order |
stopPrice | STRING | Trigger price for a STOP order |
isTriggered | BOOL | `true` for a STOP order |
quantity | STRING |  Quantity |
remainQuantity | STRING | Remaining quantity |
matchedQuantity | STRING | Matched Quantity |
orderType | STRING | Type of the order, availabe values: `MARKET`, `LIMIT`, `STOP_LIMIT`,`STOP_MARKET` |
timeInForce | STRING | Client submitted time in force. <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade) |
source | STRING | Source of the request, available values: `0`, `2`, `10`, `11`, `13`, `22`, `101`, `102`, `103`, `111`. <p>Enumeration: `0: GUI`, `2: Borrow`,  `11: REST`, `13: Websocket`, `22: Delivery`, `101: Automatic borrow`, `102: Borrow position liquidation`, `103: Contract liquidation`, `111: Automatic repayment`</p> |
createdAt | STRING | Millisecond timestamp of the order created time |
lastModifiedAt | STRING | Millisecond timestamp of the order last modified time |
lastMatchedAt | STRING | Millisecond timestamp of the order last matched time |



### POST `/v3/orders/place`

> **Request**

```
POST /v3/orders/place
```

```json
{
    "recvWindow": 20000, 
    "responseType": "FULL", 
    "timestamp": 1615430912440, 
    "orders": [
             {
                   "clientOrderId": 1612249737724, 
                   "marketCode": "BTC-USDT-SWAP-LIN", 
                   "side": "SELL", 
                   "quantity": "0.001", 
                   "timeInForce": "GTC", 
                   "orderType": "LIMIT", 
                   "price": "50007"
               }, 
               {
                   "clientOrderId": 1612249737724, 
                   "marketCode": "BTC-USDT-SWAP-LIN", 
                   "side": "BUY", 
                   "quantity": "0.002", 
                   "timeInForce": "GTC", 
                   "orderType": "LIMIT", 
                   "price": "54900"
               }
    ]
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
               {
                    "code": "710006",
                    "message": "FAILED balance check as balance (0E-9) < value (0.001)",
                    "submitted": false,
                    "clientOrderId": "1612249737724",
                    "marketCode": "BTC-USDT-SWAP-LIN",
                    "side": "SELL",
                    "price": "52888.0",
                    "quantity": "0.001",   
                    "orderType": "LIMIT",
                    "timeInForce": "GTC",
                    "createdAt": "16122497377340",
                    "source": "0"
                },
                {
                    "notice": "OrderOpened", 
                    "accountId": "1076", 
                    "orderId": "1000132664173",
                    "submitted": false,
                    "clientOrderId": "1612249737724",
                    "marketCode": "BTC-USDT-SWAP-LIN",
                    "status": "OPEN",
                    "price": "23641.0",
                    "stopPrice": null,
                    "isTriggered": "false",
                    "quantity": "0.01", 
                    "remainQuantity": "0.01",
                    "matchId": null,
                    "matchPrice": null, 
                    "matchQuantity": null, 
                    "feeInstrumentId": null,
                    "fees": null,
                    "orderType": "LIMIT", 
                    "timeInForce": "GTC", 
                    "createdAt": "1629192975532",    	
                    "lastModifiedAt": null,			
                    "lastMatchedAt": null	
                }
          ]
}
```

Place orders.

<aside class="notice">
You can place up to 8 orders at a time in REST API
</aside>

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected |
timestamp | STRING | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | |
clientOrderId | ULONG | YES | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | YES | Market code |
side | STRING | YES | `BUY` or `SELL` |
quantity | STRING | YES | Quantity |
amount | STRING | YES | Amount |
quantity | STRING | YES | displayQuantity |
timeInForce | STRING | NO | Default `GTC` |
orderType | STRING | YES | `LIMIT` or `MARKET` or `STOP` or `STOP_MARKET`|
price | STRING | NO | Limit price for the limit order |
stopPrice | STRING | NO | Stop price for the stop order |
limitPrice | STRING | NO | Limit price for the stop limit order |

Response Fields | Type | Description | 
--------------------| ---- | ----------- |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpened` |
accountId | STRING | Account ID |
code | STRING | Error code |
message | STRING | Error message |
orderId | STRING | |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | |
side | STRING | `SELL` or `BUY` |
price | STRING | |
stopPrice | STRING | |
isTriggered | STRING | false (or true for STOP order types) |
quantity | STRING | |
amount | STRING | |
displayQuantity | STRING | |
remainQuantity | STRING | Remaining quantity |
matchId | STRING | |
matchPrice | STRING | |
matchQuantity | STRING | |
matchQuantity | STRING | Matched quantity |
feeInstrumentId | STRING | Instrument ID of fees paid from this match ID |
fees | STRING | Amount of fees paid from this match ID |
orderType | STRING | `MARKET` or `LIMIT` or `STOP` or or `STOP_MARKET` |
timeInForce | STRING | |
source | STRING | Source of the request, available values: `0`, `2`, `10`, `11`, `13`, `22`, `101`, `102`, `103`, `111`. <p>Enumeration: `0: GUI`, `2: Borrow`, `11: REST`, `13: Websocket`, `22: Delivery`, `101: Automatic borrow`, `102: Borrow position liquidation`, `103: Contract liquidation`, `111: Automatic repayment`</p> |
createdAt | STRING | Millisecond timestamp of the order created time |
lastModifiedAt | STRING | Millisecond timestamp of the order last modified time |
lastMatchedAt | STRING | Millisecond timestamp of the order last matched time |


### DELETE `/v3/orders/cancel`

> **Request**

```
DELETE /v3/orders/cancel
```

```json
{
    "recvWindow": 200000, 
    "responseType": "FULL", 
    "timestamp": 1615454880374, 
    "orders": [
        {
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "orderId": "304384250571714215", 
            "clientOrderId": 1615453494726
        }, 
        {
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "clientOrderId": 1612249737724
        }
    ]
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
                {
                    "notice": "OrderClosed", 
                    "accountId": "12005486", 
                    "orderId": "304384250571714215",
                    "submitted": true,
                    "clientOrderId": "1615453494726", 
                    "marketCode": "BTC-USDT-SWAP-LIN", 
                    "status": "CANCELED_BY_USER", 
                    "side": "BUY", 
                    "price": "4870.0", 
                    "stopPrice": null,
                    "isTriggered": false,
                    "quantity": "0.001", 
                    "remainQuantity": "0.001",
                    "orderType": "LIMIT",  
                    "timeInForce": "GTC", 
                    "canceledAt": "1629712561919"
                 },
                 {
                    "code": "40035",
                    "message": "Open order not found with id",
                    "submitted": false,
                     "orderId": "204285250571714316",
                     "clientOrderId": "1612249737724",
                     "marketCode": "BTC-USDT-SWAP-LIN",
                     "canceledAt": "1615454881433"
                 }
    	]

}
```

Cancel orders.

<aside class="notice">
You can cancel up to 8 orders at a time in REST API
</aside>

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | LONG | NO | |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | |
marketCode | STRING | YES | |
orderId | STRING | Either one of orderId or clientOrderId is required | |
clientOrderId | ULONG | Either one of orderId or clientOrderId is required | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |


Response Fields | Type | Description | 
--------------------| ---- | ----------- |
notice | STRING | `OrderClosed` |
accountId | STRING | Account ID |
code | STRING | Error code |
message | STRING | Error message |
orderId | STRING | |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | |
side | STRING | `SELL` or `BUY` |
price | STRING | |
stopPrice | STRING | |
isTriggered | STRING | false (or true for STOP order types) |
quantity | STRING | |
remainQuantity | STRING | Remaining quantity |
orderType | STRING | `MARKET` or `LIMIT` or `STOP` or `STOP_MARKET` |
timeInForce | STRING | |
canceledAt | STRING | Millisecond timestamp of the order cancel time |



### DELETE ` /v3/orders/cancel-all`

> **Request**

```
DELETE  /v3/orders/cancel-all
```

```json
{
            "marketCode": "BTC-USDT-SWAP-LIN"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data":  {
                   "notice": "Orders queued for cancelation"
              }
}
```

Cancel orders.

<aside class="notice">
Cancels all open orders for the **specified market** for the account connected to the API key initiating the request.
</aside>

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
marketCode | STRING | NO | |

Response Fields | Type | Description | 
--------------------| ---- | ----------- |
notice | STRING | `Orders queued for cancelation` or `No working orders found”` |



## Trades - Private

### GET `/v3/trades`

Returns the most recent trades of the account connected to the API key initiating the request.

> **Request**

```
GET /v3/trades?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
                {
                    "orderId": "160067484555913076",
                    "clientOrderId": "123",
                    "matchId": "160067484555913077",
                    "marketCode": "FLEX-USDT",
                    "side": "SELL",
                    "matchedQuantity": "0.1",
                    "matchPrice": "0.065",
                    "total": "0.0065",	
                    "leg1Price'": "0.0065", 		
                    "leg2Price": "0.0065",			
                    "orderMatchType": "TAKER",
                    "feeAsset": "FLEX",
                    "fee":"0.0196",
                    "source": "10",
                    "matchedAt": "1595514663626"

               }
            ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | String | default most recent trades first |
limit | LONG | NO | max 500, default 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description |
-------------- | ---- | ----------- |
orderId | STRING | Order ID |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
matchId | STRING | Match ID |
marketCode | STRING | Market code |
side | STRING | Side of the order, `BUY` or `SELL` |
matchedQuantity | STRING | Match quantity |
matchPrice | STRING | Match price |
total | STRING | Total price |
leg1Price | STRING | `REPO & SPREAD` |
leg2Price | STRING | `REPO & SPREAD` |
orderMatchType | STRING | `TAKER`,`MAKER` |
feeAsset | STRING | Instrument ID of the fees |
fee | STRING | Fees |
source | STRING | Source of the request, available values: `0`, `2`, `10`, `11`, `13`, `22`, `101`, `102`, `103`, `111`. <p>Enumeration: `0: GUI`, `2: Borrow`, `11: REST`, `13: Websocket`, `22: Delivery`, `101: Automatic borrow`, `102: Borrow position liquidation`, `103: Contract liquidation`, `111: Automatic repayment`</p> |
matchedAt | STRING | Millisecond timestamp of the order matched time |


## Market Data - Public

### GET `/v3/markets`

Get a list of markets on Opnx.

> **Request**

```
GET /v3/markets?marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USDT",
            "name": "BTC/USDT",
            "referencePair": "BTC/USDT",
            "base": "BTC",
            "counter": "USDT",
            "type": "SPOT",
            "tickSize": "0.1",
            "minSize": "0.001",
            "listedAt": "1593345600000",
            "upperPriceBound": "65950.5",
            "lowerPriceBound": "60877.3",
            "markPrice": "63413.9",
            "lastUpdatedAt": "1635848576163"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market Code |
name | STRING | Name of the contract |
referencePair | STRING | Reference pair |
base | STRING | Base asset |
counter | STRING | Counter asset |
type | STRING | Type of the contract |
tickSize | STRING | Tick size of the contract |
minSize | STRING | Minimum quantity |
listedAt | STRING | Listing date of the contract |
settlementAt | STRING | Timestamp of settlement if applicable i.e. Quarterlies and Spreads |
upperPriceBound | STRING | Sanity bound |
lowerPriceBound | STRING | Sanity bound |
markPrice | STRING | Mark price |
indexPrice | STRING | index price |
lastUpdatedAt | STRING | |


### GET `/v3/assets`

Get a list of assets supported on Opnx

> **Request**

```
GET /v3/assets?asset={asset}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "USDT",
            "isCollateral": true,
            "loanToValue": "1.000000000",
            "networkList": [
                {
                    "network": "ERC20",
                    "transactionPrecision": "6",
                    "isWithdrawalFeeChargedToUser": true,
                    "canDeposit": true,
                    "canWithdraw": true,
                    "minDeposit": "0.0001",
                    "minWithdrawal": "2"
                }
            ]
        },
        {
            "asset": "LINK",
            "isCollateral": true,
            "loanToValue": "0.800000000",
            "networkList": [
                {
                    "network": "ERC20",
                    "tokenId": "0x514910771af9ca656af840dff83e8264ecf986ca",
                    "transactionPrecision": "18",
                    "isWithdrawalFeeChargedToUser": true,
                    "canDeposit": true,
                    "canWithdraw": true,
                    "minDeposit": "0.0001",
                    "minWithdrawal": "0.0001"
                }
            ]
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name |

Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING | Asset name |
isCollateral | BOOL | Indicates it is collateral or not |
loanToValue | STRING | Loan to value of the asset |
networkList | LIST | List of dictionaries |
network | STRING | Network for deposit and withdrawal |
tokenId | STRING | Token ID |
transactionPrecision | STRING | Precision for the transaction |
isWithdrawalFeeChargedToUser | BOOL | Indicates the withdrawal fee is charged to user or not |
canDeposit | BOOL | Indicates can deposit or not |
canWithdraw | BOOL | Indicates can withdraw or not |
minDeposit | STRING | Minimum deposit amount |
minWithdrawal | STRING | Minimum withdrawal amount |


### GET `/v3/tickers`

Get tickers.

> **Request**

```
GET /v3/tickers?marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USDT-SWAP-LIN",
            "markPrice": "41512.4",
            "open24h": "41915.3",
            "high24h": "42662.2",
            "low24h": "41167.0",
            "volume24h": "114341.4550",
            "currencyVolume24h": "2.733",
            "openInterest": "3516.506000000",
            "lastTradedPrice": "41802.5",
            "lastTradedQuantity": "0.001",
            "lastUpdatedAt": "1642585256002"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market code |
markPrice | STRING | Mark price |
open24h | STRING | Rolling 24 hour opening price |
high24h | STRING | Rolling 24 hour highest price |
low24h | STRING | Rolling 24 hour lowest price |
volume24h | STRING | Rolling 24 hour notional trading volume |
currencyVolume24h | STRING | Rolling 24 hour trading volume in base currency |
openInterest | STRING | Open interest |
lastTradedPrice | STRING | Last traded price |
lastTradedQuantity | STRIN | Last traded quantity |
lastUpdatedAt | STRING | Millisecond timestamp of lastest update |


### GET `/v3/funding/estimates`

> **Request**

```
GET /v3/funding/estimates?marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "WBTC-USDT-SWAP-LIN",
            "fundingAt": "1667012400000",
            "estFundingRate": "0"
        },
        {
            "marketCode": "BTC-USDT-SWAP-LIN",
            "fundingAt": "1667012400000",
            "estFundingRate": "0"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Market code |


Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market code |
estFundingRate | STRING | Estimates funding rate |
fundingAt | STRING | Millisecond timestamp |


### GET `/v3/candles`

Get candles.

> **Request**

```
GET /v3/candles?marketCode={marketCode}&timeframe={timeframe}&limit={limit}
&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true, 
    "timeframe": "3600s", 
    "data": [
        {
            "open": "35888.80000000", 
            "high": "35925.30000000", 
            "low": "35717.00000000", 
            "close": "35923.40000000", 
            "volume": "0",
            "currencyVolume": "0",
            "openedAt": "1642932000000"
        },
        {
            "open": "35805.50000000", 
            "high": "36141.50000000", 
            "low": "35784.90000000", 
            "close": "35886.60000000", 
            "volume": "0",
            "currencyVolume": "0",
            "openedAt": "1642928400000"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Market code |
timeframe | STRING | NO | Available values: `60s`,`300s`,`900s`,`1800s`,`3600s`,`7200s`,`14400s`,`86400s`, default is `3600s` |
limit | LONG | NO | Default 200, max 500 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. |
endTime | LONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. |

Response Field | Type | Description |
-------------- | ---- | ----------- |
timeframe | STRING | Available values: `60s`,`300s`,`900s`,`1800s`,`3600s`,`7200s`,`14400s`,`86400s` |
open | STRING | Opening price |
high | STRING | Highest price |
low | STRING | Lowest price |
close | STRING | Closing price |
volume | STRING | Trading volume in counter currency |
currencyVolume | STRING | Trading volume in base currency |
openedAt | STRING | Millisecond timestamp of the candle open |


### GET `/v3/depth`

Get depth.

> **Request**

```
GET /v3/depth?marketCode={marketCode}&level={level}
```

> **Successful response format**

```json
{
    "success": true, 
    "level": "5", 
    "data": {
        "marketCode": "BTC-USDT-SWAP-LIN", 
        "lastUpdatedAt": "1643016065958", 
        "asks": [
            [
                39400, 
                0.261
            ], 
            [
                41050.5, 
                0.002
            ], 
            [
                41051, 
                0.094
            ], 
            [
                41052.5, 
                0.002
            ], 
            [
                41054.5, 
                0.002
            ]
        ], 
        "bids": [
            [
                39382.5, 
                0.593
            ], 
            [
                39380.5, 
                0.009
            ], 
            [
                39378, 
                0.009
            ], 
            [
                39375.5, 
                0.009
            ], 
            [
                39373, 
                0.009
            ]
        ]
    }
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Market code |
level | LONG | NO | Default 5, max 100 |

Response Field | Type | Description |
-------------- | ---- | ----------- |
level | LONG | Level |
marketCode | STRING | Market code |
lastUpdatedAt | STRING | Millisecond timestamp of the lastest depth update |
asks | LIST of floats | Sell side depth: [price, quantity] |
bids | LIST of floats | Buy side depth: [price, quantity] |


### GET `/v3/markets/operational`

Get markets operational.

> **Request**

```
GET /v3/markets/operational?marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USDT",
            "operational": true 
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Market code |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market code |
operational | BOOL | whether the market of the marketCode is operational |


### GET `/v3/exchange-trades`

> **Request**

```
GET /v3/exchange-trades?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USDT-SWAP-LIN",
            "matchPrice": "9600.00000" ,
            "matchQuantity": "0.100000" ,
            "side": "BUY" ,
            "matchType": "TAKER" ,
            "matchedAt": "1662207330439" 
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Market code |
limit | LONG | NO | Default 200, max 500 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field |Type | Description| 
-------------------------- | -----|--------- |
marketCode | STRING    | |
matchPrice | STRING    | |
matchQuantity | STRING    | |
side | STRING    | |
matchType | STRING    | |
matchedAt | STRING    | |


### GET `/v3/funding/rates`

Get all historical financing rates

> **Request**

```
GET /v3/funding/rates?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USDT-SWAP-LIN",
            "fundingRate": "0.0",
            "createdAt": "1628362803134"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |
limit | LONG | NO | Default 200, max 500 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. |
endTime | LONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market code |
fundingRate | STRING | Funding rate |
createdAt | STRING | Millisecond timestamp |
