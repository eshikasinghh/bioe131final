IMPORTANT: Please use Incognito Mode to view everything properly.
# Link to Working Example of Database: https://eshikasinghh.github.io/bioe131final/
To access our database, click the link above click "Launch Genome Browser". Start a new session by clicking "Linear Genome View", selecting the "flu2022" assembly, and selecting "MT781550.2" to visualize the NA protein. Click "open", "open track selection", and select these options so you're screen looks like this:
<img width="1512" alt="Screenshot 2024-12-05 at 3 11 19 PM" src="https://github.com/user-attachments/assets/050a2872-2796-414b-b47b-d0f648ea95f1">
Select "MT781554.2" to visualize the HA protein, "MT781552.2" to visualize the PB2 protein, and "MT781553.2" to visualize the PB1 protein. The track selection is the same as the screenshot, minus the dnds_NA track for these 3 proteins. 

ALl screenshots of the 4 proteins will be in the "JBrowse_Screenshots" folder of our repository.

# BioE-131 Final. How to Set Up Our Data on JBrowse.

### 1.1. Mac OS setup
Open a terminal and run the line below to install homebrew, a macOS package manager. This will make it easy for you to install necessary packages like apache2 and samtools. _You can skip this step if you already have brew installed._ 

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
If this doesn't work, visit https://docs.brew.sh/Installation for further installation options, including a .pkg installer that should be convenient and easy to use.

## 2. Install necessary tools
### 2.1. Node.js
Node.js is a cross-platform JavaScript runtime environment that will make is easy to run JBrowse2 command-line tools.

First, check whether Node.js is already installed by running the following. If node v20 is already installed, you can skip to the next step.

```
node -v
```

If Node.js is not installed, install it. 

#### macOS

On macOS, you can use brew. You may need to restart the terminal (close and open a new one) to get `node -v` to run.

```
# NOTE:
# Homebrew is not a Node.js package manager.
# Please ensure it is already installed on your system.
# Follow official instructions at https://brew.sh/
# Homebrew only supports installing major Node.js versions and might not support the latest Node.js version from the 20 release line.
# download and install Node.js
brew install node@20
# verifies the right Node.js version is in the environment
node -v # should print `v20.18.0`
# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```

### 2.2. @jbrowse/cli
Run the following commands in your shell. This uses the Node.js package manager to download the latest stable version of the jbrowse command line tool, then prints out its version. This should work for both macOS and Linux.

```
sudo npm install -g @jbrowse/cli
jbrowse --version
```

You can also try installing using just `npm install -g @jbrowse/cli` if the sudo version doesn't run. 

### 2.3. System dependencies
Install wget (if not already installed), apache2, samtools, and tabix. 

wget is a tool for retrieving files over widely-used Internet protocols like HTTP and FTP. 

apache2 allows you to run a web server on your machine.

samtools and tabix, as we have learned earlier in the course, are tools for processing and indexing genome and genome annotation files.

#### macOS

```
# note that apache2 gets installed as httpd for macOS, which is the service you will launch later
brew install wget httpd samtools htslib
```
## 3. Apache server setup
### 3.1. Start the apache2 server
Starting up the web server will provide a localhost page to show that apache2 is installed and working correctly. When discussing computer networking, localhost is a hostname that refers to the current computer used to access the network. Note that in WSL2, the linux subsystem may have a different IP address from your Windows OS, and so you will want to use that IP address to be able to find it and load the web page. AWS, on the other hand, will have a public IP address that you need to identify in the aws_instructions.

#### macOS
```
sudo brew services start httpd
```
If that doesn't work, try 
```
/opt/homebrew/bin/httpd -k start
```

### 3.2. Getting the host
If you are running locally on your mac, the hostname is just `localhost`. For local hosting, the url will be `http://localhost:8080/`.

### 3.3. Access the web server
Open a browser and type the appropriate url into the address bar. You should then get to a page that says "**It works!**" (for AWS there may be some additional info). If you have trouble accessing the server, you can try checking your firewall settings and disabling any VPNs or proxies to make sure traffic to localhost is allowed.

### 3.4. Verify apache2 server folder
Apache2 web servers serve files from within a root directory. This is configurable in the httpd.conf configuration file, but you shouldn't have to change it (in fact, changing the conf file is not recommended unless you know what you are doing). 

For a normal linux installation, the folder should be `/var/www` or `/var/www/html`, whereas when you install on macOS using brew it will likely be in `/opt/homebrew/var/www` (for M1) or `/usr/local/var/www` (for Intel). You can run `brew --prefix` to get the brew install location, and then from there it is in the `var/www` folder. 

Verify that one of these folders exists (it should currently be empty, except possibly for an index file, but we will now populate it with JBrowse 2). If you have e.g. a www folder with no www/html folder, and your web server is showing the "It works!" message, you can assume that the www one is the root directory. 

Take note of what the folder is, and use the command below to store it as a command-line variable. We can reference this variable in the rest of our code, to save on typing. You will need to re-run the `export` if you restart your terminal session!
```
# be sure to replace the path with your actual true path!
export APACHE_ROOT='/path/to/rootdir'
```

