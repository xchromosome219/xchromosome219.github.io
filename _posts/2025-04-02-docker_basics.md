---
layout:     post
title:      Docker
subtitle:   Basic
date:       2025-04-02
author:     HT Vvang
header-img: assets/img/2020-03-02-post_MeSH.jpg
catalog: true
tags:
    - Shell
    - Docker
---

# Docker Basics

The official [Docker documentation](https://docs.docker.com/get-started/) is a good place to start. Below is a TL;DR version of what you need to work with Docker.

## Docker Terminologies

Docker **images** are like virtual machine operating systems. From a Docker image, you can launch a **Docker container** (your actual virtual machine), from which you can run jobs or make customizations.

- Changes inside the container **do not affect the parent image** or other containers.
- Analogy: Installing an app on your MacBook doesn't change the Mac OS for someone else.

This isolation is why Docker is useful for **reproducible research**.

## Installing Docker

Follow installation instructions based on your OS:

- [Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
- [Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
- [Linux](https://hub.docker.com/editions/community/docker-ce-server-ubuntu)

> ⚠️ If you use **Windows 10 Home Edition**, check this alternative [setup guide](dockertoolboxinstallation).

## Docker Images

### Create a Docker Image

Create a file named `Dockerfile`. Here’s an example using the Ubuntu base image:

```Dockerfile
# Base image
FROM ubuntu

# Create directories
RUN mkdir -p /research/setup

# Update and install packages
RUN apt-get update
RUN apt-get install -y locales                       xml2                       libcurl4-openssl-dev                       libxml2-dev                       libssl-dev                       ed                       less                       wget                       ca-certificates                       apt-transport-https                       software-properties-common                       gsfonts                       gnupg2                       vim-tiny

# Configure locale
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen     && locale-gen en_US.utf8     && /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
```

Build the image with:

```bash
docker build -t chualabnccs/base-raw .
```

### Use Docker Images from a Repository

You can use existing images from [Docker Hub](https://hub.docker.com/). Sharing your own images is possible but not recommended unless for public use.

## Start a Docker Container

Example command:

```bash
docker run -it -d -p 8787:8787 -v $(pwd):/home/demo/   --name mycontainer --user $(id -u):$(id -g) chualabnccs/base-raw bash
```

Explanation:
- `-i`: interactive mode
- `-t`: allocate a pseudo-TTY
- `-d`: run in detached mode
- `-p`: map ports (e.g., for RStudio)
- `-v`: mount current directory
- `--name`: name the container
- `--user`: set user ID and group ID to avoid permission issues

> Windows users may need to refer to [this setup](dockertoolboxinstallation).

## Customizing Containers

### Install Packages (in a running container)

Run scripts inside the container:

```bash
bash /demo/installpackages.sh
Rscript /demo/installpackages.R
```

### Install Anaconda Interactively

```bash
cd /tmp/
wget https://repo.anaconda.com/archive/Anaconda3-2019.07-Linux-x86_64.sh
bash Anaconda3-2019.07-Linux-x86_64.sh
```

### Save Changes as a New Image

```bash
docker commit [container-ID] chualabnccs/base-ubuntu
```

Find container ID using:

```bash
docker ps -a
```

## Save or Load Docker Image

### Save Image to a File

```bash
docker save -o base-ubuntu.tar chualabnccs/base-ubuntu
```

### Load Docker Image

```bash
docker load -i base-ubuntu.tar
```

## Delete Docker Image

List images:

```bash
docker images
```

Delete image:

```bash
docker rmi [IMAGE ID]
```
> Make sure no container is using the image.

## Docker Containers

### View All Containers

```bash
docker ps -a
```

### Restart an Exited Container

```bash
docker start [CONTAINER ID]
```

### Open Terminal in Running Container

```bash
docker exec -it [CONTAINER ID] bash
```

### Delete Containers

Remove specific container:

```bash
docker rm [CONTAINER ID]
```

Remove all stopped containers:

```bash
docker rm $(docker ps -a -q)
```

## Accessing RStudio via Docker

### If Port 8787 is NOT in Use

Start a container with RStudio (e.g., `chualabnccs/ubuntu-base`):

```bash
docker run -it -d -p 8787:8787 -v $(pwd):/home/demo/ chualabnccs/ubuntu-base bash
```

Check for active RStudio sessions:

```bash
rstudio-server active-sessions
```

Start RStudio server if not running:

```bash
rstudio-server start
```

Access RStudio in your browser:

- **Mac/Linux**: `http://localhost:8787`
- **Remote**: `[remote machine IP]:8787`

Login credentials:
- **Username**: `rstudio`
- **Password**: `password`

### If Port 8787 is Already in Use

Use a different port (e.g., `-p 8888:8787`) when starting the container.

## Jupyter Notebooks in Docker

### Setup R Kernel for Jupyter

Inside R:

```r
install.packages('IRkernel')
IRkernel::installspec()
```

> If this is already done in the image, you can skip this step.

### Launch Jupyter Notebook

Run the container:

```bash
docker run -it -p 4444:4444 -v $(pwd):/research/tmp/ chualabnccs/ubuntu-boutros bash
```

Launch notebook server:

```bash
jupyter notebook --ip=0.0.0.0 --port=4444 --allow-root
```

View in browser: `http://localhost:4444/`

Login password: `password`
