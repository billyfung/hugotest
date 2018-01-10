This is for learnings and musings about setting up a reverse proxy using nginx.

I will start off with a simple server where you have an IP address and that's all. You want people to see your web application when they go to the link, for example `123.123.123.123`

One of the main usages of [nginx](https://www.nginx.com/) is to use it as a reverse proxy that redirects and shares between different servers, a load balancer. An example of this would be using nginx to redirect all certain types of traffic to your web application. This is useful if you want certain requests to go to certain parts of a larger project, like sending api calls to a specific docker container, and non-api calls somewhere else.

## How I use nginx
When I first started off as a web developer, or a developer in general, I of course didn't know anything about nginx. Setting up all the infrastructure was done by the senior developer, the only person who knew how to do it. Of course I was able to look at the code, and see the whole process, but this isn't how I learn most things. But also with infrastructure stuff, it's very easy to get into the thought process of just getting it working and then ignoring this. This goes for software development in general, which is why best practices exist. Of course for me to learn nginx, I dug into the configuration and played around with setting up my own server.

One of the ways to use nginx is within a Docker container that allows you to containerize the application, and abstracts out your project into it's specific services within a container. Like many popular web services, there exists an official Docker repository and image for it, which makes it even easier to use nginx in your project.

### `nginx.conf`
All the configuration settings live in this file, I won't say I'm an expert and knowing all the parameters, but I know the ones that I use the most often. This is where the main redirecting work of nginx will live, instructions for the nginx server as to where to direct traffic. Nginx is pretty much the traffic operator standing at a roadway intersection. 

So you have the address `123.123.123.123` and your web application; hopefully this will also live inside a Docker container with the port of the web server being known. And what needs to be done is that nginx will redirect traffic towards your web application.

#### Workers and connections
At the top of `nginx.conf` will be worker processes and worker connections, this is how many workers/threads will be available to nginx on that server. If you want to limit the number of people who can view your web application at a certain time, then you can change these parameters.

```
worker_processes 1;

events {
    worker_connections 1024;
}
```

This means that with 1 worker process, nginx will be able to serve 1024 connections/second, but of course this is theoretical and is limited by various factors.


#### http block
The main chunk of configuration lies within this http block. This is where the instructions for nginx to handle http web traffic lives, which is the basics of web. What a basic version looks like:

```
http {
    gzip              on;
    gzip_http_version 1.0;
    gzip_proxied      any;
    gzip_min_length   500;
    gzip_disable      "MSIE [1-6]\.";
    gzip_types        text/plain text/xml text/css
                      text/comma-separated-values
                      text/javascript
                      application/json
                      application/javascript
                      application/x-javascript
                      application/atom+xml;

    # List of application servers
    # ip of docker web container 
    upstream webapp {
        
        server web;
    }


    # Server config here
    server {
        listen 80 default_server;
        
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        location / {
            auth_basic              "Restricted Content";
            proxy_pass              http://webapp;
            auth_basic_user_file    /etc/nginx/htpasswd;
            proxy_redirect          off;
            proxy_set_header        Host $host;
            proxy_set_header        X-Real-IP $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header        X-Forwarded-Host $server_name;
        }
    }

}
```

The first parameter is the (gzip module)[http://nginx.org/en/docs/http/ngx_http_gzip_module.html] which is used to compress data before sending, this will speed things up, but at the cost of cpu power. I won't go into the various parameters involved because you can read the documentation for that.

The second parameter is the (upstream module)[http://nginx.org/en/docs/http/ngx_http_upstream_module.html], used to define the servers that will be used later on during the proxying. This is essentially setting up a location variable, which in my case is set to `server web`. This is set up within my Docker networking configuration, where I link the `nginx` container to the `web` container.

And lastly, the (server module)[http://nginx.org/en/docs/http/ngx_http_core_module.html#server] which is the place where proxying happens. Apart from setting up some useful error and access logging locations, the `location` block is where nginx goes after processing the initial request. My file has set up an authentication method using a `htpasswd` file, this can be removed if no authentication is needed. 

The `proxy_pass` parameter is where the reverse proxy happens. I have this set to point at the server defined in `upstream` which is where the web applications lives. This can just as easily be done by setting it to be `proxy pass http://localhost:8080` and removing the `upstream` module (this is what I did originally). But hardcoding is against best practices for the most part. The other proxy settings are for setting up the header fields that nginx will send to the upstream app server.

## Setting up with Docker
I won't go into too much detail here, but the main thing is to have the containers named properly so nginx can see them. In the case of my `nginx.conf` shown above, I have my web container named/aliased web so then we don't need to hardcode in the ip address of the Docker container running the web application.