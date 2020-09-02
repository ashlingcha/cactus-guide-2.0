# cactus-guide-2.0
Up to date guide of running genome aligner CACTUS -https://github.com/ComparativeGenomicsToolkit/cactus  
(last edit - 01/09/2020)

DNAZoo's goal is to create 1 alignment with all dnazoo genomes + all mammals on ncbi (excluding those from the 200 mammals project)

- there are ~120 mammals assembled by DNAZoo, ~150 genomes overall
- there are 458 mammals on the NCBI DB
  (note that there will be overlap, in this instance we will use the DNAZoo genome)

-> annotations to form tree can be found here: https://www.dnazoo.org/post/announcing-the-release-of-updated-genome-annotations
https://github.com/macmanes-lab/dnazoo_annotation/blob/master/phylogeny_v2.newick


Starting Point (for testing) - DNA Zoos 9 primates

- Macaca fuscata

- Allenopithecus nigroviridis

- Mandrillus sphinx

- Saimiri boliviensis

- Cebuella pygmaea

- Propithecus coquereli

- Microcebus murinus

- Eulemur flavifrons

- Eulemur mongoz 

(((((((((Macaca_fuscata:0.0037119499999999916,Allenopithecus_nigroviridis:0.0062513899999999956):0.002054940000000005,Mandrillus_sphinx:0.00705037):0.0152843,(Saimiri_boliviensis:0.01563329999999999,Cebuella_pygmaea:0.014609800000000006):0.01602400000000001):0.024321499999999996,(Propithecus_coquereli:0.019122499999999987,(Microcebus_murinus:0.022461900000000007,(Eulemur_flavifrons:0.0037270299999999923,Eulemur_mongoz:0.004299560000000008):0.01338700000000001):0.0021540499999999907):0.0241701):0.006678949999999989):0.007462670000000005):0.008728700000000006):0.005779100000000009):0.10036899999999999);

review tree here:http://etetoolkit.org/treeview/

Pairwise alignment start: 

(Eulemur_flavifrons:0.0037270299999999923,Eulemur_mongoz:0.004299560000000008);

Eulemur flavifrons (2.1G): 
https://www.dropbox.com/s/snr3ua4qnxin6sv/Eflavifronsk33QCA_HiC.fasta.gz?dl=0

Eulemur mongoz (2.5G):
https://www.dropbox.com/s/780pcbhrfllkbph/Eulemur_mongoz_HiC.fasta.gz?dl=0


PAWSEY notes

-Zeus 96GB and 128GB nodes: https://pawsey.org.au/systems/zeus/

-Magnus 64GB nodes: https://pawsey.org.au/systems/magnus/

-Topaz 192 GB nodes: https://pawsey.org.au/systems/topaz/

- Zeus benefits over Nimbus due to higher RAM at expense of walltime

- Toil does not work with Slurm - limits to using 1 node on Zeus or Magnus (Toil is developed for cloud)

- GPU acceleration has been introduced in most recent versions of cactus - there are GPUs on Topaz cluster - issue is on Topaz cannot reun CPU-only code 

For monitoring jobs
export SACCT_FORMAT=jobid%-20,jobname,partition,user,account,submit,start,end,elapsed,nnodes,ncpus,reqmem,maxrss,maxvmsize,state,exitcode,nodelist%10
sacct -a -u ashling_charles -S 2020-08-01
-MaxRSS and MaxVMSize gives idea of maximum RAM usage


Important notes:

-typical size of mammalian genomes are 2-4GB 

-on cluster set up, VMs do not talk to each other i.e. memory (RAM) and core requirements are specified per machine not per cluster

-cactus job is a workflow made up of several tasks - can break these downs
-when downloading genomes from ncbi, need to remove spaces in the names of the chromosomes eg. 

 >NC_000001.11 Homo sapiens chromosome 1, GRCh38.p13 Primary Assembly
 to
 >NC_000001.11

-guide to AWS instructions for cactus: https://github.com/ComparativeGenomicsToolkit/cactus/blob/master/doc/running-in-aws.md 

   main points:
    
   - mammal-size genomes require (N/2)x20 c4.8xlarge instances on the spot market,
     (N/2) r3.8xlarge on-demand
         ie. for pairwise alignment 20 c4.8xlarge instances, 1 r3.8xlarge instance
    

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

Test run on PAWSEY's Zeus (Mammalian pairwise alignment)
SeqFile - testalignment.txt:
(human,greykangaroo);

*human /group/pawsey0263/ashling_charles/human.fasta (size:3.1G)

greykangaroo /group/pawsey0263/ashling_charles/mg-2k.fasta.masked (size:3.4G)

Cactus script:

#!/bin/bash -l
#SBATCH --job-name="myjob"
#SBATCH --nodes=1
#SBATCH --account=pawsey0263
#SBATCH --export=NONE
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=28
#SBATCH --output=cactusexample.%j.o
#SBATCH --error=cactusexample.%j.e

module load singularity
CONTAINER_PATH=/scratch/pawsey0263/ashling_charles/test-cactus

srun singularity exec cactus_v1.0.0.sif cactus jobStore /group/pawsey0263/ashling_charles/testalignment.txt humankangaroo.hal --binariesMode local --restart

#TOIL_SLURM_ARGS="--nodes 8 --export=ALL" srun singularity exec cactus_v1.0.0.sif cactus --binariesMode local --batchSystem slurm --workDir=tmp --defaultCores 28 --maxCores 28 --maxNodes 8 --logLevel=debug --defaultDisk 96G --defaultMemory 96G jobStore evolverMammals.txt evolverMammals.hal --disableCaching --clean=always

-> issue with script - always gets stuck in LastzRepeatMasking step


