# GDSC Drug Response Prediction (EDIT)

Predicting cancer drug response from gene expression data using machine learning.

## Overview

This project explores whether gene expression patterns in cancer cell lines can predict sensitivity to anti-cancer drugs. It uses the Genomics of Drug Sensitivity in Cancer (GDSC) dataset and a small set of machine learning models to build a precision-medicine-style analysis pipeline.

The project is designed as a portfolio piece for computational biology / bioinformatics PhD applications.

## Research question

Can cancer cell line gene expression be used to predict drug response?

## Dataset

This project uses the GDSC benchmark dataset accessed through Therapeutics Data Commons (TDC).

The target is drug response, typically represented as log normalized IC50.

## Repository structure

```text
gdsc-drug-response-project/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
├── notebooks/
├── src/
├── figures/
├── results/
├── models/
└── docs/
```

## Setup

Create a virtual environment and install the required packages:

```bash
pip install -r requirements.txt
```

If you are using Jupyter notebooks, also install the notebook kernel:

```bash
python -m ipykernel install --user --name gdsc-drug-response
```

## How to run

1. Open `notebooks/01_data_loading_and_qc.ipynb`
2. Run notebooks in order:
   - `01_data_loading_and_qc.ipynb`
   - `02_eda.ipynb`
   - `03_baseline_models.ipynb`
   - `04_model_comparison_and_interpretation.ipynb`
   - `05_final_figures.ipynb`

## Planned analyses

- Load and inspect the GDSC dataset
- Perform basic data quality checks
- Explore drug response distributions
- Train baseline ML models
- Compare model performance
- Interpret important genomic features

## Expected outputs

- Cleaned processed dataset
- Model evaluation metrics
- Plots and summary figures
- Feature importance / coefficient interpretation

## Notes

This project focuses on reproducible analysis and clear documentation rather than model complexity.

## License

MIT License

## Contact

Aarzu Miharia