If you are really struggling to find the APACHE_ROOT folder, you could try searching for it.
```
sudo find / -name "www" 2>/dev/null
```

### 3.5. Download JBrowse 2
First create a temporary working directory as a staging area. You can use any folder you want, but moving forward we are assuming you created ~/tmp in your home folder.

```
mkdir ∼/tmp
```
```
cd ∼/tmp
```

Next, download and copy over JBrowse 2 into the apache2 root dir, setting the owner to the current user with `chown` and printing out the version number. This version doesn't have to match the command-line jbrowse version, but it should be a version that makes sense.

```
jbrowse create output_folder
sudo mv output_folder $APACHE_ROOT/jbrowse2
sudo chown -R $(whoami) $APACHE_ROOT/jbrowse2
```

### 3.6. Test your jbrowse install
In your browser, now type in `http://yourhost/jbrowse2/`, where yourhost is either localhost or the IP address from earlier. For local hosting, the url will be `http://localhost:8080/jbrowse2/` Now you should see the words "**It worked!**" with a green box underneath saying "JBrowse 2 is installed." with some additional details. 

## 4. Load and process Influenza A data and genome annotations from 2022
### 4.1. Download and process reference genome
Make sure you are in the temporary folder you created. The following links may update periodically, so follow these steps below yourself if the given wget commands don't work:

This is where we are pulling our data from: `https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_039338855.1/`.  Open this link, and click on FTP. Copy Paste the URL from the new FTP site. It should look similar to this: `https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/338/855/GCA_039338855.1_ASM3933885v1/`. In the below wget commands, copy paste the fna.gz `GCA_039338855.1_ASM3933885v1_genomic.fna.gz` and gff.gz `GCA_039338855.1_ASM3933885v1_genomic.gff.gz` portions to the end of the FTP URL. It should look like the commands below. Run both wget commands. 
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/338/855/GCA_039338855.1_ASM3933885v1/GCA_039338855.1_ASM3933885v1_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/338/855/GCA_039338855.1_ASM3933885v1/GCA_039338855.1_ASM3933885v1_genomic.gff.gz
```

Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate. If the following commands don't work, here is how you do it yourself. `ls` in your tmp directory and you should see the fna.gz file that you just added. Copy paste that into the `gunzip` command. Then, `ls` again and you should see a `.fna` file. Copy paste that file and the `mv` command will use that and add the `flu2022.fa` name after. 

```
gunzip GCA_039388855.1_ASM393885v1_genomic.fna.gz
mv GCA_039388855.1_ASM393885v1_genomic.fna flu2022.fa
samtools faidx flu2022.fa
```

### 4.3. Load genome into jbrowse

```
jbrowse add-assembly flu2022.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

## 4.4. Process genome annotations

Still in the tmp folder, run the following command. The links used may update periodically, so follow these steps below yourself if the given gunzip command doesn't work: `ls` in the tmp folder and you will see a `.gff.gz` file. Copy paste that after the gunzip command.


```
gunzip GCA_039388855.1_ASM393885v1_genomic.gff.gz
```

