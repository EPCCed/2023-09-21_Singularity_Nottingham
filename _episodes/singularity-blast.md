---
title: "Using Singularity to run BLAST+"
teaching: 15
exercises: 30
questions:
- "How can I use Singularity to run BLAST+?"
objectives:
- "Show example of using Singularity with a common bioinformatics tool."
keypoints:
- "TBC"
---

We have now learned enough to be able to use Sigularity to deploy software without us
needed to install the software itself on the host system.

In this section we will demonstrate the use of a Singularity container image that 
provides the BLAST+ software.

> ## Source material
> This example is based on the example from the official [NCBI BLAST+ Docker
> container documentation](https://github.com/ncbi/blast_plus_docs#step-2-import-sequences-and-create-a-blast-database)
> Note: the `efetch` parts of the step-by-step guide do not currently work using
> Singularity version of the image so we provide a dataset with the data already
> downloaded.
>
> (This is because the NCBI BLAST+ Docker container image has the `efetch` tool
> installed in the `/root` directory and this special location gets overwritten
> during the conversion to a Singularity container image. We demonstrate how to
> build a container image with the `efetch` tool available later in the course.)
{: .callout}

## Download the required data

Download the [blast_example.tar.gz]({{ page.root }}/files/blast_example.tar.gz), e.g.:

~~~
wget {{ page.root }}/files/blast_example.tar.gz
~~~
{: .language-bash}
~~~
~~~
{: .output}

Unpack the archive:

~~~
tar -xvf blast_example.tar.gz
~~~
{: .language-bash}
~~~
~~~
{: .output}

Finally, move into the newly created directory:

~~~
cd blast
ls
~~~
{: .language-bash}
~~~
~~~
{: .output}


## Create the Singularity container image

NCBI provide official Docker containers with the BLAST+ software. We can create
a Singularity container image from the Docker container image with:

~~~
singularity build ncbi-blast.sif docker://ncbi/blast
~~~
{: .language-bash}
~~~
~~~
{: .output}

## Build and verify the BLAST database

~~~
singularity exec ncbi-blast.sif \
    makeblastdb -in fasta/nurse-shark-proteins.fsa -dbtype prot \
    -parse_seqids -out nurse-shark-proteins -title "Nurse shark proteins" \
    -taxid 7801 -blastdb_version 5
~~~
{: .language-bash}

~~~
singularity exec ncbi-blast.sif \
    blastdbcmd -entry all -db nurse-shark-proteins -outfmt "%a %l %T"
~~~
{: .language-bash}

## Run a query against the BLAST database

~~~
singularity exec ncbi-blast.sif \
    blastp -query queries/P01349.fsa -db nurse-shark-proteins \
    -out results/blastp.out
~~~
{: .language-bash}

