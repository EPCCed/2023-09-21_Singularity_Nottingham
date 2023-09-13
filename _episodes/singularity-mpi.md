---
title: "Running MPI parallel jobs using Singularity containers"
teaching: 30
exercises: 40
questions:
- "How do I set up and run an MPI job from a Singularity container?"
objectives:
- "Learn how MPI applications within Singularity containers can be run on HPC platforms"
- "Understand the challenges and related performance implications when running MPI jobs via Singularity"
keypoints:
- "Singularity images containing MPI applications can be built on one platform and then run on another (e.g. an HPC cluster) if the two platforms have compatible MPI implementations."
- "When running an MPI application within a Singularity container, use the MPI executable on the host system to launch a Singularity container for each process."
- "Think about parallel application performance requirements and how where you build/run your image may affect that."
---

We assume that everyone here is familiar with what MPI is, even if they are not experienced MPI developers.

> ## What is MPI?
> MPI - [Message Passing Interface](https://en.wikipedia.org/wiki/Message_Passing_Interface) - is a widely
> used standard for parallel programming. It is used for exchanging messages/data between processes in a
> parallel application. If you've been involved in developing or working with computational science software.
{: .callout}

Usually, when working on HPC systems, you compile your application against the MPI libraries provided on the system
or you use applications that have been compiled by the HPC system support team. This approach to portability:
*source code portability* is the traditional approach to making applications portable to different HPC platforms.

However, compiling complex HPC applications that have lots of dependencies (including MPI) is not always straightforward
and can be a significant challenge as most HPC systems differ in various ways in terms of OS and base software
available. There are a number of different approaches that can be taken to make it easier to deploy applications
on HPC systems; for example, the [Spack](https://spack.readthedocs.io) software automates the dependency resolution and compilation of
applications. Containers provide another potential way to resolve these problems but care needs to be taken 
when interfacing with MPI on the host system which adds more complexity to running containers in parallel on
HPC systems.

### MPI codes with Singularity containers

Obviously, we will not have admin/root access on the HPC platform we are using so cannot (usually) build our container 
images on the HPC system itself. However, we do need to ensure our container is using the MPI library on
the HPC system itself so we can get the performance benefit of the HPC interconnect. How do we overcome these 
contradictions?

The answer is that we install a version of the MPI library in our container image that is binary compatible with 
the MPI library on the host system and install our software in the container image using the local version of
the MPI library. At runtime, we then ensure that the MPI library from the host is used within the running container
rather than the locally-installed version of MPI.

There are two widely used open source MPI library distributions on HPC systems:

* [MPICH](https://www.mpich.org/) - in addition to the open source version, MPICH is [binary compatible](https://www.mpich.org/abi/) with many
  proprietary vendor libraries, including Intel MPI and HPE Cray MPT as well as the open source MVAPICH.
* [OpenMPI](https://www.open-mpi.org/)

This typically means that if you want to distribute HPC software that uses MPI within a container image you will 
need to maintain versions that are compatible with both MPICH and OpenMPI. There are efforts underway to provide
tools that will provide a binary interface between different MPI implementations, e.g. HPE Cray's MPIxlate software
but these are not generally available yet. 

### Building a Singularity container image with MPI software

This example makes the assumption that you'll be building a container image on a local platform and then deploying
it to a HPC system with a different but compatible MPI implementation using a combination of the *Hybrid* and *Bind*
models from the Singularity documentation. We will build our application using MPI in the container image but will
bind the MPI library from the host into the container at runtime. See
[Singularity and MPI applications](https://docs.sylabs.io/guides/3.7/user-guide/mpi.html)
in the Singularity documentation for more technical details.

The example we will build will:
* Use MPICH as the container image's MPI library
* Use the Ohio State University MPI Micro-benchmarks as the example application
* Use ARCHER2 as the runtime platform - this uses Cray MPT as the host MPI library and the HPE Cray Slingshot interconnect

The Singularity container image definition file to install MPICH and the OSU micro-benchmark we will use
to build the container image is shown below. Save this in a file called `osu_benchmarks.def`

~~~
Bootstrap: docker
From: ubuntu:20.04

%environment
    export OSU_DIR=/usr/local/libexec/osu-micro-benchmarks/mpi
    export LD_LIBRARY_PATH=/usr/lib/libibverbs:$LD_LIBRARY_PATH
    export PATH=/usr/local/libexec/osu-micro-benchmarks/mpi/startup:$PATH
    export PATH=/usr/local/libexec/osu-micro-benchmarks/mpi/pt2pt:$PATH
    export PATH=/usr/local/libexec/osu-micro-benchmarks/mpi/collective:$PATH

%post
    export DEBIAN_FRONTEND=noninteractive TZ=Europe/London
    # Install required dependencies
    apt-get update && apt-get install -y --no-install-recommends \
        apt-utils \
        build-essential \
        curl \
        libcurl4-openssl-dev \
        libzmq3-dev \
        pkg-config \
        software-properties-common
    apt-get clean
    apt-get install -y dkms
    apt-get install -y autoconf automake build-essential numactl libnuma-dev autoconf automake gcc g++ git libtool

    # Download and build an ABI compatible MPICH
    cd /
    curl -k -sSLO http://www.mpich.org/static/downloads/3.4.2/mpich-3.4.2.tar.gz \
        && tar -xzf mpich-3.4.2.tar.gz -C /root \
        && cd /root/mpich-3.4.2 \
        && ./configure --prefix=/usr --with-device=ch4:ofi --disable-fortran \
        && make -j8 install \
        && cd / \
        && rm -rf /root/mpich-3.4.2 \
        && rm /mpich-3.4.2.tar.gz

    # Download and build OSU benchmarks
    curl -k -sSLO http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.4.1.tar.gz \
        && tar -xzf osu-micro-benchmarks-5.4.1.tar.gz -C /root \
        && cd /root/osu-micro-benchmarks-5.4.1 \
        && ./configure --prefix=/usr/local CC=/usr/bin/mpicc CXX=/usr/bin/mpicxx \
        && cd mpi \
        && make -j8 install \
        && cd / \
        && rm -rf /root/osu-micro-benchmarks-5.4.1 \
        && rm /osu-micro-benchmarks-5.4.1.tar.gz

~~~
{: .output}

A quick overview of what the above definition file is doing:

 - The image is being bootstrapped from the `ubuntu:20.04` Docker image.
 - In the `%environment` section: Set environment variables that will be available within all containers run from the generated image.
 - In the `%post` section:
   - Ubuntu's `apt-get` package manager is used to update the package directory and then install the compilers and other libraries required for the MPICH and OSU benchmark build.
   - The MPICH software is downloaded, extracted, configured, built and installed. Note the use of the `--with-device` option to configure MPICH to use the correct driver to support improved communication performance on a high performance cluster. After the install is complete we delete the files that are no longer needed.
   - The OSU Micro-Benchmarks software is downloaded, extracted, configured, built and installed. After the install is complete we delete the files that are no longer needed.

> ## Build with Docker
>
> > ## Dockerfile
> >
> > ```
> > FROM ubuntu:20.04
> > 
> > ENV DEBIAN_FRONTEND=noninteractive
> > 
> > # Install the necessary packages (from repo)
> > RUN apt-get update && apt-get install -y --no-install-recommends \
> >  apt-utils \
> >  build-essential \
> >  curl \
> >  libcurl4-openssl-dev \
> >  libzmq3-dev \
> >  pkg-config \
> >  software-properties-common
> > RUN apt-get clean
> > RUN apt-get install -y dkms
> > RUN apt-get install -y autoconf automake build-essential numactl libnuma-dev autoconf automake gcc g++ git libtool
> > 
> > # Download and build an ABI compatible MPICH
> > RUN curl -sSLO http://www.mpich.org/static/downloads/3.4.2/mpich-3.4.2.tar.gz \
> >    && tar -xzf mpich-3.4.2.tar.gz -C /root \
> >    && cd /root/mpich-3.4.2 \
> >    && ./configure --prefix=/usr --with-device=ch4:ofi --disable-fortran \
> >    && make -j8 install \
> >    && cd / \
> >    && rm -rf /root/mpich-3.4.2 \
> >    && rm /mpich-3.4.2.tar.gz
> > 
> > # OSU benchmarks
> > RUN curl -sSLO http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.4.1.tar.gz \
> >    && tar -xzf osu-micro-benchmarks-5.4.1.tar.gz -C /root \
> >    && cd /root/osu-micro-benchmarks-5.4.1 \
> >    && ./configure --prefix=/usr/local CC=/usr/bin/mpicc CXX=/usr/bin/mpicxx \
> >    && cd mpi \
> >    && make -j8 install \
> >    && cd / \
> >    && rm -rf /root/osu-micro-benchmarks-5.4.1 \
> >    && rm /osu-micro-benchmarks-5.4.1.tar.gz
> > 
> > # Add the OSU benchmark executables to the PATH
> > ENV PATH=/usr/local/libexec/osu-micro-benchmarks/mpi/startup:$PATH
> > ENV PATH=/usr/local/libexec/osu-micro-benchmarks/mpi/pt2pt:$PATH
> > ENV PATH=/usr/local/libexec/osu-micro-benchmarks/mpi/collective:$PATH
> > ENV OSU_DIR=/usr/local/libexec/osu-micro-benchmarks/mpi
> > 
> > # path to mlx IB libraries in Ubuntu
> > ENV LD_LIBRARY_PATH=/usr/lib/libibverbs:$LD_LIBRARY_PATH
> > ```
> {: .solution}
{: .callout}

> ## Build and test the OSU Micro-Benchmarks image
>
> Using the above definition file, build a Singularity container image named `osu_benchmarks.sif`.
> 
> Once the image has finished building, test it by running the `osu_hello` benchmark that is found in the `startup` benchmark folder.
>
> Note: the build process can take a while. If you want to test running while the build is happening, you can log into ARCHER2 
> and use a pre-built version of the container image to test. You can find this container image at:
>
> ~~~
> ${EPCC_SINGULARITY_DIR}/osu_benchmarks.sif
> ~~~
> 
> > ## Solution
> > 
> > You should be able to build an image from the definition file as follows:
> > 
> > ~~~
> > $ singularity build osu_benchmarks.sif osu_benchmarks.def
> > ~~~
> > {: .language-bash}
> >
> >
> > Assuming the image builds successfully, you can then try running the container locally and also transfer the SIF file to a cluster platform that you have access to (that has Singularity installed) and run it there.
> > 
> > Let's begin with a single-process run of `startup/osu_hello` on _your local system_ (where you built the container) to ensure that we can run the container as expected. We'll use the MPI installation _within_ the container for this test. Note that when we run a parallel job on an HPC cluster platform, we use the MPI installation on the cluster to coordinate the run so things are a little different...
> > 
> > Start a shell in the Singularity container based on your image and then run a single process job via `mpirun`:
> > 
> > ~~~
> > $ singularity shell --contain osu_benchmarks.sif
> > Singularity> mpirun -np 1 osu_hello
> > ~~~
> > {: .language-bash}
> > 
> > You should see output similar to the following:
> > 
> > ~~~
> > # OSU MPI Hello World Test v5.7.1
> > This is a test with 1 processes
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

#### Running Singularity containers with MPI on HPC system

Assuming the above tests worked, we can now try undertaking a parallel run of one of the
OSU benchmarking tools within our container image on the remote HPC platform.

This is where things get interesting and we will begin by looking at how Singularity
containers are run within an MPI environment.

If you're familiar with running MPI codes, you'll know that you use `mpirun` (as we did 
in the previous example), `mpiexec`, `srun` or a similar MPI executable to start your
application. This executable may be run directly on the local system or cluster platform
that you're using, or you may need to run it through a job script submitted to a job
scheduler. Your MPI-based application code, which will be linked against the MPI libraries,
will make MPI API calls into these MPI libraries which in turn talk to the MPI daemon
process running on the host system. This daemon process handles the communication between
MPI processes, including talking to the daemons on other nodes to exchange information
between processes running on different machines, as necessary.

When running code within a Singularity container, we don't use the MPI executables stored
within the container, i.e. we DO NOT run:

`singularity exec mpirun -np <numprocs> /path/to/my/executable`

Instead we use the MPI installation on the host system to run Singularity and start an
instance of our executable from within a container for each MPI process. Without Singularity
support in an MPI implementation, this results in starting a separate Singularity container
instance within each process. This can present some overhead if a large number of processes
are being run on a host. Where Singularity support is built into an MPI implementation
this can address this potential issue and reduce the overhead of running code from within a
container as part of an MPI job.

Ultimately, this means that our running MPI code is linking to the MPI libraries from the MPI
install within our container and these are, in turn, communicating with the MPI daemon on the
host system which is part of the host system's MPI installation. In the case of MPICH, these
two installations of MPI may be different but as long as there is
[ABI compatibility](https://wiki.mpich.org/mpich/index.php/ABI_Compatibility_Initiative) between
the version of MPI installed in your container image and the version on the host system, your
job should run successfully.

We can now try running a 2-process MPI run of a point to point benchmark `osu_latency` on
ARCHER2.


> ## Undertake a parallel run of the `osu_latency` benchmark (general example)
>
> Move the `osu_benchmarks.sif` Singularity image onto ARCHER2 where you are going to
> undertake your benchmark run using the `scp` command or similar. Alternatively, you
> can use the pre-built container image on ARCHER2 at `${EPCC_SINGULARITY_DIR}/osu_benchmarks.sif`.
> 
> Next, create a job submission script called `submit.slurm` on the /work file system on ARCHER2 to run
> containers based on the container image across two nodes on ARCHER2. A template based on the example in the
> [ARCHER2 documentation](https://docs.archer2.ac.uk/user-guide/containers/#running-parallel-mpi-jobs-using-singularity-containers)
> is:
> 
> ~~~
> #!/bin/bash
> 
> #SBATCH --job-name=singularity_parallel
> #SBATCH --time=0:10:0
> #SBATCH --nodes=2
> #SBATCH --ntasks-per-node=1
> #SBATCH --cpus-per-task=1
> 
> #SBATCH --partition=standard
> #SBATCH --qos=short
> #SBATCH --account=[budget code]
> 
> # Load the module to make the Cray MPICH ABI available
> module load cray-mpich-abi
> 
> export OMP_NUM_THREADS=1
> export SRUN_CPUS_PER_TASK=$SLURM_CPUS_PER_TASK
> 
> # Set the LD_LIBRARY_PATH environment variable within the Singularity container
> # to ensure that it used the correct MPI libraries.
export SINGULARITYENV_LD_LIBRARY_PATH="/opt/cray/pe/mpich/8.1.23/ofi/gnu/9.1/lib-abi-mpich:/opt/cray/pe/mpich/8.1.23/gtl/lib:/opt/cray/libfabric/1.12.1.2.2.0.0/lib64:/opt/cray/pe/gcc-libs:/opt/cray/pe/gcc-libs:/opt/cray/pe/lib64:/opt/cray/pe/lib64:/opt/cray/xpmem/default/lib64:/usr/lib64/libibverbs:/usr/lib64:/usr/lib64"
> 
> # This makes sure HPE Cray Slingshot interconnect libraries are available
> # from inside the container.
> export SINGULARITY_BIND="/opt/cray,/var/spool,/opt/cray/pe/mpich/8.1.23/ofi/gnu/9.1/lib-abi-mpich:/opt/cray/pe/mpich/8.1.23/gtl/lib,/etc/host.conf,/etc/libibverbs.d/mlx5.driver,/etc/libnl/classid,/etc/resolv.conf,/opt/cray/libfabric/1.12.1.2.2.0.0/lib64/libfabric.so.1,/opt/cray/pe/gcc-libs/libatomic.so.1,/opt/cray/pe/gcc-libs/libgcc_s.so.1,/opt/cray/pe/gcc-libs/libgfortran.so.5,/opt/cray/pe/gcc-libs/libquadmath.so.0,/opt/cray/pe/lib64/libpals.so.0,/opt/cray/pe/lib64/libpmi2.so.0,/opt/cray/pe/lib64/libpmi.so.0,/opt/cray/xpmem/default/lib64/libxpmem.so.0,/run/munge/munge.socket.2,/usr/lib64/libibverbs/libmlx5-rdmav34.so,/usr/lib64/libibverbs.so.1,/usr/lib64/libkeyutils.so.1,/usr/lib64/liblnetconfig.so.4,/usr/lib64/liblustreapi.so,/usr/lib64/libmunge.so.2,/usr/lib64/libnl-3.so.200,/usr/lib64/libnl-genl-3.so.200,/usr/lib64/libnl-route-3.so.200,/usr/lib64/librdmacm.so.1,/usr/lib64/libyaml-0.so.2"
> 
> # Launch the parallel job.
> srun --hint=nomultithread --distribution=block:block \
>     singularity exec ${EPCC_SINGULARITY_DIR}/osu_benchmarks.sif \
>         osu_latency
> ~~~
> {: .language-bash}
>
> Finally, submit the job to the batch system with
>
> ~~~
> sbatch submit.slurm
> ~~~
> {: .language-bash}
> 
> > ## Solution
> > 
> > As you can see in the mpirun command shown above, we have called `srun` on the host system
> > and are passing to MPI the `singularity` executable for which the parameters are the image
> > file and the name of the benchmark executable we want to run.
> > 
> > The following shows an example of the output you should expect to see. You should have latency
> > values reported for message sizes up to 4MB.
> > 
> >~~~
> > Rank 1 - About to run: /.../mpi/pt2pt/osu_latency
> > Rank 0 - About to run: /.../mpi/pt2pt/osu_latency
> > # OSU MPI Latency Test v5.6.2
> > # Size          Latency (us)
> > 0                       0.38
> > 1                       0.34
> > ...
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

This has demonstrated that we can successfully run a parallel MPI executable from within a Singularity container.

> ## Investigate performance of native benchmark compared to containerised version
> 
> To get an idea of any difference in performance between the code within your Singularity image and the same
> code built natively on the target HPC platform, try running the `osu_allreduce` benchmarks natively on ARCHER2
> on all cores on at least 16 nodes (if you want to use more than 32 nodes, you will need to use the `standard` QoS
> rather than the `short QoS`). Then try running the same benchmark that you ran via the Singularity container. Do
> you see any performance differences?
> 
> What do you see?
>
> Do you see the same when you run on small node counts - particularly a single node?
>
> Note: a native version of the OSU micro-benchmark suite is available on ARCHER2 via `module load osu-benchmarks`.
> 
> > ## Discussion
> > 
> > Here are some selected results measured on ARCHER2:
> > 
> >  1 node:
> > - 4 B
> >   + Native: 6.13 us
> >   + Container: 5.30 us (16% faster)
> > - 128 KiB
> >   + Native: 173.00 us
> >   + Container: 230.38 us (25% slower)
> > - 1 MiB
> >   + Native: 1291.18 us
> >   + Container: 2101.02 us (39% slower)
> >
> >  16 nodes:
> > - 4 B
> >   + Native: 17.66 us
> >   + Container: 18.15 us (3% slower)
> > - 128 KiB
> >   + Native: 237.29 us
> >   + Container: 303.92 us (22% slower)
> > - 1 MiB
> >   + Native: 1501.25 us
> >   + Container: 2359.11 us (36% slower)
> >
> >  32 nodes:
> > - 4 B
> >   + Native: 30.72 us
> >   + Container: 24.41 us (20% faster)
> > - 128 KiB
> >   + Native: 265.36 us
> >   + Container: 363.58 us (26% slower)
> > - 1 MiB
> >   + Native: 1520.58 us
> >   + Container: 2429.24 us (36% slower)
> >
> > For the medium and large messages, using a container produces substantially worse MPI performance for this 
> > benchmark on ARCHER2. When the messages are very small, containers match the native performance and can
> > actually be faster.
> >
> > Is this true for other MPI benchmarks that use all the cores on a node or is it specific to Allreduce?
> > 
> {: .solution}
{: .challenge}

### Summary

Singularity can be combined with MPI to create portable containers that run software in parallel across multiple
compute nodes. However, there are some limitations, specifically:

- You must use an MPI library in the container that is binary compatible with the MPI library on the host system - 
  typically, your container will be based on either MPICH or OpenMPI.
- The host setup to enable MPI typically requires binding a large number of low-level libraries into the running 
  container. You will usually require help from the HPC system support team to get the correct bind options for
  the platform you are using.
- Performance of containers+MPI can be substantially lower than the performance of native applications using MPI
  on the system. The effect is dependent on the MPI routines used in your application, message sizes and the number of MPI 
  processes used.


