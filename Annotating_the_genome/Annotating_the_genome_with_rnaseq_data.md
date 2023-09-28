Ensembl RNASeq Practical
========================

The aim of this practical session is to use STAR to align 2 lanes of Zebrafish paired end Illumina RNASeq reads to chromosome 12 of the zebrafish GRCz11 assembly. We have restricted the analysis to chromosome 12 in order to speed up the alignment process. 

Once the reads have been aligned you can assemble transcript models using Scallop and finally display the transcript models in the Ensembl browser alongside some of our own annotation.


## Overview:

We are going to use "terminal" application to execute STAR, Scallop and samtools commands.

Command-line actions usually start with 4 spaces ("    "). Please note that many commands are wrapped across more than one line.

There are questions throughout this document, indicated with a "->", try to answer these, they should help you understand the process.


Step 0: Go to working dir
-----------------------------

Open the terminal application
You should be in /workspace . Change to the ensembl-annotation directory

    cd /workspace/ensembl-annotation

To check which directory you are in, please execute:

    pwd

> :warning: At the start of each step, you should be in /workspace/ensembl-annotation so run the `cd /workspace/ensembl-annotation` command if you get any "file / path not found" errors.

You should see a directory entitled Annotating_the_genome that contains the read Fastq and genome files

To check what is in the directories, please execute:

    ls -lh ./Annotating_the_genome/*


Step 1: Index the genome file
-----------------------------

First we need to index the genome file so that STAR can use it. We do this using the STAR genomeGenerate command. 

STAR command to index the genome fasta file:

    STAR --runMode genomeGenerate --genomeDir ./Annotating_the_genome/Genome/ --genomeFastaFiles ./Annotating_the_genome/Genome/Danio_rerio.GRCz11.dna.chromosome.12.fa --outFileNamePrefix ./Annotating_the_genome/Genome/chr12_GRCz11

Output will be printed in your terminal, STAR commands complete successfully with the output like "[main] Real time: 10 sec; CPU 15 sec".

* -> 1. What output did this command create? hint: use `ls ./Annotating_the_genome/Genome/` or use the file browser on the left to see what new files have appeared.

* -> 2. Why do we index the genome before aligning to it? 


Step 2: Align the reads to the genome
-------------------------------------

When the genome has been indexed, we can start the alignment. We are using 2 lanes of 76bp paired end reads, that gives us 4 files, 2 for each lane containing the 1st (or forward) and 2nd (or reverse) reads respectively. 

Command to align 2cell data:

    STAR --runThreadN 8 --genomeDir ./Annotating_the_genome/Genome/ --readFilesIn ./Annotating_the_genome/Fastq/2cell_chr12_R1.fastq ./Annotating_the_genome/Fastq/2cell_chr12_R2.fastq --outSAMtype BAM SortedByCoordinate --outFileNamePrefix ./Annotating_the_genome/Bam/2cell

Command to align 6hpf data:

    STAR --runThreadN 8 --genomeDir ./Annotating_the_genome/Genome/ --readFilesIn ./Annotating_the_genome/Fastq/6hpf_chr12_R1.fastq ./Annotating_the_genome/Fastq/6hpf_chr12_R2.fastq --outSAMtype BAM SortedByCoordinate --outFileNamePrefix ./Annotating_the_genome/Bam/6hpf

* -> 3. What files did you create this time? hint: use "ls ./Annotating_the_genome/Bam".



Step 3: Get basic statistics about the alignments
-------------------------------------------------

Flagstat command for 2cell data:

    samtools flagstat ./Annotating_the_genome/Bam/2cellAligned.sortedByCoord.out.bam

Flagstat command for 6hpf data:

    samtools flagstat ./Annotating_the_genome/Bam/6hpfAligned.sortedByCoord.out.bam


* -> 4. What percentage of your reads were successfully mapped?

Step 4: Index the bam files
----------------------------

In order to view the bam files in the Ensembl browser (Step 6) you will need to index them.

Command to index the 2cell BAM file:

    samtools index ./Annotating_the_genome/Bam/2cellAligned.sortedByCoord.out.bam

Command to index the 6hpf BAM file:

    samtools index ./Annotating_the_genome/Bam/6hpfAligned.sortedByCoord.out.bam

Step 5: Assemble the transcripts
--------------------------------

When the reads have been aligned, we can assemble the transcripts. 

Note: scallop writes a lot to STDOUT, so it's best to redirect this output to a file (we add this to the end of the command `>./Annotating_the_genome/GTF/2cell_scallop_stdout`)

Command to assemble transcripts from 2cell data:

    mkdir -p ./Annotating_the_genome/GTF

    scallop -i ./Annotating_the_genome/Bam/2cellAligned.sortedByCoord.out.bam -o ./Annotating_the_genome/GTF/2cell_scallop.gtf >./Annotating_the_genome/GTF/2cell_scallop_stdout

Command to assemble transcripts from 6hpf data:

    scallop -i ./Annotating_the_genome/Bam/6hpfAligned.sortedByCoord.out.bam -o ./Annotating_the_genome/GTF/6hpf_scallop.gtf >./Annotating_the_genome/GTF/6hpf_scallop_stdout

* -> 5. What files did you create this time? hint: use "ls ./Annotating_the_genome/GTF".
* -> 6. What is stored in the GTF file?

Step 6: View your results in the Ensembl browser
------------------------------------------------

BAM files are often quite large and are unsuitable for uploading to a website, so in order to view the alignments in the Ensembl browser you need to host the sorted and indexed BAM files on either a webserver or an ftp site. Both the sorted bam file and the index file (ending .bai) are needed to view the alignments on the website. The website requires the bam file URL to be entered, it then looks for a .bai file with the same name in the same directory.

For convenience we have already set up an ftp site that contains all the files you just created.

To view the ftp, enter the following URL into your web browser:
 [https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/](https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/)
 
We have chosen ENSDARG00000055381 as a good example of a chr12 gene with differential expression highlighted by the 6hpf and 2cell lanes.

To load the alignments from the Bam files:

* Go to the browser: [www.ensembl.org/](www.ensembl.org/)
* Enter ENSDARG00000055381 into the search box to take you to the gene view page, it is a gene called "bambia". We want to go on the Location view, so please select "Region in Detail"
* If you are on the Gene view, click on the Location tab at the top left hand side of the page to take you to the view of the region on the chromosome.
* Now we can load our BAM files.
* Click on the “Configure this page” button on the left panel, this opens a configuration panel.
* To load the BAM files click on “Personal data” tab at the top.
* Name the track (if you have an Ensembl account you can store the track in your account to use another time).
* Add the data using the URL of the files:
  [https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/BAM/2cellAligned.sortedByCoord.out.bam](https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/BAM/2cellAligned.sortedByCoord.out.bam)
* Then do the same for the 6hpf file
  [https://www.ebi.ac.uk/~leanne/Intro2RNA_Mar2021/Bam/6hpfAligned.sortedByCoord.out.bam](https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/BAM/6hpfAligned.sortedByCoord.out.bam)
* Click on the tick in the top right hand corner of the "Configure this page" window to return to the region view browser.
* Once you have attached the remote files you should be able to see them in the region view browser, if they do not show up you may need to turn them on by going to "Configure this page" -> "Your data" and set it to “Normal”.

Done!

* -> 7. What does your BAM track look like compared to the annotated genes? Where do the reads align? hint: look for stacks of reads.
 
To load the transcript models from the GTF files:

* Click on the “Configure this page” button on the left panel, this opens a configuration panel.
* To load the GTF files click on “Personal data” tab at the top.
* Name the track (if you have an Ensembl account you can store the track in your account to use another time).
* Add the data using the URL of the files:
  [https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/GTF/2cell_scallop.gtf](https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/GTF/2cell_scallop.gtf)
* Then do the same for the 6hpf file
  [https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/GTF/6hpf_scallop.gtf](https://ftp.ebi.ac.uk/pub/databases/ensembl/ereboperezsilva/bga23/results/GTF/6hpf_scallop.gtf)
* Click on the tick in the top right hand corner of the "Configure this page" window to return to the region view browser.
* Once you have attached the remote files you should be able to see them in the region view browser, if they do not show up you may need to turn them on by going to "Configure this page" -> "Your data" and set it to “Normal”.

Done.

* -> 8. What does your GTF track look like compared to the Ensembl annotated genes? 
