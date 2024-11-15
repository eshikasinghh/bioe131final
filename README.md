# bioe131final

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
In your browser, now type in `http://yourhost/jbrowse2/`, where yourhost is either localhost or the IP address from earlier. Now you should see the words "**It worked!**" with a green box underneath saying "JBrowse 2 is installed." with some additional details. 

## 4. Load and process test data
### 4.1. Download and process reference genome
Make sure you are in the temporary folder you created, then download the human genome in fasta format. This is the biggest file you'll be downloading, and may take 30 min or so on AWS with the lowest tier of download speeds.

```
export FASTA_ROOT=https://ftp.ensembl.org/pub/release-110/fasta/homo_sapiens
wget $FASTA_ROOT/dna/Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz
```

Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate.

```
gunzip Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz
mv Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa hg38.fa
samtools faidx hg38.fa
```

### 4.3. Load genome into jbrowse

```
jbrowse add-assembly hg38.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

## 4.4. Download and process genome annotations

Still in the temporary folder, download ENSEMBLE genome annotations in the GFF3 format. 

```
export GFF_ROOT=https://ftp.ensembl.org/pub/release-110/gff3/homo_sapiens
wget $GFF_ROOT/Homo_sapiens.GRCh38.110.chr.gff3.gz
gunzip Homo_sapiens.GRCh38.110.chr.gff3.gz
```

Use jbrowse to sort the annotations. jbrowse sort-gff sorts the GFF3 by refName (first column) and start position (fourth column), while making sure to preserve the header lines at the top of the file (which start with “#”). We then compress the GFF with bgzip (block gzip, which zips files into little blocks for rapid access), and index with tabix. The tabix command outputs a file named genes.gff.gz.tbi in the same directory, and we then refer to “genes.gff.gz” as a “tabix indexed GFF3 file”.

```
jbrowse sort-gff Homo_sapiens.GRCh38.110.chr.gff3 > genes.gff
bgzip genes.gff
tabix genes.gff.gz
```

### 4.5. Load annotation track into jbrowse

```
jbrowse add-track genes.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy
```

### 4.6. Index for search-by-gene

Run the “jbrowse text-index” command to allow users to search by gene name within JBrowse 2.

In the temporary work directory, run the following command.

```
jbrowse text-index --out $APACHE_ROOT/jbrowse2
```

## 5.0 Use your genome browser to explore a gene of interest
### 5.1. Launch JBrowse2
Open `http://yourhost/jbrowse2/` again in your web browser. There should now be several options in the main menu. Follow the guide in the "Launch the JBrowse 2 application and search for a gene in the linear genome view" section of https://currentprotocols.onlinelibrary.wiley.com/doi/10.1002/cpz1.1120 to navigate to the gene search and try browsing a few genes.
