Metagenome binning using MetaBAT
================================


Overarching Goal
----------------

-  This tutorial will contribute towards an understanding of **microbial
   metagenome analysis**

Learning Objectives
-------------------

-  Understand metagenomic binning and perform a binning exercise

Tutorial
--------

We will be running a specific tool for metagenomic binning called
MetaBAT. You can read about this tool and its applications in the
original paper, `[MetaBAT, an efficient tool for accurately
reconstructing single genomes from complex microbial communities]
<https://peerj.com/articles/1165/>`__

This tutorial takes place assuming you have an assembled metagenome and
its mapped read abundances.

1.  Preparing to bin
===============================================


So you learned about how to assemble sequences yesterday and assembled some contigs, for today's tutorial, we need a full assembled file. Let's also get some assembled and mapped data.

::

  cd ~    
	mkdir data && cd ~/data    
	wget https://s3.amazonaws.com/edamame/SRR492066.sam.bam.sorted.bam    
	wget https://s3.amazonaws.com/edamame/SRR492065.sam.bam.sorted.bam    
	wget https://s3.amazonaws.com/edamame/final.contigs.fa``
 

Q: Can you think of a reason why we couldn't use the partial
   assembled contigs from the metagenome assembly exercise?


2. Now let's do some binning!
===============================================

   ::

       cd ~/data

       nano binscript.sh

Copy and paste the following block into the script

::

       #!/bin/bash

       for i in *.sorted.bam 
       do 
           mkdir ~/data/$i.bins 
           cd ~/data/$i.bins
           ~/metabat/bin/runMetaBat.sh ~/data/final.contigs.fa ~/data/$i
       done 

Control-X to exit

::

	chmod 775 binscript.sh 

	binscript.sh



3. What if we want to bin the contigs with different threshold?
================================================================

::

       cd ~/data/SRR492065.sam.bam.sorted.bam.bins
       #First, try sensitive mode to better sensitivity
       ~/metabat/bin/metabat -i ~/data/final.contigs.fa -a final.contigs.fa.depth.txt -o bin1 --sensitive -l -v --saveTNF saved.tnf --saveDistance saved.gprob

       #Try specific mode to improve specificity further; this time the binning will be much faster since it reuses saved calculations
       ~/metabat/bin/metabat -i ~/data/final.contigs.fa -a final.contigs.fa.depth.txt -o bin2 --specific -l -v --saveTNF saved.tnf --saveDistance saved.gprob

       #Try specific mode with paired data to improve sensitivity while minimizing the loss of specificity
       ~/metabat/bin/metabat -i ~/data/final.contigs.fa -p final.contigs.fa.paired.txt -o bin3 --specific -l -v --saveTNF saved.tnf --saveDistance saved.gprob


4.  Simple annotation strategies - kraken
===============================================

If you have a simple data set with very common bacteria, you can jump right into kraken for annotation here.

::

	cd ~/data/cd SRR492065.sam.bam.sorted.bam.bins/

	/local/cluster/kraken/kraken-0.10.5-beta/kraken --db /local/cluster/kraken/krakenfullDB_20151215/KRAKEN/ --threads8 --fasta-input final.contigs.fa.metabat-bins-.1.fa --output bin1.kraken


	/local/cluster/kraken/kraken-0.10.5-beta/kraken-translate --db /local/cluster/kraken/krakenfullDB_20151215/KRAKEN/ bin1.kraken > bin1.kraken.labels


kraken has now provided a taxonomic assignment to all of the clusters.

â€œKraken is a system for assigning taxonomic labels to short DNA sequences, usually obtained through metagenomic studies. Kraken aims to achieve high sensitivity and high speed by utilizing exact alignments of k-mers and a novel classification algorithm.

For a simulated metagenome of 100 bp reads in its fastest mode of operation, , Kraken processed over 4 million reads per minute on a single core, over 900 times faster than Megablast and over 11 times faster than the abundance estimation program MetaPhlAn. Krakenâ€™s accuracy is comparable with Megablast, with slightly lower sensitivity and very high precision.â€ `Citation <http://denbi-metagenomics-workshop.readthedocs.io/en/latest/classification/kraken.html>`__

See `Kraken Home Page <https://ccb.jhu.edu/software/kraken/>`__ for more information.

5. Functional annotation strategies - prodigal
===============================================

Prodigal (Prokaryotic Dynamic Programming Genefinding Algorithm) is a microbial (bacterial and archaeal) gene finding program developed at Oak Ridge National Laboratory and the University of Tennessee. See the `Prodigal home page <http://prodigal.ornl.gov>`__ for more info.
`Citation <http://denbi-metagenomics-workshop.readthedocs.io/en/latest/geneprediction/index.html>`__

Using prodigal with the same set of data, we can get a list of predicted genes.

::

	prodigal -p meta -a final.contigs.genes.bin1.faa -d final.contigs.genes.bin1.fna -f gff -o final.contigs.genes.bin1.gff -i final.contigs.fa.metabat-bins-.1.fa




Adapted by Adelaide Rhodes, Ph.D. for Environmental Metagenomics 2017 at UC Davis

`EDAMAME-2016
wiki <https://github.com/edamame-course/2016-tutorials/wiki>`__

--------------

EDAMAME tutorials have a CC-BY
`license <https://github.com/edamame-course/2015-tutorials/blob/master/LICENSE.md>`__.
*Share, adapt, and attribute please!* \*\*\*

