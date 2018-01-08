---
title: Feature Extraction from dates
subtitle: Time is fun
date: '2017-06-14'
slug: feature-extraction
---

## Making something from nothing

More often than not, I deal with datasets where I don't have a multitude of
features to choose from. It's nothing like the Kaggle competitions where you
usually have too many features, and need to figure out how to cull down the
list. Sometimes I wonder what it would be like to work in an environment where
you have people working on building a feature complete dataset, and the
analysis team gets handed something where the data is as complete as possible.

Something I've been dealing with recent is a dataset where there are only 3
columns of data: `the date, numeric_A, numeric_B`. The plot of A vs B looks
something like:

![2017/06/Rplot.png][1]

So essentially I'm given time series data, with data that looks like it could
be linearly correlated. But how do we figure out more features, to even being
to explore?

### Seasonality

Of course this really depends on the data you're dealing with, but sometimes
you can extract a feature of what season the date is. This can simply be
looking at the month and estimating what the season it is, or you can put in
specific dates for each year and then bin them more accurately.

```R
getSeason <- function(dates) {
    d <- month(dates)

    ifelse (d >= 12 | d < 3, "Summer",
      ifelse (d >= 9 & d < 12, "Spring",
        ifelse (d >= 3 & d < 6, "Fall", "Winter")))
}
```

Is there any other nature of the data that you can quantise? Basically the
overview is to look at the year, month, and day components of the date and see
if you can sort them into bins. Then accordingly you should have a couple more
features that you can work with, and maybe combine to find some more.

[1]: /figures/Rplot.png