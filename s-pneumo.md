### Finding S.pneumo sequences 

1. Download sequences and .csv file containing sequence info from Global Pneumococcal Sequencing (GPS) database containing metadata and assemblies of serotype 3 genomes that could be used to generate an inital alignment for PrimalScheme. Genome ranges are from ~1.7-2.7mb. These sequences will download as contigs, so this will have to be accounted for in Snippy. 

### Before uploading sequences to Ruddle 

1. Before uploading serotype 3 genomes to Ruddle for Snippy runs, we have to concatenate all the contig files into one. Since there were so many serotype 3 genomes downloaded from GPS, I got 5 zip files. I consequently split these up into 5 contig files that will later be aligned into a snps core alignment. 

2. Navigate to your computer's working directory that contains all your zip files. Mine is as folllows:  

```
cd Users/flg9/Desktop/Developer/grubaugh_lab/primer_design/snippy/seqs/contigs
```
3. Then copy all the fasta files into another folder (e.g. snippy_1), where we will concatenate. This isn't the most efficient code, so you'll have to run for each zip file as follows: 

```
cp zip_1/*/*.fa zip_1/snippy_1/
```

4. Finally, concatenate within the subfolder as follows: 

```
cat zip_1/snippy_1/*.fa > seqs_1.fasta
```

5. You can now upload seqs_1-5.fasta to Ruddle to analyze with Snippy. 


### Setting up a Conda environment

The easiest way to run Snippy is to use Yale's computer cluster, Ruddle. There are a few prior steps to complete before being able to run Snippy, listed below. Using my example, I navigated to my desired working directory and set up a conda environment containing Snippy and PrimalScheme. 

1. Ask for space off the login node 

```
salloc
```

2. Load the Miniconda module (use 'ml' or 'module load')

```
ml miniconda
```

3. Use Mamba to create an environment (this is where you will include all packages that you'll need for any future analyses as well). 

```
mamba create -yn primer_design primalscheme snippy
```

4. Once built, end session and free resouces 

```
exit
```

5. Activate your environment

```
conda activate primer_design
```

6. From there you should be able to use 'conda install PACKAGENAME' inside the activated environment to install other packages in the environment if need be. If an error occurs at this step, delete the original environment and create a new one reflecting the packages you wish to use. To delete environment:

```
conda env remove -yn ENVNAME
```

### Running Primal Scheme on your Reference Sequence

1. Still working within your Conda environment and having navigated to your desired working directory, we next run [PrimalScheme](https://primalscheme.com/). 


2. First, test PrimalScheme with your reference genome to see if it will crash or not. We are building 2000bp primers with preference toward high gc content for higher melting points. 

```
#!/bin/bash
#SBATCH --job-name=primalscheme_refseq
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1024
#SBATCH --time=48:00:00
#SBATCH --output=primalscheme.out
#SBATCH --error=primalscheme.err

module load miniconda
conda activate primer_design

primalscheme multiplex ref_seq.fasta -a 2000 --high-gc
```

3. PrimalScheme will yield numerous output files. Download your .bed file and upload to Geneious to make sure that primers align with your ref seq, or sequence alignments (not shown in this step). 


### Using Snippy

1. Once your environment is activated and you are within your desired working directory on the cluster, you can run Snippy. 

2. Conda will not follow you into your bash script, so you will have to call upon it here. Since the sequences pulled from GPS are contigs (not reads), flag the concatenated .fasta file containing the S. pneumo sequences you want to align to your reference sequence with '--contigs'.

```
#!/bin/bash
#SBATCH --job-name=snippy_1
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1024
#SBATCH --time=48:00:00
#SBATCH --output=snippy_1.out
#SBATCH --error=snippy_1.err

module load miniconda
conda activate primer_design

snippy --outdir mysnps_1 --ref ref_seq.fasta --contigs seqs_1.fasta  
<cut>
```

3. This shouldn't take long to run, so you'll get output files briefly. First, check that all output directories contain the necessary files.

```
ls mysnps_*
```

4. Then, make a core SNP alignment. No need to create a bash script for this.. as long as your environment is activated, you can run directly. 

```
snippy-core --ref ref_seq.fasta --prefix core mysnps_1/ mysnps_2/ ...
```

5. Check to make sure the *.core files are present. Then run the below code (again, no need to run a bash script as long as your environment is activated): 

```
$ snippy-clean_full_aln core.full.aln > clean.full.aln
$run_gubbins.py -p gubbins clean.full.aln
% snp-sites -c gubbins.filtered_polymorphic_sites.fasta > clean.core.aln
% FastTree -gtr -nt clean.core.aln > clean.core.tree
```
### Running PrimalScheme on Snippy outputs 

1. Running my core genomes on PrimalScheme gave me the following error:

```
Writing log to output/scheme.log
Error: One or more of your references is too different in size to the primary (first) reference. The maximum difference is 500 nt.
```

2. Instead, we decided to run PrimalScheme on the consensus genomes originally outputted by Snippy and found within mysnps_1, ..., mysnps_5. Collecting each of these consensus files, I concatenated them to make a "meta consensus". Comprised of 490 S. Pneumo sequences and our refseq, this was our input for PrimalScheme:

```
#!/bin/bash
#SBATCH --job-name=primalscheme_consensus
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1024
#SBATCH --time=48:00:00
#SBATCH --output=primalscheme.out
#SBATCH --error=primalscheme.err

module load miniconda
conda activate primer_design

primalscheme multiplex sp_consensus.fa -a 2000 --high-gc --force
```

### Using Parsnp

Parsnp is a command-line-tool for efficient microbial core genome alignment and SNP detection. Parsnp was designed to work in tandem with Gingr, a flexible platform for visualizing genome alignments and phylogenetic trees; both Parsnp and Gingr form part of the Harvest suite. 

1. First, log onto your cluster (i.e. Ruddle) and create a directory to run Parsnp analysis. 

2. Following the Parsnp Github download instructions, I downloaded Parsnp onto my existing Yale Ruddle conda environment. 

```
(primer_design) [flg9@ruddle1 parsnp]$ conda install parnsp
```

3. Afterwards, upload the following:
a) reference sequence
b) slurm script 
c) s. pneumo genomes (I created an additional directory to upload these, 'genomes', for neatness)

4. Run the following script, or one similar. 

```
#!/bin/bash
#SBATCH --job-name=parsnp_psneumo
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --time=72:00:00
#SBATCH --output=parsnp.out
#SBATCH --error=parsnp.err

module load miniconda
conda activate primer_design

parsnp -r ref_seq.fasta -d ./genomes/*fa -c
```

5. Once Parsnp is done running, it will produce numerous output files including a .tree file. I visualized this tree file in FigTree(v1.4.4). Once visualized, I picked 10 (1 being the ref seq) of the most relatively divergent S. pneumo serotype 3 sequences to align and feed into PrimalScheme. 


