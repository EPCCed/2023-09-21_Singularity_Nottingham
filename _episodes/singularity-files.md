---
title: "Files in Singularity containers"
teaching: 10
exercises: 10
questions:
- "How do I make data available in a Singularity container?"
- "What data is made available by default in a Singularity container?"
objectives:
- "Understand that some data from the host system is usually made available by default within a container"
- "Learn more about how Singularity handles users and binds directories from the host filesystem."
keypoints:
- "Your current directory and home directory are usually available by default in a container."
- "You have the same username and permissions in a container as on the host system."
- "You can specify additional host system directories to be available in the container."
---

The key concept to remember when running a Singularity container, you only have the same permissions to access files as the
user on the host system that you start the container as. (If you are familiar with Docker, you may note that this is different
behaviour than you would see with that tool.)

In this episode we will look at working with files in the context of Singularity containers and how this links with Singularity's
approach to users and permissions within containers.

## Users within a Singularity container

The first thing to note is that when you ran `whoami` within the container shell you started at the end of the previous episode,
you should have seen the same username that you have on the host system when you ran the container. 

For example, if my username were `jc1000`, I would expect to see the following:

~~~
singularity shell lolcow.sif
Singularity> whoami
~~~
{: .language-bash}

~~~
jc1000
~~~
{: .output}

But wait! I downloaded the standard, public version of the `lolcow` container image from the Cloud Library. I haven't customised
it in any way. How is it configured with my own user details?!

If you have any familiarity with Linux system administration, you may be aware that in Linux, users and their Unix groups are
configured in the `/etc/passwd` and `/etc/group` files respectively. In order for the running container to know of my
user, the relevant user information needs to be available within these files within the container.

Assuming this feature is enabled within the installation of Singularity on your system, when the container is started, Singularity
appends the relevant user and group lines from the host system to the `/etc/passwd` and `/etc/group` files within the
container[\[1\]](https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf).

This means that the host system can effectively ensure that you cannot access/modify/delete any data you should not be able to on
the host system from within the container and you cannot run anything that you would not have permission to run on the host system
since you are restricted to the same user permissions within the container as you are on the host system.

## Files and directories within a Singularity container

Singularity also *binds* some *directories* from the host system where you are running the `singularity` command into the container
that you are starting. Note that this bind process is not copying files into the running container, it is making an existing directory
on the host system visible and accessible within the container environment. If you write files to this directory within the running
container, when the container shuts down, those changes will persist in the relevant location on the host system.

There is a default configuration of which files and directories are bound into the container but ultimate control of how things are
set up on the system where you are running Singularity is determined by the system administrator. As a result, this section provides
an overview but you may find that things are a little different on the system that you're running on.

One directory that is likely to be accessible within a container that you start is your *home directory*.  You may also find that
the directory from which you issued the `singularity` command (the *current working directory*) is also bound.

The binding of file content and directories from a host system into a Singularity container is illustrated in the example below
showing a subset of the directories on the host Linux system and in a running Singularity container:

~~~
Host system:                                                      Singularity container:
-------------                                                     ----------------------
/                                                                 /
├── bin                                                           ├── bin
├── etc                                                           ├── etc
│   ├── ...                                                       │   ├── ...
│   ├── group  ─> user's group added to group file in container ─>│   ├── group
│   └── passwd ──> user info added to passwd file in container ──>│   └── passwd
├── home                                                          ├── usr
│   └── jc1000 ───> user home directory made available ──> ─┐     ├── sbin
├── usr                 in container via bind mount         │     ├── home
├── sbin                                                    └────────>└── jc1000
└── ...                                                           └── ...

~~~
{: .output}

> ## Questions and exercises: Files in Singularity containers
>
> **Q1:** What do you notice about the ownership of files in a container started from the `lolcow.sif` image? (e.g. take a look at the ownership
> of files in the root directory (`/`) and your home directory (`~/`)).
> 
> **Exercise 1:** In this container, try creating a file in the root directory `/` (e.g. using `touch /myfile.dat`). What do you notice? Try
> removing the `/singularity` file. What happens in these two cases?
> 
> **Exercise 2:** In your home directory within the container shell, try and create a simple text file (e.g. `echo "Some text" > ~/test-file.txt`).
> Is it possible to do this? If so, why? If not, why not?! If you can successfully create a file, what happens to it when you exit the shell and
> the container shuts down?
>
> > ## Answers
> >
> > **A1:** Use the `ls -l /` command to see a detailed file listing including file ownership and permission details. You should see that most of
> > the files in the `/` directory are owned by `root`, as you would probably expect on any Linux system. If you look at the files in your home
> > directory, they should be owned by you.
> >
> > **A Ex1:** We've already seen from the previous answer that the files in `/` are owned by `root` so we would nott expect to be able to create
> > files there if we're not the root user. However, if you tried to remove `/singularity` you would have seen an error similar to the following:
> > `cannot remove '/singularity': Read-only file system`. This tells us something else about the filesystem. It's not just that we do not have
> > permission to delete the file, the filesystem itself is read-only so even the `root` user would not be able to edit/delete this file. We will
> > look at this in more detail shortly.
> > 
> > **A Ex2:** Within your home directory, you _should_ be able to successfully create a file. Since you're seeing your home directory on the host
> > system which has been bound into the container, when you exit and the container shuts down, the file that you created within the container
> > should still be present when you look at your home directory on the host system.
> {: .solution}
{: .challenge}

## Binding additional host system directories to the container

You will sometimes need to bind additional host system directories into a container you are using over and above those bound by default. For example:

- There may be a shared dataset in a location that you need access to in the container
- You may require executables and software libraries from the host system in the container

The `-B` option to the `singularity` command is used to specify additional binds. For example, to bind the `/mnt/c/Users/Andrew` directory (my
Windows home directory in WSL2) into a container you could use (note this directory is unlikely to exist on the host system you are using so
you will need to test this using a different directory):

```
singularity shell -B /mnt/c/Users/Andrew lolcow.sif
Singularity> ls /mnt/c/Users/Andrew
```
{: .language-bash}

Note that, by default, a bind is mounted at the same path in the container as on the host system. You can also specify where a host directory is
mounted in the container by separating the host path from the container path by a colon (`:`) in the option:

```
singularity shell -B /mnt/c/Users/Andrew:/Windows lolcow.sif
Singularity> ls /Windows
```
{: .language-bash}

You can also specify multiple binds to `-B` by separating them by commas (`,`).

You can also copy data into a container image at build time if there is some static data required in the image. We cover this later in the section
on building Singularity container images.

## References

\[1\] Gregory M. Kurzer, Containers for Science, Reproducibility and Mobility: Singularity P2. Intel HPC Developer Conference, 2017. Available
at: https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf
