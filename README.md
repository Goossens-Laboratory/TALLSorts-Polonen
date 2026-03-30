## Intro:
This repository contains a model trained - and to be used with - TALLSorts for subtyping of TALL based on RNA-seq. Briefly: It was trained on the publicly available *_counts file from Polonen 2025 after normalization in DESeq2. It expects normalized counts as input with rows as columns as ensembl gene names and rows as patient IDs:

<img width="331" height="163" alt="image" src="https://github.com/user-attachments/assets/99109d8c-6fca-472e-911d-58c527036c3f" />


## EXAMPLE RUN
[picks up after TALLSorts installation]
````
conda activate tallsortsenv
TALLSorts -m test -s normalized_counts.csv --d outdir --mp Polonen_cleaned_normalized_counts.pkl.gz
````


## T-ALL Subtype Classifier Training Pipeline:
Here I outline how this model was trained.

### Input data
- Polonen RNA-seq counts-derived matrix used as starting point for model preparation
- Clinical metadata (clin_df) containing Patient_ID and Subtype
- One-hot encoded sample sheet generated from clin_df for TALLSorts training

### Gene ID processing
- Strip Ensembl version suffixes
- ENSG00000000003.14 -> ENSG00000000003
- Collapse duplicate Ensembl IDs by mean expression

### Annotation-based filtering
- Annotate Ensembl IDs from GTF
- Retain protein-coding genes only
- Remove:
  - mitochondrial genes
  - Y chromosome genes
  - XIST
  - Result: approximately 16k genes retained

### Sample filtering
- Remove samples without subtype labels
- Keep only labeled samples present in both:
  - counts matrix
  - sample sheet

### Low-expression filtering
- Keep genes with count > 5 in >= 4 samples
- Chosen to preserve signal from rare subtypes with very small sample counts

### Sample sheet construction
- Convert clin_df subtype labels to one-hot encoded columns
- Remove samples with missing subtype
- Save filtered sample sheet for TALLSorts training

### Matrix formatting
- Transpose cleaned counts matrix to samples x genes
- Rows = sample IDs
- Columns = Ensembl gene IDs
- Output: Polonen_cleaned_normalized_counts.csv

### Model training
- Train TALLSorts on:
  - Cleaned, transposed normalized-count matrix
  - filtered one-hot sample sheet
  - hierarchy.csv
  - training_params.csv
  - TALLSorts then performs its own internal normalization / scaling during model fitting
