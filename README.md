# HydroFlowNet-XF

## Risk-Aware Spatio-Temporal Learning Framework for Internal Water Flow Dynamics in Energy Infrastructure

This repository contains the reproducibility code accompanying the manuscript:

> **Risk-Aware Spatio-Temporal Learning Framework for Internal Water Flow Dynamics in Energy Infrastructure**

The repository provides the experimental pipeline used for dataset preparation, multimodal representation generation, HydroFlowNet-XF training, baseline comparison, ablation studies, robustness analysis, statistical testing, and computational profiling.

The notebooks are provided as executed research records. **The stored outputs and execution results are part of the reported experimental record and should not be overwritten by rerunning cells unless the experiments are intentionally reproduced from scratch.**

---

## Overview

HydroFlowNet-XF is evaluated using shallow-water-equation (SWE) simulations from **PDEBench**.

The reproducibility pipeline covers:

- PDEBench SWE dataset acquisition and integrity verification
- Selection of reference trajectories
- Water-height field extraction
- Numerical gradient-based derivation of velocity components
- Virtual drifter generation
- Eulerian sensor-grid construction
- Multimodal feature generation
- Temporal window construction
- Four-zone hazard-label construction
- HydroFlowNet-XF training
- Conventional and deep-learning baseline comparisons
- Trajectory-independent evaluation
- Component-wise ablation studies
- Robustness experiments
- Statistical significance testing
- Computational latency and efficiency profiling
- Generation of tables and figures used for analysis

All comparative experiments use the same processed samples, labels, splits, and evaluation metrics within each evaluation protocol.

---

## Repository Structure

```text
HydroFlowNet-XF/
│
├── HydroFlowNet_XF_Reproducible_Baselines_and_Validation.ipynb
│   └── Main reproducibility notebook
│
├── HydroFlowNet_XF_Computational_Latency_Benchmark.ipynb
│   └── Computational latency and efficiency evaluation
│
├── requirements.txt
│   └── Python package dependencies
│
├── README.md
│   └── Repository documentation
│
└── hydroflow_experiment_results/
    ├── figures/
    └── results/
```

The large PDEBench dataset is **not included in this repository**.

---

# Dataset

## PDEBench

The experiments use the **2D shallow-water-equation (SWE)** dataset distributed through PDEBench.

The notebook obtains the dataset from the official PDEBench dataset metadata and downloads the required file automatically.

### Dataset file

```text
2D_rdb_NA_NA.h5
```

The corresponding PDEBench dataset path is:

```text
2D/shallow-water/
```

The local path used by the reproducibility notebook is:

```text
hydroflow_pde_data/2D/shallow-water/2D_rdb_NA_NA.h5
```

### Dataset integrity

The notebook records and verifies the expected MD5 checksum before using the dataset:

```text
75d838c47aa410694bdc912ea7f22282
```

The downloaded file is accepted only when the calculated MD5 checksum matches the expected value.

The notebook also records the dataset provenance, including:

- PDE category
- filename
- PDEBench path
- source URL
- expected MD5
- verified MD5
- local dataset path
- integrity status

---

## Reference trajectories

The reproducibility experiments use the five reference trajectories:

```text
0000
0001
0002
0003
0004
```

Each trajectory contains:

```text
101 time steps
128 × 128 spatial grid
1 water-height channel
```

Thus, the stored trajectory representation has shape:

```text
(101, 128, 128, 1)
```

The notebook explicitly records the selected trajectory IDs and their dimensions.

---

# Preprocessing

The preprocessing pipeline converts the PDEBench water-height fields into the multimodal representation used by HydroFlowNet-XF.

## Water-height extraction

For each selected trajectory, the water-height field is extracted from the PDEBench HDF5 data.

Let the water-height field be denoted by:

```text
h(t, x, y)
```

where `t` denotes the temporal index and `(x,y)` denotes the spatial location.

---

## Numerical velocity representation

The velocity-related components are derived numerically from the water-height field using spatial gradients.

The notebook computes:

```text
u = ∂h/∂x
v = ∂h/∂y
```

using numerical gradients.

These components are subsequently used as part of the motion representation.

---

## Virtual drifters

The preprocessing pipeline generates:

```text
100 virtual drifters
```

within each spatial domain.

The drifters provide a Lagrangian-style representation that complements the Eulerian sensor representation.

---

## Eulerian sensor grid

An Eulerian sensor grid of:

