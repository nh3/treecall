# treecall
Tree-based joint lineage inference and somatic mutation calling

## Workflow
Assume that we start with a set of mutations in the file `recallVAF_filtered.vcf.gz`

The typical treecall workflow consists of three steps,

1. Infer an initial tree relating the samples

```
# Using a neighbor-joining approach,
python treecall.py nbjoin -m 60 recallVAF_filtered.vcf.gz recall.nbjoin
```

```
# Or using a top-down partitioning approach,
python treecall.py part -m 60 recallVAF_filtered.vcf.gz recall.part
```

2. Jointly genotype the variants with the help of a lineage tree

```
python treecall.py gtype -t recall.partition.nj.nwk \
    -m 60 \
    recallVAF_filtered.vcf.gz \
    recall.partition.gtcall
```

3. Annotate the lineage tree with genotype calls

```
python treecall.py annot -t recall.partition.nj.nwk \
    recall.partition.gtcall \
    recall.partition.annotation
```

More information about the individual commands is below.

## Commands
```
usage: treecall.py [-h] <command> ...

positional arguments:
  <command>   sub-commands
    tview     view tree
    compare   compare tree topology
    compat    calculate pairwise compatibility between all pairs of sites
    nbjoin    neighbor-joining
    part      a top-down method that recursively partition samples based on partition cost
    gtype     genotype samples with help of a lineage tree
    annot     annotate lineage tree with genotype calls

optional arguments:
  -h, --help  show this help message and exit
```

### tview
```
usage: treecall.py tview [-h] [-a STR] [-l FILE] <nwk>

positional arguments:
  <nwk>       input tree in Newick format

optional arguments:
  -h, --help  show this help message and exit
  -a STR      node attributes to print given by a comma separated list
  -l FILE     leaves label
```

### compare
```
usage: treecall.py compare [-h] -t FILE [FILE ...] -r FILE

optional arguments:
  -h, --help          show this help message and exit
  -t FILE [FILE ...]  input tree(s), in Newick format
  -r FILE             reference tree, in Newick format
```

### compat
```
usage: treecall.py compat [-h] [-v INT] <vcf> <output>

positional arguments:
  <vcf>       input vcf/vcf.gz file, "-" for stdin
  <output>    output compatibility matrix

optional arguments:
  -h, --help  show this help message and exit
  -v INT      minimum evidence in Phred scale for a site to be considered, default 60
```

### nbjoin
```
usage: treecall.py nbjoin [-h] [-m INT] [-e INT] [-v INT] <vcf> output

positional arguments:
  <vcf>       input vcf/vcf.gz file, "-" for stdin
  output      output basename

optional arguments:
  -h, --help  show this help message and exit
  -m INT      mutation rate in Phred scale, default 80
  -e INT      heterozygous rate in Phred scale, default 30
  -v INT      minimum evidence in Phred scale for a site to be considered, default 60
```

### part
```
usage: treecall.py part [-h] [-m INT] [-e INT] [-v INT] <vcf> <output>

positional arguments:
  <vcf>       input vcf/vcf.gz file, "-" for stdin
  <output>    output basename

optional arguments:
  -h, --help  show this help message and exit
  -m INT      mutation rate in Phred scale, default 80
  -e INT      heterozygous rate in Phred scale, default 30
  -v INT      minimum evidence in Phred scale for a site to be considered, default 60
```

### gtype
```
usage: treecall.py gtype [-h] -t FILE [-n INT] [-m INT] [-e INT] <vcf> <output>

positional arguments:
  <vcf>       input vcf/vcf.gz file, "-" for stdin
  <output>    output basename

optional arguments:
  -h, --help  show this help message and exit
  -t FILE     lineage tree
  -n INT      number of sites processed once, default 1000
  -m INT      mutation rate in Phred scale, default 80
  -e INT      heterozygous rate in Phred scale, default 30, 0 for uninformative
```

### annot
```
usage: treecall.py annot [-h] -t FILE <gtcall> <outnwk>

positional arguments:
  <gtcall>    input gtype calls, "-" for stdin
  <outnwk>    output tree in Newick format

optional arguments:
  -h, --help  show this help message and exit
  -t FILE     lineage tree
```

## Output files

Some of the output files are explained below,

### gtcall file

The columns in the `*.gtcall` file are as follows,

1. chromosome
2. position
3. reference allele
4. null_P
5. mut_P
6. MLE_null_base_gtype
7. MLE_null_base_gtype_P
8. MLE_mut_base_gtype
9. MLE_mut_alt_gtype
10. MLE_mut_base_gtype_P
11. MLE_mut_location
12. MLE_mut_samples
