title: "Popular R Packages and Some Stats - from Jan. 2015 to May. 2015"
date: 2015-05-15 11:02:22
category: coding
tags:
  - R
  - packages
  - data.table
---

I'm switching to R recently, and one of the challenges is to get the right package from the mass world. Rstudio provides CRAN download logs from 2012-10-01, which contain all hits to <http://cran.rstudio.com/> related to packages.

<!-- more -->

Well, precisely we are going to find the popularity of packages among users who downloads from cran.rstudio.com, but I'll not emphasize this later on.

The stats are calculated using logs from 2015-01-01 to 2015-05-14.

This download script is inspired by Felix Schönbrodt in this [post](http://www.nicebread.de/finally-tracking-cran-packages-downloads/), and changed quite a log.

Download files
==============

``` r
library('lubridate')

dir.create("CRANlogs")
base_url = 'http://cran-logs.rstudio.com/'

start = as.Date('2015-01-01')
today = as.Date('2015-05-14')
all_days = seq(start, today, by = 'day')

# only download the files you don't have:
missing_days = setdiff(as.character(all_days), tools::file_path_sans_ext(dir("CRANlogs"), TRUE))

i = 0
total = length(missing_days)
for (dt in missing_days) {
  i = i + 1
  print(paste0("Processing ", i, "/", total, '...'))
  yearstr = year(as.Date(dt))
  url = paste0(base_url, yearstr, '/', dt, '.csv.gz')
  download.file(url, paste0('CRANlogs/', dt, '.csv.gz'))
}
```

Import data
===========

``` r
# This process can be quite slow.
# To speed up, try first load all files, 
# then combine and process the columns.

library(lubridate)
library(data.table)
library(dplyr)
library(tidyr)

DT = data.table()
files = list.files("CRANlogs", pattern='*.gz', full.names=TRUE)

for (file in files) {
    print(paste("Reading", file, "..."))
    dt = tbl_dt(read.csv(file, as.is=TRUE))
    dt = dt[, `:=`(datetime=ymd_hms(paste(date, time)),
                   r_version=as.factor(r_version),
                   r_arch=as.factor(r_arch),
                   r_os=as.factor(r_os),
                   package=as.factor(package),
                   date=NULL, time=NULL, ip_id=NULL, version=NULL)]
    DT = rbind(DT, dt)
}
rm(dt)
DT = tbl_dt(DT)
setkey(DT, package, country)
save(DT, file="CRANlogs/CRANlogs.RData")
gc()
```

Analysis
========

Overview
--------

Let's look through our table.

``` r
tail(DT)
```

    ## Source: local data frame [6 x 7]
    ## 
    ##   size r_version r_arch      r_os package country            datetime
    ## 1 2579     3.1.3 x86_64   mingw32    MCTM      ES 2015-05-14 17:21:06
    ## 2 2579     3.2.0   i386   mingw32    MCTM      GB 2015-05-14 19:56:34
    ## 3 2579     3.1.3   i386   mingw32    MCTM      HK 2015-05-14 11:42:55
    ## 4 2579     3.2.0 x86_64   mingw32    MCTM      JP 2015-05-14 16:30:23
    ## 5 2579     3.1.3 x86_64 linux-gnu    MCTM      US 2015-05-14 15:17:15
    ## 6 2579     3.1.3 x86_64 linux-gnu    MCTM      US 2015-05-14 19:16:00

``` r
dim(DT)
```

    ## [1] 41345731        7

``` r
sizes = tables()
```

    ##      NAME       NROW NCOL    MB
    ## [1,] DT   41,345,731    7 1,420
    ##      COLS                                                KEY            
    ## [1,] size,r_version,r_arch,r_os,package,country,datetime package,country
    ## Total: 1,420MB

We've got a 41345731 by 7 big table, using 1,420MB memory!

Now we will look into various statistics. Whenever the table is too long, we will show the top 10 rows only.

Most popular packages:
----------------------

``` r
library(dplyr)
library(ggplot2)
by_package = DT[, .N, keyby=package][order(-N)];
# Top downloads
by_package
```

    ## Source: local data table [7,845 x 2]
    ## 
    ##         package      N
    ## 1          Rcpp 614948
    ## 2       ggplot2 528963
    ## 3       stringr 474101
    ## 4        digest 471773
    ## 5          plyr 463810
    ## 6      reshape2 431532
    ## 7    colorspace 422595
    ## 8  RColorBrewer 407160
    ## 9    manipulate 356546
    ## 10       scales 350245
    ## ..          ...    ...

### A histogram of downloads of packages

Packages with download counts \<= 5e4 are ignored.

![](http://i1.tietuku.com/f8217b26ed139e74.png) 
![](http://i1.tietuku.com/af818e3ed768e1ab.png)

Downloads Time Series of Top 2 Rcpp and ggplot2
-----------------------------------------------

``` r
top2 = DT[.(c("Rcpp", "ggplot2"))]
top2[, `:=`(date=as.Date(datetime))];
```

    ## Source: local data table [1,143,911 x 8]
    ## 
    ##       size r_version r_arch         r_os package country
    ## 1  2778359     3.0.3   i386      mingw32    Rcpp      NA
    ## 2  3148107     3.1.1 x86_64      mingw32    Rcpp      NA
    ## 3  3147121     3.1.1 x86_64      mingw32    Rcpp      NA
    ## 4  2667601     3.1.2 x86_64      mingw32    Rcpp      NA
    ## 5  2170103     3.1.2 x86_64 darwin13.4.0    Rcpp      A1
    ## 6  3028965     3.1.2 x86_64    linux-gnu    Rcpp      A1
    ## 7  3029813     3.1.2 x86_64 darwin13.4.0    Rcpp      A1
    ## 8  3148107     3.1.2 x86_64 darwin13.4.0    Rcpp      A1
    ## 9  3148107     3.1.0 x86_64      mingw32    Rcpp      A1
    ## 10 3148107     3.0.3 x86_64      mingw32    Rcpp      A1
    ## ..     ...       ...    ...          ...     ...     ...
    ## Variables not shown: datetime (time), date (date)

``` r
top2[, `:=`(week=week(datetime))];
```

    ## Source: local data table [1,143,911 x 9]
    ## 
    ##       size r_version r_arch         r_os package country
    ## 1  2778359     3.0.3   i386      mingw32    Rcpp      NA
    ## 2  3148107     3.1.1 x86_64      mingw32    Rcpp      NA
    ## 3  3147121     3.1.1 x86_64      mingw32    Rcpp      NA
    ## 4  2667601     3.1.2 x86_64      mingw32    Rcpp      NA
    ## 5  2170103     3.1.2 x86_64 darwin13.4.0    Rcpp      A1
    ## 6  3028965     3.1.2 x86_64    linux-gnu    Rcpp      A1
    ## 7  3029813     3.1.2 x86_64 darwin13.4.0    Rcpp      A1
    ## 8  3148107     3.1.2 x86_64 darwin13.4.0    Rcpp      A1
    ## 9  3148107     3.1.0 x86_64      mingw32    Rcpp      A1
    ## 10 3148107     3.0.3 x86_64      mingw32    Rcpp      A1
    ## ..     ...       ...    ...          ...     ...     ...
    ## Variables not shown: datetime (time), date (date), week (int)

``` r
top2_stat = top2[, .N, by=.(package, date)]
qplot(x=date, y=N, data=top2_stat, geom='line', color=package)
```

![](http://i1.tietuku.com/0ddba7f959e2957a.png)

``` r
top2_stat = top2[, .N, by=.(package, week)]
qplot(x=week, y=N, data=top2_stat, geom='line', color=package)
```

![](http://i1.tietuku.com/266e1bfe299ec0f0.png)
Note: data for the last week is not complete, so ignore the sharp drop at the end.

R Popularity Among Countries
----------------------------

We use total downloads of all packages to estimate the popularity of R in a country.

``` r
by_country = DT[, .N, keyby=country][order(-N)];
# Top downloads
by_country
```

    ## Source: local data table [225 x 2]
    ## 
    ##    country        N
    ## 1       US 15334569
    ## 2       CN  3517081
    ## 3       DE  2087872
    ## 4       FR  1644046
    ## 5       GB  1619252
    ## 6       IN  1551254
    ## 7       JP  1394623
    ## 8       ES  1299567
    ## 9       CA  1015317
    ## 10      KR   885624
    ## ..     ...      ...

``` r
m = ggplot(by_country[1:25], aes(x=country, y=N))
m + geom_bar(stat="identity") + coord_flip()
```

![](http://i1.tietuku.com/ac3e10d8415c2495.png)

Popular R arch
--------------

``` r
by_r_arch = DT[, .N, keyby=r_arch][order(-N)];
# Top downloads
by_r_arch
```

    ## Source: local data table [17 x 2]
    ## 
    ##         r_arch        N
    ## 1       x86_64 31425351
    ## 2         i386  5356496
    ## 3           NA  4186643
    ## 4         i686   328437
    ## 5         i486    29263
    ## 6         i586     9029
    ## 7       armv7l     5964
    ## 8          arm     2268
    ## 9        amd64      928
    ## 10     powerpc      510
    ## 11      armv6l      324
    ## 12      000000      215
    ## 13   powerpc64      120
    ## 14 powerpc64le      109
    ## 15       sparc       72
    ## 16    armv5tel        1
    ## 17                    1

``` r
m = ggplot(by_r_arch[1:15], aes(x=r_arch, y=N))
title = "Downloads by R arch"
m + geom_bar(stat="identity") + scale_y_log10() + coord_flip() + labs(title=title)
```

![](http://i1.tietuku.com/7b70b896f185ee47.png)

R Versions
----------

``` r
by_r_version = DT[, .N, keyby=r_version][order(-N)];
# Top downloads
by_r_version
```

    ## Source: local data table [45 x 2]
    ## 
    ##    r_version        N
    ## 1      3.1.2 15588548
    ## 2      3.1.3  6815858
    ## 3      3.1.1  4538881
    ## 4      3.2.0  4511164
    ## 5         NA  4186643
    ## 6      3.0.2  2298008
    ## 7      3.1.0  1512864
    ## 8      3.0.3   662776
    ## 9      3.0.1   484936
    ## 10    2.15.2   123897
    ## ..       ...      ...

``` r
m = ggplot(by_r_version[1:10], aes(x=r_version, y=N))
title = "Downloads by R version"
m + geom_bar(stat="identity") + coord_flip() + labs(title=title)
```

![](http://i1.tietuku.com/ef0aeb29771b332b.png)

Popular R os
------------

``` r
by_r_os = DT[, .N, keyby=r_os][order(-N)];
# Top downloads
by_r_os
```

    ## Source: local data table [51 x 2]
    ## 
    ##            r_os        N
    ## 1       mingw32 23873832
    ## 2     linux-gnu  6681662
    ## 3            NA  4186643
    ## 4  darwin13.4.0  3502524
    ## 5  darwin10.8.0  2051859
    ## 6  darwin13.1.0   740996
    ## 7  darwin14.1.0   108816
    ## 8   darwin9.8.0    75073
    ## 9  darwin14.0.0    45906
    ## 10 darwin14.3.0    35019
    ## ..          ...      ...

``` r
m = ggplot(by_r_os[1:10], aes(x=r_os, y=N))
title = "Downloads by R os"
m + geom_bar(stat="identity") + coord_flip() + labs(title=title)
```

![](http://i1.tietuku.com/412a830d7e19320a.png)

Downloads by country\*r\_arch
-----------------------------

``` r
tt = DT[, .N, keyby=.(country, r_arch)]
top_country = by_country[1:10]
top_arch = by_r_arch[1:10]
selected = tt[.(top_country$country)]
setkey(selected, r_arch)
selected = selected[.(top_arch$r_arch)]

qplot(x=country, y=N, data=selected, fill=r_arch, geom='bar', stat = "identity")
```

![](http://i1.tietuku.com/b077596d54271358.png)

``` r
qplot(x=r_arch, y=N, data=selected, fill=country, geom='bar', stat = "identity")
```

![](http://i1.tietuku.com/c21dd8d0e10c792d.png)

Downloads by country\*r\_os
---------------------------

``` r
tt = DT[, .N, keyby=.(country, r_os)]
top_os = by_r_os[1:6]
selected = tt[.(top_country$country)]
setkey(selected, r_os)
selected = selected[.(top_os$r_os)]

qplot(x=country, y=N, data=selected, fill=r_os, geom='bar', stat = "identity")
```

![](http://i1.tietuku.com/5b128b6f7bd998ce.png)

``` r
qplot(x=r_os, y=N, data=selected, fill=country, geom='bar', stat = "identity")
```

![](http://i1.tietuku.com/b655c59da1e0a6da.png)
