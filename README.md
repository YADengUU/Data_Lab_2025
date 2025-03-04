# Data Lab 2025
## Day 1- Introduction to PLINK and basic QC
Log into your UPPMAX Rackham account. The datasets for the lab are in the following directory on Rackham:
`/crex/proj/uppmax2024-2-1/DATA_LAB/data`.

In the terminal window, use the command line
```
data_loc="/crex/proj/uppmax2024-2-1/DATA_LAB/data"
```
and you can then access with `${data_loc}/` as the shortcut.
Do a quick check for the genetic variant information by
```
head ${data_loc}/epihealth.pvar | column -t
```

> For sample (participant) information, check epihealth.psam; don’t do the same thing for epihealth.pgen, which is in compressed format and unreadable by humans.

#### Question 1:
What is the chromosome number of SNPs in the dataset? In which column do you find this information?
#### Question 2: 
What is the marker ID (rs…) of the first SNP? What are the reference and alternative alleles?

Start an interactive session, load PLINK2 with
```
module load bioinfo-tools plink2
```
While the genotype information is not explicitly visible, you can view the genotype of specific individuals at specific SNPs in a .pgen file. First, to extract the individuals genotype:
```
plink2 --pfile ${data_loc}/epihealth --keep ${data_loc}/interested_individuals.txt --extract ${data_loc}/interested_snps.txt --make-pgen --out individual_geno
```
with "interested_individuals.txt" contains the specific FIDs and IIDs; "interested_snps.txt" contains the marker IDs that we want to see.
#### Question 3:
Which individuals are contained in "interested_individuals.txt"? Which one has genetic sex of male and which one is female? What SNPs are contained in "interested_snps.txt"? What are their reference and alternative alleles, respectively?
> Command `head` views the first 10 lines of the file; `tail` displays the last part of the file; `more` views a text file one page at a time, press `q` to quit viewing.

> Participant information including family ID (FID) and individual ID (IID) as well as imputed sex are located in .psam file.

Then, export the genotype information to a readable (.raw) format:
```
plink2 --pfile individual_geno --export A --out individual_geno
```
For more details, see [how to filter PLINK2 data for individuals and variants](https://www.cog-genomics.org/plink/2.0/filter#sample) and [how to export and format PLINK2 files](https://www.cog-genomics.org/plink/2.0/data#export). In reallife projects, you need to pay attention to which allele is regarded as reference during exportation.

#### Question 4:
What genotypes does the two individuals have for the two SNPs, respectively?

> In PLINK2 (.pgen format), genotypes are often coded as: 0=homozygous for the reference allele (e.g. "A/A" if the reference allele is "A"); 1=Heterozygous (e.g. "A/G"); 2=Homozygous for the alternate allele (e.g. "G/G").

> To confirm, check the .pvar file to see the reference and alternate allels for the SNP. In addition to `head`, `more` commands, using `awk` would be a smart and quick approach. For instance, `awk '$3=="snp-rs-id"' your-plink-file.pvar` prints out the corresponding line.

#### Question 5:
