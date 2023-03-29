# REST API V2 [no longer maintained]

**TEST** site

* `https://stgapi.opnx.com`

**LIVE** site

* `https://api.opnx.com`

For clients who do not wish to take advantage of Opnx's native WebSocket API, Opnx offers a RESTful API that implements much of the same functionality.

## Rest Authentication

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
from urllib.parse import urlencode

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123
method = <API-METHOD>

# Optional and can be omitted depending on the REST method being called 
body = urlencode({'key1': 'value1', 'key2': 'value2'})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.json())
```

Public market data methods do not require authentication, however private methods require a *Signature* to be sent in the header of the request.  These private REST methods  use HMAC SHA256 signatures. 

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a clients API Secret as the key and a message string as the value for the HMAC operation.  

The message string is constructed by the following formula:-

`msgString = Timestamp + "\n" + Nonce + "\n" + Verb + "\n" + URL + "\n" + Path +"\n" + Body`

Component | Required | Example | Description| 
-------------------------- |--------- |------- |------- | 
Timestamp | Yes | 2020-04-30T15:20:30 | YYYY-MM-DDThh:mm:ss
Nonce | Yes | 123 | User generated
Verb | Yes| 'GET' | Uppercase
Path | Yes | 'v2stgapi.opnx.com' |
Method | Yes | '/v2/positions | Available REST methods: <li>`V2/positions`</li><li>`V2/orders`</li><li>`V2/balances`</li>
Body | No | instrumentID=BTC-USDT-SWAP-LIN | Optional and dependent on the REST method being called

The constructed message string should look like:-

  `2020-04-30T15:20:30\n
  123\n
  GET\n
  v2stgapi.opnx.com\n
  /v2/positions\n
  instrumentID=BTC-USDT-SWAP-LIN`

Note the newline characters after each component in the message string. 
If *Body* is ommitted it's treated as an empty string.

Finally you must use the HMAC SHA256 operation to get the hash value using the API Secret as the key and the constructed message string as the value for the HMAC operation. Then encode this hash value with BASE-64.  This output becomes the signature for the specified authenticated REST API method. 

The signature must then be included in the header of the REST API call like so:

`header = {'Content-Type': 'application/json', 'AccessKey': API-KEY, 'Timestamp': TIME-STAMP, 'Signature': SIGNATURE, 'Nonce': NONCE}`

##Methods - Private

All private REST API methods require authentication using the approach explained above. 

###GET `/v2/accountinfo`

> **Request**

```json
GET /v2/accountinfo
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/accountinfo'

params = ""

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
    "event": "accountinfo",
    "timestamp": "1648800334773",
    "accountId": "677473",
    "data": [
        {
            "accountId": "677473",
            "tradeType": "LINEAR",
            "marginCurrency": "USDT",
            "totalBalance": "33860283.6914636",
            "availableBalance": "33860283.6914636",
            "collateralBalance": "28661274.5806472",
            "portfolioVarMargin": "910198.2835522",
            "riskRatio": "31.4890",
            "timestamp": "1648800334761"
        }
    ]
}
```

```python
{
    "event": "accountinfo",
    "timestamp": "1648800334773",
    "accountId": "677473",
    "data": [
        {
            "accountId": "677473",
            "tradeType": "LINEAR",
            "marginCurrency": "USDT",
            "totalBalance": "33860283.6914636",
            "availableBalance": "33860283.6914636",
            "collateralBalance": "28661274.5806472",
            "portfolioVarMargin": "910198.2835522",
            "riskRatio": "31.4890",
            "timestamp": "1648800334761"
        }
    ]
}
```

Returns the account level information connected to the API key initiating the request. 

<sub>**Response Fields**</sub> 

Fields |Type | Description 
-------------------------- | -----|--------- |
event | STRING | `accountinfo`
timestamp | STRING | Millisecond timestamp
accountId | STRING | Account ID
data | LIST of dictionary |
accountId | STRING | Account ID
tradeType | STRING | Account type `LINEAR`
marginCurrency | STRING | Asset `USDT`
totalBalance | STRING | Total balance denoted in marginCurrency
collateralBalance | STRING | Collateral balance with LTV applied
availableBalance | STRING | Available balance
portfolioVarMargin| STRING | Portfolio margin
riskRatio | STRING | collateralBalance / portfolioVarMargin, Orders are rejected/cancelled if the risk ratio drops below 1 and liquidation occurs if the risk ratio drops below 0.5
timestamp | STRING | Millisecond timestamp


###GET `/v2/balances`

> **Request**

```json
GET /v2/balances
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/balances'

params = ""

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
    "event": "balances",
    "timestamp": "1648800661246",
    "accountId": "677473",
    "tradeType": "LINEAR",
    "data": [
        {
            "instrumentId": "1INCH",
            "total": "100.00000000",
            "available": "100.00000000",
            "reserved": "0.00000000",
            "quantityLastUpdated": "1646413705113"
        },
        {
            "instrumentId": "AAVE",
            "total": "100.00000000",
            "available": "100.00000000",
            "reserved": "0.00000000",
            "quantityLastUpdated": "1646413705827"
        }
    ]
}
```

```python
{
    "event": "balances",
    "timestamp": "1648800661246",
    "accountId": "677473",
    "tradeType": "LINEAR",
    "data": [
        {
            "instrumentId": "1INCH",
            "total": "100.00000000",
            "available": "100.00000000",
            "reserved": "0.00000000",
            "quantityLastUpdated": "1646413705113"
        },
        {
            "instrumentId": "AAVE",
            "total": "100.00000000",
            "available": "100.00000000",
            "reserved": "0.00000000",
            "quantityLastUpdated": "1646413705827"
        }
    ]
}
```

Returns all the coin balances of the account connected to the API key initiating the request. 

<sub>**Response Fields**</sub> 

Field | Type | Description | 
----- | ---- | ----------- |
event | STRING | `balances`
timestamp | STRING | Millisecond timestamp
accountId | STRING | Account ID
tradeType | STRING | `LINEAR` |
data | LIST of dictionaries |
instrumentId | STRING |Coin symbol, e.g. 'BTC' |
total| STRING| Total balance|
available |STRING| Available balance|
reserved|STRING|Reserved balance (unavailable) due to working spot orders|
quantityLastUpdated|STRING|Millisecond timestamp of when balance was last updated|


### GET `/v2/balances/{instrumentId}`

>**Request**

```json
GET /v2/balances/{instrumentId}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/balances/{instrumentId}'

params = "limit=3"

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
  "event": "balancesById",
  "timestamp": 1593627415293,
  "accountId": "<Your Account ID>",
  "tradeType": "LINEAR",
  "data": [ {   
              "instrumentId": "FLEX",
              "total": "4468.823",              
              "available": "4468.823",        
              "reserved": "0",
              "quantityLastUpdated": "1593627415001"
            } ]
}
```

```python
{
    "event": "balancesById",
    "timestamp": "1648813214792",
    "accountId": "677473",
    "tradeType": "LINEAR",
    "data": {
        "instrumentId": "FLEX",
        "total": "695180.143917630",
        "available": "695180.143917630",
        "reserved": "0",
        "quantityLastUpdated": "1648809926359"
    }
}
```

Returns the specified coin balance of the account connected to the API key initiating the request. 


<sub>**Request Paramters**</sub> 

Parameter | Type | Required | Description |
--------- | ---- | -------- | ----------- |
instrumentId | STRING | YES | Please place it in URL |

<sub>**Response Fields**</sub> 

Field | Type | Description | 
------| ---- | ----------- |
event | STRING | `balancesById`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING | Account ID
tradeType | STRING | `LINEAR` |
data | LIST of dictionary | |
instrumentId | STRING | Coin symbol, e.g. 'FLEX' |
total| STRING | Total balance |
available | STRING | Available balance |
reserved | STRING | Reserved balance (unavailable) due to working spot orders |
quantityLastUpdated | STRING | Timestamp when was balance last updated |


###GET  `/v2/positions`

> **Request**

```json
GET /v2/positions
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/positions'

