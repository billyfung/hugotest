---
title: Development vs Analysis Environments
subtitle: 
date: '2017-07-05'
slug: dev-vs-analysis-env
---

## Docker, Vagrant, Chef, Puppet, Kubernetes… etc

So when dealing with writing code that you don't have to act upon production
data, you use an environment that allows you to break things continuously
until you find something that works. This is the ideal development cycle, one
where you can break states of your program, and be able to revert quickly and
easily. All the tools listed above help in some form of that. When you're
developing something that will go up into production on a server somewhere,
you the ideal development environment is one that mirrors the production one,
so when you want to deploy to production, the process will be seamless.

I currently use [Vagrant][2] and [Ansible][3] mostly for this. With a recent
addition of using [packer][4] to build virtualbox images. I have used
[docker][5] in the past, but back then the toolchain was still not very
refined and it was a tough learning curve. These days it's gotten much better,
but I haven't had time to test it out.

I mostly use Amazon EC2 cloud servers, but that can be a whole different
discussion.

## But what if you're doing analysis?

For those that deal with not just web based services, but also have to do
analysis, how does this fit into the picture? Often I download datasets and
explore them a bit with R or Python until deciding on if the dataset is useful
at all. Up until now, I've been doing this all locally, since my thinking
never defaults to using cloud based technology first. But recently I've had to
deal with datasets that are larger than usually, 20-40gb of data. I know this
isn't really that much data, but my laptop gets a bit stuffed when trying to
deal with all this locally.

So I've been thinking that I should be doing analysis like this on the cloud,
since the analysis might lead to eventually having the data in production. But
then there is an intermediate task of being able to share the results of
analysis easily.

I've played around with setting up an EC2 instance with RStudio on it, using
[Louis Aslett's][6] RStudio AMI. This makes it incrediably easy to set up
RStudio that you can access via the web, and also let's you run potentially
long scripts on the cloud while you do other work. (I say this as I'm running
an image processing script locally that is taking hours…)

In the near future I plan on exploring this further and figuring out the ideal
workflow for doing analysis as well as displaying results on the web.
Something I've also been thinking about is turning processing scripts into AWS
Lambda functions that will output into a S3 bucket for displaying.

Stay tuned!

[2]: https://www.vagrantup.com/

[3]: https://www.ansible.com/

[4]: https://www.packer.io/

[5]: https://www.docker.com/

[6]: http://www.louisaslett.com/RStudio_AMI/