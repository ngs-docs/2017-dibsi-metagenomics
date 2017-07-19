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

After binning, a few simple annotation strategies are demonstrated.

Kraken is a system for assigning taxonomic labels to short DNA sequences, usually obtained through metagenomic studies. Kraken aims to achieve high sensitivity and high speed by utilizing exact alignments of k-mers and a novel classification algorithm.  See `Kraken Home Page <https://ccb.jhu.edu/software/kraken/>`__ for more information.

Prodigal (Prokaryotic Dynamic Programming Genefinding Algorithm) is a microbial (bacterial and archaeal) gene finding program developed at Oak Ridge National Laboratory and the University of Tennessee. See the `Prodigal home page <http://prodigal.ornl.gov>`__ for more info.
`Citation <http://denbi-metagenomics-workshop.readthedocs.io/en/latest/geneprediction/index.html>`__



1.  Preparing to bin
===============================================


So far, you have learned about how to assemble sequences yesterday and assemble some contigs, for today's tutorial, we need a full assembled file. Let's also get some assembled and mapped data.

::
	cd ~    
	mkdir binning && cd ~/binning    
	wget https://s3.amazonaws.com/edamame/SRR492066.sam.bam.sorted.bam    
	wget https://s3.amazonaws.com/edamame/SRR492065.sam.bam.sorted.bam    
	wget https://s3.amazonaws.com/edamame/final.contigs.fa
	
 
We need to grab some software from public repositories.
 
Install Metabat::
 
 	cd ~
 	wget https://bitbucket.org/berkeleylab/metabat/downloads/metabat-static-binary-linux-x64_v2.11.1.tar.gz
	tar xzvf metabat-static-binary-linux-x64_v2.11.1.tar.gz
	
Install Kraken::

	cd ~
	git clone https://github.com/DerrickWood/kraken.git
	cd ~/kraken
	mkdir ~/kraken/bin
	./install_kraken.sh ~/kraken/bin

Install Kraken Mini DB::

	mkdir ~/KRAKEN
	cd ~/KRAKEN
	wget http://ccb.jhu.edu/software/kraken/dl/minikraken.tgz
	tar -xvf minikraken.tgz

Install Prodigal::

	cd ~
	wget https://github.com/hyattpd/Prodigal/releases/download/v2.6.3/prodigal.linux
	tar -xvf v2.6.3.tar.gz
	chmod 775 ~/prodigal.linux
	


2. Now let's do some binning!
===============================================

   ::

       cd ~/binning

       nano binscript.sh

Copy and paste the following block into the script::

	for i in ~/binning/*.sorted.bam
        do
           newfile=$(basename $i *.sam.bam.sorted.bam)
           mkdir ~/binning/${newfile}.bins
           cd ~/binning/${newfile}.bins
           ~/metabat/runMetaBat.sh ~/binning/final.contigs.fa $i
        done

Control-X to exit

Make the script executable.

::

	chmod 775 binscript.sh 

Run the script.

::

	./binscript.sh

Go into one of the resulting bins and head some of the files:

	cd ~/binning/SRR492065.sam.bam.sorted.bam.bins/final.contigs.fa.metabat-bins
	head *.fa

Question:  What are the headers referring to?
Question:  What's next with these bins?


3. What if we want to bin the contigs with different threshold?
================================================================

::

       cd ~/binning/SRR492065.sam.bam.sorted.bam.bins
       
       #First, try sensitive mode to better sensitivity
       
       ~/metabat/metabat2 -i ~/binning/final.contigs.fa -a final.contigs.fa.depth.txt -o bin1 --sensitive -l -v --saveTNF saved.tnf --saveDistance saved.gprob

       #Try specific mode to improve specificity further; this time the binning will be much faster since it reuses saved calculations
       
       ~/metabat/metabat2 -i ~/binning/final.contigs.fa -a final.contigs.fa.depth.txt -o bin2 --specific -l -v --saveTNF saved.tnf --saveDistance saved.gprob

       #Try specific mode with paired data to improve sensitivity while minimizing the loss of specificity
       
       ~/metabat/metabat2 -i ~/binning/final.contigs.fa -p final.contigs.fa.paired.txt -o bin3 --specific -l -v --saveTNF saved.tnf --saveDistance saved.gprob


While MetaBat works with default parameters, it is possible to tune some of the parameters to attempt to create more complete genomes and reduce contamination.  A full tutorial can be found at the `MetaBat website <https://bitbucket.org/berkeleylab/metabat/wiki/Best%20Binning%20Practices>`__



4.  Simple annotation strategies - kraken
===============================================

If you have a simple data set with very common bacteria, you can jump right into kraken for annotation here.

::

	cd ~/binning/SRR492065.sam.bam.sorted.bam.bins/

	~/kraken/bin/kraken --db ~/KRAKEN/minikraken_20141208/ --threads 2 --fasta-input final.contigs.fa.metabat-bins/bin.1.fa --output bin1.kraken	
	
	~/kraken/bin/kraken-translate --db ~/KRAKEN/minikraken_20141208/ bin1.kraken > bin1.kraken.labels

Kraken has now provided a taxonomic assignment to all of the clusters.

Why use Kraken?

For a simulated metagenome of 100 bp reads in its fastest mode of operation, , Kraken processed over 4 million reads per minute on a single core, over 900 times faster than Megablast and over 11 times faster than the abundance estimation program MetaPhlAn. Kraken's accuracy is comparable with Megablast, with slightly lower sensitivity and very high precision.`Citation <http://denbi-metagenomics-workshop.readthedocs.io/en/latest/classification/kraken.html>`__

However, kraken is only as sensitive as the provided database, so for unusual samples, a custom database needs to be constructed . The accuracy is very sensitive to the quantity of samples in the database.



5. Functional annotation strategies - prodigal
===============================================

Using prodigal with the same set of data, we can get a list of predicted genes.

::

	cd ~/binning/SRR492065.sam.bam.sorted.bam.bins/
	~/prodigal.linux -p meta -a final.contigs.genes.bin1.faa -d final.contigs.genes.bin1.fna -f gff -o final.contigs.genes.bin1.gff -i final.contigs.fa.metabat-bins/bin.1.fa


--------------

Adapted by Adelaide Rhodes, Ph.D. for Environmental Metagenomics 2017 UC Davis DIGBSI

Authored by Fan Yang  `EDAMAME-2016
wiki <https://github.com/edamame-course/2016-tutorials/wiki>`__

--------------

EDAMAME tutorials have a CC-BY
`license <https://github.com/edamame-course/2015-tutorials/blob/master/LICENSE.md>`__.
*Share, adapt, and attribute please!* \*\*\*
