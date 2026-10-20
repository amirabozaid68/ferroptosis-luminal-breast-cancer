# Proliferation- and purity-robust ferroptosis defense shift in Luminal B breast cancer

Analysis code for the manuscript by Amira Abozaid and colleagues.

This repository contains everything needed to reproduce the figures and tables of
the paper from the two public expression datasets. It does not contain the
expression data themselves, which must be obtained from their original
repositories (see **Obtaining the raw data**).

- **Paper:** [to be completed: full citation and DOI once published]
- **Archived release:** [to be completed: Zenodo DOI]
- **Licence:** [to be completed: e.g. MIT for code, CC BY 4.0 for derived tables]
- **Contact:** Amira Abozaid, Biotechnology Program, Egypt-Japan University of
  Science and Technology (E-JUST), Alexandria, Egypt

---

## Contents

| File | Purpose |
|---|---|
| `01_GSE96058_preprocessing.Rmd` | Builds the discovery cohort from the GEO series files |
| `02_METABRIC_preprocessing.Rmd` | Builds the replication cohort from the cBioPortal archive |
| `03_analysis_lean.Rmd` | All analyses reported in the manuscript, and every figure and table |
| `10_CORE_ferroptosis_axis_gene_membership.csv` | Curated ferroptosis axis definitions. An input, not an output |
| `session_info.txt` | Package versions of the run that produced the reported results |
| `README.md` | This file |

`10_CORE_ferroptosis_axis_gene_membership.csv` is the only data file here. The
eight ferroptosis axes were curated by hand before any analysis, and this file is
the sole record of their membership. It is also the source of Supplementary
Table 1. Without it the analysis cannot run.

---

## Requirements

R 4.4.1 or later. The reported results were produced under R 4.4.1 on Windows 11;
`session_info.txt` lists the exact package versions.

```r
install.packages(c(
  # analysis
  "dplyr", "tidyr", "tibble", "readr", "purrr", "matrixStats",
  "survival", "pROC", "coin", "tidyestimate",
  # preprocessing
  "data.table", "readxl", "stringr", "here",
  # figures
  "ggplot2", "patchwork", "ggrepel", "ggtext", "ragg", "systemfonts", "scales"
))
```

`tidyestimate` provides the ESTIMATE purity scores. `coin` provides the stratified
Wilcoxon test. Both are required; the analysis reports reduced output if either is
absent rather than failing, so check the console messages.

---

## Directory layout

Create this structure, put the raw files in the two `01_raw` folders, and run from
the project root. Every other directory is created by the code.

```
<project root>/
├── 01_GSE96058_preprocessing.Rmd
├── 02_METABRIC_preprocessing.Rmd
├── 03_analysis_lean.Rmd
├── 10_CORE_ferroptosis_axis_gene_membership.csv
├── GSE96058/
│   ├── 01_raw/            <- put the GEO files here
│   ├── 02_processed/
│   └── 03_results/{figures,tables}/
├── METABRIC/
│   ├── 01_raw/            <- put the cBioPortal archive here
│   ├── 02_processed/
│   └── 03_results/{figures,tables}/
└── Lean/
    ├── 02_processed/
    └── 03_results/{figures,tables}/      <- all reported output lands here
```

All three documents resolve the project root the same way: `here::here()` when the
`here` package is installed, otherwise the `FERR_ROOT` environment variable, and
otherwise the current working directory. The simplest setup is an RStudio project
at the root, which gives `here::here()` a marker file to find. No path needs
editing.

The axis membership CSV is read from `GSE96058/03_results/tables/`. The copy in
the repository root is for reference; place a copy in that folder, or let
`01_GSE96058_preprocessing.Rmd` regenerate it.

---

## Obtaining the raw data

Neither dataset is redistributed here.

**GSE96058** (discovery, n = 2,084 luminal tumours). The Sweden Cancerome Analysis
Network Breast series, from the Gene Expression Omnibus under accession GSE96058.
Place the gene expression table and the clinical and subtype table in
`GSE96058/01_raw/`. The code finds them by filename pattern: the expression file
must contain `gene_expression` and end in `.csv`, `.txt`, `.tsv` or a gzipped form
of these, and the clinical file must contain `clinical` or `subtype`.

**METABRIC** (replication, n = 1,175 luminal tumours). The Molecular Taxonomy of
Breast Cancer International Consortium cohort, from cBioPortal as the
`brca_metabric` study archive. Place the archive, named `brca_metabric.tar.gz` or
similar, in `METABRIC/01_raw/`. The code detects and extracts it.

Record your download date. Neither source is versioned in a way the code can
capture, so the date is the only provenance a reader has.

---

## How to run

In order. Each document depends on the output of the previous one.

```r
rmarkdown::render("01_GSE96058_preprocessing.Rmd")
rmarkdown::render("02_METABRIC_preprocessing.Rmd")
rmarkdown::render("03_analysis_lean.Rmd")
```

The first two can be run in either order; both must finish before the third.

**Runtime.** The two preprocessing documents take tens of minutes, dominated by
reading the GSE96058 expression table. The analysis document takes a few minutes,
dominated by the 10,000-iteration permutation test in the `defense-correlation`
chunk and the ESTIMATE scoring in `covariates`.

**Reproducibility.** The random seed is fixed at 123 before the permutation
analysis. Everything else is deterministic.

---

## What the analysis document consumes

`03_analysis_lean.Rmd` reads exactly seven files and writes everything else.

From `GSE96058/02_processed/`:
`06_expr_mat_aligned.rds`, `07_clin_standardized.rds`,
`10_ferro_axes_CORE_scoring.rds`, `10_axes_order.rds`

