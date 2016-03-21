Pseudo-denovo assembly pipeline

About
-----
The pseudo-denovo assembly pipeline was developed to discover and map sequences missing from reference genomes.  Specifically, it maps paired and/or unpaired raw sequences to user-selected genomes, isolates sequences that cannot be mapped to the genome, de-novo assembles them, then uses paired-end relationships to predict likely loci of the contigs in the genome.  The scripts included for this pipeline are intended to automate the pipeline for ease of use for all, but advanced users are encouraged to edit parameters for maximum versatility depending on your experimental question.  

The pipeline was originally used to re-analyze publicly available data to discover large sequences that were previously missed due to population-level and individual genomic variation and the nature of commonly used genome-resequencing pipelines.  For more information on the theory and suggestions of possible uses, please consult the following manuscripts. If you use this pipeline for published research, please cite each of these papers as well.

Faber-Hammond, Joshua J., and Kim H. Brown. "Pseudo-De Novo Assembly and Analysis of Unmapped Genome Sequence Reads in Wild Zebrafish Reveal Novel Gene Content." Zebrafish (2016).

(Bowtie)
Langmead, Ben, et al. "Ultrafast and memory-efficient alignment of short DNA sequences to the human genome." Genome biol 10.3 (2009): R25.

(Mira)
Chevreux, Bastien, Thomas Wetter, and Sándor Suhai. "Genome sequence assembly using trace signals and additional sequence information." German conference on bioinformatics. Vol. 99. 1999.

Getting Started
---------------
The pseudo-denovo pipeline is simple to use, but requires the installation of several third party programs.  The pipeline is optimized for Bowtie2 and Mira3/4, but will likely work with future releases of the programs as well as long as command-line syntax remains the same.  I will try to update the pipeline to remain compatible with future releases.  These packages and installation instructions for these programs are found at the following URLs:

http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
http://mira-assembler.sourceforge.net/

Please be aware that Mira requires that the working directory is not on an NFS mount.  Beyond that, the pipeline uses command-line tools pre-installed in Unix terminal environments, such as ‘awk’ and ‘sed’.  

Now, simply copy the scripts into whichever working directory you choose containing your data.  The pipeline was tested using Ubuntu and MacOSX operating systems, but should work in any Unix environment where both Bowtie2 and Mira4 programs are compatible. 

Running the pipeline
--------------------
There are 3 scripts included for the pipeline, depending on your intended use.  The scripts ‘pdn_processing_M3.txt’ and ‘pdn_processing_M4.txt’ are the fully automated pipeline optimized for Mira3 and Mira4 respectively, while ‘pdn_mapping.txt’ requires user-specified SAM alignment files.  

The first scripts (pdn_processing_M3.txt, pdn_processing_M4.txt) require only two input files; raw sequences in fastq format and the genome you plan to align to in fasta format.  Prior to running either script, you must combine all your raw sequences into a single input file.  The single input file can include data from any number of sequencing runs and can include single and paired-end runs together if desired.  Although it is fairly obvious, the input file should include both paired-end sequencing files from a run if the user is also hoping to map assembled sequences back to the genome.

We suggest use of the ‘cat’ tool for combining fastq files.

	cat [seq-pair_1.fastq] [seq-pair_2.fastq] [single.fastq] … > [pipeline_input.fastq]

In this example, [seq-pair_1.fastq] and [seq-pair_2.fastq] are your paired-end read files, [single.fastq] may be a single-end read file for the same individual, and [pipeline_input.fastq] is the name of the input file for the pipeline.  Once you have your input and the reference genome of your choice, make sure they are in the same working directory as both ‘pdn_processing.txt’ and ‘pdn_manifest’.  You can run either pipeline with the following script.

	bash pnd_processing_M*.txt

