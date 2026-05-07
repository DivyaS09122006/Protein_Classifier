# Protein Property Predictor — Enzyme vs Non-Enzyme Classifier

A binary classifier that predicts whether a human protein is an enzyme or non-enzyme using only its amino acid sequence. No structure, no lab data — pure sequence-based machine learning.

---

## What this project does

Given a protein sequence, the model predicts:
- **Enzyme (1)** — a protein that catalyses a biochemical reaction
- **Non-enzyme (0)** — a protein with a different function (here: transcription factors)

---

## Dataset

- **Source:** UniProt Swiss-Prot, reviewed human proteins (organism ID: 9606)
- **Enzymes:** 891 sequences
- **Non-enzymes:** 891 sequences (transcription-related, subsampled for balance)
- **Total:** 1782 proteins, perfectly balanced

---

## Features

Sequences were converted to numerical features using [iFeature](https://github.com/Superzchen/iFeature):

| Feature | Description | Size |
|---|---|---|
| AAC | Amino Acid Composition — frequency of each of the 20 amino acids | 20 features |
| DPC | Dipeptide Composition — frequency of every adjacent amino acid pair | 400 features |

---

## Models trained

- Logistic Regression
- SVM (RBF kernel)
- Random Forest (100 estimators)

---

## Results

### AAC features (20 features)

| Model | Accuracy | AUC-ROC |
|---|---|---|
| Logistic Regression | 74.5% | 0.827 |
| SVM (RBF) | 77.9% | 0.843 |
| Random Forest | 78.2% | 0.824 |

### DPC features (400 features)

| Model | Accuracy | AUC-ROC |
|---|---|---|
| Logistic Regression | 73.4% | 0.808 |
| SVM (RBF) | 78.2% | 0.853 |
| Random Forest | 78.7% | 0.831 |

**Key finding:** DPC (400 features) does not dramatically outperform AAC (20 features). The limiting factor is feature representation, not model choice — all three models plateau around 78% accuracy regardless of feature set.

---

## Biological interpretation

**Random Forest feature importance (AAC)** revealed the most discriminating amino acids:

| Amino Acid | Importance | Why |
|---|---|---|
| S (Serine) | 0.101 | High in non-enzymes — marks intrinsically disordered regions common in transcription factors |
| V (Valine) | 0.094 | High in enzymes — hydrophobic core packing in tightly folded proteins |
| I (Isoleucine) | 0.078 | High in enzymes — same as above |
| L (Leucine) | 0.065 | High in enzymes — hydrophobic core residue |
| W (Tryptophan) | 0.064 | High in enzymes — buried hydrophobic anchor at active sites |

**DPC heatmap** showed:
- **S→S** pairs strongly enriched in non-enzymes — serine repeat regions typical of disordered transcription factor domains
- **P-rich pairs** enriched in non-enzymes — proline-rich activation domains
- **L→L, V→K** pairs enriched in enzymes — hydrophobic packing near active sites

---

## Limitations

- AAC and DPC lose positional information — two proteins with the same amino acid composition but different sequences get identical features
- Classical ML models require fixed-size input vectors, forcing lossy summarisation of variable-length sequences
- Next step: protein language model embeddings (ESM2, ProtBERT) which read the full sequence and capture long-range dependencies

---

## Tech stack

- Python, pandas, numpy, matplotlib, seaborn
- scikit-learn (LogisticRegression, SVC, RandomForestClassifier)
- iFeature (feature extraction from FASTA)

---

## Project structure

```
Protein_classifier/
├── data/
│   ├── enzymes.fasta
│   └── non_enzymes.fasta
├── features/
│   ├── enzymes_AAC.csv
│   ├── non_enzymes_AAC.csv
│   ├── enzymes_DPC.csv
│   └── non_enzymes_DPC.csv
└── notebooks/
    └── protein_classifier.ipynb
```

---

## How to run

1. Extract features using iFeature:
```bash
python iFeature.py --file data/enzymes.fasta --type AAC --out features/enzymes_AAC.csv
python iFeature.py --file data/non_enzymes.fasta --type AAC --out features/non_enzymes_AAC.csv
python iFeature.py --file data/enzymes.fasta --type DPC --out features/enzymes_DPC.csv
python iFeature.py --file data/non_enzymes.fasta --type DPC --out features/non_enzymes_DPC.csv
```

2. Run the notebook `notebooks/protein_classifier.ipynb` top to bottom.

> Note: iFeature outputs TSV format despite the .csv extension. Always load with `sep='\t'`.
