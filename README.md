# HMM_KunitzDomain  
### Profile Hidden Markov Model for Kunitz-Type Protein Domain Identification

---

## Overview
This repository contains a bioinformatics workflow for identifying **Kunitz-type (BPTI) protein domains** using a **Profile Hidden Markov Model (HMM)**.

The project focuses on:
- organizing raw and processed protein sequence datasets,
- constructing a domain-specific HMM from curated sequences,
- evaluating the model on positive and negative datasets.

The repository is designed for **educational and research use**, with an emphasis on clarity, reproducibility, and good data-management practices.

---

## Repository Structure

```text
HMM_KunitzDomain/
├── Data/
│   ├── Raw/            # Original, source protein sequence datasets
│   └── Processed/      # Curated datasets, alignments, and HMM artifacts
│
├── Script/             # Scripts for analysis and evaluation
├── Docs/               # Documentation and notes
├── .gitignore
├── LICENSE
└── README.md
