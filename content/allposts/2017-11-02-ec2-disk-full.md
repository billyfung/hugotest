---
title: EC2 Disk Full 
subtitle: But not really
date: '2017-11-02'
slug: ec2-disk-full
---

I was deploying one of our applications this week but ran into an error where
ansible told me `No space left on device`

This was new to me because the application is a very small Flask application,
hosted on ec2 with at least a couple gigs of storage. There was going to be no
way that it was actually full.

So I went into the machine and checked

```
    df -h
    
    Filesystem      Size  Used Avail Use% Mounted on
    udev            487M     0  487M   0% /dev
    tmpfs           100M  8.0M   92M   9% /run
    /dev/xvda1      7.8G  5.7G  1.7G  77% /
    tmpfs           496M     0  496M   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           496M     0  496M   0% /sys/fs/cgroup
    tmpfs           100M     0  100M   0% /run/user/1000
```

This sure doesn't look like a full disk to me, so I had to think about what
else it could be.

```
    for i in /usr/*; do count=`sudo find $i | wc -l`; if [ $count -gt 10000 ]; then echo $i $count; fi; done
```

This shows that the culprit is the inodes. In my case, it was a bunch of
`linux-header` files that took up a ton of space. Make sure you don't delete
the actual linux-header you want, check with `uname -rv`.

Then you can run `apt-get autoremove`