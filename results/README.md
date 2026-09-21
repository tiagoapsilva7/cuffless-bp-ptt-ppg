# Results

Artefacts of the runs stored in the notebooks.

| File | Content |
|:--|:--|
| `calibration_free_metrics.csv` | MAE, mean error and error SD for every site, target and model, calibration-free pipeline |
| `calibration_based_metrics.csv` | Same table for the calibration-based pipeline |
| `calibration_free_run_log.txt` | Full console log of the calibration-free run, including the feature rankings and the statistical tests |
| `calibration_based_run_log.txt` | Full console log of the calibration-based run |

Running the notebooks regenerates these files and additionally writes:

* `all_model_predictions.csv`, per half recording, calibration-free
* `calibration_full_results.csv`, per subject, calibration-based
* `figures/BlandAltman_*.png`, one Bland-Altman plot per site for systolic BP

Those three are not committed, since they carry the cuff reference values of the
dataset. They appear here as soon as you run the notebooks locally.
