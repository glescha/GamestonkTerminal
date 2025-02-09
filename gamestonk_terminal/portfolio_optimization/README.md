# Portfolio Optimization

This menu aims to discover optimal portfolios for selected stocks.

Currently we support two methods:

* [Property Weighting](#weighting)
    * Equal Weights
    * Market Cap Weighting
    * Divident Yield Weighting
* [Mean-Variance-Optimization](#eff_front)
    * Max sharpe ratio
    * Minimum volatility
    * Maximum returns at given risk level
    * Minimum risk level at a given return 

## Procedure
There are three ways to load stocks to be analyzed. 
* add
* select
* from ca menu
###add
Adds selected tickers to the menu to be considered

````
usage: add ticker1,ticker2,ticker3,...
````
###select
Clears current list and loads selected ticker
````
usage: select ticker1,ticker2,ticker3,...
````

### ca menu
From the ca menu, the loaded ticker and the selected similar tickers can be loaded by entering the `> po` menu.

From this `po` menu, you can return to `ca` using `ca`, which will keep your loaded stock, but reset the selected stocks from that menu.

Once your stocks are listed, you can select one of the options.
## Property weighted <a name="weighting"></a>
* equal weights
* market cap weighted
* dividend yield weighted
### equal weights
Returns an equally weighted portfolio where the weights are 1/number of stocks.
````
equal_weight [-v --value VALUE] [--pie] 
````
* -v/--value If provided, this represents an actual allocation amount for the portfolio.  Defaults to 1, which just returns the weights.
* --pie Flag that displays a pie chart of the allocations.

### market cap weighted
Returns portfolio values that are weighted by their relative Market Cap.
````
mkt_cap [-v --value VALUE] [--pie]
````
* -v/--value If provided, this represents an actual allocation amount for the portfolio.  Defaults to 1, which just returns the weights.
* --pie Flag that displays a pie chart of the allocations.

###dividend yield weighted
Returns portfolio values that are weighted by relative Dividend Yield.
````
div_yield [-v --value VALUE] [--pie]
````
* -v/--value If provided, this represents an actual allocation amount for the portfolio.  Defaults to 1, which just returns the weights.
* --pie Flag that displays a pie chart of the allocations.

## Mean Variance Optimization<a name="eff_front"></a>

These approaches are based off of the efficient frontier approach, which is meant to solve the following optimization problem.

<img src="https://latex.codecogs.com/svg.image?w^TS&space;w" title="w^TS w" />

With constraints:

<img src="https://latex.codecogs.com/svg.image?w^TR&space;>&space;R^*" title="w^TR > R^*" />

\
<img src="https://latex.codecogs.com/svg.image?w_1&plus;w_2&plus;...w_n&space;=&space;1" title="w_1+w_2+...w_n = 1" />

Where S is the covariance matrix between stocks and R is the expected returns.  The condition that all weights add up to 1
just implies that you want to have a net long portfolio (with no margin).  
A long-short portfolio can have negative weights and usually wants to have everything add up to 0 for a market-neutral strategy.

Currently, we do not allow for changing risk models or adding constraints.  If there is something specific, please submit a feature request, or if you can
write it, feel free to add a PR!

All of our current implementations use the [PyPortFolioOpt](#https://pyportfolioopt.readthedocs.io/en/latest/index.html) package.

### max_sharpe
The sharpe ratio is defined as 

(Mean Returns - Risk Free Rate)/(Standard Deviation of Returns)

The usage is:
````
max_sharpe [-p PERIOD] [-v --value VALUE] [--pie] 
````
* -p/--period Amount of time to retrieve data from yfinance. Options are: 1d,5d,1mo,3mo,6mo,1y,2y,5y,10y,ytd,max and it defaults to 3mo.
* -v/--value If provided, this represents an actual allocation amount for the portfolio.  Defaults to 1, which just returns the weights.
* --pie Flag that displays a pie chart of the allocations.

### min_vol
This portfolio minimizes the total volatility, which also means it has the smallest returns among the efficient frontier.
The usage is:
````
min_vol [-p PERIOD] [-v --value VALUE] [--pie]
````
* -p/--period Amount of time to retrieve data from yfinance. Options are: 1d,5d,1mo,3mo,6mo,1y,2y,5y,10y,ytd,max and it defaults to 3mo.
* -v/--value If provided, this represents an actual allocation amount for the portfolio.  Defaults to 1, which just returns the weights.
* --pie Flag that displays a pie chart of the allocations.

### eff_risk
This portfolio maximizes the returns at a given risk tolerance
The usage is:
````
eff_risk [-p PERIOD] [-r --risk RISK_LEVEL] [-v --value VALUE] [--pie]
````
* -p/--period Amount of time to retrieve data from yfinance. Options are: 1d,5d,1mo,3mo,6mo,1y,2y,5y,10y,ytd,max and it defaults to 3mo.
* -r/--risk Risk tolerance.  Default is 0.1 (10%)
* -v/--value If provided, this represents an actual allocation amount for the portfolio.  Defaults to 1, which just returns the weights.
* --pie Flag that displays a pie chart of the allocations.

### eff_ret
This portfolio minimizes the risk at a given return level
The usage is:
````
eff_ret [-p PERIOD] [-r --return] [-v --value VALUE] [--pie]
````
* -p/--period Amount of time to retrieve data from yfinance. Options are: 1d,5d,1mo,3mo,6mo,1y,2y,5y,10y,ytd,max and it defaults to 3mo.
* -r/--return.  Desired return.  Default is 0.1 (10%)
* -v/--value If provided, this represents an actual allocation amount for the portfolio.  Defaults to 1, which just returns the weights.
* --pie Flag that displays a pie chart of the allocations.

### show_eff
This function plots random portfolios based on their risk and returns and shows the efficient frontier.
The usage is:
````
show_eff [-p PERIOD]  [-n N_PORTFOLIOS]
````
* -p/--period Amount of time to retrieve data from yfinance. Options are: 1d,5d,1mo,3mo,6mo,1y,2y,5y,10y,ytd,max and it defaults to 3mo.
* -n Number of portfolios to simulate.

##Sample Usage
In this example, we generate weights for a list of 6 stocks using the eff_ret command.  This optimization looks to maximize returns 
at a given risk level.  We start by adding the stocks we want to analyze:
````
select aapl,amzn,msft,f,gm,ge
````
Which shows:
````
Current Tickers: GE, GM, AMZN, AAPL, F, MSFT
````
To perform the optimization, we will set a target return of 25% (.25).  Given how stocks performed during COVID,
there is a lot of volatility, so many optimizations may not get low volatility.  The module also returns annualized volatility,
so the number is your portfolio volatility * `sqrt(252`.  This optimization will be (including a pie chart!).  Note we could also supply a different
time period, which changes the expected returns and historical volatility, which changes the optimization.  We could also specify a dollar
amount that you wish to allocate using the `-v` flag.
````
eff_ret -r .25 --pie
````
The console will show (numbers will vary based on when this is done).
![console](https://user-images.githubusercontent.com/18151143/114740311-bd429c80-9d17-11eb-90e2-97430781431a.png)

And the pie chart:

![yummypie](https://user-images.githubusercontent.com/18151143/114740289-b9167f00-9d17-11eb-9c29-470785b21d09.png)

Note that since `AAPL` had zero allocation, it was omitted from the chart.