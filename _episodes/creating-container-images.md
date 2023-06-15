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
---

There are lots of reasons why you might want to create your **own** Singularity container image.
- You can't find a container image with all the tools you need online or in local resources
- You want to have a container image to "archive" all the specific software versions you ran for a project.
- You want to share your workflow with someone else.

## Interactive installation

Before creating a reproducible installation, let's experiment with installing
software inside a container. Start a container from an `alpine` Docker container image interactively:

~~~
singularity shell docker://alpine 
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

## Put installation instructions in a Singularity recipe file

A Singularity recipe file is a plain text file with keywords and commands that
can be used to create a new container image. We will create a simple recipe file
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
{: .output}

Let's break this file down:

- The first line, `Bootstrap: docker`, tells Singularity that we want to start the build from an existing Docker container image
- The next line, `From: alpine:latest`, indicates which container image we are starting from.  It is the "base" container image we are going to start from. With the line above, this is a container image from Docker Hub.
- The `%post` section specifies commands to run as part of the image build process. In this case we use the Alpine Linux `apk` command to install Python3.
- The last section, `%runscript`, indicates the default action we want a
container based on this container image to run. In this case, we ask the container to return the version of Python installed in the container.

## Create a new Singularity container image

So far, we only have a text file named `alpine-python.def` -- we do not yet have a container image.
We want Singularity to take this recipe file, run the installation commands contained within it, and then save the
resulting container as a new container image. To do this we will use the
`singularity build` command.

We have to provide `singularity build` command with two pieces of information:
- the name of the new container image file
- the name of the recipe file

All together, the build command that you should run on your computer, will have a similar structure to this:

~~~
$ singularity build <container image file name> <recipe file name>
~~~
{: .language-bash}

For example, if my recipe is in the file `alpine-python.def` and I wanted to call my
container image file `alpine-python.sif`, I would use this command:

~~~
$ singularity build alpine-python.sif alpine-python.def
~~~
{: .language-bash}


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

{% include links.md %}
