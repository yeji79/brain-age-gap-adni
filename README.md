# Structural MRI Brain Age Gap in the Alzheimer's Spectrum

> A reproducible, leakage-controlled machine-learning project that learns normal brain ageing from structural MRI and evaluates Brain Age Gap (BAG) across the Alzheimer's clinical spectrum.

[한국어 README](README_ko.md)

Public Korean documents: [project overview](docs/PROJECT_OVERVIEW_KO.md) and [research design & roadmap](docs/RESEARCH_DESIGN_AND_ROADMAP_KO.md)

## Project at a glance

This project uses ADNI ADSP PHC ComBat-harmonized FreeSurfer structural MRI ROI data to train a chronological-age prediction model in cognitively normal (CN) participants only. The frozen model is then applied to CN, MCI, and Dementia groups to calculate the bias-corrected Brain Age Gap (BAG).

**Primary question:** After accounting for chronological age, sex, education, and intracranial volume, do MCI and Dementia groups show a higher BAG than CN participants?

**Secondary question:** In the PET-available subset, is BAG associated with amyloid positivity after accounting for clinical diagnosis and demographic covariates?

## Beginner-oriented notebook design

The notebooks are written as guided learning and research records, not only as code that produces an output. Markdown sections explain what each stage does, why it is needed to prevent misleading results, and how to interpret the biological and statistical meaning of the result. The workflow deliberately progresses from MRI MVP to Amyloid PET, normative ROI analysis, and cortical-surface visualisation.

## Current status

**Last updated: 2026-08-20**

| Phase | Status | Next check |
|---|---|---|
| Structural MRI MVP | Executed and interpreted | Sensitivity analysis using held-out CN test as the reference group |
| Amyloid PET extension | Notebook implemented | Local execution and MRI–PET alignment review |
| Normative ROI analysis | Designed | Implement after PET results are confirmed |
| Cortical surface visualisation | Designed | Implement after ROI and sensitivity definitions are fixed |

## Why Brain Age Gap?

```text
BAG = bias-corrected predicted brain age − chronological age
```

A positive BAG means that the participant's structural MRI pattern resembles that of an older CN reference brain than would be expected from chronological age alone. BAG is a research metric of structural deviation from the reference model; it is **not** a clinical diagnosis, a causal biomarker, a future-risk score, or a direct measurement of individual ageing rate.

## Study design

```text
PHC structural MRI + demographics + clinical diagnosis
                    ↓
One earliest valid index MRI per participant (RID)
                    ↓
CN-only train / validation / held-out test split by RID
                    ↓
Dummy baseline + Ridge primary model + non-linear comparator
                    ↓
Bias correction fitted only on validation CN participants
                    ↓
Frozen model applied to CN / MCI / Dementia
                    ↓
Diagnosis-adjusted BAG analysis (ANCOVA)
                    ↓
Optional extensions: amyloid PET, normative ROI Z-scores, sensitivity analyses
```

### Safeguards against misleading results

- **Participant-level split:** the same `RID` never appears in more than one of train, validation, or test.
- **CN-only reference model:** MCI and Dementia data are excluded from model fitting, hyperparameter tuning, and bias correction.
- **Train-only preprocessing:** ROI completeness filtering, median imputation, scaling, and encoding are fit inside the training pipeline only.
- **Frozen calibration:** the age-bias correction is estimated on validation CN participants, then fixed before use in the held-out test and clinical groups.
- **Held-out test:** the CN test set is used only for final performance reporting.
- **Conservative interpretation:** feature importance reflects contribution to age prediction, not disease causality.

## Data

