I'm going to try and test the program with some capture data hosted on a cluster. 

The bait sequences I am attempting to find are targets for which I know probes were designed. However, I don't have a good reference for that data across all the targets. Instead I'm going to try and recreate those targets based on the output of a mysterious blackbox pipeline that looks for and assembles the target data in reads for every taxon and use the longest sequence to search through all of my own assemblies. 

Beacause these files can contain unrelated sequences, I opt to use only one sequence as the bait file rather than actual unrelated sequences. 

These files can also unfortunately contain a lot of strange ambiguities that aren't taken into account with BLAST searches so I may try an iterative process with better more numerous bait files after the intial round. 

I have 516 separate loci with 97 different taxa being used as bait sequence. The longest is 8353 long and most contain stretches of ambiguities so overall a pretty weird case.

My bait files are here: ~/regrouped/regrouped/single

My target files and associated BLAST databases are in various places at the moment: 
```
/home/CAM/aporczak/AHEloci/MBlastDB/
/home/CAM/mstukel/AHE/Kikihia/blastdb/
~/Dropbox/assemblies/*trimmedspades.assembly/contigs.fasta
/home/CAM/dhaji/AHE/Individuals_in_TaxonSets/*/spades.assembly/contigs.fasta
 ~/metagenomes/trimmedassemblies/*trimmedspades.assembly/contigs.fasta
```
I have several other assemblies I would eventually want to include but for now I will stick with just the first set.

Last, I need to get a list of the full set of taxa included to search for since I am looking through multiple assemblies which I will call "LIST.txt" and place in the folder where I have all my baits. To start I will test with 2 threads running on the cluster.   

```
bash ~/alibaseq/blast_wrapper.sh . /home/CAM/aporczak/AHEloci/MBlastDB/ 1e-05 tblastx 2 y ./LIST
```

Seems to be working so I will submit a job called blast.sh
```
#!/bin/bash
#SBATCH --job-name=blast
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 20
#SBATCH --partition=general
#SBATCH --qos=general
#SBATCH --mail-type=END
#SBATCH --mem=150G
#SBATCH --mail-user=eric.gordon@uconn.edu
#SBATCH -o myscript_%j.out
#SBATCH -e myscript_%j.err 
cd ~/regrouped/regrouped/single/
bash ~/alibaseq/blast_wrapper.sh . /home/CAM/aporczak/AHEloci/MBlastDB/ 1e-05 tblastx 20 y ./LIST
```
```
sbatch blast.sh 
```
