# antibiotic-resistance-gene-predictor
ML model predicting antibiotic resistance genes from protein sequences — XGBoost + SHAP + ESM-2
# 🧬 Antibiotic Resistance Gene Predictor

> Machine learning pipeline that predicts antibiotic resistance genes
> from protein sequences using amino acid k-mer features, XGBoost, and SHAP interpretability.
> Trained on CARD database · Validated against UniProt bacterial proteins.

[![Python](https://img.shields.io/badge/Python-3.10-blue)]()
[![XGBoost](https://img.shields.io/badge/XGBoost-2.0-red)]()
[![SHAP](https://img.shields.io/badge/SHAP-Explainable-green)]()
[![CARD](https://img.shields.io/badge/Data-CARD_Database-orange)]()
[![Status](https://img.shields.io/badge/Status-Active_Development-brightgreen)]()

---

## 🎯 Why This Matters

Antibiotic resistance is a **WHO priority global health threat** —
projected to cause 10 million deaths/year by 2050. Identifying
resistance genes in metagenomic samples is the critical first step
to tracking resistance spread across hospitals, environments, and patients.

Existing tools (RGI, ResFinder) rely on exact database matches.
This model learns **statistical patterns from sequence composition**,
with the potential to flag novel resistance genes not yet catalogued
in any database.

---

## 📊 Results

### Model Comparison (Protein k-mer Features)

| Model | AUC-ROC | F1 | Precision | Recall | Train Time |
|-------|---------|-----|-----------|--------|------------|
| Random Forest | 0.984 | 0.900 | 0.999 | 0.819 | 12.7s |
| **XGBoost** ⭐ | **0.986** | **0.927** | **0.988** | **0.873** | **11.8s** |
| SVM | 0.984 | 0.940 | 0.971 | 0.910 | 167.6s |

**XGBoost selected** as primary model — best AUC + F1 score at 14× faster than SVM.

### ROC + Precision-Recall Curves
![ROC Curves](results/roc_curves_protein.png)

### SHAP Feature Importance
> Which amino acid dipeptide patterns most strongly drive resistance predictions?

![SHAP Bar](results/shap_bar.png)
![SHAP Summary](results/shap_summary.png)

### Biological Interpretation of Top Features

| Dipeptide | SHAP Score | Biological Significance |
|-----------|-----------|------------------------|
| WE (Trp-Glu) | 0.686 | Tryptophan in active sites of beta-lactamases |
| GW (Gly-Trp) | 0.305 | Conserved in enzyme substrate-binding pockets |
| DN (Asp-Asn) | 0.276 | Common in catalytic triads of hydrolase enzymes |
| TF (Thr-Phe) | 0.231 | Found in serine beta-lactamase active sites |
| length_norm | 0.431 | Resistance genes are systematically longer — consistent with horizontal gene transfer origins |

**Key finding:** Tryptophan-containing dipeptides (WE, GW, WP) dominate
predictions — consistent with conserved active site residues in
beta-lactamase enzymes, the most clinically important resistance mechanism.

---

## 🔬 Method

```
CARD Database (6,052 resistance proteins)     ← positive class
        +
UniProt Bacterial Proteins (12,000 sampled)   ← negative class
        ↓
Amino acid 2-mer frequencies (400 features)
+ normalized sequence length (1 feature)
= 401 features per sequence
        ↓
SMOTE balancing → 80/20 stratified train/test split
        ↓
Random Forest vs XGBoost vs SVM comparison
        ↓
SHAP interpretability on best model (XGBoost)
```

### Why Protein k-mers?

DNA-level k-mers were tested first but caused data leakage — the model
learned sequence alphabet (DNA vs protein) rather than resistance biology.
Protein-level dipeptide frequencies on a consistent amino acid alphabet
produced honest, biologically meaningful results.

---

## 🗺️ Roadmap

### Completed
- [x] Protein k-mer feature engineering (401 features)
- [x] 3-model comparison (Random Forest, XGBoost, SVM)
- [x] SMOTE class imbalance handling
- [x] SHAP interpretability + biological interpretation
- [x] ROC + Precision-Recall curve analysis

### In Progress
- [ ] **ESM-2 protein embeddings** — deep learning features vs k-mers comparison
- [ ] **ESM-2 vs k-mer experiment** — which representation learns more biology?

### Planned (Publishability upgrades)
- [ ] **Beta-lactamase focused model** — go deep on clinically critical resistance class
- [ ] **Confidence calibration** — isotonic regression on XGBoost probabilities
- [ ] **Cross-database validation** — train on CARD, test on MEGARes (true generalization test)
- [ ] **Real SRA metagenomic reads** — Prodigal ORF calling + model prediction on raw reads
- [ ] **CLI prediction tool** — `python predict.py --input sequences.fasta`

---

## 📁 Repository Structure

```
antibiotic-resistance-gene-predictor/
│
├── notebooks/
│   └── arg_predictor_pipeline.ipynb   ← full Kaggle pipeline
│
├── src/
│   ├── features.py                    ← k-mer extraction functions
│   ├── train.py                       ← model training + SMOTE
│   └── evaluate.py                    ← metrics, SHAP, plots
│
├── results/
│   ├── roc_curves_protein.png         ← ROC + PR curves
│   ├── shap_bar.png                   ← top 20 dipeptides
│   └── shap_summary.png              ← SHAP beeswarm plot
│
├── predict.py                         ← CLI tool (coming soon)
├── requirements.txt
└── README.md
```

---

## 🚀 Quick Start

```bash
git clone https://github.com/YOURUSERNAME/antibiotic-resistance-gene-predictor
cd antibiotic-resistance-gene-predictor
pip install -r requirements.txt

# Predict resistance genes from your FASTA file
python predict.py --input your_sequences.fasta
```

### Reproduce Results

```bash
# 1. Download CARD + UniProt data
python src/download_data.py

# 2. Extract protein k-mer features
python src/features.py

# 3. Train and compare models
python src/train.py

# 4. Generate SHAP analysis
python src/evaluate.py
```

---

## 📦 Data Sources

| Database | Role | Sequences | Link |
|----------|------|-----------|------|
| CARD | Resistance genes (positive class) | 6,052 proteins | [card.mcmaster.ca](https://card.mcmaster.ca) |
| UniProt Swiss-Prot | Normal bacterial proteins (negative) | 12,000 sampled | [uniprot.org](https://uniprot.org) |
| MEGARes *(planned)* | Cross-database validation | ~9,000 proteins | [megares.meglab.org](https://megares.meglab.org) |
| NCBI SRA *(planned)* | Real metagenomic reads test | TBD | [ncbi.nlm.nih.gov/sra](https://www.ncbi.nlm.nih.gov/sra) |

---

## ⚠️ Limitations & Honest Notes

- Model trained and tested on CARD-derived sequences — not raw metagenomic reads
- Negative class (UniProt) was subsampled from 320,000 to 12,000 — results may shift with different sampling
- Cross-database generalization not yet validated (planned with MEGARes)
- ESM-2 embeddings pending — may significantly improve recall

---

## 🛠️ Tech Stack

`Python 3.10` `XGBoost` `scikit-learn` `BioPython` `SHAP`
`imbalanced-learn` `ESM-2 (Meta)` `pandas` `numpy` `matplotlib`

---

## 📖 References

- McArthur et al. (2023) — CARD: Comprehensive Antibiotic Resistance Database
- Lin et al. (2023) — ESM-2: Evolutionary Scale Modeling for proteins
- Lundberg & Lee (2017) — SHAP: A unified approach to interpreting model predictions

---

*Built as a bioinformatics + ML portfolio project.
Contributions and feedback welcome via Issues.*