params = "limit=3"

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```


> **Response**

```json
{
  "event": "positions",
  "timestamp": 1593627415000,
  "accountId":"<Your Account ID>",
  "data": [ {
              "instrumentId": "BTC-USDT-SWAP-LIN",
              "quantity": "0.542000000",
              "lastUpdated": "1617099855966",
              "contractValCurrency": "BTC",
              "entryPrice": "56934.8258",
              "positionPnl": "1212.0065",
              "estLiquidationPrice": "52454.3592",
              "marginBalance": "1000.32",
              "maintenanceMargin": "300.32",
              "marginRatio": "0.4",
              "leverage": "2"
            },
            ...
          ]
}
```

```python
{
    "event": "positions", 
    "timestamp": "1648813006698", 
    "accountId": "677473", 
    "data": [
        {
            "instrumentId": "ETH-USDT-SWAP-LIN", 
            "quantity": "-461.64", 
            "lastUpdated": "1648518794680", 
            "contractValCurrency": "ETH", 
            "entryPrice": "3405.16", 
            "positionPnl": "51675.9816", 
            "estLiquidationPrice": "64473.25"，
        }, 
        {
            "instrumentId": "FLEX-USDT-SWAP-LIN", 
            "quantity": "38742.4", 
            "lastUpdated": "1648174875502", 
            "contractValCurrency": "FLEX", 
            "entryPrice": "7.989", 
            "positionPnl": "-387.4240", 
            "estLiquidationPrice": "0",
        }, 
        {
            "instrumentId": "BTC-USDT-SWAP-LIN", 
            "quantity": "65.889", 
            "lastUpdated": "1648806900492", 
            "contractValCurrency": "BTC", 
            "entryPrice": "47642", 
            "positionPnl": "-441817.260", 
            "estLiquidationPrice": "0"
            "marginBalance": "1000.32",
            "maintenanceMargin": "300.32",
            "marginRatio": "0.4",
            "leverage": "2"
        }
    ]
}
```

Returns all the positions of the account connected to the API key initiating the request. 

<aside class="notice">
Returns an empty array `[]` when no positions were found
</aside>

Response Fields | Type | Description |
--------------- | ---- | ----------- |
event | STRING | `positions` |
timestamp | INTEGER | Millisecond timestamp |
accountId | STRING | Account ID |
data | LIST of dictionaries | |
instrumentId | STRING | Contract symbol, e.g. 'BTC-USDT-SWAP-LIN' |
quantity | STRING | Quantity of position, e.g. '0.94' |
lastUpdated | STRING| Timestamp when position was last updated |
contractValCurrency | STRING | Contract valuation currency |
entryPrice | STRING | Average entry price |
positionPnl | STRING | Postion profit and lost |
estLiquidationPrice | STRING | Estimated liquidation price, return 0 if it is negative(<0) |
marginBalance | STRING |Appears in the position section only for positions using isolated margin. Isolated margin + Unrealized position PnL|
maintenanceMargin | STRING |Appears in the position section only for positions using isolated margin|
marginRatio | STRING | Appears in the position section only for positions using isolated margin|
leverage | STRING | Appears in the position section only for positions using isolated margin|


###GET  `/v2/positions/{instrumentId}`

> **Request**

```json
GET /v2/positions/{instrumentId}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/positions/{instrumentId}'

params = "limit=3"

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```


> **Response**

```json
{
  "event": "positionsById",
  "timestamp": 1593617005438,
  "accountId":"<Your Account ID>",
  "data": [ {
              "instrumentId": "BTC-USDT-SWAP-LIN",
              "quantity": "0.542000000",
              "lastUpdated": "1617099855966",
              "contractValCurrency": "BTC",
              "entryPrice": "56934.8258",
              "positionPnl": "1216.6135",
              "estLiquidationPrice": "53171.16",
              "marginBalance": "1000.32",
              "maintenanceMargin": "300.32",
              "marginRatio": "0.4",
              "leverage": "2"
          } ]
}
```

```python
{
    "event": "positionsById",
    "timestamp": "1648812534064",
    "accountId": "677473",
    "data": {
        "instrumentId": "FLEX-USDT-SWAP-LIN",
        "quantity": "38742.4",
        "lastUpdated": "1648174875502",
        "contractValCurrency": "FLEX",
        "entryPrice": "7.989",
        "positionPnl": "-387.4240",
        "estLiquidationPrice": "0",
        "marginBalance": "1000.32",
        "maintenanceMargin": "300.32",
        "marginRatio": "0.4",
        "leverage": "2"
    }
}
```

Returns the specified instrument ID position of the account connected to the API key initiating the request.

<aside class="notice">
Returned an empty object `{}` when no positions were found
</aside>

Request Parameter | Type | Required | Description |
--------- | ---- | -------- | ----------- |
instrumentId | STRING | YES | Please place it in URL |

Response Field | Type | Description |
--------------- | ---- | ----------- |
event | STRING | `positionsById` |
timestamp | INTEGER | Millisecond timestamp |
accountId | STRING | Account ID |
data | LIST of dictionaries | |
instrumentId | STRING | Contract symbol, e.g. 'FLEX-USDT-SWAP-LIN' |
quantity | STRING | Quantity of position, e.g. '0.94' |
lastUpdated | STRING | Timestamp when position was last updated |
contractValCurrency | STRING | Contract valuation currency |
entryPrice | STRING | Average entry price |
positionPnl | STRING | Postion profit and lost |
estLiquidationPrice | STRING | Estimated liquidation price, return 0 if it is negative(<0) |
marginBalance | STRING |Appears in the position section only for positions using isolated margin. Isolated margin + Unrealized position PnL|
maintenanceMargin | STRING |Appears in the position section only for positions using isolated margin|
marginRatio | STRING | Appears in the position section only for positions using isolated margin|
leverage | STRING | Appears in the position section only for positions using isolated margin|


### GET `/v2/trades/{marketCode}`

> **Request**

```json
GET /v2/trades/{marketCode}?limit={limit}&startTime={startTime}&endTime={endTime}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/trades/{marketCode}'

params = "limit=3"

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
  "event": "trades", 
  "timestamp": 1595635101845, 
  "accountId": "<Your Account ID>", 
  "data": [ {
              "matchId": "160067484555913077", 
              "matchTimestamp": "1595514663626", 
              "marketCode": "FLEX-USDT", 
              "matchQuantity": "0.1", 
              "matchPrice": "0.065", 
              "total": "0.0065", 
              "orderMatchType": "TAKER", 
              "fees": "0.0096", 
              "feeInstrumentId": "FLEX", 
              "orderId": "160067484555913076", 
              "side": "SELL", 
              "clientOrderId": "123"
            },
            ...
          ]
}
```

```python
{
    "event": "trades",
    "timestamp": "1648812102411",
    "accountId": "677473",
    "data": [
        {
            "matchId": "448821476099870304",
            "matchTimestamp": "1648809926245",
            "marketCode": "FLEX-USDT",
            "matchQuantity": "1",
            "matchPrice": "10",
            "total": "10",
            "orderMatchType": "MAKER",
            "fees": "0",
            "feeInstrumentId": null,
            "orderId": "1000200608142",
            "side": "SELL",
            "clientOrderId": "1612249737434"
        },
        {
            "matchId": "448821476099870303",
            "matchTimestamp": "1648809926245",
            "marketCode": "FLEX-USDT",
            "matchQuantity": "1",
            "matchPrice": "10",
            "total": "10",
            "orderMatchType": "MAKER",
            "fees": "0",
            "feeInstrumentId": null,
            "orderId": "1000200608140",
            "side": "SELL",
            "clientOrderId": "1612249737434"
        },
        {
            "matchId": "448821476099861874",
            "matchTimestamp": "1648807731185",
            "marketCode": "FLEX-USDT",
            "matchQuantity": "1",
            "matchPrice": "10",
            "total": "10",
            "orderMatchType": "MAKER",
            "fees": "0",
            "feeInstrumentId": null,
            "orderId": "1000200596577",
            "side": "SELL",
            "clientOrderId": "1612249737434"
        }
    ]
}
```

Returns the most recent trades of the account connected to the API key initiating the request.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description | 
---------- | ---- | -------- | ----------- |
marketCode| STRING | YES | Please place it in URL | 
limit| LONG | NO | Default `500`, max `1000` | 
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

<sub>**Response Fields**</sub> 

Field | Type | Description | 
------| ---- | ----------- |
event | STRING | `trades`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
data | LIST of dictionaries |
matchId   | STRING    | Match ID          |
matchTimestamp   | STRING    | Order Matched timestamp          |
marketCode   | STRING    | Market code          |
matchQuantity   | STRING    | Match quantity          |
matchPrice   | STRING    | Match price          |
total   | STRING    | Total price          |
side   | STRING    |  Side of the match         |
orderMatchType   | STRING    | `TAKER` or `MAKER` |
fees   | STRING    |  Fees    |
feeInstrumentId   | STRING    |   Instrument ID of the fees        |
orderId   | STRING    |	Unique order ID from the exchange          |
clientOrderID   | STRING    | Client assigned ID to help manage and identify orders  |


### GET `/v2/orders`

> **Request**

```json
GET /v2/orders
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/orders'

