# GapFiller

## Description
---
GapFiller

## Requirements
---
+ [x] Perl
  - [x] File::Basename;
  - [x] Storable;
  - [x] File::Path;
  - [x] File::Copy;
  - [x] FindBin qw($Bin);
  - [x] threads;
  - [x] Text::Wrap;
  - [x] GepOpt::Std
  - [x] threads::shared;
  - [x] constant
+ [x] [Bowtie](http://bowtie-bio.sourceforge.net)
+ [x] [BWA](http://bio-bwa.sourceforge.net/)


## Changes

### v1.11

- Replace getopts.pl dependency with Getopt::Std, which is usually a core module of perl. No need to install getopts.pl from perl4 separately

- Shebang from /usr/bin/perl to /usr/bin/env perl, which is more flexible



# Maintainer
---
    卢福浩(Fu-Hao Lu)
    
    Professor, PhD
    
    作物逆境适应与改良国家重点实验室，生命科学学院
    
    State Key Labortory of Crop Stress Adaptation and Improvement
    
    College of Life Science
    
    河南大学金明校区
    
    Jinming Campus, Henan University
    
    开封 475004， 中国
    
    Kaifeng 475004, P.R.China
    
    E-mail: LUFUHAO@HENU.EDU.CN



# Original README [Untouched]


GapFiller v1.10 Marten Boetzer - Walter Pirovano, Jul 2012
email: walter.pirovano@baseclear.com

Citation
---------------
If you use GapFiller in a scientific publication, please cite:
Boetzer, M. and Pirovano, W., Toward almost closed genomes with GapFiller, Genome Biology, 13(6), 2012.

License
---------------
GapFiller can be freely used by academic institutes or non-profit organizations. Commercial parties need to acquire a license. For more information a
bout commercial licenses look at our website or email info@baseclear.com.

Description
-----------

GapFiller is a script able to close gaps produced by a scaffolding program using one or more mate pairs or paired-end libraries, or even a combination. 

Implementation and requirements
-------------------------------

GapFiller is implemented in perl and runs on linux, MAC and windows. 

PLEASE READ:
GapFiller tracks in memory all reads that are likely to be in the gaps. That means that the memory usage will increase drastically with the number of gaps in your scaffolds and the number of input reads. Just be aware of these limitations and don't be surprised if you observe a lot of data swapping to disk if you attempt to run on a machine with little RAM.  

Running GapFiller
-------------

e.g. perl GapFiller.pl -l libraries.txt -s scaffolds.fasta -m 30 -o 2 -r 0.7 -n 10 -d 50 -t 10 -g 0 -T 1 -i 1 -b standard_out 

Usage: ./GapFiller.pl

General Parameters:
-l  Library file containing two mate pate files with insert size, error and orientation indication
-s  Fasta file containing scaffold sequences used for extension.

Extension Parameters:
-m  Minimum number of overlapping bases with the sequences around the gap(default -m 30, optional)
-t  Number of bases to trim of from edges of the gap, usually containing misassemblies. (default -t 10, optional)
-o  Minimum number of reads needed to call a base during an extension (default -o 2, optional)
-r  Percentage of reads that should have a single nucleotide extension in order to close a gap in a scaffold (Default: 0.7, optional)
-d  Maximum difference between the gapsize and the number of gapclosed nucleotides (default -d 50, optional)
-n  Minimum overlap required to merge adjacent sequences in a scaffold (default -n 10, optional)
-i  Number of iterations to fill the gaps (default 1, optional)

Bowtie Parameters:
-g  Maximum number of allowed gaps during mapping with Bowtie. Corresponds to the -v option in Bowtie. *higher number of allowed gaps can lead to least accurate GapFiller* (default -g 0, optional)

Additional Parameters:
-T  Number of threads to run GapFiller. (default -T 1)
-b  Base name for your output files (default -b standard_output)

How it works
------------

The program consists of four steps, a short overview;
	- read and convert paired-read sequences
	- read and split the scaffolds on the gaps
	- Map reads to scaffolds and get for each gap the reads
	- Extend surrounding sequences and try to close the gap

In more detail:

1) Read -l library file;
	A) For each library in the -l library file. Store the reads in appropriate format. Paired reads are stored in a new file with a similar read name for easy tracking of the paired read. Format is;

>read1/1
AGCTGATAGATGAT
>read1/2
GATGATAGATAGAC

2) cut the scaffolds on the gaps (either 'n' or 'N') and store each individual sequence to a new file. In addition, store only the edges of the sequences for speeding up the mapping process of Bowtie. An example is given below;
Cutting of edges is determined by taking the maximal allowed distance inserted by the user in the library file (insert size and insert standard deviation). The maximal distance is insert_size + (insert_size * insert_stdev). For example, with a insert size of 500 and a deviation of 0.5, the maximal distance is 750. First 750 bases and last 750 bases are subtracted from the sequence, in this case. 

