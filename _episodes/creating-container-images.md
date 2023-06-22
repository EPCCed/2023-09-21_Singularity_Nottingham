---
title: "Creating Your Own Container Images"
teaching: 20
exercises: 15
questions:
- "How can I make my own Singularity container images?"
- "How do I document the 'recipe' for a Singularity container image?"
objectives:
- "Explain the purpose of a Singularity recipe file and show some simple examples."
- "Demonstrate how to build a Singularity container image from a recipe file."
- "Compare the steps of creating a container image interactively versus a recipe file."
- "Create an installation strategy for a container image."
keypoints:
- "Singularity recipe files specify what is within Singularity container images."
- "The `singularity build` command is used to build a container image from a recipe file."
- "`singularity build` requires admin/root privileges so usually needs to be prefixed with `sudo`."
---

There are lots of reasons why you might want to create your **own** Singularity container image.
- You can't find a container image with all the tools you need online or in local resources
- You want to have a container image to "archive" all the specific software versions you ran for a project.
- You want to share your workflow with someone else.

## Python 3 in Alpine Linux

Before creating a reproducible installation, let's start with a minimal Linux container image.
Create container from an `alpine` Docker container image:

~~~
singularity pull alpine.sif docker://alpine 
~~~
{: .language-bash}

~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob 31e352740f53 done
Copying config f4b9357049 done
Writing manifest to image destination
Storing signatures
2023/06/17 09:38:24  info unpack layer: sha256:31e352740f534f9ad170f75378a84fe453d6156e40700b882d737a8f4a6988a3
INFO:    Creating SIF file...
~~~
{: .output}

Now, start a shell in a container based on the new container image:

~~~
singularity shell alpine.sif
~~~
{: .language-bash}

Because this is a basic container, there's a lot of things not installed -- for
example, `python3`.

~~~
Singularity> python3
~~~
{: .language-bash}
~~~
/bin/sh: python3: not found
~~~
{: .output}

Python 3 is not provided by the `alpine` container image. However, the Alpine version of
Linux has a installation tool called `apk` that we can use to install Python 3. So, we could
build our own container image that adds Python 3 to the `alpine` container image. The command
we want to use during our build process will look like:

~~~
apk add --update python3 py3-pip python3-dev
~~~
{: .language-bash}

## Interactive installation

You may wonder why we cannot install Python 3 directly in the running container itself by
using the command above. If you try to do this, you will get an error:

~~~
ERROR: Unable to lock database: Read-only file system
ERROR: Failed to open apk database: Read-only file system
~~~
{: .output}
 
This is because the system directories where `apk` would install
Python are in read-only file system layers in the running container. Installing
software interactively is not ideal anyway from a reproducibility aspect as it 
makes it difficult to know exactly what process was followed to install the software
in the container image and track changes to this process over time and/or versions
of the container image.

