---
title: Learnings of the Week 
subtitle: 3
date: '2016-05-02'
slug: learning-3
---

# End of April

Well it's already the start of May, and it's kind of crazy how quickly the
last month went on. Definitely hope to have more interesting things learn in
the weeks to come.

## patch() and passing tests

So in a previous blog post, I talked about how I used `patch()` in test
methods in order to bypass how the production code calls databases, and how
the test code creates test databases. All tests passing means that everything
is good right? Wrong. Passing tests are only passing for what you tell it to
be passing, and if you're not testing in the proper procedure, you might end
up with a passing test, and then a failing code run. That is precisely what
happened with me, after I had thought that the tests I wrote were done.

The problem was that the routine I was testing made calls to a production
database in several different which when testing, I didn't care for. So of
course when doing another functional test from the command line, it failed.
This had me scratching my head for a second because all the tests passed
right? The system went from a state where everything was working, then I wrote
tests, and tests were green, then the system stopped working. It is very
important to break down the situation and to set back and re-evaluate what
could have gone wrong. Luckily I had a good feeling that it had to do with how
I changed the calling and connecting of the production database, and the
problem was solved. But after all this, I still think there is lots of room
for refactoring of how database connections are managed within an application,
and how to properly design the different modules such that testing is more
effective.

## Weird Postgres network issues

So this isn't really something I have finished learning, but with one of my
deployed applications, I have been running into intermittent `SSL SYSCALL
ERROR` messages that pop up, and are pretty much un-reproduceable. The errors
occur, and then fix themselves almost immediately, so a simple refresh usually
solves the problem. The main issue I have with debugging so far is that I
cannot get this to happen in my local development, and the errors only occur
in production. So the main difference between the two is that the production
version talks with databases hosted on AWS RDS. This makes me think that the
errors have something to do with the network connection between the two, or
perhaps something to do with how the Postgres database is set up in RDS.

I've spent some time trying to diagnose whether or not this bug is because of
the code I've written, which is very likely since this is my first
implementation of SQLAlchemy. But I've noticed that I get this drop of
connection after some time in `psql` as well… My next steps are to look at the
pg _stat_ activity and understand more about what the connection messages
mean. It would very well be how the connections to the database are managed
within Postgres, and not cleaned up accordingly.

## Postgres RTFM

So the Postgres manual is over 2000 pages, I guess it's time for me to RTFM.