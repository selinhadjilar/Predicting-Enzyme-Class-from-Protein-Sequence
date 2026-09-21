# Predicting Enzyme Class from Protein Sequence
As the volume of protein sequences grows significantly, accurate enzyme functional annotation remains a challenge in bioinformatics. This study investigates the classification of enzyme sequences while addressing class imbalance (115:1 in this dataset). Two models, Logistic Regression and XGBoost, were developed using embeddings from the ESM-2 protein language model. 

## Summary
Protein sequences that are accessible in the field of bioinformatics are expanding continuously. Examples include the reference
protein knowledge base UniProtKB, which increased from 180 million available sequences to over 225 million between February 2020 and February 2022. This project investigates efficient strategies for enzyme classification and proposes solutions to key challenges that are
encountered in this field, such as data imbalance. It classifies protein sequences into 7 classes (6 EC enzyme classes + non-enzyme) using embeddings from the ESM-2 protein language model, combined with a soft-voting ensemble of Logistic
Regression and XGBoost, plus a confidence-tiering system to explore unreliable predictions.

## Key Results

| Metric | Score |
|---|---|
| Test accuracy | 83.3% |
| Macro F1 | 0.49 |
| Balanced accuracy | 0.58 |
| Matthews Correlation Coefficient | 0.54 |
- **High-confidence predictions** (41.7% of test set): **97.4% accuracy**
- **Low-confidence predictions** (22.9% of test set): 55.7% accuracy — correctly flagged as needing further review
- Confirmed no overfitting: cross-validation and held-out test performance matched within ~0.3%


## Approach

1. **Data**: 39,678 protein sequences across 7 classes (Not Enzyme, Oxidoreductase,
   Transferase, Hydrolase, Lyase, Isomerase, Ligase), with extreme class imbalance
   (32,324 non-enzyme sequences vs. 282 Ligases)
2. **Embeddings**: ESM-2 (`esm2_t6_8M_UR50D`, 6 layers, 8M parameters) via mean-pooling
   over final-layer representations
3. **Class imbalance handling**: class-balanced sample weighting, macro-averaged
   evaluation metrics (F1, balanced accuracy, MCC), and a soft-voting ensemble tuned to
   balance majority-class accuracy against minority-class recall
4. **Models**: Logistic Regression (favoured minority-class recall) and XGBoost (favoured
   majority-class accuracy), combined via weighted soft voting (final weights LR=2, XGB=1)
5. **Confidence calibration**: predictions bucketed into High/Medium/Low confidence tiers
   based on maximum predicted probability, validated against actual accuracy per tier
6. **Interpretability**: heatmap analysis linking biochemical properties (hydrophobicity,
   aromaticity, instability index, secondary structure content) to each enzyme class's
   classification signal

## Findings

- Enzyme vs. non-enzyme classification was the primary source of error (680 misclassified
  sequences), more difficult than distinguishing between enzyme classes (likely due to
  overlapping structural cores in the ESM-2 embedding space)
- Minority classes (Lyase, Isomerase, Ligase) remained hard to classify reliably even with
  balanced weighting; confidence calibration was notably weaker for these classes,
  flagging where the model's own certainty shouldn't be fully trusted
- Distinct biochemical signatures were recovered per class (e.g., aromaticity for
  Oxidoreductases binding aromatic cofactors, turn-fraction for Lyases' loop-mediated
  active sites) — suggesting the embedding space captures genuine structural signal

## Tech Stack

Python 3.10 (Google Colab, T4 GPU) · PyTorch · ESM-2 (`fair-esm`) · scikit-learn ·
XGBoost · BioPython · pandas / NumPy · matplotlib / seaborn

## Repo Contents

- `enzyme_classification.ipynb` — full pipeline: data loading, sequence cleaning, ESM-2
  embedding extraction, model training, ensembling, evaluation, confidence calibration
- `report.pdf` — full write-up with methodology, results tables, and discussion

## Limitations & Future Work

- Used the smallest ESM-2 variant (8M params) for memory constraints; larger variants
  may improve minority-class performance
- SMOTE was explored but excluded due to compute limitations
- Sequences over 1022 residues were truncated, potentially losing C-terminal information
- Future work: incorporate conserved catalytic motifs or predicted active-site residues,
  which prior literature suggests are informative for this task

## Full Report

See [`report.pdf`](./report.pdf) for complete methodology, cross-validation results,
per-class metrics, and calibration analysis.


