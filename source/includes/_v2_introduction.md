# Change Log

# Introduction

OX.FUN offers a REST API and streaming WebSocket API that allows traders and developers to integrate their algorithms, trading strategies, risk management systems, and order execution software with our trading platform. 
OX.FUN's API emphasizes performance and security, enabling users to access the full range of our platform's features: order placement, account/subaccount management, and market data.

**REST API**

OX.FUN's REST API allows users to execute trades, manage their accounts, and access market data. With extensive documentation and sample python code, the REST API is easy to use and can be integrated with a wide variety of trading software. The REST API also offers SSL/TLS connections, rate limiting, and IP whitelisting, ensuring that all communication between the user and the server is secure.

**WebSocket API**

OX.FUN's WebSocket API provides real-time market data and order updates with ultra-low latency. With the WebSocket API, users can subscribe to real-time price updates and order book changes, enabling them to make faster and more informed trading decisions. The Websocket API also offers a real-time streaming interface for order placement and cancellation, enabling users to respond to market changes quickly and efficiently.

To get started register for a TEST account at [stg.ox.fun](https://stg.ox.fun) or a LIVE account at [ox.fun](https://ox.fun)


# API Key Management

An API key is required to make an authenticated API command.  API keys (public and corresponding secret key) can be generated via the OX.FUN GUI within a clients account. 

By default, API Keys are read-only and can only read basic account information, such as positions, orders, and trades. They cannot be used to trade such as placing, modifying or cancelling orders.

If you wish to execute orders with your API Key, clients must select the `Can Trade` permission upon API key creation.

API keys are also only bound to a single sub-account, defined upon creation. This means that an API key will only ever interact and return account information for a single sub-account.



# Rate Limit

OX.FUN's APIs allows our clients to access and control their accounts or view our market data using custom-written software. To protect the performance of the system, we impose certain limits:

Type                              |                            Limit |
--------------------------------- | -------------------------------- |
Rest API                          |                   100 per second |
Rest API                          |                  2500 per 5 mins |
Initialising Websocket Connection |                   200 per minute |
Websocket API (Auth)              |                    50 per second |
Websocket API (No Auth)           |                     1 per second |