params = ""

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
  "event": "orders",
  "timestamp": "1593617005438",
  "accountId": "<Your Account ID>",
  "data": [ {
              "orderId": "160039151345856176",
              "marketCode": "BTC-USDT-SWAP-LIN",
              "clientOrderId": null|"<clientOrderId>",              
              "side": "BUY",
              "orderType": "LIMIT"|"STOP",
              "quantity": "1.00",
              "remainingQuantity": "1.00",
              "price": "1.00"|null,               #for limit order, null for stop order
              "stopPrice": "<stopPrice>"|null,    #for stop order, null for limit order 
              "limitPrice": "<limitPrice>"|null,  #for stop order, null for limit order 
              "orderCreated": 1593617008698,
              "lastModified": 1593617008698,
              "lastTradeTimestamp": 1593617008698,
              "timeInForce": "GTC"
            },
            ...
          ]
}
```

```python
{
    "event": "orders", 
    "timestamp": "1648809819657", 
    "accountId": "677473", 
    "data": [
        {
            "orderId": "1000200608142", 
            "marketCode": "FLEX-USDT", 
            "clientOrderId": "1612249737434", 
            "side": "SELL", 
            "orderType": "LIMIT", 
            "quantity": "1.0", 
            "remainingQuantity": "1.0", 
            "price": "10.0", 
            "stopPrice": null, 
            "limitPrice": "10.0", 
            "orderCreated": 1648809816385, 
            "lastModified": 1648809816591, 
            "lastTradeTimestamp": 1648809816557, 
            "timeInForce": "GTC"
        }, 
        {
            "orderId": "1000200608140", 
            "marketCode": "FLEX-USDT", 
            "clientOrderId": "1612249737434", 
            "side": "SELL", 
            "orderType": "LIMIT", 
            "quantity": "1.0", 
            "remainingQuantity": "1.0", 
            "price": "10.0", 
            "stopPrice": null, 
            "limitPrice": "10.0", 
            "orderCreated": 1648809812567, 
            "lastModified": 1648809812773, 
            "lastTradeTimestamp": 1648809812680, 
            "timeInForce": "GTC"
        }
    ]
}
```

Returns all the open orders of the account connected to the API key initiating the request.

<sub>**Response Fields**</sub>

Fields | Type | Description |
-------| ---- | ----------- |
event | STRING | `orders`
timestamp | STRING | Millisecond timestamp
accountId | STRING | Account ID
data | LIST of dictionaries |
orderId | STRING | Unique order ID from the exchange |
marketCode| STRING | Market code |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
side | STRING | `BUY` or `SELL` |
orderType | STRING | `LIMIT` or `STOP` |
quantity  | STRING | Quantity submitted |
remainingQuantity|STRING | Remainning quantity |
price | STRING | Price submitted |
stopPrice | STRING | Stop price for the stop order |
limitPrice| STRING | Limit price for the stop limit order |
orderCreated| INTEGER | Timestamp when order was created |
lastModified| INTEGER | Timestamp when order was last mordified |
lastTradeTimestamp| INTEGER | Timestamp when order was last traded |
timeInForce | STRING | Time in force |


### GET `/v2.1/orders`

> **Request**

```json
GET /v2.1/orders?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}&limit={limit}&startTime={startTime}&endTime={endTime}
```
```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_key'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2.1/orders'

params = "marketCode=FLEX-USDT&limit=1"

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
    "event": "orders",
    "timestamp": "1619167719563",
    "accountId": "1076",
    "data": [
        {
            "status": "OrderClosed",
            "orderId": "304408197314577142",
            "clientOrderId": "1",
            "marketCode": "BTC-USDT-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "price": "10006.0",
            "quantity": "0.001",
            "remainQuantity": "0.001",
            "timeInForce": "GTC",
            "orderClosedTimestamp": "1619131050779"
        },
        {
            "status": "OrderOpened",
            "orderId": "304408197314577143",
            "clientOrderId": "2",
            "marketCode": "BTC-USDT-SWAP-LIN",
            "side": "SELL",
            "orderType": "LIMIT",
            "price": "60006.0",
            "quantity": "0.001",
            "remainQuantity": "0.001",
            "timeInForce": "GTC",
            "orderOpenedTimestamp": "1619131049574"
        },
        {
            "status": "OrderMatched",
            "orderId": "448528458527567629",
            "clientOrderId": "1618870087524",
            "marketCode": "FLEX-USDT-SWAP-LIN",
            "side": "BUY",
            "orderType": "MARKET",
            "price": "0.194",
            "lastTradedPrice": "0.170",
            "avgFillPrice": "0.170",
            "quantity": "12.1",
            "filledQuantity": "12.1",
            "remainQuantity": "0",
            "matchIds": [
                {
                    "448528458527567630": {
                        "matchQuantity": "12.1",
                        "matchPrice": "0.170",
                        "timestamp": "1618870088471",
                        "orderMatchType": "TAKER"
                    }
                }
            ],
            "fees": {
                "FLEX": "-0.00440786"
            },
            "timeInForce": "IOC",
            "isTriggered": "false"
        },
        {
            "status": "OrderPartiallyMatched",
            "orderId": "1000028616860",
            "clientOrderId": "1619090944648",
            "marketCode": "BTC-USDT-SWAP-LIN",
            "side": "SELL",
            "orderType": "MARKET",
            "price": "42970.5",
            "lastTradedPrice": "43026.0",
            "avgFillPrice": "43026.0",
            "quantity": "0.112",
            "filledQuantity": "0.002",
            "remainQuantity": "0.11",
            "matchIds": [
                {
                    "304515168532122149": {
                        "matchQuantity": "0.001",
                        "matchPrice": "43026.0",
                        "timestamp": "1628824216325",
                        "orderMatchType": "TAKER"
                    }
                },
                {
                    "304515168532122150": {
                        "matchQuantity": "0.001",
                        "matchPrice": "43026.0",
                        "timestamp": "1628824216326",
                        "orderMatchType": "TAKER"
                    }
                }
            ]
        }
        ...
    ]
}
```
```python
{
    "event": "orders",
    "timestamp": "1648809055324",
    "accountId": "677473",
    "data": [
        {
            "status": "OrderMatched",
            "orderId": "1000200596577",
            "clientOrderId": "1612249737434",
            "marketCode": "FLEX-USDT",
            "side": "SELL",
            "orderType": "LIMIT",
            "price": "10.000",
            "lastTradedPrice": "10.000",
            "avgFillPrice": "10.000",
            "quantity": "1",
            "filledQuantity": "1",
            "remainQuantity": "0",
            "matchIds": [
                {
                    "448821476099861874": {
                        "matchQuantity": "1",
                        "matchPrice": "10.000",
                        "timestamp": "1648807731185",
                        "orderMatchType": "MAKER"
                    }
                }
            ],
            "timeInForce": "GTC",
            "isTriggered": "false"
        }
    ]
}
```

Returns all orders of the account connected to the API key initiating the request.

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |
orderId | LONG | NO | Client assigned ID to help manage and identify orders |
clientOrderId | ULONG | NO | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
limit | LONG | NO | max `100`, default `100` |
startTime | LONG | NO | e.g. `1579450778000`, default 24 hours ago, the range between startTime and endTime should be less than or equal to 7 days(`endTime - startTime <= 7days`) |
endTime | LONG | NO | e.g. `1613978625000`, default time now, the range between startTime and endTime should be less than or equal to 7 days(`endTime - startTime <= 7days`) |

<aside class="notice">
OrderClosed orders are not in the orderbook, the remainQuantity in the OrderClosed orders is historical remaining quantity
</aside>

Response Fields | Type | Description |
--------------- | ---- | ----------- |
accountId | STRING | Account ID |
timestamp | STRING | Timestamp of this response |
status | STRING | Status of the order, available values: `OrderOpened`, `OrderPartiallyMatched`, `OrderMatched`, `OrderClosed` |
orderId | STRING | Order ID which generated by the server |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Market code |
side | STRING | Side of the order, `BUY` or `SELL` |
orderType | STRING | Type of the order, `LIMIT` or `STOP` |
price | STRING | Price submitted |
lastTradedPrice | STRING | Price when order was last traded |
avgFillPrice | STRING | Average of filled price |
stopPrice | STRING | Stop price for the stop order |
limitPrice | STRING | Limit price for the stop limit order |
quantity | STRING | Quantity submitted |
remainQuantity | STRING | Remaining quantity |
filledQuantity | STRING | Filled quantity |
matchIds | LIST of dictionaries | Exchange matched IDs and information about matching orders |
matchQuantity | STRING | Matched quantity |
matchPrice | STRING | Matched price |
orderMatchType | STRING | `MAKER` or `TAKER` |
timestamp in matchIds | STRING | Time matched at|
leg1Price | STRING | |
leg2Price | STRING | |
fees | LIST of dictionaries | Overall fees with instrument ID, if FLEX is no enough to pay the fee then USDT will be paid |
timeInForce | STRING | Time in force |
isTriggered | STRING | `true`(for stop order) or `false` |
orderOpenedTimestamp | STRING | Order opened at |
orderModifiedTimestamp | STRING | Order modified at |
orderClosedTimestamp | STRING | Order closed at |



###DELETE `/v2/cancel/orders`

> **Request**

```json
DELETE /v2/cancel/orders 
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/cancel/orders'

