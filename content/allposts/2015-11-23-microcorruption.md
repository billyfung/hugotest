---
title: Microcorruption 
subtitle: Reverse Engineering
date: '2015-11-23'
slug: microcorruption
---
# Breaking things is fun

One of the things I enjoy doing is taking things apart and figuring out how
they work at a very basic level. Reverse engineering is a very broad field,
and often requires a stubborn attitude towards trying to hack and bash at
something until either it gives up, or you give up. These days I've been
playing around with the CTF game called [Microcorruption][2]. The premise is
to figure out how to unlock an electronic lock by breaking apart it's
software.

The website makes it easier to understand low level code without having to
build a debugger and grab memory off devices, which is how it would probably
be done in the real world. So the point of the game is to figure out exploits
in the code that will make the lock open. The difficulty progressively gets
harder, so the first few locks are simple and relatively easy to figure out.
The skills learned include understanding how assembly and C code works
together within a physical device. This is a very low level programming of
where passwords are stored, and how different techniques might be used to
exploit it.

Doing this stuff is akin to what you see the _hackers_ doing on television
shows when they 're able to force password protected to do their bidding.
Things like stack overflow and stack smashing can be utilized. So far I've
been having tons of fun with the site, but it definitely gets frustrating at
times.

[2]: https://microcorruption.com/