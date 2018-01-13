Seasonal Decomposition by hand in R

## De Trending
```
library(data.table)
library(zoo)
data.dt[,trend := zoo::rollmean(value, 4 fill=NA, align = "center")]
```

The first step is to detrend the data, which is essentially smoothing out the roughness a little bit. A simple way of doing this is using a rolling average. 

Once this is found, we will subtract or divide out the trend, depending on what kind of data you are dealing with

```
data.dt[,`:=`(detrended_a = value - trend, 
              detrended_m = value/trend)]
```

This part will output the data with the trend removed, and the seasonality still there
