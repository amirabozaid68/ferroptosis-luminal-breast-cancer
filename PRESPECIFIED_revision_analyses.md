# Prespecified revision analyses

Written and committed before any of the analyses below were run. Existing analyses in the main analysis document are not modified, and every result below is reported whatever it shows.

## Common settings

- Features tested: the GCH1 minus SLC40A1 balance score and GCH1 alone, each z-scored within cohort, at the primary adjustment level (residualised on the PAM50 proliferation module and the ESTIMATE score, as in the main analysis).
- Each model is fitted with and without the METABRIC clinical covariates used in the main analysis (age at diagnosis, Nottingham Prognostic Index, endocrine therapy, chemotherapy).
- A model is fitted only where at least 10 events are available per estimated parameter. Where this fails, the model is reported as not fitted.
- Hazard ratios are per one standard deviation, with 95% confidence intervals. Two-sided tests, alpha 0.05.

## Check before any new analysis

The add-on script must reproduce the published METABRIC luminal A overall survival estimate for the balance score (HR 1.23, 95% CI 1.11 to 1.38). If it does not match to two decimal places, no new analysis is run until the difference is explained.

## A. Cause of death within luminal A (METABRIC), decisive analysis

1. Cause-specific Cox model for death from breast cancer (other causes censored).
2. Cause-specific Cox model for death from other causes (breast cancer deaths censored).
3. Lunn-McNeil stacked model: data duplicated by cause, feature-by-cause interaction tested. This is the formal test of whether the feature's effect differs between the two causes.

Patients whose cause of death is missing are excluded from A and their number is reported.

Reading, fixed in advance:
- Cancer-specific: the other-cause hazard ratio has a 95% CI including 1, and the breast-cancer hazard ratio is above 1.
- Not cancer-specific: the other-cause hazard ratio is above 1 with a 95% CI excluding 1, and the Lunn-McNeil interaction is not significant.
- Any other pattern is reported as unresolved.

## B. Competing risks within luminal A (METABRIC)

Fine-Gray subdistribution model for breast cancer death with other-cause death as the competing event, fitted in luminal A tumours only. This replaces the cross-subtype Fine-Gray model as the within-subtype estimate. The earlier model stays in the supplement.

## C. Age (METABRIC luminal A)

1. Spearman correlation of each feature with age at diagnosis.
2. Overall survival model repeated in patients younger than 70 at diagnosis.

Reading: if the association holds in patients younger than 70 with a hazard ratio in the same direction, age-related comorbidity is not considered the explanation.

## D. Relapse-free survival within luminal A (METABRIC)

Report the within-luminal A hazard ratio for relapse-free survival, which the main analysis tested only through the interaction.

## E. TCGA-BRCA locked replication (to be completed before TCGA data are downloaded)

- PAM50 calls as distributed with TCGA. Endpoints from the TCGA Pan-Cancer Clinical Data Resource (Liu et al., 2018). Progression-free interval primary, overall survival secondary.
- The main pipeline is run unchanged, reading the TCGA input files.
- One prespecified test: the luminal A hazard ratio and the feature-by-subtype interaction for the balance score, at the primary adjustment level.
- No clinicopathological model is required in TCGA.
