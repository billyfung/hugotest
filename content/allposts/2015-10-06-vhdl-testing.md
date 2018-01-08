---
title: VHDL testbenches
subtitle: Always be testing
date: '2015-10-06'
slug: vhdl-testing
---

# Testbenches

One of the key elements in writing synthesizable VHDL is to test your VHDL
before loading it onto the FPGA. For very simple and basic designs, it might
not be worthwhile to test in a separate program, but like in software
engineering, nothing is simple and basic in the real world. So tests are very
useful in debugging your design before figuring out things are working as they
should be. The typical workflow I do is I write the code in Quartus, or
whatever editor you have that allows for tab complete since it is definitely
more efficient to tab complete than to type out some of these very verbose
descriptors (std_logic_vector is quite a number of keystrokes). Then ModelSim
is used to run your test bench.

Before even starting, I’d like to point out that one of the more annoying
features is that the subset of synthesizable code is not the same in ModelSim
and Quartus, depending on which version of VHDL you are using. In my case, I’m
using VHDL 2008, which supports make functions that make designs nicer, like
the matching case statement. Just keep that in mind when your VHDL compiles
and your tests all pass in ModelSim, and then fails when you go to synthesize
in Quartus.

Moving along, the method of testing your VHDL design is quite straightforward.
You create a new test bench file, and within it you instantiate the component
you wish to test. The test bench entity will be empty, and the architecture
will consist of the signals that will propagate the design, mainly clock and
reset.

```
entity cpu_tb is
end entity;

architecture test of cpu_tb is
signal reset : std_logic; 
signal clk : std_logic;

beginDUT : controller port map(  reset => reset, clk  => clk);
```

The DUT here is a label meaning device under test, which is the component you
are testing. This simple example only uses 2 signals within the controller
entity, but if your entity has more signals, then when you port map the
signals, you will put the signals there as well. In the case of a complex
design with multiple entities, the syntax to test signals within an entity is:

```
<<signal DUT.entity.signal: std_logic_vector>> <= force ‘1’;
```

This command will force the signal to be whatever value you’d like it to be
for testing purposes. This is a very handy tool for controlling the test bench
if your signals require several cycles to change. I’ll call it a driving
signal since you can use a signal to drive the test forward in certain
situations. This kind of testing is mainly useful when you are testing
components being added to a larger more complex design. By forcing a signal,
you are able to see what changes your added component has.

The following shows how to simulate a clock in ModelSim:

```
process begin
clk <= '0’; wait for 5 ps;
clk <= '1’; wait for 5 ps;
end process;
```

So knowing the cycles, you can change the signals accordingly in each cycle.

To automate the detection of failures, the following is used:

```
report to_string(x) & “ ” & to_string(o) & “ -> ” & to_string(xnext);
assert xnext = “000010000” report “output should be 000010000” severity failure;
```

The report to_string(x) outputs the string values in the ModelSim console for
easier reading. The assert checks the value of the signal, and if it is not
equal, issues a failure result. This way you can run your test and the console
will show you where all the failures are.

The ideal test bench is one in which your input data/files is loaded into the
VHDL, and then run according with all the failures displayed. But sometimes
this is not possible, and the next best thing is to view the signal waveforms
in the waveform viewer in ModelSim. If you are testing components individually
by driving them the signal to change, it is advisable to not do this in the
final test, because if the signal you are driving is actually dependent on
another component, it does not show that the component is working properly.