------------------------------------------
           |                  |                			
------------                  ------------
   750bp                          750bp


3) Map reads to the individual sequences and determine whether they fall within a gap or not. Read pair sequences are converted to --><-- format. Based on this information, the reads that fall into a gap are determined. In figure, this looks like;

	      seq1	    gap1       seq2
scaf:	---------------NNNNNN------------------
read1:        --->	???
read2:	     --->	      <---
read3: --->	  <---
read4:	     	       ???      <---		
read5: ???	  <---

From the above five readpairs, only two are used for filling 'gap1'. Here we will go to each of the read pair in some detail;

read1: used for closing gap since the first read is found at seq1 in --> orientation and not found at the same sequence ('seq1') or the next sequence ('seq2')
read2: not used for closing the gap since the pair is found at consecutive sequences.
read3: not used for closing the gap since the pair is found at the same sequence ('seq1')
read4: used for closing gap since the second read is found at seq1 in <-- orientation and not found at the same sequence ('seq2') or the previous sequence ('seq3'
read5: not used for closing the gap since the read is found at wrong orientation

The reads falling within the gap are cut in smaller fragments called k-mers (-m + 1). For each gap there is thus a collection of reads stored in memory.

4) Gapclosing 
After collecting the reads for each gap, the gap is tried to be filled with the collected reads. K-mers are extracted having -m overlap with the edges of the gap. After collecting all k-mers, of each possible single nucleotide extension (A,C,G,T) the abundance is counted. E.g:

sequence:   AGGAAGGAATTACCCAG
k-mer:      AGGAAGGAATTACCCAGT
k-mer:      AGGAAGGAATTACCCAGA
k-mer:      AGGAAGGAATTACCCAGA
k-mer:      AGGAAGGAATTACCCAGA
k-mer:      AGGAAGGAATTACCCAGA
k-mer:      AGGAAGGAATTACCCAGA

The counts for each overlap will be: A=5, T=1, G=0, C=0. Based on GapFillers' parameters, the most likely nucleotide extension is calculated. E.g. the coverage of nucleotide 'A' is 5, which should be above parameter -o. The ratio of nucleotide 'A' is 5/(1+5) = 0.83 (=most abundant nucleotide / sum of two most abundant nucleotides), this value should be above parameter -r.

The gap is filled by a single nucleotide extension of both sequences surrounding the gap. The method looks something like;
having a overlap (-m) with the edge of the sequence. During each This method is in short;
      
sequence:	AGGAAGGAATTACCCAGNNNNNNGATTAAGGCAGCACGAA
iteration1  AGGAAGGAATTACCCAGA
iteration2  			    CGATTAAGGCAGCACGAA
iteration3   GGAAGGAATTACCCAGAT
iteration4  			   ACGATTAAGGCAGCACGA
iteration5    GAAGGAATTACCCAGATG
iteration6  			  GACGATTAAGGCAGCACG
iteration5     AAGGAATTACCCAGATGG
iteration6  			 GGACGATTAAGGCAGCAC
iteration5      AGGAATTACCCAGATGGA
iteration6  			TGGACGATTAAGGCAGCA

After each iteration, GapFiller tries to find an overlap (-n) between the two sequences surround the gap, if sufficient overlap is found and within the allowed difference (-d) between the gapsize and the number of gapclosed nucleotides, they are merged together.

Input sequences
---------------

FASTA FILES:
>ILLUMINA-52179E_0001:3:1:1062:15216#0/2
ATNGGGTTTTTCAACTGCTAAGTCAGCAGGCTTTTCACCCTTCAACATC
>ILLUMINA-52179E_0001:3:1:1062:4837#0/2
ANNAACTCGTGCCGTTAAAGGTGGTCTTGCATTTCAGAAAGCTCACCAG

