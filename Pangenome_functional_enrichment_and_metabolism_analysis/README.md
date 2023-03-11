# Anvi'o Pangenomic, Functional Enrichment, and Metabolism  Analyses
> This is a tutorial demonstrating how to create and visualize pangenomic, metabolic, and functional enrichment data. It uses a small number of genomes of the Candidate phylum Hydrogenedentota as part of the Skoog et al., 2023 study. 

### Step 1: Make a directory containing MAGs (or metagenomes see anvio's tutorials)

```
mkdir MAG_dir_anvio
```

### Step 2: Move MAG files into this new directory 

Here is an example (your path will likely look different):
```
mv *.fa ../MAG_dir_anvio
```
This analysis uses the Hydrogenedentes MAG fasta files found within this parent directory. To follow along, download those and place them within your newly created directory where we will do our Hydrogenedentes MAG analyses. 

### Step 3: Install anvi'o
To install anvi'o, follow instructions [here](https://anvio.org/install/) on anvi'o site. I found this installation to be incredibly smoothe and easy compared to prior versions. 
> Note: Here, I am installing anvi'o version 7.1.

### Step 4: Reformat fasta files (MAGs) 
> Anvi'o requires that fasta file definition lines be simplified

Create your batch file:

```nano anvi-script-reformat-fasta.sh```

Paste the following:

``` #!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=1:00:00                
#SBATCH -J contig_db            
#SBATCH --output=reformat.out  
#SBATCH --error=reformat.err

source activate anvio-7.1

for i in *.fa
do anvi-script-reformat-fasta $i -o fixed_$i -l 1000 --simplify-names
done
```
Run it:

```sbatch anvi-script-reformat-fasta.sh```

### Step 5: Generate contig databases

Create your batch file:

```nano anvi-gen-contigs-database.sh```

Paste the following:
```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=1:00:00                
#SBATCH -J contig_db            
#SBATCH --output=contig_db.out  
#SBATCH --error=contig_db.err

source activate anvio-7.1

for i in fixed*
do anvi-gen-contigs-database -f $i -o $i.db
done
```
Run it:

```sbatch anvi-gen-contigs-database.sh```

### Step 6: Set up KEGG, COG, and HMM database
In your terminal, activate your conda environment:

```conda activate anvio-7.1```

Run the following:
```
anvi-setup-kegg-kofams
```
```
anvi-setup-ncbi-cogs 
```

### Step 7: Annotate MAG databases
> I would budget about an hour per MAG for this step to complete or increase the thread count.


Create your batch file:

```nano anvi-run-kegg-kofams.sh```

Paste the following:
```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=100:00:00                
#SBATCH -J contig_db            
#SBATCH --output=kofams.out  
#SBATCH --error=kofams.err

source activate anvio-7.1

for i in *.db
do
anvi-run-kegg-kofams -c $i --num-threads 2
done
```
Run it:

```sbatch anvi-run-kegg-kofams.sh```

Create your batch file:

```nano anvi-run-ncbi-cogs.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=120G                     
#SBATCH --time=100:00:00                
#SBATCH -J cogs           
#SBATCH --output=cogs.out  
#SBATCH --error=cogs.err

source activate anvio-7.1

for i in *.db
do
anvi-run-ncbi-cogs -c $i --num-threads 4
done
```
Run it:

```sbatch anvi-run-ncbi-cogs.sh```

Create your batch file:

```nano anvi-run-hmms.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=100:00:00                
#SBATCH -J contig_db            
#SBATCH --output=hmms.out  
#SBATCH --error=hmms.err

source activate anvio-7.1

for i in *.db
do
anvi-run-hmms -c $i --num-threads 4
done
```
Run it:

```sbatch anvi-run-hmms.sh```

### Step 8: Create a file with MAG database information

We want to have text file that information conatined in two columns.  The first will be `name` and it will have the name that you want tp associate each of your MAG databases with. The second column will be the name of (or path to if in a different directory) your contig database, so your MAG databases. An example of one of mine is `fixed_SB_biofilm_MAG_10.fa.db`. Your file should look something like this:

