---
title: Slow API SQL Query 
subtitle: time it
date: '2017-04-29'
slug: slow-sql-query
---

## Situation

There's an API endpoint that runs a query against the Postgres database.
Simple stuff right? Nothing too out of the ordinary. Running the SQL query
locally against the database I get:

```
    localdb=# select * from normal_table;
    Time: 26535.223 ms
    26.535s
```

Which is again, nothing too out of the ordinary, 26s isn't too bad considering
the work that has to be done, it's a function that does a bunch of logic and
the tables are big.

Next up I use cURL to hit the endpoint locally to see how long it takes.

```
    curl -w "@curl-format.txt" -o /dev/null -s "http://localhost:5000/api/query?params=stuff"
        time_namelookup:  0.004
           time_connect:  0.005
        time_appconnect:  0.000
       time_pretransfer:  0.005
          time_redirect:  0.000
     time_starttransfer:  64.686
                        ----------
             time_total:  64.686s
```

So it takes quite a bit longer. And this is locally without having to send
information through the tubes across the internet.

## Poking around the DB layer

So the API endpoint is set up using Flask and the db connection is done using
the `flask_sqlalchemy` library. My initial thought is that maybe I'm not using
the db connection properly and there's some weirdness going on in the
abstraction layer. To test this, let's set up a direct connection using
`psycopg2` and see what we get back.

```
        time_namelookup:  0.013
           time_connect:  0.014
        time_appconnect:  0.000
       time_pretransfer:  0.014
          time_redirect:  0.000
     time_starttransfer:  62.986
                        ----------
             time_total:  62.986s
```

So again, this is locally, using a psycopg2 connection instead of SQL Alchemy
(which is built ontop of psycopg2). Looks like negligible difference to me,
although the time_connect is a bit longer.

## The problem

So the performance locally has some weird behavior, but 1 minute isn't really
the end of the world is it? The problem now lies in production, because that
query ends up causing a time out.

```
    [error] 12510#12510: *5281 upstream prematurely closed connection while reading response header from upstream, client: , server: , request: "GET /api/query?params=stuff
```

The default gunicorn worker timeout is set to 300s (I think), so this means
the query is taking longer than that which gives us the error. So back to the
drawing board to test out new theories. Perhaps it has to do with the
structure of the Postgres data object?

## The db query result

So far it appears the quick answer of the problem lying in the database
adapter library seems to be disproven to me. So the next theory is that it has
to do with the database object itself, maybe because the query uses Postgres
functions to create a table and pass it to the database adapter, something
weird is going on in there. The SQL query gives 2 parameters into a function
that then uses another function to return a table of 5000 rows. So there isn't
really that much data, but there is a bunch of logic that is done within those
functions.

So could it have something to do with the database query results? I'm not
completely sure how to test this, and it'll probably be for another blog post.
