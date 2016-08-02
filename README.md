# CAW
Nextflow Cancer Analysis Workflow Prototype developed at @SciLifeLab

## Version
0.0.2

## Authors
- Pelin Akan (@pelinakan)
- Jesper Eisfeldt (@J35P312)
- Maxime Garcia (@MaxUlysse)
- Szilveszter Juhos (@szilvajuhos)
- Max Käller
- Malin Larsson (@malinlarsson)
- Björn Nystedt (@bjornnystedt)
- Pall Olason (@pallolason)

## Quick start
Install [nextflow](http://www.nextflow.io/) and then download the project
```bash
git clone https://github.com/SciLifeLab/CAW.git
```
Then it's possible to run the full workflow on a data set specified in the tsv file. For example:
```bash
nextflow run MultiFQtoVC.nf -c milou.config --sample sample.tsv
```
will run the workflow on a small testing dataset. To try on your own data, you just have to edit your own tsv file.

## Usage
```bash
nextflow run MultiFQtoVC.nf -c <file.config> --sample <sample.tsv>
```
All variables and parameters are specified in the config and the sample files.

## Nextflow processes
Several processes are run within Nextflow
We divide them for the moment into 2 main steps

## Troubleshooting on milou
On UPPMAX's milou, there is a small problem with slurm when loading some modules. `VarDictJava/1.4.5` for example `needs java/sun_jdk1.8.0_92`. But when you load `Nextflow/0.17.3`, milou loads automatically `java/sun_jdk1.7.0_25`.
So my advice when running this script is to do first:
```bash
module load bioinfo-tools Nextflow
module unload java
module load java/sun_jdk1.8.0_92
```

### Preprocessing:
- Mapping - Mapping reads with BWA
- MergeBam - Merging BAMs if multilane samples
- RenameSingleBam - Renaming BAM if non-multilane sample
- MarkDuplicates - Using Picard
- CreateIntervals - Using GATK
- Realign - Using GATK
- CreateRecalibrationTable - Using GATK
- RecalibrateBam - Using GATK

### Variant Calling:
- RunMutect2 - Running MuTect2 shipped in GATK v3.6
- VarDict - running VarDict on multiple intervals
- VarDictCollatedVCF - merging Vardict result

## TSV file for sample
It's a Tab Separated Value file, based on: subject status sample lane fastq1 fastq2
Quite straight-forward: 
- Subject is the ID of the Patient
- Status is 0 for Normal and 1 for Tumor
- Sample is the Sample ID (It is possible to have more than one tumor sample for each patient)
- Lane is used when the sample is multiplexed on several lanes
- fastq1 is the path to the first pair of the fastq file
- fastq2 is the path to the second pair of the fastq file

## Configuration file
We're developing this workflow on UPPMAX cluster milou. So our config file is based on the config file from [NGI RNAseq Nextflow pipeline](https://github.com/SciLifeLab/NGI-RNAseq)

## Test data
For quick, easy testing of the workflow, we need a small, but representative
data set as input. We went for a full depth coverage, small chunk of the genome. "Seba" suggested using [TCGA benchmarking data][TCGA], which is already downloaded to Uppmax.

Grabbing 100kb from chr1, both tumor and normal, sorting by read name:
```
$ samtools view -bh /sw/data/uppnex/ToolBox/TCGAbenchmark/f0eaa94b-f622-49b9-8eac-e4eac6762598/G15511.HCC1143_BL.1.bam 1:100000-200000 | samtools sort -n -m 4G - tcga.cl.normal.part.nsort.bam
$ samtools view -bh /sw/data/uppnex/ToolBox/TCGAbenchmark/ad3d4757-f358-40a3-9d92-742463a95e88/G15511.HCC1143.1.bam 1:100000-200000 | samtools sort -n -m 4G - tcga.cl.tumor.part.nsort.bam
```

Then convert to fastq:
```
$ bedtools bamtofastq -i tcga.cl.normal.part.nsort.bam -fq tcga.cl.normal.part.nsort_R1.fastq -fq2 tcga.cl.normal.part.nsort_R2.fastq
$ bedtools bamtofastq -i tcga.cl.tumor.part.nsort.bam -fq tcga.cl.tumor.part.nsort_R1.fastq -fq2 tcga.cl.tumor.part.nsort_R2.fastq
```
And finally gzipping...


[TCGA]: https://cghub.ucsc.edu/datasets/benchmark_download.html