```text
4 × 4
```

is constructed over the spatial domain.

The Eulerian representation contains three features derived from the flow field.

The resulting multimodal representation therefore combines:

1. Lagrangian virtual-drifter information
2. Eulerian grid information
3. Temporal history

---

## Gaussian sensor noise

Gaussian noise is incorporated using:

```text
noise standard deviation = 0.01
```

The noise level is controlled through the reproducibility configuration.

---

## Temporal history

The temporal window length is:

```text
K = 3
```

The model therefore uses the specified historical context when constructing temporal samples.

---

# Four-Zone Representation

The spatial domain is divided into:

```text
4 zones
```

The preprocessing pipeline constructs hazard labels from the zone-level flow representation.

The zone-based formulation allows the experiments to evaluate localized flow behavior rather than treating the entire spatial field as a single undifferentiated region.

The hazard-label construction uses the specified:

```text
5% zone threshold
```

for determining the corresponding hazard condition.

---

# Reproducibility Configuration

The reference experimental configuration uses the following parameters:

```text
SEED = 42
LATENT_DIM = 256
BATCH_SIZE = 16
EPOCHS = 10
LEARNING_RATE = 1e-4
TEMPERATURE = 0.5
FOCAL_ALPHA = 5
FOCAL_GAMMA = 3
TEMPORAL_REGULARIZATION_WEIGHT = 0.1
```

The notebook also fixes the principal preprocessing parameters, including:

```text
NUM_DRIFTERS = 100
SENSOR_GRID_SIZE = 4
NUM_ZONES = 4
HISTORY_K = 3
NOISE_STD = 0.01
```

A global random seed of `42` is configured for Python, NumPy, and PyTorch.

---

# Training

The main reproducibility notebook contains the HydroFlowNet-XF implementation and training pipeline.

The reference configuration uses:

| Parameter | Value |
|---|---:|
| Random seed | 42 |
| Latent dimension | 256 |
| Batch size | 16 |
| Epochs | 10 |
| Learning rate | 1 × 10⁻⁴ |
| Temperature | 0.5 |
| Focal loss α | 5 |
| Focal loss γ | 3 |
| Temporal regularization weight | 0.1 |

The notebook stores the experimental outputs under:

```text
hydroflow_experiment_results/
```

---

# Evaluation Protocols

Two evaluation settings are considered.

## Reference random-window evaluation

The reference evaluation uses an:

```text
80% training / 20% validation
```

random-window split.

The corresponding results are used for the reference comparison reported in the manuscript.

---

## Trajectory-independent evaluation

A separate trajectory-independent protocol is used to evaluate generalization across trajectories.

This prevents windows originating from the same trajectory from being distributed across the training and evaluation sets in a way that could introduce trajectory-level information leakage.

The trajectory-independent results are reported separately from the reference random-window results.

---

# Baseline Models

The evaluation pipeline includes the following models:

### Logistic Regression

A conventional linear classification baseline.

### Random Forest

A tree-based ensemble classifier used as a non-neural baseline.

### MLP

A multilayer perceptron baseline for nonlinear classification.

### LSTM

A recurrent neural-network baseline for temporal modeling.

### TCN

A temporal convolutional network baseline.

### HydroFlowNet-XF

The proposed multimodal spatio-temporal architecture.

The baselines are evaluated using the same processed samples and corresponding evaluation protocol as the proposed model.

---

# Evaluation Metrics

The following classification and probabilistic metrics are computed:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC
- PR-AUC
- Brier score

These metrics are used to evaluate both predictive performance and probabilistic behavior.

---

# Ablation Studies

The repository includes component-wise ablation experiments for HydroFlowNet-XF.

The ablation analysis evaluates the contribution of individual components by comparing the complete model against corresponding reduced configurations.

The same evaluation data and metrics are used for the paired comparisons.

The resulting ablation tables and figures are saved under:

```text
hydroflow_experiment_results/ablations/
```

---

# Statistical Testing

The repository includes paired statistical comparisons between the full HydroFlowNet-XF model and the evaluated ablations.

## McNemar test

McNemar's test is used for paired comparison of classification outcomes on common evaluation instances.

The analysis considers the discordant prediction counts between the full model and each ablation.

## Holm correction

Multiple pairwise comparisons are corrected using the **Holm procedure**.

The corrected results are reported together with the corresponding paired comparison statistics.

Statistical outputs are stored under:

```text
hydroflow_experiment_results/statistics/
```

