# Docker installation of IronMQ & IronAuth

Some basic familiarity with Docker is not necessary, but you
should make sure that [docker is installed.](http://docs.docker.com/installation/#installation)

[The full docker CLI docs](https://docs.docker.com/reference/commandline/cli/)

### Getting the images

```
$ docker pull iron/mq
$ docker pull iron/auth
```


### Start

reference:

* `-d` runs as daemon, omit to run in foreground
* `-p HOST:CONTAINER` binds to port on host machine
* `-e KEY=value` add an environment variable to the container

```
// MQ
$ docker run -d -p 8080:8080 --net=host iron/mq

// Auth
$ docker run -d -p 8090:8090 --net=host iron/auth
```

Each of these configurations are minimal. You are welcome to further modify
the docker environment for each of these containers, e.g. by naming them.
Also, data inside of each of these containers is ephemeral to the lifetime of
the container. If you would like to persist the data, the recommended way
is to create a persistent data volume container that is mounted where you
would like to store data. Here is a brief example, see Docker docs for details:

```
// create the data container, mounted on host at /mnt/data 
// note: ironmq and ironauth both store their data at /ironmq/data in container by default
$ docker run -name irondata -v /mnt/data:/ironmq/data busybox true

// run mq with data volume
$ docker run -d --volumes-from irondata -e CONFIG_JSON="`cat path/to/mq_config.json`" iron/mq
```

### [Get started](getting_started.md)

### Modify a config

You can download or copy & paste these to your file system:

[MQ config](config_mq.json)
[Auth config](config_auth.json)

These configs are good to get you up and running (they're the same as
the ones in the docker container) but you can modify them.

To start mq or auth with the updated config, simply pass it through the
environment variable `CONFIG_JSON`. An example with mq:

```
$ docker run -d -p 8090:8090 --net=host -e CONFIG_JSON="`cat path/to/mq_config.json`" iron/mq
```
