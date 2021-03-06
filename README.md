## Supported tags and respective `Dockerfile` links

+ [`2.7.0`,`latest` (2.7.0/Dockerfile)](https://github.com/opinioapp/ember-cli/blob/2.7.0/Dockerfile)
+ [`2.6.2` (2.6.2/Dockerfile)](https://github.com/opinioapp/ember-cli/blob/2.6.2/Dockerfile)
+ [`2.6.1` (2.6.1/Dockerfile)](https://github.com/opinioapp/ember-cli/blob/2.6.1/Dockerfile)
+ [`2.5.1` (2.5.1/Dockerfile)](https://github.com/opinioapp/ember-cli/blob/2.5.1/Dockerfile)
+ [`2.5.0` (2.5.0/Dockerfile)](https://github.com/opinioapp/ember-cli/blob/2.5.0/Dockerfile)
+ [`2.4.3` (2.4.3/Dockerfile)](https://github.com/danlynn/ember-cli/blob/2.4.3/Dockerfile)
+ [`2.4.2` (2.4.2/Dockerfile)](https://github.com/danlynn/ember-cli/blob/2.4.2/Dockerfile)
+ [`2.3.0` (2.3.0/Dockerfile)](https://github.com/danlynn/ember-cli/blob/2.3.0/Dockerfile)
+ [`1.13.15` (1.13.15/Dockerfile)](https://github.com/danlynn/ember-cli/blob/1.13.15/Dockerfile)
+ [`1.13.14` (1.13.14/Dockerfile)](https://github.com/danlynn/ember-cli/blob/1.13.14/Dockerfile)
+ [`1.13.13` (1.13.13/Dockerfile)](https://github.com/danlynn/ember-cli/blob/1.13.13/Dockerfile)
+ [`1.13.8` (1.13.8/Dockerfile)](https://github.com/danlynn/ember-cli/blob/1.13.8/Dockerfile)
+ [`1.13.1` (1.13.1/Dockerfile)](https://github.com/danlynn/ember-cli/blob/1.13.1/Dockerfile)


This image is a fork of : [danlynn/ember-cli](https://registry.hub.docker.com/u/danlynn/ember-cli/) (many thanks)

This image contains everything you need to have a working development environment for ember-cli.  The container's working dir is /myapp so that you can setup a volume mapping your project dir to /myapp in the container.

ember-cli v2.7.0 + node 4.4.7 + npm 2.15.9 + bower 1.7.9 + phantomjs 2.1.1 + watchman 3.5.0

![ember-cli logo](https://raw.githubusercontent.com/opinioapp/ember-cli/master/logo.png)


## How to use

Setup a project to use this container via [docker-compose](https://www.docker.com/products/docker-compose).  docker-compose is part of the all-in-one [docker-toolbox](https://www.docker.com/products/overview#/docker_toolbox) which is the easiest way to get up and running with docker.

1. Create new project dir and add a docker-compose.yml file similar to the following:

   ```
   ember: &defaults
     image: opinioapp/ember-cli:2.7.0
     volumes:
       - .:/myapp

   npm:
     <<: *defaults
     entrypoint: ['/usr/local/bin/npm']

   bower:
     <<: *defaults
     entrypoint: ['/usr/local/bin/bower', '--allow-root']

   server:
     <<: *defaults
     command: server --watcher polling
     ports:
       - "4200:4200"
       - "35729:35729"
   ```

2. Make sure that your docker-machine is already running:

	```
	$ docker-machine start default
	```
	
	Or, if you haven't created one yet:
	
	```
	$ docker-machine create --driver virtualbox default
	```

2. Create an ember app in the current dir:

	```
	$ docker-compose run --rm ember init
	```

3. Start the ember server:

   ```
   $ docker-compose up
   ```

   This launches the ember-cli server on port 4200 in the docker container. As you make changes to the ember webapp files, they will automagically be detected and the associated files will be recompiled and the browser will auto-reload showing the changes.
   
   Note that if you get an error something like
   
   ```
   server_1 | Error: A non-recoverable condition has triggered.  Watchman needs your help!
   server_1 | The triggering condition was at timestamp=1450119416: inotify-add-watch(/myapp/node_modules/ember-cli/node_modules/bower/node_modules/update-notifier/node_modules/latest-version/node_modules/package-json/node_modules/got/node_modules/duplexify/node_modules/readable-stream/doc) -> The user limit on the total number of inotify watches was reached; increase the fs.inotify.max_user_watches sysctl
   server_1 | All requests will continue to fail with this message until you resolve
   server_1 | the underlying problem.  You will find more information on fixing this at
   server_1 | https://facebook.github.io/watchman/docs/troubleshooting.html#poison-inotify-add-watch
   ```
   
   Then watchman is running out of resources trying to track all the files in a large ember app.  To increase the `fs.inotify.max_user_watches` count to something that is more appropriate for an ember app, stop your docker-compose server by hitting ctrl-c (or `docker-compose stop server` if necessary) then execute the following command:
   
   ```
   $ docker run --rm --privileged --entrypoint sysctl opinioapp/ember-cli:2.7.0 -w fs.inotify.max_user_watches=524288
   ```
   
   Note that this will affect all containers that run on the current docker-machine from this point forward because `fs.inotify.max_user_watches` is a system-wide setting.  This shouldn't be a big deal however, so go ahead and give it a try.  Then start the docker-compose service again with
   
   ```
   $ docker-compose up
   ```

4. Launch the ember webapp:

   You will need to first determine the IP of the docker container:

   ```
   $ docker-machine ip default
   -or-
   $ boot2docker ip

   192.168.59.103
   ```

   Next open that ip address in your browser on port 4200:

   + http://192.168.59.103:4200

## Command Usage

The ember, bower, and npm commands can be executed in the container to effect changes to your local project dir as follows.  You basically put a "docker-compose run --rm" in front of any of the 3 commands and pass the normal command options as usual.

Example:

```
$ docker-compose run --rm npm install
$ docker-compose run --rm bower install bootstrap
$ docker-compose run --rm ember generate --pod model user
```

