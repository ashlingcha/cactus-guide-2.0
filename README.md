# cactus-guide-2.0
Up to date guide of running genome aligner CACTUS (last edit - 01/09/2020)

Important notes:
- typical size of genomes are 1-4GB 
- 
- on cluster set up, VMs do not talk to each other i.e. memory (RAM) and core requirements are specified per machine not per cluster
-cactus job is a workflow made up of several tasks - can break these downs


Set-up (on pawsey) 
dir="$MYSCRATCH/test-cactus"
mkdir -p $dir
cd $dir

module load singularity
singularity pull [docker://quay.io/comparative-genomics-toolkit/cactus:v1.0.0|docker://quay.io/comparative-genomics-toolkit/cactus:v1.0.0]

git clone [https://github.com/ComparativeGenomicsToolkit/cactus.git|https://github.com/ComparativeGenomicsToolkit/cactus.git]
cp -p cactus/examples/evolverMammals.txt .


Cactus-prepare command (used to list chunks of the workflow i.e. can be run seperately - commands in the same block can be executed concurrently)

singularity exec cactus_v1.0.0.sif cactus-prepare --jobStore jobstore evolverMammals.txt steps-output steps-output/evovlerMammals.txt steps-output/evolverMammals.hal &>list_commands

#inspect the output file
cat list_commands


Test run
salloc -c 28 -t 1:00:00
srun singularity exec cactus_v1.0.0.sif cactus jobStore evolverMammals.txt evolverMammals.hal --root mr --binariesMode local

when it runs out of walltime, can continue with command: 
 srun singularity exec cactus_v1.0.0.sif cactus jobStore evolverMammals.txt   evolverMammals.hal --root mr --binariesMode local --restart



PAWSEY
-Zeus benefits over Nimbus due to higher RAM at expense of walltime
