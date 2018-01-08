---
title: Flask URL Shortener
subtitle: 
date: '2016-02-26'
slug: flask-url-shorten
---

# URL shortener with Flask and Redis

source:<https://github.com/billyfung/flask_shortener>

So I was recently asked about building a URL shortener as an interview
question. And silly me, I automatically started blurting out stuff about data
structures and using a trie to optimize and what not to store the data. That's
the dangers of studying theory and not building things. So fast forward a week
or so, and I figured I should look into actually building one.

![homepage][2]

## General Outline

So the general flow of how a URL shortener works is that the user inputs a
long URL, then the web app takes it and returns a short link that corresponds
to a long link in the server. In order to reduce the amount of characters
used, one method is to map a set of characters(or numbers) onto a different
set. So what this means is that you want to express certain numbers/characters
in a different base, like switching from decimal to hexidecimal.
Mathematically this is called a [bijective function][3].

So moving forwards from the theory, the way I did this is that the user inputs
a URL into a form, then Flask routes the information, first storing the long
URL into Redis, then using the Redis integer key and converting that to a
shortened base string the shortened URL is returned.

### `index.html`

The first page will be the index page which displays a form with a field for
the URL to be submitted. With Flask, if you are using multiple html pages that
use the same general structure, it is advisable to have a `layout.html` file
that has all the common code. So for example, all the `meta` and `css`
information can be placed in `layout.html` and when you make another template
page, you can inherit from the template. Then for elements that will be
different from page to page, you utilize `block`:

```
    <div class="ui middle aligned center aligned grid">
      <div class="column">
        <h2 class="ui teal image header">
          <div class="content">
            URL Shortener
          </div>
        </h2>
        <div class="ui stackable container">  
          {% block body %}{% endblock %}
        </div>
      </div>  
    </div>
```

Then within future pages you can change what content is within those blocks.

For the index page, the `block` will be a form with a post action that will
allow Flask to grab the URL input.

```
    {% block body %}
      <form class="ui form" action="/shorten" method="post">
      <div class="field">
        <label>Input URL</label>
        <input type="text" name="input-url" placeholder="put your URL here">
      </div> 
      <button class="ui button" type="submit">Submit</button>
    </form>
    {% endblock %}
```

The important parts here is the action `action="/shorten"` which is the action
that is performed upon pressing the submit button. This will route to the
`/shorten` function defined in the Flask route. The other part is to make sure
`method="post"` is used, to POST the long url entered.

With that frontend part done, the backend part is to write what Flask will do
with the POST information.

```
    @app.route('/shorten', methods=['POST'])
    def return_shortened():
        url_to_parse = request.form['input-url']
        parts = urlparse.urlparse(url_to_parse)
        if not parts.scheme in ('http', 'https'):
            error = "Please enter valid url"
        else:
            # with a valid url, shorten it using encode to 62
            short_id = shorten(url_to_parse)
        return render_template('result.html', short_id=short_id)
```

The information submitted in the form is obtained with the `request.form`
call. First a check is done to make sure that a valid URL is entered, this is
done simply by checking for the presence of 'http' or 'https'. If a valid URL
is entered, then we pass the long URL into our custom function to shorten it.
Once it is shortened, we then render a `results.html` template that will
display the short URL.

### `shorten()`

This function takes a string input in the form of the long URL submitted, and
should return a shortened URL. First, a check will be needed in order to
determine if the URL already exists in Redis. I haven't talked much about
Redis, but for my purposes it's essentially a Python dict that is faster
(since it's in memory). So what the shorten function will do is check Redis,
then if the long URL isn't already in there, it will increment the increase
the key and associate that number with the long URL. Taking the Redis key,
that is what I use to encode the short URL, so a mapping from decimal (base
10) to a base 62 set.

```
    def shorten(url):
            short_id = redis.get('reverse-url:' + url)
            if short_id is not None:
                return short_id
            url_num = redis.incr('last-url-id')
            short_id = b62_encode(url_num)
            redis.set('url-target:' + short_id, url)
            redis.set('reverse-url:' + url, short_id)
            return short_id
    
    def b62_encode(number):
        base = string.digits + string.lowercase + string.uppercase
        assert number >= 0, 'positive integer required'
        if number == 0:
            return '0'
        base62 = []
        while number != 0:
            number, i = divmod(number, 62)
            base62.append(base[i])
        return ''.join(reversed(base62))
```

The base 62 encoding consists of all the upper and lower case characters of
the alphabet, and then all the numbers from 0 to 9. So what the shorten
function does is that it creates a Redis entry in which the long url is
stored, and the short url is stored, in the same entry.

### `results.html`

![results][4]

Once the short URL is obtained, the next step is to render the `results.html`
template to display the short URL. Again, all that needs to be done is to
extend the `layout.html` and then fill in the corresponding blocks with what
you want to display. I made use of [clipboard.js][5] to create a button that
copies the short URL to the user's clipboard.

```
    {% block body %}
    <form class="ui form">
      <div class="field">
        <label>Shortened URL</label>
        <input id='short-link' value="http://127.0.0.1:5000/{{ short_id }}">
        <button class="btn ui button" type='button' id="copy-button" data-clipboard-target="#short-link">Copy</button>
      </div> 
    </form>
    {% endblock %}
```

### Redirecting short URL calls

So the last part of the process is to redirect to the long URL when the short
URL is entered into the address bar. This is done by creating a route in the
Flask app that handles what happen when the short URL is entered.

```
    @app.route("/<short_id>")
    def expand_to_long_url(short_id):
        link_target = redis.get('url-target:' + short_id)
        if link_target is None:
            raise NotFound()
        redis.incr('click-count:' + short_id)
        return redirect(link_target)
```

The function `expand_to_long_url(short_id)` takes the short URL identifier,
and then checks Redis to see if there is a valid entry in there. If there
isn't then it will raise an error message, but otherwise it will simply
retrieve the Redis entry and redirect to the long URL.

## Extras

So once all that is there, you can add things like a count tracker to keep
track of how many times the shortened URL is visited, or create custom short
URLs. This is just a very basic way to show how a URL is shortened.

[2]: /figures/homepage.png

[3]: https://en.wikipedia.org/wiki/Bijection

[4]: /figures/shortened.png

[5]: https://clipboardjs.com