params = json.dumps({})

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'DELETE', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.delete(rest_url + method, headers=header, data=params)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
  "event": "orders",
  "timestamp": 1594412077100,
  "accountId": "<AccountID>",
  "data": {
    "msg": "All open orders for the account have been queued for cancellation"
  }
}
```

```python
{
    "event": "orders",
    "timestamp": "1648809641539",
    "accountId": "677473",
    "data": {
        "msg": "All open orders for the account have been queued for cancellation"
    }
}
```

Cancels **all** open orders of the account connected to the API key initiating the request.

If this REST method was sucessful it will also trigger a reponse message in an authenticated websocket of the account.  This is documented here [Cancel Open Orders](#websocket-api-other-responses-cancel-open-orders).

<sub>**Response Parameters**</sub> 

Parameters | Type | Description | 
-----------| ---- | ----------- |
event | STRING | `orders` |
timestamp | INTEGER | Millisecond timestamp |
accountId | STRING | Account ID |
data | Dictionary | |
msg  | STRING | Confirmation of action |


###DELETE `/v2/cancel/orders/{marketCode}`

> **Request**

```json
DELETE /v2/cancel/orders/{marketCode}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/cancel/orders/{marketCode}'

params = json.dumps({})

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'DELETE', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.delete(rest_url + method, headers=header, data=params)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
  "event": "orders",
  "timestamp": 1594412077100,
  "accountId": "<AccountID>",
  "data": {
            "marketCode": "FLEX-USDT",
            "msg": "All open orders for the specified market have been queued for cancellation"
          }
}
```

```python
{
    "event": "orders",
    "timestamp": "1648807731741",
    "accountId": "3101",
    "data": {
        "marketCode": "BTC-USDT-SWAP-LIN",
        "msg": "All open orders for the specified market have been queued for cancellation"
    }
}
```

Cancels all open orders for the **specified market** for the account connected to the API key initiating the request.

If this REST method was sucessful it will also trigger a reponse message in an authenticated websocket of the account.  This is documented here [Cancel Open Orders](#websocket-api-other-responses-cancel-open-orders).

Request Parameter | Type | Required | Description |
--------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Please place it in URL |

Response Fields | Type | Description | 
--------------- | ---- | ----------- |
event | STRING | `orders`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING | Account ID
data | Dictionary |
marketCode | STRING | Market code |
msg | STRING | Confirmation of action |


###GET `/v2.1/delivery/orders`

> **Request**

```json
GET /v2.1/delivery/orders?limit={limit}&startTime={startTime}&endTime={endTime}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2.1/delivery/orders'

params = "limit=1"

if params:
    path = method + '?' + params
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **Response**

```json
{
  "event": "deliverOrders",
  "timestamp": "1596685339910",
  "data": [ {
              "timestamp": "1595781719394",
              "instrumentId": "BTC-USDT-SWAP-LIN",
              "status": "DELIVERED",
              "quantity": null,
              "deliverPrice": "9938.480000000",
              "transferAsset": "USDT",
              "transferQty": "993.848000000",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "0.100000000",
              "deliverOrderId": "575770851486007299",
              "clientOrderId": null
            },
            {
              "timestamp": "1595786511155",
              "instrumentId": "BTC-USDT-SWAP-LIN",
              "status": "CANCELLED",
              "quantity": null,
              "deliverPrice": "9911.470000000",
              "transferAsset": "USDT",
              "transferQty": "0.000000000",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "0.000000000",
              "deliverOrderId": "575786553086246913",
              "clientOrderId": null
            },
          ]
}
```

```python
{
    "event": "deliverOrders",
    "timestamp": "1648806915444",
    "data": [
        {
            "timestamp": "1648806900463",
            "instrumentId": "BTC-USDT-SWAP-LIN",
            "status": "PENDING",
            "quantity": "0.100000000",
            "deliverPrice": "45131.000000000",
            "transferAsset": "USDT",
            "transferQty": "4513.100000000",
            "instrumentIdDeliver": "BTC",
            "deliverQty": "0.000000000",
            "deliverOrderId": "749523764730920963",
            "clientOrderId": null
        }
    ]
}
```

Returns the entire delivery history for the account connected to the API key initiating the request.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description |
---------- | ---- | -------- | ----------- |
limit | LONG | NO | Default `200`, max `500` |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

<sub>**Response Parameters**</sub> 

Parameters | Type | Description |
---------- | ---- | ----------- |
event | STRING | `deliverOrders`
timestamp | STRING | Millisecond timestamp of the repsonse
data | LIST of dictionaries |
timestamp | STRING | Millisecond timestamp of the delivery action
instrumentId | STRING | Perpetual swap market code
status | STRING | Request status
quantity | Null Type | Quantity
deliverPrice | STRING|  Mark price at delivery
transferAsset | STRING | Asset being sent
transferQty | STRING | Quantity being sent
instrumentIdDeliver | STRING |Asset being received: long position = coin, short position = USDT
deliverQty | STRING |  Quantity of the received asset
deliverOrderId | STRING | Order id
clientOrderId | Null Type |  null

