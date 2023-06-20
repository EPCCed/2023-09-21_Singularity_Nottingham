---
title: "The Singularity cache"
teaching: 10
exercises: 0
questions:
- "Why does Singularity use a local cache?"
- "Where does Singularity store images?"
objectives:
- "Learn about Singularity's image cache."
- "Learn how to manage Singularity images stored locally."
keypoints:
- "Singularity caches downloaded images so that an unchanged image isn't downloaded again when it is requested using the `singularity pull` command."
- "You can free up space in the cache by removing all locally cached images or by specifying individual images to remove."
---

## Singularity's image cache

Singularity uses a local cache to save downloaded container image files in addition to storing them as the file you specify. As we saw in the previous episode, images are simply `.sif` files stored on your local disk. 

If you delete a local `.sif` container image that you have pulled from a remote container image repository and then pull it again, if the container image is unchanged from the version you previously pulled, you will be given a copy of the container image file from your local cache rather than the container image being downloaded again from the remote source. This removes unnecessary network transfers and is particularly useful for large container images which may take some time to transfer over the network. To demonstrate this, remove the `lolcow.sif` file stored in your `test` directory and then issue the `pull` command again:

~~~
$ rm lolcow.sif
$ singularity pull lolcow.sif library://lolcow
~~~
{: .language-bash}

~~~
INFO:    Using cached image
~~~
{: .output}

As we can see in the above output, the container image has been returned from the cache and we do not see the output that we saw previously showing the container image being downloaded from the Cloud Library.

How do we know what is stored in the local cache? We can find out using the `singularity cache` command:

~~~
$ singularity cache list
~~~
{: .language-bash}

~~~
here are 1 container file(s) using 90.43 MiB and 0 oci blob file(s) using 0.00 KiB of space
Total space used: 90.43 MiB
~~~
{: .output}

This tells us how many container image files are stored in the cache and how much disk space the cache is using but it doesn't tell us _what_ is actually being stored. To find out more information we can add the `-v` verbose flag to the `list` command:

~~~
$ singularity cache list -v
~~~
{: .language-bash}

~~~
NAME                     DATE CREATED           SIZE             TYPE
sha256.cef378b9a9274c2   2020-06-20 13:20:44    90.43 MiB        library

There are 1 container file(s) using 90.43 MiB and 0 oci blob file(s) using 0.00 KiB of space
Total space used: 90.43 MiB
~~~
{: .output}

This provides us with some more useful information about the actual container images stored in the cache. In the `TYPE` column we can see that our container image type is `library` because it's a `SIF` container image that has been pulled from the Cloud Library. 

> ## Cleaning the Singularity image cache
> We can remove container images from the cache using the `singularity cache clean` command. Running the command without any options will display a warning and ask you to confirm that you want to remove everything from your cache.
>
> You can also remove specific container images or all container images of a particular type. Look at the output of `singularity cache clean --help` for more information.
{: .callout}

> ## Cache location
> By default, Singularity uses `$HOME/.singularity/cache` as the location for the cache. You can change the location of the cache by setting the `SINGULARITY_CACHEDIR` environment variable to the cache location you want to use.
{: .callout}
