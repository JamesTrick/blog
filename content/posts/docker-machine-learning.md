+++
date = 2020-02-24T03:19:00Z
heading = "Deploying Machine Learning Models with Docker"
main_content = ""
sub_heading = "Learn to create a production-ready API using nginx, gunicorn and Docker to serve your Machine Learning models."
description = "Learn to create a production-ready API using nginx, gunicorn and Docker to serve your Machine Learning models."
tags = ["docker", "machine learning", "flask", "python"]
title = "Deploying Machine Learning Models with Docker"

+++
There are a lot of articles out there explaining how to wrap Flask around your machine learning models to serve them as a RESTful API. This article assumes that you already have wrapped your model in a Flask REST API, and focuses more on getting it production ready using Docker.

### Motivation

Why do we need to further work on our Flask API to make it deployable?

* [Flask’s built-in server is not suitable for production](http://flask.pocoo.org/docs/1.0/deploying/#deployment)
* Docker allows for smoother deployments, more reliability, and better developer-production parity than attempting to run Flask on a standard Virtual Machine.

In my mind, those are the two biggest motivations on why further work is needed.

***

In this approach, we’re going to use [nginx](https://www.nginx.com/), [gunicorn](https://gunicorn.org/) and Docker Compose to create a scalable, repeatable template to deploy your machine learning models time and time again.

Let’s take a look at our proposed folder structure:

    .
    ├── README.md
    ├── nginx/|
    	├── Dockerfile|   
        └── nginx.conf
    ├── api/|   
    	├── Dockerfile|   
        ├── app.py|   
        ├── __init__.py
        |   └── models/
    ├── docker-compose.yml
    └── run_docker.sh

From this, our original Flask application lives in the `api/` folder, and there is a seperate folder `nginx/` which houses our nginx Docker container and configurations.

nginx, gunicorn work together as follows:

1. Client navigates to your URL, example.com
2. nginx handles this http request, and passes the request to gunicorn
3. gunicorn receives this request from nginx and serves the relevant content (gunicorn runs your Flask application, and handles requests to it).

As you can see, this is a bit more complicated, but a lot more reliable and scalable than Flask’s standard server.

Let’s get started.

### Flask + Gunicorn + Docker

The most logical starting point would be with our existing Flask application. Our goal for this section is to support gunicorn, and and create a Docker container for our application.

The only Flask-specific change we need to make is to ensure when we start Flask, that we specify a host of 0.0.0.0 and that we have `debug=False` .

```python
if __name__ == '__main__':
    app.run(host='0.0.0.0') # remove debug = True, or set to False.
```

`debug=False` is important, as if the user encounters an error, we don’t want a Traceback to be shown. The host, simply helps us later down the track when we configure nginx.

Before we create the Dockerfile, we first need to install gunicorn by simply running`pip install gunicorn` in your terminal. gunicorn itself will be configured later when we create the Docker Compose file.

Now’s also a good time to ensure the requirements.txt file is up to date — do this by running`pip freeze > requirements.txt` in your terminal.

BOOM! Now we move onto creating our Docker container. To create a Docker container, we need to create a file called `Dockerfile`. 

If you're new to Docker, the `Dockerfile` can be seen as a ‘recipe’ of everything we need to run our application. It's where we build our environment and copy our project files for a consistent experience everytime. To learn more about Docker itself, check our the [Getting Started pages on Docker’s website](https://docs.docker.com/get-started/).

The Dockerfile below is relatively straight-forward. We leverage of an existing base-image with Python 3.6 already installed, then we make and copy our application folders into the container.

```dockerfile
FROM python:3.6

# make directories suited to your application 
RUN mkdir -p /home/project/app
RUN mkdir -p /home/project/app/models
WORKDIR /home/project/app

# copy and install packages for flask
COPY requirements.txt /home/project/app
RUN pip install --no-cache-dir -r requirements.txt

# copy contents from your local to your docker container
COPY . /home/project/app
COPY ./models /home/project/app/models
```

You may notice, that we haven’t specified `flask run` or any equivalent command in our Dockerfile. This is because we want to use gunicorn to start our Flask application. 

We also want to it to be started alongside our nginx container when. So we’ll be doing this when we configure Docker Compose later on.

{{< subscribe heading="Like this post? Subscribe below.">}}

### nginx

nginx in our case replaces the default Flask webserver and is significantly more scalable and production-ready than Flask’s built in server.

We can set up nginx by creating a new directory within our project root, and creating a Dockerfile with the following:

```dockerfile
FROM nginx:1.15.2

RUN rm /etc/nginx/nginx.conf
COPY nginx.conf /etc/nginx/
```

This pulls down the [nginx Docker image](https://hub.docker.com/_/nginx/) and simply copies `nginx.conf` into the Docker container.

`nginx.conf` is the file where we can configure our nginx server, it looks something like below:

    worker_processes  1;
    
    http {
      
      keepalive_timeout  65;
      
      server {
          listen 80;
    
          location / {
              proxy_pass http://0.0.0.0:8000;
    
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          }
      }
    }

The main parts to this, is that we set a keepalive_time out in seconds, and tell our proxy to listen on port 80 (http) and return localhost with a port of 8000.

### Bringing it together

We now have the makings for a pretty good Flask API/Website. There’s really only one last thing to do.

Since we have multiple Docker containers, we want a way to run both of them and specify how they interact with each other. This is where [Docker Compose](https://docs.docker.com/compose/) comes in.

> Compose is a tool for defining and running multi-container Docker applications.

From Docker’s website we can see that using Docker Compose is a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker-compose up` and Compose starts and runs your entire app.

We’ve already completed step one, so now we can safely move on to step two.

Our `docker-compose.yml` file looks like this:

```docker
version: '3'

services:
  api:
    container_name: api # Name can be anything
    restart: always
    build: ./api
    ports:
      - "8000:8000"
    command: gunicorn -w 1 -b :8000 app:app

  nginx:
    container_name: nginx
    restart: always
    build: ./nginx
    ports:
      - "8001:8001"
    depends_on:
      - api
```

There are a few things to note about this file.

**services:** This is where we specify our individual Docker containers, firstly, our API and then our nginx.

_build:_ refers to the location the Dockerfile is in relation to the docker-compose.yml file.

_command:_ allows you to specify and bash commands needed to run that service. In the case of our API, we didn’t run the Flask application in the Dockerfile, so we do it here using gunicorn.

It’s important to note, that we bind the ports to 8000 so it matches the location specified in `nginx.conf.`

I highly encourage you to read more about Docker and Docker Compose [here](https://docs.docker.com/compose/overview/).

***

There we have it. We’ve now configured nginx, Docker’d our Flask Application and used Docker Compose to bring it together.

You should now simply be able to type `docker-compose up` and your server should start.