### POST `/v2/orders/place`

> **Request**

```json
POST /v2/orders/place

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
        }, 
        {
            "clientOrderId": 1612249737723, 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "side": "SELL", 
            "quantity": "0.003", 
            "timeInForce": "GTC", 
            "orderType": "LIMIT", 
            "price": "54901"
        }
    ]
}
```
```python
import requests
import hmac
import base64
import hashlib
import datetime
import json

rest_url = 'https://stgapi.opnx.com'
rest_path = 'v2stgapi.opnx.com'

api_key = 'api_key'
api_secret = 'api_secret'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

method = '/v2/orders/place'

params = json.dumps({"recvWindow":3000, "timestamp": 1737100050453, "responseType":"FULL","orders":[{"clientOrderId":1612249737434,"marketCode":"BTC-USDT-SWAP-LIN","side":"SELL","quantity":"1","timeInForce":"GTC","orderType":"LIMIT","price":"49995"}]})

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'POST', rest_path, method, params)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.post(rest_url + method, headers=header, data=params)
print(resp.url)
print(resp.request.headers)
print(resp.request.body)
print(json.dumps(resp.json(), indent=4, separators=(', ', ': ')))
```

> **RESPONSE**

```json
{
    "accountId": "13670827", 
    "event": "placeOrder", 
    "timestamp": "1615430915625", 
    "data": [
        {
            "success": "false",
            "timestamp": "1641538345068",
            "code": "710006",
            "message": "FAILED balance check as balance (0E-9) < value (0.001)",
            "clientOrderId": "2619090944648",
            "price": "52888.0",
            "quantity": "0.001",
            "side": "SELL",
            "marketCode": "BTC-USDT",
            "timeInForce": "GTC",
            "orderType": "LIMIT"
        },
        {
            "success": "true",
            "timestamp": "1641536720611",
            "clientOrderId": "3619090894340",
            "orderId": "1000132664173",
            "price": "23641.0",
            "quantity": "0.7",
            "side": "BUY",
            "status": "OPEN",
            "marketCode": "BTC-USDT-SWAP-LIN",
            "timeInForce": "GTC",
            "matchId": "0",
            "notice": "OrderOpened",
            "orderType": "LIMIT",
            "isTriggered": "false"
        },
        {
            "success": "true",
            "timestamp": "1641538343028",
            "clientOrderId": "3619090894340",
            "orderId": "1000132688133",
            "price": "43000.0",
            "quantity": "0.1",
            "side": "BUY",
            "status": "PARTIAL_FILL",
            "marketCode": "BTC-USDT-SWAP-LIN",
            "timeInForce": "GTC",
            "matchId": "304638880616112239",
            "lastTradedPrice": "42731.5",
            "matchQuantity": "0.001",
            "orderMatchType": "TAKER",
            "remainQuantity": "0.099",
            "notice": "OrderMatched",
            "orderType": "LIMIT",
            "fees": "0.00170064",
            "feeInstrumentId": "FLEX",
            "isTriggered": "false"
        }
    ]
}
```
```python
{
    "event": "placeOrder", 
    "timestamp": "1648804490345", 
    "accountId": "677473", 
    "data": [
        {
            "success": "true", 
            "timestamp": "1648804490326", 
            "clientOrderId": "1612249737434", 
            "orderId": "1000200584643", 
            "price": "49995.0", 
            "quantity": "1.0", 
            "side": "SELL", 
            "status": "OPEN", 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "timeInForce": "GTC", 
            "matchId": "0", 
            "notice": "OrderOpened", 
            "orderType": "LIMIT", 
            "isTriggered": "false"
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
timeInForce | STRING | NO | Default `GTC` |
orderType | STRING | YES | `LIMIT` or `MARKET` or `STOP` |
price | STRING | NO | Limit price for the limit order |
stopPrice | STRING | NO | Stop price for the stop order |
limitPrice | STRING | NO | Limit price for the stop limit order |

Response Fields | Type | Description | 
--------------------| ---- | ----------- |
accountId | STRING | Account ID |
event | STRING | |
timestamp | STRING | Millisecond timestamp of the repsonse |
data | LIST | |
success | STRING | Whether an order has been successfully placed |
timestamp | STRING | |
code | STRING | Error code |
message | STRING | Error message |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
orderId | STRING | |
price | STRING | |
quantity | STRING | |
side | STRING | `SELL` or `BUY` |
marketCode | STRING | |
timeInForce | STRING | |
matchId | STRING | Exchange match ID |
lastTradedPrice | STRING | Price when order was last traded |
matchQuantity | STRING | Matched quantity |
orderMatchType | STRING | `MAKER` or `TAKER` |
remainQuantity | STRING | Remainning quantity |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
orderType | STRING | `MARKET` or `LIMIT` or `STOP` |
fees | STRING | Amount of fees paid from this match ID |
feeInstrumentId | STRING | Instrument ID of fees paid from this match ID |
isTriggered | STRING | false (or true for STOP order types) |


### POST `/v2/orders/modify`
> **Request**

```json
POST /v2/orders/modify

{
    "recvWindow": 13000, 
    "responseType": "FULL", 
    "timestamp": 1614331650143, 
    "orders": [
        {
            "clientOrderId": 1614330009059, 
            "orderId": "304369975621712260", 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "side": "BUY", 
            "quantity": "0.007", 
            "price": "40001.0"
        }, 
        {
            "clientOrderId": 161224973777800, 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "side": "SELL", 
            "quantity": "0.002", 
            "price": "40003.0"
        }, 
        {
            "clientOrderId": 161224973777900, 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "side": "SELL", 
            "quantity": "0.003", 
            "price": "40004.0"
        }
    ]
}
```

> **RESPONSE**

```json
{
    "accountId": "495", 
    "event": "modifyOrder", 
    "timestamp": "1614331651243", 
    "data": [
        {
            "success": "false", 
            "timestamp": "1614331651174", 
            "code": "40032", 
            "message": "Invalid request data", 
            "clientOrderId": "161224973777800", 
            "price": "40003.0", 
            "quantity": "0.002", 
            "side": "SELL", 
            "marketCode": "BTC-USDT-SWAP-LIN"
        }, 
        {
            "success": "false", 
            "timestamp": "1614331651174", 
            "code": "40032", 
            "message": "Invalid request data", 
            "clientOrderId": "161224973777900", 
            "price": "40004.0", 
            "quantity": "0.003", 
            "side": "SELL", 
            "marketCode": "BTC-USDT-SWAP-LIN"
        }, 
        {
            "success": "true", 
            "timestamp": "1614331651196", 
            "clientOrderId": "1614330009059", 
            "orderId": "304369975621712263", 
            "price": "40001.0", 
            "quantity": "0.007", 
            "side": "BUY", 
            "status": "OPEN", 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "timeInForce": "GTC", 
            "notice": "OrderOpened", 
            "orderType": "LIMIT", 
            "isTriggered": "false"
        }
    ]
}
```

Modify orders.

<aside class="notice">
You can modify up to 8 orders at a time in REST API
</aside>

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | STRING | NO | |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | |
clientOrderId | ULONG | NO | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
orderId | STRING | YES | |
marketCode | STRING | YES | |
side | STRING | NO | |
quantity | STRING | NO | |
price | STRING | NO | |
stopPrice | STRING | NO | |
limitPrice | STRING | NO | |

Response Parameters | Type | Description | 
--------------------| ---- | ----------- |
accountId | STRING | |
event | STRING | |
timestamp | STRING | |
data | LIST | |
success | STRING | |
timestamp | STRING | |
code | STRING | Error code |
message | STRING | Error message |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
orderId | STRING | |
price | STRING | |
quantity | STRING | |
side | STRING | `SELL` or `BUY` |
status | STRING | Status of the order |
marketCode | STRING | |
timeInForce | STRING | |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
orderType | STRING | |
isTriggered | STRING | `true` or `false` |


### DELETE `/v2/orders/cancel`

> **Request**

```json
DELETE /v2/orders/cancel

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

