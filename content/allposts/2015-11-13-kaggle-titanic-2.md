---
title: Kaggle Titanic survival classification
subtitle: Who will survive?
date: '2015-11-29'
slug: kaggle-titanic-2
---

# Third Submission for Kaggle Titanic Competition

So I spent some more time on the Kaggle Titanic problem and looked more
indepth at the training data given. First off I combined both the training and
test sets in order to have a larger set of data to work off of. Using this
larger data set, and observing the data, it sort of starts to tell a story.
One of the most important aspects of data analysis is to really understand how
to use an observation. With the Titanic training set, we are given the names
of the passengers, but with some missing ages. Using the names of the
passengers, along with their titles, we can construct another feature that
tells us about the family. Knowing the family size and name will let us create
a theory that families will want to save their kids first. Also looking at the
observations from last time, we saw that certain ages and females were
favoured to survive. Putting this all into a new feature called `Dibs` let's
us assign a value to those who might be given priority onto a lifeboat.

![title][2]

So using the new features, I once again ran classification using Random
Forests with the `party` library in R. And the result was that my model
predicted `0.80383` which is a pretty high increase from the baseline no-
frills models I made originally. A 5% increase in accuracy is pretty good for
cleaning up the data, and adding some more features. Some further work that
could be done is to compute the missing ages with a better method. I think
somehow looking at the family size, and the ages of other family members would
be a good way to start filling in missing ages. Next would be to fill in the
single ages based off which class they are in, and then lastly just to fill it
in with imputation.

So far I haven't yet grouped models together to do ensemble predictions, but
that is a really strong approach to these competitions.

[2]: figures/title.png
