# Git Buildpack

a simple buildpack that pulls application code in from a remote git server

## Motivation

You may want your application source code pulled in to your buildpack processor from your git server of choice (such as Github) rather than be required to push your application code in or use a git location that is assigned to you (such as with Heroku). You can use the git buildpack in conjunction with the [heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi#heroku-buildpack-multi) and other buildpacks of your choosing to build your application.

## Usage

Create a directory for our app:

```bash
$ mkdir -p myapp/bin
$ cd myapp
```

Here is an example app that will run on 64bit linux machine, like [this one](https://github.com/jbayer/hello-world-app):

```bash
$ echo -e "#\!/usr/bin/env bash\n while true; do    echo 'hello';    sleep 5; done" > ./bin/program
$ echo -e "web: bin/program" > Procfile
$ chmod +x ./bin/program
$ ./bin/program
hello world
```

Reference an app source code on Github:

```bash
$ cd /tmp
$ git clone https://github.com/jbayer/hello-world-app.git
$ git push heroku master
$ heroku run program
Running `program` attached to terminal... up, run.8663
hello world
```

## Motivation

I wanted to run various executables (e.g. [log-shuttle](https://github.com/ryandotsmith/log-shuttle)) on Heroku without compiling them on Heroku. Thus, I compile programs on my linux 64 machine, or fetch the binary from the project, commit them to a repo and then run them on Heroku with the Ø buildpack.

## Issues

You will need to make sure that a 64bit linux machine can execute the binary.
