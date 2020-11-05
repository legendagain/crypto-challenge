# Coding Challenge

In this challenge, we create a vector of portfolio weights using mean-variance optimization.

* **Source:** [Huobi DM](https://huobiapi.github.io/docs/dm/v1/en/#get-k-line-data)
* **Type:** Current quarterly futures contracts
* **Timeframe:** 2019-11-01T05:00:00+00:00 to 2019-11-15T23:00:00+00:00
* **Frequency:** 1 hour

## Getting Started

The notebook `SingAlliance Challenge.ipynb` is self-sufficient, and has detailed inline documentation. Jupyter Notebook, which is required for viewing and editing of the code, is available [standalone](https://jupyter.org/install) or [packaged with Anaconda](https://www.anaconda.com/distribution/).

### Libraries Used

Use `pip install` to acquire the following third-party libraries:
* requests
* pandas
* numpy
* scipy
* statsmodels
* matplotlib
* seaborn

## How to use

Once you have the above installed, run the entire code. The notebook is self-sufficient and only relies on the [Huobi DM K-Line REST API](https://huobiapi.github.io/docs/dm/v1/en/#get-k-line-data). However, Huobi DM only exposes the last 2,000 instances of market data - hence it would be ideal to store the data locally and read from CSV - else the code's results will not be reproducible in the future.

While the code has inline Jupyter Markdown documentation, some of the key functionality is explained here:

### Getting Historical Data

The function `get_historical_data()` accepts a `ticker` and a `period`:
* `ticker` has the format **[Coin Code]_[CW/NW/CQ]**, where CW denotes this week; NW next week; and CQ the current quarter's futures contract
* `period` represents the timeframe of each data instance (1min, 5min, 15min, 30min, 60min, 1hour, 4hour, 1day, 1mon)

It then retrieves all available 2,000 instances of market data from Huobi DM REST API, computes the %-change `ret` and `timestamp`, and returns a pandas DataFrame.

You can find more information on the parameters for Huobi DM API [here](https://huobiapi.github.io/docs/dm/v1/en/#get-k-line-data).

## Mean-variance Optimization

With returns of all 3 cryptocurrencies in our `df_all`, we compute `mean_returns` and `covariance_matrix`. These matrices are instrumental in computing and optimizing portfolio return and volatility. The metrics are also re-expressed as monthly, to allow for a more intuitive interpretation.

With reference to Chapter 11: Statistics in Yves Hilpisch's [*Python for Finance*](https://www.amazon.com/Python-Finance-Mastering-Data-Driven-ebook/dp/B07L8NMW2P), the following optimization helper functions are defined:
* `statistics()` takes in a *1 × n* vector of `weights`, and returns an array of [`return`, `volatility` and `sharpe`].
* `port_return()`, `port_volatility()` and `port_sharpe()` each take in a *1 × n* vector of weights and calls `statistics()`, then returns the most suitable metric. They are as a function to minimize in `scipy`.

Each constraint is explained below:
* `{'type': 'eq', 'fun': lambda x: np.sum(abs(x)) - 1. }`: sum of absolute weights must equal one
* `{'type': 'ineq', 'fun': lambda x: statistics(x)[0] }`: expected return must be non-negative
* `{'type': 'eq', 'fun': lambda x: statistics(x)[0] - t_ret }`: expected return must equal the target return `t_ret`

The `bounds` are defined as *(-1, 1)* for each asset, indicating short-selling is possible. Changing them to *(0, 1)* would mean long-only, but is inapplicable in our situation as all expected returns are negative.

### Results

The code will plot an efficient frontier, with the return-volatility characteristics of the Max Sharpe and Min Volatility portfolios plotted as stars on the graph.

![Efficient Frontier](https://github.com/legendagain/crypto-challenge/blob/master/efficient-frontier.png)

Additionally, weights for these two portfolios will be exported to `portfolio_weights.csv`.

|     | Max Sharpe| Min Volatility  |
| ----|-----------| ----------------|
| BTC | -0.287    | -0.406          |
| XRP | -0.381    | -0.201          |
| LTC |  0.332    |  0.393          |
