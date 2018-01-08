---
title: Learnings of the Week 
subtitle: 2
date: '2016-04-24'
slug: learning-2
---

# Week 2

A side bonus of these blog posts is that while they allow me to reflect back
on the week, they also let me get prepared for the next week. I'm planning on
writing these posts on Sundays, so after writing them I'll have reviewed the
status of the past week and where I should be moving to for the next week.

## Write test(s)

Pretty sure this is what everybody says to do, and while doing my own
projects, it's very easy to just ignore my own advice and not write tests. But
once projects get complex, not writing tests come back to haunt you. I can't
imagine writing a complex application where you're adding new features every
week, or where new edge cases arise and there is no test suite available to
deal with it. My experience is that tests aren't always the best use of
resources, depending on the situation of course. As a backend dev, there are
always more features that I can work on, and always a long list of tech debt.

So when approached with the question of when not to write tests, I would say
this probably depends on what the feature is. My thoughts are that tests
should always be considered, but what makes them needed is if you know that
the feature will be built upon in the future. Like creating an sign up form,
you will probably change that form in the future, and doing so it might be
annoying to see if adding new things has broken anything. On the flipside, if
the feature/function is doing a solo thing, that doesn't affect anything else,
it might not be worth it to spend time to create the test.

The way I write tests is to hopefully create an easy way to understand the
underlying business logic. Creating test fixtures is an easy way to view the
state of the database, and hopefully provide a simple outside of what is
needed for the function to run. Always try to keep test fixtures to a minimum,
only `INSERT` into the tables that are required. While writing the tests, it
will most likely be very tedious because it requires that you actually
understand what the code is suppose to do. How else are you going to test the
output if you don't already know what you are suppose to get?

In addition to writing tests, it's good practice for developers to write tests
for each other. Sometimes you'll find bugs or weird behavior in code just by
getting a different perspective, or coding style. I understand that most
people don't ever want to do this, but it definitely has it's merits.
