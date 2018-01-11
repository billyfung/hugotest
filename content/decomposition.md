Signal Decomposition

First week back at work in 2018 and it's been full steam ahead. What I've been working with is time series seasonal decomposition. An example of this is if you have data on number of customers visiting a restaurant, per hour. As you would expect, factors like weekday vs weekend would affect the shape of the data. To generalise, you are taking a layered signal and then breaking it into it's components. Kind of like fixing/learning about your car engine.

![] plot of seasonality

Base R comes with a function called `decompose()` that will take your time series data, a period length, and then split the data into it's decomposed parts. The basic maths is:

```
y_t = T_t + S_t + R_t or
y_t = T_t * S_t * R_t
```

### Trend
This can range from a simple smoothing trend, used to remove outliers or a more complex Gaussian. I like to view this as the first order filter, mainly used to remove offsets and various 

### Seasonality
Something that occurs with a pattern. Weekly. Daily. Monthly. By extracting this component, we are able to understand what is left in the signal, hopefully revealing more insight into it. 

### Remainder / Uncertainty
If everything is done well, this would ideally be white noise. But nothing is ideal, so this can be used as a proxy to extend your model further. 

This is just an intro, will write more indepth into how to do this all in R on your own, without using the function. It is always good practice to break everything down, and build it yourself to prove your understand. Also it allows you to add in custom functions