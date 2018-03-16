## Some additional notes

As promised, here is some additional information on NGS approaches and protocols for dealing with non-model organisms.

### Assessing the quality of your data

Whether or not you think screening NGS reads for quality is worthwhile (see below), you should at least get some idea of the quality of your raw data.

You want to look at the average quality score ([typically Phred-encoded](https://en.wikipedia.org/wiki/Phred_quality_score)) of your data and how it changes along reads. One of the best tools you can use to do this is [fastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). There is also [fastQC dashboard](https://github.com/pnnl/fqc) which can produce reports on multiple samples at once. 

Remember that this is a tool built for high-throughput genome-sequencing. You should not rely on it entirely as some of the assumptions of the basic tests it uses are violated by RAD-seq. For example, you expect a repetition of 5-base kmers with the RAD cut site. You would also expect duplication of sequences because you are repeatedly sequencing the same RAD locus.

### Quality control and trimming

We didn't demultiplex or quality trim our data in the practicals. One way to do this quickly and simply is to use the `process_radtags` module which is bundled with `stacks`. [This utility is very flexible and extremely useful](http://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php) - I use it and it's related `process_shortreads` program all time, even for non-RAD related tasks. 

Another good tool for handling read-quality is [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic). This allows you to trim poor-quality base calls off the end of your reads.

Note that you should **NOT** use this for RAD-seq data if you are using `stacks` because the pipeline assumes all reads are the same length - which is not true if you quality trim!

I have uploaded a script for both `process_radtags` and `Trimmomatic` to my github repository for the course. If you submit a pull request (i.e. `git pull`), you will have access to them.

Finally I mentioned that there is some debate as to whether screening for quality is worthwhile. See [here](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0085024) for a paper in favour and [here](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4766705/) for one arguing it can actually change your data. 

### A protocol for RAD seq on non-model organisms

I also mentioned that a paper was recently published that actually provided a protocol for choosing *denovo* RAD-sequencing parameters. You can find it [here](http://onlinelibrary.wiley.com/doi/10.1111/2041-210X.12775/abstract).

Scripts that the paper refers to are actually bundled with `stacks`.