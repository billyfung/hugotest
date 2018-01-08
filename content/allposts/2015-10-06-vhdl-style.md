---
title: VHDL and style
subtitle: Grammar is important
date: '2015-10-06'
slug: vhdl-style
---

# VHSIC Hardware Description Language (not a programming language)

So I’ve been spending a lot more time getting intimate with VHDL and
attempting to discover good coding style versus code that will synthesize but
is probably not the best implementation. VHDL is definitely different from a
programming language in that sense because there are so many different ways to
do a single thing, but in the end there is probably only 1 or 2 proper ways to
get around to implementing it.

Like all other “programming” languages, (using quotations here because VHDL
isn’t a programming language) you need to get into the proper mindset of
knowing how your code will come out. Instead of your favourite compiler, like
gcc, you have a tool that will synthesize your code into logic gate
implementations. This creates a very direct connection to have more complex
code resulting in more complex logic implementations, which results in more
transistors being used.

So my recent coding has involved creating a simple RISC machine, which is
similar to a computer processor (at the most basic level). Implementing the
next state logic for the controller requires several inputs, and several
outputs, with the current state depending on both the input and the current
state. My first implementation involved using nested case statements such as:

```
    case current_state is 
       when state1 => next_state <= state2;
       when state2 =>
                case? inputA & inputB is
                        when input1 & inputC => next_state <= state3;
                        when input2 & “1—-’ => next_state <= state4
                        ...
```

So while this implementation in VHDL will compile and synthesize (and also
looks pretty readable to me) the usage of nested case statements makes it
quite lengthy. In addition, the case statement within a matching case? causes
extra logic within the matching. An alternative and easier way in VHDL is to
concatenate the input and current state signals:

```
    case? current_state & inputA & input B is
        when state1 & ”—–“ & ”——“ => next_state <= state2;
        when state2 & input1 & inputC => next_state <= state3;
        when state2 & input2 & "1—-” => next_state <= state3;
```

This already makes the VHDL less lines, which means there is less logic gates
being implemented. Concatenation sure is powerful! An exercise would be to
draw out the gates for each snippet of code and compare.