> **RESPONSE**

```json
{
    "accountId": "495", 
    "event": "cancelOrder", 
    "timestamp": "1615454881391", 
    "data": [
        {
            "success": "true", 
            "timestamp": "1615454881383", 
            "clientOrderId": "1615453494726", 
            "orderId": "304384250571714215", 
            "price": "55006.0", 
            "quantity": "0.006", 
            "side": "SELL", 
            "status": "CANCELED_BY_USER", 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "timeInForce": "GTC", 
            "remainQuantity": "0.006", 
            "notice": "OrderClosed", 
            "orderType": "LIMIT", 
            "isTriggered": "false"
        }, 
        {
            "success": "false", 
            "timestamp": "1615454881433", 
            "code": "40035", 
            "message": "Open order not found with id", 
            "clientOrderId": "1612249737724", 
            "marketCode": "BTC-USDT-SWAP-LIN"
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

Response Parameters | Type | Description | 
--------------------| ---- | ----------- |
accountId | STRING | |
event | STRING | |
timestamp | STRING | |
data | LIST | |
success | STRING | |
timestamp | STRING | |
code | STRING | Error code |
message | STRING | Error message |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
orderId | STRING | |
price | STRING | |
quantity | STRING | |
side | STRING | `SELL` or `BUY` |
status | STRING | Status of the order |
marketCode | STRING | |
timeInForce | STRING | |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
orderType | STRING | |
isTriggered | STRING | `true` or `false` |


### POST `/v2/mint`

Mint.

> **Request**

```json
POST /v2/mint

{

    "asset": "flexUSD",
    "quantity": 1000

}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event":"mint",
    "timestamp":"1620964317199",
    "accountId":"1532",
    "data":{
        "asset":"flexUSD",
        "quantity":"10"
    }
}
```

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING/DECIMAL | YES | Quantity of the asset |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |


### GET `/v2/mint/{asset}`

Get mint history by asset and sorted by time in descending order.

> **Request**

```json
GET /v2/mint/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event":"mintHistory",
    "timestamp":"1620964764692",
    "accountId":"1570",
    "data":[
        {
            "asset":"flexETH",
            "quantity":"0.100000000",
            "mintedAt":"1619779905495"
        },
        {
            "asset":"flexETH",
            "quantity":"97.800000000",
            "mintedAt":"1619779812468"
        },
        {
            "asset":"flexETH",
            "quantity":"0.100000000",
            "mintedAt":"1619779696705"
        },
        ...
    ]
}
```

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
limit | STRING | NO | max `100`, default `100`|
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
mintedAt | STRING | Minted time in millisecond timestamp |


### POST `/v2/redeem`

Redeem.

> **Request**

```json
POST /v2/redeem

