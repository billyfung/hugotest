---
title: Learnings of the Week 
subtitle: 1
date: '2016-04-18'
slug: learning-1
---

# First one

So I'm going to start writing about things I've learned for the week. I
originally toyed with the idea of doing it daily, but then I think that might
become too much work and I might end up with things that I haven't thought
through. Although it would also be more of "I did this today", "I fixed this
thing that I tried yesterday".

I think it would be better for me to dwell on what I learned, and then
hopefully every Sunday I'll write a short succinct post about it. So here goes
my first one.

## `patch()`

One of the larger problems I was tackling this week had to do with test
coverage. The way that the database driven application involved integration
tests, which for all tests created a scaffolding database that does not get
committed to. To do this pytest along with psycopg, within the `conftest.py`
file a Postgres testdb is created, with a transaction savepoint `SAVEPOINT
empty` then migrations/schema/objects are applied to the database. This
savepoint allows for the transaction rollback to an empty database state after
a test is complete. So for every test, the test would execute whatever queries
are needed to build and test the db, then afterwards we get an expected state
for the next test. All this is done using a single database connection, that
is created when the database test scaffolding is created.

Issues arise when trying to test a function that makes use of creating another
connection to the database. Because of how the tests don't commit to a test
database, this lead to having two different database connections that queried
the database in different states. This is an obvious problem because the tests
would always fail. So a quick fix to this, is the patch() decorator, which
allowed for a database connection to be wrapped around the function to be
tested. Of course this results in a very ugly looking thing:

```
    with patch(database_connection):
        test_function()
```

This usage looks very hack-y and probably shouldn't ever been done aside from
in testing. But within tests, all rules are off and you just have to make sure
your tests are good.

## Two phase commits in PostgreSQL

Another feature I learned about Postgres is the ability to have a single
transaction commit to two tables, or two databases. Two phase commits are
essentially a way to run conditional commits, so if either commit fails, then
rollbacks are executed to both. This is a useful tool if you need to commit to
say, a customer table and an account table only if the customer commit
succeeds. Having both the commits within a two phase transaction allows for
this behavior to be completed in a simple manner. [SQLAlchemy][2] supports
this, and although documentation is a bit lacking, it does work. One thing of
note, following the SQLAlchemy documentation,
`Session.configure(binds={User:engine1, Account:engine2})` the binds are used
to correct map the corresponding SQLAlchemy class to the proper database. So
if you have within `engine1` 4 different tables that are required for the
final commit, you will need to write all four of those classes, ie.
`table1:engine1, table2:engine1, table3:engine1`

And lastly, the configuration for the Postgres database should have two phase
commits enabled.

## Finding the next business day in Python

Python libraries exist for everything, and of course there are libraries to
find the next business day given an input date. But sometimes when working
with libraries, you might not want to use one that isn't supported or updated
often. So to find the next business day I chose to use the standard library
`dateutil` and `[holidays](https://pypi.python.org/pypi/holidays)`. With these
two libraries, one is used to obtain the datetime.date() of holidays, and the
other is used to create a set of reoccurring rules that can be applied to find
the next day. Using `dateutil`, you are able to create a rule that limits the
days to only weekdays, and then further, you create an exception for certain
dates.

```python
import datetime
import holidays
from dateutil import rrule

def next_business_day(invoice_date):
    # create rule for only weekday occurences, starting from invoice date
    r = rrule.rrule(
        rrule.DAILY,
        byweekday=[rrule.MO, rrule.TU, rrule.WE, rrule.TH, rrule.FR],
        dtstart=invoice_date + datetime.timedelta(days=1)
    )
    nz_holidays = holidays.NZ(state='AUK', years=[invoice_date.year])
    next_day = rrule.rruleset()
    next_day.rrule(r)
    # exclude all the holidays
    for holiday in nz_holidays:
        next_day.exdate(datetime.datetime(holiday.year, holiday.month, holiday.day))
    return next_day[0].date()
```

[2]: http://docs.sqlalchemy.org/en/rel_1_0/orm/session_transaction.html

