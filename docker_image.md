# What's in an Image
- App binaries and dependencies
- Metadata about the image data and how to run the image
- Official definition: An Image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime
- Not a complete OS. No kernel

# history and inspect commands
- docker image history nginx:latest
- docker image inspect nginx ; will show metadata

# Image layers

|Image|
|-----|
|Env  |
|Apt  |
|Ubuntu|  

|One  | Another|
|-----|------|
|Port |      |
|Copy | Copy | 
|Apt  | Apt  |
|   Jessie(Shared)   |

# Copy on write
- Container is a single read/write layer on the top of the image
- Each container copy files from Image and store them in its container

# Tag
- docker pull nginx ; using default tag: latest
- docker pull nginx:1.11.9 ; download specific version
- docker pull nginx:1.11.9-alpine ; download specific version with specific OS, alpine

# Tagging
- docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
- docker image tag nginx sijoonlee/nginx
- docker image tag nginx sijoonlee/nginx:testing

# Upload Newly-tagged image
- docker login 
- cat .docker/config.json ; where authorization key is stored
- docker image push sijoonlee/nginx
- docker logout

# Build an image
- docker image build [OPTIONS] PATH | URL | -
ex) docker image build --tag nginx-with-html .


Dockerfile Example

```
# NOTE: this example is taken from the default Dockerfile for the official nginx Docker Hub Repo
# https://hub.docker.com/_/nginx/
# NOTE: This file is slightly different than the video, because nginx versions have been updated 
#       to match the latest standards from docker hub... but it's doing the same thing as the video
#       describes
FROM debian:stretch-slim
# all images must have a FROM
# usually from a minimal Linux distribution like debian or (even better) alpine
# if you truly want to start with an empty container, use FROM scratch

ENV NGINX_VERSION 1.13.6-1~stretch
ENV NJS_VERSION   1.13.6.0.1.14-1~stretch
# optional environment variable that's used in later lines and set as envvar when container is running

# RUN script will be executed inside of container
RUN apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y gnupg1 \
	&& \
	NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
	found=''; \
	for server in \
		ha.pool.sks-keyservers.net \
		hkp://keyserver.ubuntu.com:80 \
		hkp://p80.pool.sks-keyservers.net:80 \
		pgp.mit.edu \
	; do \
		echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
		apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
	done; \
	test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
	apt-get remove --purge -y gnupg1 && apt-get -y --purge autoremove && rm -rf /var/lib/apt/lists/* \
	&& echo "deb http://nginx.org/packages/mainline/debian/ stretch nginx" >> /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y \
						nginx=${NGINX_VERSION} \
						nginx-module-xslt=${NGINX_VERSION} \
						nginx-module-geoip=${NGINX_VERSION} \
						nginx-module-image-filter=${NGINX_VERSION} \
						nginx-module-njs=${NJS_VERSION} \
						gettext-base \
	&& rm -rf /var/lib/apt/lists/*
# optional commands to run at shell inside container at build time
# this one adds package repo for nginx from nginx.org and installs it

RUN ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log
# forward request and error logs to docker log collector
# docker has the log colleting feature

EXPOSE 80 443
# expose these ports on the docker virtual network
# you still need to use -p or -P to open/forward these ports on host

CMD ["nginx", "-g", "daemon off;"]
# required: run this command when container is launched
# only one CMD allowed, so if there are multiple, last one wins

```

- sudo docker image build -t customnginx .
[-t] : Tag
[.] : where dockerfile is

- Result
```
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM debian:stretch-slim
---> fa41698012c7 
...[some result]... 
### ---> fa41698012c7 is hash code
### if this doesn't change, docker won't build this again next time
Step 2/7 : ENV NGINX_VERSION 1.13.6-1~stretch
...[some result]...
Step 3/7 : ENV NJS_VERSION   1.13.6.0.1.14-1~stretch
...[some result]...
Step 4/7 : RUN apt-get update 	&& apt-get install --no-install-recommends --no-install-suggests -y gnupg1 	&& 	NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; 	found=''; 	for server in 		ha.pool.sks-keyservers.net 		hkp://keyserver.ubuntu.com:80 		hkp://p80.pool.sks-keyservers.net:80 		pgp.mit.edu 	; do 		echo "Fetching GPG key $NGINX_GPGKEY from $server"; 		apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; 	done; 	test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; 	apt-get remove --purge -y gnupg1 && apt-get -y --purge autoremove && rm -rf /var/lib/apt/lists/* 	&& echo "deb http://nginx.org/packages/mainline/debian/ stretch nginx" >> /etc/apt/sources.list 	&& apt-get update 	&& apt-get install --no-install-recommends --no-install-suggests -y 	nginx=${NGINX_VERSION} 						nginx-module-xslt=${NGINX_VERSION} 						nginx-module-geoip=${NGINX_VERSION} 		nginx-module-image-filter=${NGINX_VERSION} 						nginx-module-njs=${NJS_VERSION} 						gettext-base 	&& rm -rf /var/lib/apt/lists/*
Step 5/7 : RUN ln -sf /dev/stdout /var/log/nginx/access.log 	&& ln -sf /dev/stderr /var/log/nginx/error.log
...[some result]...
Step 6/7 : EXPOSE 80 443
 ...[some result]...
Step 7/7 : CMD ["nginx", "-g", "daemon off;"]
...[some result]...
Successfully built 74b57a57013e
Successfully tagged customnginx:latest
```

# Extending Official images  
```
# this shows how we can extend/change an existing official image from Docker Hub

FROM nginx:latest
# highly recommend you always pin versions for anything beyond dev/learn

WORKDIR /usr/share/nginx/html
# change working directory to root of nginx webhost
# using WORKDIR is preferred to using 'RUN cd /some/path'

COPY index.html index.html
# copy local file to container

# I don't have to specify EXPOSE or CMD because they're in my FROM
```