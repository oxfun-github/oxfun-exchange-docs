# Change Log
**2023-03-22**

* Updated REST API [POST /v3/orders/place](#post-v3-orders-place)
    * Request params:
        * Added new field `displayQuantity`
        * Added new field `amount`
    * Response fields:
        * Added new field `displayQuantity`
        * Added new field `amount`


# Introduction

Welcome to Opnx's v2 application programming interface (API). Opnx's APIs provide clients programmatic access to control aspects of their accounts and to place orders on Opnx's trading platform. Opnx supports the following types of APIs:

* a WebSocket API
* a REST API

Using these interfaces it is possible to place both authenticated and unauthenticated API commands for public and prvate commands respectively.

To get started please register for a TEST account at `https://v2stg.opnx.com/register`


# API Key Management

An API key is required to make an authenticated API command.  API keys (public and corresponding secret key) can be generated via the Opnx GUI within a clients account. 

By default, API Keys are read-only and can only read basic account information, such as positions, orders, and trades. They cannot be used to trade such as placing, modifying or cancelling orders.

If you wish to execute orders with your API Key, clients must select the `Can Trade` permission upon API key creation.

API keys are also only bound to a single sub-account, defined upon creation. This means that an API key will only ever interact and return account information for a single sub-account.

# Rate Limit

Opnx's APIs allows our clients to access and control their accounts or view our market data using custom-written software. To protect the performance of the system, we impose certain limits:

Type                              |                            Limit |
--------------------------------- | -------------------------------- |
Rest API                          |                   100 per second |
Rest API                          |                  2500 per 5 mins |
Rest POST v2.1/delivery/orders    |                 2 per 10 seconds |
Initialising Websocket Connection |                   200 per minute |
Websocket API (Auth)              |                    50 per second |
Websocket API (No Auth)           |                     1 per second |
