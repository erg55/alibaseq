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
#SBATCH -c 12
#SBATCH --partition=general
#SBATCH --qos=general
#SBATCH --mail-type=END
#SBATCH --mem=75G
#SBATCH --mail-user=eric.gordon@uconn.edu
#SBATCH -o myscript_%j.out
#SBATCH -e myscript_%j.err 
cd ~/regrouped/regrouped/single/
bash ~/alibaseq/blast_wrapper.sh . /home/CAM/aporczak/AHEloci/MBlastDB/ 1e-05 tblastx 12 y ./LIST
```
```
sbatch blast.sh 
```

That took 3 hours 20 minutes for 43 sequence capture assemblies. 

Trying with just the tutorial command at first which extracts region between two outmost hit regions in max of one supercontig with a very permissive evalue of 1e-05 and with contig stitching allowed

```
python ~/alibaseq/alibaseq.py -x b -f M -b ./blast_results/ -c 1 -e 1e-05 --is --ac tdna-tdna -t /home/CAM/aporczak/AHEloci/MBlastDB/
```

It took half an hour....In general i now have alignments with lots of N's.

In general, the alignments look good but in some I can see some signs of bits of paralogs slipping in some of the sequences. Almost all loci have all 43 taxa represented but a significant amount have very sequences which perhaps had particularly bad bait sequences. 

For the first iteration I probably shouldn't do any contig stitching since the initial bait sequences are so odd. I'm also going to up the evalue to something a bit less permissive. I will also use the -q paramater option so can I append the results to the intial baits and see more obviously how that sequence looks in the alignment and whether that contributed to any of the problems. Also i will use the -o option so I can save the previous results

```
python ~/alibaseq/alibaseq.py -x b -f M -b ./blast_results/ -c 1 -e 1e-15  --ac tdna-tdna -t /home/CAM/aporczak/AHEloci/MBlastDB/ -q ~/regrouped/regrouped/single/ -o ./nostitching
```

This took 9 minutes. I still see some evidence of paralogs.....I think the first iteration should probably be extremely stringent so I will repeat with 1e-125 hoping to get at least one relaible in group sequence for all the loci which...might not occur. 

```
python ~/alibaseq/alibaseq.py -x b -f M -b ./blast_results/ -c 1 -e 1e-125  --ac tdna-tdna -t /home/CAM/aporczak/AHEloci/MBlastDB/ -q ~/regrouped/regrouped/single/ -o ./nostitchinghighcutoff
```


This stringency greatly improved my alignments....with longer sequences being pulled out for most taxa so perhaps less than perfect sequences were pulled out for some reason? Looking at one example for I27929 there are only two contigs with a higher evalue than 1e-25 and many with a higher evalue than 1e-15 

NODE_80_length_4862_cov_16.719122 and NODE_132_length_3988_cov_6.517003

Neither of those are chosen instead it is NODE_1731_length_1093_cov_65.862205...it has the highest identity? of 96? 

./T133_L100.fas	NODE_1731_length_1093_cov_65.862205	96.923	65	2	0	1231	1425	614	420	6.07e-38	155

Oddly that sequence is shorter than the output of that sequence?

It's actually choosing NODE_80_length_4862_cov_16.719122 but output doesn't show that? Also Node 80 has a lower bit score and evalue than the aternative but not summed up over entire length I guess? 

Anyways this is giving us better results. But there are many loci with nothing passing through the filter unfortunately. Let's adjust the evalue to be slightly less stringent

```
python ~/alibaseq/alibaseq.py -x b -f M -b ./blast_results/ -c 1 -e 1e-75 --ac tdna-tdna -t /home/CAM/aporczak/AHEloci/MBlastDB/ -q ~/regrouped/regrouped/single/ -o ./nostitchinglowercutoff
```


Hmm ok that was a problem with how i was looking at versions....but I will try a low evalue with the --amalgamate-hits with the --metric-merge-corr option. Also elimating the -q option for now to prevent confusion in the log files. 
