# Juicer-guide
How I was able to run juicer 

### Installing Juicer with singularity
On an HPC cluster it is beneficial to install juicer without root access as a singularity image. 
This command can download the compatible docker image using singularity:
```
singularity pull docker://aidenlab/juicer:latest
```
This command should download a file named ```juicer-kasm_latest.sif``` in the current working directory.

Once installed juicer commands can be executed in the image with either

**Apptainer**
```
apptainer exec -B $PWD -B /lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/hifiasm/final/ \
 /lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/juice/juicer/Docker/juicer-kasm_latest.sif juicer.sh \
 -y pandora_DpnII.txt \
 -p pandora.chromer.sizes \
 -z /lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/hifiasm/final/pacbio-ont-sana-hic-1.hic.p_ctg.fa \
 -g to_juiced \
 -S final \
 -T 30 -t 30
```

**Singularity**

```
apptainer exec -B $PWD -B /lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/hifiasm/final/ \
 /lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/juice/juicer/Docker/juicer-kasm_latest.sif juicer.sh \
 -y pandora_DpnII.txt \
 -p pandora.chromer.sizes \
 -z /lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/hifiasm/final/pacbio-ont-sana-hic-1.hic.p_ctg.fa \
 -g to_juiced \
 -S final \
 -T 30 -t 30
```

> NOTE:
> In this case both the current working directory ```$PWD``` and the directory containing the indexed fasta files ```/lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/hifiasm/final/``` were made accessible to the container using the Bind option ```-B```
> Without binding all the needed files you will get an error saying a file "does not exist"

Command line options used :
- y : path to restriction enzyme site file ( Ouput of the commmand generate_site_positions.py )
- p : path to file with chrom.sizes ( Also Output of the commmand generate_site_positions.py Exept only the first and last columns )
- z : path to the fasta genome assembly, make sure the output of '''BWA index''' is in the same directory ( Also add the path to the container with ```-B``` )
- g : genome ID ( pick a name )
- S : used to manually contiue a partially completed run of the command refer to the list of options below  :
  > - Use "chimeric" when alignment has finished or to start from previously
  >   aligned files
  > - Use "merge" when chimeric handling has finished but the merged_sort file
  >   has not yet been created.
  > - Use "dedup" when the files have been merged into merged_sort but
  >   merged_dedup has not yet been created.
  > - Use "afterdedup" when dedup is complete but statistics haven't been run
  > - Use "final" when the reads have been deduped into merged_dedup but the
  >   final hic files have not yet been created.
  > - Use "postproc" when the hic files have been created and only
  >   postprocessing feature annotation remains to be completed.
  > - Use "early" for an early exit, before the final creation of the hic files
- t : Threads to use in BWA mapping
- T : Threads to use in HiC file generation ( requires more RAM the higher the threads )
  
### Preparing the site and size files

Before running the main '''juicer.sh''' command the Restriction Enzyme site file as well as the chromosome size file need to be created 

**Before creating either of these there also has to be a directory named '''fastq/''' that contains all of your paired-end Hi-C reads**
**The Files need to end in the suffix '''_R1.fastq''' or '''_R2.fastq''' depending on which end of the pair the file is.**
**The final ```juicer.sh``` command should be run in the direcory containing the directory ```fastq/```**

Run the follwing command in the singularity shell (Change the path to wherever you have your container saved )

```
singularity exec  /lustre/isaac/scratch/madler5/blue_pandora_seq/Genome_Assembly/juice/juicer/Docker/juicer-kasm_latest.sif generate_site_positions.py  <restriction enzyme> <genome>
```
inputs:
- "restriction enzymes" : type in the name of the restriction enzymes used to generate you HiC reads if applicable.
  > List of pre-made restriction enzymes  in the file ```generate_site_positions.py``` :
  >   
  >  'HindIII'     : 'AAGCTT',
  > 
  >  'DpnII'       : 'GATC',
  > 
  >  'MboI'        : 'GATC',
  > 
  >  'Sau3AI'      : 'GATC',
  > 
  >  'Arima'       : [ 'GATC', 'GANTC' ],
  >
  > If you can't find your's or the pattern is diffrent you can change this portion of the file manually.
- genome : path to the assembly genome
- location : path to output location

Once this command finishes take note of its location.

... One last thing

The chromosome/contig size file needs to be generated using the output of the previous command 
This can be done with an AWK one-liner command:
```
awk 'BEGIN{OFS="\t"}{print $1, $NF}' mygenome_myenzyme.txt > mygenome.chrom.size
```
This command just prints the first ```$1``` and the last ```$NF``` field of each line definig the field seperator as a tab ```\t```


  
  



