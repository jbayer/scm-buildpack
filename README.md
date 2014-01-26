# Git Buildpack

a simple buildpack that pulls application code in from a remote git server

## Motivation

You may want your application source code pulled in to your buildpack processor from your git server of choice (such as Github) rather than be required to push your application code in or use a git location that is assigned to you (such as with Heroku). You can use the git buildpack in conjunction with the [heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi#heroku-buildpack-multi) and other buildpacks of your choosing to build your application.

## Usage:

* have an app with a .buildpacks file in the root that has 1 line for each buildpack. the 1st line is likely the git-buildpack and subsequent lines should be other buildpacks that ready your app code to run
* have a .gitbuildpack file in the root of the app so the git-buildpack bin/detect script returns 0
* set a GIT_URL environment variable with the URL to your git repository

## Example

Create a directory for our app:

```
$ mkdir git-app
$ cd git-app
```

Reference an app source code on Github. You'll need a file named .gitbuildpack (can be empty) and a .buildpacks file that uses the git-buildpack and the buildpack that your app should use after the git repo has been cloned.

```
$ echo -e "https://github.com/jbayer/git-buildpack.git\nhttps://github.com/ryandotsmith/null-buildpack.git" > .buildpacks
$ touch .gitbuildpack
$ ls -al
total 8
drwxr-xr-x   4 jamesbayer  wheel  136 Jan 26 08:10 .
drwxrwxrwt  20 root        wheel  680 Jan 26 08:01 ..
-rw-r--r--@  1 jamesbayer  wheel   94 Jan 26 07:50 .buildpacks
-rw-r--r--   1 jamesbayer  wheel    0 Jan 26 08:10 .gitbuildpack
```

Now push the app, remembering to set the GIT_URL variable to the git repository you want to clone from. In this case we use --no-route because this is a simple worker app printing hello ever few seconds and doesn't bind to a port.

```
$ gcf push hello --no-route -m 64M -b https://github.com/ddollar/heroku-buildpack-multi.git --no-start
$ gcf set-env hello GIT_URL https://github.com/jbayer/hello-world-app.git
$ gcf start hello
Starting app hello in org jbayer-normal-org / space development as jbayer+normal@gopivotal.com...
-----> Downloaded app package (4.0K)
OK
Initialized empty Git repository in /tmp/buildpacks/heroku-buildpack-multi/.git/
=====> Downloading Buildpack: https://github.com/jbayer/git-buildpack.git
=====> Detected Framework: Git buildpack detected
=====> starting git buildpack compile
=====> dir is /tmp/buildpackTMEPH
=====> url is https://github.com/jbayer/hello-world-app.git
=====> branch is is
=====> Downloading git repo: https://github.com/jbayer/hello-world-app.git
=====> final contents of build dir
total 36
drwxr--r-- 3 vcap vcap  4096 Jan 26 16:40 .
drwxr-xr-x 5 vcap vcap  4096 Jan 26 16:40 ..
-rwxr--r-- 1 vcap vcap    94 Jan 26 16:39 .buildpacks
-rwxr--r-- 1 vcap vcap     0 Jan 26 16:39 .gitbuildpack
-rw-r--r-- 1 vcap vcap 11325 Jan 26 16:40 LICENSE
-rw-r--r-- 1 vcap vcap    17 Jan 26 16:40 Procfile
-rw-r--r-- 1 vcap vcap    76 Jan 26 16:40 README.md
drwxr-xr-x 2 vcap vcap  4096 Jan 26 16:40 bin
=====> Downloading Buildpack: https://github.com/ryandotsmith/null-buildpack.git
=====> Detected Framework: Null
-----> Nothing to do.
       Using release configuration from last framework Null:
       --- {}
-----> Uploading droplet (8.0K)

1 of 1 instances running

App started

Showing health and status for app hello in org jbayer-normal-org / space development as jbayer+normal@gopivotal.com...
OK

requested state: started
instances: 1/1
usage: 64M x 1 instances
urls:

     state     since                    cpu    memory        disk
#0   running   2014-01-26 08:40:48 AM   0.0%   948K of 64M   84K of 1G
```

Now you can check the logs to see if the app is working as expected.

```
$ gcf logs hello
Connected, tailing logs for app hello in org jbayer-normal-org / space development as jbayer+normal@gopivotal.com...

2014-01-26T08:39:30.69-0800 [API]     OUT Created app with guid 184ab665-1d05-4994-a85d-2e980e891ec3
2014-01-26T08:40:25.01-0800 [API]     OUT Updated app with guid 184ab665-1d05-4994-a85d-2e980e891ec3 ({"environment_json"=>"PRIVATE DATA HIDDEN"})
2014-01-26T08:40:42.37-0800 [STG]     OUT total 36
2014-01-26T08:40:42.38-0800 [STG]     OUT drwxr--r-- 3 vcap vcap  4096 Jan 26 16:40 .
2014-01-26T08:40:42.38-0800 [STG]     OUT drwxr-xr-x 5 vcap vcap  4096 Jan 26 16:40 ..
2014-01-26T08:40:42.38-0800 [STG]     OUT -rwxr--r-- 1 vcap vcap    94 Jan 26 16:39 .buildpacks
2014-01-26T08:40:42.38-0800 [STG]     OUT -rwxr--r-- 1 vcap vcap     0 Jan 26 16:39 .gitbuildpack
2014-01-26T08:40:42.38-0800 [STG]     OUT -rw-r--r-- 1 vcap vcap 11325 Jan 26 16:40 LICENSE
2014-01-26T08:40:42.38-0800 [STG]     OUT -rw-r--r-- 1 vcap vcap    17 Jan 26 16:40 Procfile
2014-01-26T08:40:42.38-0800 [STG]     OUT -rw-r--r-- 1 vcap vcap    76 Jan 26 16:40 README.md
2014-01-26T08:40:42.38-0800 [STG]     OUT drwxr-xr-x 2 vcap vcap  4096 Jan 26 16:40 bin
2014-01-26T08:40:42.38-0800 [STG]     OUT =====> Downloading Buildpack: https://github.com/ryandotsmith/null-buildpack.git
2014-01-26T08:40:42.38-0800 [STG]     OUT =====> Detected Framework: Null
2014-01-26T08:40:42.38-0800 [STG]     OUT -----> Nothing to do.
2014-01-26T08:40:42.38-0800 [STG]     OUT        Using release configuration from last framework Null:
2014-01-26T08:40:42.38-0800 [STG]     OUT        --- {}
2014-01-26T08:40:42.76-0800 [STG]     OUT -----> Uploading droplet (8.0K)
2014-01-26T08:40:47.55-0800 [DEA]     OUT Registering app instance (index 0) with guid 184ab665-1d05-4994-a85d-2e980e891ec3
2014-01-26T08:40:48.77-0800 [App/0]   OUT hello
2014-01-26T08:40:48.77-0800 [App/0]   ERR
2014-01-26T08:40:53.68-0800 [App/0]   OUT hello
2014-01-26T08:40:58.69-0800 [App/0]   OUT hello
2014-01-26T08:41:03.69-0800 [App/0]   OUT hello
2014-01-26T08:41:08.69-0800 [App/0]   OUT hello
2014-01-26T08:41:13.70-0800 [App/0]   OUT hello
2014-01-26T08:41:18.70-0800 [App/0]   OUT hello

```

## Issues

* save the repo in the app's buildpack cache dir and just fetch changes
* support branches and authentication
* do not require GIT_URL env variable if the .gitbuildpack file has repo information
* more info logs