---

# Robustness Evaluation

The reproducibility notebook also evaluates model behavior under controlled perturbations, including:

- Sensor noise
- Sensor masking
- Sensor bias

These experiments assess whether the learned representation remains effective when the input sensing conditions deviate from the nominal configuration.

Results are stored under:

```text
hydroflow_experiment_results/robustness/
```

---

# Computational Profiling

The second notebook,

```text
HydroFlowNet_XF_Computational_Latency_Benchmark.ipynb
```

contains the computational profiling experiments.

The benchmark evaluates computational characteristics of the models, including inference-related performance and latency measurements.

The profiling results are intended to complement the predictive-performance evaluation by quantifying computational requirements.

Results generated by the profiling workflow are stored separately from the main predictive evaluation outputs.

---

# Expected Outputs

Running the complete experimental pipeline produces outputs in:

```text
hydroflow_experiment_results/
```

including:

```text
figures/
results/
```

Typical outputs include:

- Dataset provenance records
- Selected trajectory manifests
- Training logs
- Evaluation tables
- Baseline comparison results
- Ablation results
- Robustness results
- Statistical-test results
- Generated figures
- Saved model artifacts
- Computational profiling results

---

# Reproducibility Notes

The notebooks distributed with this repository contain stored execution outputs from the reported experiments.

For this reason, users should distinguish between:

1. **Inspecting the stored notebook outputs**, which preserves the experimental record; and
2. **Rerunning the experiments**, which constitutes a new experimental execution.

Because neural-network training and stochastic data processing can produce different numerical results across executions and environments, rerunning the notebook is not expected to overwrite or modify the originally reported experimental record.

For exact reproduction, the following should be kept consistent:

- Dataset file
- Dataset checksum
- Selected trajectories
- Preprocessing parameters
- Random seed
- Model configuration
- Training configuration
- Evaluation split
- Evaluation metrics
- Software environment
- Hardware configuration where relevant

---

# Dataset Download Policy

The PDEBench `.h5` dataset is **not included in this repository** because of its large file size.

Instead, the notebook obtains the official dataset programmatically and verifies its integrity before processing.

Therefore, the repository contains the **code required to obtain and process the dataset**, rather than redistributing the multi-gigabyte dataset file.

The expected dataset filename is:

```text
2D_rdb_NA_NA.h5
```

with the corresponding verified MD5:

```text
75d838c47aa410694bdc912ea7f22282
```

---

# Environment

The notebooks use Python and the following core scientific and machine-learning libraries:

```text
numpy
pandas
scipy
scikit-learn
torch
h5py
matplotlib
seaborn
statsmodels
```

Additional packages used directly by the notebooks may include utilities for downloading data and displaying progress.


---

# Installation

Create a Python environment and install the dependencies listed in:

```text
requirements.txt
```

For example:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

On Windows:

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

---

# Running the Notebooks

Open the notebooks using Jupyter or Google Colab.

### Main reproducibility notebook

```text
HydroFlowNet_XF_Reproducible_Baselines_and_Validation.ipynb
```

This notebook covers:

```text
Dataset
→ preprocessing
→ feature construction
→ model training
→ baseline comparison
→ ablations
→ robustness
→ statistical analysis
→ figures and tables
```

### Computational benchmark

```text
HydroFlowNet_XF_Computational_Latency_Benchmark.ipynb
```

This notebook contains the computational latency and efficiency experiments.

---

# Manuscript Correspondence

The repository is intended to provide computational support for the experimental results described in the manuscript.

In particular, the reproducibility notebook provides the implementation and evaluation pipeline associated with:

- PDEBench SWE data processing
- Reference evaluation
- Trajectory-independent evaluation
- Baseline comparisons
- HydroFlowNet-XF evaluation
- Ablation studies
- Robustness experiments
- Statistical significance analysis

The computational benchmark notebook provides the profiling experiments associated with the computational-efficiency analysis.

---

# Important File-Name Consistency

The official PDEBench SWE file used by the reproducibility pipeline is:

```text
2D_rdb_NA_NA.h5
```

All manuscript, README, notebook, provenance, and repository references should use this filename consistently.

---


# License

The PDEBench dataset remains subject to the licensing and usage conditions of its original source. This repository does not redistribute the large PDEBench `.h5` dataset.

---

# Contact

For questions regarding the implementation or reproducibility experiments, please contact the corresponding author listed in the associated manuscript.
