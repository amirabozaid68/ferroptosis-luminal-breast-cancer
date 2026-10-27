# README additions

Two blocks to paste into `README.md`. The first extends the existing Contents
table. The second is a new section, placed after **Output map** and before
**Chunks that do not contribute to the manuscript**.

Correct the filenames marked `<<check>>` against what is actually in the
repository before committing.

---

## BLOCK 1 — add these rows to the Contents table

```markdown
| `04_coupling_prior_TCGA_GTEx.Rmd` | GCH1 and SLC40A1 correlation in GTEx normal breast, TCGA adjacent normal and TCGA luminal tumours. Supports the non-replication statement in the Limitations. Not part of the main analysis |
| `PRESPECIFIED_revision_analyses.md` | Analysis plan for the revision analyses, written and committed before those analyses were run |
| `R00` to `R04` CSV files | Output of the revision analyses. See **Revision analyses** below |
| `<<check>>_revision_analyses_summary.txt` | Machine-written summary of the revision analyses, generated 2026-10-26 |
| `<<check>>_revision_analyses.Rmd` | Code that produced the `R00` to `R04` outputs |
```

---

## BLOCK 2 — new section

```markdown
---

## Revision analyses

These analyses address one question the main analysis left open: whether the
overall-survival association within luminal A reflects death from breast cancer
or death from other causes. METABRIC records cause of death, GSE96058 does not,
so they are confined to the replication cohort.

The analysis plan is in `PRESPECIFIED_revision_analyses.md`, committed before
any of the models below were fitted. The git history is the timestamp.

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

### What they show

Under the primary adjustment, the balance score is associated with other-cause
death (HR 1.31, 95% CI 1.14 to 1.51) and not with breast cancer death (HR 1.11,
0.92 to 1.33), and GCH1 behaves the same way (1.31, 1.13 to 1.51 against 1.13,
0.93 to 1.37). The Lunn-McNeil test does not reject equality of the two
cause-specific effects (P = 0.146 and P = 0.228), so the data are consistent with
a difference between causes but do not establish one. That limitation is stated
in the manuscript.

`R02` supersedes the competing-risks analysis reported in the main text. The
original Fine-Gray model was fitted across both luminal subtypes together,
because `survival::finegray()` does not accept a stratification term. `R02`
fits it within luminal A by subsetting instead, giving subdistribution HR 1.03
(0.88 to 1.21) for both features under the primary adjustment. The conclusion
does not change.

Age is a partial explanation and not a complete one. The balance score rises with
age at diagnosis (Spearman rho 0.151, P = 5.9 x 10-5) while GCH1 does not (rho
0.018, P = 0.626). Restricting to patients under 70 leaves GCH1 associated with
overall survival (HR 1.27, 1.09 to 1.50) and weakens the balance score (1.15,
0.98 to 1.33), so the composite's association is age-dependent in a way GCH1's is
not.

Relapse-free survival within luminal A supports neither feature strongly
(HR 1.10, 0.94 to 1.27 for the balance score and 1.15, 0.98 to 1.34 for GCH1
under the primary adjustment).

### A note on the summary file

The machine-written summary applies an automatic label to each cause-specific
comparison. One of those labels is wrong. For the balance score under
clinicopathological adjustment, breast cancer death gives HR 1.01 (0.84 to 1.21)
and other-cause death HR 1.13 (0.98 to 1.31), and the summary reads
"Cancer-specific". Neither estimate is significant and the point estimate is
higher for other-cause death, so that row carries no information either way. The
labelling rule falls through to "Cancer-specific" when neither confidence
interval excludes 1, which is a defect in the rule and not a result. The
manuscript does not rely on that row.
```
