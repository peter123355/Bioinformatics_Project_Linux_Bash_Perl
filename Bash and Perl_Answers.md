Read Processing 
I 
Copied the files in this /media/Data_2/BIOL550/F23/LongAssignment/ 
location to biolF23_14. 
Created separate file directories for Illumina and nanopore, named 
Illumina_quality, nanopore_ quality and nanopore_read_ quality using 
mkdir. 
 Executed following commands,   
1.fastqc -t 4 -o illumina_read_quality illumina.R1.fastq,  
2. fastqc -t 4 -o illumina_read_quality illumina.R2.fastq,  
3.fastqc -t 4 -o nanopore_read_quality nanopore.fastq 
Now to check the quality of files. The following commands have been used.  
1.xdg-open illumina_read_quality/illumina.R2_fastqc.html 
2.xdg-open illumina_read_quality/illumina.R1_fastqc.html 
3.xdg-open nanopore_read_quality/nanopore_fastqc.html 
2 
II 
Downloaded the python file from GitHub using the flowing command. 
1. wget 
https://raw.githubusercontent.com/PombertLab/Misc/main/re
 ad_len_plot.py 
2. Made the script executable using the following command. 
o chmod +x read_len_plot.py  
3. Now the script has been run, using the following commands, 
./read_len_plot.py -f nanopore.fastq.gz -c darkorange -o 
nanopore_read_distribution.svg -x 50000 --title "Nanopore 
Read Length Distribution" --title_font bold --height 10.8 - -width 19.2 --ticks 5 --yscale log 
Where: -c darkorange: Sets the color for the plot (can change if 
needed). -o nanopore_read_distribution.svg: Specifies the output 
file for the read length distribution plot. -x 50000: Sets the maximum value on the X-axis. 
 The File has been converted from .svg to .png. 
3 
Figure 1. Nanopore_read_distribution 
III 
After the plotting is done, downloaded the NanoFilt.py using the command.  
wget 
https://raw.githubusercontent.com/wdecoster/nanofilt/mast
 er/nanofilt/NanoFilt.py 
After the file has been downloaded, to check for the python dependencies we 
create a file name-requirements.txt and have added the required dependency 
into the .txt file using the below commands. 
echo "# nanofilt dependencies" > requirements.txt 
echo "numpy>=1.20.0" >> requirements.txt 
echo "pandas>=1.2.0" >> requirements.txt 
Once the requirements files have been generated, we tried to install them at 
the user level by using the following commands. 
pip install -r requirements.txt –user 
4 
 Now used the following commands to filter the data set, 
gunzip -c reads.fastq.gz | NanoFilt -q 10 -l 500 -
headcrop 50 | pigz -p 8 > trimmedreads.fastq.gz and 
generated a file named > trimmedreads.fastq.gz It took approximately 9 
minutes. 
IV 
Genome assembly 
SPAdes (Illumina) – To assemble data for Illumina using SPAdes initially the 
.gz extension file was unzipped using the following commands,  
gunzip -c illumina.R1.fastq.gz > illumina.R1.fastq 
gunzip -c illumina.R2.fastq.gz > illumina.R2.fastq 
Once the unzipping has been done, a new screen for the spades has been 
created using the command: 
screen -r spades_screen 
Once the SPAde is opened following commands has been run 
spades.py -1 illumina.R1.fastq -2 illumina.R2.fastq -o 
spades_output 
Now the output file named spades_output has been created.  
The file inside the spades_output directory has been explored using the 
following commands, 
cat spades_output/contigs.fasta 
cat spades_output/scaffolds.fasta 
cat spades_output/assembly_graph.fastg 
5 
 Shasta (Nanopore)- To run the nanopore date using Shasta this following has 
been command using the general configuration which is “Nanopore-Dec2019 
configuration”. 
gunzip trimmedreads.fastq.gz 
/opt/shasta-0.10.0/shasta \ --input trimmedreads.fastq \ --Reads.minReadLength 5000 \ --assemblyDirectory shasta_output \ --config Nanopore-Dec2019 \ --memoryMode filesystem \ --threads 8  
Output with name of shasta_output has been generated.  
Flye (Nanopore): 
New screen has been started: 
screen -S flye_nanopore 
Genome Assembly using Flye is run using the command: 
flye --nano-raw trimmedreads.fastq --out-dir 
flye_nanopore_output --threads 8 --genome-size 2.9m -
asm-coverage 50 
Output with name of flye_nanopore_output is generated. 
Unicycler (Illumina): 
 A new screen is started using the command: screen -S 
