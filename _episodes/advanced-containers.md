---
title: "Creating More Complex Container Images"
teaching: 15
exercises: 20
questions:
- "How can I make more complex container images? "
objectives:
- "Explain how you can include files within Docker container images when you build them."
- "Explain how you can access files on the Docker host from your Docker containers."
keypoints:
- Docker allows containers to read and write files from the Docker host.
- You can include files from your Docker host into your Docker container images by using the `COPY` instruction in your `Dockerfile`.
---

## Building container images with your files included

In order to create and use your own container images, you may need more information than
our previous example. You may want to use files from outside the container, 
that are not included within the container image by copying the files 
into the container image at build time.

<!--
You may also want to learn a little bit 
about how to install software within a running container or a container image. 
This episode will look at these advanced aspects of running a container or building 
a container image.
-->

Note that the examples will get gradually
more and more complex -- most day-to-day use of containers and container images can be accomplished
using the first 1--2 sections on this page.

### Create a Python script

Before we go ahead and build our next container image, we're going
to create a simple Python script on our host system and create a 
Dockerfile to have this script copied into our container image when
it is created.

In your shell, create a new directory to hold the *build context* for our new container image and
move into the directory:

~~~
$ mkdir alpine-sum
$ cd alpine-sum
~~~
{: .language-bash}

Use your text editor to create a Python script called `sum.py` with the
following contents:

~~~
#!/usr/bin/env python3

import sys
try:
   total = sum(int(arg) for arg in sys.argv[1:])
   print('sum =', total)
except ValueError:
   print('Please supply integer arguments')
~~~
{: .language-python}

Let's assume that we've finished with our `sum.py`
script and want to add it to the container image itself.

### Create the Dockerfile

Now we have our Python script, we are going to create our Dockerfile. This is going 
to be similar to the Dockerfile we used in the previous section with the addition of
one extra line. Here is the full Dockerfile:

~~~
FROM alpine
RUN apk add --update python3 py3-pip python3-dev
RUN pip install cython
COPY sum.py /home
CMD ["python3", "--version"]
~~~

The additional line we have added is:

~~~
COPY sum.py /home
~~~

This line will cause Docker to copy the file from your computer into the container's
filesystem. Let's build the container image like before, but give it a different name
and then push it to Docker Hub (remember to subsitute `alice` for your Docker Hub username):

~~~
$ docker image build --platform linux/amd64 -t alice/alpine-sum .

...output from docker build...

$ docker push alice/alpine-sum
~~~
{: .language-bash}

> ## The Importance of Command Order in a Dockerfile
> 
> When you run `docker build` it executes the build in the order specified
> in the `Dockerfile`.
> This order is important for rebuilding and you typically will want to put your `RUN` 
> commands before your `COPY` commands.
> 
> Docker builds the layers of commands in order.
> This becomes important when you need to rebuild container images.
> If you change layers later in the `Dockerfile` and rebuild the container image, Docker doesn't need to 
> rebuild the earlier layers but will instead used a stored (called "cached") version of
> those layers.
> 
> For example, in an instance where you wanted to copy `multiply.py` into the container 
> image instead of `sum.py`.
> If the `COPY` line came before the `RUN` line, it would need to rebuild the whole image. 
> If the `COPY` line came second then it would use the cached `RUN` layer from the previous
> build and then only rebuild the `COPY` layer.
> 
{: .callout}

> ## Exercise: Did it work?
>
> Can you remember how to run a container interactively on the remote HPC system? Try that with this one.
> Once inside, try running the Python script you added to the container image.
>
> > ## Solution
> >
> > You can start the container interactively on the remote HPC system like so (remember to use your
> > Docker Hub username):
> > ~~~
> > remote$ singularity pull alpine-sum.sif docker://alice/alpine-sum
> > remote$ singularity shell alpine-sum.sif
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

This `COPY` keyword can be used to place your own scripts or own data into a container image
that you want to publish or use as a record. Note that it's not necessarily a good idea
to put your scripts inside the container image if you're constantly changing or editing them.
Then, referencing the scripts from outside the container is a good idea, by bind mounting 
host data into the running container as we saw earlier in the workshop. You also want to
think carefully about size -- if you run `docker image ls` you'll see the size of each container
image all the way on the right of the screen. The bigger your container image becomes, the harder
it will be to easily download.

> ## Security Warning
> 
> Login credentials including passwords, tokens, secure access tokens or other secrets
> must never be stored in a container. If secrets are stored, they are at high risk to
> be found and exploited when made public.
{: .callout}

