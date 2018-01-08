---
title: Python Robot web app 
subtitle: Time is fun
date: '2015-12-03'
slug: python-robot
---

# Controlling robot arm via web interface

[App hosted here][2]

Aside from doing Kaggle competitions, I do actually do Python things as well.
I haven't published all the exploratory stuff for SF Crime just yet, but I was
waiting till I had more of a complete picture that satisfied me before I did.
So instead here is something else that I worked on. This is a proof of concept
I did for controlling a robotic arm via the web. I am using Google's AppEngine
to host a Flask app, which is similar to Heroku and other hosting sites were
you can host things for free.

Why Google App Engine? Because I've never used it and I've used Heroku before.
It's a bit different than Heroku and maybe a bit harder to set up, but
essentially it works the same and Google offers lots of documentations.

# Overview

The high level overview is that we use Flask to grab HTTP requests and then
pass it into the appropriate Python function to make the robot move via USB. A
Raspberry Pi or similar device will be used to host the server and also will
communicate with the robot. The current view is a couple buttons that the user
will press to operate the robot.

The repo for the project is [here][3] and currently the frontend is quite
ugly. I have yet to make it pretty, but it will be a good exercise to go
through the Flask framework and see how it compares to Rails. Currently I have
a very very barebones app, and that is what I like about Flask, it is very
minimal.

## Flask app

`main.py` is where the Flask app is run out of, and it defines the routes for
the page. I decided to keep this as simple as possible for now, so everything
is routed in main.py and then I have a template folder to hold whatever html
templates I might need. The only different routing in `main.py` is the one
where an action is required after the user presses a button.

```python
@app.route('/<action>', methods = ['POST'])
#this redirects commands from the html page which has buttons for actions
def action(action):
 #as per index.html, whenever the root url is requested, action() is called
 #if subdirectory is called, we pass the name into the function robot/foo -> action('foo')
    if action == 'base-anti-clockwise':
        #then move arm by sending command via usb, and change status
        arm.MoveArm(t=1.0, cmd = 'base-anti-clockwise')
     if action == 'base-clockwise':
        arm.MoveArm(t=1.0, cmd = 'base-clockwise')
# and so on for the rest of the commands
    response = {
    'action' : 'Do the movement ' + action`
    }

    return render_template('index.html', **response)
```

So that part of code takes the button press, and the associated tag and
executes the proper robot arm movement. After the movement, it returns the
action and renders a new view using the template and says which action was
executed. All this I know is very hacked up and not pretty, but it's a
beginning.

# Home view

I didn't put much thought into the front end, since I'm not much of a frontend
person. So all I wanted to create was a couple buttons that expressed a
certain movement, and then display that the current action was executed. If
you don't want to see horrible ugly, basic html, probably skip down a bit :P

```
<!DOCTYPE html>
<head>
   <title>Robot Arm Interface</title>
</head>

<body>
<!-- 
{% if action %}
  <h1>Current: {{ action }}</h1>
{% endif %} -->
   <h1>Status</h1>
      <h2>{{ action }}</h2>

   <h1>Press to execute command</h1>


       <form action="/base-anti-clockwise" method="POST">
            <input type="submit" value="Rotate Base Counter-clockwise">
        </form>

        <form action="/base-clockwise" method="POST">
            <input type="submit" value="Rotate Base Clockwise">
        </form>

         <form action="/grip-open" method="POST">
            <input type="submit" value="Open the Grip">
        </form>

         <form action="/grip-close" method="POST">
            <input type="submit" value="Close the Grip">
        </form>

</body>

</html>
```

And that's essentially the meat of the web app + interface.

# Setup/Config with App Engine

  1. So to set this up locally to preview the app, the [Google App Engine SDK][4] is required. 

  2. Clone/create a repo that you'd like to start off with, or just create all the barebones files required. 

  3. Install whatever libraries might be required
[code]     pip install -r requirements.txt -t lib

    
[/code]

  4. Run locally with: ` dev_appserver.py . `

The application is then located at <http://localhost:8080>

# Deploy

Once you have everything working locally, you can deploy it to the Google
Cloud for others to view. Make a project/app at <https://appengine.google.com>
and then you'll have a project id.

To deploy your app:

```
appcfg.py -A <your-project-id> --oauth2 update .
```

Your application is now live at your-app-id.appspot.com

[2]: https://swift-yew-113223.appspot.com

[3]: https://github.com/billyfung/robot-flask

[4]: https://cloud.google.com/appengine/downloads