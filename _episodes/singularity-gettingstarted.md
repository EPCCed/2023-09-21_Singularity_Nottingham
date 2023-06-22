---
title: "Singularity: Getting started"
teaching: 30
exercises: 20
questions:
- "What is Singularity and why might I want to use it?"
objectives:
- "Understand what Singularity is and when you might want to use it."
- "Undertake your first run of a simple Singularity container."
keypoints:
- "Singularity is another container platform and it is often used in cluster/HPC/research environments."
- "Singularity has a different security model to other container platforms, one of the key reasons that it is well suited to HPC and cluster environments."
- "Singularity has its own container image format (SIF)."
- "The `singularity` command can be used to pull images from Sylabs Cloud Library and run a container from an image file."
---

The episodes in this lesson will introduce you to the [Singularity](https://sylabs.io/singularity/) container platform and demonstrate how to set up and use Singularity.

> ## Work in progress...
> This lesson is new material that is under ongoing development. We will introduce Singularity and demonstrate how to work with it. As the tools and best practices continue to develop, elements of this material are likely to evolve. We welcome any comments or suggestions on how the material can be improved or extended.
{: .callout}

## What is Singularity?

[Singularity](https://sylabs.io/singularity/) (or
[Apptainer](https://apptainer.org/), we'll get to this in a minute...) is a
container platform that supports packaging and deploying software and tools in
a portable and reproducible manner.

You may be familiar with Docker, another container platform that is now used widely.
If you are, you will see that in some ways, Singularity is similar to Docker. However, in
other ways, particularly in terms of the system's architecture, it is
fundamentally different. These differences mean that Singularity is
particularly well-suited to running on shared platforms such as distributed,
High Performance Computing (HPC) infrastructure, as well as on a Linux laptop
or desktop.

Singularity runs containers from container images which, as we discussed, are essentially a
virtual computer disk that contains all of the necessary software, libraries
and configuration to run one or more applications or undertake a particular
task, e.g. to support a specific research project. This saves you the time and
effort of installing and configuring software on your own system or setting up
a new computer from scratch, as you can simply run a Singularity container from
an image and have a virtual environment that is equivalent to the one used by
the person who created the image. Singularity/Apptainer is increasingly widely
used in the research community for supporting research projects due to its
support for shared computing platforms.

System administrators will not, generally, install Docker on shared computing
platforms such as lab desktops, research clusters or HPC platforms because the
design of Docker presents potential security issues for shared platforms with
multiple users. Singularity/Apptainer, on the other hand, can be run by
end-users entirely within "user space", that is, no special administrative
privileges need to be assigned to a user in order for them to run and interact
with containers on a platform where Singularity has been installed.

### A little history...

Singularity is open source software and was initially developed within the research
community. A couple of years ago, the project was "forked" something that is
not uncommon within the open source software community, with the software
effectively splitting into two projects going in different directions. The fork
is being developed by a commercial entity, [Sylabs.io](https://sylabs.io/) who
provide both the free, open source [SingularityCE (Community
Edition)](https://sylabs.io/singularity) and Pro/Enterprise editions of the
software. The original open source Singularity project has recently been
[renamed to
Apptainer](https://apptainer.org/news/community-announcement-20211130/) and has
moved into the Linux Foundation. The initial release of Apptainer was made
about a year ago, at the time of writing. While earlier versions of this course
focused on versions of Singularity released before the project fork, we now
base the course material on recent Apptainer releases. Despite this, the basic
features of Apptainer/Singularity remain the same and so this material is
equally applicable whether you're working with a recent Apptainer release or a
slightly older Singularity version. Nonetheless, it is useful to be aware of
this history and that you may see both Singularity and Apptainer being used
within the research community over the coming months and years.

Another point to note is that some systems that have a recent Apptainer release
installed may also provide a `singularity` command that is simply a link to the
`apptainer` executable on the system. This helps to ensure that existing
scripts being used on the system that were developed before the migration to
Apptainer will still function correctly.

For now, the remainder of this material refers to Singularity but where you
have a release of Apptainer installed on your local system, you can simply
replace references to `singularity` with `apptainer`, if you wish.


Open a terminal on the system that you are using for the course and check that
the `singularity` command is available in your terminal:

~~~
$ singularity --version
~~~
{: .language-bash}

~~~
singularity-ce version 3.11.0
~~~
{: .output}

Depending on the version of Singularity installed on your system, you may see a different version.

> ## Loading a module
> HPC systems often use *modules* to provide access to software on the system so you may need to use the command:
> ~~~
> $ module load singularity
> ~~~
> {: .language-bash}
> before you can use the `singularity` command on remote systems. However, this depends on how the system is configured.
> If in doubt, consult the documentation for the system you are using or contact the support team.
{: .callout}

## Images and containers: reminder

A quick reminder on terminology: we refer to both *container images* and *containers*. What is the difference between these two terms? 

*Container images* (sometimes just *images*) are bundles of files including an operating system, software and potentially data and other application-related files. They may sometimes be referred to as a *disk image* or *image* and they may be stored in different ways, perhaps as a single file, or as a group of files. Either way, we refer to this file, or collection of files, as an image.

A *container* is a virtual environment that is based on a container image. That is, the files, applications, tools, etc that are available within a running container are determined by the image that the container is started from. It may be possible to start multiple container instances from an image. You could, perhaps, consider an image to be a form of template from which running container instances can be started.

## Getting a container image and running a Singularity container

Singularity uses the [Singularity Image Format (SIF)](https://github.com/sylabs/sif) and container images are provided as single `SIF` files (usually with a `.sif` or `.img` filename extension). Singularity container images can be pulled from the [Sylabs Cloud Library](https://cloud.sylabs.io/), a registry for Singularity container images. Singularity is also capable of running containers based on container images pulled from [Docker Hub](https://hub.docker.com/) and other Docker image repositories (e.g. [Quay.io](https://quay.io)). We will look at accessing container images from Docker Hub later in the course.

> ## Sylabs Remote Builder
> Note that in addition to providing a repository that you can pull container images from, [Sylabs Cloud Library](https://cloud.sylabs.io/) can also build Singularity images for you from a *recipe* - a configuration file defining the steps to build an image. We will look at recipes and building images later in the workshop.
{: .callout}

### Pulling a container image from Sylabs Cloud Library

Let's begin by creating a `test` directory, changing into it and _pulling_ an existing container image from Sylabs Cloud Library:

~~~
$ mkdir test
$ cd test
$ singularity pull lolcow.sif library://lolcow
~~~
{: .language-bash}

~~~
INFO:    Downloading library image
 90.4 MiB / 90.4 MiB [===============================================================================================================] 100.00% 90.4 MiB/s 1s
~~~
{: .output}

What just happened? We pulled a container image from a remote repository using the `singularity pull` command and directed it to store the container image in a file using the name `lolcow.sif` in the current directory. If you run the `ls` command, you should see that the `lolcow.sif` file is now present in the current directory.

~~~
$ ls -lh
~~~
{: .language-bash}

~~~
total 60M
-rwxr-xr-x. 1 auser group 91M Jun 13  2023 lolcow.sif
~~~
{: .output}

### Running a Singularity container

We can now run a container based on the `lolcow.sif` container image:

~~~
$ singularity run lolcow.sif
~~~
{: .language-bash}

~~~
 ______________________________
< Tue Jun 20 08:44:51 UTC 2023 >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
~~~
{: .output}

The above command ran a *lolcow* container based on the container image we downloaded from the online repository and the resulting output was shown. 

What just happened? When we use the `singularity run` command, Singularity does three things:

| 1. Starts a Running Container | 2. Performs Default Action | 3. Shuts Down the Container
| --------------------|-----------------|----------------|
| Starts a running container, based on the container image. Think of this as the "alive" or "inflated" version of the container -- it's actually doing something. | If the container has a default action set, it will perform that default action. This could be as simple as printing a message (as above) or running a whole analysis pipeline! | Once the default action is complete, the container stops running (or exits). |

### Default action

How did the container determine what to do when we ran it? What did running the container actually do to result in the displayed output?

When you run a container from a Singularity container image using the `singularity run` command, the container runs the default run script that is embedded within the container image. This is a shell script that can be used to run commands, tools or applications stored within the container image on container startup. We can inspect the container image's run script using the `singularity inspect` command:

~~~
$ singularity inspect -r lolcow.sif
~~~
{: .language-bash}

~~~
#!/bin/sh 

    date  |  cowsay  |  lolcat

~~~
{: .output}

This shows us the script within the `lolcow.sif` image configured to run by default when we use the `singularity run` command.

That concludes this introductory Singularity episode. The next episode looks in more detail at running containers.
