---
published: true
layout: post
title: "Setting Up a Rails App With Docker Compose"
author: "Sergio"
permalink: /2017/11/13/rails-docker-compose
date: '2017-11-13 16:16:01 -0600'
---

In my last post I explained a little bit about what Docker is and why it's so nifty and useful.

In this post I wanted to demonstrate some real-world Docker usage by setting up a simple Rails app within a Docker container. Let's get started!

## Useful Terms

Here are a couple of terms you'll see being thrown around in the Docker ecosystem:

**Dockerfile**: the Dockerfile contains the primary set of information and instructions for your Docker container. This is where you would set the base image for the container, and specify any instructions or commands for the container to boot up properly.

**Image:** an image is the fundamental snapshot of a container, and is generally the base for anything you'd like to build on top of your container. There are official vendor images like Ubuntu or Rails that you can use to bootstrap your container and are heavily supported, as well as user-made images that are created by the community and usually based off of an official vendor image.

**Container**: containers are the equivalent of a full-on Virtual Machine but extremely lighter. Containers only run the application itself as opposed to an entire operating system like a VM would.

These are a few key terms that will be mentioned a lot so it's nice to have a decent idea of what they mean and what they do in relation to Docker.

## Installation

You need to have the Docker engine installed on your machine to run Docker images and containers. You can [click this link](https://docs.docker.com/engine/installation/) and follow instructions to install Docker on your machine.

## Getting Started

```
mkdir DockerProject
```

Let's go ahead and create a new folder to build our Docker project. Feel free to name it something mundane or wonderful, I'll just name mine DockerProject.

## Creating a Dockerfile

```
cd DockerProject
touch Dockerfile
```

In order to run a container properly and build it into an image, we need a way to pass instructions to our container once it boots up so it can install and configure all the necessary files, directories, environment variables, and dependencies. 

To accomplish this, we're going to write out all of these instructions inside a `Dockerfile`.

> A `Dockerfile` is a text document that contains all the commands a user
> could call on the command line to assemble an image. Using `docker build`
> users can create an automated build that executes several command-line
> instructions in succession.

In our `Dockerfile`, type the following:

```dockerfile
FROM ruby:2.3.3
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
ADD Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
ADD . /myapp
```

This code will copy our application code inside an image that Docker will use to run the container along with Ruby, Bundler, and all of our dependencies.

Next we'll create a small Gemfile that will only be responsible for installing Rails, and will then be overwritten with `rails new` in just a moment. We'll also throw in an empty `Gemfile.lock` since our Dockerfile will need it to build our image.

```
touch Gemfile Gemfile.lock
```

Inside our Gemfile:

```ruby
source 'https://rubygems.org'
gem 'rails', '5.0.0.1'
```

## Docker Compose

```
touch docker-compose.yml
```

Since our simple application will be a distributed application consisting of a Rails app and a database, each of these services will be run inside their own containers. 

In order for our containers to collaborate correctly, we'll create a `docker-compose.yml` file that will describe the services that make up our app, and include the configuration needed to compose them together and expose the web app's port.

In our newly created `docker-compose.yml` file:

```
version: '3'
services:
  db:
    image: postgres
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

## Building a Docker Image

With our 4 files in place, now we can finally build our image.

In your terminal, run the command:

```
docker-compose run web rails new . --force --database=postgresql
```

This command will first build the image for our `web` service using the `Dockerfile` we created earlier, then it will generate a new rails app inside of our directory.

```bash
$ ls -l

total 64
-rw-r--r--   1 sergiomendoza  staff   222 Nov 17 21:21 Dockerfile
-rw-r--r--   1 sergiomendoza  staff  1738 Nov 17 21:27 Gemfile
-rw-r--r--   1 sergiomendoza  staff  4429 Nov 17 21:28 Gemfile.lock
-rw-r--r--   1 sergiomendoza  staff   374 Nov 17 21:27 README.md
-rw-r--r--   1 sergiomendoza  staff   227 Nov 17 21:27 Rakefile
drwxr-xr-x  10 sergiomendoza  staff   320 Nov 17 21:27 app
drwxr-xr-x   8 sergiomendoza  staff   256 Nov 17 21:28 bin
drwxr-xr-x  14 sergiomendoza  staff   448 Nov 17 21:27 config
-rw-r--r--   1 sergiomendoza  staff   130 Nov 17 21:27 config.ru
drwxr-xr-x   3 sergiomendoza  staff    96 Nov 17 21:27 db
-rw-r--r--   1 sergiomendoza  staff   211 Nov 17 21:25 docker-compose.yml
drwxr-xr-x   4 sergiomendoza  staff   128 Nov 17 21:27 lib
drwxr-xr-x   4 sergiomendoza  staff   128 Nov 17 21:38 log
drwxr-xr-x   9 sergiomendoza  staff   288 Nov 17 21:27 public
drwxr-xr-x   9 sergiomendoza  staff   288 Nov 17 21:27 test
drwxr-xr-x   7 sergiomendoza  staff   224 Nov 17 21:38 tmp
drwxr-xr-x   3 sergiomendoza  staff    96 Nov 17 21:27 vendor
```

