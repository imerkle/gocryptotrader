# GoCryptoTrader Backtester: Config package

<img src="/backtester/common/backtester.png?raw=true" width="350px" height="350px" hspace="70">


[![Build Status](https://travis-ci.org/thrasher-corp/gocryptotrader.svg?branch=master)](https://travis-ci.org/thrasher-corp/gocryptotrader)
[![Software License](https://img.shields.io/badge/License-MIT-orange.svg?style=flat-square)](https://github.com/thrasher-corp/gocryptotrader/blob/master/LICENSE)
[![GoDoc](https://godoc.org/github.com/thrasher-corp/gocryptotrader?status.svg)](https://godoc.org/github.com/thrasher-corp/gocryptotrader/backtester/config)
[![Coverage Status](http://codecov.io/github/thrasher-corp/gocryptotrader/coverage.svg?branch=master)](http://codecov.io/github/thrasher-corp/gocryptotrader?branch=master)
[![Go Report Card](https://goreportcard.com/badge/github.com/thrasher-corp/gocryptotrader)](https://goreportcard.com/report/github.com/thrasher-corp/gocryptotrader)


This config package is part of the GoCryptoTrader codebase.

## This is still in active development

You can track ideas, planned features and what's in progress on this Trello board: [https://trello.com/b/ZAhMhpOy/gocryptotrader](https://trello.com/b/ZAhMhpOy/gocryptotrader).

Join our slack to discuss all things related to GoCryptoTrader! [GoCryptoTrader Slack](https://join.slack.com/t/gocryptotrader/shared_invite/enQtNTQ5NDAxMjA2Mjc5LTc5ZDE1ZTNiOGM3ZGMyMmY1NTAxYWZhODE0MWM5N2JlZDk1NDU0YTViYzk4NTk3OTRiMDQzNGQ1YTc4YmRlMTk)

## Config package overview

### What does the config package do?
The config package contains a set of structs which allow for the customisation of the GoCryptoTrader Backtester when running.
The GoCryptoTrader Backtester runs from reading config files (`.strat` files by default under `/examples`).


### What does Simultaneous Processing mean?
GoCryptoTrader Backtester config files may contain multiple `ExchangeSettings` which defined exchange, asset and currency pairs to iterate through a period of time.

If there are multiple entries to `ExchangeSettings` and SimultaneousProcessing is disabled, then each individual exchange, asset and currency pair candle event is evaluated individually and does not know about other exchange, asset and currency pair data events. It is a way to test a singular strategy against multiple assets simultaneously. But it isn't defined as Simultaneous Processing
Simultaneous Signal Processing is a setting which allows multiple `ExchangeSettings` data events for a candle event to be considered simultaneously. This means that you can check if the price of BTC-USDT is 5% greater on Binance than it is on Kraken and choose to make signal a BUY event for Kraken and not Binance.

It allows for complex strategical decisions to be made when you consider the scope of the entire market at a given time, rather than in a vacuum when SimultaneousSignalProcessing is disabled.

### How do I customise the GoCryptoTrader Backtester?
See below for a set of tables and fields, expected values and what they can do

#### Config

| Key | Description |
| --- | ------|
| Nickname | A nickname for the specific config. When running multiple variants of the same strategy, use the nickname to help differentiate between runs |
| Goal | A description of what you would hope the outcome to be. When verifying output, you can review and confirm whether the strategy met that goal  |
| CurrencySettings | Currency settings is an array of settings for each individual currency you wish to run the strategy against. |
| StrategySettings | Select which strategy to run, what custom settings to load and whether the strategy can assess multiple currencies at once to make more in-depth decisions |
| PortfolioSettings | Contains a list of global rules for the portfolio manager. CurrencySettings contain their own rules on things like how big a position is allowable, the portfolio manager rules are the same, but override any individual currency's settings |
| StatisticSettings | Contains settings that impact statistics calculation. Such as the risk-free rate for the sharpe ratio |
| GoCryptoTraderConfigPath | The filepath for the location of GoCryptoTrader's config path. The Backtester utilises settings from GoCryptoTrader. If unset, will utilise the default filepath via `config.DefaultFilePath`, implemented [here](/config/config.go#L1460) |

#### Currency Settings

| Key | Description | Example |
| --- | ------- | ----- |
| ExchangeName | The exchange to load. See [here](https://github.com/thrasher-corp/gocryptotrader/blob/master/README.md) for a list of supported exchanges | `Binance` |
| Asset | The asset type. Typically, this will be `spot`, however, see [this package](https://github.com/thrasher-corp/gocryptotrader/blob/master/exchanges/asset/asset.go) for the various asset types GoCryptoTrader supports| `spot` |
| Base | The base of a currency | `BTC` |
| Quote | The quote of a currency | `USDT` |
| InitialFunds | The funds that the GoCryptoTraderBacktester has for the specific currency | `10000` |
| Leverage | This struct defines the leverage rules that this specific currency setting must abide by | `1` |
| BuySide | This struct defines the buying side rules this specific currency setting must abide by such as maximum purchase amount | - |
| SellSide | This struct defines the selling side rules this specific currency setting must abide by such as maximum selling amount | - |
| MinimumSlippagePercent | Is the lower bounds in a random number generated that make purchases more expensive, or sell events less valuable. If this value is 90, then the most a price can be affected is 10% | `90` |
| MaximumSlippagePercent | Is the upper bounds in a random number generated that make purchases more expensive, or sell events less valuable. If this value is 99, then the least a price can be affected is 1%. Set both upper and lower to 100 to have no randomness applied to purchase events | `100` |
| MakerFee | The fee to use when sizing and purchasing currency | `0.001` |
| TakerFee | Unused fee for when an order is placed in the orderbook, rather than taken from the orderbook | `0.002` |
| MaximumHoldingsRatio | When multiple currency settings are used, you may set a maximum holdings ratio to prevent having too large a stake in a single currency | `0.5` |

#### Strategy Settings

| Key | Description | Example |
| --- | ------- | --- |
| Name | The strategy to use. | `rsi` |
| UsesSimultaneousProcessing | This denotes whether multiple currencies are processed simultaneously with the strategy function `OnSimultaneousSignals`. Eg If you have multiple CurrencySettings and only wish to purchase BTC-USDT when XRP-DOGE is 1337, this setting is useful as you can analyse both signal events to output a purchase call for BTC. | `true` |
| CustomSettings | This is a map where you can enter custom settings for a strategy. The RSI strategy allows for customisation of the upper, lower and length variables to allow you to change them from 70, 30 and 14 respectively to 69, 36, 12 | `"custom-settings": { "rsi-high": 70, "rsi-low": 30, "rsi-period": 14 } ` |

#### PortfolioSettings

| Key | Description |
| --- | ------- |
| Leverage | This struct defines the leverage rules that this specific currency setting must abide by |
| BuySide | This struct defines the buying side rules this specific currency setting must abide by such as maximum purchase amount |
| SellSide | This struct defines the selling side rules this specific currency setting must abide by such as maximum selling amount |

#### StatisticsSettings

| Key | Description | Example |
| --- | ----------- | ------- |
| RiskFreeRate | The risk free rate used in the calculation of sharpe and sortino ratios | `0.03` |

#### APIData

| Key | Description | Example |
| --- | ----------- | ------- |
| DataType | Choose whether `candle` or `trade` data is used. If trades are used, they will be converted to candles | `trade` |
| Interval | The candle interval in `time.Duration` format eg set as`15000000000` for a value of `time.Second * 15` | `15000000000` |
| StartDate | The start date to retrieve data | `2021-01-23T11:00:00+11:00` |
| EndDate | The end date to retrieve data | `2021-01-24T11:00:00+11:00` |
| InclusiveEndDate | When enabled, the end date's candle is included in the results. ie `2021-01-24T11:00:00+11:00` with a one hour candle, the final candle will be `2021-01-24T11:00:00+11:00` to `2021-01-24T12:00:00+11:00` | `false` |

#### CSVData

| Key | Description | Example |
| --- | ----------- | ------- |
| DataType | Choose whether `candle` or `trade` data is used. If trades are used, they will be converted to candles | `candle` |
| Interval | The candle interval in `time.Duration` format eg set as`15000000000` for a value of `time.Second * 15` | `15000000000` |
| FullPath | The file to load  | `/data/exchangelist.csv` |

#### DatabaseData

| Key | Description | Example |
| --- | ----------- | ------- |
| DataType | Choose whether `candle` or `trade` data is used. If trades are used, they will be converted to candles | `trade` |
| Interval | The candle interval in `time.Duration` format eg set as`15000000000` for a value of `time.Second * 15` | `15000000000` |
| StartDate | The start date to retrieve data | `2021-01-23T11:00:00+11:00` |
| EndDate | The end date to retrieve data | `2021-01-24T11:00:00+11:00` |
| ConfigOverride | Override GoCryptoTrader's config database data with custom settings | `true` |
| InclusiveEndDate | When enabled, the end date's candle is included in the results. ie `2021-01-24T11:00:00+11:00` with a one hour candle, the final candle will be `2021-01-24T11:00:00+11:00` to `2021-01-24T12:00:00+11:00` | `false` |

#### LiveData

| Key | Description | Example |
| --- | ----------- | ------- |
| DataType | Choose whether `candle` or `trade` data is used. If trades are used, they will be converted to candles | `candle` |
| Interval | The candle interval in `time.Duration` format eg set as`15000000000` for a value of `time.Second * 15` | `15000000000` |
| APIKeyOverride | Will set the GoCryptoTrader exchange to use the following API Key | `1234` |
| APISecretOverride | Will set the GoCryptoTrader exchange to use the following API Secret | `5678` |
| APIClientIDOverride | Will set the GoCryptoTrader exchange to use the following API Client ID | `9012` |
| API2FAOverride | Will set the GoCryptoTrader exchange to use the following 2FA seed | `hello-moto` |
| RealOrders | Whether to place real orders. You really should never consider using this. Ever ever. | `true` |

##### Leverage Settings

| Key | Description | Example |
| --- | ----------- | ------- |
| CanUseLeverage | Allows the use of leverage | `false` |
| MaximumOrdersWithLeverageRatio | If the ratio of leveraged orders for a currency exceeds this, the order cannot be placed | `0.5` |
| MaximumLeverageRate | Orders cannot be placed with leverage over this amount | `100` |

##### Buy/Sell Settings

| Key | Description | Example |
| --- | ----------- | ------- |
| MinimumSize | If the order's quantity is below this, the order cannot be placed | `0.1` |
| MaximumSize | If the order's quantity is over this amount, it cannot be placed and will be reduced to the maximum amount | `10` |
| MaximumTotal | If the order's price * amount exceeds this number, the order cannot be placed and will be reduced to this figure | `1337` |

### Please click GoDocs chevron above to view current GoDoc information for this package

## Contribution

Please feel free to submit any pull requests or suggest any desired features to be added.

When submitting a PR, please abide by our coding guidelines:

+ Code must adhere to the official Go [formatting](https://golang.org/doc/effective_go.html#formatting) guidelines (i.e. uses [gofmt](https://golang.org/cmd/gofmt/)).
+ Code must be documented adhering to the official Go [commentary](https://golang.org/doc/effective_go.html#commentary) guidelines.
+ Code must adhere to our [coding style](https://github.com/thrasher-corp/gocryptotrader/blob/master/doc/coding_style.md).
+ Pull requests need to be based on and opened against the `master` branch.

## Donations

<img src="https://github.com/thrasher-corp/gocryptotrader/blob/master/web/src/assets/donate.png?raw=true" hspace="70">

If this framework helped you in any way, or you would like to support the developers working on it, please donate Bitcoin to:

***bc1qk0jareu4jytc0cfrhr5wgshsq8282awpavfahc***
