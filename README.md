# DockerProject
On this page you build a simple Python web application running on Docker Compose. The application uses the Flask framework and maintains a hit counter in Redis. While the sample uses Python, the concepts demonstrated here should be understandable even if you're not familiar with it.

## Step 1: Setup

Define the application dependencies.

Create a directory for the project:

    $ mkdir composetest
    $ cd composetest

Create a file called app.py in your project directory and paste this in:

    import time

    import redis
    from flask import Flask

    app = Flask(__name__)
    cache = redis.Redis(host='redis', port=6379)

    def get_hit_count():
        retries = 5
        while True:
            try:
                return cache.incr('hits')
            except redis.exceptions.ConnectionError as exc:
                if retries == 0:
                    raise exc
                retries -= 1
                time.sleep(0.5)

    @app.route('/')
    def hello():
        count = get_hit_count()
        return 'Hello World! I have been seen {} times.\n'.format(count)

In this example, redis is the hostname of the redis container on the application's network. We use the default port for Redis, 6379.

Create another file called requirements.txt in your project directory and paste this in:

    flask
    redis

## Step 2: Create a Dockerfile

In this step, you write a Dockerfile that builds a Docker image. The image contains all the dependencies the Python application requires, including Python itself.

In your project directory, create a file named Dockerfile and paste the following:

    FROM python:3.7-alpine
    WORKDIR /code
    ENV FLASK_APP=app.py
    ENV FLASK_RUN_HOST=0.0.0.0
    RUN apk add --no-cache gcc musl-dev linux-headers
    COPY requirements.txt requirements.txt
    RUN pip install -r requirements.txt
    EXPOSE 5000
    COPY . .
    CMD ["flask", "run"]

This tells Docker to:
- Build an image starting with the Python 3.7 image.
- Set the working directory to /code.
- Set environment variables used by the flask command.
- Install gcc and other dependencies
- Copy requirements.txt and install the Python dependencies.
- Add metadata to the image to describe that the container is listening on port 5000
- Copy the current directory . in the project to the workdir . in the image.
- Set the default command for the container to flask run.

## Step 3: Define services in a Compose file

Create a file called docker-compose.yml in your project directory and paste the following:

    version: "{{ site.compose_file_v3 }}"
    services:
      web:
        build: .
        ports:
          - "5000:5000"
      redis:
        image: "redis:alpine"

This Compose file defines two services: web and redis.

Web service
The web service uses an image that's built from the Dockerfile in the current directory. It then binds the container and the host machine to the exposed port, 5000. This example service uses the default port for the Flask web server, 5000.

Redis service
The redis service uses a public Redis image pulled from the Docker Hub registry.

## Step 4: Build and run your app with Compose

From your project directory, start up your application by running docker-compose up.

    $ docker-compose up
    

