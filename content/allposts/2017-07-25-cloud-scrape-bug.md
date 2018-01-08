---
title: Weird Cloud Scraping Issue
subtitle: 
date: '2017-07-25'
slug: cloud-scrape-bug
---
# Why doesn't it work properly in the cloud?

So the situation is this, I have a list of N webpages I need to check. My
script visits each webpage with some authentication, and checks to see what
kind of files are available to download on that page. Then I go and download
each of those files on the page, usually there are more than 5.

This looks something like (terrible pseudo code written for fun): `
pages_with_files = [list of downloads] for page in pages_with_files: r =
requests.get(page) soup = BeautifulSoup(r.text) the page to look for links
soup.findall(a href blah blah .file) d = requests.get(file to download) with
open(file_I_wanted) as j: copyfileobj(d) save d somewhere `

The task itself is something I've done a gazillion times, nothing too
difficult or tricky about it. The only difference is that the size is a bit
larger than what I'm used to. This script was run locally first, resulting in
a >20gb scraping job.

## To the cloud

Apart from taking 3 hours to scrape, it also takes up valuable resources on my
computer, which I can't really be giving away. So I figured I'd toss this into
the Amazon AWS cloud and just have it all done there. A very straight forward
Python script, so it should be no problem right?

Except the script runs into intermittent network problems. Now instead of just
pressing go and going to grab a coffee, I have to worry about the script going
and then stopping.

The error: `('Connection aborted.', BadStatusLine("''",))`

Without spending too much time on it, this means that the targeted server
isn't responding, or is giving a weird response.

## How to fix this?

I actually didn't have too much time to find the bug, largely because
sometimes it would download thousands of files without an error, but sometimes
it would only download a couple hundred. So my solution was just to keep an
eye on it, and run it again.

One of my ideas was to change timeouts, perhaps it had to do with the cloud
server taking a longer route between servers.

Next would be to just keep trying on the failed one.

Moral: I didn't have time to fix it, too many other things to work on. I got
the files in the end, and got other work done in the meantime.
