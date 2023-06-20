---
title: "Using Singularity: quick review"
start: true
teaching: 30
exercises: 0
questions:
- "What did we cover on using Singularity containers?"
objectives:
- "Review content so far on using Singularity containers."
keypoints:
- "You can obtain Singularity container images from many different places, including Docker Hub."
- "Singularity stores downloaded image files in a local cache."
- "Using `singularity run` will invoke the default action built into the container."
- "`singularity exec` allows you to execute arbitrary commands in a container."
- "You can get interactive access inside a container using `singularity shell`."
- "Directories from the host system can be made available in a container by *binding* the location. Some directories are bound in the container by default."
- "You have the same permissions in the container as you have on the host system."
---

## Obtaining container images

Unless you are building your own container images (more on this later), you will typically obtain container images from an repository of container images. This could be a public repository such as Singularity Hub or a private location. For example, your organisation may provide container images for specific tasks or tools.

To download Singularity container images from online sources you use the `singularity pull` command. For example, to pull the `lolcow` container image from the Sylabs Cloud Library and save to the file `lolcow.sif` , you would use:

~~~
singularity pull lolcow.sif library://lolcow
~~~

You can also obtain container images from the vast store of Docker container images in public Docker container image repositories such as Docker Hub using the `singularity pull` command. For example, to create a Singularity container image file called `python-slim-3.9.6.sif` from the official Python 3.9.6 Slim Buster container image, you would use:

~~~
singularity pull python-slim-3.9.6.sif docker://python:3.9.6-slim-buster
~~~

## The Singularity container image cache

As well as storing the downloaded container image files in the file name specified, Singularity stores them in a local cache to avoid additional downloads. You can inspect the contents of the cache with

~~~
singularity cache list -v
~~~

and delete items from the cache with the `singularity cache clean` command.


## Using Singularity containers

There are 3 main commands used to run Singularity containers:

 - `singularity run my-image.sif` - run a container based on the container image in the `my-image.sif` file and invoke the default action built into the container.
 - `singularity exec my-image.sif <command>` - run a container based on the container image in the `my-image.sif` file and execute the command `<command>` in the running container.
 - `singularity shell my-image.sif` - run a container based on the container image in the `my-image.sif` file and open an interactive shell within the container.

## Files and directories in Singularity containers

The files and directories available in a running container will typically be a mix of those from the container image and those from the host system. Some files/directories from the host system will be available by default in the container - these are typically configured by the person who installed Singularity on the system we are running on. We can *bind* additional files or directories from the host system into the running container by using the `-B` option to the `singularity` command. For example, to bind the host directory `/mnt/c/Users/Andrew` into a container we would use:

~~~
singularity run -B /mnt/c/Users/Andrew my-image.sif
~~~

**Important:** You have the same permissions to read and edit files in the container as you had on the host system.
