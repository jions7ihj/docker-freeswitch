# Freeswitch 1.6 for Kubernetes w/ manifests

[![Build Status](https://travis-ci.org/telephoneorg/docker-freeswitch.svg?branch=master)](https://travis-ci.org/telephoneorg/docker-freeswitch) [![Docker Pulls](https://img.shields.io/docker/pulls/telephoneorg/freeswitch.svg)](https://hub.docker.com/r/telephoneorg/freeswitch) [![Size/Layers](https://images.microbadger.com/badges/image/telephoneorg/freeswitch.svg)](https://microbadger.com/images/telephoneorg/freeswitch) [![Github Repo](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/telephoneorg/docker-freeswitch)

## Maintainer
Joe Black | <me@joeblack.nyc> | [github](https://github.com/joeblackwaslike)


## Description
Minimal image with TLS & capture server support. Cache and database are also stored on a ramfs for best performance.  This image uses a custom, minimal version of Debian Linux.


## Build Environment
Build environment variables are often used in the build script to bump version numbers and set other options during the docker build phase.  Their values can be overridden using a build argument of the same name.


This image has extensive customizations built into it allowing you to easily customize freeswitch for your own needs while keeping the public image as light as possible.
* `ERLANG_VERSION`: defaults to `19.2`
* `FREESWITCH_VERSION`: defaults to `1.6`
* `KAZOO_CONFIGS_BRANCH`: defaults to `4.2`
* `FREESWITCH_INSTALL_MODS`: defaults to `commands,conference,console,dptools,dialplan-xml,enum,event-socket,flite,http-cache,local-stream,loopback,say-en,sndfile,sofia,tone-stream`
* `FREESWITCH_INSTALL_CODECS`: defaults to `amr,amrwb,g723-1,g729,h26x,ilbc,opus,shout,silk,siren,spandsp`
* `FREESWITCH_INSTALL_LANGS`: defaults to `en,es,ru`
* `FREESWITCH_INSTALL_META`: defaults to ``
* `FREESWITCH_INSTALL_DEBUG`: defaults to `false`
* `FREESWITCH_LOAD_MODS`: defaults to `FREESWITCH_LOAD_MODULES:-g729,silk`

The following variables are standard in most of our dockerfiles to reduce duplication and make scripts reusable among different projects:
* `APP`: freeswitch
* `USER`: freeswitch
* `HOME` /opt/freeswitch


## Run Environment
Run environment variables are used in the entrypoint script to render configuration templates, perform flow control, etc.  These values can be overridden when inheriting from the base dockerfile, specified during `docker run`, or in kubernetes manifests in the `env` array.
* `FREESWITCH_LOG_LEVEL`: defaults to `info`
* `FREESWITCH_LOG_COLOR`: defaults to `true`
* `FREESWITCH_RTP_PORT_RANGE`: defaults to `16384-32768`
* `FREESWITCH_DISABLE_NAT_DETECTION`: defaults to `true`
* `MOD_KAZOO_LISTEN_IP`: defaults to `0.0.0.0`
* `FREESWITCH_SEND_ALL_HEADERS`: defaults to `true`
* `FREESWITCH_CAPTURE_SERVER`: defaults to `false`
* `FREESWITCH_CAPTURE_IP`: defaults to `127.0.0.1`
* `FREESWITCH_ENABLE_TLS`: defaults to `false`

[todo] Finish describing these.


## Usage
### Under docker (pre-built)
All of our docker-* repos in github have CI pipelines that push to docker cloud/hub.  

This image is available at:
* https://store.docker.com/community/images/telephoneorg/freeswitch
* https://hub.docker.com/r/telephoneorg/freeswitch
* `docker pull telephoneorg/freeswitch`

To run:

```bash
docker run -d \
    --name freeswitch \
    -h freeswitch.local \
    -p "11000:10000" \
    -p "11000:10000/udp" \
    -p "16384-16484:16384-16484/udp" \
    -p "8031:8031" \
    -e "FREESWITCH_DISABLE_NAT_DETECTION=false" \
    -e "FREESWITCH_RTP_PORT_RANGE=16384-16484" \
    -e "ERLANG_COOKIE=test-cookie" \
    --cap-add IPC_LOCK \
    --cap-add SYS_NICE \
    --cap-add SYS_RESOURCE \
    --cap-add NET_ADMIN \
    --cap-add NET_RAW \
    --cap-add NET_BROADCAST \
    telephoneorg/freeswitch:latest
```

**NOTE:** Please reference the Run Environment section for the list of available environment variables.

**NOTE:** In production, you may want to run this container in the host's networking stack.  In that case remove the port mappings above and insert `--net=host`.  Under Kubernetes, if you are using our manifests, this is already taken care of.


### Under docker-compose
Pull the images
```bash
docker-compose pull
```

Start application and dependencies
```bash
# start in foreground
docker-compose up --abort-on-container-exit

# start in background
docker-compose up -d
```


### Under Kubernetes
Edit the manifests under `kubernetes/<environment>` to reflect your specific environment and configuration.

Create a secret for the erlang cookie:
```bash
kubectl create secret generic erlang --from-literal=erlang.cookie=$(LC_ALL=C tr -cd '[:alnum:]' < /dev/urandom | head -c 64)
```

Deploy freeswitch:
```bash
kubectl create -f kubernetes/<environment>
```
