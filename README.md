# 🧬 Genome Annotation for GWAS — Simple Guide

This little guide will walk you through how to download your genomes 🗂️ and annotate them 📝 to prepare for a GWAS study — step by step!
I wrote this so that even if you're not a hardcore bioinformatician, you can follow along easily. 🚀

---

## 1️⃣ Download your genomes from NCBI 📥
👉 First, prepare a list of the accession numbers of your genomes (e.g. GCF_XXX...).

You can use this handy tool: [ncbi-genome-download](https://github.com/kblin/ncbi-genome-download)

💻 If you are using IBL server, you don’t need to install it — just run:

   ```bash
micromamba activate ncbi
   ```

To download your genomes:

   ```bash
ncbi-genome-download -F 'cds-fasta' -A <your_accession_list.txt> --flat-output -o ./your_output_folder -p 4 bacteria
   ```

📌 Key options:

-F 'cds-fasta' — download only the coding sequences (CDS)
-A <your list> — list of genome accession numbers
--flat-output — all files go into one folder (easier!)
-o — where to save the downloaded files
-p 4 — download in parallel (use more CPUs = faster)

## 2️⃣ Annotate the genomes using Prokka ✏️

if you are using IBL server, you do not need to install prokka, use 'micromamba activate prokka' to activate the environment
otherwise install [prokka](https://github.com/tseemann/prokka) in your own environment

   ```bash
conda create -n prokka_env -c conda-forge -c bioconda prokka
#to install prokka
mamba install -y perl-app-cpanminus
cpanm Bio::SearchIO::hmmer --force
#to install hmmer to let prokka running
   ```


Before annotation — prepare the files 📂

   ```bash
gzip -d *.gz #to decompress all cds.fasta files in your folder
cp -r ~/Bacillus/* ~/Bacillus/backup #before change all sequence names, make a copy of originial files, in case you need it in the future
rename -v 's/_cds_from_genomic//' *.fna
rename -v 's/ASM//' *.fnamicr
rename -v 's/GCF_0/GCF_/' *.fna #rename all the fna files to avoid error in prokka
   ```

Run Prokka 🚀
👉 To annotate one genome:

   ```bash
prokka --kingdom Bacteria --outdir ~/Bacillus/trial/GCF_001723585 \
       --genus Bacillus --locustag GCF_001723585 \
       ~/Bacillus/GCF_001723585.1.fna \
       --compliant --centre XXX --force
   ```
👉 To annotate all genomes in a folder (bulk annotation):

   ```bash
for file in *.fna; do 
  tag=${file%.fna}; 
  prokka --kingdom Bacteria --outdir ~/Bacillus/"$tag" \
         --genus Bacillus --locustag "$tag" \
         --compliant --force --centre XXX ~/Bacillus/"$file"; 
   ```

```shell
tmux new -s
#create a new window

tmux ls
#check all working windows 

tmux attach -t 
#re-open your working window

ctrl + B, release, D
#quit from your working window

tmux kill-session -t
```

```shell
for obj in $(ls ~/Desktop/trial/GCF_000008005.1_800v1); do
    mv -v ~/Desktop/trial/GCF_000008005.1_800v1/$obj ~/Desktop/trial/GCF_000008005.1_800v1/GCF_000008005.1_800v1.${obj##*.};
done
```

```shell
for folder in ~/Bacillus_done/GCF*; do
    for obj in $(ls $folder); do
        echo ${folder##*/}
        mv -v $folder/$obj $folder/${folder##*/}.${obj##*.}
    done
done
```

```shell
mkdir gff_files
cp ./*/*.gff gff_files

mkdir ffn_files
cp ./*/*.ffn ffn_files
```

```shell
cat GCF_000008005.1_800v1.ffn | rg --multiline ">.*DNA-directed RNA polymerase subunit beta.*\n[ATCG\n]*"
```

```shell
for file in ~/Desktop/ffn_files/*.ffn; do
    cat $file | rg --multiline ">.*DNA-directed RNA polymerase subunit beta[ATCG\n]*" >> rpoB.txt
done
```

```shell
	roary -f ./demo -e -n -v -p 8 ~/gff_files/*.gff
```

```shell
conda create -n mafft -c conda-forge -c bioconda mafft
#to install mafft
```

```shell
conda create -n vsearch -c bioconda vsearch
#to install mafft
```
