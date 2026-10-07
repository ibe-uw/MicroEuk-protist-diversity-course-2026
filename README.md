# MicroEuk: Protist diversity course, October 2026, Urwitałt, Poland

This is a protocol for the bioinformatic part of the course. 


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
  - [Merging-OTU-and-taxonomy tables](#Merging-OTU-and-taxonomy-tables)
<!--- TOC END -->



## DAY 1 - From Reads to Taxa


### Introduction

On the first day of the data analysis part of the **Practical course on analysing the diversity of microbial eukaryotes in environments**, you'll primarily work in the terminal. You will use bioinformatic software installed in [conda](https://anaconda.org/anaconda/conda) package management system. We will analyse nanopore data using [BaNaNA](https://github.com/ibe-uw/BaNaNA) pipeline and illumina data using [QIIME2](https://qiime2.org) platform with special plugins.

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



### Nanopore










### Illumina


#### 1. Downloading scripts and preparing the working environment

First, download the repository from GitHub to your computer, unzip and then navigate to the unzipped directory. Next, upload the folder 'scripts_illumina' to your home folder on the server using the command below (or another solution that works on your system)

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
for file in ../../4UProtistDiversity/raw_illumina/*; do fastqc "$file" -o ./; done
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
conda activate qiime2
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
vsearch --usearch_global sequences.fasta --db /mnt/databases/pr2_db/pr2_database-5.0.0.fasta --blast6out taxonomy.tsv --id 0.70
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

