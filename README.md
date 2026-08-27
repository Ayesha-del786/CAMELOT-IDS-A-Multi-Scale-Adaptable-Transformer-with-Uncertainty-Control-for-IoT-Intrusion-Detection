# CAMELOT-IDS

**CAMELOT-IDS: A Multi-Scale Adaptable Transformer with Uncertainty Control for IoT Intrusion Detection**

Research code and reproducibility artifacts for CAMELOT-IDS, an IoT intrusion-detection pipeline combining semantic local-global Transformer encoding, hierarchical supervision, imbalance-aware learning, uncertainty quantification, and stream monitoring.

## Overview

The executed v4 pipeline operates on the CICIoT2023 processed-feature data and uses:

- 46 sanitized numerical input features
- 34 fine-grained classes
- 8 coarse attack families
- Binary benign/malicious prediction
- Five semantic feature groups: rates, flags/counts, protocols, statistics, and other features
- Local-global Transformer encoding
- Hierarchical fine/coarse/binary supervision
- LDAM-DRW for long-tail learning
- Temperature scaling
- Deterministic Mondrian RAPS prediction sets
- ADWIN + conformal-martingale stream monitoring
- Optional TENT + pseudo-label head adaptation
- Resume-safe preprocessing caches and training checkpoints
- Publication-table and publication-figure generation

## Archived Main-Run Results

The executed v4 notebook reports the following held-out test results:

| Metric | Result |
|---|---:|
| Fine accuracy | 99.20% |
| Fine balanced accuracy | 76.56% |
| Fine macro-F1 | 76.76% |
| Coarse accuracy | 99.39% |
| Coarse balanced accuracy | 75.72% |
| Coarse macro-F1 | 78.89% |
| Malicious-class F1 | 99.79% |
| Archived RAPS coverage at alpha=0.10 | 99.95% |
| Archived RAPS mean set size at alpha=0.10 | 1.60 |
| GPU throughput | 18,141.6 samples/s |
| CPU throughput | 1,062.3 samples/s |

These aggregate results should be interpreted together with the class-level outputs. Several rare fine-grained attacks remain substantially harder than the majority classes.

The archived RAPS results are empirical diagnostics because the original archived pipeline reused the labeled calibration pool for both probability calibration and RAPS threshold estimation. The manuscript separately describes a disjoint confirmatory split-conformal analysis.

## Repository Layout

```text
CAMELOT-IDS/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
│
├── notebooks/
│   └── CAMELOT_IDS_v4_executed.ipynb
│
├── results/
│   ├── main_run/
│   │   ├── config.json
│   │   ├── split_mode.json
│   │   ├── epoch_history.csv
│   │   ├── test_metrics.json
│   │   ├── per_class_metrics_sorted_by_f1.json
│   │   ├── calibration_summary.json
│   │   ├── raps_calibration.json
│   │   ├── raps_results.json
│   │   ├── risk_coverage_table_raps.csv
│   │   ├── stream_results.json
│   │   ├── baseline_results.json
│   │   ├── final_summary.json
│   │   └── figures/
│   │
│   └── publication/
│       ├── publication_summary.md
│       ├── json/
│       ├── tables/
│       └── figures/
│
└── scripts/
```

Large preprocessing caches, temporary resume markers, raw datasets, and unchecked binary exports should not be committed to ordinary Git history.

## Dataset

The experiments use **CICIoT2023**.

Official dataset page:

https://www.unb.ca/cic/datasets/iotdataset-2023.html

The dataset is **not included in this repository**. Download it from the official source and configure the notebook's dataset path locally.

The archived primary run used a 100-file processed-feature subset. The notebook's default output directory is:

```text
./results_camelot_ids_v2_pc
```

## Installation

A CUDA-capable GPU is strongly recommended for model training. The archived manuscript reports profiling with PyTorch 2.1 and CUDA 12.1 on an RTX 3060; the code can also run on CPU, but training will be substantially slower.

Create an isolated environment and install the dependencies:

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Then install:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

For Jupyter:

```bash
jupyter lab
```

Open:

```text
notebooks/CAMELOT_IDS_v4_executed.ipynb
```

## Data Path

Do not publish your local absolute dataset path.

Before running the public notebook, set the dataset root to your local CICIoT2023 directory, for example:

```python
DATA_ROOT = r"/path/to/CICIoT2023"
```

The private workstation path used during development should be removed from the public notebook copy.

## Main Reproducibility Settings