> ## Copying alternatives
>
> Another trick for getting your own files into a container image is by using the `RUN`
> keyword and downloading the files from the internet. For example, if your code
> is in a GitHub repository, you could include this statement in your Dockerfile
> to download the latest version every time you build the container image:
>
> ~~~
> RUN git clone https://github.com/alice/mycode
> ~~~
>
> Similarly, the `wget` command can be used to download any file publicly available
> on the internet:
>
> ~~~
> RUN wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.10.0/ncbi-blast-2.10.0+-x64-linux.tar.gz
> ~~~
>
> Note that the above `RUN` examples depend on commands (`git` and `wget` respectively) that 
> must be available within your container: Linux distributions such as Alpine may require you to 
> install such commands before using them within `RUN` statements.
{: .callout}

## More fancy `Dockerfile` options (optional, for presentation or as exercises)

We can expand on the example above to make our container image even more "automatic".
Here are some ideas:

### Make the `sum.py` script run automatically

~~~
FROM alpine
RUN apk add --update python3 py3-pip python3-dev
COPY sum.py /home

# Run the sum.py script as the default command
CMD ["python3", "/home/sum.py"]
~~~

Build and push this:

~~~
$ docker image build --platform linux/amd64 -t alice/alpine-sum:v1 .
$ docker push alice/alpine-sum:v1
~~~
{: .language-bash}

You'll notice that you can run the container without arguments just fine,
resulting in `sum = 0`, but this is boring. Supplying arguments however
doesn't work:

~~~
remote$ singularity pull alpine-sum_v1.sif docker://alice/alpine-sum:v1
remote$ singularity run alpine-sum_v1.sif 10 11 12
~~~
{: .language-bash}

results in:

~~~
sum = 0
~~~
{: .output}

This is because the arguments `10 11 12` are ignored by the `CMD` syntax.

To achieve the goal of having a command that *always* runs when a
container is run from the container image *and* can be passed the arguments given on the
command line, use the keyword `ENTRYPOINT` in the `Dockerfile`.

~~~
FROM alpine

COPY sum.py /home
RUN apk add --update python3 py3-pip python3-dev

# Run the sum.py script as the default command and
# allow people to enter arguments for it
ENTRYPOINT ["python3", "/home/sum.py"]

# Give default arguments, in case none are supplied on
# the command-line
CMD ["10", "11"]
~~~

Build and push this:

~~~
$ docker image build --platform linux/amd64 -t alice/alpine-sum:v2 .
$ docker push alice/alpine-sum:v2
~~~
{: .language-bash}

~~~
remote$ singularity pull alpine-sum_v2.sif docker://alice/alpine-sum:v2
remote$ singularity run alpine-sum_v2.sif 10 11 12
~~~
{: .language-bash}
~~~
sum = 33
~~~
{: .output}

### Add the `sum.py` script to the `PATH` so you can run it directly:

~~~
FROM alpine

RUN apk add --update python3 py3-pip python3-dev

COPY sum.py /home
# set script permissions
RUN chmod +x /home/sum.py
# add /home folder to the PATH
ENV PATH /home:$PATH
~~~

Build and push this:

~~~
$ docker image build --platform linux/amd64 -t alice/alpine-sum:v3 .
$ docker push alice/alpine-sum:v3
~~~
{: .language-bash}

Build and test it:
~~~
remote$ singularity pull alpine-sum_v3.sif docker://alice/alpine-sum:v3
remote$ singularity exec alpine-sum_v3.sif sum.py 10 11 12
~~~
{: .language-bash}
~~~
sum = 33
~~~
{: .output}

> ## Best practices for writing Dockerfiles
> Take a look at Nüst et al.'s "[_Ten simple rules for writing Dockerfiles for reproducible data science_](https://doi.org/10.1371/journal.pcbi.1008316)" \[1\] 
> for some great examples of best practices to use when writing Dockerfiles. 
> The [GitHub repository](https://github.com/nuest/ten-simple-rules-dockerfiles) associated with the paper also has a set of [example `Dockerfile`s](https://github.com/nuest/ten-simple-rules-dockerfiles/tree/master/examples) 
> demonstrating how the rules highlighted by the paper can be applied.
>
> <small>[1] Nüst D, Sochat V, Marwick B, Eglen SJ, Head T, et al. (2020) Ten simple rules for writing Dockerfiles for reproducible data science. PLOS Computational Biology 16(11): e1008316. https://doi.org/10.1371/journal.pcbi.1008316</small>
{: .callout}

{% include links.md %}
