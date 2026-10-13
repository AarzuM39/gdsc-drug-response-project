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

## Results

### Test-set performance

| Model | Train \(R^2\) | Test \(R^2\) | Test RMSE | Test MAE |
|---|---:|---:|---:|---:|
| Ridge, \(\alpha = 100\) | 1.000 | 0.305 | 1.569 | 1.231 |
| Ridge, \(\alpha = 10,000\) | 0.871 | 0.352 | 1.515 | 1.199 |
| Random Forest 3 | 0.763 | 0.279 | 1.599 | 1.267 |

The strongly regularized Ridge model achieved the best held-out performance among the tested models. Although the less strongly regularized Ridge model fit the training set almost perfectly, its lower test performance indicates substantial overfitting. The selected Ridge model accepted a lower training fit in exchange for improved performance on unseen cell lines.

![Test-set model-performance comparison](figures/model_test_r2_comparison.png)

**Figure 2.** Held-out test-set \(R^2\) values for the evaluated models. Higher values indicate better out-of-sample predictive performance.

### Prediction and residual diagnostics

![Predicted versus true cisplatin response](figures/predicted_vs_true_ridge_rf.png)

**Figure 3.** Predicted versus observed cisplatin `LN_IC50` values in the held-out test set. The diagonal reference line represents perfect prediction.

The spread of points around the diagonal in the ridge test plot (left) indicates that the model captures some variation in cisplatin response but has meaningful prediction error for individual cell lines. This pattern is consistent with moderate out-of-sample performance, rather than near-perfect performance.

![Ridge test residual distribution](figures/residual_distribution_ridge_rf.png)

**Figure 4.** Distribution of Ridge and Random Forest 3 test-set residuals, defined as predicted `LN_IC50` minus observed `LN_IC50`.

![Ridge residuals versus true cisplatin response](figures/residuals_vs_true_ridge_rf.png)

**Figure 5.** Ridge and Random Forest 3 test-set residuals plotted against observed cisplatin `LN_IC50`. The horizontal reference line represents zero prediction error.

### Feature-ranking analysis

Ridge coefficients and Random Forest feature importances were examined to identify genes most strongly associated with model predictions. These rankings differed across model types, which is expected because Ridge and Random Forest quantify feature relevance differently.

Ridge coefficients represent standardized linear associations between expression and predicted `LN_IC50`, conditional on the other features in the model. Random Forest feature importance measures how much a feature contributed to prediction across tree splits. Neither ranking should be interpreted as evidence that a gene causes cisplatin sensitivity or resistance.

![Top Ridge Coefficients](figures/ridge_a10000_top_coefficients.png)

**Figure 6.** The top 15 Ridge coefficients, plotted against the genes associated with each coefficient. The genes are sorted highest to lowest.

![RF3 Feature Importances](figures/rf3_top_feature_importances.png)

**Figure 7.** The top 15 Random Forest 3 feature importances, plotted against the associated genes. The genes are sorted highest to lowest.

![Ridge Correlation Heatmap](figures/top_gene_correlation_heatmap.png)

**Figure 8.** A heatmap of the top Ridge genes based on coefficients.

## Discussion

The strongly regularized Ridge model generalized best to the held-out test set among the models evaluated. This suggests that, for this dataset, predictive signal may be represented more effectively by many small and regularized linear gene-expression effects than by the nonlinear patterns captured by the tested Random Forest configurations.

The improvement associated with stronger Ridge regularization is particularly relevant because the dataset contains many more gene-expression predictors than cell lines. In this setting, an unconstrained or weakly constrained model can fit training-specific noise, producing highly optimistic training performance that does not necessarily transfer to new observations.

The results still indicate only moderate predictive performance. Baseline gene expression alone does not provide a sufficiently precise cisplatin-response predictor for clinical use in this analysis. The models and feature rankings should therefore be treated as exploratory tools for hypothesis generation rather than validated biomarkers or causal evidence.

## Limitations

- The modeling dataset included 743 cell lines but 36,438 gene-expression predictors, creating a high-dimensional problem with a substantial risk of overfitting.
- Results were evaluated using one 80/20 train-test split; performance may vary under alternative splits or repeated cross-validation.
- Only Ridge regression and a limited set of Random Forest configurations were evaluated.
- The analysis used baseline RNA-seq expression alone and did not incorporate mutation, copy-number, methylation, tissue-of-origin, or other molecular features.
- Cancer cell lines do not fully reproduce patient tumors, including tumor microenvironment effects, pharmacokinetics, and clinical treatment conditions.
- Feature rankings represent model-specific predictive associations, not causal mechanisms or clinically validated biomarkers.
- The findings were not validated in an independent external dataset.

## Repository Structure

```text
.
├── notebooks/
│   ├── 01_data_loading_and_qc.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_baseline_models.ipynb
│   ├── 04_model_comparison_and_interpretation.ipynb
│   └── 05_final_figures.ipynb
├── figures/
│   ├── cisplatin_response_distribution.png
│   ├── model_test_r2_comparison.png
│   ├── predicted_vs_true_ridge_rf.png
│   ├── residual_distribution_ridge_rf.png
│   ├── residuals_vs_true_ridge_rf.png
│   ├── rf3_top_feature_importances.png
│   ├── ridge_a10000_top_coefficients.png
│   └── top_gene_correlation_heatmap.png
├── src/
│   ├── __init__.py
│   ├── data_utils.py
│   ├── evaluation.py
│   ├── modeling.py
│   └── preprocessing.py
├── data/
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

The notebook order is:

| Notebook | Purpose |
|---|---|
| `01_data_loading_and_qc.ipynb` | Loads, filters, reshapes, and merges drug-response and expression data |
| `02_eda.ipynb` | Examines data structure, missingness, and cisplatin-response distribution |
| `03_baseline_models.ipynb` | Trains baseline Ridge and Random Forest models |
| `04_model_comparison_and_interpretation.ipynb` | Compares models and examines coefficients or feature importances |
| `05_final_figures.ipynb` | Produces README-ready figures |

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