The v4 implementation is cache- and checkpoint-aware. Important archived defaults include:

```text
SEED = 42
RESUME = True
CACHE_PREPROCESSED = True
CACHE_LOGITS = True
CACHE_STREAM_INPUTS = True
SAVE_LAST_EVERY_EPOCH = True
MAX_FILES = 100
BATCH_SIZE = 1024
EPOCHS = 70
USE_AMP = True
RUN_BASELINES = False
STREAM_BATCH = 50000
STREAM_MICRO_BATCH = 16384
```

The model configuration includes:

```text
D_MODEL = 256
NHEAD = 8
LOCAL_LAYERS = 2
GLOBAL_LAYERS = 6
FF_DIM = 1024
DROPOUT = 0.10
```

RAPS defaults:

```text
ALPHAS = (0.01, 0.05, 0.10)
RAPS_KREG = 3
RAPS_LAMBDA = 0.01
RAPS_MONDRIAN = pred_coarse
```

## Important Output Files

The main notebook writes the following primary evidence files:

```text
config.json
split_mode.json
epoch_history.csv
test_metrics.json
per_class_metrics_sorted_by_f1.json
calibration_summary.json
raps_calibration.json
raps_results.json
risk_coverage_table_raps.csv
stream_results.json
final_summary.json
confusion_matrix_coarse.png
stream_accuracy.png
stream_signal.png
stream_martingale.png
```

The publication extension additionally generates publication-ready tables, figures, JSON summaries, and model-export artifacts under:

```text
results_camelot_ids_v2_pc/publication_package/
```

## Checkpoints and Large Artifacts

Training creates:

```text
results_camelot_ids_v2_pc/checkpoints/best_model.pt
results_camelot_ids_v2_pc/checkpoints/last_checkpoint.pt
```

The publication extension also creates a selected publication checkpoint.

Do not commit large model files to ordinary Git without first checking their sizes. Use Git LFS, a GitHub Release, or an archival service such as Zenodo when appropriate.

The following directories are intentionally excluded from normal Git tracking by `.gitignore`:

```text
results_camelot_ids_v2_pc/cache/
results_camelot_ids_v2_pc/stages/
results_camelot_ids_v2_pc/checkpoints/
```

## Baseline and Fast Diagnostic Runs

The archived main v4 configuration has:

```text
RUN_BASELINES = False
```

Accordingly, baseline summaries from the primary run must not be presented as newly executed baseline evidence unless the corresponding baseline experiments, logs, and predictions are actually reproduced.

The `fast_extra_publication` block is a capped short-run diagnostic workflow and should not be mixed with the primary manuscript results.

## Export Status

Model export should be treated separately from scientific test results. Only publish an ONNX or TorchScript artifact as a validated deployment model when its corresponding validation step succeeds.

The archived notebook includes export-repair logic and a static-batch ONNX fallback, but deployment artifacts should be accompanied by their export/validation status rather than assumed to be valid.

## Reproducibility Notes

For a release-quality archive, preserve:

1. The executed notebook with outputs.
2. Exact result JSON/CSV files.
3. Publication figures and tables.
4. The exact split/provenance manifest used for the reported run.
5. The selected model checkpoint, distributed through an appropriate large-file mechanism.
6. Preprocessing metadata required for inference.
7. An exact environment lock or `conda env export` / `pip freeze` file from the machine used for the final release.

`requirements.txt` in this repository records the direct Python dependencies used by the notebook. It is not a substitute for an exact environment lock file because the notebook itself does not record exact versions for every imported package.

## Authors

- Ayesha Qureshi
- Imran Sarwar Bajwa
- Nadeem Sarwar
- Monia Hamdi
- Ali Samad

## Citation

Citation metadata will be updated after the article and archival repository receive their final persistent identifiers.

```text
A. Qureshi, I. S. Bajwa, N. Sarwar, M. Hamdi, and A. Samad,
"CAMELOT-IDS: A Multi-Scale Adaptable Transformer with Uncertainty Control
for IoT Intrusion Detection."
```

## License

This repository currently follows the manuscript's stated **Creative Commons Attribution 4.0 International (CC BY 4.0)** licensing declaration. See `LICENSE`.

## Research-use Notice

This repository is a research artifact. High aggregate accuracy does not imply uniformly reliable detection across all 34 fine-grained classes. Review the class-level metrics, uncertainty outputs, split/provenance documentation, and limitations before drawing operational conclusions.
