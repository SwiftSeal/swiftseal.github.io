+++
date = '2026-06-24'
title = 'Hey Org.At.tair.db - where's my gene?'
+++

When creating summary tables for RNA-seq, I like to provide human readable gene IDs and descriptions alongside the standard TAIR IDs.
The bioconductor package `org.At.tair.db` is good for this:

```
# where results is a df of RNA-seq results with TAIR ids in column "gene"
library(org.At.tair.db)
results$ID <- mapIds(org.At.tair.db, keys = results$gene, column = "SYMBOL", keytype = "TAIR")
results$Description <- mapIds(org.At.tair.db, keys = results$gene, column = "GENENAME", keytype = "TAIR")
```

While looking through some results today, I noticed that the gene `AT3G52870` wasn't being assigned an ID or description despite being described as IQM3 on the [TAIR website](https://www.arabidopsis.org/locus?key=37498).
TAIR periodically release gene IDs on their website, the latest being available [here](https://www.arabidopsis.org/download/file?path=Public_Data_Releases/TAIR_Data_20250331/gene_aliases_20250331.txt.gz).
I had to do some sleuthing on how Bioconductor builds and releases the `org.At.tair.db` data and eventually found the pipeline at https://github.com/Bioconductor/BioconductorAnnotationPipeline.
The issue seems to be that the URL for the gene ID file is hardcoded in https://github.com/Bioconductor/BioconductorAnnotationPipeline/blob/master/annosrc/tair/script/env.sh, and has been stuck on TAIR_Data_20230630 for the past couple years.

You can get an idea of the number of missing gene IDs via:

```
library(readr)
library(dplyr)
al <- read_tsv("gene_aliases_20250331.txt",
               col_names = c("locus","symbol","full_name"),
               col_types = "ccc", skip = 1)

tair_syms <- al |> filter(!is.na(symbol)) |> distinct(locus)
pkg <- mapIds(org.At.tair.db, tair_syms$locus, "SYMBOL", "TAIR")

sum(is.na(pkg)); mean(is.na(pkg))
# [1] 3479
# [1] 0.1880541
```

So about 1/5th of currently available IDs are missing - though I'm sure there are plenty of minor differences that are being picked up on in this coarse check.
I've never really dug into the inner workings of Bioconductor packages and found them fairly opaque, so it took a while to figure out where the package data comes from etc.
I tried to reach out via the support forum, but it has a habit of sending me 502s and 504s so I just left an issue on the github repo to see if it gets picked up (assuming I'm not completely off track...).
In the meantime, I'd probably use the latest TAIR files directly for handling this, although I'm not a fan of constantly tracking down and saving files into my working directory.

I think TAIR is also slightly to blame as in general their strategy for releasing gene data is quite confusing - I have a mini-rant saved for another time on the various formatting issued I've encountered with the latest TAIR12 `.gff` release...

Anyway, a good hours worth of [yak shaving](https://en.wiktionary.org/wiki/yak_shaving) - back to the heatmap I was supposed to be making :-)
