---
title: "Using Singularity to run BLAST+"
teaching: 30
exercises: 30
questions:
- "How can I use Singularity to run bioinformatics workflows with BLAST+?"
objectives:
- "Show example of using Singularity with a common bioinformatics tool."
keypoints:
- "We can use containers to run software without having to install it"
- "The commands we use are very similar to those we would use natively"
- "Singularity handles a lot of complexity around data and internet access for us"
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
> during the conversion to a Singularity container image.)
{: .callout}

## Download the required data

Download the [blast_example.tar.gz]({{ page.root }}/files/blast_example.tar.gz).

Unpack the archive which contains the downloaded data required for the BLAST+ example:

~~~
tar -xvf blast_example.tar.gz
~~~
{: .language-bash}
~~~
x blast/
x blast/blastdb/
x blast/queries/
x blast/fasta/
x blast/results/
x blast/blastdb_custom/
x blast/fasta/nurse-shark-proteins.fsa
x blast/queries/P01349.fsa
~~~
{: .output}

Finally, move into the newly created directory:

~~~
cd blast
ls  
~~~
{: .language-bash}
~~~
blastdb        blastdb_custom fasta          queries        results
~~~
{: .output}

## Create the Singularity container image

NCBI provide official Docker containers with the BLAST+ software hosted on Docker Hub. We can create
a Singularity container image from the Docker container image with:

~~~
singularity pull ncbi-blast.sif docker://ncbi/blast
~~~
{: .language-bash}
~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob f3b81f6693c5 done  
Copying blob 9e3ea8720c6d done  
Copying blob f1910abb61ed done  
Copying blob 5ac33d4de47b done  
Copying blob 8402427c8382 done  
Copying blob 06add1a477bc done  
Copying blob d9781f222125 done  
Copying blob 4aae31cc8a8b done  
Copying blob 6a61413c1ffa done  
Copying blob c657bf8fc6ca done  
Copying blob 1776e565f5f8 done  
Copying blob d90474a0d8c8 done  
Copying blob 0bc89cb1b9d7 done  
Copying blob b8a272fccf13 done  
Copying blob 891eb09f891f done  
Copying blob 4c64befa8a35 done  
Copying blob 7ab0b7afbc21 done  
Copying blob b007c620c60b done  
Copying blob f877ffc04713 done  
Copying blob 6ee97c348001 done  
Copying blob 03f0ee97190b done  
Copying config 28914b3519 done  
Writing manifest to image destination
Storing signatures
2023/06/16 08:26:53  info unpack layer: sha256:9e3ea8720c6de96cc9ad544dddc695a3ab73f5581c5d954e0504cc4f80fb5e5c
2023/06/16 08:26:53  info unpack layer: sha256:06add1a477bcffec8bac0529923aa8ae25d51f0660f0c8ef658e66aa89ac82c2
2023/06/16 08:26:53  info unpack layer: sha256:f3b81f6693c592ab94c8ebff2109dc60464d7220578331c39972407ef7b9e5ec
2023/06/16 08:26:53  info unpack layer: sha256:5ac33d4de47beb37ae35e9cad976d27afa514ab8cbc66e0e60c828a98e7531f4
2023/06/16 08:27:03  info unpack layer: sha256:8402427c8382ab723ac504155561fb6d3e5ea1e7b4f3deac8449cec9e44ae65a
2023/06/16 08:27:03  info unpack layer: sha256:f1910abb61edef8947e9b5556ec756fd989fa13f329ac503417728bf3b0bae5e
2023/06/16 08:27:03  info unpack layer: sha256:d9781f222125b5ad192d0df0b59570f75b797b2ab1dc0d82064c1b6cead04840
2023/06/16 08:27:03  info unpack layer: sha256:4aae31cc8a8b726dce085e4e2dc4671a9be28162b8d4e1b1c00b8754f14e6fe6
2023/06/16 08:27:03  info unpack layer: sha256:6a61413c1ffa309d92931265a5b0ecc9448568f13ccf3920e16aaacc8fdfc671
2023/06/16 08:27:03  info unpack layer: sha256:c657bf8fc6cae341e3835cb101dc4c6839ba4aad69578ff8538b3c1eba7abb21
2023/06/16 08:27:04  info unpack layer: sha256:1776e565f5f85562b8601edfd29c35f3fba76eb53177c8e89105f709387e3627
2023/06/16 08:27:04  info unpack layer: sha256:d90474a0d8c8e6165d909cc0ebbf97dbe70fd759a93eff11a5a3f91fa09a470e
2023/06/16 08:27:04  warn rootless{root/edirect/aux/lib/perl5/Mozilla/CA/cacert.pem} ignoring (usually) harmless EPERM on setxattr "user.rootlesscontainers"
2023/06/16 08:27:04  warn rootless{root/edirect/aux/lib/perl5/Mozilla/CA.pm} ignoring (usually) harmless EPERM on setxattr "user.rootlesscontainers"
2023/06/16 08:27:04  warn rootless{root/edirect/aux/lib/perl5/Mozilla/mk-ca-bundle.pl} ignoring (usually) harmless EPERM on setxattr "user.rootlesscontainers"
2023/06/16 08:27:04  info unpack layer: sha256:0bc89cb1b9d7ca198a7a1b95258006560feffaff858509be8eb7388b315b9cf5
2023/06/16 08:27:04  info unpack layer: sha256:b8a272fccf13b721fa68826f17f0c2bb395de377e0d22c98d38748eb5957a4c6
2023/06/16 08:27:04  info unpack layer: sha256:891eb09f891ff2c26f24a5466112e134f6fb30bd3d0e78c14c0d676b0e68d60a
2023/06/16 08:27:04  info unpack layer: sha256:4c64befa8a35c9f8518324524dfc27966753462a4c07b2234811865387058bf4
2023/06/16 08:27:04  info unpack layer: sha256:7ab0b7afbc21b75697a7b8ed907ee9b81e5b17a04895dc6ff7d25ea2ba1eeba4
2023/06/16 08:27:04  info unpack layer: sha256:b007c620c60b91ce6a9e76584ecc4bc062c822822c204d8c2b1c8668193d44d1
2023/06/16 08:27:04  info unpack layer: sha256:f877ffc04713a03dffd995f540ee13b65f426b350cdc8c5f1e20c290de129571
2023/06/16 08:27:04  info unpack layer: sha256:6ee97c348001fca7c98e56f02b787ce5e91d8cc7af7c7f96810a9ecf4a833504
2023/06/16 08:27:04  info unpack layer: sha256:03f0ee97190baebded2f82136bad72239254175c567b19def105b755247b0193
INFO:    Creating SIF file...
~~~
{: .output}

