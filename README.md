# SARS-CoV-2 viral genomics

### A portal for sharing data, analyses, and results analyzing genomic variation for SARS-CoV-2


**This work is on-going - please feel free to reach out with corrections, additions, or questions.**

This analysis was performed primarily by Matt Olm (mattolm@stanford.edu) in Justin Sonnenburg's lab at Stanford University and Alex Crits-Christoph (crist-christoph@berkeley.edu) from Jill Banfield's lab at University of California, Berkeley.


## Interpatient variation

### Data processing

We are working with viral genomes from NCBI's data hub:
https://www.ncbi.nlm.nih.gov/labs/virus/vssi/#/virus?SeqType_s=Nucleotide&VirusLineage_ss=SARS-CoV-2,%20taxid:2697049

But a similar analysis could be done on the larger collection of sequences available in [GISAID](gisaid.org). Currently there are about ~170 genomes in NCBI and over 800 in GISAID.

A FASTA file of all currently available NCBI sequences can be found in `./interpatient/ncbi_mar20.fna`.

Filtering:
The script `./interpatient/filter_seqs.py` will perform some quality checks on the NCBI sequences. Firstly, several of the sequences are too short, and do not represent full viral genomes. We filter out all sequences < 29 Kb, as the viral genome is ~30 Kb. There are also several sequences that are too short in GISAID.

This script then filters out viral genomes that are too divergent to represent accurate genomes from the 2019 SARS-CoV-2 pandemic. This second step requires the program `fastANI` be installed in your path. It compares all genomes to the reference SARS-CoV-2 (`./interpatient/reference.fna`) and removes those that are <99% nucleotide identity. 

We then align the resulting filtered sequences using MAFFT - `mafft --thread 16 final_filtered_seqs.fna > mar20_filtered.aln` to generate a full genome alignment of the high quality viral genomes.

We generate a phylogenetic tree of these sequences using IQTree:

`iqtree -nt 10 -s final_filtered.aln`
and the generated tree is available in `./interpatient/mar20_filtered.aln.iqtree`.

**The viral genome alignment is made available in `./interpatient/mar20_filtered.aln.

### Notebooks and analysis

We conduct a number of analyses on the alignment. In general, we want to know which positions in the genome are variable, and how variable they are. The notebook `interpatient_pi.ipynb` calculates nucleotide diversity in the alignment, with respect to the reference genome's gff annotation. It produces the following figures illustrating nucleotide diversity across the entire genome and by each gene and/or coding sequence:

Tabular data on the raw nucleotide diversity per reference position can be found in `./interpatient/nucleotide_diversity.txt`. Note that this calculation ignores any insertions or deletions from the reference and only tracks single nucleotide polymorphisms.

The notebook `./interpatient_snps.ipynb` then generated the following table of genomic substitutions across patients in `./interpatient/substitutions.txt`:

```
Position,Reference_base,Alternative_base,Ref_frequency,Alt_frequency,CodingVariant
100,G,A,0.8,0.2,S
101,G,A,0.9,0.1,S:V
```
Describing the position, nucleotides, allele frequencies, and impact on amino acid sequence (non-synonymous vs synonymous) of each substitution.

## Intrapatient variation

Goals: detecting and tracking viral genetic variation within individuals. Within-patient data can have the following applications:

* Identification of genomic loci less likely to mutate during infection.
* Identification of genomic loci under different modes of selection.
* Comparison of viral evolution within individuals versus global evolution.
* Possible estimation of the number of viral particles acquired to start infection.

### Raw data

You can acquire either sequencing reads or assembled viral genomes from:

https://www.ncbi.nlm.nih.gov/genbank/sars-cov-2-seqs/

https://gisaid.org


### Parsed data tables (`./datatables`)

* SRA_metadata_v1.csv - Metadata of the SRA samples used for intra-patient analysis

* COVID_genome_coverage_v1.csv - Genome-wide coverage, breadth, nucleotide diversity, etc. of reads mapping to a COVID-19 reference genome

* COVID_positional_coverage_v1.csv - Position by position coverage and nucleotide diversity along the genome in sequencing reads

* COVID_SNPs_v1.csv - Genomic locations of intrapatient SNPs

### Intrapatient methods

This method requires access to the raw sequencing reads generated when sequencing covid19 genomes. Usually these reads are used to generate a viral genome, the genome is deposited into a public database, and the raw reads are never uploaded publicly. **If you are involved in covid-19 genome sequencing efforts, please consider uploading the raw reads as well so that analyses like this can continue.**


### Quality control

