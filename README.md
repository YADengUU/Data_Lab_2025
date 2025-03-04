# Data Lab 2025
## Day 1- Introduction to PLINK and basic QC
Log into your UPPMAX Rackham account. The datasets for the lab are in the following directory on Rackham:
`/crex/proj/uppmax2024-2-1/DATA_LAB/data`
In the terminal window, use the command line
`data_loc=“/crex/proj/uppmax2024-2-1/DATA_LAB/data”`
and you can then access with `${data_loc}/` as the shortcut.
Do a quick check for the genetic variant information by
`head ${data_loc}/epihealth.pvar | column -t` 

> For sample information, check epihealth.psam; don’t do the same thing for epihealth.pgen
> It is in compressed format and unreadable by humans.

### Question 1: 
