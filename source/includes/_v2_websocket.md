
# Websocket API

> **Subscription request format**

```json
{
  "op": "<value>",
  "tag": "<value>",
  "args": ["<value1>", "<value2>",.....]
}
```

> **Subscription success response format**

```json
{
  "event": "<opValue>",
  "success": True,
  "tag": "<value>",
  "channel": "<argsValue>",
  "timestamp": "1592491945368"
}
```

> **Subscription failure response format**

```json
{
  "event": "<opValue>",
  "success": False,
  "tag": "<value>",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592498745368"
}
```

> **Command request format**

```json
{
  "op": "<value>",
  "tag": "<value>",
  "data": {"<key1>": "<value1>",.....}
}
```

> **Command success response format**

```json
{
  "event": "<opValue>",
  "success": True
  "tag": "<value>",
  "timestamp": "1592498745368",
  "data": {"<key1>": "<value1>",.....}
}
```

> **Command failure response format**

```json
{
  "event": "<opValue>",
  "success": False,
  "message": "<errorMessage>",
  "code": "<codeCode>",
  "timestamp": "1592498745368",
  "data": {"<key1>": "<value1>",.....}
}
```

**TEST** site

* `wss://stgapi.opnx.com/v2/websocket`

**LIVE** site

* `wss://api.opnx.com/v2/websocket`

Opnx's application programming interface (API) provides our clients programmatic access to control aspects of their accounts and to place orders on the Opnx trading platform. The API is accessible via WebSocket connection to the URIs listed above. Commands, replies, and notifications all traverse the WebSocket in text frames with JSON-formatted payloads.

Websocket commands can be sent in either of the following two formats:

**For subscription based requests**

`{"op": "<value>", "args": ["<value1>", "<value2>",.....]}`

`op`: can either be:

* subscribe
* unsubscribe

`args`: the value(s) will be the instrument ID(s) or asset ID(s), for example:

* order:BTC-oUSD-SWAP-LIN
* depth:ETH-USDT-REPO-LIN
* position:all

**All other commands**

`{"op": "<command>", "data": {"<key1>":"<value1>",.....}}`

`op`: can be:

* login
* placeorder
* cancelorder
* modifyorder

`data`: JSON string of the request object containing the required parameters

Further information regarding the error codes and corresponding error messages from a failed subscription or order command request can be found in a later section of this documentation [Error Codes](#error-codes).


##Authentication

> **Request format**

```json
{
  "op": "login",
  "tag": "<value>",
  "data": {
            "apiKey": "<string>",
            "timestamp": "<string>",
            "signature": "<string>"
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = 'API-KEY'
api_secret = 'API-SECRET'
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

msg_auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}

async def subscribe():
    async with websockets.connect('wss://stgapi.opnx.com/v2/websocket') as ws:
        await ws.send(json.dumps(msg_auth))
        while ws.open:
            resp = await ws.recv()
            print(resp)

asyncio.get_event_loop().run_until_complete(subscribe())
```
```javascript
const CryptoJS = require("crypto-js");
const WebSocket = require('ws');

var apiKey = "API-KEY";
var secretKey = "API-SECRET";
const ts = '' + Date.now();

var sign = CryptoJS.enc.Base64.stringify(CryptoJS.HmacSHA256(ts +'GET/auth/self/verify', secretKey));
var msg = JSON.stringify({  
                            "op": "login",
                            "tag": 1,
                            "data": {
                              "apiKey": apiKey,
                              "timestamp": ts,
                              "signature": sign
                            }
                          });

var ws = new WebSocket('wss://stgapi.opnx.com/v2/websocket');

ws.onmessage = function (e) {
  console.log('websocket message from server : ', e.data);
};

ws.onopen = function () {
    ws.send(msg);
};
```

> **Success response format**

```json
{
  "event": "login",
  "success": true,
  "tag": "<value>",
  "timestamp": "1592491803978"
}
```
```python
{
  "event": "login",
  "success": true,
  "tag": "1",
  "timestamp": "1592491808328"
}
```
```javascript
{
  "event": "login",
  "success": true,
  "tag": "1",
  "timestamp": "1592491808329"
}
```

> **Failure response format**

```json
{
  "event": "login",
  "success": false,
  "code": "<errorCode>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492069732"
}
```
```python
{
  "event": "login",
  "success": false,
  "code": "<errorCode>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492031972"
}
```
```javascript
{
  "event": "login",
  "success": false,
  "code": "<errorCode>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492031982"
}
```

The Websocket API consists of public and private methods. The public methods do not require authentication.  The private methods requires an authenticated websocket connection.

To autenticate a websocket connection a "login" message must be sent containing the clients signature.

The signature is constructed using a HMAC SHA256 operation to get a hash value, which in turn requires the clients API Secret as the key and a constructed message string as the value for the HMAC operation. This hash value is then encoded as a BASE-64 value which becomes the signature used for authentication.

API keys (public and corresponding secret key) can be generated via the GUI within the clients account.

The message string used in the HMAC SHA256 operation is constructed in the following way:

* `current millisecond timestamp + 'GET/auth/self/verify'`

The signature can therefore be summarised by the following:

* `Base64(HmacSHA256(current_ms_timestamp + 'GET/auth/self/verify', API-Secret))`

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | **'login'** |
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
data | DICTIONARY object | Yes |
apiKey | STRING | Yes | Clients public API key, visible in the GUI when created |
timestamp | STRING | Yes | Current millisecond timestamp |
signature | STRING | Yes | `Base64(HmacSHA256(current_ms_timestamp + 'GET/auth/self/verify', API-Secret))` |

## Session Keep Alive

To maintain an active WebSocket connection it is imperative to either be subscribed to a channel that pushes data at least once per minute (Depth) or periodically send a text “ping” (without the quotation mark) to the server within an minute and then the server will reply text “pong” (without quotation mark) to you.

## Self Trade Prevention Modes

Self trade prevention (STP) helps traders avoid costly self-matching within an account and between subaccounts.
There are 4 STP modes available to traders `NONE`, `EXPIRE_MAKER`, `EXPIRE_TAKER`, `EXPIRE_BOTH`.

`NONE` - has no protection meaning self-matching is possible.

`EXPIRE_MAKER` - cancels the resting (would-be maker) order **regardless** of its STP mode, the aggressing order continues to match and/or is placed into the book.

`EXPIRE_TAKER` - cancels the aggressing order to avoid self-matching, acts similar to an IOC that is cancelled in the event of a self-match.

`EXPIRE_BOTH` - cancels both the agressing order and the resting (would-be maker) order **regardless** of the resting order's STP mode.

Note that the STP system uses the aggressing (would-be taker) order to decide which action to take, for example:

Placing an `EXPIRE_TAKER` order which collides with a resting `EXPIRE_MAKER` order causes the aggressing `EXPIRE_TAKER` order to be cancelled, while the `EXPIRE_MAKER` order will remain.


## Order Commands

### Place Limit Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": 1.5,
            "timeInForce": "GTC",
            "price": 9431.48
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_order = \
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": 1.5,
            "timeInForce": "GTC",
            "price": 9431.48
          }
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue

asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json

{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "1.5",
            "timeInForce": "GTC",
            "orderId": "1000000700008",
            "price": "9431.48",
            "limitPrice": "9431.48",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "1.5",
            "timeInForce": "GTC",
            "price": "9431.48",
            "limitPrice": "9431.48",
            "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
data | DICTIONARY object | Yes |
clientOrderId | ULONG | No | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Yes | Market code e.g. `BTC-oUSD-SWAP-LIN` |
orderType | STRING | Yes |  `LIMIT` |
price | FLOAT |  Yes | Price |
quantity |  FLOAT | Yes | Quantity (denominated by contractValCurrency) |
displayQuantity |  FLOAT | NO |If given, the order becomes an iceberg order, and denotes the quantity to show on the book|
side | STRING | Yes | `BUY` or `SELL` |
timeInForce | ENUM | No | <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade)</li></ul>
timestamp | LONG | No | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | No | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
selfTradePreventionMode | STRING | No | `NONE`, `EXPIRE_MAKER`, `EXPIRE_TAKER`, `EXPIRE_BOTH`|


### Place Market Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": 5
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = '' 
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_order = \
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": 5
          }
}


