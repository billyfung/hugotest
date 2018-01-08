---
title: Learnings of the Week 
subtitle: 4
date: '2016-05-19'
slug: learning-4
---

I will admit that I missed my Sunday blog posting routine, it was a good 4
week run. My excuse is that I was too tired to do anything on Sunday after
going for a 100 mile bike ride that completely exhausted me on Saturday. Most
of my days follow a pretty standard schedule of working, biking, resting. With
some beer drinking thrown into that. And on top of that, I try my best to
maintain contact with friends, and sometimes even try to go and meet new
people. Working at home really doesn't help the whole networking thing, so I
try to stay ontop of my meetup game and go out to attend things. But at the
same time, I have bike racing that I enjoy doing, which takes up 2 days of the
week at least.

# Learning of the week

So to hopefully keep things on track, I will still continue writing about
things I've learned. No more promises on keeping it a weekly Sunday thing, but
I've considered shorter posts spread out during the week, since it's easier
for me to write about things while they are still fresh in my head.

One of the main problems we ran into last week involved a weird behavior where
the deployed feature didn't seem to be doing the expected behavior of writing
to the database. The weird behavior comes from the fact that everything worked
locally, but then upon deployment to the staging server, it didn't seem to
work.

## Step one - Logging

The first steps taken to debug this problem was to look at code, because as
programmers, what else could break? The new feature involved code for writing
to the database if the user completed the email verification, one of those
things where you get a link, and then click it and your account is verified.
Looking at the code (node.js) at first, I would've thought maybe it had
something to do with the query to the database not being completed. So throw
print statements all over the place so you can see which parts of the code get
executed. Strangely enough, we weren't getting any of the database queries
getting executed, which makes sense since the database isn't getting updated.

## Step two - Check system variables

In hindsight, it's really easy to explain where to look after you've found it.
Once we realized that the query wasn't getting executed, we figured it might
have something to do with how the system is set up. Similar queries seem to
run fine in other parts of the code, so it can't be the code itself. Perhaps
it could be due to specific edge cases, but those are usually the last things
to check. Instead, the problem turned out to be the hardcoded URL. Doing our
testing, we were doing it on the production URL, instead of the staging URL
which meant that the production server was getting queries which didn't exist.
A simple but elusive error.

## Step three - Variables instead of hardcoding

And that is why programmers always should use variables whenever possible, and
avoid hardcoding. Hardcoding limits you to very specific situations, and often
things change, and your hardcoded situation doesn't work anymore. Except that
it might half work, leaving you with a weird bug.

So always make use of environmental variables when you can.
