---
title: Choosing an EC2 type
subtitle: AWS fun
date: '2017-08-31'
slug: ec2-options
---

# So many options, which one is best?

One of my current projects requires me to use an EC2 for some beefier
computation, but I actually don't know too much about EC2 apart from using the
most basic `t2` instances. Nothing I've worked on has really needed a server
with 60gb of RAM. Until now that is.

## How much memory do you need?

One of the first options that will limit you is memory for your usage. There
are so many different instance types depending on how fast you want your
CPU(s), but largely the difference in price is from storage. You can sacrifice
a couple ghz in speed for a couple GB in RAM, and vice versa.

Working in R, my dataset has 44,000,000 rows and 17 columns. Approximating all
the data to be 10 bytes, this gives us:

```
    bytes
    >>> 44000000*17*10
    7480000000
```

Which is rounded up to be 7.5GB of data. This sets a baseline of how much RAM
you will need at a minimum for your instance type. You will also need to
account for OS overhead, which will be something like ~.5GB

## What does your program do?

In my case, I'm building a tree based model, which I know will be memory
intensive. Apart from doing that, even setting up the data will require more
memory overhead for manipulation of data. This is why when dealing with large
datasets, using an appropriate memory-efficient data structure is very
important. For R, this means using `data.table`, which will let you perform in
place calculations without having to duplicate your data. Duplicating data
will come at a very high memory cost when you have large amounts of data.

This is where memory estimation gets a bit tricky. Depending on the algorithm
used for your model, this will determine how much memory is required.
Essentially, this is the space complexity of an algorithm, remember all that
stuff you learned from algorithms class but forgot? Yeah that stuff.

Each algorithm will be different, for my case I'm looking at gradient boosted
regression trees (xgboost). The algorithm will have built in measures to
optimise for memory, such as using [histograms][2] to optimise the growing of
the trees based on differences between parent and sibling.

With that said, knowing the baseline requirement, I would suggest running the
script for a baseline version of N trees and M depth, and then scale it
appropriate and approximate up. My parameters for the model are:

```
    ntrees = 500
    max_depth = 10
```

With this, I know that 7.5GB baseline will probably require more than 30GB RAM
just from thinking about the number of trees built. Of course this has a lot
to do with how the algorithm will work, but it's always better to have too
much RAM than too little.

## CPU requirements

Something that is also important is how fast you need your CPU to be.
Obviously there will be a point in which paying for an hour of a slower CPU
will be the better option if you need to satisfy a memory requirement.

For me, I went with [AWS R4 instance type][3] which is a memory optimised
instance that offers the best value in terms of decent CPU speed and large
amounts of RAM. The `r4.2xlarge` has 61GB of RAM and 8 vCPUs for $0.4 an hour.

That said, I'm still very new to all the offerings of AWS and the different
compute and memory types. I've tried using `c4` instance types in the past and
didn't feel like they were the right choice for my application because of how
little RAM they have.

[2]: https://github.com/dmlc/xgboost/issues/1950

[3]: https://aws.amazon.com/about-aws/whats-new/2016/11/introducing-amazon-ec2-r4-instances-the-next-generation-of-memory-optimized-instances/