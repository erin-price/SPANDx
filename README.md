# SPANDx - a comparative genomics pipeline for microbes

SPANDx was written by Derek Sarovich ([@DerekSarovich](https://twitter.com/DerekSarovich)) and Erin Price ([@Dr_ErinPrice](https://twitter.com/Dr_ErinPrice)) at University of the Sunshine Coast, Queensland, Australia.
## Contents

- [Updates](updates)
- [Introduction](#introduction)
- [Installation](#Installation)
- [Resource Managers](#resource-managers)
- [SPANDx Workflow](#spandx-workflow)
- [Usage](#usage)
- [Parameters](#parameters)
- [Important Information](#important-information)
- [mGWAS and SPANDx](#mGWAS-and-spandx)
- [How to find a snpEff database](#How-to-find-a-snpEff-database)
- [Citation](#citation)

## Updates

#### SPANDx 4.0
As of SPANDx version 4.0 we've completely overhauled the software and added a few cool features. SPANDx now uses Nextflow for job and pipeline management (https://www.nextflow.io/docs/latest/index.html). This means that we no longer use the clunky job management script that had issues running on HPCs with odd resource management systems or strict resource requesting requirements. SPANDx is now mixture aware (!), at least when it comes to SNP identification and annotation. Mixed SNPs will still be excluded from phylogenetic analysis, which is intentional, but can be found in the individual gVCFs/VCFs. We've switched to the GATK 4.0+ and now use gVCFs instead of the standard VCF format. Standard VCFs are still produced but will probably be removed in future versions. We've done away with using paup or RAXML for tree construction and just use Fastree for both ML and MP. Trees are now drawn as part of the pipeline rather than having to do an additional step after the pipeline completes.

------------------
#### SPANDx 3.2
As of SPANDx version 3.2 we've upgraded to BWA-mem alignment (BWA v0.7+). This algorithm provides vastly improved intra-genus SNP/indel identification, and improved intra-species variant identification, and will be used as default for SPANDx versions post v3.2. If you would like to use a pre 0.7 version of bwa with the aln/sampe algorithm, please use a version of SPANDx pre v3.2. All version are available on sourceforge https://sourceforge.net/projects/spandx/

## Introduction

SPANDx (Synergised Pipeline for Analysis of NGS Data in Linux) is a genomics pipeline for comparative analysis of haploid whole genome re-sequencing datasets. 

**Why use SPANDx?**

SPANDx is a pipeline for identifying SNP and indel variants in haploid genomes using NGS data. SPANDx performs alignment of raw NGS reads against your chosen reference genome or pan-genome, followed by accurate genome-wide variant calling and annotation, and locus presence/absence determination. SPANDx produces handy SNP and indel matrices for downstream phylogenetic analyses. Annotated SNPs and indels are identified and output in human-readable format. A presence/absence matrix is generated to allow you to identify the core/accessory genome content across all your genomes. The outputs generated by SPANDx can also be imported into PLINK for microbial genome-wide association study (mGWAS) analyses.

## Installation

**Github**

1) Download the latest installation with git clone and install into a directory called SPANDx

```
git clone https://github.com/dsarov/SPANDx.git ./SPANDx
```

2) Install a spandx environment using conda

```
conda env create --name spandx -f ./SPANDx/env.yaml
```
**Conda**

Short version for those that just want to get started and understand how environments in conda work.
Note that a recent change in the conda-forge channel means this method will likely fail until I can fix the issues.
SPANDx is available on our development channel and its dependencies can be installed with:
```
conda install -c dsarov -c bioconda spandx
```

Update to the latest version from github
```
nextflow pull dsarov/spandx
```

The pipeline itself is run with Nextflow from a local cache of the repository:
```
nextflow run dsarov/spandx
```
The local cache can be updated with
```
nextflow pull dsarov/spandx
```
If you want to make changes to the default nextflow.config file clone the workflow into a local directory and change parameters in nextflow.config:
```
nextflow clone dsarov/spandx install_dir/
```
Or navigate to the conda install path of SPANDx and change the nextflow.config in that location.

Long version for those unfamiliar with environments or just want all the steps for recommended installation
Make sure you have the conda package manager installed (e.g. Anaconda, miniconda). You can check this by testing if you can find the conda command (which conda). If you do have conda installed then it's a good idea to update conda so you have the latest version conda update conda. If you don't have this software installed then go to the miniconda install page and follow the instructions for your OS. After the install, make sure your install is up-to-date conda update conda.

Create a new environment with conda called "spandx" and install the software with 
```
conda create --name spandx -c dsarov -c bioconda spandx
```
Follow the instructions and the software should fully install with all dependencies.

Activate the spandx environment that was installed by conda, 

```
conda activate spandx
```

To run SPANDx, 
```
nextflow run /path/to/SPANDx/main.nf
```
or
```
nextflow run dsarov/spandx
```
If you are running this pipeline on a HPC/submission system (e.g. PBS) then using the screen command will allow you to detach the terminal while the pipeline is still running in the background## Installation


## Resource Managers

From v4.0 SPANDx should work with any resource manager and can also be run directly without a resource using Nextflow. To specify a specific reource manager, use the`` --executor`` flag. E.g. ```--executor pbspro```. For more information see the Nextflow documentation here https://www.nextflow.io/docs/latest/executor.html

## SPANDx Workflow

To achieve high-quality variant calls, SPANDx incorporates the following programs into its workflow:

- ART
- Burrows Wheeler Aligner (BWA)
- BEDTools
- seqtk
- Trimmomatic
- Mosdepth
- Picard
- SAMTools
- Picard
- Genome Analysis Toolkit (GATK)
- BEDTools
- SNPEff
- VCFtools
- Fasttree

## Usage (Version 4+)

Input Parameter:

```--fastq```

Input PE read file wildcard (default: "*_{1,2}.fastq.gz")
                
```--ref ```       

Reference genome for alignment. If annotation is requested, then this genome must match the annotated genome that is used by snpEff (i.e. the genome specified by the --database flag)
                
Optional Parameters:

```--annotation  ``` 
  
Optionally output annotated variant tables. If you want to annotate the variant output then set this parameter to the name of the variant file in snpEff
(default: false)
                 
```--database   ```

If you want to annotate the variant output then set this parameter to the name of the variant file in snpEff
(default: false)
                
```--phylogeny ```   

If you would like to switch off phylogenetic reconstruction and just generate a list of SNPs/indels then swith this parameter to false. 
(default: true)
            
```--window    ```

Default window size used in the bedcov coverage assessment
(default: 1kb)

```--assemblies  ``` 

Optionally include a directory of assembled genomes in the analysis. Set this parameter to 'true' if you wish to included assembled genomes and place all assembled genomes in a
subdirectory called 'assemblies'. 
(default: false)

```--size    ```  

SPANDx can optionally down-sample your read data to run through the pipeline quicker. Set to 0 to skip downsampling (default: 0). NB the number specified here refers to the number of reads kept in each pair. Genome coverage will vary with genome size and sequence length.

```--tri_allelic  ```

Set to true if you would like tri-allelic SNPs/indels used in the phylogenetic analysis 
(default: false).

```--indels```

Set to true if you would like indels used in the phylogenetic analysis 
(default: false).

```--mixtures```  

Optionally perform within species mixtures analysis. Set this parameter to 'true' if you are dealing with multiple strains within the same WGS sample 
(default: false). Note that this feature is not currently implemented.

```--structural```   

Set to true if you would like to identify structural variants Note that this step can take a considerable amount of time if you have deep sequencing data 
(default: false).

```--notrim```       

Although not generally recommended to switch off, set to true if you want to skip the timmomatic step 
(default: false).

As a feature of Nextflow, SPANDx can resume a failed run attempt, using the previously generated, intermediate files. To use this feature, add the ```-resume ```flag to the command line when running SPANDx.

If you want to make changes to the default `nextflow.config` file
clone the workflow into a local directory and change parameters
in `nextflow.config`:
    nextflow clone dsarov/spandx outdir/
Update to the local cache of this workflow:
    nextflow pull dsarov/spandx

## Usage (prior to version 4)

```bash
SPANDx.sh -r REFERENCE
```
## Parameters

    Required Parameters:
      -r            reference, without .fasta extension
      
    Optional Parameters:
      -o            Organism name
      -m            Generate SNP matrix - yes/no  
      -i            Generate indel matrix - yes/no 
      -a	        Include annotation - yes/no
      -v            Variant genome file - Name must match the SnpEff database 
      -s            Specify read prefix to run single strain 
      -t            Sequencing technology used Illumina/Illumina_old/454/PGM (default: Illumina)
      -p            Pairing of reads - PE/SE (default: PE)
      -w            Window size in base pairs for BEDcoverage module (default: 1000)
      -z            Include tri- and tetra-allelic SNPs in the SNP matrix - yes/no

## Important Information

### File names
SPANDx, by default, expects reads to be paired-end, Illumina data in the following format: 

```
STRAIN_1.fastq.gz (first pair) 
STRAIN_2.fastq.gz (second pair)
```
Reads not in this format will be ignored. If your data are not paired, you must set the -p parameter to SE to denote unpaired reads.

### SPANDx requires a reference file in FASTA format

For compatibility with all steps in SPANDx, FASTA files should conform to the specifications [listed here](http://www.ncbi.nlm.nih.gov/BLAST/blastcgihelp.shtml). Note that the use of nucleotides other than A, C, G, or T is not supported by certain programs in SPANDx so should not be used in reference FASTA files. In addition, Picard, GATK and SAMtools handle spaces within contig names differently. Therefore, please do not use spaces or special characters (e.g. $/*) in contig names.

By default, all reads in SPANDx format (i.e. strain_1_sequence.fastq.gz) in the present working directory are processed. Sequence files are aligned against the reference using BWA. Alignments are subsequently filtered and converted using SAMtools and Picard Tools. SNPs and indels are identified with GATK and coverage assessed with BEDtools. 


### Variant Identification

All variants identified in the single genome analysis are merged and re-verified across the entire dataset to minimise incorrect variant calls. This error-correction step is an attempt at establishing a "Best Practice" guideline for datasets where variant recalibration is not yet possible due to a lack of comprehensive quality-assessed variants (i.e. most haploid datasets!). Such datasets are available for certain eukaryotic species (e.g. human, mouse), as detailed in GATK "Best Practice" guidelines.

Finally, SPANDx merges high-quality, re-verified SNP and indel variants into user-friendly .nex matrices, which can be used for phylogenetic resconstruction using various phylogenetics tools (e.g. PAUP, PHYLIP, RAxML).

## mGWAS and SPANDx

The main comparative outputs of SPANDx (SNP matrix, indel matrix, presence/absence matrix, annotated SNP matrix and annotated indel matrix in $PWD/Outputs/Comparative/) can be used as input files for mGWAS. From version 2.6 onwards, SPANDx is distributed with GeneratePlink&#46;sh. The GeneratePlink&#46;sh script requires two input files: an ingroup.txt file and an outgroup.txt file. The ingroup.txt file should contain a list of the taxa of interest (e.g. antibiotic-resistant strains) and the outgroup.txt file should contain a list of all taxa lacking the genotype or phenotype of interest (e.g. antibiotic-sensitive strains). The ingroup.txt and outgroup.txt files must include only one strain per line. Although larger taxon numbers in the ingroup and outgroup files will increase the statistical power of mGWAS, it is better to only include relevant taxa i.e. do not include taxa that have not yet been characterised, or that have equivocal data. The GeneratePlink&#46;sh script will generate .ped and .map files for SNPs, and presence/absence loci and indels if these were identified in the initial analyses. The .ped and .map files can be directly imported into PLINK. For more information on mGWAS and how to run PLINK, please refer to the [PLINK website](http://pngu.mgh.harvard.edu/~purcell/plink/)

### GeneratePLINK usage:

```bash 
GeneratePLINK.sh -i ingroup.txt -o outgroup.txt -r reference (without .fasta extension)
```
Comparing microbial genomes with the above methods will test for associations with orthologous and non-orthologous SNPs, indels and a presence/absence analysis. For more thorough mGWAS the accessory genome also needs to be taken into account and is a non-trivial matter in microbes due to the presence of multiple paralogs. To accurately characterise the accessory genome an accurate pan-genome is required. To construct a pan-genome we recommend the excellent pan-genome software, [Roary](https://sanger-pathogens.github.io/Roary/). Once a pan-genome has been created the script GeneratePLINK_Roary.sh can be used to analyse associations between the groups of interest. GeneratePLINK_Roary&#46;sh is run similarly to GeneratePLINK.sh and requires both an ingroup.txt file and an outgroup.txt file. Once this script has been run the .ped and .map files should be directly importable into PLINK.

### GeneratePLINK_Roary.sh usage

```bash
GeneratePLINK_Roary.sh -i ingroup.txt -o outgroup.txt -r Roary.csv output (if different than the default gene_presence_absence.csv)
```
**Note that this script has an additional requirement for R and Rscript with the dplyr package installed. If this script can’t find these programs in your path then it will fail**

### How to find a snpEff database

SPANDx is able to annotate all variants in a combined SNP/indel matrix that makes searching for specific mutations easy. If you are running SPANDx with an new bacterial species, and you are unsure if there is a required database for annotation, you can check to see what exists in snpEff and to see if it is already installed on your system. Post SPANDx v4.0, to see what version of snpEff is being used first load the SPANDx environment ```conda activate spandx``` then ```snpEff databases | grep "Genus_species"```

snpEff has a massive number of databases, so if you are using a published reference sequence, chances are it will have an existing database. One caveat is that the chromosome name of the reference fasta file must match that of the snpEff database. The name of the bacterial chromosome is normally '>Chromosome' so the start of the fasta file should be named with >Chromosome rather than the default NCBI naming. If these don't match, you'll get an empty output for the annotated variants. Also as of version 4.0, SPANDx (and snpEff) will try to automatically download and install any new databases requested by the user. Although this generally works, the download may fail and the database may need to be installed manually. Most of the hurdles that we've encountered with automatic installation are due to proxy settings on the host system, not letting snpeEff download and install automatically. This limitation has been addressed in v4.0 but keep this in mind if annotations are not working properly.

## Citation

Sarovich DS & Price EP. 2014. SPANDx: a genomics pipeline for comparative analysis of large haploid whole genome re-sequencing datasets. <i>BMC Res Notes</i> 7:618.

Please send bug reports to derek.sarovich@gmail.com.
