---
title: Improving Multiple Inserts with Psycopg2
subtitle: execute_values
date: '2017-06-30'
slug: psycopg2-multiple-insert
---

## You don't know till you know

I find that whenever I'm in a rush to get something working, I have a bad
habit of just getting into writing code and getting the thing done. More often
than not, this results in messy code, and/or slow code. And then when I need
to do something similar again, I just re-use the original code and end up with
a potential compounded problem.

One of the more common tasks that I repeatedly do is take a bunch of files and
insert the data into a Postgres database. Up until recently, I haven't had to
deal with large enough datasets, so my poorly written code was still
acceptable in terms of execution time. Usually this means executing the insert
script locally on a test database first, and then on production. I'm using
Python 3 and the [Psycopg2][2] postgres driver.

## Psycopg2 execute and execute_values

The original code looked something like:

```
    def insert_data(filename, date):
    
        sql = """
        INSERT INTO test (a, b, c, d)
        VALUES (%s, %s, %s, %s)
        """
    
        with open(filename) as csvfile, get_cursor() as c:
            reader = csv.reader(csvfile)
            header = next(reader)
    
            for row in reader:
                n = row[0][2:]
                values = (row[1], date, n, row[2])
                c.execute(sql, values)
```

This looks like a pretty innocent looking insert function, it takes the file
loops over every row and inserts it into the table.

The refactored function looks like:

```
    def insert_data(filename, date):
    
        sql = """
        INSERT INTO test (a, b, c, d)
        VALUES %s
        """
    
        with open(filename) as csvfile, get_cursor() as c:
            reader = csv.reader(csvfile)
            header = next(reader)
            values_list = []
    
            for row in reader:
                n = row[0][2:]
                values = (row[1], date, n, row[2])
                values_list.append(values)
    
            execute_values(c, sql, values_list)
```

The difference between these two functions is the `execute` and
`execute_values`. Each time you use `execute`, psycopg2 does a complete return
trip from the database to your computer, so this means it will execute the row
INSERT to the database server, and then return. A functionality with Postgres
is that you can insert multiple rows at a time, and this is what
[`execute_values`][3] does.

Instead of inserting a single row query, the refactored version creates a
query with multiple rows to insert. This reduces the number of round trips to
the database server drastically, and results in much faster performance.

## Run times

I ran the two different functions on a small subset of data being 288478 rows,
which happens to be 3% of the files I was inserting.

_`execute`_ code:

```
    real    0m54.761s
    user    0m12.752s
    sys     0m4.876s
```

_`execute_values`_ code:

```
    real    0m7.545s
    user    0m2.092s
    sys     0m0.544s
```

Well that's a 700% increase on just a small number of rows. I didn't bother to
compare what the times would be like for the complete dataset, but running the
refactored version took about 25 minutes, so the original version would've
taken hours!

## Lesson learned

Would I have saved time if I had looked more in depth at the docs before
writing my code? Most likely. I think lessons like this is what separates
senior and junior level programmers, those who are able to grasp the scope of
a problem and it's solutions before they even begin writing code. Meanwhile,
the juniors have to take time to write bad code in order to learn.

## Update, another test

So my friend pointed me to another Python package called [dataset][4] saying
it's what he uses because he's a lazy Python user who is allergic to SQL.
[He][5] also said that Python > SQL, so I decided to prove him wrong, and also
because I didn't believe that another package that uses SQLAlchemy would be
faster than just using Psycopg2. (SQLAlchemy is built on Psycopg2)

```
    db = dataset.connect('postgresql://test@127.0.0.1:5432/testdb')
    def insert_demand_data(filename, date):
        with open(filename) as csvfile:
            reader = csv.reader(csvfile)
            header = next(reader)
            values_list = []
            for row in reader:
                n = row[0][2:]
                values = dict(node=row[1], date=date, n=n, r=row[2])
                values_list.append(values)
            table.insert_many(values_list)
```

And what do you know.

```
    real    0m53.392s
    user    0m17.512s
    sys     0m3.112s
    
    testdb=# select count(*) from test ;
     count
    --------
     288478
    (1 row)
    
```

[2]: http://initd.org/psycopg/

[3]: http://initd.org/psycopg/docs/extras.html#fast-exec

[4]: https://dataset.readthedocs.io/en/latest/index.html

[5]: https://www.jinpark.net/