Now we have a container with the software in, we can use it.

## Build and verify the BLAST database

Our example dataset has already downloaded the query and database sequences. We first
use these downloaded data to create a custom BLAST database by using a container to run
the command `makeblastdb` with the correct options.

~~~
singularity exec ncbi-blast.sif \
    makeblastdb -in fasta/nurse-shark-proteins.fsa -dbtype prot \
    -parse_seqids -out nurse-shark-proteins -title "Nurse shark proteins" \
    -taxid 7801 -blastdb_version 5
~~~
{: .language-bash}
~~~

Building a new DB, current time: 06/16/2023 14:35:07
New DB name:   /home/auser/test/blast/blast/nurse-shark-proteins
New DB title:  Nurse shark proteins
Sequence type: Protein
Keep MBits: T
Maximum file size: 3000000000B
Adding sequences from FASTA; added 7 sequences in 0.0199499 seconds.

~~~
{: .output}

To verify the newly created BLAST database above, you can run the
`blastdbcmd -entry all -db nurse-shark-proteins -outfmt "%a %l %T"` command to display
the accessions, sequence length, and common name of the sequences in the database.

~~~
singularity exec ncbi-blast.sif \
    blastdbcmd -entry all -db nurse-shark-proteins -outfmt "%a %l %T"
~~~
{: .language-bash}

~~~
Q90523.1 106 7801
P80049.1 132 7801
P83981.1 53 7801
P83977.1 95 7801
P83984.1 190 7801
P83985.1 195 7801
P27950.1 151 7801
~~~
{: .output}

Now we have our database we can run queries against it.

## Run a query against the BLAST database

Lets execute a query on our database using the `blastp` command:

~~~
singularity exec ncbi-blast.sif \
    blastp -query queries/P01349.fsa -db nurse-shark-proteins \
    -out results/blastp.out
~~~
{: .language-bash}

At this point, you should see the results of the query in the output file `results/blastp.out`.
To view the content of this output file, use the command `less results/blastp.out`.

~~~
less results/blastp.out
~~~
{: .language-bash}

~~~
...output trimmed...

