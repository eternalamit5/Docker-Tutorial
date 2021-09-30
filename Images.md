# Images

### What's in an image (and what Isn't)

What's in

- App binaries and dependencies

- Metadata about the image data and how to run the image

  

  Binaries can be small (even one file) or even large binaries like ubuntu with apt

What's not

- Not a complete OS
- No kernel
- No kernel modules (ex: drivers)

Images comes in two variant in  https://hub.docker.com 

- official (always written "official" under the image repo icon)
- not-official

Official image is only one for the give repo and the name of the image is just one single word without any slash in their names. Typically **<repo_name>:< tag-name>**

Not-official images are many for a repo and the name of the image contains atleast two word with a slash. Typically it will be **< username>/<repo_name>:< tag-name>** or  **< organization_name>/<repo_name>: < tag-name>** 

Images is actually a tag. There can be multiple images or tags in a repo with different version of software.

:octopus: To view attributes pertaining to all image in host 

```bash
docker image ls
```

:octopus: To pull an image with specific version

```bash
docker pull <image-name>:<version>
```

:octopus: To pull latest image or default pull

```bash
docker pull <image-name>
```

- :bell: Note latest is just to indicate its latest tag. 



### Image layers & caching

Images are broken in multiple layers. In host we only store only one copy of each layer. This saves a lot of space in the host system. These layers are combined as described by docker-compose file to run instantiate a container. If any changes are made then more layers are created (only pertaining to the changes made). Only one copy of each layer is saved in host disk.

:octopus: To know the history of the image

```bash
docker history <image-name>
```

:bell: Note: in the command output, it mentions <missing> it does not mean something is missing or wrong. It just indicates these are the layers within the image and so they wont get SHA or image ID .

:octopus: Inspect the image, this give metadata about the image such as port opened, environment variables, processor architecture and many more important stuffs

```bash
docker image inspect <image-name>
```

How to recognize the image

Images are of the format

- official image:   <repo-name>:<tag-name>
- not official image: <user-name>/<repo-name>:<tag-name>

Tag are just a label in human readable format to a corresponding image ID. Image ID is SHA ID. Thus an image ID can be corresponded with one or more tag name.

The docker is checks the image ID and downloads it only if image is'nt there.  So it does not actually take account of tag name while downloading. While during image inspect it might show 2 images with multiple tag name , on close look at image ID they have same Image ID and are downloaded only once. This saves lot of disk space in host computer. 

:octopus: Tagging a image

```bash
docker image tag <image-name-old> <image-name-new>
```

- :bell: **Latest** tag does not mean latest, it just means default tag name.

:octopus: Pushing image

```bash
docker image push <image-name>
```

### Building Images

To build a image we need "Dockerfile" (this name is fixed). 

:octopus: Build docker image from current directory. Assuming Dockerfile is this directory.

```bash
docker build .
```

:octopus: To build docker image from files other than Dockerfile

```bash
docker build -f <path-to-Dockerfile> .
```

- -f option finds the the "Dockerfile" in the specified location

:octopus: build and tagging docker image

```bash
docker image build -t <tag-name> .
```

- ​	**-t** stands for tag

:octopus:  Re-tagging and pushing to docker hub

```bash
docker image tag <tag-name>:<version> <user-name>/<tag-name>:<version>
```

### Dockerfile

Let's look into the contents in the sample Dockerfile 

##### Sample-1

```dockerfile
# our base image (IMAGE LAYER)
FROM alpine:3.5

# Install python and pip (IMAGE LAYER)
RUN apk add --update py2-pip

# upgrade pip (IMAGE LAYER)
RUN pip install --upgrade pip

# copy requirement.txt file (IMAGE LAYER)
COPY requirements.txt /usr/src/app/

# install Python modules needed by the Python app (IMAGE LAYER)
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt

# copy files required for the app to run (IMAGE LAYER)
COPY app.py /usr/src/app/

# copy files required for the app to run (IMAGE LAYER)
COPY templates/index.html /usr/src/app/templates/

# tell the port number the container should expose (IMAGE LAYER)
EXPOSE 5000

# run the application 
CMD ["python", "/usr/src/app/app.py"]
```

