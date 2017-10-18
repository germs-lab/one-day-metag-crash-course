Running a bioinformatic program
================================


Learning objectives
-------------------

This is a tutorial for running a BLAST alignment for three metagenomes against a reference gene set. The learning objectives for this tutorial are as follows:

1.  To be able to run a specific bioinformatic program, BLAST.
2.  To format a reference gene database for BLAST.
3.  To estimate the number of genes and their corresponding annotations in multiple sequencing datasets.
4.  To run a for loop in the shell.

You will need to know some things prior to this tutorial:

1.  Ability to navigate in the unix shell.

Install BLAST - Students working on a shared computer can likely skip this step
-------------------------------------------------------------------------------

If BLAST is not installed, follow these steps.

Navigate to any directory and execute the following::

    sudo apt-get install ncbi-blast+
    
Now BLAST is installed locally on your machine.  You will now need collect/download files to run BLAST on!    

Running BLAST
-------------

First! We need some data.  In the last tutorial, we learned how to get data for a reference genome from an API, specifically NCBI's API.  In this tutorial, we will use a subset of this reference database.  

Also, we will imagine that we have three metagenomes from three soil types:  a corn field, a soybean field, and a prairie field.  We are going to identify known nitrogen fixation genes in these metagenomes.

You can grab this data by the following command.  Navigate to your home directory and execute the following::

    git clone https://github.com/germs-lab/blast-tutorial-fungene.git

This will go to a version-controlled server on Github and make a copy of this data onto your local computer in a folder called "blast-tutorial-data".  You should be aware of where this folder and data exist.  Git is a great program and VERY useful for bioformatics -- I would highly suggest you learn more about it and maybe we will have time to discuss it more.

Let's navigate to a the data folder and UPDATE this folder and look at this data::

    cd blast-tutorial-fungene
    cd data
    ls
    
The first thing we need to do is to tell BLAST that our nifH reference genes contained in fungene_9.3_nifH_1122_unaligned_nucleotide_seqs.fa are (a) a database, and (b) a nucleotide database.  That's done by calling 'makeblastdb'::

    makeblastdb -in nifh-ref.fa -dbtype nucl

Next, we can run BLAST, or more specifically blastn (nucleotide against nucleotides alignment) by using the following command::

    blastn -query metags/corn.fa -db nifh-ref.fa

You'll see a BLAST output print to the screen really fast.  To save it, you can identify the name of an output file::

    blastn -query metags/corn.fa -db nifh-ref.fa -out metags/corn.fa.x.nifh.blastnout.txt

Maybe you don't want to see all the alignments but would rather have a tabular output.  You can check out how to do this `in the manual <http://www.ncbi.nlm.nih.gov/books/NBK279675/>`_ and with the command::

    blastn -query metags/corn.fa -db nifh-ref.fa -out corn.fa.x.nifh.blastnout.tsv -outfmt 6
 
You can save the BLAST output to a tabular output that can be easily parsed.

Automation exercise
-------------------

You can save a lot of time if you learn how to automate things in the shell.  Try the following command when in the data folder::

    for x in metags/*fa; do blastn -query $x -db nifh-ref.fa -outfmt 6 -out $x.x.nifh.blastnout.tsv; done

What does it do?

Getting the best hit
--------------------

BLAST will find every gene which "hits" a sequence read and a single read will have multiple hits.  Often, you will only want to associate one annotation per read.  I have written a very simple script to parse only the best hit for each read.  Try from in the data directory::

    python ../scripts/best-hit.py metags/corn.fa.x.nifh.blastnout.tsv > metags/corn.fa.x.nifh.blastnout.tsv.best

Compare the metags/corn.fa.x.nifh.blastnout.tsv and metags/corn.fa.x.nifh.blastnout.tsv.best.

For loop exercise - best hits
-----------------------------

Using a for loop, save files for each blast output that only include the best hits. Make sure you end the file names with the word "best".  Also, make sure you have only 3 files like this.  You can delete ones from previous exercises if necessary.

More practice executing a program/script
----------------------------------------

So now we have a file where we have all the best hit reads associated with known nifH genes for each metagenome.  However, we want to compare the number of genes per metagenome.  I've written a script for this and you can try this from the data directory::

    python ../scripts/count-up.py metags/*best

This creates a file in the data folder called "summary-count.tsv".  

Take a look at this file.

Adding annotations
------------------

Finally, you might think to yourself that the NCBI accession numbers aren't that useful.  But we can grab more details for these IDs, after all we have the sequence file with the associated gene description, nif-ref.fa.  Try this script::

	 python ../scripts/import-ann.py nifh-ref.fa summary-count.tsv > summary-count.annotations.tsv

Take a look at the summary-count.annotations.tsv

Conclusion
----------

So now you've executed at least 3 programs within this single tutorial.  There is a lot more to learn about how to write your own scripts, but this is the first step towards understanding the value of being able to code.  And actually, you've been coding along! executing for loops in shell.  How much have you learned in one day?  Hopefully its an incentive to keep learning!

