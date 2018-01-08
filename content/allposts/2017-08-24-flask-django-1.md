---
title: Differences between Flask and Django 
subtitle: 1
date: '2017-08-24'
slug: flask-django-1
---
# Differences between Flask and Django - Part 1

This blog post is highly biased because I have worked a lot with Flask, and
not at all with Django. So coming from a Flask background where I don't always
follow a rigid MVC framework, I'll write about what I discover when comparing
the two.

The list of tasks comes from me working on a website to host podcasts that my
[friend][2] and I record.

# 1\. Rendering a static page

So I have a webpage where I want to display an "About Us". This will be static
information and won't be require any models (well I won't say no to a model…)
Simple, easy task to learn the ins and outs of the framework.

## Flask:

  * app.py
  * templates/about.html

You could have the html all within the route return as well if you wanted to
be gross about it.

```
    from flask import Flask, render_template
    app = Flask(__name__)
    
    @app.route('/about')
    def about():
        return render_template('template/about.html')
```

## Django

  * templates/about.html
  * urls.py
  * views.py

Since Django comes with "batteries included", when you start the project,
everything you need is already there. You just have to know which pieces to
poke around in.

The `urls.py` file is where the routing is done within Django, similar to the
route decorator in Flask. The routing is a bit more powerful in Django though,
with the `django.conf.urls` utility functions.

The normal usage is to define a list of `url()` instances that take on the
following parameters:

```
    url(regex, view, kwargs=None, name=None)
    urlpatterns = [
        url(r'^index/$', views.index, name='main-view'),
        url(r'^about/',  views.about),
        ...
    ]
```

The regex usage in Django comes built in, and is quite powerful although I
personally have never thought about such a utility until I saw it in Django.
With Flask, using `werkzeug.routing` you can build your own [regex converter
class][3] that inherits from `BaseConverter`

The `views.py` change is just so that the Django project is already well
organised and data is abstracted through it's proper layers. All it's doing is
taking the request for the `/about` page and then returning the render of
`about.html`

### Summary

I know this was a very simple task, but it helps to explain how Django
operates, and how different/flexible Flask can be for certain things. If I
wanted to make a web application that only served static pages, then it would
seem like Django has so much extra overhead that isn't needed. It's like
buying a formula 1 car to drive 1km to work.

[2]: https://jinpark.net

[3]: http://werkzeug.pocoo.org/docs/0.12/routing/#custom-converters
