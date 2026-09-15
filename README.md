# Code-for-FedSenseFormer-UQ-conformal-uncertainty-quantification

# FedSenseFormer-UQ

Analysis code for the manuscript:

**FedSenseFormer-UQ: Conformal Uncertainty Quantification and Horizon Limits in Personalized Federated Vital-Sign Forecasting from Wearable Physiological Sensors**

R. Augustian Isaac (corresponding author), A. Jaffar Sadiq Ali, S. Mercy Gnana Gandhi

*Submitted to BMC Medical Informatics and Decision Making.*

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USERNAME/REPO/blob/main/FedSenseFormer_UQ_Q1_Healthcare_Colab_MULTIHORIZON.ipynb)

---

## What this repository contains

A single self-contained Colab notebook that reproduces every result reported in the paper: data download, preprocessing, model training, uncertainty calibration, baseline comparison, statistical testing and figure generation.

`FedSenseFormer_UQ_Q1_Healthcare_Colab_MULTIHORIZON.ipynb`

## What the notebook does

Given a rolling 60-second history window of multimodal physiological observations for a patient, the model forecasts that patient's **heart rate (HR), respiratory rate (RR) and SpO₂** at a configurable horizon ahead.

The pipeline covers:

- **Preprocessing** — 125 Hz PPG and respiration waveforms are summarised into per-second features and merged with the 1 Hz monitor numerics to build a subject-wise multimodal feature table.
- **Residual target reformulation** — the model predicts the *change* in each vital (`y[t+h] − y[t]`) rather than its absolute value. Predictions are reconstructed to absolute clinical units using each window's last observed value before any metric is computed, so results remain directly comparable with the persistence baseline.
- **Personalized federated training** — subject-level clients, 12 communication rounds, 12 clients sampled per round, 2 local epochs per round.
- **Uncertainty quantification** — conformal prediction intervals plus Monte Carlo dropout (40 samples), evaluated for empirical coverage against the nominal level.
- **Explainability** — sensor-group occlusion to quantify each modality's contribution.
- **Baselines** — persistence, Ridge regression, ExtraTrees, and ablated variants of the proposed architecture.
- **Evaluation** — MAE, RMSE and R² per target and macro-averaged, with subject-block bootstrap confidence intervals (resampled by subject, not by window, since consecutive windows overlap heavily), Wilcoxon signed-rank and Friedman tests.

An automated driver runs the full **4 horizons × 5 seeds = 20 configurations** (horizons 30, 60, 120 and 300 seconds; seeds 42–46) in a single long-lived runtime. Completed runs are skipped on restart, so the driver is safe to re-run after a Colab disconnect.

## Data

This study uses the open-access **BIDMC PPG and Respiration Dataset** (v1.0.0) from PhysioNet:
https://doi.org/10.13026/C2208R

**No patient data is redistributed in this repository.** The notebook downloads the dataset directly from PhysioNet at runtime. The data are fully de-identified, contain no protected health information, require only a standard PhysioNet account, and are released under the Open Data Commons Attribution License v1.0. All 53 subjects are used in the full-scale configuration.

Dataset citation: Pimentel MAF, Johnson AEW, Charlton PH, Birrenkott D, Watkinson PJ, Tarassenko L, Clifton DA. Toward a robust estimation of respiratory rate from pulse oximeters. *IEEE Trans Biomed Eng.* 2017;64(8):1914–1923.

## How to run

1. Click the **Open in Colab** badge above (or upload the notebook to Colab manually).
2. Select a GPU runtime: *Runtime → Change runtime type → GPU*. The notebook falls back to CPU automatically, but the full 20-run driver is impractical without a GPU.
3. Confirm the configuration cell reads `QUICK_RUN = False` and `DELTA_TARGET = True` — this is the configuration used for all published results.
4. *Runtime → Run all*, then leave it running. The driver executes all 20 (horizon, seed) combinations and writes results to `/content/fedsenseformer_outputs_seed{N}/`.
5. Run the final aggregation cell to produce the pooled multi-horizon, multi-seed tables reported in the paper.

For a fast smoke test, set `QUICK_RUN = True` (16 subjects, 12 epochs, 3 federated rounds). These settings do **not** reproduce the published numbers.

## Requirements

No installation is needed in Colab. The notebook installs and imports:

| Component | Specification |
|---|---|
| Language | Python 3 |
| Deep learning | PyTorch (CUDA where available, CPU fallback) |
| Scientific stack | NumPy, pandas, SciPy, scikit-learn, Matplotlib, tqdm, requests |
| Classical baselines | scikit-learn: Ridge, ExtraTreesRegressor (via MultiOutputRegressor) |
| Statistical testing | scipy.stats: Wilcoxon signed-rank, chi-squared (Friedman) |
| Environment | Google Colaboratory, single long-lived runtime |

Full hyperparameter values and layer-by-layer architecture specifications are given in Additional file 1 of the manuscript and in the notebook's configuration cell.

## Reproducibility notes

- Seeds 42–46 are set for NumPy, PyTorch and Python's `random` at the start of each run; minor variation may still occur from non-deterministic GPU kernels.
- Calibration and test partitions are split by subject (20% validation, 25% test), never by window, to prevent leakage between overlapping windows from the same patient.

## Citation

If you use this code, please cite the manuscript (details to be updated on acceptance) and this archived release:

```
R. Augustian Isaac, A. Jaffar Sadiq Ali, S. Mercy Gnana Gandhi.
FedSenseFormer-UQ: analysis code. Zenodo. https://doi.org/10.5281/zenodo.XXXXXXX
```

## License

Released under the MIT License — see [LICENSE](LICENSE).

## Contact

Questions about the code or the paper: R. Augustian Isaac (corresponding author).
