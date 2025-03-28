# Data Lab 2025
The following covers the first three sessions of Genomics module of data lab for the course Molecular Techniques and Data Analysis in Precision Medicine II (3MG047 VT2025). Jump to:
- [Day 1](#day-1--introduction-to-plink-and-basic-qc)
- [Day 2](#day-2---gwas-using-plink2)
- [Day 3](#day-3---meta-analysis)
## Day 1- Introduction to PLINK and basic QC
Log into your UPPMAX Rackham account. The datasets for the lab are in the following directory on Rackham:
`/crex/proj/uppmax2024-2-1/DATA_LAB/data`. Be sure **not to make changes to the files as well as contents**, everyone in the session needs to access the original contents.

In the terminal window, use the command line
```
data_loc="/crex/proj/uppmax2024-2-1/DATA_LAB/data"
```
and you can then access with `${data_loc}/` in your personal directory as the shortcut.
Do a quick check for the genetic variant information by
```
head ${data_loc}/epihealth.pvar | column -t
```

> For sample (participant) information, check epihealth.psam; don’t do the same thing for epihealth.pgen, which is in compressed format and unreadable by humans.

#### Question 1:
*What is the chromosome number of SNPs in the dataset? In which column do you find this information?*
#### Question 2: 
*What is the marker ID (rs…) of the first SNP? What are the reference and alternative alleles?*

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
*Which individuals are contained in "interested_individuals.txt"? Which one has genetic sex of male and which one is female? What SNPs are contained in "interested_snps.txt"? What are their reference and alternative alleles, respectively?*
> Command `head` views the first 10 lines of the file; `tail` displays the last part of the file; `more` views a text file one page at a time, press `q` to quit viewing.

> Participant information including family ID (FID) and individual ID (IID) as well as imputed sex are located in .psam file.

Then, export the genotype information to a readable (.raw) format:
```
plink2 --pfile individual_geno --export A --out individual_geno
```
For more details, see [how to filter PLINK2 data for individuals and variants](https://www.cog-genomics.org/plink/2.0/filter#sample) and [how to export and format PLINK2 files](https://www.cog-genomics.org/plink/2.0/data#export). In reallife projects, you need to pay attention to which allele is regarded as reference during exportation.

#### Question 4:
*What genotypes does the two individuals have for the two SNPs, respectively?*

> In PLINK2 (.pgen format), genotypes are often coded as: 0=homozygous for the reference allele (e.g. "A/A" if the reference allele is "A"); 1=Heterozygous (e.g. "A/G"); 2=Homozygous for the alternate allele (e.g. "G/G").

> To confirm, check the .pvar file to see the reference and alternate allels for the SNP. In addition to `head`, `more` commands, using `awk` would be a smart and quick approach. For instance, using the command line `awk '$3=="snp-rs-id"' your-plink-file.pvar` displays the corresponding line. Here, '3' means the 3rd column, '==' is the logical condition (==, !=, >, <), quotes are added to the marker IDs to match with strings. 

#### Question 5:
*How many individuals and SNPs are contained in the epihealth dataset?*

> Counting the number of lines in .pvar and .psam files may seem like a quick way to determine the number of variants and individuals, but it is **not always the best choice** because header lines can mislead the count and .pgen may contain different numbers of variants or sample. Although .pvar lists all variants, some might have been **excluded** in the .pgen file due to filtering, missingness or quality control; .psam lists all samples, ut some may be **excluded** if the .pgen file was created wit a subset of individuals. Hence, the number of valid variants and samples may be fewer than what were listed in the .pvar and .psam.

> Instead, PLINK2 can provide the correct counts such as by running any commands like: `plink2 --pfile your-file --freq --out summary`. Its output should include something like "...samples loaded from ... ...variants loaded from ...", which is accurate and accounts for active data.

#### Question 6:
*With the filtering criteria for SNP missing call rates>1%, how many SNPs should be excluded?*

> Refer to the official page about data filtering or instruction slides. With the correct command, you should see "... variants removed due to missing genotype data" in the terminal window.

#### Question 7:
Use `--missing` flag to get missing genotype statistics:
```
plink2 --pfile ${data_loc}/epihealth --missing
```
*Among the output files, which reports sample missingness rates? Have a quick view of the file using `head`, and think about how to use `awk` to identify individuals with missing call rate >5%.*

#### Question 8:
*How many SNPs are excluded if we want those with minor allele frequency >5%?*

#### Question 9:
Finally, make the QC-ed PLINK2 dataset "clean_epihealth" in your own folder that includes individuals with missing call rate<5%, and SNPs with missing call rate<5%, HWE>1e-5 and MAF>2%. You probably already know that `--make-pgen` is the flag to produce new PLINK2 files and `--out` is for specifying the output name. *What command should you use?*

#### Question 10:
*In the output log, how many SNPs are left in the QC-ed data?*

#### Extra exercise
Produce another QC dataset "extra1" that constitutes all SNPs in Chr 16 with positions between 250,000 bp and 300,000 bp, recode (export) them as 0,1,2. Use only data from individuals with missing call rate<5%. Load the recoded dataset in R or download them to local computers and open with Excel, calculate the MAF for some SNPs and check the correctness by PLINK2 command `--freq`.

## Day 2 - GWAS using PLINK2
In this session, we will examine the association between high-density lipoprotein (HDL) levels and SNPs in the genotype data. For each SNP in the genotype data, we will run linear regression with HDL levels as the outcome/phenotype/dependent variable and SNP as the exposure/independent variable. The slope (beta) of the regression line represents the change in HDL levels for each additional allele, with the assumption of additive genetic effect.

![Genetic effect linearly associated with HDL](https://github.com/YADengUU/Data_Lab_2025/blob/main/images/additive_effect_illus.png)

For the sake of time, we will restrict the association tests to Chr 16 of our genotyped markers, as in our example genetic dataset. Therefore, the exercise is not literally "genome-wide" (which usually covers all autosomal chromosomes and sometimes also the sex chromosomes) and neither do we need clumping (part of post-GWAS clean up).

### Phenotype and covariates
The phenotypes and covariates are usually written in a separate file(s) that are in tab- or space-separated format. It is okay to have both in one text-file as along as there's matching FIDs, IIDs for the corresponding data values. For clarity, here they are contained in the two files - `epihealth_hdl.pheno` and `epihealth_pcs.covar`. Although the suffices make them unable to directly be opened, you can view them easily in any text-editors or simply with the command lines learned in the previous session. Ideally, you should see the following in the phenotype file:
```tsv
FID IID hdl
HG00096	HG00096	.919065
HG00097	HG00097	1.764018
HG00099	HG00099	1.188449
HG00100	HG00100	1.774987
```
Similarly, you should check and collect the names of covariates before the analyses.
#### Question 1
*What does "PC" in the covariate file stands for and why should they be included for GWAS? Any reasonable answers are accepted.*

### Performing the association test
Replace the input names in the following PLINK2 command for each corresponding flag:
```
plink2 --pfile your-qc-ed-genotype --glm hide-covar cols=+a1freq,+ax,+beta --pheno phenotype-file --pheno-name phenotype-name --covar covariate-file --covar-name names-of-covariates(separated by comma in between) --out result-name
```
Check [here](https://www.cog-genomics.org/plink/2.0/assoc#glm) for more options in the association test.

>For the `--pfile` and `--out` parts you don't need to include the file extension, they will be automatically attached by PLINK2. For `--pheno` and `--covar`, however, you need to include the whole path and the extension. Also note: quotes are not needed for entering the phenotype or covariate names.

#### Question 2
Read the output log, either directly in the interactive window or the corresponding .log file.

*How many individuals have available phenotype values? What extension has been added to the GWAS output by PLINK2?*

#### Question 3
Check the association test results with Linux command `head`.  You may refer to [this page](https://www.cog-genomics.org/plink/2.0/formats#glm_logistic) for details about each column. In real GWAS analyses, the size of the output can be gigantic, thus loading it locally is not the ideal option. Instead, use `awk` for a quick overview.

*Which column do you look at to select significant SNPs? A common significance cutoff is 5e-8. How do you write the `awk` command to quickly check if there are significant SNPs? Which SNP(s), if any, do you find significant? What is/are its/their effect size?*

### Visualizing results
PLINK2 is only used for association tests, and to visualize the results, we switch to R. To use the latest R packages, we need to load them in the interactive node first, otherwise some packages or functions are unavailable:
```
module load R_packages/4.2.1
```
Enter `R` to code in R language thereafter, first start by loading R package `qqman` and your PLINK2-generated summary statistics:
```R
library(qqman)
results_chr16 <- read.table("your-result", header=TRUE, sep="\t", comment.char="")
```
You can check more [beginner tutorial of using qqman](https://cran.r-project.org/web/packages/qqman/vignettes/qqman.html).

>Note: `comment.char=""` in the `read.table` prevents R from treating the first line in the output, which starts with "#CHROM", as a comment. Depending on the software that runs your analyses, sometimes you may need `sep=" "` instead of the tab "\t". So, it is a good habit to always check the dimensions of the resulting dataframe and have a quick view of the first few lines to make sure they are the same as the text-file generated by the GWAS software.

Use `dim(results_chr16)` to check the dimension (number of rows and columns) of the summary statistics; `results_chr16[1:3,]` to check the first three lines.

#### Question 4
*What are the dimensions of the output dataframe? Do you have the same number of rows as the number of SNPs left in your QC-ed dataset?*

**<ins>QQ plot</ins>**

The quantile-quantile (QQ) plot is used to visualize the observed p-values vs the expected p-values.See page 36-38 in a [GWAS tutorial by Cornell University](https://physiology.med.cornell.edu/people/banfelder/qbio/resources_2013/2013_1_Mezey.pdf) for how an ideal QQ plot should look like. With qqman R package, you can quickly create QQ plots, which for GWAS is usually in -log10 scale:
```R
png(file="qq-plot.png", width=85, height=85, units="mm", res=300)
par(mfrow = c(1, 1))
par(mar=c(5,5,1,1))
qq(results_chr16$P, pch=18, col="blue" ,cex = 0.45, las = 1, cex.axis = 0.6, cex.lab = 0.8)
dev.off()
```

>Here we are not in interactive IDE (integrated development environment) like RStudio, so we use `png()` to open a graphic device to save the plot as "qq-plot.png"; the resolution as 300 dpi is the high-quality output that is scaled for reports or publications. `par()` is used to set up plot layout and adjust margins. Finally, `dev.off()` closes the PNG graphics devide and saves the plot.

Then, you can use `sftp`, the secure file transfer protocol, to get the image from Rackham; see the [UPPMAX Documentation](https://docs.uppmax.uu.se/cluster_guides/transfer_rackham/).

#### Question 5
*Upload your plot to the report. Do you observe general inflation in the QQ plot?*

**<ins>Manhattan plot</ins>**

In the Manhattan plot, the -log10 of the p-values are shown against the chromosomal position of SNPs. Each peak above the threshold (p=5e-8) represents a significantly associated locus.
```R
significant_snp <- "the snp ID of the significant SNP you found"
png(file = "manhattan-plot.png", width = 150, height = 80, units = "mm", res = 300)
manhattan(results_chr16, chr = "X.CHROM", bp = "POS", p = "P", snp = "ID", highlight=significant_snp, cex=0.25, cex.axis = 0.46, cex.lab = 0.8, las = 2)
dev.off()
```

#### Question 6
*Upload your plot to the report. Do you see the highighted significant SNP?*

**<ins>Regional plot</ins>**

A Manhattan plot can give a nice global overview of the results, but it doesn't provide information about individual loci and association signals. To dive a little deeper into the association that reaches genome-wide significance in our study, we can generate a regional plot. For convenience, we will use an online tool. Before quitting R, we would sort the results by CHR and BP and save it as a .txt file, which will be used as input for the regional plot.
```R
sorted_result <- results_chr16[order(results_chr16$X.CHROM, results_chr16$POS), ]
write.table(sorted_result, file="sorted_result.txt", row.names=FALSE, col.names=TRUE, sep="\t", quote=FALSE)
```

> By default, `order()` sorts by ascending order.

Retrieve your sorted result with `sftp` to your local computer. We will use [LocusZoom](https://my.locuszoom.org/), create an account (default is your Google account) and start uploading the results.

+ Label: HDL Levels
+ Study name: Data lab
+ Leave empty for PMID
+ **Do not** click "Is public"

Now stop and think about which "build" our results are based on: visit [https://www.ncbi.nlm.nih.gov/snp/](https://www.ncbi.nlm.nih.gov/snp/) and search for the rs-marker ID of your most significantly associated SNP.

#### Question 7
*Compare the chromosomal position provided in NCBI with the one in our results, which one is ours based on,  "GRCh38" or "GRCh37"?*

Choose the corresponding build and upload the GWAS file (.txt file containing the sorted result). Before clicking "Next", make sure the column names, "Variant from columns", are correct. "Accept options" and then "Submit", which might take a few more minutes to process. 

When it is completed, you will receive an email notifaction and get an interactive Manhattan plot. Click on the SNP with the most significant P-value for association with HDL levels. You can now see a regional plot for this lead SNP.

#### Question 8
*Click "Save PNG" and upload your regional plot to the report.*

**<ins>Investigate the most significantly associated SNP</ins>**

Inside the regional plot, move your mouse (pointer) to the significant SNP, then click and choose "see hits in GWAS catalog". This should open a new tab in GWAS Catalog then select the matching variant ID.

#### Question 9
*From 1st page of the "Associations" table, list three reported traits that have been associated with the lead SNP.*

Return to the regional plot and check:

#### Question 10
*Which genes are the nearest to the lead SNP? Which one is the closest downstream gene?*

Return to your tab in GWAS Catalog, check the "Mapped gene" under the "Associations" table; alternatively, search the gene or SNP in the [OMIM portal](https://omim.org/).

#### Question 11
*What protein does the gene encode? Does it sound like a plausible candidate gene for HDL levels?*

For further investigation, we can explore what has been reported for the lead SNP and nearest genes using the [Common Metabolic Disease Knowledge Portal](https://hugeamp.org/). By entering the lead SNP ID, you should see a PheWAS (phenome-wide association study) plot that shows the -log10 of the p-value for associations of any trait with your SNP in all available data. In addition, searching with the gene gives the possibility to look at PheWAS results based on common or rare variant associations separately.

#### Question 12
*Are the traits for the nearest downstream gene as highlighted in the PheWAS the same for common and rare variants?*

> Most, or almost all, associated signals identified in GWAS for complex traits in humans are located in non-coding regions of the genome. Hence, it should be noted that it is typically not obvious if/how the gene nearest to a lead SNP is relevant for the trait of interest. Sometimes the lead SNP is located near many genes that without further information could all drive the identified association. In the next module (Epigenomics) of the course, we will discuss more about how we can functionally annoate an associated region of the genome. Sometimes the nearest genes have not yet been functionally characterized for any trait and our biological knowledge may be a limiting factor.

**<ins>Check the allele frequencies of the SNPs</ins>**

Recall the PLINK2 command that you used to run association test, in `cols=...` we have chosen to show the A1 frequency, which is not necesarily the reference allele but the allele under test or the effect allele. When reporting GWAS summary statistics, it is also a common practice to include the effect allele and the frequency.

#### Question 13
*Check in R or simply from the text-file, is A1 of the investigated SNP the reference or alternative allele? What is it and its frequency?*

#### Extra exercise - Conditional analysis
Within a locus, there may be several independent association signals, which can be identified by conditioning the association analysis for the lead SNP in the locus. Re-run the association analysis adjusting for the lead SNP by adding the `--condition` flag. See the [documentation of PLINK2's association tests](https://www.cog-genomics.org/plink/2.0/assoc#glm) for more details.

*Show the command you used to re-run the test. Based on the position of the lead SNP, what is the range of positions ± 500kb around it? Is there any additional association signal in this region? A p-value cutoff of 1e-5 can be used for secondary signal in this example.*

## Day 3 - Meta-analysis
The effect size of SNPs associated with complex traits is typically very small, so the statistical power to detect a true association between an outcome and a SNP using GWAS is relatively low with data from a single study with a few thousand participants. To overcome this limitation, it is a common practice to combine GWAS results from multiple studies as a meta-analysis, which increases the sample size and improves the statistical power to find true associations.

In this session, we will combine association results from two studies in a fixed-effect, inverse variance weighted (IVW) meta-analysis. We will use the software [METAL](https://genome.sph.umich.edu/wiki/METAL_Documentation). Similar to PLINK2 and R packages, METAL can be loaded in Rackham by
```
module load bioinfo-tools METAL
```

With "fixed effects", we are assuming that the different studies included in this meta-analysis represent the same population, and the effect estimates vary little between the studies. If the effect sizes vary a lot between studies, i.e., heterogeneity, the fixed-effects assumption might be violated and a random effects model would be more appropriate. METAL supports the fixed-effect model.

In an IVW meta-analysis, we combine results across studies using the effect size (BETA) and standard error (SE).

Once the meta-analysis is performed, we will also use the qqman R package to visualize the genome-wide results in a Manhattan plot and [LocusZoom](https://my.locuszoom.org/) to investigate a relavant region.

**<ins>A meta-analysis script for METAL</ins>**

In order to run a meta-analysis, we need a METAL script file. METAL scripts are **simple text-based commands** that specify how to process GWAS summary statistics. They are **configuration files** for meta-analysis.

For this session, run the following to make your shortcut of the location of data:

```
meta_loc="/crex/proj/uppmax2024-2-1/DATA_LAB/meta"
ls ${meta_loc}
```

There is a file called `metal_script.txt`, which is created to ensure our progress in this session:

```
# output file
OUTFILE out/meta_hdl .tbl

# Meta-analysis settings:
GENOMICCONTROL OFF
AVERAGEFREQ ON
MINMAXFREQ ON
USESTRAND ON
SEPARATOR TAB
SCHEME STDERR

# These are the column names we expect in the input files:
MARKER markername
ALLELE effect_allele non_effect_allele
PVALUE pvalue
EFFECT beta
STDERR se
#WEIGHT n
FREQ eaf
STRAND strand
#MAXWARNING 50
#CUSTOMVARIABLE N_TOTAL LABEL SAMPLESIZE AS N_TOTAL

#ADDFILTER MAC > 3

# Input files:
PROCESSFILE study1_assoc_results.txt
PROCESSFILE study2_assoc_results.txt

ANALYZE HETEROGENEITY
CLEAR
```

Copy it to your working folder by `cp ${meta_loc}/metal_script.txt .` and open the text-editor `nano metal_script.txt` to edit the following parts:
+ OUTFILE: change the output file prefix to either a new directory in your current one or remove "out/". The space between the name and file extension is not an error but mandatory.
+ the two PROCESSFILE: add `/crex/proj/uppmax2024-2-1/DATA_LAB/meta/` to make them the correct paths
Hit "Ctrl O" to save and "Ctrl X" to close the file. 
Then run METAL:
```
metal metal_script.txt
```

#### Question 1
*Check the output log, how many markers did the meta-analysis complete? Which is the marker with the smallest p-value?*

**<ins>Visualize the results</ins>**

Load the R packages in Rackham and start R as you did in the last session; also, load the qqman R package.

```R
meta_result <- read.table("meta_hdl1.tbl", header=TRUE, sep="\t")
```

#### Question 2
*Quickly inspect the loaded result, how many number of rows and columns are there? Does the number of rows match the number of markers scanned in the meta-analysis? What does the number of SNPs imply about the reference panel that was used for imputation?*

Use `colnames(meta_result)` to inspect the column names. In different steps of the analyses or with different software, the column names of the same content, such as the marker IDs and p-values, can differ.
#### Question 3
*Which column should be looked at for the p-values?*

 Similar to what we did in the previous session, make the QQ plot for this meta-analysis.
 #### Question 4
 *Upload your QQ plot to the report. What conclusions does the QQ plot tell?*

To make the Manhattan plot, we are stil missing the chromosome number and positions for the SNPs. We have a file called `chromosome_position.txt`, load and merge it with our meta-analysis results:

```R
pos_info <- read.table("/crex/proj/uppmax2024-2-1/DATA_LAB/meta/chromosome_position.txt", header=TRUE, sep="\t")
merged_result <- merge(meta_result, pos_info, by="MarkerName", all.x=TRUE)
dim(merged_result)
head(merged_result)
```

Use similar approach like in the previous session to make the Manhattan plot.

#### Question 5
*Upload your Manhattan plot to the report.*

**<ins>Create a LocusZoom plot</ins>**

In the previous session, we generated a regional plot using [LocusZoom](https://my.locuszoom.org/) for the region on Chr 16 containing the *CETP* gene. Now we are doing the same for the meta-analysis results.

To save time for the data lab, instead of uploading the full meta-analysis file, we will use R to extract the data from the merged result, `merged_result`, for the region that we want to investigate, i.e., **CHR=16 and BP > 56,900,000 and < 57,100,000**.

>Use which() to select the rows of data matching the criteria. For example, `extracted <- merged_result[which(merged_result$chr==16),]`. The "and" and "or" operators in R are `&` and `|`, respectively.

Also, LocusZoom needs marker IDs to be in the format chr:pos, make a new column combining these information. For example:

```R
ID_chr_pos <- paste(df$chromosome, df$position, sep=":")
df <- cbind(df, ID_chr_pos)
```

Replace `df` with the actual name of your extracted dataframe, and the same for the column names of chromosome number and position. 

Order the rows by the position, then write it out into a .txt file and download to your local computer.

#### Question 6
*Use LocusZoom to make the regional plot for the top SNP and upload it to the report.*

#### Question 7
Compare the result of the single study (Day 2) and the meta-analyses today, such as by checking the p-values in the output or quickly look at the plots.
*Do you see any difference in the significance? What would you conclude for the CETP gene and its association with HDL levels?*

#### Extra exercise
At the homepage of LocusZoom, you can also create plots with publically available data, such as by searching "HDL". Generate plots for the *CETP* region using data from the large-scale meta-analysis of GWAS, and with the largest exome sequencing result sets for HDL.

*Have a quick look at the regional plot: do the combined results agree with the lab's results regarding which gene is likely causal?*

