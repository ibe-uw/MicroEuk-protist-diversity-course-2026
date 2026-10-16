# MicroEuk: Protist diversity course, October 2026, Urwitałt, Poland

This is a protocol for the bioinformatic part of the **Practical Course on Analysing the Diversity of Microbial Eukaryotes in Environments**. 


<!--- TOC START -->
Table of Contents
-----------------
- [Day-1-From-Reads-to-Taxa](#Day-1-From-Reads-to-Taxa)
  - [Introduction](#Introduction)
  - [Nanopore](#Nanopore)
  - [Illumina](#Illumina)
- [Day-2-From-Taxa-to-Diversity](#Day-2-From-Taxa-to-Diversity)
  - [Introduction](#Introduction)
  - [Diversity-analysis](#Diversity-analysis)
  - [Merging-OTU-and-taxonomy-tables](#Merging-OTU-and-taxonomy-tables)
<!--- TOC END -->



## DAY 1 - From Reads to Taxa



### Introduction

On the first day of the data analysis part of the course, you'll primarily work in the terminal. You will use bioinformatic software installed in [conda](https://anaconda.org/anaconda/conda) package management system. We will analyse nanopore data using [BaNaNA](https://github.com/ibe-uw/BaNaNA) pipeline and illumina data using [QIIME2](https://qiime2.org) platform with special plugins.

We will be using [ICM UW](https://icm.edu.pl) server. To login to the server using terminal type:

```
ssh login@hpc.icm.edu.pl
```

Then type password and OTP number from authentication app.

To copy a file from the server to your computer or the opposite use the commands below. You have to type it from the location of your computer, not the server.

```
scp login@hpc.icm.edu.pl:~/location_of_the_file ./ # To copy a file to your computer
scp -r login@hpc.icm.edu.pl:~/location_of_the_file ./ # To copy a folder to your computer
scp file login@hpc.icm.edu.pl:~/location_of_the_file # To copy a file from your computer to the server
```

After you login to the server activate script:

```
/lu/topola/projects/microeuk/microeuk_startup_script.sh
```

The script activates `screen` terminal.

To check which screen you have type:

```
screen -ls
```

To login to the screen type:

```
screen -r XXX.microeuk
```

To exit the screen press ***CtrA+D*** on your keyboard.



### Nanopore


To analyse nanopore data we will be using [BaNaNA](https://github.com/ibe-uw/BaNaNA) pipeline. 


#### 1. Quality check


Firstly, on the server create a folder for nanopore analysis.

```
mkdir nanopore
```

And copy raw reads there.

```
cp ../../projects/microeuk/nanopore_raw nanopore
```

To check quality of the data we will use [NanoPlot](https://github.com/wdecoster/NanoPlot). 

```
conda activate nanoplot
cd nanopore
for folder in *; do NanoPlot --fastq "$folder"/*.fastq --tsv_stats --info_in_report -o "$folder"/nanoplot_raw ; done
```

NanoPlot produces a lot of outputs. One of them is general statistics `NanoStats.txt`. The other ones are plots. Download to your computer two of them `LengthvsQualityScatterPlot_dot.png` and `Non_weightedHistogramReadlength.png` and inspect.


#### 2. Copy the repository from shared folder into the `nanopore` folder

```
cp ../../projects/microeuk/BaNaNA .
```


#### 3. Fill in the config file

Inside the main folder you will find `config.yaml` file. Download it to your computer and Fill it according to the instruction.

* `sample_name`: Provide names of you samples.

The name should only contain `-` sign (make sure you avoid `_` sign as well). Also, they need to be the same as the names of the FASTQ files. For example, if your files are `BAB10.fastq` and `MIK3.fastq`, then the `sample_name` option should look like this:

```
sample_name:
 - BAB10
 - MIK3
```

* `threads`: Number of threads to use by the pipeline. 
* `min_len_filtering`: Lower threshold for your amplicon length filtering. 
* `max_len_filtering`: Upper threshold for your amplicon length filtering. 
* `rrnas`: Your amplicons may be longer than 18S rRNA gene and contain multiple rRNA genes (like 5.8S rRNA and 28S rRNA for eukaryotes). Specify all rRNA genes included in your whole amplicon and their minimal length contained in the amplicon.

If your amplicon contains three genes, this option should look like this:

```
rrnas: 18S_rRNA:1000,5_8S_rRNA:90,28S_rRNA:400
```

And if you amplicon contains only 18S rRNA gene, this option should look like this:

```
rrnas: 18S_rRNA:1000
```

* `chosen_rrna`: Specify which rRNA gene, included in the `rrnas` option you want to keep for futher analysis. Choose only one gene, ***the default one is `18S_rRNA` gene***.
* `db_location`: Provide absolute path to the reference database you want to use to assign the taxonomy with.
* `db_id`: Specify minimal identity of OTUs to the closest reference sequence for taxonomic annotation. ***Default is `0.7`***.
* `db_query_cov`: Specify minimal coverage of OTUs to the closest reference sequence for taxonomic annotation. ***Default is `0.9`***.
* `enable_optional_taxonomy_format`: It's an optional step applied only for the PR2 database UTAX version (look ***Download PR2 database*** chapter), which creates a tab-separated table from raw VSEARCH output. If you are using a different database than PR2 or a different version of PR2 database, set this option to `false`.


#### 4. Preparing the repository

Upload `config.yaml` file into the BaNaNA folder.


Make all the scripts executable

```
chmod a+x BaNaNA-main/scripts/*
```

Copy raw reads to the `samples` folder


#### 5. Run the pipeline

Add condo channels.

```
conda config --add channels conda-forge
conda config --add channels bioconda
```

Run the command to install or the dependencies (You have to be located in the BaNaNA folder).

```
snakemake --use-conda --conda-create-envs-only
```

If everything installed correctly run the analysis.

```
snakemake --configfile config.yaml --use-conda --cores 4
```


#### 6. Find the final output

The piepline creates a folder `final` and places there all four final output files:
* `otus.fasta` - final OTUs sequences
* `otu_table.tsv` - table of abundance of each OTU in each sample
* `taxonomy.tsv` - taxonomy of each OTU directly from VSEARCH
* `taxonomy_table.tsv` - taxonomy of each OTU in a form of a TSV table (created based on `taxonomy.tsv`)










### Illumina


#### 1. Downloading scripts and preparing the working environment

First, download the repository from GitHub to your computer, unzip and then navigate to the unzipped directory. Next, upload the folder `scripts_illumina` to your home folder on the server using the command below (or another solution that works on your system)

```
scp -r scripts_illumina login@hpc.icm.edu.pl:~
```

Make all the Python scripts executable.

```
chmod a+x ./scripts_illumina/*.py
```

In the next step create and enter the folder illumina:

```
mkdir illumina
cd illumina
```

<details>
  <summary>Your home folder should look like this </summary>

  ![Description of the image](imgs/folders.png)
</details> 


#### 2. Raw reads quality check

First, check read quality using [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). The Illumina sequencing reads from the experiment are available on the server.

```
mkdir fastqc
Honda activate fastqc_env
for file in /lu/topola/projects/microeuk/illumina_raw_data/*; do fastqc "$file" -outdir fastqc; done
```

Download `.html` files on your computer and open them in the browser.

Please take a look at the command below that you will run using a terminal window on your local machine.

```
scp login@hpc.icm.edu.pl:~/illumina/*.html localdirectory
``` 

***Is the quality good or bad? What else did you notice?***


#### 3. Activating the QIIME 2 environment

Now you will start working in the QIIME 2 environment.

```
conda activate qiime2-2024.2_env
```


#### 4. Importing data

The first step is to import your `.fastq` files to a special `.qza` artifacts file. To do that you will use `manifest.tsv` file, which contains locations of the raw reads.

```
qiime tools import --type "SampleData[PairedEndSequencesWithQuality]" --input-format PairedEndFastqManifestPhred33V2 --input-path ../scripts_illumina/manifest.tsv --output-path ./demultiplexed-seqs.qza
```

#### 5. Primer trimming

To cut amplification primers you will use the [Cutadapt](https://cutadapt.readthedocs.io/en/stable/) plugin. In this step, you will remove the amplification primers from the file imported in previous step `demultiplexed-seqs.gza`.

```
qiime cutadapt trim-paired --i-demultiplexed-sequences ./demultiplexed-seqs.qza --p-cores 4 --p-front-f CCAGCASCYGCGGTAATTCC --p-front-r ACTTTCGTTCTTGATYRA --o-trimmed-sequences trimmed_demux.qza
```

#### 6. Visualisation of trimmed data quality

In this step, you will create QIIME 2 artifact .qzv which allows data visualisation. 

```
qiime demux summarize --i-data trimmed_demux.qza --o-visualization trimmed_demux.qzv
```

Download `trimmed_demux.qzv` on your computer and upload the file on the [QIIME2View](https://view.qiime2.org) website.

***Investigate both tabs***.


#### 7. Quality filtering, denoising, merging and chimera removal

You will use the [DADA2](https://benjjneb.github.io/dada2/) software to create Amplicon Sequence Variants (ASVs).

This step inculdes length trimming. ***How much would you trim forward and reverse reads, to find balance between quality and merging?*** Use `trimmed_demux.qzv` quality plots as a clue.

```
qiime dada2 denoise-paired --p-n-threads 4 --i-demultiplexed-seqs trimmed_demux.qza --p-trunc-len-f ? --p-trunc-len-r ?  --output-dir dada2
```

<details>
  <summary>Suggested command</summary>
  
  ```
  qiime dada2 denoise-paired --p-n-threads 4 --i-demultiplexed-seqs trimmed_demux.qza --p-trunc-len-f 205 --p-trunc-len-r 200  --output-dir dada2
  ```
</details> 


#### 8. Visualisation of DADA2 outputs

Create `.qzv` for all the outputs.

```
qiime metadata tabulate --m-input-file dada2/denoising_stats.qza --o-visualization denoising_stats.qzv
qiime feature-table summarize --i-table dada2/table.qza --o-visualization table.qzv
qiime feature-table tabulate-seqs --i-data dada2/representative_sequences.qza --o-visualization representative_sequences.qzv
```

Again upload files on the [QIIME2View](https://view.qiime2.org) website and ivestigate them.

***What % of your reads merged successfully?***

Additionally, when reviewing the representative sequences (your ASVs) you have obtained (`representative_sequences.qzv`), download them from the QIIME2View website in FASTA format. 

<details>
  <summary>Help here</summary>

  ![Description of the image](imgs/rep_seq.png)
</details>

 
 And finally upload raw representative sequences (fasta format) to your working directory on the server (folder `illumina`).


#### 9. Exporting files

To assign taxonomy for further diversity analysis, you need to download two files: `ASV table` (table.qza) and `representative sequences` in a fasta format.

To read the feature table, which represents your ASVs table, you need to export it using QIIME2:

```
qiime tools export --input-path dada2/table.qza --output-path exported
biom convert --to-tsv -i exported/feature-table.biom -o exported/feature-table.tsv
```

Created files you will find in the folder 'exported'

Download `feature-table.tsv` to your computer.


#### 10. Taxonomic annotation

You will assign taxonomy using the [VSEARCH](https://github.com/torognes/vsearch) software, which uses the global alignment method. For reference, you will use the [PR2](https://pr2-database.org) database.

```
conda activate vsearch_env
vsearch --usearch_global sequences.fasta --db /lu/topola/projects/microeuk/pr2_version_5.1.1_SSU_UTAX.fasta --blast6out taxonomy.tsv --id 0.70
```

***Which other databases and methods of assigning taxonomy do you know?***


#### 11. Modifying outputs for diversity analyses 

Open the downloaded `feature-table.tsv` in Excel, remove the first raw, and change the name in cell A2 from `#OTU ID` to `ASV` and save the changes.

<details>
  <summary>Help here</summary>

  ![Description of the image](imgs/table.png)
</details>

***What information does this file contain?***


Using Python script you will modify `taxonomy.tsv`

```
../scripts_illumina/get_taxonomy_table.py -i taxonomy.tsv -o taxonomy_table.tsv
```

<details>
  <summary>In the case of permission denied error</summary>

  In some cases, scripts will run, only after changing their permission rules.

  You can easily fix it by navigating to the 'scripts' folder and setting new permissions.

  ```
chmod 777 *
```
</details>


Download taxonomy_table.tsv to your computer and open it in Excel. Take a look at the data to determine which taxa are present.

***Which taxa are most represented in your samples?***

#### !!! <ins>FINAL OUTCOMES</ins> !!!

Congratulations! You've just finished the illumina part of today's data analysis!
As a final outputs, you obtained:
* `feature-table.tsv` - table of ASV abundances
* `taxonomy_table.tsv` - table of taxonomy for each ASV




## DAY 2 - From Taxa to Diversity


### Introduction

During this part you will use your OTU and taxonomy tables and metadata table to perform different statistical analysis. You will work in [RStudio](https://posit.co/download/rstudio-desktop/) and use R programming language to analyse your data.

### Diversity analysis

For the diversity analysis we will use `diversity_analysis.Rmd` file and already published freshwater nanopore data.

### Merging OTU and taxonomy tables

To merge OTU and taxonomy tables coming from different analysis and sequencing technologies we will use `merging_tables.Rmd` file. The file are merged by the assigned species.

