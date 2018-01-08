---
title: Cholesky decomposition
subtitle: Correlation
date: '2016-11-21'
slug: cholesky-corr
---

## Linear Algebra and probability distributions

Well those were two things I definitely thought that I wouldn't be needing
again after graduating university. Turns out I was very wrong, and I wish I
paid more attention learning probability, and applying linear algebra. In my
experience so far, the intuition that you can gain from studying probability
and linear algebra is actually quite useful in the field of anything
numerical. If you ever need to look at a large data sample, and figure out if
variables are correlated, or how to best classify the information, you will
most likely run into some form of probability and linear algebra.

Given information in a dataset, for example the `mtcars` dataset within R. We
can analyze the correlation coefficients of each variable by using the handy
`cor` function within R. ` df = mtcars hist(df$hp) cor_df = cor(df) `

The correlation matrix that is returned displays a matrix where each variable
is given a correlation coefficient that displays the dependence of each
variable pair permutation. The default method is using Pearson correlation
coefficients, which is a procedure which comes with it's set of assumptions.
Using the Pearson method comes from the assumption that the data you are
looking at is somewhat a normally distributed dataset and linearlly dependent.
This means that each variable in the dataset follows a normal distribution
roughly, and we can assume that each variable is linear in relation to the
other variables.

It's useful to look at the distributions of your data as part of your
exploratory analysis, this way it will give you insight into outliers if they
occur in the future. Or if your data starts off looking normal, and over time
there is a skewing trend towards another distribution.

## Cholesky decomposition

One of the most important things you learn without knowing (at least I didn't
know) in your introductary linear algebra class is how to decompose matricies.
LU decomposition is the basic one that everybody learns by hand, because this
method is easy to teach by handle, and easy to compute by hand. Decomposition
solves the equation of: ` A = Bx ` Solving for the matrix B will let you solve
for your variable of choice. But LU decomposition isn't actually the most
efficient, or the fastest if you are able to use a computer. This is where
something called the Cholesky decomposition comes into play. As an example,
using `mtcars` say we know that some model of cars are correlated with each
other, and we want to find out that correlation. A Honda Civic and a Toyota
Corolla are pretty similar right? For the sake of this example, say we had
more continuous information in the dataset.

```
    library(reshape2)
    df.wide = dcast(df, cyl+disp+hp ~ rownames(df), value.var = 'mpg')
```

Now the df.wide dataframe will contain the information that shows wide format
data of each car model and it's mpg. Using this information, we can make some
sort of statement where we are wondering about what kind of mpg we will get
given a car model, a cylinder number, a displacement number, and horsepower.

## Predicting mpg based on car model features

Given the wide format information, we can use that to create our correlaton
matrix. The correlation matrix is used in the Cholesky decomposition process,
to transform a matrix describing correlation coefficients into a lower
triangle matrix that is then used to project the correlation onto another
variable.

These variables in our case are going to be error distributions. This way, we
can make predictions on the mpg a car will get, look at the error distribution
of the car, and be able to create a joint distribution that contains the
correlations of the mpg error of each car model. While writing this I
understand that this probably sounds very confusing, it's because I'm not too
clear about it myself, and I haven't picked the best example to work with.

Where Cholesky decomposition is used widely is in the financial sector, where
they take correlation of products, and then use Cholesky decomposition in
order to form a joint distribution that gives correlation of errors, then
apply this joint distribution in simulating prices over a time frame. This
will allow for simulation to see how prices move, with correlation of products
built in.