- **FROM** is starting point for the image, base image. If we are starting from scratch without any base image, then we use **FROM  scratch**

- **RUN** executes command (like in the terminal)

- **COPY** copies the file from <host-path> to <container-path>

- **EXPOSE** exposes the port to host PC and thus the network

- **CMD** is the command run to launch application

  

##### Sample-2

```bash
FROM nginx
ENV AUTHOR=Docker

WORKDIR /usr/share/nginx/html
COPY Hello_docker.html /usr/share/nginx/html

CMD cd /usr/share/nginx/html && sed -e s/Docker/"$AUTHOR"/ Hello_docker.html > index.html ; nginx -g 'daemon off;'
```

- **ENV** sets environment variables in the container. Environment variables are usually as set of key/value pair use in configuration

- **WORKDIR** setup acurrent working directory within the container. This is equivalent to **RUN cd <path> ** into the directory

  

#### Using && in Run

By using **&&** in **RUN** we reduce the number of layers we build. Each **RUN** will lead to a separate layer and will have separate SHA value associated with. Every time something changes in that layer, SHA value changes and thus docker will rebuild the entire layer and layer below it and add it to its image cache.

In this example, sample-3 and sample-4 does the same job. But in Sample-4 RUN is split into 2 RUN command and hence extra layer compared Sample-3

##### Sample-3

```dockerfile
FROM ubuntu:18.04

WORKDIR /project
COPY . .

RUN apt-get update \
  && apt-get install -y python3-pip python3-dev \
  && cd /usr/local/bin \
  && ln -s /usr/bin/python3 python \
  && pip3 install --upgrade pip
  && apt-get install nano \
  && apt-get -y install git wget libncurses-dev flex bison gperf \
  && pip3 install setuptools pyserial click cryptography future pyparsing pyelftools \
  && apt-get -y install cmake ninja-build ccache \
  && apt-get -y install gawk gperf grep gettext python python-dev automake bison flex texinfo help2man libtool libtool-bin make \
  && pip3 install --user -r requirements.txt
```

##### Sample-4

```dockerfile
FROM ubuntu:18.04

WORKDIR /project
COPY . .

RUN apt-get update \
  && apt-get install -y python3-pip python3-dev \
  && cd /usr/local/bin \
  && ln -s /usr/bin/python3 python \
  && pip3 install --upgrade pip
  && apt-get install nano 
  
RUN apt-get -y install git wget libncurses-dev flex bison gperf \
  && pip3 install setuptools pyserial click cryptography future pyparsing pyelftools \
  && apt-get -y install cmake ninja-build ccache \
  && apt-get -y install gawk gperf grep gettext python python-dev automake bison flex texinfo help2man libtool libtool-bin make \
  && pip3 install --user -r requirements.txt
```

:sparkling_heart:  **Good practice**: Build changing layer separately and order is in lower section of the file. If changing or constantly updating layer is placed on top, this will cause all layers below it to build irrespective they are changed or not. Thus longer build cycles.

#### Application logging  in Docker

**Always direct application logging to stdout and stderr**. Avoid logging into log files inside container. docker provides logging drivers for handling application logs for all application in container. 

Below is the sample for directing the logs from access.log and error.log file to stdout and stderr respectively

```dockerfile
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
&& ln -sf /dev/stderr /var/log/nginx/error.log
```

##### Image Prune

Use "prune" commands to clean up images, volumes, build cache, and containers

:octopus:  To clean up just "dangling" images

```bash
docker image prune
```

:octopus: This will clean up all images from system

```bash
docker system prune
```

:octopus: This will remove all image which we are not using

```bash
docker image prune -a
```

:octopus: To view disk space

```bash
docker system df
```

