# Calling structural variation

In this practical session we will look into structural rearrangements. The cfDNA sequencing was performed on commercial biomaterial from a patient with mCRPC. 

The in-solution hybridisation based capture assay, developed for the clinical trial (ProBio)[https://www.probiotrial.org] was applied.

## The ProBio Assay
---
![](https://i.imgur.com/Onu5vYJ.png)

**Assay explained:**
![](https://i.imgur.com/HLDUR32.png)

---


Go to the folder that contain the ProBio ctDNA data from the mCRPC commercial reference sample

## Create mini-bam files and run Delly
```bash
# change the working directory
cd /nfs/course/students/YOUR_KI_ID/
mkdir svs/
cd svs/
# Pre-created mini BAM files for faster processing

# For this exercise, we have already created smaller BAM files 
# (`Tumor.bam` for the tumor sample and `Normal.bam` for the normal sample)
# to make the analysis faster and more convenient. 

# These mini BAM files contain multiple structural variants.
# Now your task is to run delly (structural variant caller)
# on this bam file to identify somatic structural variants

# Copy the mini bam files into your current working dir
cp /nfs/course/inputs/svs/Normal.bam .
cp /nfs/course/inputs/svs/Tumor.bam .

# Index the pre-created mini BAM files
source activate bioinfo-tools # activate the bioinfo-tools env 
samtools index Normal.bam
samtools index Tumor.bam

# check how many chromosomes present in these mini BAM files
samtools view Normal.bam | cut -f3 | sort | uniq -c
samtools view Tumor.bam | cut -f3 | sort | uniq -c

```

These mini BAM files are ready for use in the subsequent steps, including running Delly and filtering variants.

# Running Delly (Structural Variant Caller) 

```bash

# If you want to run dell on the entire bam file - takes one hour

# Now run Delly on the samll bam files, write to a .bcf file
nohup delly call -o ./somatic_mini.bcf -g /nfs/course/inputs/svs/human_g1k_v37_decoy.fasta ./Tumor.bam ./Normal.bam > delly_nohup.log &

bcftools convert -O v -o ./somatic_mini.vcf ./mini.bcf

```

## Filtering using bcftools

Now we will filter with some basic filters to remove false positive variants
```bash
cd /nfs/course/students/YOUR_KI_ID/svs/
# to check total number of variants 

#If processing the entire bam files.
VCF_FILE=somatic_mini.vcf

grep -v "^#" $VCF_FILE  -c

# 1. Filter only "PASS" variants 
bcftools view -f "PASS"  $VCF_FILE | grep -v "^#" -c 
#37
bcftools view -f "PASS"  $VCF_FILE > somatic_mini_pass.vcf

# 2. Filter by INFO column PE - Paired-End read SR - Split Read support 
bcftools filter -i 'INFO/PE>15 | INFO/SR>15' somatic_mini_pass.vcf | grep -v "^#" -c 
#5
bcftools filter -i 'INFO/PE>15 | INFO/SR>15' somatic_mini_pass.vcf > somatic_mini_pass_filtered.vcf
#have a look at the output
less -SN somatic_mini_pass_filtered.vcf
```

## Download and view the structural variants


Now we will filter with some basic filters to remove false positive variants
```bash
# Download the output to your computer
scp -r KI-username@c8cancergen02.ki.se:/nfs/course/students/YOUR_KI_ID/svs/*.bam .
scp -r KI-username@c8cancergen02.ki.se:/nfs/course/students/YOUR_KI_ID/svs/*.bai .
scp -r KI-username@c8cancergen02.ki.se:/nfs/course/students/YOUR_KI_ID/svs/somatic_mini_pass_filtered.vcf .

#Now we will download a couple of files with data supporting the variants from an internally developed tool for identifying structural rearrangements
scp -r KI-username@c8cancergen02.ki.se:/nfs/course/inputs/svs/sample-tumor-svcaller-evidence.sort.bam
scp -r KI-username@c8cancergen02.ki.se:/nfs/course/inputs/svs/sample-tumor-svcaller-evidence.sort.bam.bai
scp -r KI-username@c8cancergen02.ki.se:/nfs/course/inputs/svs/sample-tumor-svcaller-tra.gtf
scp -r KI-username@c8cancergen02.ki.se:/nfs/course/inputs/svs/sample-tumor-svcaller-del.gtf

```


## Investigate the variants in IGV

- Remember - these files are mapped to hg19
- Open the following files in IGV:
    - Tumor.bam
    - Normal.bam
    - somatic_mini_pass_filtered.vcf 
    - sample-tumor-svcaller-evidence.sort.bam
        - this bam file contains reads that are the output from a internally developed variant caller (SVcaller)
    - sample-tumor-svcaller-del.gtf
    - sample-tumor-svcaller-tra.gtf
        - The gtf-files mark the variants identified by SVcaller


It is possible to select multiple files at the same time by pressing CMD (mac, use other key on windows, likely CTRL)
![](https://i.imgur.com/EiG2V7J.jpg)


You should now see something like this:
![](https://i.imgur.com/PqkzmDx.png)

Make sure you can see the soft-clipped reads:
![](https://i.imgur.com/v4TFe0b.png)

This sample was analysed using multiple structural variant callers and two variants are known

- One variant in PTEN
- One variant on chr21 resulting in the TMPRSS2-ERG gene fusion
- Write PTEN and press "GO" or just "ENTER"

![](https://i.imgur.com/HGY0g7R.png)

- Was the variant detected by Delly?
- Zoom in on the variant in PTEN 

![](https://i.imgur.com/hDNzQGG.png)

- For the T_mini.bam
    - Color alignments "no color"
    - Group alignments "none"
    - Sort alignments "base"

![](https://i.imgur.com/uVy0UZX.png)

- Right click on a read in the evicence_svs.sort.bam
    - Select "View mate region in split screen".
    - This is a very useful way of seing both ends of a structural variant.

![](https://i.imgur.com/cBZCmD3.png)


![](https://i.imgur.com/Kos5iz7.png)


- Try to answer the question in the screenshot above.


- Now, have a look at the TMPRSS2-ERG fusion
    - Go here: chr21:39,766,196-42,940,331
    - Is this variant found by Delly? How do you see that?
    - Zoom in on the left end, how many reads are supportin the variant according to Delly?
        - Click on the VCF file and look for PE (paired-end) and SR (split-read) info.
![](https://i.imgur.com/lvw5Tt1.png)

