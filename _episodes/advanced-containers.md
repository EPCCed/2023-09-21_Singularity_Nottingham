---
title: "Creating More Complex Container Images"
teaching: 30
exercises: 30
questions:
- "How can I make more complex container images? "
objectives:
- "Explain how you can include files within Singularity container images when you build them."
- "Explain how you can access files on the Singularity host from your Singularity containers."
keypoints:
- Singularity allows containers to read and write files from the Singularity host.
- You can include files from your Singularity host into your Singularity container images by using the `%files` section in your Singularity recipe file.
---

In order to create and use your own container images, you may need more information than
our previous example. You may want to include files from outside the container by copying the files 
into the container image. You may also want to learn a little bit more
about how to install software within a running container or a container image. 
This episode will look at these advanced aspects of building 
a container image. 

## Refresher: using scripts and files from outside the container at runtime

We are assuming that you are still in the same directory where you created your 
`alpine-python.sif` container image in the last lesson.

Use your text editor to create a Python script called `sum.py` with the following
contents

~~~
#!/usr/bin/env python3

# Comment added in container

import sys
try:
   total = sum(int(arg) for arg in sys.argv[1:])
   print('sum =', total)
except ValueError:
   print('Please supply integer arguments')
~~~
{: .language-python}


> ## Running containers
>
> What command would we use to run Python 3 from the `alpine-python` container?
>
> > ## Solution
> >
> > We would use:
> >
> > ~~~
> > singularity exec alpine-python.sif python3
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

If we try running the container and Python script, what happens?

~~~
singularity exec alpine-python.sif python3 sum.py
~~~
{: .language-bash}
~~~
sum = 0
~~~
{: .output}


