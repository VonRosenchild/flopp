# flopp : fast local polyploid phasing from long read sequencing.

## Release Notes:

### v0.2.0 flopp - Aug 25, 2021

- Added options; user can now manipulate error rates, block lengths, and more
- flopp can now output the read partition obtained by phasing using the -P option
- flopp outputs MEC to stdout by default now
- fixed a bug where the UPEM normalization was too small
- Can now input non-polyploid VCF file. If your genotypes are not confident or if you use a diploid variant caller, you can use -c (VCF) instead. 

## Introduction

**flopp** is a software package for single individual haplotype phasing of polyploid organisms from long read sequencing. flopp is extremely fast, multithreaded, and written entirely in the rust programming language. flopp offers an order of magnitude speedup and better accuracies compared to other polyploid haplotype phasing algorithms.

Given 

1. a list of variants in .vcf format
2. a set of reads mapped to a reference in .bam format

for a single individual, **flopp** outputs a set of phased haplotypes.

### Requirements 

1. [rust](https://www.rust-lang.org/tools/install) and associated tools such as cargo are required and assumed to be in PATH.
### Install

```
git clone https://github.com/bluenote-1577/flopp
cd flopp
cargo build --release
./target/release/flopp -h
```

`cargo build --release` builds the **flopp** binary, which is found in the ./target/release/ directory. 

## Using flopp

```
# with VCF and BAM
flopp -b bamfile.bam -v vcffile.vcf -p (ploidy) -o results.txt -P partition_directory 

# same as above but -c allows VCF non (ploidy) number of genotypes. e.g. diploid or population VCFs
flopp -b bamfile.bam -c vcffile.vcf -p (ploidy) -o unpolished_results.txt -P partition_directory

# with fragment file 
flopp -f fragfile.frags -p (ploidy) -o unpolished_results.txt 
```
The ploidy of the organism must be specified. The number of threads (default 10) can be specified using the -t option. See `flopp -h` for more information.  

For a quick test, we provide a VCF and BAM files in the tests folder. Run ``flopp -b tests/test_bams/pds_ploidy3.bam -v tests/test_vcfs/pds.vcf -p 3 -o results.txt -P test_partition_directory`` to run flopp on a 3 Mb section of a simulated 3x ploidy potato chromosome with 30x read coverage.

### BAM + VCF
The standard mode of usage is to specify a bam file using the option **-b** and a vcf file using the option **-v** or **-c**. Use **-v** if the genotypes in your .vcf file has the same ploidy as your organism. Use **-c** if you the .vcf file has differing ploidy than your organism, i.e. the value in the **-p** option.

The output is written to a text file with value of option **-o**. If **-P** is specified, then the partition of the input reads according to haplotypes is also output.

flopp currently only uses SNP information and does not take into account indels. However, the user may define their own fragments which can be indexed by other types of variants. See the **Fragment file** section at the bottom.

The bam file may contain multiple contigs/references which the reads are mapped to as long as the corresponding contigs also appear in the vcf file. 

### Output
#### Phased haplotype output (-o option)
flopp outputs a phased haplotype file in the following format:

1. Column 1 is (variant) : (genome position) where (variant) is the i-th variant, and the genome position is the the position of the genome on the reference.
2. The next k columns are the k phased haplotypes for an organism of ploidy k. 0 represents the reference allele, 1 the first alternate, and so forth. 
3. The next k columns are of the form (allele):(# calls)|(allele):(# calls) where (allele) = 0,1,... and (# calls) is the number of reads assigned to a specific haplotype which contain that allele. For example, 0:10|1:5 indicates that 10 reads assigned to this haplotype have allele 0 at this position, and 5 reads have allele 1. 

If using a bam file with multiple contigs being mapped to, the output file contains multiple phased haplotypes of the above format which are delimited by `**(contig name)**`.

#### Read partition output (-P option)
If also using `-P` option, flopp outputs the read partition obtained by flopp. That is, set of reads corresponding to each haplotype. The format looks like:
```
#1 (partition #1)
(read_name1) (first SNP position covered) (last SNP position covered)
(read_name2) (first SNP position covered) (last SNP position covered)
...
#2 (partition #2)
...
```

To get a set of BAM files which correspond to the output read partition (i.e. the haplotypes), use

``python scripts/get_bam_partition.py (-P output file) (original BAM file) (prefix name for output)``

This will output a set of bams labelled `prefix_name1.bam`, `prefix_name2.bam` and so forth. This script requires pysam.

### Misc.

#### Fragment file
A user can also input a fragment file using the option **-f**. The fragment file is a file where each line is a read which is indexed by variants; see https://github.com/MinzhuXie/H-PoPG or https://github.com/realabolfazl/AltHap for more details about the fragment file specifcation (called the *input snp matrix* by H-PoP). Specifying a compatible VCF file with a fragment file uses genotyping information to produce a higher quality output; only SNPs will be processed in the VCF.  

For testing purposes and compatibility with other haplotype phasing algorithms, the binary **frag-dump** is provided in the same folder as the **flopp** binary. 

`frag-dump -b bamfile.bam -v vcffile.vcf -o frags.txt` gives a fragment file a.k.a input snp matrix which is compatible with H-PoP and other haplotype phasing algorithms. 

#### VCF requires contig headers
We found that some variant callers don't put contig headers in the VCF file. In this situation, run `python scripts/write_contig_headers_vcf.py (vcf_file)` to get a new VCF with contig headers.

## Citing flopp

Jim Shaw and Yun William Yu; Practical probabilistic and graphical formulations of long-read polyploid haplotype phasing. (Accepted to [RECOMB2021](https://www.recomb2021.org/))


