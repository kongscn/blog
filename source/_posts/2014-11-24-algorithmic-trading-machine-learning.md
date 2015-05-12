---
layout: post
title:  "Power Up Algorithmic Trading with Predictiving Modeling -- A Demo"
date:   2014-11-24
categories:
  - quant
tags: 
  - zipline 
  - machine learning
---

# Algorithmic Trading with Python

I found a Python package for quant trading called [zipline][zipline], which is currently used in production as the backtesting engine powering [Quantopian][quantopian].

<!-- more -->

> Zipline is a Pythonic algorithmic trading library. The system is fundamentally event-driven and a close approximation of how live-trading systems operate. Currently, backtesting is well supported, but the intent is to develop the library for both paper and live trading, so that the same logic used for backtesting can be applied to the market.

# Machine Learning with Python

The most popular package for machine learning in python is [scikit-learn][scikit-learn].

> `scikit-learn` is a Python module for machine learning built on top of SciPy and distributed under the 3-Clause BSD license.


<!-- # Why I prefer Python for various tasks

* Readable. Python code are usually very beautiful and easy to understand.
* Packages. Generally you'll alwayse find a package that fills your need there ready to pick up.
* 
*  -->

# Combine algorithmic trading with machine learning models

As it becomes more and more easy to collect data, we have so much information that is beyond our traditional understanding machnism of our brain. Also, the full past is summarized into the data. Rules like behind.

Algo trading is not new. Nor is machine learning, thought it's still quite young. But they have someting in common: they both try to draw the future according to history. Machine learning tries very hard to fit a flexible pattern on the past and use it to predict what we dont know yet, and in some circumstance its quite sucessfull. So I wondered the possibility of integrating this two methods together.


# Implementation

This post is not aimed at showing a successfull trading algo. Instead, I will just show the way how you can embed a machine learning model into a quant trading algo. With Python and the zipline environemnt, you only neeed several lines to do that. I do all of this on [Quantopian.com][quantopian], and I suggest you try it too.

First, let's look at a MACD demo provided by Quantopian, of which the algo will never lead you a attractive profit of course, just a demo of the code. (I changed it slightly, the original version can be found [here](https://www.quantopian.com/help#macd))

```python
# This example algorithm uses the Moving Average Crossover Divergence (MACD) indicator as a buy/sell signal.

# When the MACD signal less than 0, the stock price is trending down and it's time to sell.
# When the MACD signal greater than 0, the stock price is trending up it's time to buy.

# Because this algorithm uses the history function, it will only run in minute mode. 
# We will constrain the trading to once per day at market open in this example.

import talib
import numpy as np
import pandas as pd

# Setup our variables
def initialize(context):
    context.sec = symbol('AAPL')
    context.pct_per_stock = 0.5
    # Create a variable to track the date change
    context.date = None

def handle_data(context, data):
    stock = context.sec
    todays_date = get_datetime().date()
    
    # Do nothing unless the date has changed and its a new day,
    # as we can also run this algo in minute data.
    if todays_date == context.date:
        return
    
    # Set the new date
    context.date = todays_date
    
    # Load historical data(40 days here, including current day) for the stocks
    prices = history(40, '1d', 'price')
    
    # Create the MACD signal and pass in the three parameters: fast period, slow period, and the signal.
    # This is a series that is indexed by sids.
    macd = prices.apply(MACD, fastperiod=12, slowperiod=26, signalperiod=9)
    
    current_position = context.portfolio.positions[stock].amount

    # Close position for the stock when the MACD signal is negative and we own shares.
    if macd[stock] < 0 and current_position > 0:
        order_target(stock, 0)

        # Enter the position for the stock when the MACD signal is positive and 
        # our portfolio shares are 0.
    elif macd[stock] > 0 and current_position == 0:
        order_target_percent(stock, context.pct_per_stock)
           
        
    record(aapl=macd[symbol('AAPL')])
   

# Define the MACD function   
def MACD(prices, fastperiod=12, slowperiod=26, signalperiod=9):
    '''
    Function to return the difference between the most recent 
    MACD value and MACD signal. Positive values are long
    position entry signals 

    optional args:
        fastperiod = 12
        slowperiod = 26
        signalperiod = 9

    Returns: macd - signal
    '''
    macd, signal, hist = talib.MACD(prices, 
                                    fastperiod=fastperiod, 
                                    slowperiod=slowperiod, 
                                    signalperiod=signalperiod)
    return macd[-1] - signal[-1]


```

The code is simple to understand. If you are not familar with `zipline` I suggest you read this [tutorial][tutorial] first.

