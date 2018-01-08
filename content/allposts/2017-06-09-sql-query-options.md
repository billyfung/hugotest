---
title: SQL Query Options
subtitle: Time is fun
date: '2016-06-09'
slug: sql-query-options
---

# PostgreSQL

Everytime I discover something new about Postgres, or just use it in general,
I wonder why I didn't learn database stuff earlier. It has been by far the
most useful skill I've been learning recently, and although these days there
are many services available that make things easier, knowing SQL is a very
useful tool.

That said, in SQL there are often many different ways to obtain the same
result. To me, this is like solving a puzzle multiple ways, and I've always
enjoyed that analysis. I'll present without context 3 queries that all output
the same things.

```
    SELECT distinct distributor_price_category_code 
    FROM icp_detail
    EXCEPT 
    SELECT distinct pricing_code
    FROM vector_tariff
```

```
    SELECT distinct distributor_price_category_code 
    FROM icp_detail a
    WHERE 
        distributor_price_category_code NOT IN (SELECT distinct pricing_code
    FROM vector_tariff b)
```

```
    SELECT DISTINCT distributor_price_category_code 
    FROM icp_detail a
    WHERE 
        NOT EXISTS (SELECT 1
    FROM vector_tariff b 
    WHERE a.distributor_price_category_code = b.pricing_code)
```

As presented above, the first query makes use of the `EXCEPT` function in
Postgres, which allows you to essentially select two seperate sets, and then
take away those that are in the 2nd set.

The second query obtains the same result using a `NOT IN`, and it selects a
column and compares it against another selected column.

The last query uses a `NOT EXISTS` that takes a selected set where the two
columns are equal. The last query isn't as straightforward, and you might
think that

# Which one is best?

That's always the question that you will want to ask, especially if
performance is important. Obviously there will be a query that is faster than
the others, but how do you determine that?

The answer is to use `EXPLAIN ANALYZE` which is a Postgres tool that explains
how the query is run, and then it will also run it to determine how long the
query takes. `EXPLAIN` will display the query plan, and then `ANALYZE` will
run the query and tell you how long planning took, andhow long execution took.

Based on the 3 queries above, the 2nd query was the fastest for my situation.
It is important to note that the time depends on a number of factors, such as
complexity of the tables, size of the tables, and the information just to name
a few. As tables grow in size, it is very possible that using one query will
become slower than another option. I'll save it for another post, but reading
into the query plan from `EXPLAIN` is a useful way to understand how Postgres
breaks down the query, and then finds you the information you want.