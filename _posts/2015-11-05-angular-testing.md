---
layout: post
title:  "Angular JS Testing in Docker"
date:   2015-11-05 12:00:00
categories: angular
comments: true
---

I've been working on a project that uses Angular JS for the web front end. Angular has a lot of support for testing and I wanted to automate those tests in CI.

I was fairly successful getting my tests to run in Docker using PhantomJS. However, PhantomJS was always problematic and in the end it was more trouble than it was worth. Even the Protractor developers [recommend against it](https://github.com/angular/protractor/blob/master/docs/browser-support.md).

Thanks to snippets from around the web, I was able to get Chrome running headless within Docker. I use my [angular-test](https://github.com/objectuser/angular-test) Docker image for running tests and have updated it to install Chrome. 

There are a few steps needed to get Chrome working headless in both Karma and Protractor tests. These are:

1. Install Chrome
1. Install Xvfb
1. Tell Karma where Chrome is
1. Run Xvfb to create a display port and set the DISPLAY variable

## Install Chrome in Ubuntu

My Docker image is derived from the [Java](https://hub.docker.com/_/java/) image, which is based on Debian. To install Chrome, I did [this](https://github.com/objectuser/angular-test/blob/master/install-chrome.sh):

```
curl -O https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
dpkg -i google-chrome-stable_current_amd64.deb
apt-get -f install -y
```

The second statement exits with a non-zero exit code, but the subsequent statement resolves the dependency issues.

## Install Xvfb

Just install the `xvfb` package:

```
RUN apt-get install -y xvfb
```

## Tell Karma where Chrome Is

You can use `which google-chrome` to find the binary. Then tell Karma:

```
export CHROME_BIN=/usr/bin/google-chrome
```

## Run Xvfb on a Virtual Display

This creates and sets a virtual display that Chrome will render to. We don't need to see it, we just need it to run.

```
Xvfb :10 -ac &
export DISPLAY=:10
```

## Summary

PhantomJS works for some cases, but there are far too many gotchas that ended up wasting days of my time. Chrome, however, seems to work all the time.

If you don't want to setup all of this yourself, you can use my [angular-test](https://github.com/objectuser/angular-test) Docker image, which sets everything up and provides a hook for actually running your test suite.
