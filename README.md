# Workflow single-cell

wf-single-cell is a research pipeline designed to identify the cell barcode
and UMI sequences present in nanopore sequencing reads generated from single-cell gene expression libraries. 

It was initially created as a Nextflow port of [Sockeye](https://github.com/nanoporetech/sockeye).




## Introduction

The following single-cell kits from 10x Genomics are currently supported:
- Chromium Single Cell `3ʹ gene expression <https://teichlab.github.io/scg_lib_structs/methods_html/10xChromium3.html>`_, versions 2 and 3
- Chromium Single Cell `5ʹ gene expression <https://teichlab.github.io/scg_lib_structs/methods_html/10xChromium5.html>`_, version 1
- Chromium Single Cell `Multiome (ATAC + GEX) <https://teichlab.github.io/scg_lib_structs/methods_html/10xChromium_multiome.html>`_, version 1

Oxford Nanopore has developed a protocol for sequencing single-cell libraries from 10X, which can be found on the Nanopore Community `website <https://community.nanoporetech.com/docs/prepare/library_prep_protocols/single-cell-transcriptomics-10x/v/sst_v9148_v111_revb_12jan2022>`_.

The inputs to Sockeye are raw nanopore reads (FASTQ) generated from the sequencing
instrument and reference files that can be downloaded from `10X
<https://support.10xgenomics.com/single-cell-gene-expression/software/downloads/latest>`_.
The pipeline outputs a gene x cell expression matrix, as well as a BAM file of
aligned reads tagged with cell barcode and UMI information.

Prerequisites
-------------

``conda`` must be installed in order to create the base environment where the
Sockeye snakemake pipeline will run. Installation instructions can be found in
the conda `documentation <https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html>`_.

Package dependencies
--------------------

The Sockeye pipeline makes use of the following dependencies. No manual
installation is required, as these are all installed automatically into a series
of ``conda`` environments that are created throughout the course of a pipeline
run:

- bedtools [1_]
- bioframe [2_]
- biopython [3_]
- editdistance [4_]
- matplotlib [5_]
- minimap2 [6_]
- numpy [7_]
- pandas [8_]
- parasail-python [9_]
- pysam [10_]
- samtools [11_]
- scikit-learn [12_]
- seqkit [13_]
- tqdm [14_]
- umap-learn [15_]
- vsearch [16_]

Additionally, while no explicit dependency exists for the
`UMI-tools <https://github.com/CGATOxford/UMI-tools>`_ package  [17_], the script
``bin/cluster_umis.py`` makes significant use of several functions from
the package. More detailed acknowledgements can be found in the source code.
## Quickstart

The workflow uses [nextflow](https://www.nextflow.io/) to manage compute and 
software resources, as such nextflow will need to be installed before attempting
to run the workflow.

The workflow can currently be run using either
[Docker](https://www.docker.com/products/docker-desktop) or
[conda](https://docs.conda.io/en/latest/miniconda.html) to provide isolation of
the required software. Both methods are automated out-of-the-box provided
either docker of conda is installed.

It is not required to clone or download the git repository in order to run the workflow.
For more information on running EPI2ME Labs workflows [visit out website](https://labs.epi2me.io/wfindex).

**Workflow options**

To obtain the workflow, having installed `nextflow`, users can run:

```
nextflow run epi2me-labs/wf-template --help
```

to see the options for the workflow.

**Workflow outputs**

The pipeline output will be written to a directory defined by ``OUTPUT_BASE`` in the ``config/config.yml`` file. For instance, using the example ``config/config.yml`` and ``config/sample_sheet.csv`` files shown above, the pipeline output would be written to three separate directories, one for each ``run_id``:

::

   /PATH/TO/OUTPUT/BASE/DIRECTORY/run1
   /PATH/TO/OUTPUT/BASE/DIRECTORY/run2
   /PATH/TO/OUTPUT/BASE/DIRECTORY/run3
   /PATH/TO/OUTPUT/BASE/DIRECTORY/run4

Each run_id-specific output folder will contain the following subdirectories:

::

   /PATH/TO/OUTPUT/BASE/DIRECTORY/run1
   |
   |-- adapters   # contains output from the characterization of read structure based on adapters
   |-- align      # output from the alignment to the reference
   |-- demux      # demultiplexing results, primarily in the tagged.sorted.bam file
   |-- matrix     # gene expression matrix and UMAP outputs
   \-- saturation # plots describing the library sequencing saturation

The most useful outputs of the pipeline are likely:

* ``adapters/configs.stats.json``: provides a summary of sequencing statistics and observed read configurations, such as

  - ``n_reads``: number of total reads in the input fastq(s)
  - ``rl_mean``: mean read length
  - ``n_fl``: total number of reads with the read1-->TSO or TSO'-->read1' adapter configuration (i.e. full-length reads)
  - ``n_plus``: number of reads with the read1-->TSO configuration
  - ``n_minus``: number of reads with the TSO'-->read1' configuration

* ``demux/tagged.sorted.bam``: BAM file of alignments to the reference where each alignment contains the following sequence tags

  - CB: corrected cell barcode sequence
  - CR: uncorrected cell barcode sequence
  - CY: Phred quality scores of the uncorrected cell barcode sequence
  - UB: corrected UMI sequence
  - UR: uncorrected UMI sequence
  - UY: Phred quality scores of the uncorrected UMI sequence

* ``matrix/gene_expression.processed.tsv``: TSV containing the gene (rows) x cell (columns) expression matrix, processed and normalized according to the parameters defined in the ``config/config.yml`` file:

  - ``MATRIX_MIN_GENES``: cells with fewer than this number of expressed genes will be removed
  - ``MATRIX_MIN_CELLS``: genes present in fewer than this number of cells will be removed
  - ``MATRIX_MAX_MITO``: cells with more than this percentage of counts belonging to mitochondrial genes will be removed
  - ``MATRIX_NORM_COUNT``: normalize all cells to this number of total counts per cell
## Useful links

* [nextflow](https://www.nextflow.io/)
* [docker](https://www.docker.com/products/docker-desktop)
* [conda](https://docs.conda.io/en/latest/miniconda.html)
