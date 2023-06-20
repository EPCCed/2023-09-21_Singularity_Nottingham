---
title: "Using Singularity containers to run commands"
teaching: 10
exercises: 5
questions:
- "How do I run different commands within a container?"
- "How do I access an interactive shell within a container?"
objectives:
- "Learn how to run different commands when starting a container."
- "Learn how to open an interactive shell within a container environment."
keypoints:
- "The `singularity exec` is an alternative to `singularity run` that allows you to start a container running a specific command."
- "The `singularity shell` command can be used to start a container and run an interactive shell within it."
---

## Running specific commands within a container

We saw earlier that we can use the `singularity inspect` command to see the run script that a container is configured to run by default. What if we want to run a different command within a container?

If we know the path of an executable that we want to run within a container, we can use the `singularity exec` command. For example, using the `lolcow.sif` container that we've already pulled from Singularity Hub, we can run the following within the `test` directory where the `lolcow.sif` file is located:

~~~
singularity exec lolcow.sif /bin/echo Hello World!
~~~
{: .language-bash}

~~~
Hello World!
~~~
{: .output}

Here we see that a container has been started from the `lolcow.sif` image and the `/bin/echo` command has been run within the container, passing the input `Hello World!`. The command has echoed the provided input to the console and the container has terminated.

Note that the use of `singularity exec` has overriden any run script set within the image metadata and the command that we specified as an argument to `singularity exec` has been run instead.

> ## Basic exercise: Running a different command within the "hello-world" container
>
> Can you run a container based on the `lolcow.sif` image that *prints the current date and time*?
> 
> > ## Solution
> >
> > ~~~
> > singularity exec lolcow.sif /bin/date
> > ~~~
> > {: .language-bash}
> > 
> > ~~~
> > Fri Jun 26 15:17:44 BST 2020
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}


### Difference between `singularity run` and `singularity exec`

Above, we used the `singularity exec` command. In earlier episodes of this
course we used `singularity run`. To clarify, the difference between these
two commands is:

 - `singularity run`: This will run the default command set for containers
   based on the specified image. This default command is set within
   the image metadata when the image is built (we'll see more about this
   in later episodes). You do not specify a command to run when using
   `singularity run`, you simply specify the image file name. As we saw 
   earlier, you can use the `singularity inspect` command to see what command
   is run by default when starting a new container based on an image.

 - `singularity exec`: This will start a container based on the specified
   image and run the command provided on the command line following
   `singularity exec <image file name>`. This will override any default
   command specified within the image metadata that would otherwise be
   run if you used `singularity run`.

## Opening an interactive shell within a container

If you want to open an interactive shell within a container, Singularity provides the `singularity shell` command. Again, using the `lolcow.sif` image, and within our `test` directory, we can run a shell within a container from the hello-world image:

~~~
$ singularity shell lolcow.sif
~~~
{: .language-bash}

~~~
Singularity> whoami
[<your username>]
Singularity> ls
lolcow.sif
Singularity> 
~~~
{: .output}

As shown above, we have opened a shell in a new container started from the `lolcow.sif` image. Note that the shell prompt has changed to show we are now within the Singularity container.

Use the `exit` command to exit from the container shell.


