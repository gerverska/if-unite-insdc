IF + UNITE + INSDC: improved fungal taxonomy for ITS reference sequences
================
Kyle A. Gervers
2024-03-25

## Overview

While the [UNITE general FASTA
releases](https://unite.ut.ee/repository.php) represent high-quality,
dynamically clustered references for fungal taxon assignment, most
releases consistently ignore taxa that I care about. Also, I’ve noticed
that the taxonomic ranks applied are not always the most up-to-date when
compared with what [Index Fungorum](http://www.indexfungorum.org/)
reports, and many of the included sequences lack detailed taxonomic
resolution.

Starting from the most recent (as of 2024-03-25) fungal UNITE+INSD
release, this repo does the following:

- Removes sequences not identified to genus
- De-replicates sequences with the same header taxonomy and sequence
- Updates the taxonomy of the remaining sequences according to Index
  Fungorum
- De-replicates sequences (again) with the same header taxonomy and
  sequence
- Formats sequences headers in a format compatible with the
  `assignTaxonomy()` function from `dada2`

Because UNITE does not include the authority in the applied taxonomy,
this approach ends up shedding more sequences as a conservative measure,
only including sequences with unambiguous taxonomy. Also, because data
dumps of Index Fungorum taxonomy (which are difficult to find anyways!)
appear to only allow indexing at the genus-level, a genus-level ID
requirement is placed on the UNITE+INSD release.

This whole repo will (hopefully) become obsolete if:

- future releases include species hypotheses of taxa currently only
  found in UNITE+INSD releases
- fungal taxonomy of future releases matches Index Fungorum’s

The 2023-07-25 UNITE general releases seem to have species hypotheses
for taxa previously only found in UNITE+INSD releases (e.g.,
*Nothophaeocryptopus*). This is good news, but some species (e.g.,
*Micraspis strobilina*) are still missing. Additionally, some ranks
still are not in line with current Index Fungorum taxonomy (e.g.,
*Rhabdocline* in Hemiphacidiaceae vs Cenangiaceae). For this reason, I
will still use the UNITE+INSD releases moving forward.

**The updated fungal taxonomy reference is found in the `02-rename`
directory and named `if-unite-insdc.fa.gz`.**

## Plant-associated database subsets

As of 2023-03-22, this repo comes with the script `03-host.r`, which
uses the [United States National Fungus Collections Fungus-Host
Dataset](https://doi.org/10.15482/USDA.ADC/1524414) to subset the
updated reference for fungi associated with a requested plant genus. In
addition to producing a gzipped `fasta` file, a corresponding `csv`
including all the information in the United States National Fungus
Collections Fungus-Host Dataset (with corrected taxonomy for all ranks)
is also produced.

To use `03-host.r`, run the following in project directory:

    Rscript code/03-host.r 32 Pseudotsuga

Here, `32` corresponds to the number threads requested for parallel
processing, while `Pseudotsuga` is the host request. Only genus
arguments are currently accepted, but other ranks will work soon. The
main purpose of this script is to generate taxonomic references for use
as priors with the [DADA2 R package](https://github.com/benjjneb/dada2).

## ITS1, 5.8S, and ITS2-specific datasets

As of 2023-03-25, each subregion (ITS1, 5.8S, and ITS2) in
`if-unite-insdc.fa.gz` also exists in a separate file. This
functionality was added to help remove taxonomy assignment noise
associated with the `assignTaxonomy()` function in DADA2, which
implements the RDP Naive Bayesian Classifier algorithm described in
[Wang *et al*. 2007.](https://doi.org/10.1128/aem.00062-07) Essentially,
if separate databases are used (ITS1 for ITS1, ITS2 for ITS2, etc.),
this removes the chances for non-target *k*mer matches to occur (e.g.,
where an 8mer in an ITS1 ASV/OTU sequence just happens to match
somewhere in the ITS2 region of the full ITS reference).

**These split references are found in the `its1`, `5.8s`, and `its2`
subdirectories within the `04-extract` directory. Use files following
the `fun.[ITS1/5.8S/ITS2].fasta.gz` or `euk.5.8S.fasta.gz` pattern as
taxonomic references; the other files in each subdirectory are outputs
of [`ITSx`](https://microbiology.se/software/itsx/), available for
inspection.**

5.8S sequences prefixed with `euk` originate from the most current,
non-singleton eukaryote general UNITE release, facilitating non-fungal
ASV/OTU filtering prior to taxonomy assignment using one of the `fun`
ITS2 references (which originate from a processed UNITE+INSD fungal
release).

This separation also facilitates the use of both 5.8S and ITS2 for
taxonomic assignment, as demonstrated in [Heeger *et al*.
2019](https://doi.org/10.1111/2041-210X.13266). They found that ITS1 and
ITS2 sequences alone can fail to obtain taxonomic assignments at even
high ranks (e.g., kingdom, phylum, and class) when assignment reference
files are incomplete (as they always are).

## Planned improvements include:

- retention of the original accession number associated with each
  UNITE+INSD sequence entry
- more plant taxonomic ranks for `03-host.r`
- inclusion of fungal sequences at higher ranks (dependent on Index
  Fungorum data dumps)

## Reproducibility

All packages were installed and managed with `conda`.

    conda 24.1.2
    name: /home/gerverska/projects/if-unite-insdc/env
    channels:
      - conda-forge
      - bioconda
      - nodefaults
    dependencies:
      - bioconductor-biostrings=2.66.0
      - r-base=4.2.2
      - r-dplyr=1.1.2
      - r-markdown=1.6
      - r-readr=2.1.4
      - r-rmarkdown=2.21
      - r-stringr=1.5.0
      - r-tidyr=1.3.0
      - itsx=1.1.3
      - vsearch=2.22.1
      - pigz=2.8
    prefix: /home/gerverska/projects/if-unite-insdc/env

Install the above bioinformatic environment from `config.yml` using the
script `00-build.sh`

    # Clone the repo (using the GitHub CLI tool) ####
    gh repo clone gerverska/if-unite-insdc

    # Run the build script ####
    bash code/00-build.sh
