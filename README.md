VaporUp is an actively developed and maintained version of Vapor, a tool for classification of Influenza samples from raw short read sequence data. From a fasta file of (preferably thousands of) full-length viral reference sequences for a given segment, and a set of reads, VaporUp attempts to identify the reference that is closest to the sample strain.

**Installation**

- From the development repo:

  `pip install git+https://github.com/wm75/vaporup`

- From PyPI:

  coming soon

- From bioconda:

  coming soon


**Testing**

A test dataset is provided in the tests folder of the developemnt repo.
After cloning, tests can be run with:

    vapor.py -fq tests/test_reads.fq -fa tests/HA_sample.fa

which should yield:

    0.9782480893592005  186719.0    1701    109.77013521457965  1000    >cds:ADO12563 A/Chile/3935/2009 2009/07/07 HA H1N1 Human

Where the tab-delimited fields correspond to: approximate fraction of query bases found in reads; total score; query length; mean score; number of reads surviving culling; query description


**Usage**

    usage: vapor.py [-h] [-q] [-k K] [-s S] [-fa FA]
                              [-fq FQ [FQ ...]]

    optional arguments:
        -h, --help          Show this help message and exit
        -q, --quiet         Suppresses output to stderr
        --return_seqs       Returns a fasta of sequences, instead of hits
        --return_best_n     Returns the highest scoring n queries

        -o                  Combined output to files with prefix O, none by default
        -k K                Kmer length [21]
        -t T                Pre-Filtering score threshold [0.2]
        -s S                Number of reads to sub-sample
        -c, --min_kmer_cov  Minimum kmer coverage for culling [5]
        -m, --min_kmer_prop
                            Minimum proportion of kmers required for query [0.1]
        -fa FA              Fasta file
        -fq FQ [FQ ...]     Fastq file/files, can be gzipped
        -f, --top_seed_frac
                            Fraction of best seeds to extend [0.2]
        --low_mem           Does not store reference kmer arrays, produces same result, marginally slower but less memory [False]
        -v, --version       Show version

Example:

    vapor.py -fa HA_sequences.fa -fq reads_1.fq.gz reads_2.fq.gz


**Use for other viruses**

VaporUp can in principle be used for other viruses (although performance has not yet been comprehensively benchmarked as with influenza, future versions will benchmark generalizability). For example, for ~79,448 HIV env sequences downloaded from https://www.hiv.lanl.gov/components/sequence/HIV/search/search.html, and public read sets with the ENA run accession SRR8389950, derived from HIV BF520.W14M (see https://www.ebi.ac.uk/ena/data/view/SRR8389950). A BF520 reference can be retrieved using the following parameters:

    vapor.py -c 100 --subsample 1000000 --low_mem -m 0.2 -f 0.1 -fq SRR8389950_1.fastq.gz SRR8389950_2.fastq.gz -fa env_db.fasta

Which returns:

    0.8030480656506448  235867226.0 2559    92171.63970300899   11899117    >A1.KE.1994.BF520.W14M.C2.KX168094

Mapping to this reference should result in a mismatch rate of < 2e-03%.

In this case we have customized the parameters:

- Since the reference space is larger than for example, influenza A HA sequences, we use --low mem to reduce memory (this does not affect the result, but may increase run-time slightly)

- We also assume, with the larger database, that there are sufficient close sequences to our sample, and use -m 0.2, requiring at least 20% exact matches to improve run-time (this does not affect the result as long as there are enough close references to the sample). If the reference space was very sparse, or our sample very novel, we may need to use -m 0.0

- Again, due to the larger database, we decrease -f to 0.1 in order to extend fewer sequences with high-scoring exact matches.

- Because the sample has over 12,000,000 reads, we improve run-time by subsampling 1,000,000 reads (--subsample 1000000). If depth is extremely skewed, sub-sampling may result in zero coverage in some sequence regions. In general, sub-sampling may decrease performance, and should be avoided where possible.

- Since depth is expected to be very high, we can also cull any k-mers with coverage less than 100 (assuming that they are, for example, errors or minor quasispecies variants). In some cases, this can affect perfomance (either increase or decrease), but will reduce memory and improve run-time as well. As with sub-sampling, this may also cull legitimate k-mers, and reduce performance, especially where depth is low.

- If we had reason to believe our virus includes many k-mers that are also present in non-viral background sequences, we could increase -t, or -k, although the latter may have more implications for performance in general (sensitivity/specificity).


**CPU and memory requirements**

For .fastq files with up to 10,000,000 reads, and influenza A segment .fasta files with approximately 47,000 references, VaporUp (single thread) requires < 4 Gb of memory and can generally be run within 4 minutes on an Intel(R) Core(TM) i7-6600U CPU @ 2.60GHz. For future datasets, memory requirements may be larger, and parameters may need to be optimized (see the above discussion of HIV). As such, we recommend 8 Gb of RAM for general usage, and any modern CPU.


**Acknowledgments**

Original author of VAPOR: Joel Southgate

