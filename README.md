# cactus-guide-2.0
Up to date guide of running genome aligner CACTUS (last edit - 01/09/2020)

DNAZoo's goal is to create 1 alignment with all dnazoo genomes + all mammals on ncbi (excluding those from the 200 mammals project)

PAWSEY notes
-Zeus benefits over Nimbus due to higher RAM at expense of walltime
-Toil does not work with Slurm - limits to using 1 node on Zeus or Magnus (Toil is developed for cloud)
-GPU acceleration has been introduced in most recent versions of cactus - there are GPUs on Topaz cluster - issue is on Topaz cannot reun CPU-only code 
For monitoring jobs
export SACCT_FORMAT=jobid%-20,jobname,partition,user,account,submit,start,end,elapsed,nnodes,ncpus,reqmem,maxrss,maxvmsize,state,exitcode,nodelist%10
sacct -a -u ashling_charles -S 2020-08-01
-MaxRSS and MaxVMSize gives idea of maximum RAM usage


Important notes:
- typical size of genomes are 1-4GB 
- 
- on cluster set up, VMs do not talk to each other i.e. memory (RAM) and core requirements are specified per machine not per cluster
-cactus job is a workflow made up of several tasks - can break these downs
-when downloading genomes from ncbi, need to remove spaces in the names of the chromosomes eg. 
>NC_000001.11 Homo sapiens chromosome 1, GRCh38.p13 Primary Assembly
to
>NC_000001.11


Set-up (on pawsey) 
dir="$MYSCRATCH/test-cactus"
mkdir -p $dir
cd $dir

module load singularity
singularity pull docker://quay.io/comparative-genomics-toolkit/cactus:v1.0.0

git clone https://github.com/ComparativeGenomicsToolkit/cactus.git
cp -p cactus/examples/evolverMammals.txt .


Cactus-prepare command (used to list chunks of the workflow i.e. can be run seperately - commands in the same block can be executed concurrently)

singularity exec cactus_v1.0.0.sif cactus-prepare --jobStore jobstore evolverMammals.txt steps-output steps-output/evovlerMammals.txt steps-output/evolverMammals.hal &>list_commands

#inspect the output file
cat list_commands


Test run (example given in github repository)
salloc -c 28 -t 1:00:00
srun singularity exec cactus_v1.0.0.sif cactus jobStore evolverMammals.txt evolverMammals.hal --root mr --binariesMode local

when it runs out of walltime, can continue with command: 
 srun singularity exec cactus_v1.0.0.sif cactus jobStore evolverMammals.txt   evolverMammals.hal --root mr --binariesMode local --restart

Test run (Mammalian pairwise alignment)
testalignment.txt:
(human,greykangaroo);

*human /group/pawsey0263/ashling_charles/human.fasta (size:3.1G)
greykangaroo /group/pawsey0263/ashling_charles/mg-2k.fasta.masked (size:3.4G)