![](https://i.imgur.com/slgYnpM.png)


I named this file ```external-genomes.txt``` and used it to generate a genomes storage database with MAG info in the following step:


### Step 9: Generate a genomes storage database

Create your batch file:

```nano anvi-gen-genomes-storage.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 20            
#SBATCH --mem=120G                     
#SBATCH --time=100:00:00                
#SBATCH -J storage            
#SBATCH --output=storage.out  
#SBATCH --error=storage.err

source activate anvio-7.1

anvi-gen-genomes-storage -e external-genomes.txt -o GENOMES.db
```
Run it:

```sbatch anvi-gen-genomes-storage.sh```

### Step 10: Create pangenome

Create your batch file:

```nano anvi-pan-genome.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 20            
#SBATCH --mem=120G                     
#SBATCH --time=100:00:00                
#SBATCH -J pan-genome            
#SBATCH --output=pan-genome.out  
#SBATCH --error=pan-genome.err

source activate anvio-7.1

anvi-pan-genome -g GENOMES.db \
                -n GENOME \
                -o PAN \
                --num-threads 10
```
Run it:

```sbatch anvi-pan-genome.sh```


### Step 11: Determine average nucleotide identity (ANI)

Create your batch file:

```nano anvi-compute-genome-similarity.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=100:00:00                
#SBATCH -J compute-genome-similarity           
#SBATCH --output=compute-genome-similarity.out  
#SBATCH --error=compute-genome-similarity.err

source activate anvio-7.1

anvi-compute-genome-similarity -e external-genomes.txt \
                               --program pyANI \
                               -o ANI \
                               -T 6 \
                               --pan-db PAN/GENOME-PAN.db  
```
Run it:

```sbatch anvi-compute-genome-similarity.sh```


### Step 12: Create another file with additional MAG information

I create a text file called ```info_overlay.txt``` which comtains information for each MAG inclusing taxonomy, completeness, contamination, etc. This file can include whatever information you would like to present in an eventual figure. An example of mine:

![](https://i.imgur.com/UyySQk7.png)

### Step 13: Import this metadata into pangenome database

Create your batch file:

```nano anvi-import-misc-data.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=100:00:00                
#SBATCH -J anvi-import-misc-data         
#SBATCH --output=anvi-import-misc-data.out  
#SBATCH --error=anvi-import-misc-data.err

source activate anvio-7.1

anvi-import-misc-data -p PAN/GENOME-PAN.db \
                      --target-data-table layers \
                      info_overlay.txt 
```
Run it:

```sbatch anvi-import-misc-data.sh```

### Step 14: Compute functional enrichment

Create your batch file:

```nano anvi-compute-functional-enrichment-in-pan.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=100:00:00                
#SBATCH -J compute-genome-similarity           
#SBATCH --output=compute-genome-similarity.out  
#SBATCH --error=compute-genome-similarity.err

source activate anvio-7.1

anvi-compute-functional-enrichment-in-pan -p PAN/GENOME-PAN.db \
                                          -g GENOMES.db \
                                          --category-variable Location \
                                          --annotation-source COG20_PATHWAY \
                                          -o functional-enrichment-txt \
                                          --functional-occurrence-table-output FUNC_OCCURRENCE.TXT
```
Run it:

```sbatch anvi-compute-functional-enrichment-in-pan.sh```


> I later used the resulting `functional-enrichment-txt` text file to generate a figure in R demonstrating functional enrichment of specific (COG) pathways within each sample site (location).

### Step 15: Estimate metabolisms of all MAGs

#### Sub-step 1: Run anvi-estimate-metabolism

Create your batch file:

```nano anvi-estimate-metabolism.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=1:00:00                
#SBATCH -J estimate-metabolism            
#SBATCH --output=estimate-metabolism.out  
#SBATCH --error=estimate-metabolism.err

source activate anvio-7.1

anvi-estimate-metabolism -e external_info.txt \
                         -O GENOMES \
                         --matrix-format

```
Run it:

```sbatch anvi-estimate-metabolism.sh```

#### Sub-step 2: Convert outputted matrix file to newick format

Create your batch file:

```nano anvi-matrix-to-newick.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=1:00:00                
#SBATCH -J matrix-to-newick           
#SBATCH --output=matrix-to-newick.out  
#SBATCH --error=matrix-to-newick.err

source activate anvio-7.1

anvi-matrix-to-newick GENOMES-completeness-MATRIX.txt

```
Run it:

```sbatch anvi-matrix-to-newick.sh```

### Step 16: Generate a metabolisms database

Make sure to include the ```--dry-run``` as we are just generating additional information and not trying to make a figure (at this moment, of course!)

Create your batch file:

```nano anvi-interactive-dry.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=10:00:00                
#SBATCH -J interactive_dry          
#SBATCH --output=interactive_dry.out  
#SBATCH --error=interactive_dry.err

source activate anvio-7.1

anvi-interactive -d GENOMES-completeness-MATRIX.txt \
                 -p GENOMES_metabolism_PROFILE.db \
                 --manual-mode \
                 --dry-run

```
Run it:

```sbatch anvi-interactive-dry.sh```

### Step 17: KEGG modules file
Alrighty folks, I don't know about you, but this step was a pain. Anvi'o spriveds a few simples lines of code [here](https://merenlab.org/tutorials/infant-gut/#some-obligatory-background-on-metabolism-prediction) which show how to retrieve KEGG modules to add for annotation to your metabolisms figure; however, this went super sideways for me as nothing worked. I went through a convoluted series of steps and eventually got the necessary file. SO, I am going to just post my created file here in this preository (named `modules_info.txt`), so you can use it, otherwise feel free to try anvio's method. I hope it works for you!
>Note, there are also different updates to KEGG modules so make sure to read up on it before deciding that my file is best (depending on when you get to this page it could be a bit outdated).

Here is a visual of the file:

![](https://i.imgur.com/8SvAWxU.png)


### Step 18: Generate a metabolisms database

Make sure to include the ```--dry-run``` as we are just generating additional information and not trying to make a figure (at this moment, of course!)

Create your batch file:

```nano anvi-import-misc-data2.sh```

Paste the following:

```#!/bin/bash
#!/bin/bash
#SBATCH -p sched_mit_g4nier 
#SBATCH -N 1                          
#SBATCH -n 1            
#SBATCH --mem=10G                     
#SBATCH --time=10:00:00                
#SBATCH -J import-misc-data2         
#SBATCH --output=import-misc-data2.out  
#SBATCH --error=import-misc-data2.err

source activate anvio-7.1

anvi-import-misc-data modules_info.txt \
                      -p GENOMES_metabolism_PROFILE.db \
                      -t items

```
Run it:

```sbatch anvi-import-misc-data2.sh```

### Step 19: Visualize your pangenome, functinoal enrichment, and estimated metabolism figures

Chances are, you are running this analysis on an HPC because of the beast of a program and computational power that this program provides. I have run everything on an HPC but have also installed anvio on my locally (on my computer) to make the visualizatino process easier, as anvi'o pulls up a web browser for visualization. Some people choose to visualize their data by setting up an ssh tunnel. Personally, I moved over my necessary files and ran the interactive interface from my local terminal window. 

#### Sub-step 1: Move over all needed files from HPC to computer 
OR if you don't want to install anvi'o locally (on your computer) or don't want to deal with moving over all these files to your local computer, you can simply use SSH tunneling. Follow these steps [here](https://github.com/emilieskoog/SSH-tunneling/blob/main/SSH%20tunneling%20(specific%20example%20for%20anvi%E2%80%99o).md).

#### Sub-step 2: Go into this directory on your computer with your files and activate your anvio

``` conda activate anvio-7.1```

(Obviously make sure you installed anvio locally on your computer before running this)

#### Sub-step 3: Run anvi-interactive

Pangenome analysis
```
anvi-display-pan -g GENOMES.db \
                 -p PAN/GENOME-PAN.db \
                 --title "Hydrogenedentes Pangenome"
 ```                
![](https://i.imgur.com/AtxYxlq.jpg)

Metabolism analysis:
```
anvi-interactive --manual-mode \
                 -d GENOMES-completeness-MATRIX.txt \
                 -t GENOMES-completeness-MATRIX.txt.newick \
                 -p GENOMES_metabolism_PROFILE.db \
                 --title "Hydrogenedentes Metabolism Map"
```                
![](https://i.imgur.com/uUTMZxO.png)




I fyou are working on an HPC, you can refer to [this SSH tunneling tutorial](https://github.com/emilieskoog/SSH-tunneling) (specific for anvio) to avoid moving over these files to your local computer and having to install anvio locally. 
