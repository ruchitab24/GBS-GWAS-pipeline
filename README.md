# Bioinformatics Workflow Description

The bioinformatics workflow is split into 'Pipeline' and 'Post-pipeline' steps, consisting of steps that can be run separately for each sample and steps that require inputs from all samples, respectively.

![alt text](https://github.com/Leacavalli/GBS-GWAS-pipeline/blob/main/Bioinfo_flowchart.png)


# Pipeline Description
This Nextflow pipeline was built to integrate the following steps:
<br>   

| Steps  | Description  | Tool |
| ------------- | ------------- | ------------- |
| 1 | Trim sequencing primers  | [Trimmomatic](https://github.com/timflutre/trimmomatic)  |
| 2 | Reads Quality Control | [FastQC](https://github.com/s-andrews/FastQC)  |
| 3 | Determine the serotype and ST of isolates | [SRST2](https://github.com/katholt/srst2), with [GBS-SG](https://github.com/swainechen/GBS-SBG)  
| 4 | Mapping to reference genome and variant calling | [Snippy](https://github.com/tseemann/snippy)  |
| 5 | Assemble reads  | [Unicycler](https://github.com/rrwick/Unicycler)  |
| 6 | Genome Annotation | [Prokka](https://github.com/tseemann/prokka)  |
| 7 | Identify contaminated samples  | [FastANI](https://github.com/ParBLiSS/FastANI) |
| 8 | Assembly Quality Control (N50, # Contigs, GC %) | [quast](https://github.com/ablab/quast) |
| 9 | Obtain Sequence Cluster (SC) classification | [POPPUNK](https://github.com/bacpop/PopPUNK) |
| 10 | Obtain Clonal Complex (CC) classification | Costume code using [POPPUNK]() |
| 11 | Generate a core genome SNP alignment | [snippy-core, snippy-clean_full_aln](https://github.com/tseemann/snippy) |
| 12.1 | Make accurate ML phylogeny | [RAxML](https://cme.h-its.org/exelixis/web/software/raxml/) |
| 12.2 | Make fast ML phylogeny | [FastTree ](http://www.microbesonline.org/fasttree/) |
| 13.1 | Pangenome analysis: identify accessory COGs | [panaroo](https://github.com/gtonkinhill/panaroo) |
| 13.2 | Pangenome analysis: identify accessory COGs | [roary](https://sanger-pathogens.github.io/Roary/) |
| 13.3 | Pangenome analysis: re-classify accessory COGs to reduce redundancy | [CLARC]() |
| 14 | Call unitigs | [unitig-counter](https://github.com/bacpop/unitig-counter) |
| 15 | SNP-GWAS, DB-GWAS and PanGWAS | [Pyseer](https://pyseer.readthedocs.io/en/master/) |

# Pipeline Architecture

Nextflow_pipeline\
&nbsp;&nbsp;| --- scripts\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- nextflow.config\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- Nextflow_pipeline.nf\
&nbsp;&nbsp;| --- Files\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- AP018935.1.fa\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- GBS-SBG.fasta\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- Streptococcus_agalactiae.fasta\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- profiles_csv\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- GBS_CC_profiles.csv\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- Snippy_environment.yml\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- Panaroo_environment.yml\
&nbsp;&nbsp;|--- 0.RAW_READS\
&nbsp;&nbsp;|--- 1.TRIMMED_READS\
&nbsp;&nbsp;|--- 2.FASTQC\
&nbsp;&nbsp;|--- 3.SEROTYPE_MLST\
&nbsp;&nbsp;|--- 4.SNIPPY\
&nbsp;&nbsp;|--- 5.ASSEMBLIES\
&nbsp;&nbsp;|--- 6.ANNOTATION\
&nbsp;&nbsp;|--- 7.FastANI\
&nbsp;&nbsp;|--- 8.QUAST\
&nbsp;&nbsp;|--- 9.POPPUNK\
&nbsp;&nbsp;|--- 10.ASSIGN_CC\
&nbsp;&nbsp;|--- 11.SNIPPY_MULTI\
&nbsp;&nbsp;|--- 12.Phylogeny\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 12.1.RAxML\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 12.2.FastTree\
&nbsp;&nbsp;|--- 13.Pangenome_analysis\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 13.1.Panaroo\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 13.2.ROARY\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 13.3.Panaroo_CLARC\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 13.4.ROARY_CLARC\
&nbsp;&nbsp;|--- 14.UNITIGS\
&nbsp;&nbsp;|--- 15.Pyseer\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 15.1.Main_analysis\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 15.2.Subanalysis_1\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|--- 15.3.Subanalysis_2\



# Running the Pipeline
The pipeline contains the steps that can be run for each sample individually. From read trimming and filtering to genome annotation.
## 1. Clone this Github
```
git clone https://github.com/Leacavalli/GBS-GWAS-pipeline.git
```
## 2. Install Java + Nextflow
### 2.1. Install Java v.11 to run Nextflow
```
module load python
conda create -n java      # Only run once to set up; All following times, skip this
conda activate java
conda install openjdk=11  # Only run once to set up; All following times, skip this
java -version             # Check java version
```
### 2.2. Install Nextflow
```
# Download the executable package:
wget -qO- https://get.nextflow.io | bash   
# Make the binary executable on your system by running:  
chmod +x nextflow  
# Add executable to $PATH                           
nano  ~/.bashrc      
# add line at the end of file:
export PATH="<path_to_nextflow>:$PATH"
# exist file with ctlX and Y
source ~/.bashrc      
```

## 3. Install Pipeline Dependencies
### 3.1. Install Trimmomatic
```
cd Files
wget https://github.com/usadellab/Trimmomatic/files/5854859/Trimmomatic-0.39.zip
unzip Trimmomatic-0.39.zip
```
### 3.2. Install FastTree
```
curl -O http://www.microbesonline.org/fasttree/FastTree.c
gcc -O3 -finline-functions -funroll-loops -Wall -o FastTree FastTree.c -lm
FastTree -h
```
### 3.3. Install RAxML
```
# download
cd Files
wget https://github.com/stamatak/standard-RAxML/archive/master.zip
unzip master.zip
# compile source code
cd standard-RAxML-master/
make -f Makefile.SSE3.PTHREADS.gcc # parallelized and x86 processor optimized version
ls raxmlHPC*
```
### 3.4. Install CLARC
```
cd Files
git clone https://github.com/IndraGonz/CLARC.git
cd CLARC/envs
module load python
conda env create --file clarc_env.yml
source activate clarc_env
cd ../
python setup.py install
```
### 3.5. Install Pyseer
```
cd Files
mkdir pyseer
cd pyseer
wget https://github.com/mgalardini/pyseer/blob/4b8d22f43bc5943483d9a54df1e22c6a35cd0121/scripts/phylogeny_distance.py
git clone https://github.com/mgalardini/pyseer
```
## 4. Prepare input data
### 4.0. (Optional) Download Raw reads from NCBI
Note: SraAccList.txt contains the list of accessions you want
```
cd 0.RAW_READS
sbatch -p shared -t 1-00:00 --mem=100000 --wrap="prefetch --option-file SraAccList.txt"
for i in *RR*
do
  sbatch -p shared  -t 0-00:10 --mem=10000 --wrap="fasterq-dump --split-files $i/*sra"
done
```
### 4.1. Place your gunzipped raw reads in /NEXTFLOW_PIPELINE/0.RAW_READS/
### 4.2. Place your your _phenotypes.txt_ file in /NEXTFLOW_PIPELINE/Files/

## 5. Run Nextflow
The following options are required to run the pipeline for each isolates:
```
Usage: nextflow run <Nextflow_script> [args...]
Arguments:
  Necessary
        -C                     <Nextflow configuration file (Available in scripts directory)>
        --path_nextflow_dir    <Path to your Nextflow Directory>
        --path_conda_envs      <Path to your local conda environments Directory>
  Options:
        --main                 <Runs the Main Analysis: SNP-GWAS, DB-GWAS and Pan-GWAS, with the
                                population structure controlled using a RAxML phylogeny and the accessory genome defined per Panaroo.>
        --sub1                 <Runs Sub-Analysis #1: SNP-GWAS, DB-GWAS and Pan-GWAS, with the population
                                structure controlled using a Fastree phylogeny, Mash distances and Sequence Cluster (SC) classifications.>
        --sub2                 <Runs Sub-Analysis #2: Pan-GWAS, with the accessory genome defined per
                                Roary, CLARC-redefined Panaroo genes, and CLARC-redefined Roary genes.>
        --Fasttree             <Runs Main analysis with population structure controlled using a Fastree phylogeny.>
        --Mash                 <Runs Main analysis with population structure controlled using Mash distances>
        --SC                    <Runs Main analysis with population structure controlled using Sequence Cluster (SC) classifications.>

```
For example, the command to run the pipeline for is the following:  
```
nextflow -C nextflow.config run test.nf --main --sub1 --sub2 --path_nextflow_dir Path/to/your/Nextflow/Directory --path_conda_envs Path/to/your/conda/env/Directory
```
The command to submit the pipeline as a batch job is:  
```
sbatch -p sapphire -c 112 -t 3-00:00 --mem=100000 --wrap="nextflow -C nextflow.config run test.nf --main --sub1 --sub2 --path_nextflow_dir Path/to/your/Nextflow/Directory --path_conda_envs Path/to/your/conda/env/Directory"
```