Query= sp|P01349.2|RELX_CARTA RecName: Full=Relaxin; Contains: RecName:
Full=Relaxin B chain; Contains: RecName: Full=Relaxin A chain

Length=44
                                                                      Score     E
Sequences producing significant alignments:                          (Bits)  Value

P80049.1 RecName: Full=Fatty acid-binding protein, liver; AltName...  14.2    0.96


>P80049.1 RecName: Full=Fatty acid-binding protein, liver; AltName: Full=Liver-type
fatty acid-binding protein; Short=L-FABP
Length=132

...output trimmed...
~~~
{: .output}

With your query, BLAST identified the protein sequence P80049.1 as a match with a score
of 14.2 and an E-value of 0.96.

## Accessing online BLAST databases

As well as building your own local database to query, you can also access databases that are
available online. For example, to see which databases are available online in the Google Compute
Platform (GCP):

~~~
singularity exec ncbi-blast.sif update_blastdb.pl --showall pretty --source gcp
~~~
{: .language-bash}

~~~
Connected to GCP
BLASTDB                                                      DESCRIPTION                                                                                                              SIZE (GB)      LAST_UPDATED
nr                                                           All non-redundant GenBank CDS translations+PDB+SwissProt+PIR+PRF excluding environmental samples from WGS projects        369.4824      2023-06-10
swissprot                                                    Non-redundant UniProtKB/SwissProt sequences                                                                                 0.3576      2023-06-10
refseq_protein                                               NCBI Protein Reference Sequences                                                                                          146.5088      2023-06-12
landmark                                                     Landmark database for SmartBLAST                                                                                            0.3817      2023-04-25
pdbaa                                                        PDB protein database                                                                                                        0.1967      2023-06-10
nt                                                           Nucleotide collection (nt)                                                                                                319.5044      2023-06-11
pdbnt                                                        PDB nucleotide database                                                                                                     0.0145      2023-06-09
patnt                                                        Nucleotide sequences derived from the Patent division of GenBank                                                           15.7342      2023-06-09
refseq_rna                                                   NCBI Transcript Reference Sequences                                                                                        47.8721      2023-06-12

...output trimmed...
~~~
{: .output}

Similarly, for databases hosted at NCBI:

~~~
singularity exec ncbi-blast.sif update_blastdb.pl --showall pretty --source ncbi
~~~
{: .language-bash}

~~~
Connected to NCBI
BLASTDB                                                      DESCRIPTION                                                                                                              SIZE (GB)      LAST_UPDATED
env_nr                                                       Proteins from WGS metagenomic projects (env_nr).                                                                            3.9459      2023-06-11
SSU_eukaryote_rRNA                                           Small subunit ribosomal nucleic acid for Eukaryotes                                                                         0.0063      2022-12-05
LSU_prokaryote_rRNA                                          Large subunit ribosomal nucleic acid for Prokaryotes                                                                        0.0041      2022-12-05
16S_ribosomal_RNA                                            16S ribosomal RNA (Bacteria and Archaea type strains)                                                                       0.0178      2023-06-16
env_nt                                                       environmental samples                                                                                                      48.8599      2023-06-08
LSU_eukaryote_rRNA                                           Large subunit ribosomal nucleic acid for Eukaryotes                                                                         0.0053      2022-12-05
ITS_RefSeq_Fungi                                             Internal transcribed spacer region (ITS) from Fungi type and reference material                                             0.0067      2022-10-28
Betacoronavirus                                              Betacoronavirus                                                                                                            55.3705      2023-06-16

...output trimmed...
~~~
{: .output}

## Notes

You have now completed a simple example of using a complex piece of bioinformatics software
through Singularity containers. You may have noticed that some things just worked without
you needing to set them up even though you were running using containers:

1. We did not need to explicitly bind any files/directories in to the container. This worked
   because Singularity automatically binds the current directory into the running container, so
   any data in the current directory (or its subdirectories) will generally be available in
   running Singularity containers. (If you have used Docker containers, you will notice that
   this is different from the defalt behaviour there.)
2. Access to the internet is automatically available within the running container in the same
   way as it is on the host system without us needed to specify any additional options.
4. Files and data we create within the container have the right ownership and permissions for
   us to access outside the container.

In addtion, we were able to use the tools in the container image provided by NCBI without having
to do any work to install the software irrespecetive of the computing platform that we are using.
(In fact, the example this is based on runs the pipeline using Docker on a cloud computing platform
rather than on your local systeam.)


