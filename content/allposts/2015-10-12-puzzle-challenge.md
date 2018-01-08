---
title: Puzzle Challenge
subtitle: 
date: '2015-10-12'
slug: puzzle-challenge
---

# What is this?

So I'm lucky to have a bunch of friends who send me awesome things to solve
whenever they encounter them. Today my friend send me this interesting
challenge she was given and of course I spent some time trying to solve it.

![challenge][2]

Looking at it, it's a bunch of hex that appears to repeat a certain number of
times. So the first step when given a hexdump is to try the easiest things
possible. So I converted a few lines into ASCII and then attempted to figure
it out. Came out a bunch of jibberish so I temporarily looked online to see if
the repeating characters were actually magic numbers for certain programs. Oh
also the 0xac at the end threw me off, converted into decimal it gives you
some random number which made no sense to me, so I thought maybe it might
specify what kind of program this would be.

But looking at the hex some more, it's obvious that there is some sort of
repeating structure to it, and I ended up converting all the hex into ASCII.
Direct conversion gave me:

[code]

    sllfcemnlsjef9esr@deablloosnceruti.yoc
    mt-le lema obtuy uo rafovireth ca-kl
    fselncsmel9jsf@eerbdlaolnoesucirytc.mo-
    etllm  ebauo toyruf varoti eahkc
    -sllfcemnlsjef9esr@deablloosnceruti.yoc
    mt-le lemncsmel9jsf rafovireth ca-kl
    fselncsmel9jsf@eerbdlaolnoesucirytc.mo-
    etllm  ebauo toyruf varoti eahkc
    -sllfcemnlsjef9esr@deablloosnceruti.yoc
    mt-le lema obtuy uo rafovireth aa-kl
    fselncrmelncsf@eer1h0mzm
    
[/code]

Looking at that, you can see that the byte order mattered in the conversion
into ASCII. I had totally forgotten about little endian vs big endian, but
this is now obvious. So instead of converting it all again, I just wrote a
quick python command to reverse it for me. I take the string and then slice it
into 2s, swap the pairs using [::-1], and then join it all back together.

```python
    ''.join([ temp2[x:x+2][::-1] for x in range(0, len(temp2), 2) ])
```

Doing all this gave me the email for the company for further questioning. I
love these kind of challenges from companies looking to hire!

[2]: /figures/challenge.jpg