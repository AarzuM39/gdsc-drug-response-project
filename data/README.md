# Data directory

Large raw and processed datasets are not stored in this GitHub repository.

## Obtain the source data

Download the required GDSC2 drug-response and processed RNA-seq expression datasets from the [Cell Model Passports download portal](https://cellmodelpassports.sanger.ac.uk/downloads).

This project expects the following local structure:

```text
data/
├── raw/
│   ├── drug.csv
│   ├── models.csv
│   └── expr.csv
└── processed/
    └── cisplatin_rnaseq_modelready_clean.csv
```

## Reproduce processed files

After placing the downloaded source files in `data/raw/`, run:

```text
notebooks/01_data_loading_and_qc.ipynb
```

This notebook filters drug-response data to cisplatin, selects corresponding cancer cell lines from the RNA-seq data, reshapes expression data into a wide feature matrix, merges response and expression data, and writes project-specific processed output files.

## Version note

Cell Model Passports files and file names may change over time. The notebooks are designed around the dataset fields used in this project, including a cell-model identifier, drug name, cisplatin `LN_IC50`, gene symbol, and RSEM TPM expression value.

### Data versioning

The analysis used the GDSC2 and processed RNA-seq resources available from Cell Model Passports at the time of download. Because upstream data releases may change, readers reproducing the workflow at a later date may obtain slightly different record counts or results. The notebooks document the filtering, merging, and preprocessing steps used in this analysis.