From `GSE96058/03_results/tables/`:
`10_CORE_ferroptosis_axis_gene_membership.csv`

From `METABRIC/02_processed/`:
`09_expr_PAM50_primary_repSAFE.rds`, `09_clin_PAM50_primary_repSAFE.rds`

If any is missing, the document stops with a message naming it.

---

## Output map

Everything reported in the manuscript is written to `Lean/03_results/`.

### Figures (`Lean/03_results/figures/`, as PDF, PNG and TIFF)

| File | Manuscript |
|---|---|
| `Fig1_Axis_Activity_Adjustment` | Figure 1 |
| `Fig2_Discovery_Replication_Concordance` | Figure 2 |
| `Fig3_Gene_Adjustment_Cascade` | Figure 3 |
| `Fig4_Iron_Flux_and_Balance` | Figure 4 |
| `Fig5_GCH1_SLC40A1_by_Subtype` | Figure 5 |
| `FigS1_Covariates_by_Subtype` | Supplementary Figure 1 |
| `FigS2_Defense_Axis_Coexpression` | Supplementary Figure 2 |
| `FigS3_Proliferation_Stratified` | Supplementary Figure 3 |
| `FigS4_Survival_Balance` | Supplementary Figure 4 |
| `Working_Hepcidin_Adjustment` | Not shown. Supports the post hoc hepcidin paragraph |

### Tables (`Lean/03_results/tables/`)

| File | Content |
|---|---|
| `T01_cohort_summary.csv` | Table 1 |
| `T02_covariates_by_subtype.csv` | Table 2 |
| `T03_axis_results_all_adjustments.csv` | Axis effects at all four adjustment levels |
| `T04_frozen_discovery_gene_set.csv` | The 36 genes frozen in discovery |
| `T05_gene_results_all_adjustments.csv` | Gene effects at all four adjustment levels |
| `T06_gene_replication_and_core_rule.csv` | Cross-cohort replication and the robust-core rule |
| `T07_replication_summary.csv` | Concordance summary per adjustment level |
| `T08_iron_flux_and_balance.csv` | Iron subcomponents and the balance score |
| `T09_iron_gene_availability.csv` | Which iron genes were present in each cohort |
| `T10_GCH1_SLC40A1_within_subtype_correlation.csv` | Within-subtype correlation of the two genes |
| `T11_proliferation_stratified_summary.csv` | Pooled within-quintile effects |
| `T12_proliferation_stratified_per_quintile.csv` | Per-quintile effects |
| `T13_defense_coregulation_permutation.csv` | Co-regulation permutation test |
| `T14` to `T18` | Post hoc hepcidin analysis |
| `T19_survival_followup_and_events.csv` | Follow-up and event counts per cohort and endpoint |
| `T20_survival_cox_subtype_stratified.csv` | Cox models across both subtypes |
| `T21_survival_cox_within_subtype.csv` | Cox models within each subtype |
| `T22_survival_subtype_interaction.csv` | Feature-by-subtype interaction tests |
| `T23_balance_score_constraint_LRT.csv` | Likelihood-ratio test of the balance-score constraint |
| `T24_DSS_competing_risks_finegray.csv` | Fine-Gray competing-risks sensitivity analysis |
| `Methods_statistics_notes.txt` | Machine-written summary of the statistical choices |
| `session_info.txt` | Package versions of that run |

Supplementary Table 1 is `10_CORE_ferroptosis_axis_gene_membership.csv`, which is
an input rather than a generated output.

---

## Chunks that do not contribute to the manuscript

The two preprocessing documents are longer than the paper needs, because they also
carry earlier exploratory work from the same project: an estrogen-receptor-positive
against triple-negative comparison, differential co-expression network analysis,
and an independent METABRIC survival analysis later superseded by the one in
`03_analysis_lean.Rmd`. None of it is reported in this paper. The co-expression
network results did not replicate between cohorts and were abandoned.

These chunks are left in place rather than deleted, so that the deposited code is
the project as it actually ran. The seven files listed under **What the analysis
document consumes** are the only preprocessing outputs the paper depends on. The
chunks producing them are:

- `01_GSE96058_preprocessing.Rmd`, chunks 00 to 10
- `02_METABRIC_preprocessing.Rmd`, chunks 00 to 09

Everything after those chunks in each document is exploratory and can be skipped
without affecting any reported result.

`03_analysis_lean.Rmd` contains no abandoned analyses. Its `hepcidin` chunk is a
prespecified post hoc analysis that is reported in the manuscript as a negative
result.

---

## Notes on specific analysis choices

These are documented fully in the manuscript Methods. Three are worth flagging here
because they are easy to miss when reading the code.

**Standardisation is within cohort.** Expression is z-scored gene by gene inside
each cohort separately. No cross-platform normalisation is performed, and no value
is comparable between cohorts in absolute terms. Every reported difference is a
relative position within one cohort.

**Subtype calls are taken as supplied.** PAM50 labels come from the field
distributed with each dataset, not from a reclassification. No classifier was
retrained. Samples labelled outside the five intrinsic classes, including the
claudin-low group recorded in METABRIC, are set to missing and excluded.

**The proliferation adjustment is deliberately conservative.** The PAM50
proliferation module is used as a covariate even though the PAM50 classifier that
defines the two subtypes is itself built in part on those genes. This over-adjusts
and may remove real biology along with confounding, which is why results are
reported at four adjustment levels and repeated without any regression model in the
proliferation-stratified analysis.

---

## Citing

[to be completed: paper citation]

If you use the code or the derived tables, please cite both the paper and the
archived release.
