# SCM Buildpack

a simple buildpack that pulls application code in from a remote location. currently supports:

* remote git urls
* remote http resources ending in `.zip`, `.tgz`, `.tar.gz`, and `.war`

it is intended to be used in conjunction with [heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi#heroku-buildpack-multi) and other user-specified buildpacks in a `.buildpacks` file. 

## Motivation

You may want your appl source code pulled in to your buildpack compatible PaaS from your git server of choice (such as Github) or a remote archive. Contrast this experience with being required use a git server that is assigned to you (such as with Heroku) or have to push your application source code into the PaaS (Cloud Foundry) over a potentially slow upload link.

## Usage

* create an app directory with a `.buildpacks` file in the root path that has 1 buildpack url per line. the 1st line is likely the scm-buildpack and subsequent lines should be other buildpacks that ready your app code to run
* have a `.scmbuildpack` file in the root of the app so the scm-buildpack `bin/detect` script returns a 0 exit code
* set an `SCM_URL` environment variable with the URL to your git repository or your http URL that ends in `.tgz` or `.zip`
* optionally set an `SCM_BRANCH` environment variable with the branch name you want to use

The `SCM_URL` supports optional authentication when using `username:password@` format such as `https://username:password@blob.example.com/some-file.zip` or `https://username:somepassword@github.com/username/some-repo.git`

## Example

Create a directory for our app:

```
$ mkdir git-app
$ cd git-app
```

Reference an app source code on Github. You'll need a file named `.scmbuildpack` (can be empty) and a `.buildpacks` file that references the scm-buildpack and other buildpack(s) that your app should use after the scm buildpack retrieves your app source files.

```
$ echo -e "https://github.com/jbayer/scm-buildpack.git\nhttps://github.com/cloudfoundry/ruby-buildpack.git" > .buildpacks
$ touch .scmbuildpack
```

Alternatively, use a `manifest.yml` file like this if you prefer to have a `cf push` experience without CLI options:

```
---
applications:
- name: git-app
  memory: 512M
  buildpack: https://github.com/ddollar/heroku-buildpack-multi.git
  path: .
  env:
     SCM_URL: https://github.com/jbayer/hello-sinatra.git
```

Sample directory listing before the push.

```
$ ls -al
total 16
drwxr-xr-x   5 jamesbayer  wheel  170 Feb  3 11:41 .
drwxrwxrwt  23 root        wheel  782 Feb  3 10:27 ..
-rw-r--r--@  1 jamesbayer  wheel   94 Feb  3 11:40 .buildpacks
-rw-r--r--   1 jamesbayer  wheel    0 Feb  3 11:41 .scmbuildpack
-rw-r--r--@  1 jamesbayer  wheel  190 Feb  3 11:44 manifest.yml
```

Now push the app, remembering to set the `SCM_URL` env variable to the URL you want to use.

Below is the syntax using multiple CLI commands with options:

```
$ cf push
Using manifest file /private/tmp/git-app/manifest.yml

Creating app git-app in org jbayer-org / space development as jbayer@gopivotal.com...
OK

Using route git-app.a1-app.cf-app.com
Binding git-app.a1-app.cf-app.com to git-app...
OK

Uploading git-app...
Uploading from: /private/tmp/git-app
661, 3 files
OK

Starting app git-app in org jbayer-org / space development as jbayer@gopivotal.com...
OK
-----> Downloaded app package (4.0K)


Initialized empty Git repository in /tmp/buildpacks/heroku-buildpack-multi/.git/
=====> Downloading Buildpack: https://github.com/jbayer/scm-buildpack.git
=====> Detected Framework: SCM
=====> SCM_URL is https://github.com/jbayer/hello-sinatra.git
=====> SCM_BRANCH is master
=====> Cloning remote git repository
       Initialized empty Git repository in /tmp/cache/scm/.git/
=====> build dir listing
       total 56
       drwxr--r-- 3 vcap vcap  4096 Feb  3 19:52 .
       drwxr-xr-x 5 vcap vcap  4096 Feb  3 19:52 ..
       -rwxr--r-- 1 vcap vcap   101 Feb  3 19:52 .buildpacks
       -rwxr--r-- 1 vcap vcap     0 Feb  3 19:52 .scmbuildpack
       -rw-r--r-- 1 vcap vcap    43 Feb  3 19:52 Gemfile
       -rw-r--r-- 1 vcap vcap   260 Feb  3 19:52 Gemfile.lock
       -rw-r--r-- 1 vcap vcap 11325 Feb  3 19:52 LICENSE
       -rw-r--r-- 1 vcap vcap    37 Feb  3 19:52 Procfile
       -rw-r--r-- 1 vcap vcap   105 Feb  3 19:52 README.md
       drwxr-xr-x 2 vcap vcap  4096 Feb  3 19:52 assets
       -rw-r--r-- 1 vcap vcap    41 Feb  3 19:52 config.ru
       -rw-r--r-- 1 vcap vcap   248 Feb  3 19:52 foo.rb
       -rwxr--r-- 1 vcap vcap   106 Feb  3 19:52 manifest.yml
=====> Downloading Buildpack: https://github.com/cloudfoundry/heroku-buildpack-ruby.git
=====> Detected Framework: Ruby/Rack
-----> Using Ruby version: ruby-1.9.3
-----> Installing dependencies using Bundler version 1.3.2
       Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin --deployment
       Fetching gem metadata from http://rubygems.org/..........
       Fetching gem metadata from http://rubygems.org/..
       Installing rack (1.5.1)
       Installing rack-protection (1.3.2)
       Installing tilt (1.3.3)
       Installing sinatra (1.3.4)
       Using bundler (1.3.2)
       Your bundle is complete! It was installed into ./vendor/bundle
       Cleaning up the bundler cache.
-----> WARNINGS:
       You have not declared a Ruby version in your Gemfile.
       To set your Ruby version add this line to your Gemfile:"
       ruby '1.9.3'"
       # See https://devcenter.heroku.com/articles/ruby-versions for more information."
       Using release configuration from last framework Ruby/Rack:
       ---
       addons: []
       default_process_types:
         rake: bundle exec rake
         console: bundle exec irb
         web: bundle exec rackup config.ru -p $PORT
-----> Uploading droplet (23M)

1 of 1 instances running

App started

Showing health and status for app git-app in org jbayer-org / space development as jbayer@gopivotal.com...
OK

requested state: started
instances: 1/1
usage: 512M x 1 instances
urls: git-app.a1-app.cf-app.com

     state     since                    cpu    memory          disk
#0   running   2014-02-03 11:52:55 AM   0.0%   68.5M of 512M   51.9M of 1G
```

Notice that if you push the app again that this time it will fetch changes instead of cloning the repo again. 

## Issues and Roadmap

* support authentication in a way that doesn't put credentials in log output 
* do not require SCM_URL env variable if the .gitbuildpack file has information
* support other remote transports like FTP/SCP
* investigate `git clone URL --depth 1` as either an option or the default behavior
