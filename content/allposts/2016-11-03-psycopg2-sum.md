---
title: Psycopg2 Decimal() sum
subtitle: 
date: '2016-11-03'
slug: psycopg2-decimal-sum
---

# Funky stuff

Just a quick note to jot down how when doing a `select sum(values) from table` from psycopg2, the values you get out from the sum will be a `Decimal()`.

This behavior made me have to look into my initial view because I forgot that certain variables could be duplicated, resulting in a wrong final sum. But the problem was that once I put the `sum()` into the query, something else broke when the query into the database resulted in a `Decimal()` when it was just
expecting an integer
