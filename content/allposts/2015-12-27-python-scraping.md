---
title: Web Scraping with Python
subtitle: 
date: '2015-12-27'
slug: python-web-scraping
---

# The options

With a popular programming language like Python, there are many available
tools whenever you have a problem to solve. When it comes to scraping data, I
always find that it's a good skill to have, be it with Python or any other
programming language. We live in an age where the internet contains so much
information, but sometimes that information goes away, or isn't easily
accessible in one location. So when I began looking at the options for my
scraping project, I looked at [Scrapy][2] and then I also looked at
[BeautifulSoup][3] library.

# Scrapy

Scrapy is a very popular web crawling framework, which means it has a lot more
features to offer compared to a typical html scrape and parse. With Scrapy the
general idea is that you can set spiders loose onto a website, and then the
spiders will crawl around and grab whatever information you want it to. The
framework has also been polished to maximize efficiency, but to be honest I'm
not completely sure if it's always faster. The power of Scrapy is that you can
easily write a spider in minutes, and then set it loose with very general
requirements. This is very useful if say you wanted to grab all images from a
certain domain, and save it to a database, or grab all links from a domain.
The spiders will start off at the beginning page and then crawl their way
through the domain. There also exists lots of documentation for Scrapy, making
it easy to adopt and adjust certain examples for your use.

# Why not Scrapy

Lots of Python users will tell you to learn and use Scrapy for all your
scraping/crawling needs, and sure that might be a good plan, until Scrapy
doesn't work. Since it's a framework, the error messages you get might be very
cryptic, and if you look at the [list of issues on their github repo][4]
you'll see that while some get solved, some also don't. So this means that you
might spend a couple minutes (or more) in frustration trying to get your web
crawler going and working and then realize that when you started it with the
conditions you wanted, it gives you some weird error. So it's always good to
have another tool ready to go, which for me is using the [Requests library][5]
and [BeautifulSoup][3]. Requests makes it easy to do HTTP interaction, and
then BeautifulSoup makes it easy to deal with parsing the tree. For example,
to find all the tables with a certain id or classin the page:

```python
    tables = soup.find_all('table', id="tableid")
```

I usually prefer to use this way of scraping if the data I'm looking for is in
a specific format (like a table) within a page, and the html is messy. I'm not
an expert at scraping, but using BeautifulSoup and Requests in my arsenal of
tools has been important to me. But like most Python problems, use whatever
tool you are more comfortable with.

[2]: http://doc.scrapy.org/en/latest/index.html

[3]: http://www.crummy.com/software/BeautifulSoup/

[4]: https://github.com/scrapy/scrapy/issues

[5]: http://docs.python-requests.org/en/latest/