unicycler_illumina 
 And then the assembly is run by using the command: 
unicycler -1 illumina.R1.fastq -2 illumina.R2.fastq -l 
nanopore_filtered.fastq -o unicycler_output --threads 8 --spades_path /opt/SPAdes-3.13.0-Linux/bin/spades.py 
 Output with name of unicycler_output is generated. 
6 
Unicycler (Nanopore): 
 A new screen is started parallel to the Illumina running: screen -S 
unicycler_nanopore and the assembly is run by using the command: 
unicycler -l nanopore_filtered.fastq -o 
unicycler_nanopore_output --threads 8 
 Output with name of unicycler_nanopore_output is generated. 
Contamination Removal (From Assemblies) 
V. Identifying the Listeria contigs using BLAST homology: 
For Illumina Assembly: 
Taxonomized BLASTN search: 
runTaxonomizedBLAST.pl \ -t 8 \ -p blastn \ -a megablast \ -db /media/Data_1/NCBI/REPGENOMES/ref_prok_rep_genomes 
\ -q spades_illumina_output/contigs.fasta \   -e 1e-30 \ -c 1 \ -o BLAST_illumina 
Parsing the results: 
parseTaxonomizedBLAST.pl \ -v \ -b BLAST_illumina.6 \ -f spades_illumina_output/contigs.fasta \ -n "Listeria monocytogenes" \ -c sscinames \ -o L_monocytogenes_spades.fasta 
7 
For Nanopore Assembly (Shasta): 
Taxonomized BLASTN search: 
runTaxonomizedBLAST.pl \ -t 8 \ -p blastn \ -a megablast \ -db /media/Data_1/NCBI/REPGENOMES/ref_prok_rep_genomes 
\ -q shasta_nanopore_output/Assembly.fasta \ -e 1e-30 \ -c 1 \ -o BLAST_shasta_nanopore 
Parsing the results: 
parseTaxonomizedBLAST.pl \ -v \ -b BLAST_shasta_nanopore.6 \ -f shasta_nanopore_output/Assembly.fasta \ -n "Listeria monocytogenes" \ -c sscinames \ -o L_monocytogenes_shasta.fasta 
For Nanopore Assembly (Flye): 
Taxonomized BLASTN search: 
runTaxonomizedBLAST.pl \ -t 8 \ -p blastn \ -a megablast \ -db /media/Data_1/NCBI/REPGENOMES/ref_prok_rep_genomes 
\ -q flye_nanopore_output/assembly.fasta \ -e 1e-30 \ -c 1 \ -o BLAST_flye_nanopore 
Parsing the results: 
parseTaxonomizedBLAST.pl \ -v \ -b BLAST_flye_nanopore.6 \ -f flye_nanopore_output/assembly.fasta \ -n "Listeria monocytogenes" \ -c sscinames \-o L_monocytogenes_flye.fasta 
8 
For Hybrid Assembly (Unicycler): 
Taxonomized BLASTN search: 
runTaxonomizedBLAST.pl \ -t 8 \ -p blastn \ -a megablast \ -db /media/Data_1/NCBI/REPGENOMES/ref_prok_rep_genomes 
\ -q 
unicycler_output/miniasm_assembly/all_segments.fasta \ -e 1e-30 \ -c 1 \ -o BLAST_illumina 
Parsing the results: 
parseTaxonomizedBLAST.pl \ -v \ -b BLAST_illumina.6 \ -f 
unicycler_output/miniasm_assembly/all_segments.fasta \ -n "Listeria monocytogenes" \ -c sscinames \ -o L_monocytogenes_unicycler.fasta 
9 
Assembly comparisons 
VI. 
Note: 
contigs – for spades illumina output. 
Assembly – output of shasta. 
assembly – flye nanopore output 
all_segments – for unicycler 
Quast on Raw Assemblies, with no filtering: 
quast.py --min-contig 500 spades_illumina_output/contigs.fasta -o 
quast_results_spades_illumina 
Output file generated is:  
quast_results_spades_illumina 
quast.py --min-contig 500 shasta _output/Assembly.fasta -o 
quast_results_shasta_nanopore 
Output file generated is:  
quast_results_shasta_nanopore 
quast.py --min-contig 500 flye_nanopore_output/assembly.fasta -o 
quast_results_flye_nanopore 
Output file generated is:  
 quast_results_flye_nanopore 
