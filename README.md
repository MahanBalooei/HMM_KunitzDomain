# 🧬 Structure-Based HMM Profiling for Kunitz-Type Protease Inhibitor Domains

![Python](https://img.shields.io/badge/python-3.10+-blue)
![HMMER](https://img.shields.io/badge/HMMER-3.3.2-green)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Platform](https://img.shields.io/badge/platform-linux%20%7C%20macOS-lightgrey)

> Building structure-informed Profile HMMs for Kunitz/BPTI-type protease inhibitor domain detection, comparing sequence-based (MUSCLE) vs. structure-based (PDBeFold) approaches using 2-fold cross-validation on Swiss-Prot data.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Objectives](#objectives)
- [Installation](#installation)
- [Methodology](#methodology)
- [Results](#results)
- [Contact](#contact)

---

## 📌 Overview

This repository contains the code, datasets, and results for a project focused on developing structure-informed Profile Hidden Markov Models (HMMs) for the detection and annotation of Kunitz/BPTI-type protease inhibitor domains. Two HMMs were built:

- **Sequence-based HMM** — from MUSCLE multiple sequence alignment
- **Structure-based HMM** — from PDBeFold structural alignment

The models were evaluated using curated datasets from Swiss-Prot, with performance assessed via 2-fold cross-validation. The structure-based approach demonstrated superiority in capturing remote homologs (peak MCC: **0.997** vs. **0.991**).

---

## 📁 Repository Structure

```
HMM_KunitzDomain/
│
├── Data/                      # Input datasets
│   ├── Raw/                   # Original downloaded sequences (Swiss-Prot, PDB structures)
│   └── Processed/             # Curated datasets, alignments, HMM models, evaluation files
│
├── Docs/                      # Project documentation
│   └── report/                # Final project report (PDF)
│
├── Script/                    # Custom scripts for analysis
│   └── performance.py         # HMM performance evaluation (MCC, TPR, PPV)
│
├── environment.yml            # Conda environment specification
├── requirements.txt           # Python dependencies
├── .gitignore                 # Files to ignore in Git
├── LICENSE                    # MIT License
└── README.md                  # This file
```

---

## 🎯 Objectives

- Construct profile HMMs from sequence-based and structure-based alignments of Kunitz domains.
- Evaluate model performance on positive and negative datasets using classification metrics, addressing annotation incompleteness in databases like Pfam.
- Demonstrate the advantage of incorporating structural information for improved sensitivity to remote homologs.

---

## ⚙️ Installation

### Option 1 — Conda (recommended)

```bash
git clone https://github.com/MahanBalooei/HMM_KunitzDomain.git
cd HMM_KunitzDomain
conda env create -f environment.yml
conda activate hmm-kunitz
```

### Option 2 — pip

```bash
git clone https://github.com/MahanBalooei/HMM_KunitzDomain.git
cd HMM_KunitzDomain
pip install -r requirements.txt
```

> **External tools required:** HMMER ≥ 3.3.2, MUSCLE v5, MMseqs2, CD-HIT ≥ 4.8.1, UCSF ChimeraX (for structural visualization)

---

## 🔬 Methodology

### Data Preparation

**Positive set** — Retrieved from UniProtKB/Swiss-Prot (release 2025_03):
- Annotated with Pfam PF00014, InterPro IPR036880, or Prosite (PS00280, PS50279)
- 368 high-confidence Kunitz proteins after filtering, split into two folds (`pos_1`, `pos_2`)

**Negative set** — All Swiss-Prot proteins lacking Kunitz annotations

**PDB structures** — Pfam PF00014, resolution ≤ 3.5 Å, length 45–80 aa, clustered at 90% identity with CD-HIT

**Contamination removal** — BLASTp to filter sequences ≥95% identical to structural seeds

### Multiple Structural Alignment (MStA)
Structures aligned using PDBeFold (SSM), exported in FASTA format. All structures showed the canonical Kunitz fold: two β-strands, central α-helix, inhibitory loop, three disulfide bridges.

### Multiple Sequence Alignment (MSA)
Generated using MUSCLE v5 on the same representative sequences for fair comparison.

### HMM Construction

```bash
hmmbuild sequence_based.hmm muscle_alignment.ali
hmmbuild structure_based.hmm pdbe_alignment.ali
```

### HMM Search

```bash
hmmsearch -Z 1000 --max --tblout pos_1.out structure_based.hmm pos_1.fasta
hmmsearch -Z 1000 --max --tblout neg_1.out structure_based.hmm neg_1.fasta
```

### Performance Evaluation

```bash
for i in `seq 1 12`; do python Script/performance.py set_1.class 1e-$i; done
```

Computes confusion matrix, Q2 (accuracy), MCC, TPR (sensitivity), PPV (precision) across E-value thresholds (10⁻¹ to 10⁻¹²).

---

## 📊 Results

| Model | Peak MCC | Optimal E-value |
|---|---|---|
| Structure-based HMM | **0.997** | 10⁻⁵ – 10⁻⁶ |
| Sequence-based HMM | 0.991 | 10⁻⁵ – 10⁻⁶ |

Key findings:
- **E-value separation**: Positives scored 10⁻³⁶ to 10⁻⁶⁰; negatives scored 1–10
- **Cross-validation**: Near-perfect — 0 false positives, 0–2 false negatives per fold
- **Misclassification**: Apparent false positives were genuine Kunitz domains missed by Pfam
- **Domain-level scoring** outperformed full-sequence scoring for multidomain proteins

See the full report in `Docs/report/Mahan_Balooei_LAB1_Report.pdf` for figures (MCC curves, confusion matrices, E-value distributions).

---

## 📧 Contact

**Mahan Balooei**
- 🏛️ Department of Pharmacy and Biotechnology, Alma Mater Studiorum – Università di Bologna
- 📧 [mahan.balooei@studio.unibo.it](mailto:mahan.balooei@studio.unibo.it)
- 🔬 ORCID: [0009-0006-5358-0784](https://orcid.org/0009-0006-5358-0784)

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
