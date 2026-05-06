# Hito 1 — Baseline Model & Temporal Validation

**Team:** Feligma  
**Course:** IIT414W  
**Submission:** May 6, 2026

Hito 1 implements a calibrated baseline model to predict F1 `is_top10` outcomes using pre-race features. This submission packages the framing document, executable baseline notebook, AI log, and reproducible runbook for the F1 Race Strategy Advisor capstone project.

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

**hito1_baseline.ipynb** — Baseline Model & Temporal Validation

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
Capstone/Hito 1/
├── framing.md                      # Team decision sheet 
├── hito1_baseline.ipynb            # Baseline model notebook 
├── PROMPTS.md                      # AI usage documentation 
├── README.md                       # How to reproduce 
└── f1_strategy_race_level.csv      # F1 race data 
```