Choose ‘pdn_processing_M3.txt’ or ‘pdn_processing_M4.txt’ depending on the version of Mira you have installed.  The pipeline will prompt you for 4 variables.  First, enter the name of your project, which will serve as the name of your output folder.  Second, enter the prefix of your fastq input file, for example if your input is named ‘pipeline_input.fastq’ enter ‘pipeline_input’.  Third, enter the sequencing technology based on Mira3 syntax (i.e. ‘solexa’, ‘iontor’, etc).  Fourth, enter the full name of your reference genome in fasta format. 

Along with our pipeline, we’ve included example data for E. coli:

GCA_000005845.2_ASM584v2_genomic.fna  (genome)
SRR1002800_1_trim.fastq  (raw sequence reads)
SRR1002800_2_trim.fastq  (raw sequence reads)

To run the pipeline with example data, unzip and combine the two paired-end sequence files into a single fastq file as described above, then run the pipeline.  This dataset took ~30 minutes to run on a performance computing cluster.  If you don’t need these files, feel free to delete them as they are somewhat large.

Parameters
----------
Please adjust any parameters for Mira4 assembly in the ‘pdn_manifest.txt’ file.  The manifest file is currently set to use 4 threads.  For the Mira3 pipeline, adjust any necessary parameters in the script itself.  See Mira documentation for more information on running Mira.

The pipeline utilizes default parameters for Bowtie2.  Advanced users are encouraged to alter Bowtie2 parameters in the script if necessary for special case alignments.  

Mapping steps only
------------------
You may want to map de-novo assembled contigs to a reference genome but prefer other sequence alignment or assembly software than Bowtie2 or Mira.  If this is the case, we’ve also included the ‘pdn_mapping.txt’ script.  This script requires only 2 input files, specifically SAM format alignment files of combined paired-end raw sequences in both your assembly (1) and genome (2).  To run this pipeline, you can enter the following command.

	bash pnd_mapping.txt

Enter the full names of your SAM files when prompted, and the pipeline will do the rest.

Output files
------------
After running the full pipeline, your output directory will contain 7 files and a Mira assembly output folder (see Mira documentation for description of Mira output files).  Your assembly of unmapped contigs is contained in this Mira output folder, called ‘unaligned_assembly’. Here are the descriptions of additional files:

1) ‘log_align_genome.txt’ contains Bowtie2 alignment statistics regarding the number and portions of raw reads mapped to the reference genome.
2) ‘log_align_assembly.txt’ contains Bowtie2 alignment statistics regarding the number and portions of raw reads mapped to the Mira denovo assembly of unmapped contigs.
3) ‘predicted_loci.txt’ contains loci information for the assembled unmapped contigs in the reference genome.  The respective columns columns refer to; 
	(1) name of contig in assembly
	(2) base pair mapping location of raw read in contig
	(3) Bowtie2 mapping score in contig 
	(4) name of scaffold in reference genome
	(5) base pair mapping location of raw read in scaffold 
	(6) Bowtie2 mapping score in genome
4) ‘*_in_genome’ is your zipped Bowtie2 genome alignment output in SAM format.  See Bowtie2 documentation for description of file. 
5) ‘*_in_assembly’ is your zipped Bowtie2 assembly alignment output in SAM format.  See Bowtie2 documentation for description of file. 
6) ‘*_unaligned.fastq’ (zipped) contains reads from your raw sequence input that do not align to your reference genome.
7) ‘unaligned_in.fastq’ (zipped) is a re-formatted version of output file #6 used for Mira input.  The sequence headers are renamed so that Mira doesn’t produce an error resulting from duplicate sequence IDs (from both paired-end reads).  If you wish to re-assemble the unmapped sequences for any reason, use this file for input.

* This will be the fastq prefix entered at the start of the pipeline.

Final notes
-----------
Thank you for using the pseudo-denovo assembly pipeline.  I hope you found it useful. For any questions, comments, or suggestions, feel free to reach out.

Josh Faber-Hammond
jfaberha@gmail.com