url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "5",
            "orderId": "1000000700008",
            "limitPrice": "1700.00",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "5",
            "limitPrice": "1700.00",
            "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
data | DICTIONARY object | Yes |
clientOrderId | ULONG | No | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Yes | Market code e.g. `BTC-oUSD-SWAP-LIN` |
orderType | STRING | Yes |  `MARKET` |
quantity |  FLOAT | Yes | Quantity (denominated by contractValCurrency), not required if an `amount` is provided |
amount | STRING | NO | An amount of USDT can be specified instead of a quantity. Only valid for spot market buy orders|
side | STRING | Yes | `BUY` or `SELL` |
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
selfTradePreventionMode | STRING | No | `NONE`, `EXPIRE_MAKER`, `EXPIRE_TAKER`, `EXPIRE_BOTH`|

### Place Stop Limit Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": 10,
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": 100,
            "limitPrice": 120
         }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_order = \
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": 10,
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": 100,
            "limitPrice": 120
         }
}


url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY_REPRICE",
            "price": "120",
            "limitPrice": "120",
            "stopPrice": "100",
            "orderId": "1000000700008",
            "source": 0
            "triggerType": "MARK_PRICE"
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY_REPRICE",
            "price": "120",
            "stopPrice": "100",
            "limitPrice": "120",
            "source": 0,
            "triggerType": "MARK_PRICE"
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
data | DICTIONARY object | Yes |
clientOrderId | ULONG | No | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode| STRING| Yes| Market code e.g. `ETH-oUSD-SWAP-LIN`|
orderType|STRING| Yes|  `STOP_LIMIT` for stop-limit orders |
quantity|FLOAT|Yes|Quantity (denominated by contractValCurrency)|
side|STRING| Yes| `BUY ` or `SELL`|
limitPrice| FLOAT |Yes | Limit price for the stop-limit order. <p><p>For **BUY** the limit price must be greater or equal to the stop price.<p><p>For **SELL** the limit price must be less or equal to the stop price.|
stopPrice| FLOAT |Yes|Stop price for the stop-limit order.<p><p>Triggered by the best bid price for the **SELL** stop-limit order.<p><p>Triggered by the best ask price for the **BUY** stop-limit order. |
timeInForce | ENUM | No | <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade)</li></ul>
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
selfTradePreventionMode | STRING | No | `NONE`, `EXPIRE_MAKER`, `EXPIRE_TAKER`, `EXPIRE_BOTH`|

### Place Stop Market Order

Stop market orders are only available in Perp markets.

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1679907302693,
            "recvWindow": 500,
            "clientOrderId": 1679907301552,
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "side": "SELL",
            "orderType": "STOP_MARKET",
            "quantity": 0.012,
            "stopPrice": 22279.29
         }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_order = \
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1679907302693,
            "recvWindow": 500,
            "clientOrderId": 1679907301552,
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "side": "SELL",
            "orderType": "STOP_MARKET",
            "quantity": 0.001,
            "stopPrice": 22279.29
         }
}