{

    "asset": "flexUSD",
    "quantity": 1000,
    "type": "Normal"

}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event":"redeem",
    "timestamp":"1620964351508",
    "accountId":"1532",
    "data":{
        "asset":"flexUSD",
        "quantity":"10",
        "redeemAt":"1620964800000",
        "type":"NORMAL"
    }
}
```

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING/DECIMAL | YES | Quantity of the asset |
type | STRING | YES | Redeem type, available types: `Normal`, `Instant` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
redeemAt | STRING | Redeemed time |
type | STRING | Redeem type, available types: `Normal`, `Instant` |


### GET `/v2/redeem/{asset}`

Get redemption history by asset and sorted by time in descending order.

> **Request**

```json
GET /v2/redeem/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
  "event":"redemptionHistory",
  "timestamp":"1620964856842",
  "accountId":"1570",
  "data":[
    {
      "asset":"ETH",
      "quantity":"0.001000000",
      "requestedAt":"1619788358578",
      "redeemedAt":"1619788860219"
    },
    {
      "asset":"ETH",
      "quantity":"0.001000000",
      "requestedAt":"1619788328760",
      "redeemedAt":"1619788328963"
    },
    ...
  ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
limit | STRING | NO | max `100`, default `100`|
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
requestedAt | STRING | when the redeem request was made |
redeemedAt | STRING | when the redemption was actually processed |


### GET `v2/borrow/{asset}`

Get borrow history by asset and sorted by time in descending order.

> **Request**

```json
GET v2/borrow/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "borrowHistory",
    "timestamp": "1626433904976",
    "accountId": "3123",
    "data": [
        {
            "borrowAsset": "USDT",
            "borrowedAmount": "296.2158828",
            "collateralAsset": "BTC",
            "collateralizedAmount": "0.01000000",
            "nonCollateralizedAmount": "0",
            "rateType": "FLOATING_RATE",
            "status": "COMPLETED",
            "borrowedAt": "1626433827613"
        },
        {
            "borrowAsset": "USDT",
            "borrowedAmount": "0.0000000",
            "collateralAsset": "BTC",
            "collateralizedAmount": "0.00000000",
            "nonCollateralizedAmount": "1",
            "rateType": "FLOATING_RATE",
            "status": "CANCELED",
            "borrowedAt": "1626432797124"
        },
        ...
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Collateral asset name |
limit | STRING | NO | max `100`, default `100`|
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |


Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
borrowAsset | STRING | Borrow asset, can only be `USDT` for now |
borrowedAmount | STRING | Borrowed amount of the borrow asset |
collateralAsset | STRING | Collateral asset |
collateralizedAmount | STRING | Collateralized amount of the collateral asset |
nonCollateralizedAmount | STRING | `nonCollateralizedAmount` = `collateralAmount` - `collateralizedAmount` |
rateType | STRING | `FLOATING_RATE` or `FIXED_RATE` |
status | STRING | `COMPLETED` or `PARTIAL` or `CANCELLED` |
borrowedAt | STRING | The time of borrowed at |

### GET `v2/repay/{asset}`

Get repay history by asset and sorted by time in descending order.

> **Request**

```json
GET v2/repay/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "repayHistory",
    "timestamp": "1626433953585",
    "accountId": "3123",
    "data": [
        {
            "repayAsset": "USDT",
            "repaidAmount": "0",
            "regainAsset": "BTC",
            "regainedAmount": "0",
            "nonRegainedAmount": "0.001",
            "status": "CANCELLED",
            "repaidAt": "1626747403329"
        },
        {
            "repayAsset": "USDT",
            "repaidAmount": "31863.01677",
            "regainAsset": "BTC",
            "regainedAmount": "1",
            "nonRegainedAmount": "0",
            "status": "COMPLETED",
            "repaidAt": "1626676776330"
        },
        ...
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Regain asset name |
limit | STRING | NO | max `100`, default `100`|
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
repayAsset | STRING | Repay asset, can only be `USDT` for now |
repaidAmount | STRING | Repaid amount of the repay asset |
regainAsset | STRING | Regain asset |
regainedAmount | STRING | Already regained amount of the regain asset |
nonRegainedAmount | STRING | `nonRegainedAmount` = `regainAmount` - `regainedAmount` |
status | STRING | `COMPLETED` or `PARTIAL` or `CANCELLED` |
borrowedAt | STRING | The time of borrowed at |
repaidAt | STRING | The time of repaid at |


### GET `/v2/funding-payments`

Get funding payments by marketCode and sorted by time in descending order.

> **Request**

```json
GET v2/funding-payments?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | NO | e.g. `BTC-USDT-REPO-LIN` |
limit | LONG | NO | default is `500`, max is `500` |
startTime | LONG | NO | millisecond timestamp, e.g. `1579450778000`, default is `500 hours ago` |
endTime | LONG | NO | millisecond timestamp, e.g. `1613978625000`, default is time now |

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "fundingPayments",
    "timestamp": "1627626010074",
    "accountId": "276007",
    "data": [
        {
            "marketCode": "BTC-USDT-SWAP-LIN",
            "payment": "-122.17530872",
            "rate": "-0.00005",
            "position": "-61.093",
            "markPrice": "39996.5",
            "timestamp": "1627617632190"
        },
        {
            "marketCode": "BTC-USDT-SWAP-LIN",
            "payment": "98.71895684",
            "rate": "0.00005",
            "position": "-61.093",
            "markPrice": "32317.6",
            "timestamp": "1627041622046"
        },
        ...
    ]
}
```

Response Fields | Type | Description |
------------------- | ---- | ----------- |
timestamp | STRING | Timestamp of this response |
marketCode | STRING | Market code |
payment | STRING | Funding payment |
rate | STRING | Funding rate |
position | STRING | Position |
markPrice | STRING | Mark price |
timestamp(in the data list) | STRING | Updated time |


### POST `/v2/AMM/create`

Create AMM

> **Request**

```json
POST /v2/AMM/create

{
    "direction": "NEUTRAL",
    "marketCode": "BTC-USDT-SWAP-LIN",
    "collateralAsset":"BTC",
    "assetQuantity":"0.1",
    "collateralCounterAsset":"USDT",
    "counterAssetQuantity":"4000",
    "minPriceBound":"28000",
    "maxPriceBound":"52000"
}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "createAMM",
    "timestamp": "1628491099751",
    "accountId": "3101",
    "data": {
        "hashToken": "CF-BTC-AMM-mz7WAOZc",
        "direction": "NEUTRAL",
        "marketCode": "BTC-USDT-SWAP-LIN",
        "collateralAsset": "BTC",
        "assetQuantity": "0.1",
        "collateralCounterAsset": "USDT",
        "counterAssetQuantity": "4000",
        "minPriceBound": "28000",
        "maxPriceBound": "52000"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
leverage | STRING | NO | Maximum leverage, with USDT the range of leverage is 1 to 40X otherwise is 1 to 10X |
direction | STRING | YES | Value can be `BUY` or `SELL` or `NEUTRAL` |
marketCode | STRING | YES | Market code e.g. `BCH-USDT-SWAP-LIN` |
collateralAsset | STRING | NO | Required unless unleveraged and direction is `BUY` |
assetQuantity | STRING | NO | Required unless unleveraged and direction is `BUY`. Minimum is $200 notional |
collateralCounterAsset | STRING | NO | Required if unleveraged and direction is `NEUTRAL` or `BUY` |
counterAssetQuantity | STRING | NO | Required if unleveraged and direction is `NEUTRAL` or `BUY`. Minimum is $200 notional |
minPriceBound | STRING | YES | When unleveraged and direction is `NEUTRAL`, minPriceBound and maxPriceBound should be stuck to the formula `mid = counterAssetQuantity/collateralAssetQuantity` `maxPriceBound - mid = mid - minPriceBound` |
maxPriceBound | STRING | YES | When unleveraged and direction is `NEUTRAL`, minPriceBound and maxPriceBound should be stuck to the formula `mid = counterAssetQuantity/collateralAssetQuantity` `maxPriceBound - mid = mid - minPriceBound` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
hashToken | STRING | Identity of the AMM |
leverage | STRING | Leverage of the AMM |
direction | STRING | Value can be `BUY` or `SELL` or `NEUTRAL` |
marketCode | STRING | Market code e.g. `BCH-USDT-SWAP-LIN` |
collateralAsset | STRING | Collateral asset |
assetQuantity | STRING | Quantity of the collateral asset |
collateralCounterAsset | STRING | Collateral counter asset |
counterAssetQuantity | STRING | Quantity of the collateral counter asset |
minPriceBound | STRING | Minimum price of the range |
maxPriceBound | STRING | Maximum price of the range |


### POST `/v2/AMM/redeem`

Redeem AMM

> **Request**

```json
POST /v2/AMM/redeem

{
    "hashToken": "CF-BTC-AMM-WJRzxzb",
    "redeemType": "DELIVER"
}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "redeemAMM",
    "timestamp": "1628502282362",
    "accountId": "3101",
    "data": {
        "hashToken": "CF-BTC-AMM-mz7WAOZc",
        "redeemType": "DELIVER"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | Identity of the AMM |
redeemType | STRING | YES | Value can be `DELIVER` or `MANUAL` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
hashToken | STRING | Identity of the AMM |
redeemType | STRING | YES | Value can be `DELIVER` or `MANUAL` |


### GET `/v2/AMM`

Get AMMs.

> **Request**

```json
GET v2/AMM?hashToken=CF-BTC-AMM-RoBwokR&marketCode=BTC-USDT-SWAP-LIN
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "infoAMM",
    "timestamp": "1628508877030",
    "accountId": "3101",
    "data": [
        {
            "hashToken": "CF-BTC-AMM-RoBwokR",
            "direction": "NEUTRAL",
            "marketCode": "BTC-USDT-SWAP-LIN",
            "status": "EXECUTING",
            "collateralAsset": "BTC",
            "assetQuantity": "0.4",
            "collateralCounterAsset": "USDT",
            "counterAssetQuantity": "5000",
            "minPriceBound": "8750",
            "maxPriceBound": "16250",
            "assetBalance": "0",
            "counterAssetBalance": "19894.5719512",
            "position": "-0.433",
            "entryPrice": "37758.38",
            "usdEarned": "95",
            "flexReward": "675",
            "apr": "86.64",
            "createdAt": "1628066890178",
            "updatedAt": "1628476562875"
        }
    ]
}
```

Request Parameters | Type | Required |Description| 
-------------------------- | -----|--------- | -------------|
hashToken | STRING | NO | Multiple hashTokens can be separated by a comma |
marketCode| STRING | NO | Market code e.g. `BTC-USDT-SWAP-LIN` |
status | STRING | NO | Value can be `ENDED` or `EXECUTING` or `PENDING` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
hashToken | STRING | Identity of the AMM |
direction | STRING | Value can be `BUY` or `SELL` or `NEUTRAL` |
marketCode | STRING | Market code e.g. `BTC-USDT-SWAP-LIN` |
status | STRING | Value can be `ENDED` or `EXECUTING` or `PENDING` |
collateralAsset | STRING | Collateral asset |
assetQuantity | STRING | Quantity of the collateral asset |
collateralCounterAsset | STRING | Collateral counter asset |
counterAssetQuantity | STRING | Quantity of the collateral counter asset |
minPriceBound | STRING | Minimum price of the range |
maxPriceBound | STRING | Maximum price of the range |
position | STRING | Current position |
usdEarned | STRING | USDT already earned |
flexReward | STRING | Amount of FLEX reward | 
apr | STRING | APR(annual percentage rate) |
createdAt | STRING | The time that AMM was created at |
updatedAt | STRING | The time that AMM was updated at |


##Methods - Public

###GET `/v2/all/markets`


> **Request**

```json
GET/v2/all/markets`
```

> **RESPONSE**


```json
{
    "event": "markets",
    "timestamp": "1620810144786",
    "data": [
        {
            "marketId": "2001000000000",
            "marketCode": "BTC-USDT",
            "name": "BTC/USDT",
            "referencePair": "BTC/USDT",
            "base": "BTC",
            "counter": "USDT",
            "type": "SPOT",
            "tickSize": "0.1",
            "qtyIncrement": "0.001",
            "listingDate": 2208988800000,
            "endDate": 0,
            "marginCurrency": "USDT",
            "contractValCurrency": "BTC",
            "upperPriceBound": "11000.00",
            "lowerPriceBound": "9000.00",
            "marketPrice": "10000.00",
            "marketPriceLastUpdated": "1620810131131"
        }
    ]
}
```
Get a list of all available markets on Opnx.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
marketId | STRING | |
marketCode| STRING    | Market Code                  |
name      | STRING    | Name of the contract                  |
referencePair| STRING | Reference pair                  |
base      | STRING    | Base asset                  |
counter   | STRING    | Counter asset                  |
type      | STRING    | Type of the contract                  |
tickSize  | STRING    | Tick size of the contract                  |
qtyIncrement| STRING  |Minimum increamet quantity|
listingDate| STRING   | Listing date of the contract                  |
endDate    |STRING    | Ending date of the contract                  |
marginCurrency|STRING | Margining currency                  |
contractValCurrency| STRING| Contract valuation currency|
upperPriceBound| STRING| Upper price bound                 |
lowerPriceBound| STRING| Lower price bound                 |
marketPrice    | STRING| Market price                 |
marketPriceLastUpdated | LONG | The time that market price last updated at |


###GET `/v2/all/assets`

> **Request**

```json
GET/v2/all/assets
```

> **RESPONSE**

```json
{
    "event": "assets",
    "timestamp":"1593617008698",
    "data": [
        {
            "instrumentId": "BTC-USDT-200626-LIN",
            "name": "BTC/USDT 20-06-26 Future (Linear)",
            "base": "BTC",
            "counter": "USDT",
            "type": "FUTURE",
            "marginCurrency": "USDT",
            "contractValCurrency": "BTC",
            "deliveryDate": null,
            "deliveryInstrument": null
        },
        {
            "instrumentId": "BTC",
            "name": "Bitcoin",
            "base": null,
            "counter": null,
            "type": "SPOT",
            "marginCurrency": null,
            "contractValCurrency": null,
            "deliveryDate": null,
            "deliveryInstrument": null
        }
    ]
}
```

Get a list of all assets available on Opnx. These include coins and bookable contracts.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
instrumentId| STRING    | Instrument ID                   |
name      | STRING    | Name of the asset                  |
base      | STRING    | Base of the asset                  |
counter   | STRING    | Counter of the asset                  |
type      | STRING    | type of the asset                  |
marginCurrency| STRING | Margining currency                 |
contractValCurrency| STRING| Contract valuation currency              |
deliveryDate       | STRING| Delivery date             |
deliveryInstrument | STRING| Delivery instrument             |

###GET `/v2/publictrades/{marketCode}`

> **Request**

```json
GET/v2/publictrades/{marketCode}?limit={limit}&startTime={startTime}&endTime{endTime}
```

> **RESPONSE**


```json
{
  "event": "publicTrades", 
  "timestamp": "1595636619410", 
  "marketCode": "BTC-USDT-SWAP-LIN", 
  "data": [
    {
      "matchId": "160070803925856675", 
      "matchQuantity": "0.100000000", 
      "matchPrice": "9600.000000000", 
      "side": "BUY", 
      "matchTimestamp": "1595585860254"
      },
      ...
  ]
}
```
Get most recent trades.

Request Parameters | Type | Required |Description| 
-------------------------- | -----|--------- | -------------|
marketCode| STRING | YES |  | 
limit| LONG | NO | Default 100, max 300 | 
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
marketCode | STRING    | |
matchID | STRING    | |
matchQuantity | STRING    | |
matchPrice | STRING    | |
side | STRING    | |
matchTimestamp | STRING    | |


###GET `/v2/ticker`

> **Request**

```json
GET/v2/ticker
```

> **RESPONSE**

```json
{
  "event":"ticker",
  "timestamp":"123443563454",
  "data" :
  [
      {
            "marketCode": "BTC-USDT-SWAP-LIN",
            "last": "43.259", 
            "markPrice": "11012.80409769",  
            "open24h": "49.375",
            "volume24h": "11295421",
            "currencyVolume24h": "1025.7",                       
            "high24h": "49.488",
            "low24h": "41.649",
            "openInterest": "1726003",
            "lastQty": "1"
      },
      ...
  ]
}
```

Get a list of all of the tickers.

Response Parameters | Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING | Timestamp of this response|
marketCode | STRING | "BTC-USDT-SWAP-LIN",
last| STRING | Last traded price
markPrice| STRING | Mark price
open24h| STRING | Daily opening price
volume24h| STRING | 24 hour volume (USDT)
currencyVolume24h| STRING | 24 hour volume (coin)
high24h| STRING | 24 hour high
low24h| STRING | 24 hour low
openInterest| STRING | Current open interest
lastQty| STRING | Last traded quantity


### GET `/v2/candles/{marketCode}`

Get historical candles of active and expired markets.

> **Request**

```json
GET /v2/candles/{marketCode}?timeframe={timeframe}&limit={limit}&startTime={startTime}&endTime={endTime}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | When marketCode is expired market like `BTC-USDT-201225-LIN`, the startTime and the endTime should be explicitly set in `2020` |
timeframe | STRING | NO | e.g. `60s`, `300s`, `900s`, `1800s`, `3600s`, `7200s`, `14400s`, `86400s`, default `3600s ` |
limit | LONG | NO | max `5000 `, default `500`|
startTime | LONG | NO | Millisecond timestamp, e.g. `1579450778000`, default is `limit` times `timeframe` ago, if the limit is `300` and the timeframe is `3600s` then the default startTime is `time now - 300x3600s`, if the limit is not present and the timeframe is `3600s` then the default startTime is `time now - 500x3600s` |
endTime | LONG | NO | Millisecond timestamp, e.g `1579450778000`, default time now |

> **RESPONSE**

```json
{
    "event": "candles",
    "timestamp": "1616743098781",
    "timeframe": "60s",
    "data": [
        {
            "timestamp": "1616713140000",
            "open": "51706.50000000",
            "high": "51758.50000000",
            "low": "51705.50000000",
            "close": "51754.00000000",
            "volume24h": "0",
            "currencyVolume24h": "0"
        },
        {
            "timestamp": "1616713200000",
            "open": "51755.50000000",
            "high": "51833.00000000",
            "low": "51748.00000000",
            "close": "51815.00000000",
            "volume24h": "0",
            "currencyVolume24h": "0"
        },
        ...
    ]
}
```

Response Fields | Type | Description |
----------------| ---- | ----------- |
timestamp(outer) | STRING | |
timeframe | STRING | Selected timeframe |
timestamp(inner) | STRING | Beginning of the candle |
open | STRING | |
high | STRING | |
low | STRING | |
close | STRING | |
volume24h | STRING | 24 hour rolling trading volume in counter currency |
currencyVolumn24h | STRING | 24 hour rolling trading volume in base currency |


### GET `/v2/depth/{marketCode}/{level}`

Get order book by marketCode and level.

> **Request**

```json
GET /v2/depth/BTC-USDT-SWAP-LIN/5 
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "depthL5", 
    "timestamp": "1615457834446", 
    "data": [
        {
            "asks": [
                [
                    54792, 
                    0.001
                ], 
                [
                    54802.5, 
                    0.366
                ], 
                [
                    54803, 
                    0.75
                ], 
                [
                    54806, 
                    1.5
                ], 
                [
                    54830.5, 
                    0.687
                ]
            ], 
            "bids": [
                [
                    54786.5, 
                    0.1
                ], 
                [
                    54754.5, 
                    0.375
                ], 
                [
                    54752, 
                    0.394
                ], 
                [
                    54749.5, 
                    0.001
                ], 
                [
                    54745.5, 
                    0.339
                ]
            ], 
            "marketCode": "BTC-USDT-SWAP-LIN", 
            "timestamp": "1615457834388"
        }
    ]
}
```

Response Fields | Type | Description |
----------------| ---- | ----------- |
event | STRING | |
timestamp | STRING | |
data | LIST | |
asks | LIST of floats | Sell side depth: <ol><li>price</li><li>quantity</li></ol> |
bids | LIST of floats | Buy side depth: <ol><li>price</li><li>quantity</li></ol> |
marketCode | STRING | |
timestamp | STRING | |


### GET `/v2/ping`

Get API service status.

> **Request**

```json
GET /v2/ping
```

> **SUCCESSFUL RESPONSE**

```json
{
    "success": "true"
}
```

Response Fields | Type | Description |
----------------| ---- | ----------- |
sucess | STRING | `"true"` indicates that the API service is OK otherwise it will be failed |
