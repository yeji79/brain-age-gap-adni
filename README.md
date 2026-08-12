# Structural MRI-based Brain Age Gap Analysis in the Alzheimer's Spectrum

> A leakage-controlled structural-MRI brain-age framework that tests whether bias-corrected Brain Age Gap (BAG) differs by clinical diagnosis and amyloid status in ADNI.

## Research question

Can a brain-age model trained **only in cognitively normal (CN) participants** identify structural MRI patterns that are older than expected for chronological age in MCI and dementia? As a secondary analysis, is BAG associated with amyloid positivity in the PET-available subset?

This is an observational association study. BAG is a model-derived deviation from the reference CN pattern; it is **not** a clinical diagnosis, a causal measure of neurodegeneration, or a direct measure of an individual's rate of ageing.

## Design safeguards

- **Baseline MRI only:** one earliest usable T1/FreeSurfer record per `RID` is retained. Clinical diagnosis is aligned to the closest assessment within a prespecified window.
- **Participant-level splitting:** `RID`, never rows, defines the train/validation/test split and all cross-validation folds.
- **CN-only reference model:** only CN records train and tune chronological-age prediction; MCI and dementia remain untouched until final inference.
- **Bias correction:** a linear correction is estimated on the CN validation set, then frozen before use in the CN test, MCI, dementia, and amyloid analyses.
- **MRI-only age features:** diagnostic, cognitive, amyloid, and tau variables are excluded from the age model. Sex is included as a non-imaging covariate; sensitivity analyses can omit it.
- **Feature-specific handling:** FreeSurfer volumes and cortical thickness are retained in their original units. Cortical thickness is never divided by intracranial volume (ICV). The downloaded `UCSFFSX7` table does not itself provide eTIV/ICV, so the core analysis does not claim ICV-normalised volumes. If an exact, compatible ICV table is added, residualisation must be performed inside each training fold only.
- **QC and harmonisation:** FreeSurfer QC, scanner field strength/site, processing version, and protocol effects are reported and handled as prespecified exclusions/covariates where sample size permits.

## Required local data

The notebook looks in `data/adni_all/` by default. The raw ADNI directory must never be committed.

```text
data/adni_all/
├── Imaging/
│   ├── UCSFFSX7_22Jan2026.csv
│   ├── UCBERKELEY_AMY_6MM_22Jan2026.csv      # optional secondary analysis
│   └── MRIMETA_22Jan2026.csv                 # scanner metadata, optional but recommended
├── Assessments/
│   └── DXSUM_22Jan2026.csv
├── Subject_Characteristics/
│   └── PTDEMOG_22Jan2026.csv
└── Quick_Start/
    └── DATADIC_21Jan2026.csv
```

The repository currently contains an `ADNI_data/` download for local work. Before publishing, either rename it to `data/adni_all/` or configure `ADNI_ROOT` in the notebook, and ensure it remains ignored by Git.

## Outputs

The notebook writes non-identifying aggregate outputs only:

- `results/tables/model_performance.csv`
- `results/tables/ancova_diagnosis.csv`
- `results/tables/ancova_amyloid.csv` (when PET data are available)
- `results/tables/permutation_importance.csv`
- figures in `results/figures/`

Participant-level processed data are written locally under `data/processed/` and are ignored.

## Important interpretation rules

- Report MAE and R² on the untouched CN test set, alongside a `DummyRegressor` baseline.
- Report the correlation of corrected BAG with chronological age; it should be near zero in the held-out CN test set.
- Interpret diagnostic-group differences with an age-, sex-, and education-adjusted ANCOVA/regression model. Pairwise contrasts are exploratory unless multiplicity correction is added.
- Feature importance indicates contribution to **age prediction**, not a causal Alzheimer’s disease biomarker. MRI ROI importance is reported at region level, not as voxelwise localisation.
- The amyloid analysis is restricted to the PET-available subset and can be selection-biased; it is secondary.

## Reproducibility

Run [`notebooks/01_brain_age_gap_adni.ipynb`](notebooks/01_brain_age_gap_adni.ipynb) in Google Colab or a Python 3.10+ environment. It installs its own lightweight dependencies in Colab and records the random seed. Read the markdown cells before changing any analysis threshold.

## Data acknowledgement

Data used in this project were obtained from the Alzheimer’s Disease Neuroimaging Initiative (ADNI) database. ADNI investigators contributed to the design and implementation of ADNI and/or provided data but did not participate in analysis or writing of this project. See https://adni.loni.usc.edu/ for complete acknowledgement and citation guidance.
