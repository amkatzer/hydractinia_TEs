# hydractinia_TEs
-----------------------------------
## Annotate a TE database for hydractinia based on the HsymV2.1 genome (https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_029227915.1/)

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
Make TE GTF
Repeatmasker
```
#!/bin/bash
#SBATCH --job-name=repeatmod
#SBATCH --time=06:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=24GB
#SBATCH --output=out_%j.log

module load repeatmasker

RepeatMasker -pa 3 -lib ./RM_418309.WedApr220931362026/consensi.fa.classified -dir . ./genome/HsymV2.1_genomic.fasta
```

https://github.com/CSU-KangHu/HiTE/blob/master/RNA_seq/makeTEGTF.pl
https://zenodo.org/records/18927862

```
perl makeTEgtf.pl -c 5 -s 6 -e 7 -o 9 -t 10 HsymV2.1_genomic.fasta.out > HsymV2.1_repeat.gtf
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

pip install h5py==2.10.0

```

Run DeepTE
```
module load mamba_install
conda activate deepte2
source /home/katzeram/bin/mymamba 
conda activate deepte2
module load hmmer

#generate the domain file
mkdir working_deepte
mkdir output_deepte

python DeepTE/DeepTE_domain.py -d working_deepte/ -o output_deepte/ -i repeatmodeler.lib --hmmscan hmmscan -s ./DeepTE/supfile_dir/

python DeepTE/DeepTE.py -d working_deepte/ -o output_deepte/ -i repeatmodeler.lib -sp M -m_dir Metazoans_model -modify output_deepte/opt_te_domain_pattern.txt
```

remove special characters from the opt_DeepTE.fasta (change the character for each special character found)
```
tr -d '#' <opt_DeepTE.fasta >opt_DeepTE_nosym.fasta
```

# Mine short read data for TEs

Install McClintock protocol
```
module load mamba_install
source /home/katzeram/bin/mymamba
module load snakemake
git clone git@github.com:bergmanlab/mcclintock.git
cd mcclintock
conda activate mcclintock
python mcclintock.py --install

```