Use jbrowse to sort the annotations. jbrowse sort-gff sorts the GFF3 by refName (first column) and start position (fourth column), while making sure to preserve the header lines at the top of the file (which start with “#”). We then compress the GFF with bgzip (block gzip, which zips files into little blocks for rapid access), and index with tabix. The tabix command outputs a file named genes.gff.gz.tbi in the same directory, and we then refer to “genes.gff.gz” as a “tabix indexed GFF3 file”.

The links used may update periodically, so follow these steps below yourself if the given `sort-gff` command doesn't work: Use the same file as the previous command and remove the `.gz`. Copy paste that after the sort-gff command.

```
jbrowse sort-gff GCA_039388855.1_ASM393885v1_genomic.gff > flu2022.gff
bgzip flu2022.gff
tabix flu2022.gff.gz
```

### 4.5. Load annotation track into jbrowse

```
jbrowse add-track flu2022.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames flu2022
```

## 5. Load and process Influenza A data and genome annotations from 2021
### 5.1. Download and process reference genome
Make sure you are in the temporary folder you created. The following links may update periodically, so follow these steps below yourself if the given wget commands don't work:

This is where we are pulling our data from: `https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_039230675.1/`.  Open this link, and click on FTP. Copy Paste the URL from the new FTP site. It should look similar to this: `https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/230/675/GCA_039230675.1_ASM3923067v1/`. In the below wget commands, copy paste the fna.gz `GCA_039230675.1_ASM3923067v1_genomic.fna.gz` and gff.gz `GCA_039230675.1_ASM3923067v1_genomic.gff.gz` portions to the end of the FTP URL. It should look like the commands below. Run both wget commands. 
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/230/675/GCA_039230675.1_ASM3923067v1/GCA_039230675.1_ASM3923067v1_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/230/675/GCA_039230675.1_ASM3923067v1/GCA_039230675.1_ASM3923067v1_genomic.gff.gz
```

Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate. If the following commands don't work, here is how you do it yourself. `ls` in your tmp directory and you should see the fna.gz file that you just added. Copy paste that into the `gunzip` command. Then, `ls` again and you should see a `.fna` file. Copy paste that file and the `mv` command will use that and add the `flu2021.fa` name after. 

```
gunzip GCA_039230675.1_ASM3923067v1_genomic.fna.gz
mv GCA_039230675.1_ASM3923067v1_genomic.fna flu2021.fa
samtools faidx flu2021.fa
```

### 5.3. Load genome into jbrowse

```
jbrowse add-assembly flu2021.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

## 5.4. Process genome annotations

Still in the tmp folder, run the following command. The links used may update periodically, so follow these steps below yourself if the given gunzip command doesn't work: `ls` in the tmp folder and you will see a `.gff.gz` file. Copy paste that after the gunzip command.


```
gunzip GCA_039230675.1_ASM3923067v1_genomic.gff.gz
```

Use jbrowse to sort the annotations. jbrowse sort-gff sorts the GFF3 by refName (first column) and start position (fourth column), while making sure to preserve the header lines at the top of the file (which start with “#”). We then compress the GFF with bgzip (block gzip, which zips files into little blocks for rapid access), and index with tabix. The tabix command outputs a file named genes.gff.gz.tbi in the same directory, and we then refer to “genes.gff.gz” as a “tabix indexed GFF3 file”.

The links used may update periodically, so follow these steps below yourself if the given `sort-gff` command doesn't work: Use the same file as the previous command and remove the `.gz`. Copy paste that after the sort-gff command.

```
jbrowse sort-gff GCA_039230675.1_ASM3923067v1_genomic.gff > flu2021.gff
bgzip flu2021.gff
tabix flu2021.gff.gz
```

### 5.5. Load annotation track into jbrowse

```
jbrowse add-track flu2021.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames flu2021
```

## 6. Load and process Influenza A data and genome annotations from 2018
### 6.1. Download and process reference genome
Make sure you are in the temporary folder you created. The following links may update periodically, so follow these steps below yourself if the given wget commands don't work:

This is where we are pulling our data from: `https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_039050005.1/`.  Open this link, and click on FTP. Copy Paste the URL from the new FTP site. It should look similar to this: `https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/050/005/GCA_039050005.1_ASM3905000v1/`. In the below wget commands, copy paste the fna.gz `GCA_039050005.1_ASM3905000v1_genomic.fna.gz` and gff.gz `GCA_039050005.1_ASM3905000v1_genomic.gff.gz` portions to the end of the FTP URL. It should look like the commands below. Run both wget commands. 
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/050/005/GCA_039050005.1_ASM3905000v1/GCA_039050005.1_ASM3905000v1_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/039/050/005/GCA_039050005.1_ASM3905000v1/GCA_039050005.1_ASM3905000v1_genomic.gff.gz
```

Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate. If the following commands don't work, here is how you do it yourself. `ls` in your tmp directory and you should see the fna.gz file that you just added. Copy paste that into the `gunzip` command. Then, `ls` again and you should see a `.fna` file. Copy paste that file and the `mv` command will use that and add the `flu2018.fa` name after. 

```
gunzip GCA_039050005.1_ASM39050000v1_genomic.fna.gz
mv GCA_039050005.1_ASM39050000v1_genomic.fna flu2018.fa
samtools faidx flu2018.fa
```

### 6.3. Load genome into jbrowse

```
jbrowse add-assembly flu2018.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

## 6.4. Process genome annotations

Still in the tmp folder, run the following command. The links used may update periodically, so follow these steps below yourself if the given gunzip command doesn't work: `ls` in the tmp folder and you will see a `.gff.gz` file. Copy paste that after the gunzip command.


```
gunzip GCA_039050005.1_ASM3905000v1_genomic.gff.gz
```

Use jbrowse to sort the annotations. jbrowse sort-gff sorts the GFF3 by refName (first column) and start position (fourth column), while making sure to preserve the header lines at the top of the file (which start with “#”). We then compress the GFF with bgzip (block gzip, which zips files into little blocks for rapid access), and index with tabix. The tabix command outputs a file named genes.gff.gz.tbi in the same directory, and we then refer to “genes.gff.gz” as a “tabix indexed GFF3 file”.

The links used may update periodically, so follow these steps below yourself if the given `sort-gff` command doesn't work: Use the same file as the previous command and remove the `.gz`. Copy paste that after the sort-gff command.

```
jbrowse sort-gff GCA_039050005.1_ASM3905000v1_genomic.gff > flu2018.gff
bgzip flu2018.gff
tabix flu2018.gff.gz
```

### 6.5. Load annotation track into jbrowse

```
jbrowse add-track flu2018.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames flu2018
```

## 7.0 Use your genome browser to explore a gene of interest
### 7.1. Launch JBrowse2
Open `http://yourhost/jbrowse2/` again in your web browser. For local hosting, the url will be `http://localhost:8080/jbrowse2`.There should now be several options in the main menu. Search for a gene in the "linear genome view" section, and you will see our flu2022, flu2021, and flu2018 as dropdown options to select.
