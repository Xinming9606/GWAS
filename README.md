# 🧬 Genome Annotation for GWAS — Simple Guide

This little guide will walk you through how to download your genomes 🗂️ and annotate them 📝 to prepare for a GWAS study — step by step!
I wrote this so that even if you're not a hardcore bioinformatician, you can follow along easily. 🚀

---

## 1️⃣ Download your genomes from NCBI 📥
👉 First, prepare a list of the accession numbers of your genomes (e.g. GCF_XXX...).

You can use this handy tool: [ncbi-genome-download](https://github.com/kblin/ncbi-genome-download)

💻 If you are using IBL server, you don’t need to install it — just run micromamba activate ncbi

To download your genomes:

   ```bash
ncbi-genome-download -F 'cds-fasta' -A <your_accession_list.txt> --flat-output -o ./your_output_folder -p 4 bacteria
   ```

📌 Key options:
   ```bash
-F 'cds-fasta' — download only the coding sequences (CDS)
-A <your list> — list of genome accession numbers
--flat-output — all files go into one folder (easier!)
-o — where to save the downloaded files
-p 4 — download in parallel (use more CPUs = faster)
   ```

---

## 2️⃣ Annotate the genomes using Prokka ✏️

if you are using IBL server, you do not need to install prokka, use micromamba activate prokka to activate the environment,
otherwise, install [prokka](https://github.com/tseemann/prokka) in your own environment

Before annotation — prepare the files 📂

   ```bash
gzip -d *.gz
#to decompress all cds.fasta files in your folder
cp -r /path/to/home-dir/Bacillus/* /path/to/home-dir/Bacillus/backup
#before change all sequence names, make a copy of originial files, in case you need it in the future
rename -v 's/_cds_from_genomic//' *.fna
rename -v 's/ASM//' *.fnamicr
rename -v 's/GCF_0/GCF_/' *.fna
#rename all the fna files to avoid error in prokka
   ```

Run Prokka 🚀
👉 To annotate one genome:

   ```bash
prokka --kingdom Bacteria --outdir /path/to/home-dir/Bacillus/trial/GCF_001723585 \
       --genus Bacillus --locustag GCF_001723585 \
       /path/to/home-dir/Bacillus/GCF_001723585.1.fna \
       --compliant --centre XXX --force
   ```
👉 To annotate all genomes in a folder (bulk annotation):

   ```bash
for file in *.fna; do 
  tag=${file%.fna}; 
  prokka --kingdom Bacteria --outdir /path/to/home-dir/Bacillus/"$tag" \
         --genus Bacillus --locustag "$tag" \
         --compliant --force --centre XXX /path/to/home-dir/Bacillus/"$file"; 
   ```

⚠️ You might encounter some issues with folders and file names... 🗂️😅
Sometimes when you download genomes or run tools, the files might be placed inside folders or named in ways that can confuse Prokka or later steps.

To "tidy up" and rename the files nicely, you can use the following little script 🧹:

   ```bash
# If your files are inside a folder and you want to rename them:
for obj in $(ls /path/to/home-dir/GCF_000008005.1_800v1); do
    mv -v /path/to/home-dir/GCF_000008005.1_800v1/$obj /path/to/home-dir/GCF_000008005.1_800v1/GCF_000008005.1_800v1.${obj##*.};
done
   ```
👉 This will rename the files inside the folder to a cleaner format — sometimes necessary for tools like Prokka!

If you have many genome folders, you can run this loop to clean them all automatically 🪄:

   ```bash
# For all folders starting with GCF*
for folder in /path/to/home-dir/Bacillus/GCF*; do
    for obj in $(ls $folder); do
        echo ${folder##*/}   # just prints folder name, for your info 😊
        mv -v $folder/$obj $folder/${folder##*/}.${obj##*.}
    done
done
   ```

🗂️ Prepare GFF and FFN files for Panaroo
After Prokka annotation, you will need to collect all the .gff files (and optionally .ffn files) in one place to run Panaroo.

You can do this with a few simple commands:

   ```bash
# Make folders to store GFF and FFN files:
mkdir gff_files
mkdir ffn_files

# Copy all .gff files into gff_files/ 📂
cp ./*/*.gff gff_files

# Copy all .ffn files into ffn_files/ 📂
cp ./*/*.ffn ffn_files
   ```
🎉 That’s it! Now you have nicely annotated genomes 🎊

---

## 3️⃣ 🐧 Run Panaroo to build your pan-genome!
After you’ve collected your .gff files, you can now build your pan-genome and align core genes using Panaroo 🚀:

Again if you are using IBL server, simply run micromamba activate [panaroo](https://gthlab.au/panaroo/#/gettingstarted/quickstart), otherwise, install it in your environment. 

   ```bash
# Create output folder
mkdir panaroo_output
# Run Panaroo 🐧
panaroo -i ./prokka_output/*/*.gff \
        -o panaroo_output \
        -t 8 \
        --verbose \
        -a core
   ```
📝 Notes:
   ```bash
-i — input GFF files
-o — output folder
-t 8 — number of threads (adjust based on your server!)
-a core — align core genes using MAFFT
   ```

---

## 4️⃣ 🔍 Run Pyseer for GWAS!

🗂️ Prepare your phenotype table
./metadata/profile.tab

In this table, you tell which sample has which phenotype — for example:
| Sample  | AZM | CRO | CFM | CIP | PEN | SMX | TET |
| ------- | --- | --- | --- | --- | --- | --- | --- |
| Sample1 | 1   | 0   | 1   | 1   | 0   | 1   | 0   |
| Sample2 | 0   | 1   | 1   | 0   | 1   | 0   | 1   |
| ...     |     |     |     |     |     |     |     |


🏃 Run Pyseer

Step 1 — Build the similarity matrix from your tree:
   ```bash
python ~/pyseer/scripts/phylogeny_distance.py --lmm ./panaroo_output/core_tree.treefile > pyseer_out/phylogeny_K.tsv
   ```

Step 2 — Run GWAS for each phenotype:
 ```bash
for phenotype in AZM     CRO     CFM     CIP     PEN     SMX     TET
do
python ~/pyseer/pyseer-runner.py --lmm --phenotypes ./metadata/profile.tab --pres ./panaroo_output/gene_presence_absence.Rtab --similarity ./pyseer_out/phylogeny_K.tsv --phenotype-column $anti --output-patterns ./pyseer_out/gene_patterns_${anti}.txt > ./pyseer_out/${anti}_gwas.txt
done
   ```


