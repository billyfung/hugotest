---
title: Multiple Blogs with Middleman
subtitle: 
date: '2017-01-03'
slug: middleman-blogs
---

## Changing things up for 2017

You might have noticed that I added another section to my website, called
"Thoughts". This is done by adding another blog to Middleman, so now I have
two blogs within the same page. Using Middleman, this is relatively
straightforward, not much fiddling required at all.

### `config.rb`

The first step is to add the second blog to the `config.rb`, so that it should
look something like this:

```
    # Blog Configuration
    activate :blog do |blog|
      blog.name = "cats"
      blog.prefix = 'cats'
      blog.permalink = ':year/:month/:title.html'
      blog.sources = ':year/:month/:day-:title.html'
      blog.summary_separator = //
      blog.year_link = '{year}.html'
      blog.paginate = true
      blog.per_page = 10
      blog.page_link = '{num}'
      blog.layout = 'layouts/post'
    end
    
    activate :blog do |blog|
      blog.name = 'dogs'
      blog.prefix = 'dogs'
      blog.permalink = ':year/:month/:title.html'
      blog.sources = ':year/:month/:day-:title.html'
      blog.summary_separator = /READMORE/
      blog.year_link = '{year}.html'
      blog.paginate = true
      blog.per_page = 10
      blog.page_link = '{num}'
      blog.layout = 'layouts/post'
    end
```

This tell Middleman that there will be two blogs, so you can structure your
project accordingly. For me, this means having separate folders for each of
the blog `middleman/dogs` and `middleman/cats`.

### Organizing the partials

In order for the pages to be created according to each of the blogs, you will
either have to put the blog name in the frontmatter like so

```
    ---
    title: Cats gone wild
    blog: cats
    date: 2017-01-03 23:18 UTC
    tags:
    ---
```

The other option is to have the blog helper function reference the blog name,

```
    <% blog('cats').articles.each do |article| %>
      <article>
        <h2>
          <a href="<%= article.url %>"><%= article.title %></a>
          <time><%= article.date.strftime('%Y') %></time>
        </h2>
    
        <a href="<%= article.url %>">Read more</a>
      </article>
    <% end %>
```

### Writing new articles

And the last step is to tell Middleman which blog you want to create a new
article for. This is done by using the `--blog` flag like so:

```
    middleman article --blog=cats "Cats are better than dogs"
```

And that's it, nice and simple with Middleman
