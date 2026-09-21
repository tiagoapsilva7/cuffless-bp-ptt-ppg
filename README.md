# Cuffless blood pressure prediction in the Pulse Transit Time PPG dataset

Cuffless systolic and diastolic blood pressure estimation from photoplethysmography
(PPG) morphology, evaluated on all six PPG channels of the
[Pulse Transit Time PPG dataset](https://physionet.org/content/pulse-transit-time-ppg/1.1.0/)
(PhysioNet, v1.1.0, 22 subjects).

Two complementary pipelines are provided:

| Notebook | Setting | Input to the model | Target |
|:--|:--|:--|:--|
| [`01_calibration_free.ipynb`](notebooks/01_calibration_free.ipynb) | Calibration-free | PPG morphology and demographics only | Cuff BP of the same half recording |
| [`02_calibration_based.ipynb`](notebooks/02_calibration_based.ipynb) | Calibration-based | PPG morphology, demographics, one cuff calibration value per subject, and the PPG change since calibration | Cuff BP at the end of the recording |

Both are validated with Leave-One-Subject-Out cross-validation, and both run over
the distal and proximal measurement sites at infrared, red and green wavelengths.

## What the pipelines do

1. **Signal conditioning.** Detrending and a 4th order zero-phase Butterworth
   bandpass between 0.5 Hz and 10 Hz, followed by z-score normalisation.
2. **Beat detection and quality gating.** Systolic peaks are detected with a
   refractory distance of 0.35 s. Every beat is scored with NeuroKit2
   template matching, and beats below 0.95 contribute no morphology features.
3. **Fiducial points.** Foot, systolic peak, dicrotic notch and diastolic peak.
   The notch and the diastolic peak are selected as a pair, scored against the
   timing of the previous accepted beat, with a relaxed fallback search when the
   main search returns nothing.
4. **Features.** 21 morphological descriptors per beat (amplitudes, relative
   timings, amplitude ratios, first and second derivative extrema and their
   timings, systolic upstroke times at 25%, 50% and 75% of the beat amplitude),
   reduced to their median per recording half, plus mean heart rate, heart rate
   variability and four demographic variables.
5. **Ensemble feature selection.** Pearson correlation, recursive feature
   elimination, ReliefF and mRMR are combined by Borda count, inside the training
   fold only.
6. **Model selection.** Random Forest, XGBoost, SVM (RBF) and Gaussian Process,
   with a joint search over the number of retained features (5, 10, 15, 20, 25)
   and the hyperparameter grid, by 3-fold inner cross-validation.
7. **Reporting.** MAE, bias and error SD per model, Shapiro-Wilk normality tests,
   pairwise paired t-test or Wilcoxon signed-rank comparisons, Bland-Altman plots,
   and CSV export of every prediction.

## Repository layout

```
.
|- notebooks/
|  |- 01_calibration_free.ipynb     calibration-free pipeline, step by step
|  \- 02_calibration_based.ipynb    calibration-based pipeline, step by step
|- data/
|  \- README.md                     where to download and place the dataset
|- results/
|  |- calibration_free_metrics.csv
|  |- calibration_based_metrics.csv
|  |- calibration_free_run_log.txt
|  \- calibration_based_run_log.txt
|- requirements.txt
\- README.md
```

## Getting started

```bash
git clone https://github.com/<user>/cuffless-bp-ptt-ppg.git
cd cuffless-bp-ptt-ppg

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scriptsctivate

pip install -r requirements.txt
```

### Where to put the data

The dataset is not included in this repository. Download it from PhysioNet and
copy its `csv/` folder into `data/`, so that the tree reads:

```
data/csv/subjects_info.csv
data/csv/s1_sit.csv
data/csv/s2_sit.csv
...
```

Full instructions, including the column meanings and how to convert the WFDB
records if you downloaded those instead, are in [`data/README.md`](data/README.md).

### Run

```bash
jupyter lab
```

Open either notebook and run the cells in order. Both notebooks resolve their
paths relative to `notebooks/`, so no path editing is needed as long as the data
sits in `data/csv/`.

A full run evaluates 6 channels x 2 targets x 4 models x 22 Leave-One-Subject-Out
folds, each with an inner grid search, so expect several hours on a normal laptop.
To try a single channel first, cut down the `wavelengths` dictionary
(notebook 01) or `signals_config` (notebook 02) in step 2.

### Output

| Path | Content |
|:--|:--|
| `results/all_model_predictions.csv` | Calibration-free predictions, one row per half recording |
| `results/calibration_full_results.csv` | Calibration-based predictions, one row per subject |
| `results/figures/BlandAltman_*.png` | Bland-Altman plot of the best systolic model per site |

## Results

Lowest MAE per measurement site and target, over 22 subjects.

### Calibration-free

| Site | Target | Best model | MAE (mmHg) | ME (mmHg) | SD (mmHg) |
|:--|:--|:--|--:|--:|--:|
| Distal IR | Systolic | Gaussian Process | 9.96 | -1.48 | 13.36 |
| Distal IR | Diastolic | SVM (RBF) | 6.25 | -0.01 | 7.75 |
| Distal Red | Systolic | SVM (RBF) | 8.98 | -1.25 | 11.37 |
| Distal Red | Diastolic | SVM (RBF) | 7.30 | 0.42 | 8.88 |
| Distal Green | Systolic | Gaussian Process | 10.32 | -0.87 | 13.86 |
| Distal Green | Diastolic | SVM (RBF) | 5.88 | -0.19 | 6.95 |
| Proximal IR | Systolic | Gaussian Process | 10.24 | -2.01 | 14.15 |
| Proximal IR | Diastolic | SVM (RBF) | 6.21 | -0.05 | 8.05 |
| Proximal Red | Systolic | Random Forest | 10.10 | -0.47 | 14.00 |
| Proximal Red | Diastolic | Gaussian Process | 6.60 | 0.68 | 8.14 |
| Proximal Green | Systolic | SVM (RBF) | 10.51 | -1.77 | 14.88 |
| Proximal Green | Diastolic | SVM (RBF) | 6.84 | -0.27 | 8.65 |

### Calibration-based

| Site | Target | Best model | MAE (mmHg) | ME (mmHg) | SD (mmHg) |
|:--|:--|:--|--:|--:|--:|
| Distal IR | Systolic | XGBoost | 4.27 | -1.64 | 5.45 |
| Distal IR | Diastolic | SVM (RBF) | 5.72 | 0.33 | 6.77 |
| Distal Red | Systolic | XGBoost | 4.72 | -0.52 | 6.21 |
| Distal Red | Diastolic | Random Forest | 7.16 | 0.21 | 8.24 |
| Distal Green | Systolic | Random Forest | 5.09 | 0.97 | 6.99 |
| Distal Green | Diastolic | SVM (RBF) | 5.14 | 0.39 | 6.74 |
| Proximal IR | Systolic | Random Forest | 6.03 | 0.10 | 8.10 |
| Proximal IR | Diastolic | XGBoost | 4.58 | 0.25 | 6.11 |
| Proximal Red | Systolic | Random Forest | 6.03 | -0.73 | 7.89 |
| Proximal Red | Diastolic | XGBoost | 4.44 | -0.38 | 5.25 |
| Proximal Green | Systolic | Gaussian Process | 7.15 | -0.59 | 10.75 |
| Proximal Green | Diastolic | XGBoost | 4.88 | -0.65 | 6.45 |

Per model MAE tables and the full console logs, including feature rankings and
the pairwise statistical tests, are in [`results/`](results/) and in step 11 of
each notebook.

## Dataset citation

Mehrgardt, P., Khushi, M., Poon, S., & Withana, A. (2022). *Pulse Transit Time
PPG Dataset* (version 1.1.0). PhysioNet. https://doi.org/10.13026/jpan-6n92

Goldberger, A., Amaral, L., Glass, L., Hausdorff, J., Ivanov, P. C., Mark, R.,
Mietus, J. E., Moody, G. B., Peng, C. K., & Stanley, H. E. (2000). PhysioBank,
PhysioToolkit, and PhysioNet: Components of a new research resource for complex
physiologic signals. *Circulation*, 101(23), e215-e220.

## License

Code released under the MIT License, see [LICENSE](LICENSE). The dataset keeps
its own PhysioNet license and is not redistributed here.