FASTQ files:
@ILLUMINA-52179E_0001:3:1:1062:15216#0/2
ATNGGGTTTTTCAACTGCTAAGTCAGCAGGCTTTTCACCCTTCAACATC
+ILLUMINA-52179E_0001:3:1:1062:15216#0/2
OOBOLJ[HHO`_aaa`a_]aaaY[`Za[Y[F]]VZWX]WZ^Z^^^O[XY
@ILLUMINA-52179E_0001:3:1:1062:4837#0/2
ANNAACTCGTGCCGTTAAAGGTGGTCTTGCATTTCAGAAAGCTCACCAG
+ILLUMINA-52179E_0001:3:1:1062:4837#0/2
OBBOO^^^^^bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb`bbbb`

General points:
-Files present in the -l library file should either be in .fastA or .fastQ format, which is automatically determined by the program. For each paired read, one of the reads should be in the first file, and the other one in the second file. The paired reads are required to be on the same line in both files.
-the header (given after "@" character for .fastaQ or ">" for .fastaA) of scaffold and paired-read data files could be of any format. No typical naming convention is needed. Duplicate names are also allowed. 
-Quality values of the fastQ files are not used.
-To be considered, sequences have to be longer than 16 nt or -m (but can be of different lengths).  If they are shorter, the program will simply omit them from the process. 
-Reads containing ambiguous bases, like <N> and <.>, and characters other than ACGT will be ignored entirely in input fasta/fastaq files inserted with -l option.
-Spaces in any .fastq and .fasta file are NOT permitted and will either not be considered or result in execution failure

Output files
------------
Each file is starting with a basename given at the -b parameter. First, four main files are generated in the current working directory;;

(basename).filled.final.txt       :: text file; Detailed information about the gapclosing process. see the next section for information about this file.
(basename).gapclosed.fa	:: fastA file; Gapclosed scaffolds
(basename).closed.evidence.final.fa	:: text file; Evidence of the gapclosing process, including the coverage and the path it has chosen.
(basename).summaryfile.final.txt	:: text file; Summary statistics of the process.


Understanding the .filled.*.txt file
-------------------------------------
A common output of this file would be;

>scaffold1.1|size54656:
	start=4265	end=4288	gapsize=23	reads=599	filled=25	ext1=48	ext2=7	merged=30	remaining=0	closed=yes
	start=8200	end=8233	gapsize=33	reads=554	filled=36	ext1=66	ext2=0	merged=30	remaining=0	closed=yes
	start=19213	end=19342	gapsize=129	reads=445	filled=142	ext1=16	ext2=156	merged=30	remaining=0	closed=yes
>scaffold2.1|size114173:
	start=2115	end=2116	gapsize=1	reads=443	filled=26	ext1=19	ext2=7	merged=0	remaining=1	closed=no
	start=2557	end=2558	gapsize=1	reads=491	filled=29	ext1=17	ext2=12	merged=0	remaining=1	closed=no

After the lines starting with '>' the original names of the scaffold headers are given. Scaffolds that did not have any gaps are not displayed in this file. 

The other lines include nine columns, each line in the file represent one gap;
	1. start: original start of the gap in the scaffold
	2. end: original end of the gap in the scaffold
	3. gapsize: size of the gap in the scaffold
	4. reads: number of reads used for filling the gap
	5. filled: total number of nucleotides in the gap that are filled. This represents 'ext1' + 'ext2' - 'merged'
	6. ext1: number of nucleotides extended from sequence 1
	7. ext2: number of nucleotides extended from sequence 2
	8. merged: number of nucleotides from sequence1 and sequence2 that are merged. Only if closed=yes.
	9. remaining: remaining number of gaps. Only if closed=no. If 'remaining=1', it is likely that there is an overlap, but probably with mismatches which GapFiller could not merge together..
	10. closed: is the gap closed or not?

Understanding the .closed.evidence.*.txt file
-------------------------------------
A common output of this file would be;

>scaffold1gap1
	closed=0/5	dir3: Total:27	A:0	T:0	G:0	C:27 => extending with C
	closed=1/5	dir5: Total:20	A:0	T:20	G:0	C:0 => extending with T
	closed=2/5	dir3: Total:30	A:0	T:0	G:30	C:0 => extending with G
	closed=3/5	dir5: Total:24	A:0	T:0	G:0	C:24 => extending with C
	closed=4/5	dir3: Total:28	A:0	T:0	G:0	C:28 => extending with C
	closed=5/5	dir5: Total:27	A:27	T:0	G:0	C:0 => extending with A
	closed=6/5	dir3: Total:28	A:28	T:0	G:0	C:0 => extending with A
	closed=7/5	dir5: Total:30	A:0	T:0	G:30	C:0 => extending with G
	closed=8/5	dir3: Total:27	A:0	T:0	G:27	C:0 => extending with G
	closed=9/5	dir5: Total:31	A:0	T:1	G:0	C:0 => coverage too low
	closed=10/5	dir3: Total:25	A:1	T:0	G:0	C:24 => extending with C
	sequences are merged

After the lines starting with '>' the original names of the scaffold headers are given. Scaffolds that did not have any gaps are not displayed in this file. 

Information about the columns;
	1. closed 3/5: of the 5 nucleotides long gap, 3 nucleotides are already closed
	2. Dir: Direction of the extension 3' = ------>, 5' = <-------. Thus either left sequence or right sequence of the gap.
	3. Total number of k-mers having -m overlap with the sequence
	4-7. Nucleotide abundance.
	8. Information about the extension, either: "extending with ...", "coverage too low" or "extension stopped"

If two sequences are merged, the line 'sequences are merged' is displayed at the end of the extension, otherwise 'sequences are NOT merged' is displayed.
