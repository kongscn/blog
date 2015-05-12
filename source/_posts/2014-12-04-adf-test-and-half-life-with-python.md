---
layout: post
title:  "ADF Test and Half-Life with Python"
date:   2014-12-04
categories:
  - quant
tags:
  - Algorithmic Trading
  - ADF Test
  - half life
  - statsmodels
---

I am reading Chen's *Algorithmic Trading* and plan to rewrite the strategies in Python. This is the first post of this series. I'll put the core code here. For the result and comparison with the original Matlab code please refer to my ipython notebook [here](http://nbviewer.ipython.org/gist/kongscn/9de3a9bdac9bf5f632d9).

<!-- more -->

For short, ADF test can be achieved by [`statsmodels.tsa.stattools.adfuller`](http://statsmodels.sourceforge.net/devel/generated/statsmodels.tsa.stattools.adfuller.html), and [`statsmodels.regression.linear_model.OLS`](http://statsmodels.sourceforge.net/devel/generated/statsmodels.regression.linear_model.OLS.html) can be used to fit a linear regression.

```python
def adf_hl(price):
    """Calculate ADF test and Half-Life time.
    :param price: A pandas.Series of data.  
    :return: (adf test result, half-life, lookback)  
    """
    # ADF test
    adftest = ts.adfuller(price, maxlag=1)
    # Half-Life
    md = sm.OLS(price.diff(), sm.add_constant(price.shift()), missing='drop')
    mdf = md.fit()
    half_life = -np.log(2)/mdf.params[1]
    lookback = np.round(half_life)
    return adftest, half_life, lookback
```
