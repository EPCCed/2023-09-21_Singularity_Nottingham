---
title: "Creating Your Own Container Images"
teaching: 20
exercises: 15
questions:
- "How can I make my own Docker container images?"
- "How do I document the 'recipe' for a Docker container image?"
objectives:
- "Explain the purpose of a `Dockerfile` and show some simple examples."
- "Demonstrate how to build a Docker container image from a `Dockerfile`."
- "Compare the steps of creating a container image interactively versus a `Dockerfile`."
- "Create an installation strategy for a container image."
- "Demonstrate how to upload ('push') your container images to the Docker Hub."
- "Describe the significance of the Docker Hub naming scheme."
keypoints:
- "`Dockerfile`s specify what is within Docker container images."
- "The `docker image build` command is used to build a container image from a `Dockerfile`."
- "You can share your Docker container images through the Docker Hub so that others can create Docker containers from your container images."
---

There are lots of reasons why you might want to create your **own** container images.
- You cannot find a container image with all the tools you need in an online repository (such as Docker Hub).
- You want to have a container image to "archive" all the specific software versions you ran for a project.
- You want to share your workflow with someone else.

## Why Docker, not Singularity?

In this section we will use Docker, rather than Singularity, to create our own container images. However,
we will still use Singularity on the remote HPC platform to run containers based on the container images
we have created. Why do we use a separate tool to create our container images rather than just using 
Singularity to build container images too?

One of the most important points to note is that, generally you must have administrator privileges on the
system where you want to build container images. On the remote HPC system where we have been *running* 
Singularity containers we do not have administrator privileges so we cannot use these systems to build 
our own container images. In this course, we will use our laptop/workstation to build container images though
it is also possible ot build them on commercial cloud resources and through other systems such as
Github actions - using these more advanced options is beyond the scope of this course.

This still does not explain why we are using Docker rather than Singularity to build our container
images. Using Docker on our laptop/workstation has a number of advantages:

- Docker/Docker Desktop is much easier to install than SingularityCE/Apptainer (particularly on macOS/Windows systems)
- Docker can build cross-platform - you can build container images for x86 systems on Arm-based systems (such as Mac M1/M2 systems)
- Docker is generally more efficient in dealing with uploading/downloading container image data that makes it better for moving your container images to remote HPC facilities

As we have already seen, Docker images can be used by Singularity (it converts them to Singularity image format
automatically) so using Docker to build container images is typically an easier option.

## More on Docker?

