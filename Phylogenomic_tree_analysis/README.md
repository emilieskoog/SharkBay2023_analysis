# Anvi'o Phylogenomic Analysis
#### Here you can find a walkthrough of how I used anvio to construct phylogenomic trees for Candidate phylum Hydrogenedentes MAGs.

>This analysis follows the phylogenomic tutorial created by the anvi'o team which can be found [here](https://merenlab.org/2017/06/07/phylogenomics/). For simplicity, I have also narrowed down the number of MAGs used in this demonstration. These can be found in the Hydrogenedentes fasta file repository. 
## Step 1: Download the latest version of anvio
> I am running anvio version 7.1 in this analysis.

To do this, follow [these](https://anvio.org/install/) instructions.


## Step 2: Import all files into your directory

## Step 3: Activate your anvio conda environment (if using conda)
```
conda activate anvio-7.1
```
> On some HPCs you may need to use `source activate anvio-7.1`
## Step 4: Run `anvi-script-reformat-fasta`
> Before working with these MAGs, the def lines need to be simplified


```
for i in *.fa
do anvi-script-reformat-fasta $i -o fixed_$i -l 1000 --simplify-names
done
```
## Step 5: Run `anvi-gen-contigs-database`
>You will need to generate your contigs database for each fasta file:

```
for i in `ls fixed* | awk 'BEGIN{FS=".fa"}{print $1}'`
do
    anvi-gen-contigs-database -f $i.fa -o $i.db -T 4
    anvi-run-hmms -c $i.db
done
```
## Step 6: Run `anvi-get-sequences-for-hmm-hits`

to either place them phylogenomically based on ribosomal genes:
```
anvi-get-sequences-for-hmm-hits --external-genomes external-genomes.txt \
                                -o ribosomal_concatenated-proteins.fa \
                                --hmm-source Bacteria_71 \
                                --gene-names Ribosomal_L1,Ribosomal_L2,Ribosomal_L3,Ribosomal_L4,Ribosomal_L5,Ribosomal_L6 \
                                --return-best-hit \
                                --get-aa-sequences \
                                --concatenate
```

or all genes:

```
anvi-get-sequences-for-hmm-hits --external-genomes external-genomes.txt \
                                -o all_genes_concatenated-proteins.fa \
                                --hmm-source Bacteria_71 \
                                --return-best-hit \
                                --get-aa-sequences \
                                --concatenate
```

## Step 7: Run `anvi-gen-phylogenomic-tree`

Generate files to construct your phylogenomic tree for your tree incorporating only ribosomal genes:

```
anvi-gen-phylogenomic-tree -f ribosomal_concatenated-proteins.fa \
                           -o ribosomal_phylogenomic-tree.txt
```

or for your tree incorporating all bacterial genes:

```
anvi-gen-phylogenomic-tree -f all_genes_concatenated-proteins.fa \
                           -o all_genes_phylogenomic-tree.txt
```

## Step 8: Visulaize your trees

Run the following depending on which tree you are building:

```
anvi-interactive -p ribosomal_phylogenomic-profile.db \
                 -d info_overlay.txt \
                 -t ribosomal_phylogenomic-tree.txt \
                 --title "Hydrogenedentes phylogenomic tree based on concatenated ribosomal proteins " \
                 --manual

```
or
```
anvi-interactive -p all_genes_phylogenomic-profile.db \
                 -d info_overlay.txt \
                 -t all_genes_phylogenomic-tree.txt \
                 --title "Hydrogenedentes phylogenomic tree based on all bacterial proteins " \
                 --manual                 

```
I fyou are working on an HPC, you can refer to [this SSH tunneling tutorial](https://github.com/emilieskoog/SSH-tunneling) (specific for anvio) to avoid moving over these files to your local computer and having to install anvio locally. 
