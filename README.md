# Root transcriptional dynamics reveal root tissues influenced by developmental timing and genotype in *Phaseolus vulgaris*

Code and processed data for the article:

> **Ortiz Londoño V, Cichy KA, Buell CR, Haus MJ.** Root transcriptional dynamics reveal root tissues influenced by developmental timing and genotype in *Phaseolus vulgaris*. *The Plant Genome* (2026).

**Code author:** Viviana Ortiz Londoño, Root Haus Lab, Michigan State University

---

## Overview

This repository contains the R Markdown scripts used to analyze 3' tag RNA-seq (QuantSeq) data from root and shoot tissues of two common bean genotypes, **Red Hawk** and **Stampede**, sampled at **day 2** and **day 5**.

Tissues: leaf, hypocotyl, root crown, mature and young branched zones of the taproot, apical unbranched zone (taproot and basal roots), and root tips (taproot and basal roots).

The analysis includes:

1. Sample-level QC, variance stabilization, PCA, PERMANOVA, and sample-distance and correlation heatmaps
2. Differential expression (DESeq2) and identification of unique and shared DEGs across tissues, days, and genotypes
3. Functional annotation (Phytozome / biomaRt) and Gene Ontology over-representation analysis (clusterProfiler)
4. Weighted gene co-expression network analysis (WGCNA), hub genes, and expression of marker and suberin genes

## Raw data

Raw sequencing reads are available at the NCBI Sequence Read Archive (SRA); see the Data Availability statement in the article for the BioProject accession.

Reads were processed with the [nf-core/rnaseq](https://nf-co.re/rnaseq/3.14.0) pipeline (STAR + Salmon) against the *P. vulgaris* v2.1 reference genome (Phytozome). Pairwise differential expression tables were produced with the nf-core differential abundance pipeline. The pipeline outputs used by the scripts are in `Results_nf_pipelines/`.

## Repository structure

```
.
├── 1.DEAnalysis/
│   ├── code/
│   │   ├── 1_RDGA_AllTissues_nfstarsalmon.Rmd
│   │   └── 2_RDGA_DifferentialExpression.Rmd
│   ├── data/          # sample metadata
│   ├── figures/       # output figures (png, pdf, svg, tiff)
│   └── output/        # output tables (DEG counts, unique/shared DEGs, Venn intersections)
├── 2.FunctionalAnalysis/
│   ├── code/
│   │   └── 3_RDGA_FunctionalAnalysis.Rmd
│   └── output/        # gene annotations, GO enrichment results
├── 3.NetworkAnalysis/
│   ├── code/
│   │   └── 4_RDGA_NetworkAnalysis.Rmd
│   ├── figures/
│   └── output/        # module assignments, eigengenes, hub genes, edge lists
├── 4.Arabidopsis_orthologs/   # P. vulgaris – A. thaliana ortholog and synteny results
└── Results_nf_pipelines/      # nf-core output used as input (counts, DE tables, contrasts)
```

## Scripts

Run the scripts in order. Each script saves an `.RData` file that the next one loads.

| # | Script | What it does | Main inputs | Main outputs |
|---|--------|--------------|-------------|--------------|
| 1 | `1_RDGA_AllTissues_nfstarsalmon.Rmd` | Imports Salmon gene counts, builds the DESeq2 object, variance stabilization, PCA (all tissues, roots only, and subsets), PERMANOVA, sample-distance and Pearson correlation heatmaps, top variable genes, and number of expressed genes per tissue | `salmon.merged.gene_counts.tsv` / `.rds`, sample metadata (`1.DEAnalysis/data/`) | PCA and heatmap figures, replicate correlation tables, `1_RDGA_Alltissues.RData` |
| 2 | `2_RDGA_DifferentialExpression.Rmd` | Reads the pairwise DESeq2 results from the nf-core pipeline, counts up- and downregulated genes, identifies unique and shared DEGs within each group of contrasts, and makes Venn/UpSet plots and DEG heatmaps | `1_RDGA_Alltissues.RData`, DE tables and contrast file in `Results_nf_pipelines/` | DEG count tables, unique/shared DEG lists, Venn intersections, `2_DE_results_list.RData` |
| 3 | `3_RDGA_FunctionalAnalysis.Rmd` | Annotates genes with GO, KEGG, and Pfam terms from Phytozome (biomaRt), and runs Gene Ontology over-representation analysis on DEG sets with clusterProfiler | `2_DE_results_list.RData`, Venn intersection RDS files | Annotated gene tables, GO enrichment tables and plots, `3_Functional_Analysis.RData` |
| 4 | `4_RDGA_NetworkAnalysis.Rmd` | Builds a signed co-expression network with WGCNA (soft-threshold power = 14), tests module–trait associations (limma), identifies hub genes, and plots expression of root marker and suberin genes | `3_Functional_Analysis.RData` (normalized counts and metadata) | Module assignments, eigengene heatmaps, hub gene tables, edge lists, marker and suberin gene plots |

## How to run

1. Clone or download this repository.
2. Open the project folder in RStudio (or VS Code with the R extension).
3. Install the required packages (see below).
4. Run the scripts in order (1 → 4).

The scripts set the working directory to the project folder (`knitr::opts_knit$set(root.dir = ...)`), so file paths are relative to the repository root.

Some steps take a long time or need a lot of memory (especially the WGCNA network construction). Intermediate results are saved as `.RData` / `.rds` files so you can start from any step.

## Software

Analyses were run in R. Main packages:

- **Bioconductor:** DESeq2, tximport, limma, clusterProfiler, enrichplot, DOSE, biomaRt, AnnotationHub, AnnotationDbi, GO.db, org.At.tair.db, WGCNA, impute, ComplexHeatmap
- **CRAN:** tidyverse, ggplot2, ggpubr, ggrepel, ggnewscale, ggforce, ggVennDiagram, ggvenn, VennDiagram, UpSetR, ggupset, pheatmap, tidyplots, patchwork, vegan, rstatix, igraph, ggraph, tidygraph, flextable, khroma, RColorBrewer, viridis
- **GitHub:** ggConvexHull, ggbiplot

Installation:

```r
install.packages("BiocManager")
BiocManager::install(c("DESeq2", "tximport", "limma", "clusterProfiler", "enrichplot",
                       "DOSE", "biomaRt", "AnnotationHub", "GO.db", "org.At.tair.db",
                       "impute", "ComplexHeatmap"))
install.packages(c("tidyverse", "WGCNA", "ggpubr", "ggrepel", "ggnewscale", "ggforce",
                   "ggVennDiagram", "VennDiagram", "UpSetR", "ggupset", "pheatmap",
                   "tidyplots", "patchwork", "vegan", "rstatix", "igraph", "ggraph",
                   "tidygraph", "flextable", "khroma", "viridis"))
```

The full list of package versions is printed by `sessionInfo()` at the end of the scripts.

## Citation

If you use this code or data, please cite:

> Ortiz Londoño V, Cichy KA, Buell CR, Haus MJ. (2026). Root transcriptional dynamics reveal root tissues influenced by developmental timing and genotype in *Phaseolus vulgaris*. *The Plant Genome*.

