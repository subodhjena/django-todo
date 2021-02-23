# Overview

This article is part 1 of the series I am writing on developing DRF applications using Docker. We will be creating a TODO API server throughout the series. We'll be also using the Postgres database as a database for the project.

Containerizing the project will mainly help us with two aspects

1. Ease of deployment
2. Ease of setting a local development environment with a production-like setup

Assuming you have a Django project do the following things in order to get started

## 1. Create Dockerfile

```docker
FROM python:3.8
ENV PYTHONUNBUFFERED 1

# Allows docker to cache installed dependencies between builds
COPY ./requirements.txt requirements.txt
RUN pip install -r requirements.txt

# Adds our application code to the image
COPY . code
WORKDIR code

EXPOSE 8000

# Run the production server
CMD newrelic-admin run-program gunicorn --bind 0.0.0.0:$PORT --access-logfile - todo.wsgi:application
```

## 2. Create a docker-compose file

```docker
version: "3.4"

services:
  postgres:
    image: postgres:11.6
  web:
    restart: always
    environment:
      - DJANGO_SECRET_KEY=local
    image: web
    build: ./
    command: >
      bash -c "python wait_for_postgres.py &&
               ./manage.py migrate &&
               ./manage.py runserver 0.0.0.0:8000"
    volumes:
      - ./:/code
    ports:
      - "8000:8000"
    depends_on:
      - postgres
```

## 3. Local Development

Start the dev server for local development:

```bash
docker-compose up
```

Run a command inside the docker container:

```bash
docker-compose run --rm web [command]
```

The source code for this article can be fetched by cloning the _feature/part1_ branch [here](https://github.com/subodhjena/django-todo/tree/feature/part1)

> _Without wasting too much time you can easily create a DRF project with best practices using the cookie-cutter generator for DjangoREST Framework. You can download the cookiecutter project from [here](https://github.com/agconti/cookiecutter-django-rest)_
