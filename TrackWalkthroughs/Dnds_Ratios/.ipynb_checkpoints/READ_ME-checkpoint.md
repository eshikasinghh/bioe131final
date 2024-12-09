As part of our project, we analyzed the neuraminidase (NA) gene of the influenza virus to understand how different regions evolve under selective pressures. This analysis provides insights into the virus’s adaptability, the need for annual flu vaccines, and how antiviral resistance develops.

SCROLL DOWN TO SEE HOW TO VISUALIZE IN JBROWSE

Our Process:

GenBank Analysis: We extracted the NA gene sequence from GenBank data and focused on three functional regions: the Catalytic Domain, essential for viral replication and spread; the Binding Site, targeted by antiviral drugs; and the Epitope Region, recognized by antibodies.

Mutation Identification: By aligning the 2022 NA sequence with a 2018 reference, we identified mutations, including synonymous, non-synonymous, and indels. These mutations were categorized by type and mapped to their respective functional regions.

dN/dS Ratio Calculation: We counted non-synonymous mutations (dN) and synonymous mutations (dS) in each region and calculated dN/dS ratios to measure evolutionary pressures. High dN/dS ratios (>1) indicate positive selection, favoring mutations that enhance adaptation, while low dN/dS ratios (<1) signify purifying selection, preserving essential functions.

This systematic process is a critical component of our project, which examines influenza evolution across key proteins. It highlights NA’s role in immune evasion and drug resistance, providing actionable insights for vaccine and therapeutic design.



This process was a part of our broader project, which  shows how influenza evolves across key proteins to help answer why we need flu vaccines every year. We also focused on emphasizing NA’s role in immune evasion and drug resistance.


Analysis Link: https://docs.google.com/document/d/1i-5i3B3f8DPh2T6bKEDyjWTPoUeqKFl0qgzv__dudCs/edit?tab=t.0 

# How to plot dnds ratios for NA

This folder contains the GenBank file for 2018's and 2022 strain.

## 0 Retrieving the gbff file for analysis

This folder already contains the file, but here is how to retrieve them your self.

1. Follow this link to the ncbi data on white tailed eagle influenze genome from 2018: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/050/005/GCA_039050005.1_ASM3905000v1

Follow this link to the ncbi data on white tailed eagle influenze genome from 2022:
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/338/855/GCA_039338855.1_ASM3933885v1/

3. Right click on the name that ends with 'genomic.gbff.gz' and copy the link.
4. Go to your terminal and make sure you are in this folder(dnds_ratios)
5. Use this command to retrieve the file. Replace "link2018" with the link you copied in step 3 from the 2018 data. Replace "link2022" with the link you copied in step 3 from the 2022 data. 
```
wget link2018
gunzip GCA_039050005.1_ASM3905000v1_genomic.gbff.gz
mv GCA_039050005.1_ASM3905000v1_genomic.gbff genbank2018.gbff
wget link2022
gunzip GCA_039338855.1_ASM3933885v1_genomic.gbff.gz 
mv GCA_039338855.1_ASM3933885v1_genomic.gbff genbank2022.ggbff
```

## 1 Run python scripts

Run both python scripts in this folder to calculate the dnds for NA.

Once you run both notebooks you should see 2 .wig files: dnds_NA_2022.wig and dnds_NA_2018.wig

## 2 Convert to BigWig Files

Jbrowse is only able to visualize bigwig files so let's convert our wig files.

Open up a new terminal and make sure you are in this folder. Then run these commands:

### 2.1 Install Converter
```
conda install -c bioconda ucsc-wigtobigwig
```
### 2.2 Use the converter
```
echo -e "MT781550.2\t1520" > chrom.sizes
wigToBigWig dnds_NA_2018.wig chrom.sizes dnds_NA_2018.bw
echo -e "LC775581.1\t1435" > chrom.sizes
wigToBigWig dnds_NA_2022.wig chrom.sizes dnds_NA_2022.bw
```

After successfully completing step 2, you should see 2 new BigWig files in this folder.

## 3 Download to local computer

In this step, you will add these BigWig files you generated to your local computer.

1. Right click the BigWig(.bw) files and you should see an option "Copy Download Link". Click it
2. Go to you local terminal or AWS terminal and make sure you are in your tmp folder.
3. Use wget to download:

Make sure to put the link inside quotes.
```
wget -O nameofFile "download link"
```
Replace nameofFile with ex: dnds_NA_2022.bw

Do this for all 2 BigWig files: dnds_NA_2022.bw, dnds_NA_2018.bw

Use ls to make sure that it worked. You should see 2 new files in your tmp folder

## 4 Visualize on JBrowse

Lets visualize them now. 

In your terminal, make sure you are in the tmp folder

```
jbrowse add-track dnds_NA_2022.bw --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames flu_2022
jbrowse add-track dnds_NA_2018.bw --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames flu_2018
```

### Done!

You should be able to see them in your Jbrowse now.
