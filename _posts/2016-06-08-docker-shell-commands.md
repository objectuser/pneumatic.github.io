---
layout: post
title:  "Making Docker Shell Commands a Bit Easier"
date:   2016-06-08 08:00:00
categories: docker
comments: true
---

Docker has a nice command line interface, but I wanted them to be a bit less verbose.

I work with Docker a lot from the command prompt (Bash). After typing longer Docker commands a few times, I had to make things a bit easier. I wanted to share some of the Bash aliases and functions I use in case they might be helpful.

First, I have some aliases I use to make things a bit shorter:

```bash
alias dkr=docker
alias dkr-cmp='docker-compose'
alias dkr-mcn='docker-machine'
alias dkr-mcn-create='docker-machine create --driver virtualbox'
alias dkr-mcn-stop='docker-machine stop $DOCKER_MACHINE_NAME'
```

The `dkr-mcn-stop` alias assumes there is a machine in the current environment. When you start a machine and set your environment, the `DOCKER_MACHINE_NAME` variable is set.

Then I have some functions I find useful:

```bash
# Stop all running containers
function dkr-stop-all {
  for c in $(docker ps -aq); do
    docker stop $c
  done
}

# Stop and remove all running containers
function dkr-rm-all {
  dkr-stop-all
  for c in $(docker ps -aq); do
    docker rm $c
  done
}

# Remove all images in the current environment
function dkr-rmi-all {
  dkr-stop-all
  dkr-rm-all
  for c in $(docker images -q); do
    docker rmi $c
  done
}

# Stop everything and remove the given image
# This is useful when developing an image
function dkr-down {
  echo 'Stopping all containers'
  dkr-stop-all
  echo 'Removing all containers'
  dkr-rm-all
  echo "Removing image: $1"
  docker rmi $1
}

# Start a machine and set it to the environment
function dkr-mcn-start {
  dkr-mcn start $1
  eval $(dkr-mcn env $1)
}

# Open up a shell into a container
function dkr-bash {
  docker run -t -i $1 /bin/bash
}
```

The `dkr-mcn-start` function is nice for starting up a machine in the current shell. I use it all the time.

I also use `dkr-rm-all` each time I restart my application if I want a clean environment.

I'd me very interested to know if anyone else has commands or aliases they use to make working with Docker a bit more productive.
