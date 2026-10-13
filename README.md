# Predicting Cisplatin Response from Gene Expression Using GDSC2

## Overview

This project evaluates whether baseline RNA-seq gene-expression profiles can predict cisplatin response across cancer cell lines in the Genomics of Drug Sensitivity in Cancer 2 (GDSC2) dataset. Cisplatin response was modeled as the supplied continuous `LN_IC50` value using both Ridge regression and Random Forest regression.

The final analysis used 743 cell lines and 36,438 gene-expression predictors. The best-performing model was strongly regularized Ridge regression, which achieved a held-out test-set \(R^2\) of 0.339.

## Research Question

Can baseline RNA-seq gene-expression profiles predict the supplied cisplatin `LN_IC50` response measure in held-out GDSC2 cancer cell lines?

## Background

Due to alterations in cancer genomes, different cancer cells can vary greatly in their response to anticancer drugs. This means that, in many cases, these alterations can be used as effective molecular biomarkers for response to drugs. Understanding these genomic markers and the resulting variation in response to different anticancer drugs can help create more informed hypotheses about treatment response and even potentially identify which patients might benefit most from a certain treatment. The Genomics of Drug Sensitivity in Cancer (GDSC) database is a large public resource for information on drug sensitivity in cancer cells and molecular markers of drug response. This database can help facilitate the discovery of new therapeutic biomarkers for different anticancer drugs. [1]

Cisplatin is a widely used chemotherapy agent. It has been shown to be effective against various types of cancers such as bladder, lung, ovarian, and testicular cancers. It seems to work due to its ability to damage DNA and cause apoptosis in cancer cells by crosslinking with purine bases. [2] However, like many anticancer drugs, the development of resistance remains a major challenge in the use of cisplatin. Using a database such as the GDSC database could help predict biomarkers that might affect how a patient responds to treatment with cisplatin.

## Objective

This project investigates the research question above by comparing regularized linear regression (Ridge) with Random Forest regression. The analysis emphasizes out-of-sample performance, overfitting control, residual diagnostics, and feature-ranking analysis. Drug response is represented by the dataset's supplied log-transformed IC50 measure, which was used directly as the value predicted by the tested regression models. It is important to note that results of this project are only intended as exploratory predictive associations, not clinical recommendations or evidence of causal mechanisms.

## Data

This project uses GDSC2 cancer cell-line data obtained through the [Cell Model Passports download portal](https://cellmodelpassports.sanger.ac.uk/downloads). Two data sources were integrated:

| Data source | Information used | Role in analysis |
|---|---|---|
| Drug-response data | Drug name, Sanger cell-model ID, and `LN_IC50` | Regression target |
| RNA-seq expression data | Sanger cell-model ID, gene symbol, and RSEM TPM expression value | Predictor variables |

The drug-response dataset was filtered to cisplatin. RNA-seq records were filtered to the corresponding cell lines and reshaped from long format into a wide cell-line-by-gene matrix, where rows represented cell lines and columns represented genes. Drug-response and expression data were then matched using the Sanger cell-model ID.

The final modeling dataset contained **743 cell lines** and **36,438 baseline gene-expression predictors**. The outcome variable was the dataset-provided cisplatin `LN_IC50` value, which was used directly as a continuous regression target; no additional log transformation was applied. Cell-line identifiers and drug-name columns were excluded from model training.

### Cisplatin response distribution

The cisplatin response values were unimodal and approximately bell-shaped, with most observations concentrated around intermediate `LN_IC50` values and fewer observations at the low and high extremes. Therefore, the dataset covers a broad response range, although the response values are not evenly represented across that range.

![Distribution of cisplatin response values](figures/cisplatin_response_distribution.png)

**Figure 1.** Distribution of the supplied cisplatin `LN_IC50` values in the 743-cell-line modeling dataset.

## Methods

Data were split into 80% training and 20% held-out test sets using `random_state=42`, producing 594 training cell lines and 149 test cell lines. Missing expression values were mean-imputed using training data only, then the learned imputation values were applied to the test set.

The predictor matrix contained many more gene-expression features than observations. This high-dimensional setting creates a substantial risk of overfitting, which motivated the use of regularized regression and held-out test-set evaluation.

### Models

Two model families were compared:

| Model | Preprocessing | Purpose |
|---|---|---|
| Ridge regression | Mean imputation and standardization | Regularized linear prediction using an L2 penalty |
| Random Forest regression | Mean imputation; no standardization | Nonlinear tree-based ensemble prediction |

Two Ridge models were evaluated using different regularization strengths. The final Ridge regularization value was selected using five-fold cross-validation on the training set. The selected model used \(\alpha = 10{,}000\).

The final Random Forest configuration used 300 trees, `max_depth=7`, `max_features=200`, `min_samples_leaf=5`, and `random_state=42`.

### Evaluation

Models were evaluated on the held-out test set using:

- \(R^2\), which compares the model with predicting the mean response.
- Root mean squared error (RMSE), which gives greater weight to larger prediction errors.
- Mean absolute error (MAE), which measures the average magnitude of the prediction error.

Predicted-vs-observed and residual plots were also examined to assess model behavior beyond aggregate metrics.

## Key Results

- Ridge regression with alpha=10,000 performed best on the test set, with R2 = 0.339, RMSE = 1.530, and MAE = 1.223.
- Stronger Ridge regularization improved generalization relative to Ridge alpha=100, which showed near-perfect training fit but weaker test performance.
- Random Forest 3 (RF3) had lower test performance than the best Ridge model (R2 = 0.279, RMSE = 1.599, MAE = 1.267).
- Ridge coefficient rankings and Random Forest feature-importance rankings differed, suggesting that the models emphasized different gene-expression signals.

Test Set Performance Comparison
![Test-set performance comparison](figures/model_test_r2_comparison.png)

Predicted vs. True Cisplatin Response
![Predicted versus true cisplatin response](figures/predicted_vs_true_ridge_rf.png)

## Interpretation

The strongly regularized Ridge model generalized best to the test set, which suggests that a linear model using many small gene-expression effects is more effective for this dataset than the tested Random Forest configurations. However, the overall performance was still moderate, so these results should be treated as exploratory rather than as a clinical prediction tool. Other models could also be tested to determine their effectiveness.

## How to Run

Create and activate a virtual environment, then install the dependencies:

```bash
python -m venv .venv
source .venv/bin/activate        # macOS/Linux
.venv\Scripts\activate           # Windows
pip install -r requirements.txt
```

Install the Jupyter kernel if needed:

```bash
python -m ipykernel install --user --name gdsc-drug-response
```

Run the notebooks in order:

1. `notebooks/01_data_loading_and_qc.ipynb`
2. `notebooks/02_eda.ipynb`
3. `notebooks/03_baseline_models.ipynb`
4. `notebooks/04_model_comparison_and_interpretation.ipynb`
5. `notebooks/05_final_figures.ipynb`

## References

[1] Yang, W., Soares, J., Greninger, P., et al. (2013). Genomics of Drug Sensitivity in Cancer (GDSC): a resource for therapeutic biomarker discovery in cancer cells. *Nucleic Acids Research*, 41(Database issue), D955–D961. [DOI](https://doi.org/10.1093/nar/gks1111)

[2] Dasari, S., & Tchounwou, P. B. (2014). Cisplatin in cancer therapy: molecular mechanisms of action. *European Journal of Pharmacology*, 740, 364–378. [DOI](https://doi.org/10.1016/j.ejphar.2014.07.025)
