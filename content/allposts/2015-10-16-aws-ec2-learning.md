---
title: Learning AWS EC2
subtitle: To the cloud!
date: '2015-10-16'
slug: aws-ec2-learning
---

# To the Cloud!

Decided to play around with the free-tier AWS EC2 offering for a year. It gets
pretty confusing trying to figure out which AWS service is best for you usage
when you go onto their page. So many different services for specific things.
Well today I wanted to put an IRC bot onto a cloud server and not have to
worry about keeping a local copy running always. The bot is written in Ruby,
so I opted to use an EC2 Linux server with Ruby already pre-installed. Setting
up the EC2 instance was quick and easy, press the button and save the .pem
file.

This is my first time using a Linux instance, previously I've set up a Windows
Server and toyed around with sysadmin stuff like setting up accounts and
linking with Google. Since I didn't need to have a GUI, using Linux seemed
like the proper choice. SSHing into the server is quick and easy to do, and
then everything is already installed apart from the specific packages. Using
the yum manager, it was easy to configure everything. Installing gems was done
using gem install command except for a couple issues.

When attempting to installing Nokogiri straight off the bat, it didn't work
and gave me some errors to do with the extensions.

```
/usr/bin/ruby extconf.rb
mkmf.rb can't find header files for ruby at /usr/lib/ruby/ruby.h
```

This was from the `gem install nokogiri` command and clearly something wasn't
working. To solve this, I used the yum manager to search for a nokogiri gem
and then install it using yum to solve the problem.

```
yum install rubygems20-nokogiri
```

After all that, my script didn't want to run, giving me a error to do with
'require'.

```
usr/share/ruby/vendor_ruby/2.0/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file
```

This was a weird error that I think had to do with the path? To be honest I'm
not relaly sure what it was that caused it. I didn't really solve this
problem, but instead I went into root and then ran the script from there and
it worked.