# Change Log
**2023-09-18**

* Added new REST API  [GET /v3/mm/rewards/history](#get-v3-mm-rewards-history)

* Added new REST API  [GET /v3/mm/rewards/summary](#get-v3-mm-rewards-summary)



**2023-07-27**
* Added [Self Trade Prevention Modes](#self-trade-prevention-modes) to WebSocket and REST place order requests

**2023-07-18**

* Added new REST API  [GET /v3/positions/largest](#get-v3-positions-largest)

* Added new REST API  [GET /v3/positions/liquidation/closest](#get-v3-positions-liquidation-closest)


**2023-07-10**

* Updated REST API [GET /v3/account](#get-v3-account)
    * Introduced `STANDARD` and `PORTFOLIO` account types
* Updated WebSocket API [Balance Channel](#balance-channel)
    * Introduced `STANDARD` and `PORTFOLIO` trade types

**2023-05-08**

* Updated REST API  [GET /v3/assets](#get-v3-assets)
    * Response fields:
        * Added new field  `loanToValueFactor`
        
             
* Added new REST API  [GET /v3/leverage/tiers](#get-v3-leverage-tiers)


**2023-04-03**

* Updated REST API  [DELETE /v3/orders/cancel](#delete-v3-orders-cancel)
    * Response fields:
        * Updated field `canceledAt` to  `closedAt`



**2023-03-31**

* Updated WebSocket Error Codes [Error Codes](#error-codes)
* Added new error code and message:
    * 000101    Internal server is unavailable temporary, try again later
    * 000201    Trade service is busy, try again later
    * 100008    Quantity cannot be less than the quantity increment xxx
    * 100015    recvWindow xxx has expired
    * 300001    Invalid account status xxx, please contact administration if any questions
    * 710001    System failure, exception thrown -> xxx
    * 710007    Insufficient position
    
    
* Updated error code and message:
    * 100005    Open order not found
    * 100006    Open order is not owned by the user
    * 200050    The market xxx is inactive
    * 710002    The price is lower than the minimum
    * 710003    The price is higher than the maximum
    * 710004    Position quantity exceeds the limit
    * 710005    Insufficient margin
    * 710006    Insufficient balance
    
    
* Updated REST API [GET /v3/orders/status](#get-v3-orders-status) and [GET /v3/orders/working](#get-v3-orders-working) and [POST /v3/orders/place](#post-v3-orders-place) and [DELETE /v3/orders/cancel](#delete-v3-orders-cancel)
    * Response fields:
        * Added new field  `amount`
        * Added new field  `displayQuantity`
        * Added new field  `triggerType`


**2023-03-28**

* Removed WebSocket Adapter
* Updated WebSocket API [Order Commands](#order-commands)
    * Response fields:
        * Added new field `limitPrice`
        * Added new field `triggerType`
* Updated WebSocket API Order Channel
    * Response fields:
        * Added new field `limitPrice`
        * Added new field `triggerType`
* Updated Websocket Ticker Channel
    * Response fields:
        * Added new field `indexPrice`
* Updated Websocket Market Channel
    * Response fields:
        * Removed field `listingDate`
        * Removed field `marketPriceLastUpdated`
        * Added new field `exclusive`
* Updated REST API [GET /v3/account](#get-v3-account) and [GET /v3/positions](#get-v3-positions)
    * Response fields:
        * Removed field `marginBalance`
        * Removed field `maintenanceMargin`
        * Removed field `marginRatio`
        * Removed field `leverage`
        * Removed field `riskRatio`
* Updated REST API [GET /v3/withdrawal](#get-v3-withdrawal)
    * Response fields:
        * Added `IN SWEEPING` enum to the `status` field
* Updated REST API [GET /v3/orders/status](#get-v3-orders-status) and [GET /v3/orders/working](#get-v3-orders-working)
    * Response fields:
        * Added new source enums to the `source` field
    
**2023-03-27**

* Add new OrderType `STOP_MARKET`
* Updated REST  API [POST /v3/orders/place](#post-v3-orders-place)
* Add WebSocket API [Place Stop Market Order](#place-stop-market-order)  WebSocket api

**2023-03-22**

* Updated REST  API [POST /v3/orders/place](#post-v3-orders-place)
* Updated WebSocket API [Order Commands](#order-commands) place/modify WebSocket api
    * Request params:
        * Added new field `displayQuantity`
        * Added new field `amount`
    * Response fields:
        * Added new field `displayQuantity`
        * Added new field `amount`

# Introduction

OPNX offers a REST API and streaming WebSocket API that allows traders and developers to integrate their algorithms, trading strategies, risk management systems, and order execution software with our trading platform. 
OPNX's API emphasizes performance and security, enabling users to access the full range of our platform's features: order placement, account/subaccount management, and market data.

**REST API**

OPNX's REST API allows users to execute trades, manage their accounts, and access market data. With extensive documentation and sample python code, the REST API is easy to use and can be integrated with a wide variety of trading software. The REST API also offers SSL/TLS connections, rate limiting, and IP whitelisting, ensuring that all communication between the user and the server is secure.

**WebSocket API**

OPNX's WebSocket API provides real-time market data and order updates with ultra-low latency. With the WebSocket API, users can subscribe to real-time price updates and order book changes, enabling them to make faster and more informed trading decisions. The Websocket API also offers a real-time streaming interface for order placement and cancellation, enabling users to respond to market changes quickly and efficiently.

To get started please register for a TEST account at [stg.opnx.com/register](https://stg.opnx.com/register) or a LIVE account at [opnx.com/register](https://opnx.com/register)


# API Key Management

An API key is required to make an authenticated API command.  API keys (public and corresponding secret key) can be generated via the Opnx GUI within a clients account. 

By default, API Keys are read-only and can only read basic account information, such as positions, orders, and trades. They cannot be used to trade such as placing, modifying or cancelling orders.

If you wish to execute orders with your API Key, clients must select the `Can Trade` permission upon API key creation.

API keys are also only bound to a single sub-account, defined upon creation. This means that an API key will only ever interact and return account information for a single sub-account.


# API Library

We recommend implementing your own API connector to minimize dependencies on external software, as well as optimize performance and security.

However, to help speed up your development we have produced a lightweight Java connector, with complete API coverage, supporting synchronous and asynchronous requests, and event streaming using WebSockets. 

[github.com/opnx-github/opnx-api-client](https://github.com/opnx-github/opnx-api-client)


# Rate Limit

Opnx's APIs allows our clients to access and control their accounts or view our market data using custom-written software. To protect the performance of the system, we impose certain limits:

Type                              |                            Limit |
--------------------------------- | -------------------------------- |
Rest API                          |                   100 per second |
Rest API                          |                  2500 per 5 mins |
Initialising Websocket Connection |                   200 per minute |
Websocket API (Auth)              |                    50 per second |
Websocket API (No Auth)           |                     1 per second |
