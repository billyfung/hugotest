---
title: Revisiting Docker and Docker Hub
subtitle: 
date: '2017-08-30'
slug: docker-revisit
---

## It began in 2016

Recently I had a coworker ask me "Why did you guys choose to use Docker?" (and
various other tools). To which I didn't really have a reply except that "it
was hyped up and popular at the time". Which really was the case for my first
usage of Docker. Back in 2016, Docker hype was building and lots of people
were exploring into using it.

So we hopped abroad the train for one of our newer and smaller scoped
projects. The regular stack used Vagrant and Virtualbox and virtual machines,
the regular battle tested stack. But Docker offered flexibility and ease of
use, the phrase "disposable VM" comes to mind. But in 2016 the toolchain for
Docker and OSX wasn't full matured yet. There were still many bugs and the
experience itself of setting up and being able to do what we wanted was
difficult.

### Documentation

Documentation is probably one of the most feared tasks for any large software
project. Especially one that becomes popular and people start criticising it.
I should not be one to talk about documentation, but using Docker in 2016
largely lead to issues because documentation wasn't adequate, and the
community wasn't mature enough yet. More often than not, stackoverflow became
what we used to get answer to problems, which isn't ideal.

## Docker in 2017

Now in 2017, the Docker toolchain has matured a lot, no more fiddling around
with `docker-machine` and playing around with networking. Although all the
time spent figuring out networking was valuable knowledge, the networking
between containers has been much improved. The OSX docker app works
beautifully, and there are no more issues with memory leaks or crashing.

I wish I documented more my issues with Docker from a year ago, which is why
I'm going to try to write more about my experiences with Docker in 2017.

### Disposable VM

Currently I'm using Docker to set up images where I'm slowly building up
various tools that work with [GAMS][2]. Docker is useful for this because I
can write a Dockerfile that is based off a Linux OS, Ubuntu in my case, and
then have it download GAMS, and setup GAMS in Linux.

Then after having GAMS set up, I saved the image and pushed it to [Docker
hub][3] to share with the community. This allows anybody to pull the GAMS
Linux image and use it for whatever they want. This also makes it a lot easier
for me to slowly add more software tools to the image, and then build it up
and share again. As it is, the image is 2GB, which seems fairly large, but
compared to setting up a Windows OS and installing it, it's not much at all.

## What's next

Next will be to try out Docker swarm and set up a multiple containers that
talk to each other, and see how easy it is to manage. Although these days I
hear [Kubernetes][4] is all the hype.

[2]: https://www.gams.com/

[3]: https://hub.docker.com/r/bfung/gams/tags/

[4]: https://kubernetes.io/
