---
title: Gravity Gradient Stabilised Satellite
subtitle: Processing simulation
date: '2015-10-06'
slug: satellite-simulation
---

# Physics Simulation using Java

So once in awhile I like to go and clean up my files, and put everything into
the proper directories and just to refresh my memory on what I have lying
around. One of the tasks that I've been meaning to do is to try to put more of
my work online for people to view, in case it might be helpful. One of these
such projects is the work I did on simulating a gravity gradient stabilized
satellite. The general introduction to this is that for low earth orbit (LEO)
satellites, we would like to use as little power as possible to keep a
satellite in orbit. Using Earth's gravitation field as a passive power source,
it would then be possible to orient the satellite.

[Final Report here][2]

## Implementation

In order to test and simulate this gravity gradient stabilization, we decided
to do this all with the [Processing framework][3], which is essentially Java
but with visual arts as the intent. The main reasoning was that we wanted
something that was already supported with lots of graphics libraries to create
something visually appealing. Much easier to appeal to the general audience if
we give them something that they can interact with, and something with pretty
graphics. To keep things short, we modeled the satellite as an elongated rod,
so the main payload with attached stabilizing boom. Then we implemented all
the physics in Processing, along with the numerical solvers required. You can
view all this in this [Github repo][4]. Then we made the simulation showing a
satellite and it's orbit around Earth, with both rotating at their respective
speeds.

## Reflection on Processing/Java

Since this was all before the day when web graphics became much more popular,
and Java was still relevant in the realm of web, we had a little Java applet
that allowed people to run the simulation and play around with different
values and observe the stabilization. It is definitely interesting to see how
the web progressed now, and how much easier it is to not have to deal with
Java. After uncovering the project on the computer, I decided to try to run
the Java app using Java installed on Windows, not in browser. Turns out you
cannot run Java apps not unless they have security certification, which
essentially means you can't run it on your computer.

## Processing.js

Moving onto the web world, [Processing.js][5] was created to stop the reliance
on Java when ported to the web. Processing.js essentially turns everything
written in Processing Java into Javascript for the web, and aligned with HTML5
standards. Unfortunately, when we made our simulation, the various physics
libraries and camera libraries were all in Java, and not Javascript. So
porting the app to be Java free wasn't successful for me. Which brings me to
where I am now, trying to figure out the best way to put this simulation
online without subjecting the user to have to load Java. I think the best
option is to re-write the code using compatible Javascript libraries.

[2]: /satellitepaper.pdf

[3]: https://processing.org

[4]: https://github.com/billyfung/satellite_boom/tree/gh-pages

[5]: http://processingjs.org

