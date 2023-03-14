# Change Log

**2022-06-28**

Added new REST API [GET /v3/orders/status](?json#rest-api-v3-orders-private-get-v3-orders-status) to get latest order status

**2022-05-25**

* Added a limit of max size `32` to the parameter `tag` in the associated Websocket APIs

**2022-05-20**

* Updated REST API [GET /v2/positions](?json#rest-api-methods-private-get-v2-positions)
    * Returned an empty array `[]` instead of `null` when no positions were found
* Updated REST API [GET /v2/positions/{instrumentId}](?json#rest-api-methods-private-get-v2-positions-instrumentid)
    * Returned an empty object `{}` instead of `null` when no positions were found

**2022-05-10**

* Updated REST API [GET /v3/withdrawal](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-withdrawal)
    * Returned an empty array `[]` instead of the "no result, please check your parameters" error message when no withdrawals were found

**2022-05-04**

* [Rate limit](?json#rest-api-v3-rate-limits) increased from `2/10s` to `4/10s` for the AMM POST requests

**2022-04-18**

* Updated REST API [POST /v3/AMM/redeem](?json#rest-api-v3-amm-private-post-v3-amm-redeem)
    * Added new request parameter `accountId` and new response field `accountId`
* Updated REST API [GET /v3/AMM](?json#rest-api-v3-amm-private-get-v3-amm)
    * Added new response field `volume`


**2022-04-15**

* Updated REST API [GET /v3/withdrawal](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-withdrawal)
    * Added new parameter `id`
* Updated REST API [POST /v3/withdrawal](?json#rest-api-v3-deposits-amp-withdrawals-private-post-v3-withdrawal)
    * Added new response field `id`


**2022-04-12**

* Added new REST API [GET /v3/AMM/hash-token](?json#rest-api-v3-amm-private-get-v3-amm-hash-token) to get AMM hashTokens

**2022-03-18**

* Updated REST API [GET /v3/assets](?json#rest-api-v3-market-data-public-get-v3-assets)
    * Changed response field `name` to `asset`
    * Changed response field `network` to `networkList` and wrap network in the list
    * Added new response field `tokenId`
    * Added new response field `transactionPrecision`
    * Added new response field `isWithdrawalFeeChargedToUser`

**2022-03-11**

* Added new REST API [GET /v3/tickers](?json#rest-api-v3-market-data-public-get-v3-tickers) to get tickers
* Added new REST API [GET /v3/candles](?json#rest-api-v3-market-data-public-get-v3-candles) to get candles
* Added new REST API [GET /v3/depth](?json#rest-api-v3-market-data-public-get-v3-depth) to get depth
* Added new REST API [GET /v3/flexasset/balances](?json#rest-api-v3-flex-assets-public-get-v3-flexasset-balances) to get flexAsset balances
* Added new REST API [GET /v3/flexasset/positions](?json#rest-api-v3-flex-assets-public-get-v3-flexasset-positions) to get flexAsset positions
* Added new REST API [GET /v3/flexasset/yields](?json#rest-api-v3-flex-assets-public-get-v3-flexasset-yields) to get flexAsset yields


**2022-02-21**

* Updated REST API [GET /v2/flex-protocol/balances/{flexProtocol}](?json#rest-api-methods-public-get-v2-flex-protocol-balances-flexprotocol)
    * Added new response field `markPrice`

**2022-01-05**

* One account can only place up to 50 orders per second via websocket. Affected websocket APIs:
    * [Place Limit Order](?json#websocket-api-order-commands-place-limit-order)
    * [Place Market Order](?json#websocket-api-order-commands-place-market-order)
    * [Place Stop Limit Order](?json#websocket-api-order-commands-place-stop-limit-order)
    * [Modify Order](?json#websocket-api-order-commands-modify-order)

**2021-12-23**

* Added new REST API [GET /v3/account](?json#rest-api-v3-account-amp-wallet-private-get-v3-account) to get account information
* Added new REST API [POST /v3/AMM/create](?json#rest-api-v3-amm-private-post-v3-amm-create) to create AMM
* Added new REST API [POST /v3/AMM/redeem](?json#rest-api-v3-amm-private-post-v3-amm-redeem) to redeem AMM
* Added new REST API [GET /v3/AMM](?json#rest-api-v3-amm-private-get-v3-amm) to get AMM information
* Added new REST API [GET /v3/AMM/balances](?json#rest-api-v3-amm-private-get-v3-amm-balances) to get AMM balances
* Added new REST API [GET /v3/AMM/positions](?json#rest-api-v3-amm-private-get-v3-amm-positions) to get AMM positions
* Added new REST API [GET /v3/AMM/orders](?json#rest-api-v3-amm-private-get-v3-amm-orders) to get AMM orders
* Added new REST API [GET /v3/AMM/trades](?json#rest-api-v3-amm-private-get-v3-amm-trades) to get AMM trades

**2021-12-21**

* Updated REST API [GET /v2/trades/{marketCode}](?json#rest-api-methods-private-get-v2-trades-marketcode)
    * Changed parameter `startTime` default to 24 hours ago
    * Changed parameter `endTime` default to current time
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET /v2.1/delivery/orders](?json#rest-api-methods-private-get-v2-1-delivery-orders)
    * Added parameter `limit` with default 200 max 500
    * Added parameter `startTime` with default 24 hours ago
    * Added parameter `endTime` with default current time
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET /v2/mint/{asset}](?json#rest-api-methods-private-get-v2-mint-asset)
    * Changed parameter `startTime` default to 24 hours ago
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET /v2/redeem/{asset}](?json#rest-api-methods-private-get-v2-redeem-asset)
    * Changed parameter `startTime` default to 24 hours ago
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET v2/borrow/{asset}](?json#rest-api-methods-private-get-v2-borrow-asset)
    * Changed parameter `startTime` default to 24 hours ago
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET v2/repay/{asset}](?json#rest-api-methods-private-get-v2-repay-asset)
    * Changed parameter `startTime` default to 24 hours ago
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET /v2/publictrades/{marketCode}](?json#rest-api-methods-public-get-v2-publictrades-marketcode)
    * Changed parameter `startTime` default to 24 hours ago
    * Changed parameter `endTime` default to current time
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET /v2/flex-protocol/trades/{flexProtocol}/{marketCode}](?json#rest-api-methods-public-get-v2-flex-protocol-trades-flexprotocol-marketcode)
    * Changed parameter `startTime` default to 24 hours ago
    * `startTime` and `endTime` must be within 7 days of each other
* Updated REST API [GET /v2/flex-protocol/delivery/orders/{flexProtocol}](?json#rest-api-methods-public-get-v2-flex-protocol-delivery-orders-flexprotocol)
    * Changed parameter `startTime` default to 24 hours ago
    * `startTime` and `endTime` must be within 7 days of each other

**2021-11-30**

* Updated websocket API [Place Limit Order](?json#websocket-api-order-commands-place-limit-order)
    * Added parameter `timestamp` and paramter `recvWindow`
* Updated websocket API [Place Market Order](?json#websocket-api-order-commands-place-market-order)
    * Added parameter `timestamp` and paramter `recvWindow`
* Updated websocket API [Place Stop Limit Order](?json#websocket-api-order-commands-place-stop-limit-order)
    * Added parameter `timestamp` and paramter `recvWindow`
* Updated websocket API [Place Batch Orders](?json#websocket-api-order-commands-place-batch-orders)
    * Added parameter `timestamp` and paramter `recvWindow`
* Updated websocket API [Modify Order](?json#websocket-api-order-commands-modify-order)
    * Added parameter `timestamp` and paramter `recvWindow`
* Updated websocket API [Modify Batch Orders](?json#websocket-api-order-commands-modify-batch-orders)
    * Added parameter `timestamp` and paramter `recvWindow`

**2021-11-22**

* Added new REST API [POST /v3/flexasset/mint](?json#rest-api-v3-flex-assets-private-post-v3-flexasset-mint) to mint
* Added new REST API [POST /v3/flexasset/redeem](?json#rest-api-v3-flex-assets-private-post-v3-flexasset-redeem) to redeem
* Added new REST API [GET /v3/flexasset/mint](?json#rest-api-v3-flex-assets-private-get-v3-flexasset-mint) to get mint history
* Added new REST API [GET /v3/flexasset/redeem](?json#rest-api-v3-flex-assets-private-get-v3-flexasset-redeem) to get redeem history
* Added new REST API [GET /v3/flexasset/earned](?json#rest-api-v3-flex-assets-private-get-v3-flexasset-earned) to get earn history
* Added new REST API [GET /v3/markets](?json#rest-api-v3-market-data-public-get-v3-markets) to get all markets
* Added new REST API [GET /v3/assets](?json#rest-api-v3-market-data-public-get-v3-assets) to get all assets

**2021-10-30**

Here come the API V3!

* Added new REST API [GET /v3/deposit-addresses](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-deposit-addresses) to get deposit addresses
* Added new REST API [GET /v3/deposit](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-deposit) to get deposit history
* Added new REST API [GET /v3/withdrawal-addresses](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-withdrawal-addresses) to get withdrawal addresses
* Added new REST API [GET /v3/withdrawal](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-withdrawal) to get withdrawal history
* Added new REST API [POST /v3/withdrawal](?json#rest-api-v3-deposits-amp-withdrawals-private-post-v3-withdrawal) to withdraw
* Added new REST API [GET /v3/withdrawal-fee](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-withdrawal-fee) to get estimated withdrawal fee
* Added new REST API [POST /v3/transfer](?json#rest-api-v3-deposits-amp-withdrawals-private-post-v3-transfer) to transfer
* Added new REST API [GET /v3/transfer](?json#rest-api-v3-deposits-amp-withdrawals-private-get-v3-transfer) to get transfer history

**2021-10-26**

* Updated REST API [GET /v2/funding-payments](?json#rest-api-methods-private-get-v2-funding-payments)
    * Changed parameter startTime default from 0 to 500 hours ago
    * Changed parameter limit default & max from 50 to 500

**2021-09-23**

* Updated REST API [GET /v2.1/orders](?json#rest-api-methods-private-get-v2-1-orders)
    * Changed startTime default from 0 to 24 hours ago
    * The range between startTime and endTime should be less than or equal to 7 days(`endTime - startTime <= 7days`)

**2021-09-17**

* Updated REST API [GET /v2/depth/{marketCode}/{level}](?json#rest-api-methods-public-get-v2-depth-marketcode-level)
    * Removed last two elements in depth

**2021-08-23**

* Updated REST API [GET /v2.1/orders](?json#rest-api-methods-private-get-v2-1-orders)
  * Added new order status `OrderPartiallyMatched` which will be returned when the order has been partially filled
  * Return orders with distinct order id and latest order status

**2021-08-11**

* Added new REST API [POST /v2/AMM/create](?json#rest-api-methods-private-post-v2-amm-create) to create AMM
* Added new REST API [POST /v2/AMM/redeem](?json#rest-api-methods-private-post-v2-amm-redeem) to redeem AMM
* Added new REST API [GET /v2/AMM](?json#rest-api-methods-private-get-v2-amm) to get AMM
* Added Python example of request for websocket API [Market](?python#websocket-api-subscriptions-public-market)
* Added Python example of request for websocket API [Liquidation RFQ](?python#websocket-api-subscriptions-public-liquidation-rfq)
* Added Python example of request for websocket API [Candles](?python#websocket-api-subscriptions-public-candles)
* Added Python example of request for websocket API [Ticker](?python#websocket-api-subscriptions-public-ticker)
* Added Python example of request for websocket API [Trade](?python#websocket-api-subscriptions-public-trade)
* Added Python example of request for websocket API [Orderbook Depth](?python#websocket-api-subscriptions-public-orderbook-depth)
* Added Python example of request for websocket API [Order Channel](?python#websocket-api-subscriptions-private-order-channel)
* Added Python example of request for websocket API [Position Channel](?python#websocket-api-subscriptions-private-position-channel)
* Added Python example of request for websocket API [Balance Channel](?python#websocket-api-subscriptions-private-balance-channel)

**2021-08-09**

* Added Python example of request for websocket API [Place Batch Orders](?python#websocket-api-order-commands-place-batch-orders)
* Added Python example of request for websocket API [Cancel Order](?python#websocket-api-order-commands-cancel-order)
* Added Python example of request for websocket API [Cancel Batch Orders](?python#websocket-api-order-commands-cancel-batch-orders)
* Added Python example of request for websocket API [Modify Order](?python#websocket-api-order-commands-modify-order)
* Added Python example of request for websocket API [Modify Batch Orders](?python#websocket-api-order-commands-modify-batch-orders)
* Added link to Opnx's [Historical Data](?python#historical-data) from third party API

**2021-08-05**

* Added Python example of request for websocket API [Limit Order](?python#websocket-api-order-commands-limit-order)
* Added Python example of request for websocket API [Market Order](?python#websocket-api-order-commands-market-order)
* Added Python example of request for websocket API [Stop Limit Order](?python#websocket-api-order-commands-stop-limit-order)

**2021-08-03**

* Added new REST API [GET /v2/funding-payments](?json#rest-api-methods-private-get-v2-funding-payments) to get funding payments

**2021-07-28**

* Added new REST API [GET /v2/borrow/{asset}](?json#rest-api-methods-private-get-v2-borrow-asset) to get borrow history
* Added new REST API [GET /v2/repay/{asset}](?json#rest-api-methods-private-get-v2-borrow-asset) to get repay history
* Added new REST API [GET /v2/borrowingSummary](?json#rest-api-methods-private-get-v2-borrowingsummary) to get borrowing summary


**2021-05-24**

* Updated websocket API [Ticker](?json#websocket-api-subscriptions-public-ticker)
    * If you subcribe "ticker:all", you would get one whole message containing all markets but not individual message for each market any more
    * With lower channel update frequency 500 ms instead of 100 ms

**2021-05-21**

* Added new REST API [GET /v2/flex-protocol/balances/{flexProtocol}](?json#rest-api-methods-public-get-v2-flex-protocol-balances-flexprotocol) to get flexAsset balances
* Added new REST API [GET /v2/flex-protocol/positions/{flexProtocol}](?json#rest-api-methods-public-get-v2-flex-protocol-positions-flexprotocol) to get get flexAsset positions
* Added new REST API [GET /v2/flex-protocol/orders/{flexProtocol}](?json#rest-api-methods-public-get-v2-flex-protocol-orders-flexprotocol) to get flexAsset orders
* Added new REST API [GET /v2/flex-protocol/trades/{flexProtocol}/{marketCode}](?json#rest-api-methods-public-get-v2-flex-protocol-trades-flexprotocol-marketcode) to get flexAsset trades
* Added new REST API [GET /v2/flex-protocol/delivery/orders/{flexProtocol}](?json#rest-api-methods-public-get-v2-flex-protocol-delivery-orders-flexprotocol) to get flexAsset delivery orders

**2021-05-13**

* Added new REST API [GET /v2/ping](?json#rest-api-methods-public-get-v2-ping) to check API service status
* Added new REST API [POST /v2/mint](?json#rest-api-methods-private-post-v2-mint) to mint
* Added new REST API [GET /v2/mint/{asset}](?json#rest-api-methods-private-get-v2-mint-asset) to get mint history
* Added new REST API [POST /v2/redeem](?json#rest-api-methods-private-post-v2-redeem) to redeem
* Added new REST API [GET /v2/redeem/{asset}](?json#rest-api-methods-private-get-v2-redeem-asset) to get redeem history

**2021-05-12**

* Updated REST API [GET /v2/positions](?json#rest-api-methods-private-get-v2-positions) & [GET /v2/positions/{instrumentId}](?json#rest-api-methods-private-get-v2-positions-instrumentid)
    * Response fields:
        * Field `lastUpdated` get changed from LONG type to STRING type
* Updated REST API [GET/v2/all/markets](?json#rest-api-methods-public-get-v2-all-markets)
    * Response fields:
        * Field `name` get changed with new naming rule, e.g. from `BAND/USD Spot` to `BAND/USD`

**2021-04-28**

* Updated REST API [GET /v2.1/orders](?json#rest-api-methods-private-get-v2-1-orders)
    * Request params:
        * Moved request parameters from body to URL
        * Changed default & max of `limit` from `1000` to `100`
        * Changed default of `startTime` from `24 hours ago` to `0`
    * Response fields:
        * Data list in response are ordered by created time in descending
        * Added new field `lastTradedPrice`
        * Added new field `avgFillPrice`
        * Added new field `filledQuantity`
        * Added new field `avgLeg1Price`
        * Added new field `avgLeg2Price`
        * Added new field `orderOpenedTimestamp`
        * Added new field `orderModifiedTimestamp`
        * Added new field `orderClosedTimestamp`
        * Changed field `matchId` to `matchIds` with more changes, please refer to the response example
        * Changed field `fees`, please refer to the response example
        * Changed field `isTriggered` from `True/False` to `"true"/"false"`
        * Removed field `timestamp` in data list

**2021-04-19**

* Added new response fields `positionPnl` and `estLiquidationPrice` to websocket API [Position Channel](?json#websocket-api-subscriptions-private-position-channel)
* Added new response field `estLiquidationPrice` to REST APIs [GET /v2/positions](?json#rest-api-methods-private-get-v2-positions) and [GET /v2/positions/{instrumentId}](?json#rest-api-methods-private-get-v2-positions-instrumentid)

**2021-03-31**

* Updated REST API [GET /v2/candles/{marketCode}](?json#rest-api-methods-public-get-v2-candles-marketcode) to get historical candles of active and expired markets, big change on request parameters and response fields, so please take it as new

**2021-03-18**

* Added [OrderClosed Failure Message](?json#websocket-api-subscriptions-private-order-channel-orderclosed-failure) to the websocket API documentation
* Added [OrderModified Failure Message](?json#websocket-api-subscriptions-private-order-channel-ordermodified-failure) to the websocket API documentation

**2021-03-10**

* Added new REST API [POST /v2/orders/place](?json#rest-api-methods-private-post-v2-orders-place) to place orders
* Added new REST API [POST /v2/orders/modify](?json#rest-api-methods-private-post-v2-orders-modify) to modify orders
* Added new REST API [DELETE /v2/orders/cancel](?json#rest-api-methods-private-delete-v2-orders-cancel) to cancel orders
* Added new REST API [GET /v2/depth/{marketCode}/{level}](?json#rest-api-methods-public-get-v2-depth-marketcode-level) to get order book depth by marketCode and level


**2021-02-20**

* Added new websocket API [Liquidation RFQ](?json#websocket-api-subscriptions-public-liquidation-rfq), a subsription channel publishing upcoming liquidations
* Added new websocket API [Market](?json#websocket-api-subscriptions-public-market), a subsription channel publishing market information for each order book

**2021-02-05**

* Added new REST API [GET /v2.1/orders](?json#rest-api-methods-private-get-v2-1-orders) to get all orders of current user
* Added new REST API [GET /v2/candles](?json#rest-api-methods-public-get-v2-candles) to get candlestick data for the current candle
* Updated REST API [GET /v2/orders](?json#rest-api-methods-private-get-v2-orders)
    * type of timestamp changed from INTEGER to STRING
    * changed field name from `remainQuantity` to `remainingQuantity`
    * type of `orderCreated` and `lastModified` and `lastTradeTimestamp` changed from STRING TO INTEGER

**2021-01-26**

* Correction to REST API for [GET /v2/accountinfo](?json#rest-api-methods-private-get-v2-accountinfo), [GET /v2/balances](?json#rest-api-methods-private-get-v2-balances) and [GET /v2/positions](?json#rest-api-methods-private-get-v2-positions)

**2021-01-18**

* Updated rate limits to reflect new Rest and unauthenticated websocket rate limits

**2020-12-14**

* Added new websocket API [Place Batch Orders](?json#websocket-api-order-commands-place-batch-orders)
* Added new websocket API [Cancel Batch Orders](?json#websocket-api-order-commands-cancel-batch-orders)
* Added new websocket API [Modify Batch Orders](?json#websocket-api-order-commands-modify-batch-orders)

**2020-12-10**

* Added market order and stop-limit order websocket API details

**2020-09-25**

* Added physical delivery API endpoints
* Added position WebSocket channel
* Added balance WebSocket channel
* Added GET /v2/accountinfo
* Added GET /v2/ticker
* Added rate limits
* Added guidance for maintaining connections

**2020-07-25**

* Added GET /v2/publictrades/{marketCode}
* Added GET /v2/trades/{marketCode}

**2020-06-16**

* Added general guidance for getting a login/password to create tokens
* Updated some addresses for REST API

**2020-06-15**

* First beta version of API endpoints. Websocket and REST

# Introduction

Welcome to Opnx's v2 application programming interface (API). Opnx's APIs provide clients programmatic access to control aspects of their accounts and to place orders on Opnx's trading platform. Opnx supports the following types of APIs:

* a WebSocket API
* a REST API

Using these interfaces it is possible to place both authenticated and unauthenticated API commands for public and prvate commands respectively.

To get started please register for a TEST account at `https://v2stg.opnx.com/user-console/register`


# API Key Management

An API key is required to make an authenticated API command.  API keys (public and corresponding secret key) can be generated via the CoinFLEX GUI within a clients account. 

By default, API Keys are read-only and can only read basic account information, such as positions, orders, and trades. They cannot be used to trade such as placing, modifying or cancelling orders.

If you wish to execute orders with your API Key, clients must select the `Can Trade` permission upon API key creation.

API keys are also only bound to a single sub-account, defined upon creation. This means that an API key will only ever interact and return account information for a single sub-account.

# Historical Data

Opnx's historical L2 order book data (depth data), can be found at [https://docs.tardis.dev/historical-data-details/opnx] (https://docs.tardis.dev/historical-data-details/opnx) (third party API), which includes historical market data details - instruments, data coverage and data collection specifics for all of our instruments since 2020-07-14.

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
