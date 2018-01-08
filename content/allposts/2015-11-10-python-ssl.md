---
title: Python and SSL 
subtitle: Bug hunting
date: '2015-11-10'
slug: python-ssl
---
# Frustrations with SSL

I have to admit, one of the most annoying things I can think of is when a new
technology comes out and you want to go and try it on your machine, and it
doesn't install/load. I feel like that is the biggest barrier for beginners to
pick up and learn something new, the installation process might end up in a
couple hours of frustration. Those couple hours are not going to keep the
dedicated (or stubborn) away, but if I were a new user and didn't know my way
around the OS, I might've given up.

So today Google research released their open source [TensorFlow][2]
programming system, which lets you create workflows for machine learning. It
essentially makes it much easier to start creating models instead of taking
time to write hundreds of lines of code first. So naturally I figured I would
give it a try and install it. The easy option was to use `pip install` so I
gave that a try and was given. `SSLError: [SSL: CERTIFICATE_VERIFY_FAILED]
certificate verify failed (_ssl.c:590)`

What a cryptic error, Googling lead me to many different answers and problems
and seem to point back at Python being the problem. The simple act of Google
usually involves going through many Github repo issue comments, stack overflow
threads, and then maybe some documentation. If the answer isn't actually
there, then you might have to start writing and asking for help.

## Docker

So I originally gave up and tried using Docker to set up a container to run
TensorFlow. Since the easier option wasn't working, I figured now would be a
good time to try using Docker. Unfortunately that didn't seem to work for me,
giving me some SSH error and IP error which I suspect has something to do with
my network at home. So with this problem I also decided that this might take
me too long and went back to looking at my ssl error with Python.

## The fix

So after deciding that I needed to look over my Python installation, I
remembered that I was running an Anacondas build of Python which lead me
instantly to the fix. All I had to run was `conda update openssl` and
everything was good to go for the TensorFlow installation. Now that I've done
all that, I'll be playing around with TensorFlow on my 6 year old laptop and
running some models to see how it does.

[2]: tensorflow.org