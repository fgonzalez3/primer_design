### Finding Mycobacterium tuberculosis sequences

1. First, I downloaded tb sequences and the .csv file containing sequence info from Genbank. Here, I selected for a) complete b) refseqs from c) 2020-2023. This yielded 158 sequences. 

### Before uploading sequences to Ruddle

1. Before uploading tb genomes to Ruddle for future analyses, we have to concatenate all the sequence files into one. 

2. Navigate to your computer's working directory that contains all your .fasta files. Mine is as folllows:  

```
cd Users/flg9/Desktop/Developer/grubaugh_lab/primer_design/tb/seqs/genbank/data
```
3. Then copy all the fasta files into another folder (e.g. cat_seqs), where we will concatenate. 

```
cp */*.fna cat_seqs
```

4. Finally, concatenate within the subfolder as follows: 

```
cd cat_seqs
cat *.fna > tb_seqs.fasta
```

5. Finally, upload this file to Ruddle for future analyses. 

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
c) tb genomes (I created an additional directory to upload these, 'genomes')
*Note on this last part: Parsnp parses through each individual .fasta file, so I did not use the concatenated file here. 

4. Run the following script, or one similar. 

```
#!/bin/bash
#SBATCH --job-name=parsnp_tb
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --time=72:00:00
#SBATCH --output=parsnp.out
#SBATCH --error=parsnp.err

module load miniconda
conda activate primer_design

parsnp -r ref_seq.fasta -d ./genomes/*.fna -c
```

5. Once Parsnp is done running, it will produce numerous output files including a .tree file. I visualized this tree file in FigTree(v1.4.4).
