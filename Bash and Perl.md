You received a genomic dataset from collaborators consisting of Illumina (paired ends) and Oxford 
Nanopore sequencing data. Your role in the project is to assemble and annotate the genome. Your 
collaborators are expecting data from a Listeria monocytogenes strain (expected genome size ~ 2.9 
Mbp). 
## Automation with Bash and/or Perl scripts could save you a lot of time…  
 Read processing  
i. Investigate the read quality of the Illumina and Oxford Nanopore datasets. 
## The error below can show up with FASTQ files containing long reads: 
## Exception in thread "Thread-3" java.lang.OutOfMemoryError: Java heap space 
## If so, use more threads with -t, e.g. -t 4  
ii. Plot the read length distribution / calculate the metrics of the nanopore dataset. 
## You can use Dr. Pombert’s python script to do so: 
## https://github.com/PombertLab/Misc/blob/main/read_len_plot.py 
## Read lengths are very important for 3rd generation platforms. Because read sizes 
## influence the assemblies, we should always verify their overall lengths  
iii. If required (and possible), filter accordingly. 
## When using nanofilt, you can safely ignore the warning about pandas  
Genome assembly  
iv. Assemble each dataset using SPAdes (Illumina), Shasta (Nanopore; read the manual; does not 
work with .gz files), Flye (Nanopore) and Unicycler (Illumina + Nanopore; read the manual). 
Remember that different parameters may lead to different results. These processes can take a 
long time, make sure to use screen (screen -S name_of_screen) to run the analyses so that you 
can detach (ctrl+A+D), then re-attach (screen -r name_of_screen) to the shell as needed.  
Contamination removal (from assemblies)  
v. For each assembly, identify the listerial contigs using BLAST homology searches against the 
ref_prok_rep_genomes database (located in /media/Data_1/NCBI/REPGENOMES), then parse 
out the results with Perl to keep only the contigs that are from Listeria monocytogenes (see 
runTaxonomizedBLAST.pl/ parseTaxonomizedBLAST.pl
## To save computation time, use the megablast algorithm rather than the default one  
## (blastn). Because a representative genome for this staphylococcal species might not be in 
## this database, assume that any listerial hit is a valid one.  
Assembly comparisons  
vi. Compare the various assemblies (before and after filtering) using Quast. Select the best 
assembly for downstream analysis. Explain why you selected this assembly. Does it match the 
expected size? Based on your knowledge, also explain which sequencing approach appears 
better for this project and why.  
Genome annotation  
vii. Use PROKKA and DFAST to annotate the best Listeria monocytogenes assembly (without the 
contaminants). Use F23 as the locus_tag prefix and autoincrement values by 10. For PROKKA, 
enforce GenBank compliance.  
viii. Compare the total number of proteins predicted by PROKKA and DFAST for your assembly. 
Are the results congruent? Explain your answer.  
Contamination removal (from sequencing datasets)  
ix. Let’s filter out the sequencing datasets to keep only the reads from Listeria monocytogenes. 
Use your best assembly as reference. We can use get_SNPs.pl (--rmo + --bam), minimap2 and 
samtools bam2fq for this purpose. GZIP your FASTQ outputs. Compare the number of reads 
before/after with FASTQC. Compare the *.fastq.gz sizes with du -sh too. 
