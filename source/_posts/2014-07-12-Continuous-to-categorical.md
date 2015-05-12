---
layout: post
title: "Convert Coutinuous Variable to Categorical"
categories:
  - coding
tags:
  - continuous
  - categorical
  - stata
  - matlab
  - R
---

Suppose x is a continuous variable, we want to convert it to categorical. For example, we want to assign 1 to top 30%, 2 to middle 40%, and 3 to last 30%.

<!-- more -->

## Stata
We can use `xtile` or `egen cut` in stata.

```
set obs 1000
gen x =  rnormal()

xtile xc = x,  nquantiles(10)
tabulate xc

// equal frequency grouping intervals
egen xc2 = cut(x), group(10)
tabulate xc2
```

## R
`cut{base}` divides the range of x into intervals and codes the values in x according to which interval they fall. The leftmost interval corresponds to level one, the next leftmost to level two and so on.

```R
cut(x, 
    quantile(x, prob=seq(0,1,1/5)), 
    labels=c(1:5), 
    include.lowest=TRUE)
```

## Matlab
`ordinal` is what you want, [ref](http://www.mathworks.cn/cn/help/stats/categorize-numeric-data.html).
See also [`categorical`](http://www.mathworks.cn/cn/help/matlab/matlab_prog/ordinal-categorical-arrays.html).

```matlab
p = 0:.25:1;
x = randn(1000,1);
breaks = quantile(x, p);
xc = ordinal(x, {'Q1','Q2','Q3','Q4'}, [], breaks);
getlevels(xc)
```