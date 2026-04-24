# hydractinia_TEs
-----------------------------------
# Annotate a TE database for hydractinia based on the HsymV2.1 genome (https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_029227915.1/)

Follows steps listed in Duran-Fuentes, et al. 2025 https://doi.org/10.1186/s12864-025-11591-0

## Run RepeatModeler

```
#!/bin/bash
#SBATCH --job-name=repeatmod
#SBATCH --time=3-00:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=20
#SBATCH --mem=40GB
#SBATCH --output=out_%j.log

module load repeatmodeler

BuildDatabase -name hydractinia -dir /data/katzeram/popgen/TEs/genome/

RepeatModeler -engine ncbi -database hydractinia -threads 20 -LTRStruct

```

## Extract Unknown sequence from the RepeatModeler output

```
grep "^>" hydractinia-families.fa | sed -e "s/>//" > headers.txt
grep 'Unknown' headers.txt > unknown_families.txt

module load seqkit
seqkit grep -n -f unknown_families.txt hydractinia-families.fa > hydractinia_unknown_families.fa
```

## Reannotate unknowns with DeepTE & TESorter

TESorter: https://github.com/zhangrengang/TEsorter
DeepTE: https://github.com/LiLabAtVT/DeepTE

Install TESorter
```
module load mamba_install
source /home/katzeram/bin/mymamba/

mamba create --name tesorter
mamba activate tesorter
conda install -c bioconda tesorter
```

Run TESorter
```
#!/bin/bash
#SBATCH --job-name=tesorter
#SBATCH --time=06:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=20
#SBATCH --mem=40GB
#SBATCH --output=tesorter_out_%j.log

module load mamba_install

mamba activate tesorter

TEsorter -pre hydractinia_tesort -p 20 hydractinia_unknown_families.fa

```


Install DeepTE
```
module load mamba_install
source /home/katzeram/bin/mymamba/

mamba create --name deepte python=3.6
mamba activate deepte

conda install biopython
conda install keras=2.2.4
conda install numpy=1.16.0

conda config --env --set channel_priority flexible

```

