# webserver ![CI](https://github.com/nickjj/webserver/workflows/CI/badge.svg?branch=master)

A zero dependency Python 3 web server to echo back an HTTP request's headers
and data.

- [Why?](#why)
- [Installation](#installation)
- [Usage](#usage)
- [FAQ](#faq)
  - [What if your web app is running in Docker?](#what-if-your-web-app-is-running-in-docker)
  - [What about exposing this server to the internet?](#what-about-exposing-this-server-to-the-internet)
- [About the Author](#about-the-author)

## Why?

I found myself developing a web app that sends out a webhook when a specific
event occurs. I wanted a quick and painless way to set up a local server to
make sure my web app was correctly sending out the webhook.

I mainly wanted to verify if it was sending the correct headers and data, so
here we are.

## Installation

All you have to do is download the `webserver` script, make sure it's executable
and place it somewhere on your system path.

#### 1 liner to get `webserver` downloaded to `/usr/local/bin`:

```sh
sudo curl \
  -L https://raw.githubusercontent.com/nickjj/webserver/v0.2.0/webserver \
  -o /usr/local/bin/webserver && sudo chmod +x /usr/local/bin/webserver
```

That will download the latest release. If you want to download the bleeding
edge version you can replace the version number with master to download it
from the master branch.

## Usage

You can start the server by running `webserver`.

That will listen on `http://localhost:8008` by default. You can customize the
bind address and port by running it with `webserver 0.0.0.0:5000` or any 
`host:port` you want.

You can hit `CTRL+C` to stop the server.

### Testing it out with curl

With your `webserver` running, now you can send requests to it however you see
fit. 

```sh
# Send a GET request with a query string.
curl localhost:8008/?just=testing

# Response (empty body):
127.0.0.1 - - [09/Sep/2020 14:23:24] "GET /?just=testing HTTP/1.1" 200 -
Host: localhost:8008
User-Agent: curl/7.68.0
Accept: */*
```

```sh
# Send a POST request with inline JSON data.
curl localhost:8008/ \
  -H "Content-Type: application/json" \
  -X POST --data '{"foo":"bar"}'

# Response:
127.0.0.1 - - [09/Sep/2020 14:09:33] "POST / HTTP/1.1" 200 -
Host: localhost:8008
User-Agent: curl/7.68.0
Accept: */*
Content-Type: application/json
Content-Length: 13


{"foo":"bar"}
```

```sh
# Send a POST request with file based JSON data.
echo '{"hello":"world"}' > test.json

curl localhost:8008/ \
  -H "Content-Type: application/json" \
  -X POST --data @test.json

# Response:
127.0.0.1 - - [09/Sep/2020 14:10:06] "POST / HTTP/1.1" 200 -
Host: localhost:8008
User-Agent: curl/7.68.0
Accept: */*
Content-Type: application/json
Content-Length: 17


{"hello":"world"}
```

```sh
# Send a POST request with form data.
curl localhost:8008/ \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -X POST --data "foo=bar"

# Response:
127.0.0.1 - - [09/Sep/2020 14:10:36] "POST / HTTP/1.1" 200 -
Host: localhost:8008
User-Agent: curl/7.68.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Content-Length: 7


foo=bar
```

If you had a Python application you could use the [requests
library](https://github.com/psf/requests) to send requests instead of curl.  Or
feel free to use whatever HTTP client your programming language has available.

## FAQ

###  What if your web app is running in Docker?

Without Docker, your web app might be running on `localhost:8000` and then
this `webserver` might be running on `localhost:8008`, and then you can
instruct your web app to send requests to `localhost:8008`. That's easy.

With Docker, things are slightly different. Your app web might still be running
on `localhost:8000` but you'll want to bind the `webserver` to `0.0.0.0:8008`
instead of `localhost:8008`. This will allow anyone on your network to connect.

Then from within your Dockerized web app you'll want to send requests to
`host.docker.internal` if you're using Docker Desktop. This will let your
container connect to a service running on your local dev box outsider of
Docker. On native Linux you can use your dev box's local network IP address
instead.

Alternatively you can run `webserver` as a service within your Docker Compose
set up. Then you could connect to it as if you would connect to any container,
such as using its service name from what's defined in `docker-compose.yml`.

Since this application has no dependencies and Python is installed by default
pretty much everywhere I would recommend running `webserver` outside of Docker,
but feel free to do what you think is best for your work flow.

### What about exposing this server to the internet?

Sure. There's lots of tools you can use. One of them that's free is
[ngrok](https://ngrok.com/). No port forwarding needed!

After installing ngrok you can start your `webserver` and then run `ngrok http
8008`. A few seconds later you'll have a publicly accessible HTTPS version of
this local server that you can access over the internet.

Keep in mind that's only practical for end to end tests, not a real production
environment.

The above could be handy if you had a test server on the cloud that you've
deployed your web app to and you wanted it to send a webhook response to your
local `webserver`. You could configure your deployed web app to send the
webhook to the ngrok address instead of `localhost:8008`.

Keep in mind if you're only dealing with local development you don't need to
use ngrok at all.

## About the Author

I'm a self taught developer and have been freelancing for the last ~20 years.
You can read about everything I've learned along the way on my site at
[https://nickjanetakis.com](https://nickjanetakis.com/). There's hundreds of
[blog posts](https://nickjanetakis.com/blog/) and a couple of [video
courses](https://nickjanetakis.com/courses/) on web development and deployment
topics. I also have a [podcast](https://runninginproduction.com) where I talk
to folks about running web apps in production.
