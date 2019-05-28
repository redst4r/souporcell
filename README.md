# souporcell

souporcell is a method for clustering mixed-genotype scRNAseq experiments by individual.

The inputs are just the possorted_genome_bam.bam (with index), and barcodes.tsv as output from cellranger.
souporcell is comprised of 6 steps with the first 3 using external tools and the final using the code provided here.
1. Remapping (minimap2, https://github.com/lh3/minimap2)
2. Calling candidate variants (freebayes, https://github.com/ekg/freebayes)
3. Cell allele counting (vartrix, https://github.com/10XGenomics/vartrix)
4. Clustering cells by genotype (souporcell.py)
5. Calling doublets (troublet)
6. Calling cluster genotypes and inferring amount of ambient RNA (consensus.py)

## 1. Remapping
We discuss the need for remapping in our manuscript (to be posted on biorxiv soon). We need to keep track of cell barcodes and and UMIs, so we first create a fastq with those items encoded in the readname.
Requires python 3.0, modules pysam, argparse (pip install/conda install depending on environment)
Easiest to first add the souporcell directory to your PATH variable with export PATH=/path/to/souporcell:$PATH
***
> python renamer.py --bam possorted_genome_bam.bam --barcodes barcodes.tsv --out fq.fq
***
Then we must remap these reads using minimap2 (similar results have been seen with hisat2)
***
> minimap2 -ax splice -t 8 -G50k -k 21 -w 11 --sr -A2 -B8 -O12,32 -E2,1 -r200 -p.5 -N20 -f1000,5000 -n2 -m20 -s40 -g2000 -2K50m --secondary=no <reference_fasta_file> fq3.fq > minimap.sam
***
(note the -t 8 as the number of threads, change this as needed)
Now we must retag the reads with their cell barcodes and UMIs
***
> python retag.py --sam minimap.sam --out minitagged.bam
***
Then we must sort and index our bam
***
> samtools sort minitagged.bam > minitagged_sorted.bam
> samtools index minitagged_sorted.bam
***