This repository does **not** include ADNI raw data or participant-level processed data. Access and use must follow the [ADNI Data Use Agreement](https://adni.loni.usc.edu/).

Place locally downloaded ADNI data under `data/adni_all/`:

```text
data/adni_all/
├── ADSP_PHC/
│   ├── ADSP_PHC_T1_FS_22Jan2026.csv
│   ├── ADSP_PHC_T1_FS_DATADIC_22Jan2026.csv
│   └── ADSP_PHC_PET_Amyloid_Simple_22Jan2026.csv  # PET extension
├── Assessments/
│   └── DXSUM_22Jan2026.csv
├── Subject_Characteristics/
│   └── PTDEMOG_22Jan2026.csv
```

| Dataset | Role in this project |
|---|---|
| `ADSP_PHC_T1_FS` | Main MRI input: ComBat-harmonized cortical thickness, cortical volume, and subcortical ROI features; MRI age, sex, diagnosis, and scan date |
| `ADSP_PHC_T1_FS_DATADIC` | Anatomical ROI labels and variable metadata |
| `PTDEMOG` | Education-years covariate |
| `DXSUM` | Fallback clinical diagnosis aligned to MRI date when PHC diagnosis is missing |
| `ADSP_PHC_PET_Amyloid_Simple` | PET QC, amyloid negative/positive status, Centiloid, and PET scan date |

The MVP selects one earliest valid T1 MRI (`index MRI`) per `RID`. It uses `PHC_Age_T1` as the MRI-timepoint age. Diagnosis is taken from PHC when available and otherwise filled from the nearest DXSUM assessment within ±180 days.

## Models and evaluation

The primary model is **Ridge regression**, chosen because structural MRI ROIs are correlated and the model is stable and interpretable for tabular neuroimaging data. `DummyRegressor` establishes the no-MRI baseline, and `ExtraTreesRegressor` is a non-linear comparator.

CN participants are split by `RID` into:

- train: 70% — five-fold `GroupKFold` tuning and pipeline fitting
- validation: 15% — age-bias correction only
- held-out test: 15% — final evaluation only

Reported held-out CN test metrics:

- MAE and RMSE in years
- R²
- MAE improvement over the dummy baseline
- correlation between bias-corrected BAG and chronological age

The primary clinical analysis is:

```text
BAG ~ diagnosis × AGE + sex + education + eTIV
```

The current MVP retains ComBat ROI values as provided. It does not perform an additional simple `volume / ICV` ratio transform; `EstimatedTotalIntraCranialVol_combat` is considered as an ANCOVA covariate.

## Repository structure

```text
brain-age-gap-adni/
├── data/
│   ├── adni_all/              # Local ADNI source data — ignored by Git
│   └── processed/             # Participant-level derived files — ignored by Git
├── docs/
│   ├── PROJECT_OVERVIEW_KO.md
│   └── RESEARCH_DESIGN_AND_ROADMAP_KO.md
├── notebooks/
│   ├── 01_mri_brain_age_bag_mvp.ipynb
│   └── 02_mri_amyloid_pet_bag.ipynb
├── results/
│   ├── figures/               # Aggregate figures for reporting
│   └── tables/                # Aggregate result tables for reporting
├── private/                   # Personal action checklist — ignored by Git
├── src/                       # Reusable functions (planned as the project matures)
├── LICENSE
└── README.md
```

## Quick start (local VS Code)

1. Download the required files from ADNI and place them in the directory structure above.
2. Open the `brain-age-gap-adni` folder in VS Code and select a Python kernel.
3. Run [`01_mri_brain_age_bag_mvp.ipynb`](notebooks/01_mri_brain_age_bag_mvp.ipynb) from top to bottom.
4. Then run [`02_mri_amyloid_pet_bag.ipynb`](notebooks/02_mri_amyloid_pet_bag.ipynb) for the PET extension.

The notebooks automatically locate the project root by finding `data/adni_all`; no hard-coded local path is required.

The notebooks deliberately use CSV output instead of Parquet so that `pyarrow` is not required.

## Result-file naming

Each execution uses an analysis ID and version folder, for example `results/tables/01_mri_bag_mvp_20260820_v1/`. Increase the version only when the analysis definition changes; use the same folder for a repeated run of the same definition. See [results/README.md](results/README.md) for the complete convention.

## Planned progression

| Phase | Goal | Status |
|---|---|---|
| MVP | PHC data audit, CN brain-age model, bias correction, held-out test, diagnosis ANCOVA | In progress |
| Extension 1 | Amyloid PET-aligned analysis | Planned |
| Extension 2 | CN-reference normative ROI Z-score profiles | Planned |
| Extension 3 | Sensitivity analyses for ROI sets, alignment window, eTIV handling, and model choice | Planned |
| Extension 4 | FDR-controlled cortical surface visualization after analysis definitions are fixed | Planned |
| Portfolio | Modular `src/`, polished figures/tables, reproducibility documentation | Planned |

## Interpretation and limitations

- This is a cross-sectional observational analysis; it cannot establish causality or individual ageing velocity.
- Dementia is retained as a clinical label and is not treated as pathologically confirmed Alzheimer's disease.
- Amyloid PET analysis is secondary and may be affected by PET-subset selection.
- ComBat harmonization is supplied by the ADSP PHC resource. Its use reduces multicenter technical variation, but the harmonization procedure remains a limitation to describe transparently.
- ROI-level results cannot be interpreted as voxel-level localisation.
- Internal ADNI test performance does not guarantee performance in a new hospital or population.

## Data acknowledgement

Data used in the preparation of this project were obtained from the Alzheimer's Disease Neuroimaging Initiative (ADNI) database. ADNI investigators contributed to the design and implementation of ADNI and/or provided data but did not participate in the analysis or writing of this project. For the complete acknowledgement and citation guidance, see [ADNI](https://adni.loni.usc.edu/).

## License

The analysis code is released under the repository's [MIT License](LICENSE). ADNI data are governed separately by the ADNI Data Use Agreement and must not be redistributed through this repository.