quast.py --min-contig 500 
unicycler_output/miniasm_assembly/all_segments.fasta -o 
quast_results_unicycler 
Output file generated is: 
quast_results_unicycler 
Quat to compare all Raw assemblies: 
quast.py --min-contig 500 spades_illumina_output/contigs.fasta 
shasta_output/Assembly.fasta flye_nanopore_output/assembly.fasta 
unicycler_output/miniasm_assembly/all_segments.fasta -o quast_comparison 
10 
Figure 2. Shows the Statistics work. 
Figure 3. Shows the graph of Contigs. 
11 
Quast on Assemblies after filtering: 
 quast.py --min-contig 500 L_monocytogenes_unicycler.fasta 
spades_illumina_output/contigs.fasta -o 
quast_results_spades_illumina_filtered  
Output file generated is: 
 quast_results_spades_illumina_filtered 
quast.py --min-contig 500 L_monocytogenes_unicycler.fasta 
shasta_output/Assembly.fasta -o quast_results_shasta_nanopore_filtered  
Output file generated is: 
 quast_results_shasta_nanopore_filtered 
quast.py --min-contig 500 L_monocytogenes_unicycler.fasta 
flye_nanopore_output/assembly.fasta -o quast_results_flye_nanopore_filtered  
Output file generated is: 
 quast_results_flye_nanopore_filtered 
quast.py --min-contig 500 L_monocytogenes_unicycler.fasta 
unicycler_output/miniasm_assembly/all_segments.fasta -o 
quast_results_unicycler_filtered  
Output file generated is:  
 quast_results_unicycler_filtered 
Now to determine the best assembly, we have generated a multiqc report using multiqc library, 
by using the commands: 
“pip install multiqc” 
“multiqc .” 
12 
Results: 
Figure 4. Shows the Assembly Statistics 
Figure 5. Shows the Number of Contigs 
The best assembly from the image is flye_nanopore_output_assembly. It has the lowest 
total length (2907490 bp) and the highest N50 (2901476 bp). This means that it is the most 
complete and contiguous assembly. 
The other assemblies have higher total lengths and lower N50 values. This means that 
they are less complete and contiguous. 
13 
The N50 value is a measure of the contiguity of an assembly. It is the length of the 
shortest contig that represents at least half of the total length of the assembly. A higher N50 
value indicates a more contiguous assembly. 
The contiguity of an assembly is important for downstream analyses, such as gene 
annotation and comparative genomics. A more contiguous assembly is easier to annotate and 
compare to other assemblies. 
14 
Genome annotation 
VII 
Annotating the best Listeria monocytogenes assembly using PROKKA: 
The following has been run. 
 prokka --addgenes --compliant --genus Listeria --species 
monocytogenes --strain F23 \--gcode 11 --gram pos --cpus 
4 flye_nanopore_output/assembly.fasta 
Annotating the best Listeria monocytogenes assembly using DFAST: 
 dfast -g flye_nanopore_output/assembly.fasta -o DFAST -
strain F23 --cpu 4 
VIII 
1) To count the proteins predicted by PROKKA: 
 grep -c '/locus_tag=' PROKKA_12062023/PROKKA_12062023.gbk 
Result: 8202 
Figure 6. Shows the Results of counting the proteins predicted by PROKKA. 
15 
2) To count proteins predicted v=by DFAST: 
 grep -c '/locus_tag=' DFAST/genome.gbk 
Result: 4044 
Figure 7. Shows the Results of counting the proteins predicted by v=by DFAST. 
This shows that the results are not congruent. PROKKA and DFAST do not give the same 
protein count. This is because they use different algorithms to identify coding sequences (CDS) 
in genomes. 
The vast difference in the protein counts for PROKKA and DFAST has been observed. 
On checking the log files, it seems that each locus tag has been counted twice which led to a 
value double the actual value. The following image clearly shows the reason for the abnormal 
difference. 
16 
Figure 8. Shows the observation of the tags have been repeated twice, which led to the multiple 
counts of the proteins. 
17 
Contamination removal (From Sequencing Datasets) 
IX 
Generated a script file named ‘run_get_SNPs.sh’ and the following code has been added 
into the script file: 
Figure 9. Shows the code in the script file. 
Once the script is generated, it is made executable using the command: 
 ‘chmod +x run_get_SNPs.sh’ 
And then the script is run using the command: ‘./run_get_SNPs.sh’ 
18 
Then run the following command to compare the *.fastq.gz sizes: 
 du -sh nanopore.fastq mapped_R1.fastq.gz 
mapped_R2.fastq.gz 
