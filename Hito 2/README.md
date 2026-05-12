# Hito 2 — Dual-Target Model & Error Analysis

**Team:** Feligma  
**Course:** IIT414W  
**Submission:** May 13, 2026

Hito 2 extends the baseline model to predict F1 race outcomes across two targets: the original `is_top10` from Hito 1 and an expansion target `is_top3`. This dual-target approach reveals strategy trade-offs invisible in binary classification alone. This submission packages calibrated models on both targets, structured error analysis segmented by strategy and circuit type, what-if comparisons, leakage audit, risk mitigations, and reproducible runbook for the F1 Race Strategy Advisor capstone project.

### Dual-Target Design

- **Primary Target (`is_top10`):** Binary classification for top-10 finishes. Calibrated via Platt scaling on 2022 data. Performance: Brier 0.1364, ROC-AUC 0.8792 (docent reference: Brier 0.1320, ROC-AUC 0.8920).
- **Expansion Target (`is_top3`):** Binary classification for podium finishes. Calibrated via Platt scaling on 2022 data. Performance: Brier 0.0768, ROC-AUC 0.9202 (exceeds null baseline by ~41.4%).

Both targets use **identical feature sets and locked temporal split** (train 2019–2021, calibration 2022, test 2023–2024) to enable interpretable side-by-side comparison.

---

## Dataset Requirements

This project requires the F1 race-level dataset:

**File:** `f1_strategy_race_level.csv` 

The notebook loads this file from `../Hito 2/f1_strategy_race_level.csv`. If you receive a path error:
1. Verify the CSV exists in `Capstone/Hito 2/`
2. Update `DATA_PATH` in the config cell of `hito2_modeling.ipynb` if your directory structure differs

---

## Prerequisites

Before running this project, ensure you have:

- **Python 3.8 or higher** installed on your system
- **pip** (Python package manager)
- **Git** (for version control)

## Installation

### 1. Create a Virtual Environment

It's recommended to use a Python virtual environment to isolate project dependencies.

**On Windows (PowerShell or CMD):**

```bash
python -m venv venv
venv\Scripts\activate
```

**On macOS/Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

After activation, your terminal should show `(venv)` at the beginning of the prompt.

### 2. Install Dependencies

Install all required packages from `requirements.txt`:

```bash
pip install -r requirements.txt
```

This will install:

- `jupyterlab` — Interactive Jupyter notebook environment
- `numpy`, `pandas` — Data manipulation and analysis
- `matplotlib`, `seaborn` — Data visualization and calibration curves
- `scikit-learn` — Machine learning models & metrics (Brier, ROC-AUC, calibration)
- `scipy` — Statistical functions
- `shap` — Feature importance & SHAP values for model interpretability
- `ipykernel` — IPython kernel for Jupyter

---

## Running the Notebook

### Open the Notebook in VS Code:

**hito2_modeling.ipynb** — Baseline Model & Temporal Validation

### Select the Virtual Environment Kernel

When you first open a notebook or click **"Run All"**, VS Code will prompt you to select a kernel:

1. Click **"Select Kernel"** (if prompted)
2. Choose **"Python (venv)"** — the virtual environment you created earlier
3. If not listed, click **"+ Select Another Kernel"** → **"Python Environments"** → Select the venv path

Once selected, all cells will execute using Python from your virtual environment.

### Execute Cells

- **Run All:** Click the **"Run All"** button at the top of the notebook to execute all cells sequentially
- **Run Single Cell:** Click the **▶ Run** button next to a specific cell (code or markdown) to execute only that cell
- **Keyboard Shortcut:** Press **Ctrl + Alt + Enter** (VS Code default) to run a cell

---

## Project Structure

```
Capstone/Hito 2/
├── hito2_modeling.ipynb             # Reproducible training & evaluation notebook
├── baseline_comparison.md           # Baseline comparison table on both targets
├── error_analysis.md                # Structured error analysis 
├── whatif_comparison.md             # Strategy comparisons where expansion target reveals
│                                    # recommendations that is_top10 alone cannot produce
├── leakage_audit.md                 # Leakage & confounding checklist results
├── mitigations.md                   # Risks and mitigations for final report & deployment
├── PROMPTS.md                       # AI usage documentation 
├── README.md                        # How to reproduce & project overview
├── requirements.txt                 # Python dependencies
└── f1_strategy_race_level.csv       # F1 race data 
```
