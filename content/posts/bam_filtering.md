+++
date = '2026-02-18'
title = 'BAM subsampling or: How I Learned to Stop Worrying and Love the AWK'
+++

I've recently been working with some genomic ONT reads that have the kindness to be decently long, and high depth.
Too high depth in fact - about 200x genome size by my estimates.
For assembling with `hifiasm` I don't *think* this causes issues (e.g. degraded assembly quality) apart from an extended run time, but I figured it would be a good idea to subsample the data anyway to save some time for downstream methylation analysis and such.

I wanted to do some pretty straightforward filtering - get rid of super low quality reads and retain the largest reads that accounted for ~50x of the total genome.
For working with FASTQ data this is trivial, with various tools available ([`filtlong`](https://github.com/rrwick/Filtlong) is probably the easiest for this), but I couldn't find any that would support BAM files, which is frustrating as I needed the various DNA modification data and [move tables](https://software-docs.nanoporetech.com/dorado/latest/basecaller/move_table/) for later polishing.

It's probably worth mentioning here that the sensible approach would have been to just try some arbitrary cutoffs until I reached "good enough" coverage, but I wanted a solution I could come back to when genome size or read distribution might be different.

My initial idea was to write a simple filtering program that would simply:

1. Filter out low-quality reads and save the lengths of passing reads to a list.
2. Sort the list, then iterate through and sum until the desired number of bases was reached.
3. Use the landed-on read length as the cutoff, then filter the BAM accordingly.

For this, I went down a bit of a rabbit hole looking at various [htslib](https://github.com/samtools/htslib) library wrappers including [hts-nim](https://github.com/brentp/hts-nim), [rust-htslib](https://github.com/rust-bio/rust-htslib), and [noodles](https://github.com/zaeleus/noodles), the latter being essentially a reimplementation of htslib without the need for non-rust libraries.

I made some progress with each, but after lunch decided I don't actually want to write in rust and simply don't care about reading about reader/writer threadpools today.
My progress with nim extended to installing and running `nimble init` before again being overcome with the feeling of "why am I doing this" and stopping post-post-lunch-lunch.

With a heavy heart, and channeling the spirit of a round-glasses equipped face-bearded flared-trouser wearing 1970s Bell Labs UNIX whisperer, I instead put together a script that solves the issue with a mix of `samtools`, `awk`, and `sort`:

```
#!/bin/env bash

BAM=...
OUT=...

MIN_QS=10
GENOME_SIZE=300000000
COVERAGE=50
TARGET_BASES=$(( GENOME_SIZE * COVERAGE ))

THRESHOLD=$(samtools view -@ $SLURM_CPUS_PER_TASK -e "[qs] >= $MIN_QS" "$BAM" | \
            awk '{print length($10)}' | \
            sort -rn --parallel=$SLURM_CPUS_PER_TASK -S 2G | \
            awk -v t="$TARGET_BASES" 'BEGIN{sum=0} {sum+=$1; if(sum>=t){print $1; exit}}')

echo "Length threshold: $THRESHOLD"

samtools view -@ $SLURM_CPUS_PER_TASK \
    -e "[qs] >= $MIN_QS && length(seq) >= $THRESHOLD" \
    -b -o $OUT $BAM
```

It took 30 minutes against a 150Gb BAM file with 4 threads and the memory of a goldfish, which is pretty good considering it's throwing several million lines at `sort` without much optimisation.
Speaking of optimisation, there is probably some amount of context-switching going on with the number of threads I'm requesting, and I'm not sure if a massive `sort` is actually the best approach, perhaps a binned histogram of lengths followed by walkback in `awk` would have been faster.

I'm not sure if filtering via the `dorado`-generated `qs` tag is the best approach, but it's pretty convenient and falls into that "good enough" category, when estimating read quality is so tricky in the first place.
In fact, I noticed during testing some of the FASTQ filtering tools that there may be some risks associated with filtering based on calculated read scores, for example in [this issue](https://github.com/OpenGene/fastplong/issues/20) for `fastplong` which seems to massively overinflate ONT read scores (Q50 here we come!!!!).

![Fastplong output for ONT reads](/images/fastplong.png)

In summary, everything I write from now on will be unannotated Bash scripts and I encourage everyone to stop using nextflow and instead just use `make` at all times.