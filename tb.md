### Finding Mycobacterium tuberlusis sequences

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

5. Once Parsnp is done running, it will produce numerous output files including a .tree file. I visualized this tree file in FigTree(v1.4.4). Once visualized, I picked 10 (1 being the ref seq) of the most relatively divergent tb sequences to align and feed into PrimalScheme.
