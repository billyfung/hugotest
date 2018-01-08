---
title: GAMS - General Algebraic Modeling System
subtitle: 
date: '2017-06-29'
slug: gams-intro
---

## Sometimes you have to use proprietary software

R and Python are both tools I use for exploring data, and building
mathematical models. More often than not, they are all you really need to use
and probably all you will ever encounter. But when you start dealing with
linear systems, where there isn't a single solution to a problem, but multiple
solutions that differ in minute amounts, you will need to start optimising for
an objective function.

Linear programming looks something like having multiple equations of lines as
constraints, and then adjusting the lines such that they cross at an optimal
point.

![Site][2]

The common textbook example is optimising supply and demand. You need to
transport products from one location to another, so you have multiple
products, multiple locations. The optimisation will be to transport as many
products as possible, to maximise the profit. Each product is located in a
location, with different costs to ship to different locations. Then you can
start to build the model of linear equations that constraint the system.

With these type of problems, you end up with a system of linear equations that
constrain the output. Luckily enough, very smart people have written solvers
that are built for certain types of systems specifically. So if you have a
linear programming problem that fits into a certain criteria, chances are
there is a solver that somebody has written already for it. The problem is
that it is usually not free.

## Hardness

After reading all that, linear programming doesn't really seem that difficult
to solve right? Linear programming is P hard in complexity, so this means that
the domain of the problem can be approached with algorithms that allow linear
programming to solved more easily than others. To make things a bit more
complex, another type of optimisation problem involves solving for Integer
Programming, or Mixed Integer Programming adds another level of complexity.
Integer Programming restricts all the variables to be integers, and results in
a NP-hard problem. With LP, polynomial time is achievable. And with MIP, being
NP-hard means there is no best case, only worst case exponential.

This means another couple PhD papers were written for proprietary solvers.

## Electricity Grid solving

All this has to do with my current work because I'm working on using GAMS, a
programming language that is written for building algebraic models. A lot of
solvers are written so that they can interface with GAMS, and GAMS itself is a
language that is based off set theory, so it is very mathematical in nature to
begin with. This makes it easier to build the scaffolding for your
mathematical model.

GAMS projects are usually structured so that you have you input data, your
variables, your objective functions, and then you tell it to solve.

An electricity grid is essentially managed by a very large number of
constraints. You have transmission constraints, like outages between certain
nodes. You have generation constraints, where there are a number of power
stations and only so much power that can be provided. You have distribution
constraints, which involve which the fact that only certain distributors exist
in certain areas. All this is a very generalised overview of what goes into
the model before it is solved.

Formulation of the problem requires consideration of the available solvers,
and of course the available time. It can be possible that you create a model
that will take too long to converge to a desireable outcome. For a power grid,
this model formulation usually comes down to being Mixed Integer Programming.

## Choosing a solver

So with GAMS already being proprietary software, I was out of my element. I
learned further that the best solvers are licensed. There are solvers included
within GAMS, so that's what I started off with. But I've learned that IBM's
CPLEX solver is the defacto industry standard for MIP problems, so look for a
review/comparison in the future.

[2]: /figures/linear_p.gif