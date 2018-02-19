# Dockerizing GUI Apps

## What's the idea?

The idea is to *Dockerize* each application in ICLOAK to be able to accomplish two things : 

* Isolate each application with kernel namespaces
* Be able to update applications separately

As a side effect, we will also reduce the size of the core image of ICLOAK and also hence reduce the update size, and the boot time! 

## How are we doing it?

We're taking an Alpine Linux docker image (Super light) and installing the dependencies required to run the application. We then set the entry point of the image to the application itself. When running the docker image, we forward the X11 socket and any other devices (like sound) we need to. 

### Example

Here's a `Dockerfile` for LibreOffice. 

```yaml
FROM alpine:latest
LABEL maintainer "Jessie Frazelle <jess@linux.com>"

RUN apk --no-cache add \
	--repository https://dl-3.alpinelinux.org/alpine/edge/testing \
	libreoffice \
	ttf-dejavu

ENTRYPOINT [ "libreoffice" ]
```

Once you build this, you can run it like this : 

```shell
docker run -d \
	-v /etc/localtime:/etc/localtime:ro \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-e DISPLAY=unix$DISPLAY \
	-v $HOME/slides:/root/slides \
	-e GDK_SCALE \
	-e GDK_DPI_SCALE \
	--name libreoffice \
	libreoffice 								# Name of the image
```

## Challenges

* We're not sure of the increase in load time of the apps if we're not going to load them into the RAM from the get-go.
* We need to create an application to manage all these images
* Because the app will be running inside a docker image, we need to figure out how to "sync" files and settings between the host OS and the docker container. 

## Resources

[https://blog.jessfraz.com/post/docker-containers-on-the-desktop/](https://blog.jessfraz.com/post/docker-containers-on-the-desktop/)

[https://github.com/jessfraz/dockerfiles](https://github.com/jessfraz/dockerfiles)

[http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/](http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/)

[https://medium.com/@pigiuz/hw-accelerated-gui-apps-on-docker-7fd424fe813e](https://medium.com/@pigiuz/hw-accelerated-gui-apps-on-docker-7fd424fe813e)

[http://wiki.ros.org/docker/Tutorials/GUI](http://wiki.ros.org/docker/Tutorials/GUI)