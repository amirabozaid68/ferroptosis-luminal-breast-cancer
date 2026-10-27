# GCH1 predicts survival within luminal A but not luminal B breast cancer

Analysis code for the manuscript by Amira Abozaid and colleagues.

This repository contains everything needed to reproduce the figures and tables of
the paper from the two public expression datasets. It does not contain the
expression data themselves, which must be obtained from their original
repositories (see **Obtaining the raw data**).

- **Paper:** [to be completed: full citation and DOI once published]
- **Archived release:** [10.5281/zenodo.23492209](https://doi.org/10.5281/zenodo.23492209)
  — this is the concept DOI and always resolves to the newest version. Version
  v1.1.0 ([10.5281/zenodo.23685813](https://doi.org/10.5281/zenodo.23685813)) is
  the one that accompanies the submitted manuscript.
- **Licence:** MIT (see `LICENSE`)
- **Contact:** Amira Abozaid, Biotechnology Program, Egypt-Japan University of
  Science and Technology (E-JUST), Alexandria, Egypt.
  <amira.abozaid@ejust.edu.eg>

---

## Contents

| File | Purpose |
|---|---|
**Scripts**

| File | Purpose |
|---|---|
| `01_GSE96058_preprocessing.Rmd` | Builds the discovery cohort from the GEO series files |
| `02_METABRIC_preprocessing.Rmd` | Builds the replication cohort from the cBioPortal archive |
| `03_analysis_lean.Rmd` | All analyses reported in the manuscript, and every figure and table |
| `04_coupling_prior_TCGA_GTEx.Rmd` | GCH1 and SLC40A1 correlation in GTEx normal breast, TCGA adjacent normal and TCGA luminal tumours. Supports the non-replication statement in the Limitations. Not part of the main analysis |
| `revision_survival_checks.Rmd` | Cause-of-death analyses added during revision. See **Revision analyses** |

**Inputs**

| File | Purpose |
|---|---|
| `10_CORE_ferroptosis_axis_gene_membership.csv` | Curated ferroptosis axis definitions. An input, not an output |
| `PRESPECIFIED_revision_analyses.md` | Analysis plan for the revision analyses, committed before they were run |

**Deposited output**

| Folder or file | Content |
|---|---|
| `results_tables/` | T01 to T24, every table behind the reported numbers |
| `revision/` | R00 to R04, output of the revision analyses |
| `coupling_prior_output/` | Output of `04_coupling_prior_TCGA_GTEx.Rmd`, including `VERDICT.txt` |
| `Methods_statistics_notes.txt` | Machine-written summary of the statistical choices |
| `session_info.txt` | Package versions of the run that produced the reported results |
| `LICENSE` | MIT |
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

Two further documents stand outside that chain and can be run at any time.
`revision_survival_checks.Rmd` needs the METABRIC objects produced by
`02_METABRIC_preprocessing.Rmd`. `04_coupling_prior_TCGA_GTEx.Rmd` needs neither
preprocessing document and no local data at all, because it fetches what it needs
from the UCSC Xena hubs; it does need an internet connection.

**Runtime.** The two preprocessing documents take tens of minutes, dominated by
reading the GSE96058 expression table. The analysis document takes a few minutes,
dominated by the 10,000-iteration permutation test in the `defense-correlation`
chunk and the ESTIMATE scoring in `covariates`.

**Chunk order.** `03_analysis_lean.Rmd` runs: `setup`, `load-cohorts`,
`covariates`, `axis-level`, `gene-level`, `iron-and-balance`, `hepcidin`,
`stratified`, `defense-correlation`, `fig-helpers`, `survival`, `figures`,
`save`. The `fig-helpers` chunk defines the shared scales and label functions and
must run before `survival`, because Figure 6 is drawn there.

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

### Figures (written to `Lean/03_results/figures/`, as PDF, PNG and TIFF)

| File | Manuscript |
|---|---|
| `Fig1_Covariates_by_Subtype` | Figure 1 |
| `Fig2_Axis_Activity_Adjustment` | Figure 2 |
| `Fig3_Gene_Adjustment_Cascade` | Figure 3 |
| `Fig4_Iron_Flux_and_Balance` | Figure 4 |
| `Fig5_GCH1_SLC40A1_by_Subtype` | Figure 5 |
| `Fig6_Survival_Within_Subtype` | Figure 6 |
| `FigS1_Discovery_Replication_Concordance` | Supplementary Figure 1 |
| `FigS2_Defense_Axis_Coexpression` | Supplementary Figure 2 |
| `FigS3_Proliferation_Stratified` | Supplementary Figure 3 |
| `FigS4_Hepcidin_Adjustment` | Supplementary Figure 4 |

Figure 6 is built in the `survival` chunk, not in `figures`, because it needs the
fitted Cox models. Supplementary Figure 4 is built in the `hepcidin` chunk. The
remaining eight are built in `figures`.

### Tables (written to `Lean/03_results/tables/`, deposited here in `results_tables/`)

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

The paths above are where the code writes on the machine that runs it. The
deposited copies of the tables are in `results_tables/`, so a reader can check any
reported number without running the pipeline. Figures are not deposited, because
they are in the manuscript.

---

## Revision analyses

These address one question the main analysis left open: whether the
overall-survival association within luminal A reflects death from breast cancer
or death from other causes. METABRIC records cause of death, GSE96058 does not, so
they are confined to the replication cohort.

The plan is in `PRESPECIFIED_revision_analyses.md`, committed before any of the
models below were fitted. The git history is the timestamp. The code is
`revision_survival_checks.Rmd` and the output is in `revision/`.

| File | Content |
|---|---|
| `R00_reproduction_check.csv` | Reproduces the published within-luminal-A estimate before anything is changed. HR 1.23 (1.11 to 1.38), matching the manuscript |
| `R01_cause_specific_LumA.csv` | Cause-specific hazards for breast cancer death and other-cause death, with the Lunn-McNeil test of whether the two differ |
| `R02_finegray_LumA.csv` | Fine-Gray subdistribution hazards fitted within luminal A alone |
| `R03a_age_correlation_LumA.csv` | Correlation of each feature with age at diagnosis |
| `R03b_OS_under70_LumA.csv` | Overall survival restricted to patients under 70 |
| `R04_RFS_LumA.csv` | Relapse-free survival within luminal A |

One luminal A patient has no recorded cause of death and is excluded from the
cause-specific and Fine-Gray analyses.

**What they show.** Under the primary adjustment the balance score is associated
with other-cause death (HR 1.31, 95% CI 1.14 to 1.51) and not with breast cancer
death (HR 1.11, 0.92 to 1.33), and GCH1 behaves the same way (1.31, 1.13 to 1.51
against 1.13, 0.93 to 1.37). The Lunn-McNeil test does not reject equality of the
two cause-specific effects (P = 0.146 and P = 0.228), so the data are consistent
with a difference between causes but do not establish one. That limitation is
stated in the manuscript.

`R02` supersedes the competing-risks analysis reported in the main text. The
original Fine-Gray model was fitted across both luminal subtypes together, because
`survival::finegray()` does not accept a stratification term. `R02` fits it within
luminal A by subsetting instead, giving subdistribution HR 1.03 (0.88 to 1.21) for
both features under the primary adjustment. The conclusion does not change.

Age is a partial explanation and not a complete one. The balance score rises with
age at diagnosis (Spearman rho 0.151, P = 5.9e-05) while GCH1 does not (rho 0.018,
P = 0.626). Restricting to patients under 70 leaves GCH1 associated with overall
survival (HR 1.27, 1.09 to 1.50) and weakens the balance score (1.15, 0.98 to
1.33), so the composite's association is age-dependent in a way GCH1's is not.

Relapse-free survival within luminal A supports neither feature strongly (HR 1.10,
0.94 to 1.27 for the balance score and 1.15, 0.98 to 1.34 for GCH1 under the
primary adjustment).

---

## The coupling prior analysis, a negative result

`04_coupling_prior_TCGA_GTEx.Rmd` was run after the main analysis, to test whether
the inverse correlation between GCH1 and SLC40A1 seen within luminal A in
GSE96058 and METABRIC also holds in normal breast tissue and in a third tumour
cohort. It does not. Output is in `coupling_prior_output/`, and `VERDICT.txt`
states the conclusion in plain words.

Adjusted for epithelial content, the correlation is +0.052 in GTEx normal breast
(n = 80), +0.442 in TCGA adjacent normal (n = 113), +0.024 in TCGA luminal A
(n = 420) and +0.148 in TCGA luminal B (n = 192). The prediction was a negative
correlation in normal breast and in luminal A. It failed.

Two cautions a reader should apply to those numbers. GTEx at n = 80 has a
confidence interval of -0.170 to +0.268, which contains the pooled tumour estimate
of -0.121, so the GTEx result is uninformative rather than negative. And the
strongly positive value in adjacent normal may be technical: the epithelial
adjustment barely moved it, 0.431 raw to 0.442 adjusted, and RNA quality varying
across 113 archival normals inflates correlation between any two genes.

This analysis is deposited because the manuscript Limitations cites it. The
planned matched-normal tissue study was not started as a result. The script is
exploratory and was not preregistered; its decision rule was fixed before the run
and its prediction was not, which the file states in its own header.

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

`03_analysis_lean.Rmd` contains no abandoned analyses. Its `hepcidin` chunk is an
exploratory post hoc analysis, run after the prespecified analyses were complete,
and is reported in the manuscript as a negative result and as Supplementary
Figure 4.

`04_coupling_prior_TCGA_GTEx.Rmd` is also post hoc, returned a negative result,
and is deposited for the reason given above rather than because the paper depends
on it.

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

Abozaid, A. (2026). Analysis code for *GCH1 predicts survival within luminal A
but not luminal B breast cancer* (v1.1.0) [Software]. Zenodo.
https://doi.org/10.5281/zenodo.23492209

If you use the code or the derived tables, please cite both the paper and the
archived release.

## Licence

The code in this repository is released under the MIT Licence; see `LICENSE` for
the full text. The derived tables in `Lean/03_results/tables/` are covered by the
same licence. Neither expression dataset is redistributed here, and each remains
under the terms set by its original repository.
