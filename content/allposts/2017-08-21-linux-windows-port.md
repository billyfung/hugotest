---
title: Porting Windows to Linux
subtitle: Time is fun
date: '2017-08-21'
slug: linux-windows-port
---

# MS-DOS to bash

One of the programs I've run into is written for Windows only, I could
determine this when I saw all the filepaths.

## Filepaths

Most of the difference lies in changing out a forward slash for a back slash.

```
    '%system.fp%..\Input\'
```

becomes ` '%system.fp%../Input/' `

## `erase` `move` `copy` `if exist`

These are all MSDOS commands that I had to convert to bash ones.

The corresponding ones are: `rm`, `mv`, `cp`

`if exist` doesn't need a corresponding command, because if if bash doesn't
find anything, then the command just isn't executed.

## `mkdir`

Stays the same!

## The hidden one

One difference I wasn't able to just see immediately was `cls`. Corresponding
command is `clear`