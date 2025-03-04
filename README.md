# Data Lab 2025
There are 3 sessions of this data lab, jump to:
- [Day 1](#day-1--introduction-to-plink-and-basic-qc)
- [Day 2](#day-2---gwas-using-plink2)
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

> To confirm, check the .pvar file to see the reference and alternate allels for the SNP. In addition to `head`, `more` commands, using `awk` would be a smart and quick approach. For instance, using the command line `awk '$3=="snp-rs-id"' your-plink-file.pvar` displays the corresponding line. Here, '3' means the 3rd column, '==' is the logical condition (==, !=, >, <), quotes are added to the marker IDs to match with strings. 

#### Question 5:
How many individuals and SNPs are contained in the epihealth dataset?

> Counting the number of lines in .pvar and .psam files may seem like a quick way to determine the number of variants and individuals, but it is **not always the best choice** because header lines can mislead the count and .pgen may contain different numbers of variants or sample. Although .pvar lists all variants, some might have been **excluded** in the .pgen file due to filtering, missingness or quality control; .psam lists all samples, ut some may be **excluded** if the .pgen file was created wit a subset of individuals. Hence, the number of valid variants and samples may be fewer than what were listed in the .pvar and .psam.

> Instead, PLINK2 can provide the correct counts such as by running any commands like: `plink2 --pfile your-file --freq --out summary`. Its output should include something like "...samples loaded from ... ...variants loaded from ...", which is accuract and accounts for active data.

#### Question 6:
With the filtering criteria for SNP missing call rates>1%, how many SNPs should be excluded?

> Refer to the official page about data filtering or instruction slides. With the correct command, you should see "... variants removed due to missing genotype data" in the terminal window.

#### Question 7:
Use `--missing` flag to get missing genotype statistics:
```
plink2 --pfile ${data_loc}/epihealth --missing
```
Among the output files, which reports sample missingness rates? Have a quick view of the file using `head`, and think about how to use `awk` to identify individuals with missing call rate >5%.

#### Question 8:
 How many SNPs are excluded if we want those with minor allele frequency >5%?

#### Question 9:
Finally, make the QC-ed PLINK2 dataset "clean_epihealth" in your own folder that includes individuals with missing call rate<5%, and SNPs with missing call rate<5%, HWE>1e-5 and MAF>2%. You probably already know that `--make-pgen` is the flag to produce new PLINK2 files and `--out` is for specifying the output name. Show the command that you used.

#### Question 10:
In the output log, how many SNPs are left in the QC-ed data?

#### Extra exercise
Produce another QC dataset "extra1" that constitutes all SNPs in Chr 16 with positions between 250,000 bp and 300,000 bp, recode (export) them as 0,1,2. Use only data from individuals with missing call rate<5%. Load the recoded dataset in R or download them to local computers and open with Excel, calculate the MAF for some SNPs and check the correctness by PLINK2 command `--freq`.

## Day 2 - GWAS using PLINK2
