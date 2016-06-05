---
layout: post
title:  "Creating a Local Development Environment with Docker"
date:   2016-06-06 08:00:00
categories: docker
comments: true
---

In the past, setting up a local development environment involved installing a few of the key infrastructure components on my development computer and then making some compromises on things that were either too much of a pain to setup or had similar light-weight equivalents that didn't compromise the systems behavior.

For example, I would always have my target database installed on my computer. But I might use a light-weight messaging system (ActiveMQ or whatever) and a simple cache instead of the enterprise system that might be the deployment target (WebSphere MQ and Redis, for example).

Vagrant is a much better approach. With Vagrant you can create a configuration for an environment and share it across developers. That eliminates a lot of problems. However, using Vagrant still requires quite a bit of configuration to install and use components like MySQL and Redis. This can be simplified with tools like Ansible, Puppet or Salt, but it's still a lot of configuration before you can get started.

Docker and Docker Hub provide images of many of the most common components used in web and enterprise applications. They work "out of the box", so it's much easier to get started and refine things as you go. Also, the repository images are updated often, so it's often easier to move from one version to the next.

Whether or not you're using Docker for deployment, Docker makes an excellent tool for building a local development environment and a CI environment (more on that in a future post).

In this post, I want to discuss my approach for creating a development environment using Docker tooling.

My current system uses the following major components:
1. Angular JS (1.x)
1. Spring Boot
1. Redis
1. MySQL

Just substitute your application environment, database, etc. and I think it's a template for a common web app configuration.

In order to have an environment that closely mirrors production, I want the versions of MySQL and Redis to mirror production. It's a pain to update these things whenever the production versions change and using a shared environment has always been problematic in my experience.

I also like to be able to vary how I run my business logic, because sometimes I want to run Spring Boot within my IDE for debugging purposes, and sometimes I want to run everything in a production configuration and hit that with my web browser or automated testing tool.

This is where Docker Compose comes in. Docker Compose allows me to create different compositions of Docker containers based on how I want to run my application. I use two primary compositions during development:

1. Application, Redis and MySQL in Docker
1. Application in IDE and Redis and MySQL in Docker

## Docker Machine

I use Docker Machine to run Docker on my computer (Mac).

I then also setup my `/etc/hosts` file to give that machine easy to remember names:

```
192.168.99.100	app
192.168.99.100	mysql
192.168.99.100	redis
```

Use `docker-machine ls` to find out the IP address of your machine and then you can plug it into your hosts file.

My application is configured to use `mysql` for the MySQL host name and `redis` for the Redis host name in development. The settings are different for each environment.

## Running in Docker

When I'm running everything in Docker, I use a compose file like this:

```
---
# A composition for running the complete application in Docker
app:
  image: objectuser/run-java-jar
  environment:
    SPRING_PROFILES_ACTIVE: dev
  links:
    - "mysql"
    - "redis"
  volumes:
    - "../../../build/ci:/app"
  ports:
    - "8080:8080"
mysql:
  image: mysql:5.7.12
  environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: dev
  ports:
    - "3306:3306"
redis:
  image: redis:2.8.23
  ports:
    - "6379:6379"
```

Note:
* The image `objectuser/run-java-jar` is [an image I've made](https://github.com/objectuser/run-java-jar) for running a Java JAR file. It works great for Spring Boot.
* The `links` item provides a way to map a network name to another container in the composition.

I can then point my browser to `app` to use my application.

## Running in an IDE and Docker

When I'm running my app in my IDE, I don't want to host the app in the machine, so I have a shorter Docker Compose file:

```
---
# Run a MySQL and Redis instance in Docker.
mysql:
  image: mysql:5.7.12
  environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: dev
  ports:
    - "3306:3306"
redis:
  image: redis:2.8.23
  ports:
    - "6379:6379"
```

I then need to start my machine, and then start the composition within that machine. If you're in the same directory as your Docker Compose file, and it's called `docker-compose.yml`, then:

```
docker-machine start dev
eval $(docker-machine env dev)
docker-compose up
```

Once that's started, I can start the app within my IDE and it uses the Docker-hosted MySQL and Redis servers.


## Conclusion

Outside of the benefits to running Docker in production, Docker is a great development tool that can help teams share a common configuration, stay in sync with production and reduce the amount of configuration needed to get up and running.

