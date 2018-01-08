---
title: Postgres Grouping and Ordering
subtitle: 
date: '2017-05-23'
slug: pg-group-order
---

```
    SELECT
        col1 as named_column,
        sum(col2),
        col3
    FROM table
    group by 1, col3
    order by col1
```

Sometimes I find that Postgres has unintuitive behavior when writing queries.
Or maybe I have non-conventional thinking when it comes to writing SQL. The
query as I have written above doesn't actually work, it won't execute because
`col1` is renamed to `named_column`.

How about this? ` SELECT col1 as named_column, sum(col2), col3 FROM table
group by col1, col3 order by col1 `

Well this runs at least, but then it doesn't actually do what I wanted. When
you rename the output using a SELECT AS, you will need to pass it onto the
group and order by. But if you're lazy, then this is the lazy (less typing)
solution:

```
    SELECT
        col1 as named_column,
        sum(col2),
        col3
    FROM table
    group by 1, col3
    order by 1
```

Downsides of this notation is that at a quick glance you don't see the column
names.