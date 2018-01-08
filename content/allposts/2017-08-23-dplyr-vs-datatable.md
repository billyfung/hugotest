---
title: Hadleyverse vs data.table
subtitle: 
date: '2017-08-23'
slug: datatable-dplyr
---

# Is the Hadleyverse the only option?

One of the major downsides to learning R is figuring out which packages to
use. R comes with many standard packages, but then sometimes they aren't the
most intuitive way to handle your data, or they can be straight up bad. Many
people often direct new learners to use the packages writtein by [Hadley
Wickham][2], which I will say are great packages.

## dplyr

Most of the packages deal with handling data in an intuitive way that makes
your work easier to follow, and easier to read. One such package is `dplyr`,
which is used to manipulate data, mainly data frames.

Example of filtering a dataframe for rows where a column is a specific value.

```
    starwars %>%
      filter(species == "Droid")
```

The beauty in `dplyr` lies in being able to chain multiple operations
together, so you don't need to create multiple dataframes. In base R, I found
myself creating many `tmp` dataframes that I used to hold data as I was doing
multiple operations on it.

```
    starwars %>%
      group_by(species) %>%
      summarise(
        n = n(),
        mass = mean(mass, na.rm = TRUE)
      ) %>%
      filter(n > 1)
```

# But is there something faster?

One of the downsides of manipulating dataframes is that they aren't the most
nimble things to be moving around. I've been using `dplyr` and other
Hadleyverse packages for quite awhile now, but lately I've been dealing with
datasets that are much larger, like <5m rows. So the manipulation of data
becomes much slower, and grouping operations start to lag.

## data.table

This is a package that lets a `data.table` inherit from `data.frame`.
Essentially somebody created this package to improve on memory usage, along
with speed. There isn't any piping or elegant usage of writing code that makes
a datatable stand out, because that is a very subjective topic.

What it does do better is that I have found `fread/fwrite` to be way faster at
reading/writing files. Much much faster compared to `read_csv/write_csv`.

With operations that modify the data, data.table can update by reference, this
saves computation and memory cost in assigning the result back to a variable.

```
    DT[x > 0, y:= 'positive']
```

That simple code will update the data.table y column in-place. As table size
grows, data.table will do that a lot quicker compared to dplyr and data.frame.

The dplyr and data.frame equivalent.

```
    new_DF <- DF %>%
        mutate(y = replace(y, which(x > 0), 'positive'))
```

So far I am still quite new to data.table, but I have found that I am enjoying
using it because of the speed and memory gains, and I don't find that I wish
it had dplyr like syntax. Aggregating and joining is so much faster with large
tables that I wish I found out about data.table earlier so then I could gain
the intuitive for writing consistent code.

Every bit of optimisation counts when your datasets start growing, so learning
the fastest way first will save you time in the long run.

[2]: http://hadley.nz/