url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "1679907301552",
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "side": "SELL",
            "orderType": "STOP_MARKET",
            "quantity": "0.012",
            "timeInForce": "IOC",
            "price": "25000",
            "limitPrice": "25000",
            "stopPrice": "22279.29",
            "orderId": "1000001680990",
            "triggerType": "MARK_PRICE",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1679907302693",
  "data": {
           "clientOrderId": "1679907301552",
           "marketCode": "BTC-oUSD-SWAP-LIN",
           "side": "SELL",
           "orderType": "STOP_MARKET",
           "quantity": "0.012",
           "timeInForce": "IOC",
           "price": "25000",
           "limitPrice": "25000",
           "stopPrice": "22279.29"
           "triggerType": "MARK_PRICE",
           "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
data | DICTIONARY object | Yes |
clientOrderId | ULONG | No | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode| STRING| Yes| Market code e.g. `BTC-oUSD-SWAP-LIN`|
orderType|STRING| Yes|  `STOP_MARKET` |
quantity|FLOAT|Yes|Quantity (denominated by contractValCurrency)|
side|STRING| Yes| `BUY ` or `SELL`|
stopPrice| FLOAT |Yes|Stop price for the stop-market order.<p><p>Triggered by the best bid price for the **SELL** stop-market order.<p><p>Triggered by the best ask price for the **BUY** stop-market order. |
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
selfTradePreventionMode | STRING | No | `NONE`, `EXPIRE_MAKER`, `EXPIRE_TAKER`, `EXPIRE_BOTH`|


### Place Batch Orders

> **Request format**

```json
{
  "op": "placeorders",
  "tag": 123,
  "dataArray": [{
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 1,
                  "marketCode": "ETH-oUSD-SWAP-LIN",
                  "side": "BUY",
                  "orderType": "LIMIT",
                  "quantity": 10,
                  "timeInForce": "MAKER_ONLY",
                  "price": 100
                }, 
                {
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 2,
                  "marketCode": "BTC-USDT",
                  "side": "SELL",
                  "orderType": "MARKET",
                  "quantity": 0.2
                }]
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_batch_order =\
{
  "op": "placeorders",
  "tag": 123,
  "dataArray": [{
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 1,
                  "marketCode": "ETH-oUSD-SWAP-LIN",
                  "side": "BUY",
                  "orderType": "LIMIT",
                  "quantity": 10,
                  "timeInForce": "MAKER_ONLY",
                  "price": 100
                },
                {
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 2,
                  "marketCode": "BTC-USDT",
                  "side": "SELL",
                  "orderType": "MARKET",
                  "quantity": 0.2
                }]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_batch_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue

asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY",
            "price": "100",
            "limitPrice": "100",
            "orderId": "1000003700008",
            "source": 0
          }
}

AND

{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739136",
  "data": {
            "clientOrderId": "2",
            "marketCode": "BTC-USDT",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "0.2",
            "orderId": "1000004700009",
            "limitPrice": "20000",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-oUSD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY",
            "price": "100",
            "limitPrice": "100",
            "source": 0
          }
}

AND

{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503457",
  "data": {
            "clientOrderId": "2",
            "marketCode": "BTC-USDT",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "0.2",
            "limitPrice": "20000",
            "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

All existing single order placement methods are supported:-

* LIMIT
* MARKET
* STOP LIMIT
* STOP MARKET

The websocket reply from the exchange will repond to each order in the batch separately, one order at a time, and has the same message format as the reponse for the single order placement method.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorders`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
dataArray | LIST of dictionaries | Yes | A list of orders with each order in JSON format, the same format/parameters as the request for placing a single order. The max number of orders is still limited by the message length validation so by default up to 20 orders can be placed in a batch, assuming that each order JSON has 200 characters. |
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |


### Cancel Order

> **Request format**

```json
{
  "op": "cancelorder",
  "tag": 456,
  "data": {
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "orderId": 12
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
cancel_order = \
{
  "op": "cancelorder",
  "tag": 456,
  "data": {
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "orderId": 12
          }
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(cancel_order))
            elif 'event' in data and data['event'] == 'cancelorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "cancelorder",
  "submitted": True,
  "tag": "456",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "clientOrderId": "1",
            "orderId": "12"
          }
}
```

> **Failure response format**

```json
{
  "event": "cancelorder",
  "submitted": False,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "orderId": "12"
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderClosed etc......).

This command can also be actioned via the trading GUI using the **Cancel** button next to an open order in the **Open Orders** blotter for both Spot and Derivative markets.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `cancelorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
data | DICTIONARY object | Yes |
marketCode|STRING|Yes|Market code e.g. `BTC-oUSD-SWAP-LIN`|
orderId|INTEGER|Yes|Unique order ID from the exchange|


### Cancel Batch Orders

> **Request format**

```json
{
  "op": "cancelorders",
  "tag": 456,
  "dataArray": [{
                  "marketCode": "BTC-oUSD-SWAP-LIN",
                  "orderId": 12
                },
                {
                  "marketCode": "BCH-USDT",
                  "orderId": 34
                }]
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
cancel_batch_order = \
{
  "op": "cancelorders",
  "tag": 456,
  "dataArray": [{
                  "marketCode": "BTC-oUSD-SWAP-LIN",
                  "orderId": 12
                },
                {
                  "marketCode": "BCH-USDT",
                  "orderId": 34
                }]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(cancel_batch_order))
            elif 'event' in data and data['event'] == 'cancelorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "cancelorder",
  "submitted": True,
  "tag": "456",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "clientOrderId": "1",
            "orderId": "12"
          }
}

AND

{
  "event": "cancelorder",
  "submitted": True,
  "tag": "456",
  "timestamp": "1592491173978",
  "data": {
            "marketCode": "BCH-USDT",
            "orderId": "34"
          }
}

```

> **Failure response format**

```json
{
  "event": "cancelorder",
  "submitted": False,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "orderId": "12"
          }
}

AND

{
  "event": "cancelorder",
  "submitted": False,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491173989",
  "data": {
            "marketCode": "BCH-USDT",
            "orderId": "12"
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderClosed etc......).

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `cancelorders`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
dataArray | LIST of dictionaries | Yes |A list of orders with each order in JSON format, the same format/parameters as the request for cancelling a single order. The max number of orders is still limited by the message length validation so by default up to 20 orders can be placed in a batch, assuming that each order JSON has 200 characters.|


### Modify Order

> **Request format**

```json
{
  "op": "modifyorder",
  "tag": 1,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "orderId": 888,
            "side": "BUY",
            "price": 9800,
            "quantity": 2
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
modify_order = \
{
  "op": "modifyorder",
  "tag": 1,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "marketCode": "BTC-oUSD-SWAP-LIN",
            "orderId": 888,
            "side": "BUY",
            "price": 9800,
            "quantity": 2
          }
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(modify_order))
            elif 'event' in data and data['event'] == 'modifyorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "modifyorder",
  "submitted": True,
  "tag": "1",
  "timestamp": "1592491032427",
  "data":{
      "clientOrderId": "1",
      "orderId": "888",
      "side": "BUY",
      "quantity": "2"
      "price": "9800",
      "limitPrice": "9800",
      "orderType": "LIMIT",
      "marketCode": "BTC-oUSD-SWAP-LIN"
  }
}
```

> **Failure response format**

```json
{
  "event": "modifyorder",
  "submitted": False,
  "tag": "1",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491032427",
  "data": {
            "orderId": "888",
            "side": "BUY",
            "quantity": "2",
            "price": "9800",
            "limitPrice": "9800",
            "marketCode": "BTC-oUSD-SWAP-LIN"
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderModified etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

Currently only LIMIT orders are supported by the modify order command.

* The price and/or quantity of an order can be modified.
* Reducing the quantity will leave the modified orders position in the order queue **unchanged**.
* Increasing the quantity will **always** move the modified order to the back of the order queue.
* Modifying the price will **always** move the modified order to the back of the order queue.

Modified orders retain their original orderId.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `modifyorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
data | DICTIONARY object | Yes |
marketCode|STRING|Yes| Market code e.g. `BTC-oUSD-SWAP-LIN`|
orderId|INTEGER|Yes|Unique order ID from the exchange|
side| STRING|No| `BUY` or `SELL`|
price|FLOAT|No|Price for limit orders|
quantity|FLOAT|No|  Quantity (denominated by `contractValCurrency`)|
timestamp | LONG | No | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | No | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |


### Modify Batch Orders

> **Request format**

```json
{
  "op": "modifyorders",
  "tag": 123,
  "dataArray": [{
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "marketCode": "ETH-oUSD-SWAP-LIN",
                  "side": "BUY",
                  "orderID": 304304315061932310,
                  "price": 101,
                }, 
                {
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "marketCode": "BTC-USDT",
                  "orderID": 304304315061864646,
                  "price": 10001,
                  "quantity": 0.21
                }]
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
modify_batch_order = \
{
  "op": "modifyorders",
  "tag": 123,
  "dataArray": [{
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "marketCode": "ETH-oUSD-SWAP-LIN",
                  "side": "BUY",
                  "orderID": 304304315061932310,
                  "price": 101,
                }, 
                {
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "marketCode": "BTC-USDT",
                  "orderID": 304304315061864646,
                  "price": 10001,
                  "quantity": 0.21
                }]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(modify_batch_order))
            elif 'event' in data and data['event'] == 'modifyorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```


> **Success response format**

```json
{
  "event": "modifyorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "100"
            "orderId": "304304315061932310",
            "side": "BUY",
            "quantity": "5",
            "price": "101",
            "limitPrice": "101",
            "orderType": "LIMIT",
            "marketCode": "ETH-oUSD-SWAP-LIN"
          }
}

AND

{
  "event": "modifyorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739136",
  "data": {
            "clientOrderId": "200"
            "orderId": "304304315061864646",
            "side": "SELL",
            "quantity": "0.21",
            "price": "10001",
            "limitPrice": "10001",
            "orderType": "LIMIT",
            "marketCode": "BTC-USDT"
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503359",
  "data": {
            "orderID": "304304315061932310",
            "side": "BUY",
            "price": "101",
            "limitPrice": "101",
            "marketCode": "ETH-oUSD-SWAP-LIN"
          }
}

AND

{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503457",
  "data": {
            "orderID": "304304315061864646",
            "quantity": "0.21",
            "price": "10001",
            "limitPrice": "10001",
            "marketCode": "BTC-USDT"
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

The websocket responses from the exchange will come separately for each order in the batch, one order at a time, and the message has the same format as the single `modifyorder` method.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `modifyorders`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
dataArray | LIST of dictionaries | Yes | A list of orders with each order in JSON format, the same format/parameters as the request for modifying a single order.  The max number of orders is still limited by the message length validation so by default up to 20 orders can be modified in a batch, assuming that each order JSON has 200 characters.
timestamp | LONG | No | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | No | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |


## Subscriptions - Private

All subscriptions to private account channels requires an authenticated websocket connection.

Multiple subscriptions to different channels both public and private can be made within a single subscription command: 

`{"op": "subscribe", "args": ["<value1>", "<value2>",.....]}`


### Balance Channel

> **Request format**

```json
{
  "op": "subscribe",
  "args": ["balance:all"],
  "tag": 101
}

OR

{
  "op": "subscribe", 
  "args": ["balance:USDT", "balance:OX", ........], 
  "tag": 101
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
balance = \
{
  "op": "subscribe",
  "args": ["balance:all"],
  "tag": 101
}
url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(balance))
            elif 'event' in data and data['event'] == 'balance':
                 continue
asyncio.get_event_loop().run_until_complete(subscribe())

```

> **Success response format**

```json
{
  "success": True, 
  "tag": "101", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

> **Balance channel format**

```json
{
  "table": "balance",
  "accountId": "<Your account ID>",
  "timestamp": "1599693365059",
  "tradeType": "STANDARD",
  "data":[
      {
          "total": "10000",
          "reserved": "1000",
          "instrumentId": "USDT",
          "available": "9000",
          "locked": "0"
          "quantityLastUpdated": "1599694369431",
       },
       {
          "total": "100000",
          "reserved": "0",
          "instrumentId": "OX",
          "available": "100000",
          "locked": "0"
          "quantityLastUpdated": "1599694343242",
        }
  ]
}
```

**Channel Update Frequency** : On update

The websocket will reply with the shown success response format for subscribed assets with **changed** balances.

If a subscription has been made to **balance:all**, the data array in the message from this balance channel will contain a JSON **list**, otherwise the data array will contain a **single** JSON corresponding to one spot asset per asset channel subscription.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | LIST | Yes | `balance:all` or a list of individual assets `balance:<assetId>`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |

<sub>**Channel Update Fields**</sub> 

Fields |Type| Description |
--------|-----|---|
table | STRING| `balance`
accountId | STRING|  Account identifier
timestamp|STRING | Current millisecond timestamp
tradeType|STRING | `LINEAR`, `STANDARD`, `PORTFOLIO`
data | LIST of dictionaries |
total | STRING | Total spot asset balance
reserved | STRING | Reserved asset balance for working spot and repo orders
instrumentId | STRING |  Base asset ID e.g. `BTC`
available | STRING| Remaining available asset balance (total - reserved)
locked | STRING | Temporarily locked asset balance
quantityLastUpdated|STRING | Millisecond timestamp

### Position Channel

> **Request format**

```json
{
  "op": "subscribe", 
  "args": ["position:all"], 
  "tag": 102
}

OR

{
  "op": "subscribe",
  "args": ["position:BTC-oUSD-SWAP-LIN", "position:BCH-oUSD-SWAP-LIN", ........], 
  "tag": 102
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
position = \
{
  "op": "subscribe", 
  "args": ["position:all"], 
  "tag": 102
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(position))
            elif 'event' in data and data['event'] == 'position':
                 continue
asyncio.get_event_loop().run_until_complete(subscribe())

```

> **Success response format**

```json
{
  "success": True, 
  "tag": "102", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

> **Position channel format**

```json
{
  "table": "position",
  "accountId": "<Your account ID>",
  "timestamp": "1607985371481",
  "data":[ {
              "instrumentId": "ETH-oUSD-SWAP-LIN",
              "quantity" : "0.1",
              "lastUpdated": "1616053755423",
              "contractValCurrency": "ETH",
              "entryPrice": "1900.0",
              "positionPnl": "-5.6680",
              "estLiquidationPrice": "0",
              "margin": "0",
              "leverage": "0"
            },
            {
              "instrumentId": "ETH-oUSD-SWAP-LIN",
              "quantity" : "50.54",
              "lastUpdated": "1617099855968",
              "contractValCurrency": "ETH",
              "entryPrice": "2000.0",
              "positionPnl": "1220.9494164000000",
              "estLiquidationPrice": "1317.2",
              "margin": "0",
              "leverage": "0"
            },
            ...
          ]
}
```

**Channel Update Frequency** : real-time, on position update

The websocket will reply with the shown success response format for **each** position channel which has been successfully subscribed to.

If a subscription has been made to **position:all**, the data array in the message from this position channel will contain a JSON **list**. Each JSON will contain position details for a different instrument. Otherwise the data array will contain a **single** JSON corresponding to one instrument.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | LIST | Yes | `position:all` or a list of individual instruments `position:<instrumentId>`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |

<sub>**Channel Update Fields**</sub> 

Fields | Type | Description |
------ | ---- | ----------- |
table | STRING | `position` |
accountId | STRING | Account identifier |
timestamp | STRING | Current millisecond timestamp |
data | LIST of dictionaries | |
instrumentId | STRING | e.g. `ETH-oUSD-SWAP-LIN` |
quantity | STRING | Position size (+/-) |
lastUpdated | STRING | Millisecond timestamp |
contractValCurrency | STRING | Base asset ID e.g. `ETH` |
entryPrice | STRING | Average entry price of total position (Cost / Size) |
positionPnl | STRING | Postion profit and lost |
estLiquidationPrice | STRING | Estimated liquidation price, return 0 if it is negative(<0) |
margin | STRING | Currently always reports 0
leverage | STRING | Currently always reports 0


### Order Channel

> **Request format**

```json
{
  "op": "subscribe", 
  "args": ["order:all"], 
  "tag": 102
}

OR

{
  "op": "subscribe", 
  "args": ["order:OX-USDT", "order:ETH-oUSD-SWAP-LIN", .....], 
  "tag": 102
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
order = \
{
  "op": "subscribe", 
  "args": ["order:all"], 
  "tag": 102
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(order))
            elif 'event' in data and data['event'] == 'order':
                 continue
asyncio.get_event_loop().run_until_complete(subscribe())

```

> **Success response format**

```json
{
  "success": True, 
  "tag": "102", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

**Channel Update Frequency** : real-time, on order update

Every order update for a particular sub-account will be relayed to all of its active connections. This implies that for each sub-account, the order responses for all connected markets will be sent to all active subscriptions, even if a specific connection is only subscribed to a single market.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | LIST | Yes | `order:all` or a list of individual markets `order:<marketCode>`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |


#### OrderOpened

> **OrderOpened message format - LIMIT order**

```json
{
  "table": "order",
  "data": [ 
      {
          "notice": "OrderOpened",
          "accountId": "<Your account ID>",
          "clientOrderId": "16",
          "orderId" : "123",
          "price": "9600",
          "limitPrice": "9600",
          "quantity": "2",
          "remainQuantity": "2",
          "amount": "0.0",
          "side": "BUY",
          "status": "OPEN",
          "marketCode": "BTC-oUSD-SWAP-LIN",
          "timeInForce": "MAKER_ONLY",
          "timestamp": "1594943491077",
          "orderType": "LIMIT",
          "isTriggered": "False",
          "displayQuantity": "2"
       }
  ]
}
```

> **OrderOpened message format - STOP MARKET order**

```json
{
  "table": "order",
  "data": [
      {
          "accountId": "<Your account ID>", 
          "clientOrderId": "1", 
          "orderId": "1000021706785", 
          "price": "12000.0", 
          "quantity": "0.001", 
          "amount": "0.0", 
          "side": "BUY", 
          "status": "OPEN", 
          "marketCode": "BTC-oUSD-SWAP-LIN", 
          "timeInForce": "IOC", 
          "timestamp": "1680042503604", 
          "remainQuantity": "0.001", 
          "stopPrice": "10000.0", 
          "limitPrice": "12000.0", 
          "notice": "OrderOpened", 
          "orderType": "STOP_MARKET", 
          "isTriggered": "false", 
          "triggerType": "MARK_PRICE", 
          "displayQuantity": "0.001"
      }
  ]
}

```

<sub>**Channel Update Fields**</sub>

Fields |Type| Description |
--------|-----|---|
table | STRING | `order`
data | LIST of dictionary |
notice | STRING| `OrderOpened`
accountId | STRING| Account identifier
clientOrderId |  STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING | Unique order ID from the exchange
price |STRING | Limit price submitted (only applicable for LIMIT order types)
quantity | STRING| Quantity submitted
amount | STRING | "0.0" if not provided in the request
side|STRING|`BUY` or `SELL`
status|STRING|  Order status
marketCode | STRING |  Market code e.g. `OX-USDT`
timeInForce|STRING| Client submitted time in force, `GTC` by default
timestamp|STRING |Current millisecond timestamp
remainQuantity | STRING | Working quantity
orderType| STRING | `LIMIT`, `STOP_LIMIT`, or `STOP_MARKET`
stopPrice| STRING |Stop price submitted (only applicable for STOP order types)
limitPrice|STRING|Limit price submitted
isTriggered|STRING|`False` or `True`
triggerType|STRING| Stops are triggered on `MARK_PRICE`
displayQuantity |STRING| Quantity displayed in the book, primarily used for iceberg orders, otherwise echos the quantity field|



#### OrderClosed

> **OrderClosed message format - LIMIT order**

```json
{
  "table": "order", 
  "data": [
      {
          "accountId": "<Your account ID>", 
          "clientOrderId": "1", 
          "orderId": "1000021764611", 
          "price": "20000.0", 
          "quantity": "0.001", 
          "amount": "0.0", 
          "side": "BUY", 
          "status": "CANCELED_BY_USER", 
          "marketCode": "BTC-oUSD-SWAP-LIN", 
          "timeInForce": "GTC", 
          "timestamp": "1680043402806", 
          "remainQuantity": "0.001", 
          "limitPrice": "20000.0", 
          "notice": "OrderClosed", 
          "orderType": "LIMIT", 
          "isTriggered": "false", 
          "displayQuantity": "0.001"
      }
  ]
}

```

> **OrderClosed message format - STOP LIMIT order**

```json
{
  "table": "order", 
  "data": [
      {
          "accountId": "<Your account ID>", 
          "clientOrderId": "1", 
          "orderId": "1000021852415", 
          "price": "21000.0", 
          "quantity": "0.001", 
          "amount": "0.0", 
          "side": "BUY", 
          "status": "CANCELED_BY_USER", 
          "marketCode": "BTC-oUSD-SWAP-LIN", 
          "timeInForce": "GTC", 
          "timestamp": "1680044038047", 
          "remainQuantity": "0.001", 
          "stopPrice": "20000.0", 
          "limitPrice": "21000.0", 
          "notice": "OrderClosed", 
          "orderType": "LIMIT", 
          "isTriggered": "true", 
          "triggerType": "MARK_PRICE", 
          "displayQuantity": "0.001"
      }
  ]
}


```

There are multiple scenarios in which an order is closed as described by the **status** field in the OrderClosed message.  In summary orders can be closed by:-

* `CANCELED_BY_USER` - the client themselves initiating this action or the liquidation engine on the clients behalf
* `CANCELED_BY_MAKER_ONLY` - if a maker-only order is priced such that it would actually be an agressing taker trade, the order is automatically canceled to prevent this order from matching as a taker
* `CANCELED_BY_FOK` - since fill-or-kill orders requires **all** of the order quantity to immediately take and match at the submitted limit price or better, if no such match is possible then the whole order quantity is canceled
* `CANCELED_ALL_BY_IOC` - since immediate-or-cancel orders also requires an immediate match at the specified limit price or better, if no such match price is possible for **any** of the submitted order quantity then the whole order quantity is canceled
* `CANCELED_PARTIAL_BY_IOC` - since immediate-or-cancel orders only requires **some** of the submitted order quantity to immediately take and match at the specified limit price or better, if a match is possible for only a **partial** quantity then only the remaining order quantity which didn't immediately match is canceled
* `CANCELED_BY_SELF_TRADE_PROTECTION` - orders canceled by your selected selfTradePreventionMode settings

<sub>**Channel Update Fields**</sub>

Fields | Type | Description
-------------------------- | -----|--------- |
table | STRING | `order`
data | LIST of dictionary |
notice | STRING | `OrderClosed`
accountId | STRING  |  Account identifier
clientOrderId|STRING |  Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING  |  Unique order ID from the exchange
price|STRING |Limit price of closed order (only applicable for LIMIT order types)
quantity|STRING |Original order quantity of closed order
amount | STRING | "0.0" if not provided in the request
side|STRING |`BUY` or `SELL`
status|STRING | <ul><li>`CANCELED_BY_USER`</li><li>`CANCELED_BY_MAKER_ONLY`</li><li>`CANCELED_BY_FOK`</li><li>`CANCELED_ALL_BY_IOC`</li><li>`CANCELED_PARTIAL_BY_IOC`</li></ul>
marketCode|STRING |  Market code e.g. `BTC-oUSD-SWAP-LIN`
timeInForce|STRING |Time in force of closed order
timestamp|STRING |Current millisecond timestamp
remainQuantity|STRING |Historical remaining order quantity of closed order
stopPrice|STRING|Stop price of closed stop order (only applicable for STOP order types)
limitPrice|STRING|Limit price
ordertype | STRING | `LIMIT` or `STOP_LIMIT`
isTriggered | STRING | `False` or `True`
triggerType|STRING| Stops are triggered on `MARK_PRICE`
displayQuantity |STRING| Quantity displayed in the book, primarily used for iceberg orders, otherwise echos the quantity field|


#### OrderClosed Failure

> **OrderClosed failure message format**

```json
{
  "event": "CANCEL", 
  "submitted": False, 
  "message": "Order request was rejected : REJECT_CANCEL_ORDER_ID_NOT_FOUND", 
  "code": "100004", 
  "timestamp": "0", 
  "data": {
            "clientOrderId": 3,
            "orderId": 3330802124194931673, 
            "displayQuantity": 0.0, 
            "lastMatchPrice": 0.0, 
            "lastMatchQuantity": 0.0, 
            "lastMatchedOrderId": 0, 
            "lastMatchedOrderId2": 0, 
            "matchedId": 0, 
            "matchedType": "MAKER", 
            "remainQuantity": 0.0, 
            "side": "BUY", 
            "status": "REJECT_CANCEL_ORDER_ID_NOT_FOUND", 
            "timeCondition": "GTC", 
            "marketCode": "BTC-oUSD-SWAP-LIN", 
            "timestampEpochMs": 1615377638518, 
            "orderType": "LIMIT",
            "price": 0.0, 
            "quantity": 0.0, 
            "isTriggered": False
          }
}
```

This order message can occur if:- 

* an order has already been matched by the time the cancel order command is recieved and processed by the exchange which means this order is no longer active and therefore cannot be closed.
* multiple cancel order commands for the **same** orderID have been sent in quick sucession to the exchange by mistake and only the first cancel order command is accepted and processed by the exchange which means this order is no longer active and therefore cannot be closed again.

* For more error messages and code, you can see them here [Error Codes](#error-codes)

<sub>**Channel Update Fields**</sub>

Fields | Type | Description
-------------------------- | ----- |--------
event | STRING |
submitted | BOOL |
message | STRING |
code | STRING | 
timestamp | STRING |
data | LIST of dictionary |
clientOrderId|STRING|  Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING|   Unique order ID from the exchange
displayQuantity | DECIMAL
lastMatchPrice | DECIMAL
lastMatchQuantity | DECIMAL
lastMatchedOrderId | DECIMAL
lastMatchedOrderId2 | DECIMAL
matchedId | DECIMAL
matchedType | STRING
remainQuantity | DECIMAL | Historical remaining quantity of the closed order |
side|STRING
status | STRING
timeCondition | STRING 
marketCode | STRING 
timestampEpochMs | LONG
orderType | STRING
price | DECIMAL | `LIMIT` or `STOP_LIMIT` |
quantity | DECIMAL
isTriggered | BOOL | `False` or `True`


#### OrderModified

> **OrderModified message format**

```json
{
  "table": "order", 
  "data": [
      {
          "accountId": "<Your account ID>",
          "clientOrderId": "1",
          "orderId": "1000021878849",
          "price": "30.0",
          "quantity": "0.001",
          "amount": "0.0",
          "side": "BUY",
          "status": "OPEN",
          "marketCode": "BTC-oUSD-SWAP-LIN",
          "timeInForce": "GTC",
          "timestamp": "1680044356374",
          "remainQuantity": "0.001",
          "limitPrice": "30.0",
          "notice": "OrderModified",
          "orderType": "LIMIT",
          "isTriggered": "false",
          "displayQuantity": "0.001"
      }
  ]
}

```

As described in a previous section [Order Commands - Modify Order](?json#modify-order), the Modify Order command can potentially affect the queue position of the order depending on which parameter of the original order has been modified. Orders retain their orderIds after modification.

<sub>**Channel Update Fields**</sub>

Fields |Type| Description |
--------|-----|---|
table | STRING | `order`
data | LIST of dictionary |
notice | STRING | `OrderModified`
accountId | STRING | Account identifier
clientOrderId |  STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING | Unique order ID from the exchange
price |STRING | Limit price of modified order (only applicable for LIMIT order types)
quantity | STRING| Quantity of modified order
remainQuantity | STRING | Working quantity
amount|STRING| "0.0" if not provided in the request
side|STRING|`BUY` or `SELL`
status|STRING|  Order status
marketCode | STRING |  Market code e.g. `BTC-oUSD-SWAP-LIN`
timeInForce|STRING| Client submitted time in force, `GTC` by default
timestamp|STRING |Current millisecond timestamp
orderType| STRING | `LIMIT` or `STOP_LIMIT`
stopPrice|STRING|Stop price of modified order (only applicable for STOP order types)
limitPrice|STRING|Limit price of modified order
isTriggered|STRING|`False` or `True` |
triggerType|STRING| Stops are triggered on `MARK_PRICE`|
displayQuantity |STRING| Quantity displayed in the book, primarily used for iceberg orders, otherwise echos the quantity field|


#### OrderModified Failure

> **OrderModified failure message format**

```json
{
  "event": "AMEND", 
  "submitted": False, 
  "message": "Order request was rejected : REJECT_AMEND_ORDER_ID_NOT_FOUND", 
  "code": "100004", 
  "timestamp": "0", 
  "data": {
            "clientOrderId": 3, 
            "orderId": 3330802124194931673, 
            "displayQuantity": 917.5, 
            "lastMatchPrice": 0.0, 
            "lastMatchQuantity": 0.0, 
            "lastMatchedOrderId": 0, 
            "lastMatchedOrderId2": 0, 
            "matchedId": 0, 
            "matchedType": "MAKER", 
            "remainQuantity": 0.0, 
            "side": "BUY", 
            "status": "REJECT_AMEND_ORDER_ID_NOT_FOUND", 
            "timeCondition": "GTC", 
            "marketCode": "BTC-oUSD-SWAP-LIN", 
            "timestampEpochMs": 1615377638518, 
            "orderType": "LIMIT", 
            "price": 22, 
            "quantity": 917.5, 
            "isTriggered": False
          }
}
```

This order message can occur if an order has already been matched by the time the modify order command is recieved and processed by the exchange which means this order is no longer active and therefore cannot be modified.
For more error messages and code, you can see them here [Error Codes](#error-codes)

<sub>**Channel Update Fields**</sub>

Fields | Type | Description
-------------------------- | ----- |--------
event | STRING |
submitted | BOOL |
message | STRING |
code | STRING | 
timestamp | STRING |
data | LIST of dictionary |
clientOrderId|STRING|  Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING|   Unique order ID from the exchange
displayQuantity | DECIMAL
lastMatchPrice | DECIMAL
lastMatchQuantity | DECIMAL
lastMatchedOrderId | DECIMAL
lastMatchedOrderId2 | DECIMAL
matchedId | DECIMAL
matchedType | STRING
remainQuantity | DECIMAL
side|STRING
status | STRING
timeCondition | STRING 
marketCode | STRING 
timestampEpochMs | LONG
orderType | STRING
price | DECIMAL
quantity | DECIMAL
isTriggered | BOOL


#### OrderMatched

> **OrderMatched message format**

```json
{
  "table": "order", 
  "data": [
      {
          "accountId": "<Your account ID>",
          "clientOrderId": "1680044888401",
          "orderId": "1000021937814",
          "price": "27282.73",
          "quantity": "0.004",
          "amount": "0.0",
          "side": "BUY",
          "status": "FILLED", 
          "marketCode": "BTC-oUSD-SWAP-LIN",
          "timeInForce": "GTC",
          "timestamp": "1680044888565",
          "matchId": "300016799886154670",
          "matchPrice": "27279.39",
          "matchQuantity": "0.001",
          "orderMatchType": "TAKER",
          "remainQuantity": "0.0",
          "limitPrice": "27282.73",
          "notice": "OrderMatched",
          "orderType": "LIMIT",
          "fees": "0.019095573", 
          "feeInstrumentId": "USDT",
          "isTriggered": "false",
          "displayQuantity": "0.004"
      }
  ]
}

```

<sub>**Channel Update Fields**</sub>

Fields | Type | Required
-------------------------- | -----|--------- |
table | STRING | `order`
data | LIST of dictionary |
notice | STRING | `OrderMatched`
accountId | STRING | Account identifier
clientOrderId|STRING|  Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING|   Unique order ID from the exchange
price|STRING| Limit price submitted (only applicable for LIMIT order types)
stopPrice|STRING| Stop price submitted (only applicable for STOP order types)
limitPrice|STRING| Limit price submitted
quantity|STRING|Order quantity submitted
amount|STRING| "0.0" if not provided in the request
side|STRING|`BUY` or `SELL`
status|STRING|`FILLED` or `PARTIAL_FILL`
marketCode|STRING| Market code i.e. `BTC-oUSD-SWAP-LIN`
timeInForce|STRING|Client submitted time in force (only applicable for LIMIT and STOP LIMIT order types)
timestamp|STRING|Millisecond timestamp of order match
matchID|STRING|Exchange match ID
matchPrice|STRING|Match price of order from this match ID
matchQuantity|STRING|Match quantity of order from this match ID
orderMatchType|STRING|`MAKER` or `TAKER`
remainQuantity|STRING|Remaining order quantity
orderType|STRING|<ul><li>`LIMIT`</li><li>`MARKET`</li></ul> triggered `STOP_LIMIT` and `STOP_MARKET` orders are converted to `LIMIT` and `MARKET` orders respectively
fees|STRING|Amount of fees paid from this match ID 
feeInstrumentId|STRING|Instrument ID of fees paid from this match ID 
isTriggered|STRING|`False` (or `True` for STOP order types)
triggerType|STRING| Stops are triggered on `MARK_PRICE`|
displayQuantity |STRING| Quantity displayed in the book, primarily used for iceberg orders, otherwise echos the quantity field|


## Subscriptions - Public

All subscriptions to public channels do **not** require an authenticated websocket connection.

Multiple subscriptions to different channels both public and private can be made within a single subscription command:

`{"op": "subscribe", "args": ["<value1>", "<value2>",.....]}`

### Fixed Size Order Book

> **Request format**

```json
{
  "op": "subscribe",
  "tag": 103,
  "args": ["depthL10:BTC-oUSD-SWAP-LIN"]
}
```
```python
import websockets
import asyncio
import json

orderbook_depth = \
{
  "op": "subscribe",
  "tag": 103,
  "args": ["depthL10:BTC-oUSD-SWAP-LIN"]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(orderbook_depth))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())

```


> **Success response format**

```json
{
    "success": true,
    "tag": "103",
    "event": "subscribe",
    "channel": "depthL10:BTC-oUSD-SWAP-LIN",
    "timestamp": "1665454814275"
}
```

> **Order Book depth channel format**

```json
{
    "table": "depthL10",
    "data": {
        "seqNum": 2166539633781384,
        "asks": [
            [
                19024.0,
                1.0
            ],
            [
                19205.0,
                4.207
            ],
            [
                19395.0,
                8.414
            ]
        ],
        "bids": [
            [
                18986.0,
                1.0
            ],
            [
                18824.0,
                4.207
            ],
            [
                18634.0,
                8.414
            ]
        ],
        "marketCode": "BTC-oUSD-SWAP-LIN",
        "timestamp": "1665454814328"
    },
    "action": "partial"
}
```

**Channel Update Frequency:** 100ms

This order book depth channel sends a snapshot of the entire order book every 50ms.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe` |
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
args | LIST | Yes | List of individual markets `<depth>:<marketCode>` e.g: `[depthL10:BTC-oUSD-SWAP-LIN]`, valid book sizes are: `depthL5` `depthL10` `depthL25` |

<sub>**Channel Update Fields**</sub>

Fields | Type | Description|
-------------------------- | -----| -------------|
table | STRING | `depthL10` |
data | DICTIONARY |
seqNum | INTEGER | Sequence number of the order book snapshot |
asks| LIST of floats | Sell side depth; <ol><li>price</li><li>quantity</li> |
bids| LIST of floats | Buy side depth; <ol><li>price</li><li>quantity</li> |
marketCode | STRING |marketCode |
timestamp| STRING | Millisecond timestamp |
action| STRING |  |
  

### Full Order Book

> **Request format**

```json
{
  "op": "subscribe",
  "tag": 103,
  "args": ["depth:BTC-oUSD-SWAP-LIN"]
}
```
```python
import websockets
import asyncio
import json

orderbook_depth = \
{
  "op": "subscribe",
  "tag": 103,
  "args": ["depth:BTC-oUSD-SWAP-LIN"]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(orderbook_depth))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())

```


> **Success response format**

```json
{
    "success": true,
    "tag": "103",
    "event": "subscribe",
    "channel": "depth:BTC-oUSD-SWAP-LIN",
    "timestamp": "1665454814275"
}
```

> **Order book depth channel format**

```json
{
    "table": "depth",
    "data": {
        "seqNum": 2166539633781384,
        "asks": [
            [
                19024.0,
                1.0
            ],
            [
                19205.0,
                4.207
            ],
            [
                19395.0,
                8.414
            ]
        ],
        "bids": [
            [
                18986.0,
                1.0
            ],
            [
                18824.0,
                4.207
            ],
            [
                18634.0,
                8.414
            ]
        ],
        "checksum": 3475315026,
        "marketCode": "BTC-oUSD-SWAP-LIN",
        "timestamp": 1665454814328
    },
    "action": "partial"
}
```

**Channel Update Frequency:** 100ms

This order book depth channel sends a snapshot of the entire order book every 100ms.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe` |
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
args | LIST | Yes | List of individual markets `<depth>:<marketCode>` e.g: `[depth:BTC-oUSD-SWAP-LIN]`|

<sub>**Channel Update Fields**</sub>

Fields | Type | Description|
-------------------------- | -----| -------------|
table | STRING | `depth` |
data | DICTIONARY |
seqNum | INTEGER | Sequence number of the order book snapshot |
asks| LIST of floats | Sell side depth; <ol><li>price</li><li>quantity</li> |
bids| LIST of floats | Buy side depth; <ol><li>price</li><li>quantity</li> |
checksum | INTEGER |checksum |
marketCode | STRING |marketCode |
timestamp| INTEGER | Millisecond timestamp |
action| STRING |  |


### Incremental Order Book

> **Request format**

```json
{
    "op": "subscribe",
    "tag": "test1",
    "args": [
        "depthUpdate:BTC-oUSD-SWAP-LIN"
    ]
}
```

> **Success response format**

```json
{
    "success": true,
    "tag": "test1",
    "event": "subscribe",
    "channel": "depthUpdate:BTC-oUSD-SWAP-LIN",
    "timestamp": "1665456142779"
}
```

> ** depth update channel format**

```json
{
    "table": "depthUpdate-diff",
    "data": {
        "seqNum": 2166539633794590,
        "asks": [],
        "bids": [],
        "checksum": 364462986,
        "marketCode": "BTC-oUSD-SWAP-LIN",
        "timestamp": "1665456142843"
    },
    "action": "increment"
}
```

```json
{
    "table": "depthUpdate",
    "data": {
        "seqNum": 2166539633794591,
        "asks": [
            [
                19042.0,
                1.0
            ]
        ],
        "bids": [
            [
                19003.0,
                1.0
            ]
        ],
        "checksum": 2688268653,
        "marketCode": "BTC-oUSD-SWAP-LIN",
        "timestamp": "1665456142843"
    },
    "action": "partial"
}
```

**Channel Update Frequency:** 100ms

Incremental order book stream

Usage Instructions:
1. Connect to websocket wss://api.opnx.com/v2/websocket
2. Subscribe to **depthUpdate** and you will get a message reply saying your subscription is successful
3. Afterwards you will get a snapshot of the book with **table:depthUpdate**
4. If you receive a reply with **table:depthUpdate-diff** first, keep it locally and wait for snapshot reply in step 3
5. The first incremental depthUpdate-diff message should have the same seqNum as the depthUpdate snapshot
6. After that, each new incremental update should have an incrementally larger seqNum to the previous update
7. The data in each event represents the absolute quantity for a price level. 
8. If the quantity is 0, remove the price level.


<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe` |
tag | INTEGER or STRING | No | If given it will be echoed in the reply |
args | LIST | Yes | List of individual markets `<depthUpdate>:<marketCode>` e.g: `["depthUpdate:BTC-oUSD-SWAP-LIN"]`|

<sub>**Channel Update Fields**</sub>

Fields | Type | Description|
-------------------------- | -----| -------------|
table | STRING | `depthUpdate-diff` `depthUpdate` |
data | DICTIONARY |
seqNum | INTEGER | Sequence number of the order book snapshot |
asks| LIST of floats | Sell side depth; <ol><li>price</li><li>quantity</li> |
bids| LIST of floats | Buy side depth; <ol><li>price</li><li>quantity</li> |
marketCode | STRING |marketCode |
checksum | LONG |  |
timestamp| STRING | Millisecond timestamp |
action| STRING | `partial` `increment` |


###  Best Bid/Ask 

> **Request format**

```json
{
    "op": "subscribe",
    "tag": "test1",
    "args": [
        "bestBidAsk:BTC-oUSD-SWAP-LIN"
    ]
}
```

> **Success response format**

```json
{
    "success": true,
    "tag": "test1",
    "event": "subscribe",
    "channel": "bestBidAsk:BTC-oUSD-SWAP-LIN",
    "timestamp": "1665456882918"
}
```

> ** depth update channel format**

```json
{
    "table": "bestBidAsk",
    "data": {
        "ask": [
            19045.0,
            1.0
        ],
        "checksum": 3790706311,
        "marketCode": "BTC-oUSD-SWAP-LIN",
        "bid": [
            19015.0,
            1.0
        ],
        "timestamp": "1665456882928"
    }
}
```

**Channel Update Frequency:** Real-time

This websocket subscription streams the best bid and ask in real-time. Messages are pushed on every best bid/ask update in real-time

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe` |
tag | INTEGER or STRING | No | If given it will be echoed in the reply |
args | LIST | Yes | List of individual markets `<bestBidAsk>:<marketCode>` e.g: `["bestBidAsk:BTC-oUSD-SWAP-LIN"]` |

<sub>**Channel Update Fields**</sub>

Fields | Type | Description|
-------------------------- | -----| -------------|
table | STRING | `bestBidAsk` |
data | DICTIONARY |
ask| LIST of floats | Sell side depth; <ol><li>price</li><li>quantity</li> |
bid| LIST of floats | Buy side depth; <ol><li>price</li><li>quantity</li> |
checksum | LONG |  |
marketCode | STRING |marketCode |
timestamp| STRING | Millisecond timestamp |


### Trade

> **Request format**

```json
{
  "op": "subscribe",
  "tag": 1,
  "args": ["trade:BTC-oUSD-SWAP-LIN"]
}
```
```python
import websockets
import asyncio
import json

trade = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["trade:BTC-oUSD-SWAP-LIN"]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(trade))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())

```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": ["trade:BTC-oUSD-SWAP-LIN"], 
  "success": True, 
  "tag": "1", 
  "timestamp": "1594299886880"
}
```

> **Trade channel format**

```json
{
  "table": "trade",
  "data": [ {
              "side": "buy",
              "tradeId": "2778148208082945",
              "price": "5556.91",
              "quantity": "5",
              "matchType": "MAKER",
              "marketCode": "BTC-oUSD-SWAP-LIN",
              "timestamp": "1594299886890"
            } ]
}
```

**Channel Update Frequency:** real-time, with every order matched event

This trade channel sends public trade information whenever an order is matched on the order book. 

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe` |
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
args | LIST | Yes | list of individual markets `trade:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `trade`
data | LIST of dictionary |
tradeId   | STRING    | Transaction Id|
price | STRING    | Matched price|
quantity|STRING   | Matched quantity|
matchType|STRING   | `TAKER` or `MAKER`, orders that match via the implied mechanism show as MAKERs in their respective markets|
side    |STRING   | Matched side|
timestamp| STRING | Matched timestamp|
marketCode| STRING | Market code |


### Ticker

> **Request format**

```json
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["ticker:all"]
}

OR

{
  "op": "subscribe", 
  "tag": 1,
  "args": ["ticker:OX-USDT", ........]
}
```
```python
import websockets
import asyncio
import json

ticker = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["ticker:all"]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(ticker))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": "<args value>",
  "success": True,
  "tag": "1",
  "timestamp": "1594299886890"
}
```

> **Channel update format**

```json
{
    "table": "ticker",
    "data": [
        {
            "last": "0",
            "open24h": "2.80500000",
            "high24h": "3.39600000",
            "low24h": "2.53300000",
            "volume24h": "0",
            "currencyVolume24h": "0",
            "openInterest": "0",
            "marketCode": "1INCH-USDT",
            "timestamp": "1622020931049",
            "lastQty": "0",
            "markPrice": "3.304",
            "lastMarkPrice": "3.304",
            "indexPrice": "3.304"
        },
        {
            "last": "0",
            "open24h": "2.80600000",
            "high24h": "3.39600000",
            "low24h": "2.53300000",
            "volume24h": "0",
            "currencyVolume24h": "0",
            "openInterest": "0",
            "marketCode": "1INCH-oUSD-SWAP-LIN",
            "timestamp": "1622020931046",
            "lastQty": "0",
            "markPrice": "3.304",
            "lastMarkPrice": "3.304",
            "indexPrice": "3.304"
        },
        ...
    ]
}
```

**Channel Update Frequency:** 500 ms

The ticker channel pushes live price and volume information about the contract.

The websocket will reply with the shown success response format for **each** ticker channel which has been successfully subscribed to.

The data array in the message from this ticker channel will contain a single JSON corresponding to one ticker subscription.

If you subcribe "ticker:all", you would get one whole message containing all markets.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
args | LIST | Yes | `ticker:all` or a list of individual markets `ticker:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `ticker`
data | LIST of dictionary |
marketCode    | STRING   | Market code |
last          | STRING   | Last traded price|
markPrice     | STRING   | Mark price|
open24h       | STRING   | 24 hour rolling opening price|
volume24h     | STRING   | 24 hour rolling trading volume in counter currency |
currencyVolume24h     | STRING   | 24 hour rolling trading volume in base currency|
high24h     | STRING   | 24 hour highest price|
low24h     | STRING   | 24 hour lowest price|
openInterest     | STRING   | Open interest|
lastQty     | STRING   | Last traded price amount|
timestamp   | STRING   | Millisecond timestamp|
lastMarkPrice| STRING | Previous mark price reading|
indexPrice | STRING | Index price |


### Candles

> **Request format**

```json
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["candles60s:BTC-oUSD-SWAP-LIN"]
}
```
```python
import websockets
import asyncio
import json

candles = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["candles60s:BTC-oUSD-SWAP-LIN"]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(candles))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": ["candles60s:BTC-oUSD-SWAP-LIN"], 
  "success": True, 
  "tag": "1", 
  "timestamp": "1594313762698"
}
```

> **Channel update format**

```json
{
  "table": "candle60s",
  "data": [ {
              "marketCode": "BTC-oUSD-SWAP-LIN",
              "candle": [
                "1594313762698", //timestamp
                "9633.1",        //open
                "9693.9",        //high
                "9238.1",        //low
                "9630.2",        //close
                "45247",         //volume in counter currency
                "5.3"            //volume in base currency
              ]
          } ]
}
```

**Channel Update Frequency**: 500ms

**Granularity**: 60s, 180s, 300s, 900s, 1800s, 3600s, 7200s, 14400s, 21600s, 43200s, 86400s

The candles channel pushes candlestick data for the current candle.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
args | LIST | Yes | list of individual candle granularity and market `candles<granularity>:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `candles<granularity>`
data | LIST of dictionary |
marketCode | STRING   | Market code |
candle | LIST of strings  | <ol><li>timestamp</li><li>open</li><li>high</li><li>low</li><li>close</li><li>volume in counter currency</li><li>volume in base currency</li></ol>


### Liquidation RFQ

> **Request format**

```json
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["liquidationRFQ"]
}
```
```python
import websockets
import asyncio
import json

liquidation = \
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["liquidationRFQ"]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(liquidation))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```


> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": "liquidationRFQ", 
  "success": True, 
  "tag": "1", 
  "timestamp": "1613774604469"
}
```

> **Channel update format**

```json
{
  "table": "liquidationRFQ",
  "data": [ {
              "marketCode": "BTC-oUSD-SWAP-LIN"
              "timestamp": "1613774607889"
          } ]
}
```

**Channel Update Frequency**: real-time, whenever there is planned position or collateral liquidation.
 
The liquidation RFQ (request for quotes) channel publishes a message 500ms before a liquidation event is due to occur.  A liquidation event can be classed as one of the following:-

* liquidation of a clients perp position (in the perp books)
* liquidation of a clients spot collateral (in the spot books)

The message will contain the market code and is designed to give liquidity providers and traders an opportunity to make a 2-way market for the upcoming liquidation event.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
args | Single element LIST| Yes | `liquidationRFQ`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `liquidationRFQ`
data | LIST of dictionary |
marketCode | STRING   | Market code of liquidation |
timestamp | STRING  |  Millisecond timestamp | 


### Market

> **Request format**

```json
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["market:all"]
}

OR

{
  "op": "subscribe", 
  "tag": 1,
  "args": ["market:OX-USDT", ........]
}
```
```python
import websockets
import asyncio
import json

market = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["market:all"]
}

url= 'wss://stgapi.opnx.com/v2/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(market))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": "<args value>",
  "success": True,
  "tag": "1",
  "timestamp": "1594299886890"
}
```

> **Channel update format**

```json
{
  "table": "market",
  "data": [ 
      {
          "marketId": "3001000000000",
          "marketCode": "OX-USDT",
          "name": "OX/USDT Spot",
          "referencePair": "OX/USDT",
          "base": "OX",
          "counter": "USDT",
          "type": "SPOT",
          "exclusive": "false",
          "tickSize": "0.001",
          "qtyIncrement": "0.1",
          "marginCurrency": "USDT",
          "contractValCurrency": "OX", 
          "upperPriceBound": "0.0495",
          "lowerPriceBound": "0.041",
          "marketPrice": "0.045",
      }, 
      ........
   ]
}
```

**Channel Update Frequency:** 1s

The market channel pushes live information about the market such as the current market price and the lower & upper sanity bounds as well as reference data related to the market.

The websocket will reply with the shown success response format for **each** market which has been successfully subscribed to.

If a subscription has been made to **market:all**, the data array in the message from this channel will contain a JSON **list** of all markets. Each JSON will contain information for each market seperately. Otherwise the data array will contain a single JSON corresponding to one market per market channel subscription.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe`
tag | INTEGER or STRING | No | If given it will be echoed in the reply and the max size of `tag` is 32 |
args | LIST | Yes | `market:all` or a list of individual markets `market:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `market`
data | LIST of dictionaries |
marketPrice | STRING   | Mark price|
qtyIncrement | STRING   | Quantity increment |
upperPriceBound | STRING   | Upper sanity price bound|
lowerPriceBound     | STRING   | Lower sanity price bound |
counter     | STRING   | 
type     | STRING   | 
marketId     | STRING   | 
referencePair     | STRING   | 
tickSize     | STRING   | Tick size |
marketPriceLastUpdated     | STRING | Millisecond timestamp|
contractValCurrency     | STRING   | 
name   | STRING   | 
marketCode   | STRING  | 
marginCurrency   | STRING |
base     | STRING   | 

## Other Responses

By subscribing to an authenticated websocket there may be instances when a REST method will also generate a websocket reponse in addition to the REST reply.  There are also some GUI commands which will generate a websocket reponse.

### Cancel All Open Orders

> **Success response format**

```json
{
  "event": "CANCEL",
  "submitted": True,
  "timestamp": "1612476498953"
}
```

Documentation for the REST method for cancelling **all** open orders for an account can be found here [Cancel All Orders](#delete-v3-orders-cancel).

In both these instances a successful action will generate the shown repsonse in an authenticated websocket.

This action can also be executed via the trading GUI using the **Cancel All** button on the **Open Orders** blotter for both Spot and Derivative markets.

## Error Codes

> **Failure response format**

```json
{
  "event": "<opValue>",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "success": False
}
```

Both subscription and order command requests sent via websocket can be rejected and the failure response will return an error code and a corresponding error message explaining the reason for the rejection.

Code | Error Message 
-----------| -------- 
05001| Your operation authority is invalid
20000| Signature is invalid 
20001| Operation failed, please contact system administrator 
20002| Unexpected error, please check if your request data complies with the specification. 
20003| Unrecognized operation
20005| Already logged in
20006| Quantity must be greater than zero 
20007| You are accessing server too rapidly 
20008| clientOrderId must be greater than zero if provided 
20009| JSON data format is invalid 
20010| Either clientOrderId or orderId is required 
20011| marketCode is required 
20012| side is required 
20013| orderType is required 
20014| clientOrderId is not long type 
20015| marketCode is invalid 
20016| side is invalid 
20017| orderType is invalid 
20018| timeInForce is invalid 
20019| orderId is invalid 
20020| stopPrice or limitPrice is invalid 
20021| price is invalid 
20022| price is required for LIMIT order 
20023| timestamp is required 
20024| timestamp exceeds the threshold 
20025| API key is invalid 
20026| Token is invalid or expired 
20027| The length of the message exceeds the maximum length 
20028| price or stopPrice or limitPrice must be greater than zero 
20029| stopPrice must be less than limitPrice for Buy Stop Order 
20030| limitPrice must be less than stopPrice for Sell Stop Order
20031 | The marketCode is closed for trading temporarily |
20032 | Failed to submit due to timeout in server side |
20033 | triggerType is invalid |
20034 | The size of tag must be less than 32 |
20034 | The size of tag must be less than 32 |
20050 | selfTradePreventionMode is invalid |
300001| Invalid account status xxx, please contact administration if any questions
300011| Repo market orders are not allowed during the auction window
300012| Repo bids above 0 and offers below 0 are not allowed during the auction window
100005| Open order not found
100006| Open order is not owned by the user
100008| Quantity cannot be less than the quantity increment xxx
100015| recvWindow xxx has expired
200050| The market is not active
710001| System failure, exception thrown -> xxx
710002| The price is lower than the minimum
710003| The price is higher than the maximum
710004| Position quantity exceeds the limit
710005| Insufficient margin
710006| Insufficient balance
710007| Insufficient position
000101| Internal server is unavailable temporary, try again later
000201| Trade service is busy, try again later