The besktest summary:

![alt text][pure-macd]


Now, let's say, we'd like a double check on the long/short signal. For example, we decid to apply a SVM regression model using the prices of related stocks, and predict current price. Only if the predicted price is greatter than 0.9*current_price(predicted a not-too-low price), we shot a long order. This logic is not likely to imporve our algo, but I just want to demonstrade that it's simple and easy to embed a machine leraning node into a trading algo. 


```python
# This example algorithm uses the Moving Average Crossover Divergence (MACD) indicator as a buy/sell signal.

# When the MACD signal less than 0, the stock price is trending down and it's time to sell.
# When the MACD signal greater than 0, the stock price is trending up it's time to buy.

# Because this algorithm uses the history function, it will only run in minute mode. 
# We will constrain the trading to once per day at market open in this example.

import talib
import numpy as np
import pandas as pd
from sklearn import svm

# Setup our variables
def initialize(context):
    context.sec = symbol('AAPL')
    context.auxsec = symbols('MSFT', 'YHOO', 'INTC')
    context.pct_per_stock = 0.5
    
    # Create a variable to track the date change
    context.date = None

def handle_data(context, data):
    sec = context.sec
    todays_date = get_datetime().date()
    
    # Do nothing unless the date has changed and its a new day.
    if todays_date == context.date:
        return
    
    # Set the new date
    context.date = todays_date
    
    # Load historical data for the stocks
    prices = history(40, '1d', 'price')
    hist = history(200, '1d', 'price')
    # Create the MACD signal and pass in the three parameters: fast period, slow period, and the signal.
    # This is a series that is indexed by sids.
    macd = prices.apply(MACD, fastperiod=12, slowperiod=26, signalperiod=9)
    
    
    # apply SVM learning model
    X = hist[context.auxsec].values[:-1,]
    y = hist[context.sec].values[:-1,]
    newx = hist[context.auxsec].values[-1,]
    
    predicted = ml(X, y, newx)
        
    current_position = context.portfolio.positions[sec].amount
    
    # Close position for the stock when the MACD signal is negative and we own shares.
    if macd[sec] < 0 and predicted < 1.1*data[sec].price and current_position > 0:
        order_target(sec, 0)
    
    # Enter the position for the stock when the MACD signal is positive and 
    # our port)folio shares are 0.
    elif macd[sec] > 0 and predicted > 0.9*data[sec].price and current_position == 0:
        order_target_percent(sec, context.pct_per_stock)
           
        
    record(macd=macd[sec], 
           real=data[sec].price,
           pred=predicted)
   

# Define the MACD function   
def MACD(prices, fastperiod=12, slowperiod=26, signalperiod=9):
    '''
    Function to return the difference between the most recent 
    MACD value and MACD signal. Positive values are long
    position entry signals 

    optional args:
        fastperiod = 12
        slowperiod = 26
        signalperiod = 9

    Returns: macd - signal
    '''
    macd, signal, hist = talib.MACD(prices, 
                                    fastperiod=fastperiod, 
                                    slowperiod=slowperiod, 
                                    signalperiod=signalperiod)
    return macd[-1] - signal[-1]

def ml(X, y, newx):
    '''
    A demo SVM regression
    '''
    clf = svm.SVR()
    clf.fit(X, y)
    return clf.predict(newx)[0]

```

The magic is a new function `ml` for training a SVM model and predicting, and then we  combine the result with original long/short decision rule.

![alt text][macd-ml]

Amazingly there's some improvement! No they don't count because we don't consider about various bias and the logics behind, but using algorithmic trading and machine learning is not that hart.

[Quantopian][quantopian] platform for backtesting provide a very detailed backtest result, as shown below. Personaly I found this platform very attractiving.


![result-1]

![result-2]

![result-3]

[zipline]: https://github.com/quantopian/zipline
[scikit-learn]: http://scikit-learn.org/
[quantopian]: https://www.quantopian.com/
[tutorial]: http://nbviewer.ipython.org/github/quantopian/zipline/blob/master/docs/tutorial.ipynb

[pure-macd]: http://i2.tietuku.com/5c8f7e3aa894bafd.png "MACD result"
[macd-ml]: http://i2.tietuku.com/dc472ee3a13e61d1.png "MACD & SVM"
[result-1]: http://i2.tietuku.com/f16d4d72aa3a25af.png
[result-2]: http://i2.tietuku.com/0d2d4432ec67b648.png
[result-3]: http://i2.tietuku.com/3990335dcef38db5.png