> ## Exercise: Explore the script
>
> What happens if you use the `singularity exec` command above
> and put numbers after the script name?
>
> > ## Solution
> >
> > This script comes from [the Python Wiki](https://wiki.python.org/moin/SimplePrograms)
> > and is set to add all numbers
> > that are passed to it as arguments.
> {: .solution}
{: .challenge}

> ## Exercise: Interactive use
>
> We can also use the script interactively within the running container. What commands would
> you use to run the `sum.py` script interactively in a container based on the `alpine-python.sif`
> container image.
>
> > ## Solution
> >
> > The Singularity command to run the container interactively is:
> > ~~~
> > singularity shell alpine-python.sif
> > Singularity> python3 sum.py 10 12 10
> > ~~~
> > {: .language-bash}
> >
> > ~~~
> > sum = 32
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

This all works without any further options because Singularity binds the current directory into the
running container by default. If the `sum.py` was in a directory other than the current directory 
(or its subdirectories) then you would need to bind it into the running container explicitly using
the `-B` option as described earlier in the course.

In other situations, you may want to save or archive an authoritative version of your data by adding
it to the container image permanently. That's what we will cover next.

## Including your scripts and data within a container image

Our next project will be to add our own files to a container image -- something you
might want to do if you're sharing a finished analysis or just want to have
an archived copy of your entire analysis including the data. Let's assume that we've finished with our `sum.py`
script and want to add it to the container image itself.

In your shell, you should still be in the same folder as before (where you have your `sum.py` script).
~~~
ls
~~~
{: .language-bash}
~~~
alpine-python.def  alpine-python.sif  sum.py
~~~
{: .language-bash}

Let's add a new line to the Singularity recipe we've been using so far to create a copy of `sum.py`.
We can do so by using the `%files` section.

~~~
%files
    sum.py /home
~~~

This line will cause Singularity to copy the file from your computer into the container's
filesystem. 

Create a new Singularity recipe file called `alpine-sum.def` with the following contents:

~~~
Bootstrap: docker
From: alpine:latest

%files
    sum.py /home

%post
    apk add --update python3 py3-pip python3-dev

%runscript
    python3 --version
~~~

This is the same definition file as we had before but with the `%files` section added.

Let's build the container image like before, but give it a different name:

~~~
sudo singularity build alpine-sum.sif alpine-sum.def
~~~
{: .language-bash}

> ## Exercise: Did it work?
>
> Can you remember how to run a container interactively? Try that with this one.
> Once inside, try running the Python script.
>
> > ## Solution
> >
> > You can start the container interactively like so:
> > ~~~
> > singularity shell alpine-sum.sif
> > ~~~
> > {: .language-bash}
> >
> > You should be able to run the python command inside the container like this:
> > ~~~
> > Singularity> python3 /home/sum.py
> > ~~~
> > {: .language-bash}
> >
> {: .solution}
{: .challenge}

This `%files` section can be used to place your own scripts or own data into a container image
that you want to publish or use as a record. Note that it's not necessarily a good idea
to put your scripts inside the container image if you're constantly changing or editing them.
Then, referencing the scripts from outside the container is a good idea, as we
did in the previous section. You also want to think carefully about size -- if you
run `ls -lh *.sif` you'll see the size of each container image in the current directory. The
bigger your container image becomes, the harder it will be to easily download.

> ## Security Warning
> 
> Login credentials including passwords, tokens, secure access tokens or other secrets
> must never be stored in a container image. If secrets are stored, they are at high risk to
> be found and exploited when made public.
{: .callout}

> ## Copying alternatives
>
> Another approach for getting your own files into a container image is by using the `%post`
> section and adding commands that download the files from the internet. For example, if your code
> is in a GitHub repository, you could include this statement in your recipe file
> to download the latest version every time you build the container image:
>
> ~~~
> %post
>     ...other installation commands...
>     git clone https://github.com/alice/mycode
> ~~~
>
> Similarly, the `wget` command can be used to download any file publicly available
> on the internet:
>
> ~~~
> %post
>     ...other installation commands...
>     wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.10.0/ncbi-blast-2.10.0+-x64-linux.tar.gz
> ~~~
>
> Note that the above examples depend on commands (`git` and `wget` respectively) that 
> must be available within your container: Linux distributions such as Alpine may require you to 
> install such commands before using them.
{: .callout}

> ## Make the `sum.py` script run automatically
> 
> Can you modify the `alpine-sum.def` recipe file so that the `sum.py` is run
> automatically when using the `singularity run` command?
>
> > ## Solution
> > ~~~
> > Bootstrap: docker
> > From: alpine:latest
> > 
> > %post
> >     apk add --update python3 py3-pip python3-dev
> > 
> > %files
> >     sum.py /home
> > 
> > %runscript
> >     python3 /home/sum.py
> > ~~~
> >
> {: .solution}
{: .challenge

Build and test it:
~~~
sudo singularity build alpine-sum.sif alpine-sum.def
singularity run alpine-sum.sif
~~~
{: .language-bash}

You'll notice that you can run the container without arguments just fine,
resulting in `sum = 0`, but this is boring. Supplying arguments however
doesn't work:
~~~
singularity run alpine-sum.sif 10 11 12
~~~
{: .language-bash}

still results in

~~~
sum = 0
~~~
{: .output}

This is because the arguments `10 11 12` are not interpreted by
the *runscript* in the container.

To achieve the goal of having a command that *always* runs when a
container is run from the container image *and* can be passed the arguments given on the
command line, we need to tell the *runscript* to use the arguments:

~~~
Bootstrap: docker
From: alpine:latest

%post
    apk add --update python3 py3-pip python3-dev

%files
    sum.py /home

%runscript
    python3 /home/sum.py "$@"
~~~

Build and test it:
~~~
sudo singularity build alpine-sum.sif alpine-sum.def
singularity run alpine-sum.sif 10 11 12
~~~
{: .language-bash}

~~~
sum = 33
~~~
{: .output}

## More advanced definition files

Here we've looked at a very simple example of how to create an image. At this stage, you might want to have a go at creating your own definition file for some code of your own or an application that you work with regularly. There are several definition file sections that were _not_ used in the above example, these are:

 - `%setup`
 - `%environment`
 - `%startscript`
 - `%test`
 - `%labels`
 - `%help`

The [`Sections` part of the definition file documentation](https://docs.sylabs.io/guides/3.11/user-guide/definition_files.html) details all the sections and provides an example definition file that makes use of all the sections.

## Additional Singularity features

Singularity has a wide range of features. You can find full details in the [Singularity User Guide](https://docs.sylabs.io/guides/3.11/user-guide/index.html) and we highlight a couple of key features here that may be of use/interest:

**Remote Builder Capabilities:** If you have access to a platform with Singularity installed but you don't have root access to create containers, you may be able to use the [Remote Builder](https://docs.sylabs.io/guides/3.11/user-guide/cloud_library.html#remote-builder) functionality to offload the process of building an image to remote cloud resources. You'll need to register for a _cloud token_ via the link on the [Remote Builder](https://cloud.sylabs.io/builder) page.

**Signing containers:** If you do want to share container image (`.sif`) files directly with colleagues or collaborators, how can the people you send an image to be sure that they have received the file without it being tampered with or suffering from corruption during transfer/storage? And how can you be sure that the same goes for any container image file you receive from others? Singularity supports signing containers. This allows a digital signature to be linked to an image file. This signature can be used to verify that an image file has been signed by the holder of a specific key and that the file is unchanged from when it was signed. You can find full details of how to use this functionality in the Singularity documentation on [Signing and Verifying Containers](https://docs.sylabs.io/guides/3.11/user-guide/signNverify.html).


> ## Best practices for writing container image definition files
> Take a look at Nüst et al.'s "[_Ten simple rules for writing Dockerfiles for reproducible data science_](https://doi.org/10.1371/journal.pcbi.1008316)" \[1\] 
> for some great examples of best practices to use when writing Dockerfiles. 
> The [GitHub repository](https://github.com/nuest/ten-simple-rules-dockerfiles) associated with the paper also has a set of [example `Singularityfile`s](https://github.com/nuest/ten-simple-rules-dockerfiles/tree/master/examples) 
> demonstrating how the rules highlighted by the paper can be applied.
>
> <small>[1] Nüst D, Sochat V, Marwick B, Eglen SJ, Head T, et al. (2020) Ten simple rules for writing Dockerfiles for reproducible data science. PLOS Computational Biology 16(11): e1008316. https://doi.org/10.1371/journal.pcbi.1008316</small>
{: .callout}



{% include links.md %}
