# How to generate and plot Conservation Scores

This folder contains all the reference genomes and genome annotations for 3 strains of Influenza in White-tailed Eagle.

## 0 Retrieving the fasta and gff files for analysis

This folder already contains the files, but here is how to retrieve them your self.

Using the same wget commands from steps 4-6 in ReadMe in main folder, get the fasta and gff files.
Open a new terminal and make sure you are in this folder. Then use the wget commands.
Unzip the files using gunzip.

## 1 Run Python Scripts

Run the python script in this folder to calculate the conservation scores for each protein that we want to observe. 

Once you have ran it, you should see 4 new .wig files appear. You can ignore the other generated files.

## 2 Convert to BigWig Files

Jbrowse is only able to visualize bigwig files so let's convert our wig files.

Open up a new terminal and make sure you are in this folder. Then run these commands:

### 2.1 Install Converter
```
conda install -c bioconda ucsc-wigtobigwig
```

### 2.2 Use the converter
```
echo -e "MT781552.2\t2295" > chrom.sizes
wigToBigWig pb2_conservation.wig chrom.sizes pb2_conservation.bw
echo -e "MT781553.2\t2274" > chrom.sizes
wigToBigWig pb1_conservation.wig chrom.sizes pb1_conservation.bw
echo -e "MT781554.2\t1704" > chrom.size
wigToBigWig HA_conservation.wig chrom.sizes HA_conservation.bw
echo -e "MT781550.2\t1520" > chrom.sizes
wigToBigWig NA_conservation.wig chrom.sizes NA_conservation.bw
```

After successfully completing step 2, you should see 4 new files in this folder.


## 3 Download to local computer

In this step, you will add these BigWig files you generated to your local computer.

1. Right click the BigWig(.bw) files and you should see an option "Copy Download Link". Click it
2. Go to you local terminal or AWS terminal and make sure you are in your tmp folder.
3. Use wget to download:

Make sure to put the link inside quotes.
```
wget -O nameofFile "download link"
```
Replace nameofFile with ex: pb2_conservation.bw

Do this for all 4 BigWig files

Use ls to make sure that it worked. You should see 4 new files in your tmp folder

## 4 Visualize on JBrowse

Lets visualize them now. For the purpose of being able to see all our comparision data in one place, you will be loading them onto flu2018 track. 

In your terminal, make sure you are in the tmp folder

```
jbrowse add-track nameofFile --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames flu_2018
```
Replace nameofFile with ex: pb2_conservation.bw

Run this command for all 4 conservation files. 

### Done!

You should be able to see them in your Jbrowse now.