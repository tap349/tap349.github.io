---
layout: post
title: Docker (Legacy)
date: 2017-04-06 17:58:18 +0300
access: public
comments: true
categories: [docker]
---

<!-- more -->

* TOC
{:toc}
<hr>

getting started
---------------

`docker` deals with containers, `docker-compose` deals with services.

### codeschool

1. <http://campus.codeschool.com/courses/try-docker/level/1/section/1/containers--images>

- container (linux container) - isolated environment that
  can run code while sharing a single operating system

  run container and execute command inside running container:

  ```sh
  $ docker container run -p 80:80 httpd:2.4
  $ docker container ls
  $ docker container exec <container-name> du -mh
  $ docker container exec -it <container-name> /bin/bash
  ```

- image - a blueprint for creating a container

  pre-built images are available in [Docker Hub](hub.docker.com).
  custom image can be built using _Dockerfile_ (which contains instructions
  to create custom image).

  _Dockerfile_ example:

  ```dockerfile
  FROM httpd:2.4
  EXPOSE 80
  RUN apt-get update
  RUN apt-get install -y fortunes
  LABEL maintainer="moby-dock@example.com"
  ```

  build custom image and run container using that image:

  ```sh
  $ docker image build -t web-server:1.0 .
  $ docker image ls
  $ docker container run -d -p 80:80 web-server:1.0
  ```

- volume - exposes files on the host to the container (it's like mounting
  host directory inside the container)

  - copy files to container:

    ```sh
    $ docker container cp page.html <container-name>:/usr/local/apache2/htdocs/
    ```

  - copy files to custom image in _Dockerfile_:

    ```dockerfile
    RUN ...
    COPY page.html /usr/local/apache2/htdocs/
    ```

  - use data volume to share data:

    ```sh
    $ docker run -d -p 80:80 -v /my-files:/usr/local/apache2/htdocs web-server:1.0
    ```

### docker-compose

1. <http://stackoverflow.com/a/35585573/3632318>

_.docker-compose.yml_ file lists services along with their images, commands
and volumes (= mounted directories).

service is used to run command within specified image, service is run by 1+
containers.

for `docker-compose` commands to work _.docker-compose.yml_ file must be
present in current or one of parent directories.

- `docker-compose up`

  create and start containers for all services (pull required Docker images
  from Docker hub if necessary).

  use `-d` option to do it in the background:

  `docker-compose up -d`

- `docker-compose start/stop/restart app`

  start/stop/restart specified service.

- `docker-compose ps`

  list containers (name, command, state, ports):

  ```
         Name                      Command               State           Ports
  -------------------------------------------------------------------------------------
  server_app_1          rackup -o 0.0.0.0 -p 4321        Up      0.0.0.0:4321->4321/tcp
  server_db_1           docker-entrypoint.sh postgres    Up      5432/tcp
  ```

  use `-a` option to show all containers (only running containers are shown by
  default):

  `docker ps -a`

- `docker-compose logs app`

  view output from all containers or from containers of specified service.

  `app` above is a service name.

  use `-f` option to follow log output (= `tail -f`):

  `docker-compose logs -f app`

  use `-t` option to show timestamps:

  `docker-compose logs -t app`

- `docker-compose run app sh`

  run a one-off command on a service - service name must be specified.

  if command (like `sh`) is not specified service command is run.

  there are no `-t` and `-i` options just like for `docker run` to start
  interactive session - container is run in attached mode by default.

  use `-d` option to run container (and command inside it) in the background:

  `docker-compose run -d app sh`

  use `-p` option to publish container's port(s) to the host:

  `docker-compose run -p <host_port>:<container_port> app sh`

- `docker-compose exec app sh`

  execute a command in a running container - service name must be specified.

  command MUST be specified.

  there are no `-t` and `-i` options just like for `docker exec` to start
  interactive session - command is run in attached mode by default.

  use `-d` option to run command in the background:

  `docker-compose exec -d app service motd start`

  NOTE: using `-d` option doesn't make sense for all commands: e.g. for
        command above it doesn't matter if `-d` option is used or not at
        all - the result is the same. or else using `-d` option for `sh`
        command causes all output to disappear from screen (yet control
        is not returned to the host).

### docker

- `docker images`/`docker image ls`

  list images.

- `docker ps`

  list containers (container id, image, command, created, status, ports, names):

  ```
  CONTAINER ID   IMAGE       COMMAND                  PORTS                    NAMES
  732******500   v***n/n**   "rackup -o 0.0.0.0..."   0.0.0.0:4321->4321/tcp   server_app_1
  e0e******c7c   postgres    "docker-entrypoint..."   5432/tcp                 server_db_1
  ```

  note that `CREATED` and `STATUS` columns have been removed from output for
  the sake of readability.

- `docker logs server_app_1`

  same as `docker-compose logs app` but for container instead of service.

- `docker build -t v***n/n** .`

  build image from _Dockerfile_ (the latter must be present in CWD).
  it's necessary to rebuild the whole image when you make any changes that
  must be persisted (for `app` service: when you modify ruby source code).

- `docker run -ti v***n/n** sh`

  NOTE: any changes made within image (like installing packages, etc.)
        are lost when container exits (except changes in mounted volume)!

  run command in a new container - image name must be specified.

  if command (like `sh`) is not specified default command is run
  (argument passed to `CMD` in base image's _Dockerfile_).

  use `-t` option to allocate pseudo-TTY (without it no prompt is displayed).

  use `-i` option to make session interactive (without it session exits after
  the first user input inside container).

  use `-v` to bind mount a volume inside container:

  `docker run -ti -v <local_dir>:<remote_dir> v***n/n** sh`

- `docker exec -ti 732******500 /bin/bash`

  run command in a running container - container id from `docker ps` output
  must be specified (unlike image name for `docker run`).

  command MUST be specified.

  use `-i` and `-t` options in the same way as for `docker run`.

tips
----

### debugging inside container

this is just one way and probably not the most efficient one:

- set breakpoint by making `byebug` call somewhere in your code
- stop `app` container

  `docker-compose stop app`

- run shell command in `app` container

  `docker-compose run -p 4321:1234 app sh`

- start rack server manually

  `rackup -o 0.0.0.0 -p 1234`

- refresh the page that would allow you to hit breakpoint
- start debugging inside running container as you usually do
- exit from running container when debugging session is over

### ping another container

1. <https://docs.docker.com/v17.09/engine/userguide/networking>

Docker creates a network between all containers managed by the same
daemon so it's possible to access other containers by their names.

say, we have 2 services - `app` and `db`:

```sh
$ docker-compose exec app sh
# ping db
PING db (172.18.XXX.XXX): 56 data bytes
64 bytes from 172.18.XXX.XXX: icmp_seq=0 ttl=64 time=0.359 ms
```

troubleshooting
---------------

### Cannot connect to the Docker daemon. Is the docker daemon running on this host?

**solution**

just install Docker app via `brew cask`.

it's possible to install all the necessary docker tools (`boot2docker`,
`docker`, `docker-compose`, etc.) manually but I didn't manage to configure
them all in reasonable time. so it's better to install Docker app - it does
all the heavy lifting for you.

```sh
$ brew cask install docker
```

don't forget to run Docker app after installation (otherwise `docker` command
will not be available in terminal).

### lookup registry-1.docker.io on 192.168.65.1:53: no such host

when running `docker run hello-world` command host to pull `hello-world`
image from is not found.

**solution**

host lookup fails when using some DNS servers (in particular when using
DNS server of my Internet provider) - use Google DNS server 8.8.8.8 instead.
