# Using GROOT

GROOT uses a subcommand syntax; call the main program with `groot` and follow it by the subcommand to indicate what  action to take.

This page will cover a worked example, details on the available GROOT commands and some tips for using the program.

***

## An example

This example will take us from raw reads to resistome profile. We will use some metagenome data from a recent paper by [Winglee et al.](https://doi.org/10.1186/s40168-017-0338-7), where they identify differences in the microbiota in rural versus recently urban subjects from the Hunan province of China (see the tutorial for a more complete example using this dataset).

Get some sequence data:
```
fastq-dump SRR4454621 SRR4454610
```

Get a pre-clustered ARG database:
```
groot get -d arg-annot
```

Create variation graphs from the ARG-database and index:
```
groot index -i arg-annot.90 -o groot-index -p 8 -l 100
```

Align the reads and report ARGs
```
groot align -i groot-index -f SRR4454621.fastq -p 8 | groot report > resistome-profile-rural.txt

groot align -i groot-index -f SRR4454610.fastq -p 8 | groot report > resistome-profile-urban.txt
```

***

## GROOT subcommands

Here are some details on the subcommands....

### get

The ``get`` subcommand is used to download a pre-clustered ARG database that is ready to be indexed. Here is an example:

```
groot get -d resfinder -i 90 -o .
```

The above command will get the [ResFinder](https://cge.cbs.dtu.dk/services/ResFinder/) database, which has been clustered at 90% identity, and save it to the current directory. The database will be named ``resfinder.90`` and contain several ``*.msa`` files.

Flags explained:

* ``-d``: which database to get
* ``-i``: the identity threshold at which the database was clustered (note: only 90 currently available)
* ``-o``: directory to save the database to

The following databases are available:

* arg-annot (default)
* resfinder
* card

These databases were clustered by sequence identity and stored as **Multiple Sequence Alignments** (MSAs). They were clustered using the following commands:

```
vsearch --cluster_size ARGs.fna --id 0.90 --msaout MSA.tmp

awk '!a[$0]++ {of="./cluster-" ++fc ".msa"; print $0 >> of ; close(of)}' RS= ORS="\n\n" MSA.tmp && rm MSA.tmp
```

The ``get`` subcommand performs the following steps:

* downloads a tarball containing the MSAs generated from the specified database (clustered using the specified identity)
* checks checks the file integrity
* unpacks the files into the specified directory


### index

The ``index`` subcommand is used to create variation graphs from a pre-clustered ARG database and then index them. Here is an example:

```
groot index -i arg-annot.90 -o groot-index -p 8 -l 100
```

The above command will create a variation graph for each cluster in the ``arg-annot.90`` database and initialise an LSH forest index. It will then move through each graph traversal using a 100 node window, creating a MinHash signature for each window. Finally, each signature is added to the LSH Forest index. The index will be named ``groot-index`` and contain several files.

Flags explained:

* ``-i``: which database to use
* ``-o``: where to save the index
* ``-p``: how many processors to use
* ``-l``: length of window to use (should be similar to the length of query reads)

The same index can be used on multiple samples, however, it should be re-indexed if you are using different read lengths or wish to change the seeding parameters.

Some more flags that can be used:

* ``-j``: Jaccard similarity threshold for seeding a read (used to calibrate LSH Forest index)
* ``-k``: size of k-mer to use for MinHashing
* ``-s``: length of MinHash signature
* ``-w``: offset to use when moving window along graph traversals (i.e. distance between MinHash signatures)


### align

The ``align`` subcommand is used to align reads against the indexed variation graphs. Here is an example:

```
groot align -i groot-index -f file.fastq -p 8 > ARG-reads.bam
```

The above command will seed the fastq reads against the indexed variation graph. It will then perform a hierarchical local alignment of each seed against the variation graph traversals. The output alignment is essentially the ARG classified reads (which may be useful) and can then be used to report full-length ARGs (using the report subcommand).

Flags explained:

* ``-i``: which index to use
* ``-f``: what FASTQ reads to align
* ``-p``: how many processors to use

Data can streamed in and out of the align subcommand. For example:

```
gunzip -c file.gz | ./groot align -i groot-index -p 8 | ./groot report
```
> this feature is being worked on and will bring more utility in future releases.

Multiple FASTQ files can be specified as input, however all are treated as the same sample and paired-end info isn't used. To specify multiple files, make sure they are comma separated (``-i fileA.fq,fileB.fq``).

Some more flags that can be used:

* ``--trim``: enable quality based trimming of read prior to alignment
* ``-q``: minimum base quality during trimming
* ``-l``: minimum read length post trimming
* ``-c``: maximum number of bases to clip from 3' end during local alignment


### report

Will update soon...

***

## Tips

Will update soon...