> ## Writable container images
>
> There is a way to create an image in a way that can be written to
> but it is a bit convoluted and not as useful as you might first expect; due, in a large
> part to the reproducibility issues discussed above. To be able to install
> Python 3 in a running Alpine container we need to build and run the container in a different
> way. You need to use the `--sandbox` flag:
>
> ~~~
> singularity build --sandbox alpine-writable.sandbox docker://alpine
> ~~~
> {: .language-bash}
>
> Once the sandbox container image has been built, we need to open a shell in a container based
> on the sandbox container image:
>
> ~~~
> singularity shell --writable alpine-writable.sandbox
> ~~~
> {: .language-bash}
>
> Now, finally, we can use the `apk add --update python3 py3-pip python3-dev` command in
> the running container to install Python 3. Note, the installation will persist in
> the sandbox container image even if you shut down the running container and start
> a new one.
>
> If you then want to convert the sandbox container image to a standard container image
> you can use the build command:
>
> ~~~
> singularity build alpine-python.sif alpine-writable.sandbox
> ~~~
> {: .language-bash}
>
> This approach can be useful for exploring the install commands to use to create
> your container images but it is not generally a good way to create reproducible
> container images.
>
> [Singularity CE docs on sandbox images](https://docs.sylabs.io/guides/3.10/user-guide/build_a_container.html#creating-writable-sandbox-directories)
{: .callout}

## Put installation instructions in a Singularity recipe file

A Singularity recipe file is a plain text file with keywords and commands that
can be used to create a new container image. This is a much more reproducible approach
than installing things interactively as it allows us to have a record of exactly how we
installed software in the container image and, as it is plain text, it lends itself well
to being placed under version control (e.g. using git and Github/Gitlab) to track and manage
versions of the recipe file. We will create a simple recipe file
to create our own Singularity container image which is based on Alpine Linux with 
Python 3 installed.

Using your favourite text editor, create a file called `alpine-python.def` and add 
the following lines:

~~~
Bootstrap: docker
From: alpine:latest

%post
    apk add --update python3 py3-pip python3-dev

%runscript
    python3 --version
~~~

Let's break this file down:

- The first line, `Bootstrap: docker`, tells Singularity that we want to start the build from an existing Docker
  container image.
- The next line, `From: alpine:latest`, indicates which container image we are starting from.  It is the "base"
  container image we are going to start from. As no Docker container image repository URL is specified, Singularity 
  assumes that this is a container image from Docker Hub.
- The `%post` section specifies commands to run as part of the image build process. In this case we use the
  Alpine Linux `apk` command to install Python3.
- The last section, `%runscript`, indicates the default action we want a container based on this container image
  to run (when used with `singularity run`). In this case, we ask the container to return the version of Python
  installed in the container.

## Create a new Singularity container image

So far, we only have a text file named `alpine-python.def` -- we do not yet have a container image.
We want Singularity to take this recipe file, run the installation commands contained within it, and then save the
resulting container as a new container image. To do this we will use the `singularity build` command.

We have to provide `singularity build` command with two pieces of information:
- the name of the new container image file
- the name of the recipe file

As we are building a container image we need admin/root privileges so we need to use the `sudo` command to 
run our `singularity build` command.

> ## sudo Password
> As you are using `sudo`, you may be asked by the system for your password when you run this
> command. Your system will typically ask for the password when using `sudo` for the first time
> after an expiry period is reached (this can be every 5 mins but is sometimes longer, it
> depends on the system you are using).
{: .callout} 

All together, the build command that you should run on your computer, will have a similar structure to this:

~~~
sudo singularity build <container image file name> <recipe file name>
~~~
{: .language-bash}

For example, if my recipe is in the file `alpine-python.def` and I wanted to call my
container image file `alpine-python.sif`, I would use this command:

~~~
sudo singularity build alpine-python.sif alpine-python.def
~~~
{: .language-bash}

~~~
INFO:    Starting build...
2023/06/17 10:20:54  info unpack layer: sha256:31e352740f534f9ad170f75378a84fe453d6156e40700b882d737a8f4a6988a3
INFO:    Running post scriptlet
+ apk add --update python3 py3-pip python3-dev
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/community/x86_64/APKINDEX.tar.gz
(1/27) Installing libbz2 (1.0.8-r5)
(2/27) Installing libexpat (2.5.0-r1)
(3/27) Installing libffi (3.4.4-r2)
(4/27) Installing gdbm (1.23-r1)
(5/27) Installing xz-libs (5.4.3-r0)
(6/27) Installing libgcc (12.2.1_git20220924-r10)
(7/27) Installing libstdc++ (12.2.1_git20220924-r10)
(8/27) Installing mpdecimal (2.5.1-r2)
(9/27) Installing ncurses-terminfo-base (6.4_p20230506-r0)
(10/27) Installing libncursesw (6.4_p20230506-r0)
(11/27) Installing libpanelw (6.4_p20230506-r0)
(12/27) Installing readline (8.2.1-r1)
(13/27) Installing sqlite-libs (3.41.2-r2)
(14/27) Installing python3 (3.11.4-r0)
(15/27) Installing python3-pycache-pyc0 (3.11.4-r0)
(16/27) Installing pyc (0.1-r0)
(17/27) Installing py3-setuptools-pyc (67.7.2-r0)
(18/27) Installing py3-pip-pyc (23.1.2-r0)
(19/27) Installing py3-parsing (3.0.9-r2)
(20/27) Installing py3-parsing-pyc (3.0.9-r2)
(21/27) Installing py3-packaging-pyc (23.1-r1)
(22/27) Installing python3-pyc (3.11.4-r0)
(23/27) Installing py3-packaging (23.1-r1)
(24/27) Installing py3-setuptools (67.7.2-r0)
(25/27) Installing py3-pip (23.1.2-r0)
(26/27) Installing pkgconf (1.9.5-r0)
(27/27) Installing python3-dev (3.11.4-r0)
Executing busybox-1.36.1-r0.trigger
OK: 142 MiB in 42 packages
INFO:    Adding runscript
INFO:    Creating SIF file...
INFO:    Build complete: alpine-python.sif
~~~
{: .output}

> ## Exercise: Review!
>
> 1. Think back to earlier. What command can you run to check if your container image file was created
> successfully?
>
> 2. What command will run a container based on the container image you've created and perform the default action?
>
> 3. Can you make it do something different, like print "hello world"?
>
> > ## Solution
> >
> > 1. To see your new image, run `ls`. You should see the name of your new
> > container image file listed.
> >
> > 2. We want to use `singularity run alpine-python.sif` to run a container based on a container image and perform the default action.
> >
> > 3. We want to use `singularity exec alpine-python.sif echo "hello world"` to run a container based on a container image and perform the default action.
> >
> > {: .language-bash}
> {: .solution}
{: .challenge}

While it may not look like you have achieved much, you have already effected the combination of a lightweight Linux operating system with your specification to run a given command that can operate reliably across different platforms.

## Boring but important notes about installation

There are a lot of choices when it comes to installing software -- sometimes too many!
Here are some things to consider when creating your own container image:

- **Start smart**, or, don't install everything from scratch! If you're using Python
as your main tool, start with a [Python container image](https://hub.docker.com/_/python). Same with [R](https://hub.docker.com/r/rocker/r-ver/). We've used Alpine Linux as an example
in this lesson, but it's generally not a good container image to start with for initial development and experimentation because it is
a less common distribution of Linux; using [Ubuntu](https://hub.docker.com/_/ubuntu), [Debian](https://hub.docker.com/_/debian) and [CentOS](https://hub.docker.com/_/centos) are all
good options for scientific software installations. The program you're using might
recommend a particular distribution of Linux, and if so, it may be useful to start with a container image for that distribution.
- **How big?** How much software do you really need to install? When you have a choice,
lean towards using smaller starting container images and installing only what's needed for
your software, as a bigger container image means longer download times to use.
- **Know (or Google) your Linux**. Different distributions of Linux often have distinct sets of tools for installing software. The `apk` command we used above is the software package installer for Alpine Linux. The installers for various common Linux distributions are listed below:
    - Ubuntu: `apt` or `apt-get`
    - Debian: `deb`
    - CentOS: `yum`
  Most common software installations are available to be installed via these tools.
  A web search for "install X on Y Linux" is usually a good start for common software
  installation tasks; if something isn't available via the Linux distribution's installation
  tools, try the options below.
- **Use what you know**. You've probably used commands like `pip` or `install.packages()`
before on your own computer -- these will also work to install things in container images (if the basic scripting
language is installed).
- **README**. Many scientific software tools have a README or installation instructions
that lay out how to install software. You want to look for instructions for Linux. If
the install instructions include options like those suggested above, try those first.

In general, a good strategy for installing software is:
- Make a list of what you want to install.
- Look for pre-existing container images.
- Read through instructions for software you'll need to install.
- Create a Singularity recipe file and then try to build the container image from that.

## Sharing your Singularity container images

You have a few different options available to share your container image files with other people,
including:

- Place them on shared storage on the shared facility so other can access them
- Place on external storage that is publicly available via the web
- Add them to an online repository such as the Sylabs Cloud Library

{% include links.md %}
