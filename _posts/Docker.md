====== Docker basics ======

The official ''docker'' [[https://docs.docker.com/get-started/|documentation]] is a good place to start; below is a TLDR version of what you need to do to work with ''docker''.

===== Docker terminologies =====

Docker images are essentially virtual machines “operating systems”. With a docker image, you can then launch a docker container (i.e. your actual virtual machine), from which you can run jobs or customise further. Once a container is launched, it loads the image “operating system”, but **any changes in the container only affects the container and not the parent image nor other containers**. Analogously, when you purchase a MacBook and install an application on it, you have not changed the Mac OS for another person using another MacBook.

This is why docker images are useful for reproducible research.

===== Installing docker =====

Follow installation instructions for [[https://hub.docker.com/editions/community/docker-ce-desktop-mac|Mac]], [[https://hub.docker.com/editions/community/docker-ce-desktop-windows|Windows]] or [[https://hub.docker.com/editions/community/docker-ce-server-ubuntu|Linux]].

**If you use Windows 10 Home Edition, please check this other [[:dockertoolboxinstallation|link]] for setup information.**

====== Docker images ======

===== Create a docker image =====

Docker images are built using a ''%%Dockerfile%%'' (and this file must be named as such). More documentation is available [[https://docs.docker.com/get-started/part2/|here]]. We can inherit a base docker image and build off it. For instance, as seen in the following ''%%Dockerfile%%'', we inherit the ''%%ubuntu%%'' image from [[https://hub.docker.com/|Docker Hub]].

<code>
# Base image
FROM ubuntu

## create directories
RUN mkdir -p /research/setup

## Update ubuntu
RUN apt-get update
RUN apt-get install -y locales \
                      xml2 \
                      libcurl4-openssl-dev \
                      libxml2-dev \
                      libssl-dev \
                      ed \
                      less \
                      wget \
                      ca-certificates \
                      apt-transport-https \
                      software-properties-common \
                      gsfonts \
                      gnupg2 \
                      vim-tiny

# Configure default locale, see https://github.com/rocker-org/rocker/issues/19
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
	&& locale-gen en_US.utf8 \
	&& /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
</code>

Next, build a new image using the ''%%Dockerfile%%''. (For more details, see [[https://docs.docker.com/engine/reference/commandline/build/|this]]). The docker image is named using the ''%%-t%%'' option.

<code>
docker build [OPTIONS] PATH
</code>
For example, we can do:

<code>
docker build -t chualabnccs/base-raw .
</code>
===== Accessing docker images from a repository =====

At mentioned previously, it is possible to build off existing images in [[https://hub.docker.com/|Docker Hub]]. We can also use the same platform to share our docker images, but this is not recommended unless we have public applications to share.

===== Start a docker container =====
Next, we can run docker containers from images created previously.

<code>
docker run -it -d -p 8787:8787  -v $(pwd):/home/demo/ --name mycontainer --user $(id -u):$(id -g) chualabnccs/base-raw bash
</code>
The ''-i'' option opens an interactive shell, while the ''-p'' specifies the port mapping for accessing ''RStudio''. The ''-t'' and ''-d'' commands are to run the container in a way that keeps the container active. Don’t worry about the details here. More [[https://docs.docker.com/engine/reference/run/|information]] is available if desired. ''--name'' is for naming the container so that it will be easy to recognise your container.

The ''-v'' specifies to which source and destination folders to map. If the destination folder (e.g. ''/home/demo/'' in the above example) is not present, it is created when the container starts. For more information, check the ''docker run'' [[https://docs.docker.com/engine/reference/run/|documentation]].

To ensure any created files in mapped volume of the container can be accessed locally, also specify the user and group mapping with "--user $(id -u):$(id -g)".

In summary, the above command starts a container in interactive ''%%bash%%'' mode, using the ''%%chualabnccs/base-raw%%'' image. Files and folders from the local present working directory are mapped to the ''%%/research/%%'' folder of the container.

If you’re using Windows, some of these commands may not work as previously described. Please check this other [[:dockertoolboxinstallation|documentation]] for more information on how to set up docker on your machine.

===== Adding packages in the container =====

It is sometimes not possible to include all necessary packages into the ''%%Dockerfile%%'' to initialise an image. For instance, sometimes an interactive installation is required or preferred. We can therefore customise the container and then save it as a separate container.

==== Start a docker container (see above) ====
First, create a container running a base docker image. Then, you can install and add packages as you wish. For example, you can add packages by running a bash script, or an R script, or interactively.
<code>
bash /demo/installpackages.sh
Rscript /demo/installpackages.R
</code>
==== Install anaconda interactively ====
<code>
cd /tmp/
wget https://repo.anaconda.com/archive/Anaconda3-2019.07-Linux-x86_64.sh
bash Anaconda3-2019.07-Linux-x86_64.sh
</code>
==== Save the container to an updated docker image ====
<code>
docker commit [container-ID] chualabnccs/base-ubuntu
</code>

Make sure to use your ''%%container-ID%%''. You can determine that by running ''docker ps -a''.

===== Upload docker image to a repository =====

Avoid this for now, but this is a placeholder just to mention that it is possible to share docker images on a public repository.

===== Save docker image to a file =====

Saving a docker image to a file makes it easy to share images with others. The size of the images also helps with estimating the disk size to allocate to docker.

<code>
docker save -o base-ubuntu.tar chualabnccs/base-ubuntu
</code>
===== Load docker image =====

With a docker image file, we can load docker images to another computer. This recreates the exact same image.

<code>
docker load -i base-ubuntu.tar
</code>
===== Delete docker image =====

List all available docker images with:

<code>
docker images
</code>
If you would like you remove any docker image, simply type:

<code>
docker rmi [IMAGE ID]
</code>
where ''IMAGE ID'' can be found in the list of images from ''docker images''. You will not be able to remove an image if there is a container using it. First stop and remove the container before removing the image.

====== Docker containers ======

Once docker images are created, we can start containers with these images as the operating system. See above on how to start a docker container.

===== Viewing all containers =====

You can check all active and stopped containers on docker by:

<code>
docker ps -a
</code>
===== Restarting a container =====

You can restart an exited container by:

<code>
docker start [CONTAINER ID]
</code>
where ''%%[CONTAINER ID]%%'' is obtained by checking the containers listed by ''%%docker ps -a%%''.

===== Opening a terminal in container =====

Once a container is running, you can reopen a terminal with:

<code>
docker exec -it [CONTAINER ID] bash
</code>
===== Delete containers =====

It is useful to periodically check the list of containers. An exited container still takes up space. If you do not need to use the container any longer, you can delete it with:

<code>
docker rm [CONTAINER ID]
</code>
To purge all stopped containers, do:

<code>
docker rm $(docker ps -a -q)
</code>
====== Accessing RStudio using docker ======
===== The 0.0.0.0:8787 port is NOT in use =====

First check if any ports are currently in use by another docker container. To do that, type:
<code>
docker ps -a
</code> 

If the ''0.0.0.0:8787'' port is not in use (see the ''PORTS'' column), then you can proceed as follows. 
  
The ''%%chualabnccs/ubuntu-base%%'' has RStudio server installed, so that it is possible to run RStudio within any container running this image. “Server” is a bit of a misnomer – no internet access is necessary; this is just how the container sends information to the local machine to access RStudio.

All files in the mapped folder (see the input to the ''%%-v%%'' option in ''%%docker run%%'') are accessible to the container.

Once a container is up and running, and you have launched an interactive terminal, you can check if there are any RStudio servers running.

<code>
rstudio-server active-sessions
</code>
If there are no active sessions, run:

<code>
rstudio-server start
</code>
Once an active rstudio session is initiated, you can access RStudio from a web browser using either ''localhost:8787'' (Mac or Linux) or ''%%[virtual machine predefined IP]:8787%%''. If accessing from a remote machine, use ''%%[remote machine IP]:8787%%''. (Note that ''%%8787%%'' is the default port used by RStudio. The actual port to access RStudio depends on what was set in the ''%%-p%%'' option.)

If necessary, login to the server with ''%%rstudio%%'' and ''%%password%%'' for the userid and password respectively.

===== The 0.0.0.0:8787 port IS in use =====
If the 8787 port is already in use, start your container with a different port mapping. (This in general may be a good idea as port switching after a container has been created is painful and requires ''root'' access.)

===== Cannot start container due to port clash =====
TBD: Need to update json file

====== Jupyter notebooks ======
===== Setting up =====
If you're using 'R' with ''jupyter'', you first have to set up the appropriate kernel. Launch the version of 'R' you're planning to use, then
<code>
install.packages('IRkernel')
IRkernel::installspec()  # to register the kernel in the current R installation
</code>
If this has already been done in the ''docker'' image (e.g. for ''chualabnccs/ubuntu-boutros''), then you need not repeat the process

===== Launching jupyter notebook =====
Run a ''docker'' container with the 4444 port (or your desired port) available for ''jupyter''. Note that you may encounter port clashes with your local jupyter, so using something uncommon like 4444 should be safe. An example of folder mapping is shown, but feel free to add more.

<code>
docker run -it -p 4444:4444 -v $(pwd):/research/tmp/ chualabnccs/ubuntu-boutros  bash
</code>

Then launch your jupyter notebook from the folder of choice.
<code>
jupyter notebook --ip=0.0.0.0 --port=4444 --allow-root
</code>

To view from your web browser, simply type ''http://localhost:4444/'' or replace with your choice folder from above. If a password is requested, simply type ''password''.