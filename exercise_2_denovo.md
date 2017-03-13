## _denovo_ assembly RAD analysis

### Introduction

As you are no doubt aware by now, the morning session was lengthy and had a lot to take on board! Now that we are all more familiar with how referenced-based RAD-seq works, hopefully this next exercise will be a bit more straightforward. 

Our aim for this exercise is to do exactly the same as we did before - call variants from a RAD-seq dataset. Unlike before we will not map to a reference genome. Instead we will perform a _denovo_ assembly using the Stacks pipeline before calling SNPs and then outputting the data for genomic analysis. Like before, we will generate _F_<sub>ST</sub> estimates for our variants and if time allows, we will also use a method to detect selection.

For this analysis, we are going to work on a Nicuagaran cichlid species - _Amphilophus_ from a single site in a Nicuagaran crater lake. There are two species in this analysis, the thick lipped _A labiatus_ and the ancestral _A citronellus_. Our aim is to find SNPs under divergent selection between them.

---


### 1. Getting started

Login to the EVOP server like so:

	ssh mark@evopserver.bioinf.uni-leipzig.de

Navigate to your home directory and make a new directory to work in.

	cd ~
	mkdir denovo_rad
	cd denovo_rad
	
Next make a directory for the raw data and copy the data into it.

	mkdir raw 
	cd raw
	cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/raw_10k/* ./
	
Like before, have a look at the data. How many individuals are there for each species? Can you remember what each line in the fastq format represents?

---

### 2. Running the stacks pipeline

#### A bit of background

Unlike with our previous example, we do not have the luxury of a reference genome in this exercise. So we turn to the Stacks pipeline to perform our _denovo_ assembly.

Stacks is very well supported and has a **LOT** of information on its [website](http://catchenlab.life.illinois.edu/stacks/). There are many components to the Stacks pipeline and it can also be used for reference aligned data too.

Typically, I would run Stacks each component at a time as it allows more control over how the pipeline runs. However this is not recommended for beginners so for today's purposes I am going to show you how to run the whole pipeline using the `denovo_map.pl` program.

How does this work? The _denovo pipeline_ has a number of steps:

1. `ustacks` - this identifies read stacks, matches them to form loci and then calls SNPs within individuals
2.  `cstacks` - this creates a catalogue of all loci in the dataset and matches amongst individuals
3.  `sstacks` - this matches each sample to the catalogue in order to identify loci. 


#### Starting denovo_map.pl

There are many options for `denovo_map.pl` so the easiest way to understand what is going on is to actually run it and then we can break them all down.

Since there are so many options and it is very easy to make a mistake typing them all in, I will let you run a bash script to make the process easier. Input the following to get the analysis started.
	
	cd ~/denovo_rad
	cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/\
	scripts/denovo_cichlid.sh ./
	
Before running this script, you will also need a 'map' of the population samples - i.e. identifying which population is which. More on this later.
	
	cd ~/denovo_rad
	cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/other/cichlid.popmap ./
	
Take a quick look at this using `cat`. You will see it is a tab-delimited file with the sample name and population assignment (species in this case).

We will also take this opportunity to learn the useful utility, `screen`. This allows you to run multiple analyses in different screens and also preserves the analysis if your shell session is terminated for some reason. Start a new screen and then initiate the script in it.

	cd ~/denovo_rad
	mkdir stacks
	screen -S stacks
	bash denovo_cichlid.sh

You will now see the script is running. Press `ctrl + A + D` to move back to the main terminal. You can see the screens running (and return to your screen - named 'stacks' here) with the following:

	screen -ls
	screen -R stacks

Use `ctrl + A + D` to return to the main screen again. Now it's time to understand what the `denovo_cichlid.sh` script actually does. **NB. Don't run the code below, you'll ruin your currently running analysis!**

```
denovo_map.pl -m 3 -M 4 -n 4 -T 4 -b 1 -t \
-S -i 1 \
-O cichlid.popmap \
-o ./stacks \
-s ./raw/citronellus_10.fq.gz \
-s ./raw/citronellus_11.fq.gz \
-s ./raw/citronellus_12.fq.gz \
-s ./raw/citronellus_13.fq.gz \
-s ./raw/citronellus_14.fq.gz \
-s ./raw/citronellus_15.fq.gz \
-s ./raw/citronellus_16.fq.gz \
-s ./raw/citronellus_1.fq.gz \
-s ./raw/citronellus_2.fq.gz \
-s ./raw/citronellus_3.fq.gz \
-s ./raw/citronellus_4.fq.gz \
-s ./raw/citronellus_5.fq.gz \
-s ./raw/citronellus_6.fq.gz \
-s ./raw/citronellus_7.fq.gz \
-s ./raw/citronellus_8.fq.gz \
-s ./raw/citronellus_9.fq.gz \
-s ./raw/labiatus_10.fq.gz \
-s ./raw/labiatus_11.fq.gz \
-s ./raw/labiatus_12.fq.gz \
-s ./raw/labiatus_13.fq.gz \
-s ./raw/labiatus_1.fq.gz \
-s ./raw/labiatus_2.fq.gz \
-s ./raw/labiatus_3.fq.gz \
-s ./raw/labiatus_4.fq.gz \
-s ./raw/labiatus_5.fq.gz \
-s ./raw/labiatus_6.fq.gz \
-s ./raw/labiatus_7.fq.gz \
-s ./raw/labiatus_8.fq.gz \
-s ./raw/labiatus_9.fq.gz
```

This looks daunting but actually it's pretty simple. `denovo_map.pl` options are:

* `-m` - minimum number of identical reads to create a stack within an individual (therefore minimum locus depth is 2x this number)
* `-M` - minimum number of mismatches within an individual - i.e. to match two stacks into a single locus
* `-n`- minimum number of mismatches between loci when matching to build the catalogue
* `-T` - number of threads to run the analysis on
* `-b` - batch number for stacks catalogue - this can be set to 1 - it is necessary to run the pipeline but since we are not using SQL is not important.
* `-t` - break up repetitive stacks in ustacks - any stack with excessive reads will be split or removed to prevent repetitive regions being incorporated into the analysis.
* `-S` - disable SQL database settings.
* `-i` - initiate individual ids - set to 1.

There are also a number of input/output options

* `-O` - population map - a tab-delimited file denoting population
* `-o` - path to the output directory
* `-s` - path to a fasta file for each individual sample

###### A brief note on SQL interaction

One of the features of Stacks is that it allows mySQL integration so that you can view your RAD-seq data in an interactive database. Personally I prefer to run Stacks without this option but some users find it useful. If you want to use it for your own projects in the future, you can rerun the above code with the `-S` flag removed. This will make Stacks load the results of the analysis into a mySQL database which is named with the `-D` flag. Type `denovo_map.pl -h` to see more options related to mySQL interaction.

Note that if you run an analysis without SQL interaction (like we have here) you can also later load it into an SQL database using the `load_radtags.pl` program.


#### What does stacks output?

Check your stacks run using `screen -R stacks` - is it done yet? If you have had problems running the analysis, you can copy the output of the pipeline into your stacks directory like so:

	cd ~/denovo_rad/stacks
	cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/stacks/* ./
	
Use `ls -lah` to have a look at what is available here. Firstly you'll see a series of files that start with `batch_1`. These are the batch files produced by the catalogue.

For each individual you will see a series of files named `XXX.alleles.tsv.gz`, `XXX.matches.tsv.gz`, `XXX.snps.tsv.gz` and `XXX.tags.tsv.gz`. The main difference between these and `batch_1` are that the batch files are for the entire catalogue.

Use `zcat`and `head`to have a closer look at these. A quick summary:

* `XXX.alleles.tsv.gz` - alleles for the snp calls
* `XXX.matches.tsv.gz` - matches for the loci into the catalogue
* `XXX.snps.tsv.gz`- SNP calls for each locus
* `XXX.tags.tsv.gz`- denovo RAD tags identified by `ustacks`

You don't need to worry about these files so much as they will be used by the pipeline without your input. However you should have a close look at the `batch_1.catalog.tags.tsv.gz`. This is essentially a file of the consensus sequence for each locus identified in the population and can be useful for downstream analyses (i.e. when BLASTing RAD tags).

Chances are that when the pipeline was run, it failed to effectively run the `populations` module due to some sort of memory error. This usually happnes when there isn't enough memory to deal with this highly demanding step. This isn't fatal - we can run it again with more stringent filtrs to generate some output data using the `populations` module...

#### Filtering and exporting the data
 
The `populations` module allows us to filter the dataset based on the samples/loci we want to include. This is similar to the `vcftools` filtering step we carried out for our ref-aligned dataset.

Run populations like so:

```
cd ~/denovo_rad/
populations -b 1 -P ./stacks -M cichlid.popmap -t 4 \
-r 0.75 -p 2 -m 5 --fstats --vcf --genepop
```

With the reduced dataset, this should run quite quickly. As you will see from looking at the output, there are very few loci included in the final dataset - like with our reference dataset we will need to switch to some 'real data' for our downstream analyses to make any real sense. For now though, the populations options are:

* `-b` - batch number. Set to 1 - must be set but not important with SQL turned off.
* `-P` - path to Stacks output files
* `-M` - path to population map
* `-t` - thread number for running in parallel
* `-r` - minimum proportion of individuals present in a population to include a locus - 0.75 here.
* `-p` - number of populatiosn a locus must occur in - set to 2 here to ensure only loci present in both populations are included
* `-m` - minimum stack depth to include a locus. This is stack depth and *NOT* locus depth. 
* `--fstats` - output F statistics for SNPs and haplotypes
* `--vcf` - output vcf format
* `--genepop` - output genepop format

**NB: Stacks can output a wide-range formats - i.e. Structure, Phylip - for now though we only need these formats.**

Another point that is worth making - Stacks is well maintained and is continually updated. A lot of new features for outputing data in various different formats have been added since this tutorial was first put together and new features are being added all the time. Be sure to check the different options [on the pipeline website](http://catchenlab.life.illinois.edu/stacks/comp/populations.php)

Now we have run the _denovo_ Stacks pipeline in full. The next step is to look at this data in more detail.

---

### 3. Analysing the data

First things first, let's get the output from a full analysis.

```
cd ~/denovo_rad
mkdir results
cd results
cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/\
populations_full/* ./
```

####What does the output mean?

Before actually looking into this data in great detail, let's just get a sense of what we have actually produced here. Use `less` or `head` to look at each of the files in turn and examine the contents. The output is described below:

* `cichlid.vcf` - this should be familar after the last exercise - this is a vcf file for the SNP calls from each of the RAD loci. A special feature of vcf files output from Stacks is that the ID field contains the RAD locus identifier.
* `cichlid.sumstats_summary.tsv`- this is a summary of the summary statistics across all loci output by the pipeline. Useful for getting a general idea of what the data is showing. Here we can see there are 4929 private variants within _A. labiatus_ for instance.
* `cichlid.sumstats.tsv` - a more detailed version of the previous file. This is the summary statistics per site (**per variant not per locus**). Note that because we required loci to occur in both populations, there should be two rows for each locus in this file.
* `cichlid.hapstats.tsv` - similar to the previous file but now based on RAD loci, not SNPs.
* `cichlid.phistats.tsv` - Phi statistics for haplotype based analyses - these are analogous to _F_<sub>ST</sub> for the entire locus.
* `cichlid.haplotypes.tsv` - This is a matrix of haplotypes for each individual - this can be used for downstream analyses if necessary.
* `cichlid.genepop` - a genepop file generated from RAD data.
* `cichlid.fst_summary.tsv` - a summary file giving mean pairwise _F_~ST~. Here there is only one value because we are only comparing two populations.
* `cichlid.fst_citrinellus-labiatus.tsv ` - pairwise _F_<sub>ST</sub> for each SNP site. One pairwise file is generated for each population in the analysis, since there are only two populations in this analysis, we have only a single file.
* `cichlid.phistats_citrinellus-labiatus.tsv` - as above but for Phi instead of _F_~ST~. In this case only a single file and also Phi values should be identical to those in `cichlid.phistats.tsv`.
* `cichlid.populations.log` - a log of the populations run - useful for understanding why certain loci are dropped from the analysis. For example, the first part of this file shows the distribution of loci with a specific number of samples. You can see clearly that 891189 loci are missing samples (i.e. most probably removed after Stacks validity checks), the majority of loci also occur in only a single individual (249788 in this case).

For more information on all of these file formats and their fields, see the [Stacks manual](http://catchenlab.life.illinois.edu/stacks/manual/#pfiles).

####Assessing the output in more detail


We can get a basic idea of how well our analysis has performed from using some basic Unix commands on the output files. First of all - how many RAD loci did the full analysis produce?


```
cat cichlid.sumstats.tsv | tail -n +4 | cut -f 2 | uniq | wc -l
```

Here we used `cat` to view the file, `tail` to skip the first three lines, `cut` to extract the second column, `uniq` to retain only the unique identifiers and `wc -l` to count the number of lines. You should see we have >17 000 RAD loci... nice!

What about variants? We can use the vcf file for this. Ignore the `bcftools` warning - this is just because of the way Stacks handles vcf files.

```
bcftools view -H cichlid.vcf | wc -l
```
You will get a warning when you do this, but you can ignore it. More importantly, we have over >25 000 SNPs, which means we have ~1.4 SNPs per RAD locus. It is important to remember that RAD loci often have multiple SNPs and that these will be in linkage. This can have some serious repercussions for downstream analyses like detecting selection. One way to overcome this is to use RAD locus haplotypes instead of SNPs or alternatively to randomly select a single SNP from each locus. 

We could do this manually in the vcf or we could rerun `populations` with either the `--write_single_snp` or `--write_random_snp` options. The first of these	write the first SNP occurring on a haplotype while the second will randomly select one. For the rest of this exercise though, we will proceed with the data as it is.

#### Dealing with so much data

One issue you will have no doubt already realised is that it can be very difficult to really get an idea of how to get a handle on this amount of data. This is why it is really very useful to be able to plot your data properly.

If you have time, try plotting some of the data in `R` to get a feel for it. For instance, the distribution of _F_<sub>ST</sub> or Phi statistics may be useful. The easiest way to do this will be download the files to your local machine (we can show you how to do this in the class).

I have written an interactive `R` script for you to use locally. Unlike before, this will need a bit more input from you. Play around with it and don't be afraid to ask for help if you need it.

The script is here:

```
/homes/evopserver/lectures/NGS-NonModelOrganism/denovo/\
scripts/plotting_RAD_data.R
```

---

### 4. Identifying population structure using PCA

Now that we have generated our population genomic data, one thing we can do is look for any population structuring. For example, we might expect some divergence between the two _Amphilophus_ species.

A quick and straightforward way to investigate this is to use [Principal Components Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis) on allele frequencies. This will essentially tease apart the main axes of divergence between samples, based on allele frequency differences. 

There are lots of ways to perform PCA, but one of the simplest is using the `R` package `adegenet` which is [a versatile tool for population genomics](http://adegenet.r-forge.r-project.org/).

However before we can make use of `adegenet` we need to convert our data into a different format using `plink` which is [another really useful tool for genomic data](https://www.cog-genomics.org/plink2/). **NB:** `plink` actually has it's own PCA tool but today I want you to use the `adegenet` version to expose you to as many tools as possible.

#### Converting to plink raw format

First things first, we need to convert our `cichlid.vcf` into a binary. To do so, run the following command:

```
plink --vcf cichlid.vcf --double-id --allow-extra-chr --recodeA
```

Note that since `plink` was originally written for human data, it can sometimes be a little fiddly to run. Let's break these command down:

* `--vcf` - specifies our input file is a vcf
* `--double-id` - the just tells plink to ignore the sample names in the vcf header; the program tries to assign samples to families and pedigrees and all this does is to tell it to use each name twice for that.
* `--allow-extra-chr` - tells `plink` to ignore the chromosome configuration in the vcf. Since this is denovo, we actually don't have any chromosomes (they are listed as `Un` for unknown) but otherwise, `plink` would expect 23 chromosomes, as in humans.
* `--recodeA` - this is the most important command as it tells `plink` how to recode the data into a new format. We need a raw output with an additive component (i.e. 0 for homozygote for the first allele, 1 for heterozygote and 2 for homozygote for the second allele)

Take a look in the directory using `ls`, you should see three files; `plink.log`, `plink.nosex` and `plink.raw`. Move out of the directory and create a pca directory, then move these files into it. Like so:

```
cd ..
mkdir pca
mv results/plink.* pca
cd pca
```

The next step is to perform a PCA on these files! Just in case this step didn't work for any reason, you can get the plink files like so:

```
cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/plink/* ./
```


#### Performing PCA with adegenet

Creating the PCA is quite straightforward and luckily, like most good tools, `adegenet` is very well documented. In fact, there are a series of [tutorials online](https://github.com/thibautjombart/adegenet/wiki/Tutorials) which show you how to perform most of the major analyses, including PCA. 

Because of that and also because of time constraints we will use a custom R script to make our PCA today. But again, feel free to look at the script in detail. First of all, let's get the script:

```
cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/scripts/make_pca_plot.R ./
```

Now run it. If you want to know what the commands do, use `--help`

```
Rscript make_pca_plot.R -i plink.raw -o pca_plot.pdf
```

Take a look at the PCA plot you've created. Can you see anything odd about it? Firstly you can see there is a good chance that one of the *A. labiatus* individuals has been mislabelled. Secondly, it seems like the main axis of differentiation between the species is well, not really between the species at all. In fact there are two clear outliers - this suggests there may be an error in the data. PCA is quite sensitive to missing data, so this can often be a good way to diagnose issues. 

---



### 5. Detecting selection using Bayescan

We saw in the section before last that Stacks produced some population genomic analyses on our behalf. But what if we want to go further than this and identify whether any of the SNP loci we identified are under divergent selection between these two morphs?

There are several different approaches we can take to achieve this but today we are going to use `Bayescan`, [a Bayesian method](http://cmpg.unibe.ch/software/BayeScan/) for that decomposes _F_<sub>ST</sub> into 'local and global effects.

One immediate issue we have is that Stacks does not output the format required to use `Bayescan`. Not to worry - we can produce one! The easiest way to do this is using a SNP matrix generated with the `vcftools` `perl`utility.

```
vcf-to-tab < cichlid.vcf > cichlid.geno
```
Use `head` to take a look at the file - this is a really nice, useful SNP genotype matrix for each individual in our dataset at each SNP position.

We can then convert this to a Bayescan input using a custom R script I have written. Use the following code to copy this to your working directory and then run it on the input.

```
cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/\
scripts/bayescan_convert.R ./
Rscript bayescan_convert.R -i cichlid.geno -o cichlid_bayes.txt
```
As before, feel free to look at this script to see what it is doing. The exact mechanics of it are beyond the scope of this tutorial but it should work with most datasets.

Now we are ready to run Bayescan! Run the following line, as before I'll break it down afterwards.

```
BayeScan2.1_linux64bits ./cichlid_bayes.txt -o cichlid_bayes_run \
-threads 4 -n 5000 -thin 10 -nbp 20 -pilot 5000 \
-burn 50000 -pr_odds 10 
```

What do we have here?

* `-o` - the output prefix
* `-threads` - number of threads to run in parallel
* `-n` - number of samples of the MCMC we want to take
* `-thin` - the thinning interval - basically how often to take a sample, also controls length of MCMC (so 5000*10 = 50000 iterations in this case)
* `-nbp` - the number of pilot runs the program makes to initiate the MCMC - Bayescan does this to optimise the MCMC search
* `-pilot` - length of MCMC for pilot runs
* `-burn`- the length of the burnin - here it is %50 of the total MCMC.

Chances are that this will take far too long to run in the class so you can copy some finished results here:

```
cp -r /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/outlier/bayes/ ./
```

Note that here, the `-r` flag for `cp` means copy recursively - so it will copy the entire directory. Enter the directory you just copied and have a look at the output. The main one we are interested in is `cichlid_bayes_run_fst.txt` which is the _F_<sub>ST</sub> estimate for each locus. The qval in this file is the posterior proability, corrected for false positives that a locus is under selection.

As with most of these analyses, the easiest way to understand the output is to plot it. Again, I have written an `R` script to do this for you. Use the following code:

```
cp /homes/evopserver/lectures/NGS-NonModelOrganism/denovo/\
scripts/Bayes_plot.R ./

Rscript Bayes_plot.R -i cichlid_bayes_run_fst.txt -o Cichlid_bayes_plot.pdf
```
Have a look at the plot and the output of the `R` script to the screen. How many loci are under selection? **Tip:** The dashed vertical line represents our cut-off threshold for determining whether a locus is under selection. 

---



	