You can find a full introduction to Docker in the
[Introduction to Docker lesson](https://carpentries-incubator.github.io/docker-introduction/)
in the Carpentries Incubator. Here, we will use a subset of that lesson to build our own
container images.

## Interactive installation

We could, potentially, install software in a running container and then save the updated
state of the running container as a new container image but we are not going to
use this approach to create our own container images. The main reason why we do not
do this is that it it reduces the reproducibility of our images. It is also not 
typical practice for creating container images.

Instead of installing software interactively, we are going to create our container image
from a reproducible recipe, known as a `Dockerfile`.

If you haven't already, exit out of the interactively running container.

## Check that Docker is working

As mentioned above, we will be building container images on your *local* system
(usually your laptop) rather than on the remote HPC system. You can keep your terminal
to the remote system open as we will be using later to run containers based on the
images we are creating. Open a new terminal on your 

Start the Docker application that you installed in working through the setup instructions for the workshop. Note that this might not be necessary if your laptop is running Linux or if the installation added the Docker application to your startup process. 

> ## You may need to login to Docker Hub
> The Docker application will usually provide a way for you to log in to the Docker Hub using the application's menu (macOS) or systray
> icon (Windows) and it is usually convenient to do this when the application starts. This will require you to use your Docker Hub
> username and your password. We will not actually require access to the Docker Hub until later in the course but if you can login now,
> you should do so.
{: .callout}

> ## Determining your Docker Hub username
> If you no longer recall your Docker Hub username, e.g., because you have been logging into the Docker Hub using your email address,
> you can find out what it is through the steps:
> - Open <https://hub.docker.com/> in a web browser window
> - Sign-in using your email and password (don't tell us what it is)
> - In the top-right of the screen you will see your username
{: .callout}

Once your Docker application is running, open a shell (terminal) window, and run the following command to check that Docker is installed and the command line tools are working correctly. Below is the output for a Mac version, but the specific version is unlikely to matter much: it does not have to precisely match the one listed below.

~~~
$ docker --version
~~~
{: .language-bash}
~~~
Docker version 20.10.5, build 55c4c88
~~~
{: .output}

The above command has not actually relied on the part of Docker that runs containers, just that Docker
is installed and you can access it correctly from the command line.

A command that checks that Docker is working correctly is the `docker container ls` command (we cover this command in more detail later in the course).

Without explaining the details, output on a newly installed system would likely be:
~~~
$ docker container ls
~~~
{: .language-bash}
~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
~~~
{: .output}
(The command `docker system info` could also be used to verify that Docker is correctly installed and operational but it produces a larger amount of output.)

However, if you instead get a message similar to the following
~~~
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
~~~
{: .output}

then you need to check that you have started the Docker Desktop, Docker Engine, or however else you worked through the setup instructions.

## Put installation instructions in a `Dockerfile`

As mentioned above, building images using Docker should be done by adding the build
and installaiton instructions into a `Dockerfile`.

A `Dockerfile` is a plain text file with keywords and commands that can be used by
Docker to create a new container image. In this first example, we are going to create
a container image based on the lightweight Alpine Linux distribution with Python 3
added.

From your shell, create a new folder to contain the Dockerfile. Typically, all the
data required to build a container image is placed in the same folder - you will 
have one folder per container image you build.

~~~
$ mkdir alpine-python
$ cd alpine-python
~~~

Once you are in the directory, use your favourite text editor to create the Dockerfile
to build the new container image. You should create a file called `Dockerfile` with the 
following contents.

{: .language-bash}
~~~
FROM alpine
RUN apk add --update python3 py3-pip python3-dev
RUN pip install cython
CMD ["python3", "--version"]
~~~
{: .output}

Let's break this file down:

- The first line, `FROM`, indicates which container image we're starting with.  It is the "base" container image we are going to start from. In this case, the official Docker Alpine Linux image.
- The next two lines `RUN`, will indicate installation commands we want to run. These commands first install Python 3 and pip and then use the newly installed `pip` command to install Cython.
- The last line, `CMD`, indicates the default command we want a
container based on this container image to run, if no other command is provided. It is recommended
to provide `CMD` in *exec-form* (see the
[`CMD` section](https://docs.docker.com/engine/reference/builder/#cmd)
of the Dockerfile documentation for more details). It is written as a
list which contains the executable to run as its first element,
optionally followed by any arguments as subsequent elements. The list
is enclosed in square brackets (`[]`) and its elements are
double-quoted (`"`) strings which are separated by commas. In this example,
we run the `python3` command with the `--version` option.

> ## *shell-form* and *exec-form* for CMD
> Another way to specify the parameter for the
> [`CMD` instruction](https://docs.docker.com/engine/reference/builder/#cmd)
> is the *shell-form*. Here you type the command as you would call it
> from the command line. Docker then silently runs this command in the
> image's standard shell.  `CMD cat /etc/passwd` is equivalent to `CMD
> ["/bin/sh", "-c", "cat /etc/passwd"]`. We recommend to prefer the
> more explicit *exec-form* because we will be able to create more
> flexible container image command options and make sure complex commands
> are unambiguous in this format.
{: .callout}

## Create a new Docker container image

So far, we only have a text file named `Dockerfile` -- we do not yet have a container image.
We want Docker to take this `Dockerfile`, run the installation commands contained within it,
and then save the resulting container as a new container image. To do this we will use the
`docker image build` command.

We have to provide `docker image build` with three pieces of information:
- The platform to build the container image for. This will usually be `linux/amd64` to make
  a container image compatible with x86 HPC systems.
- The name of the new container image. To make uploading to Docker Hub easier, you should
  name your new image with your Docker Hub username and a name for the container image,
  like this: `USERNAME/CONTAINER_IMAGE_NAME`.
- The location of the `Dockerfile`.

All together, the build command that you should run on your computer, will have a similar
structure to this:

~~~
$ docker image build --platform linux/amd64 -t USERNAME/CONTAINER_IMAGE_NAME .
~~~
{: .language-bash}

The `-t` option names the container image; the final dot indicates that the `Dockerfile` is in
our current directory.

For example, if my user name was `alice` and I wanted to call my
container image `alpine-python`, I would use this command:
~~~
$ docker image build --platform linux/amd64 -t alice/alpine-python .
~~~
{: .language-bash}

> ## Build Context
>
> Notice that the final input to `docker image build` isn't the Dockerfile -- it's
> a directory! In the command above, we've used the current working directory (`.`) of
> the shell as the final input to the `docker image build` command. This option provides
> what is called the *build context* to Docker -- if there are files being copied
> into the built container image [more details in the next episode](/advanced-containers)
> they're assumed to be in this location. Docker expects to see a Dockerfile in the
> build context also (unless you tell it to look elsewhere).
>
> Even if it will not need all of the files in the build context directory, Docker does
> "load" them before starting to build, which means that it's a good idea to have
> only what you need for the container image in a build context directory, as we've done
> in this example.
{: .callout}

## Listing the images you have locally on your laptop

To show the images you have available in Docker on your laptop (hopefully including 
the one you have just built!) you use the `docker image ls` command:

~~~
$ docker image ls
~~~
{: .language-bash}
~~~
REPOSITORY                   TAG               IMAGE ID       CREATED          SIZE
alice/alpine-python          latest            aa0400bf60c3   14 seconds ago   166MB
~~~
{: .output}

## Share your new container image on Docker Hub

Container images that you release publicly can be stored on the Docker Hub for free.  If you
name your container image as described above, with your Docker Hub username, all you need to do
is run the opposite of `docker image pull` -- `docker image push`.

~~~
$ docker image push alice/alpine-python
~~~
{: .language-bash}

Make sure to substitute the full name of your container image!

In a web browser, open <https://hub.docker.com>, and on your user page you should now see your
container image listed, for anyone to use or build on.

> ## Logging In
>
> Technically, you have to be logged into Docker on your computer for this to work.
> Usually it happens by default, but if `docker image push` doesn't work for you,
> run `docker login` first, enter your Docker Hub username and password, and then
> try `docker image push` again.
{: .callout}

> ## Exercise: Run your new container image using Singularity
>
> Now your container image is on Docker Hub, you should be able to use what you learned earlier
> to use Singularity download the new container image to the HPC platform, convert it to 
> a Singularity SIF file and run a container based on the image.
>
> 1. What command (on the remote HPC system) will download your new container image from Docker
>    Hub and save to a Singularity image file (SIF)?
> 2. What command would you then use to run a container based on your new container image on the
>    remote HPC system?
> 3. Can you make your running container do something different, like print "hello world"?
>
> > ## Solution
> >
> > 1. To download your new image from Docker Hub and save it as a SIF file called 
> >    `alpine-python.sif` you would use something like 
> >    `singularity pull alpine-python.sif docker://alice/alpine-python` (.remember to
> >    use your Docker Hub username instead of `alice`!)
> > 2. To run the default command using the new container image, you would use
> >    `singularity run alpine-python.sif`. This should print the version of Python in
> >    the container.
> > 3. To run a different command, we would use something like: 
> >    `singluarity exec alpine-python.sif echo "hello world"`
> >
> > {: .language-bash}
> {: .solution}
{: .challenge}

While it may not look like you have achieved much, you have already effected the combination of a lightweight Linux operating system with your specification to run a given command that can operate reliably on macOS, Microsoft Windows, Linux and on a remote HPC platform!

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
- Create a `Dockerfile` and then try to build the container image from that.

## What's in a name? (again)

You don't *have* to name your containers images using the `USERNAME/CONTAINER_IMAGE_NAME:TAG`
naming scheme. On your own computer, you can call container images whatever you want, and refer to
them by the names you choose. It's only when you want to share a container image that it
needs the correct naming format.

You can rename container images using the `docker image tag` command. For example, imagine someone
named Alice has been working on a workflow container image and called it `workflow-test`
on her own computer. She now wants to share it in her `alice` Docker Hub account
with the name `workflow-complete` and a tag of `v1`. Her `docker image tag` command
would look like this:
~~~
$ docker image tag workflow-test alice/workflow-complete:v1
~~~
{: .language-bash}

She could then push the re-named container image to Docker Hub,
using `docker image push alice/workflow-complete:v1`

{% include links.md %}
