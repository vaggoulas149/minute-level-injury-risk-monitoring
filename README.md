# Landmark-Based Discrimination of Injury-Associated Athlete-Sessions

Reproducible analysis code for a landmark-based study using minute-resolution multimodal football monitoring data.

## Study question

SoccerMon provides minute-resolution monitoring signals, but the injury supervision used in this study is available only at the athlete-session level and does not include an exact within-session injury-onset timestamp.

Accordingly, this repository does **not** treat every minute as an independently supervised injury label. Instead, it evaluates fixed elapsed-time landmarks (10, 20, 30, 40, 50, and 60 minutes). At each landmark, one representation is constructed per athlete-session using only data available up to that point, while the target remains the same session-level injury-associated indicator.

The analysis therefore asks how discrimination of the same athlete-session target changes as more within-session information becomes available. It does **not** estimate minute-specific injury risk, localize injury onset, or evaluate an operational alerting policy.

## Cohort

The reconstructed 2020 resource contains:

- 3,743 athlete-sessions;
- 48 athletes;
- 22 injury-associated athlete-sessions;
- 5 athletes contributing all positive sessions.

All positive sessions occur in Team A. The primary supervised modelling cohort therefore contains 2,259 Team-A athlete-sessions from 27 athletes. Team B contains no positive sessions and is not used as an injury-positive external validation cohort.

## Canonical feature representation

The primary representation is **CUM+DYN (63 features)**:

- 30 cumulative within-session features;
- 33 dynamic within-session features.

A separate 14-variable PRE contextual family is used in controlled ablation analyses.

## Notebook execution order

Run notebooks in this exact order:

1. `01_Data_Loading_Preprocessing_2020.ipynb`
2. `02_Master_Session_EDA_2020.ipynb`
3. `03_Landmark_Dataset_Preprocessing_2020.ipynb`
4. `04_Landmark_Modelling_2020.ipynb`
5. `05_TabPFN_Benchmark.ipynb`
6. `06_Augmentation_Benchmark.ipynb`

Each notebook reads canonical artifacts produced by upstream notebooks rather than reconstructing folds, features, or baselines independently.

## Data configuration

The raw SoccerMon dataset is not included in this repository.

Notebook 01 currently contains the author's local default raw-data path for convenience. Before public release, either:

- keep that path only in a private/local branch; or
- replace it with an environment-variable/config-file default for a fully portable public repository.

The public repository should never contain private credentials, access tokens, or restricted raw data.

## Primary evaluation design

The main benchmark uses five deterministic athlete-disjoint outer folds. Each fold holds out exactly one athlete with positive sessions plus a deterministic subset of negative athletes.

Performance is measured on pooled held-out athlete-session predictions using ROC-AUC and average precision. Uncertainty and model contrasts use athlete-cluster bootstrap resampling of the out-of-fold predictions.

Because only five athletes contribute positive sessions, all inferential claims are necessarily limited by the effective positive-cluster count.

## Additional analyses

The repository includes:

- common-cohort sensitivity restricted to sessions observable through 60 minutes;
- 100 alternative allocations of negative athletes across outer folds;
- equal-athlete-weighted sensitivity;
- controlled feature-family ablations;
- Logistic Regression, Random Forest, XGBoost, and TabPFN benchmarks;
- paired athlete-cluster bootstrap model comparisons;
- SMOTE and CTGAN training-set augmentation;
- leave-one-positive-athlete-out augmentation sensitivity;
- equal-athlete-weighted augmentation sensitivity;
- CTGAN memorization/drift diagnostics.

Synthetic observations are used only as training-set interventions and are never counted as new independent athletes, sessions, or injury events.

## Reproducibility status

All six notebooks have been cleaned to use explicit input/output contracts and have been run successfully in the author's environment. Remaining release-level caveats are documented in `docs/reproducibility_notes.md`.

## Repository structure

```text
notebooks/   Canonical analysis notebooks
docs/        Reproducibility and provenance notes
env/         Environment specifications
results/     Generated locally by the notebooks; large outputs should not be committed unless intentionally versioned
```

## Claims discipline

The study does not support claims of:

- minute-specific injury probability;
- known within-session injury onset;
- causal effects of workload or sensor variables;
- external injury-positive validation;
- synthetic-data fidelity;
- universal superiority of TabPFN or augmentation.

The strongest defensible contribution is methodological: aligning the supervised unit of analysis with the resolution of the available injury labels while preserving minute-resolution monitoring information through fixed landmark representations.
