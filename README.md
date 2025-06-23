# HMM_KunitzDomain
# 🧬 Modeling the BPTI/Kunitz Domain: A Profile HMM Approach for Protein Domain Annotation

## 📄 Table of Contents

- [Overview](#-overview)
- [Repository Structure](#-repository-structure)
- [Objectives](#-objectives)
- [Installation](#-installation)
- [Methodology](#-methodology)
- [Results](#-results)
- [Contact](#-contact)

## ‍💻 Overview

This repository contains the code, datasets, and results for a project focused on the identification of the **BPTI/Kunitz domain** using a structure-informed **Profile Hidden Markov Model (HMM)**. The model was constructed from structurally aligned domain sequences and evaluated for domain annotation performance.

## 📁 Repository Structure

```text

Bioinformatics-Project-HMM-Kunitz-Domain/
│
├── data/                      # Input datasets
│   ├── pdb_files_msa/         # PDB files used for MSA
│   ├── preprocessed/          # Cleaned/filtered sequences
│   └── raw/                   # Original downloaded sequences
│
├── docs/                      # Project documentation
│   └── report/                # Final project report (PDF)
│
├── results/                   # Output data
│   ├── evaluation/            # Classification metrics and confusion matrices
│   ├── hmm_profile/           # Final HMM profile
│   └── msa/                   # Multiple sequence and structural alignments
│
├── scripts/                   # Custom Python scripts
│   └── performance.py         # Script to evaluate HMM performance
│
└── README.md                  # This file
```

## 📌 Objectives

- Construct a profile HMM from structurally aligned Kunitz domain sequences.
- Evaluate its performance on positive and negative datasets using classification metrics.

## 🛠 Installation

Make sure you have the following dependencies installed:

- `Python >= 3.13`
- `BLAST+ >= 2.16.0`
- `CD-HIT >= 4.8.1`
- `HMMER >= 3.4`
- `UCSF Chimera >= 1.20` (optional, for visualization)

## 🚀 Methodology

### 💿 Data preparation (provided in data/)

**1. Retrieve protein sequences from [UniProtKB/Swiss-Prot] (https://www.uniprot.org) with the following criteria:**
  • Taxonomy: _Homo sapiens_ (NCBI Taxonomy ID: 9606)
  • Reviewed: Yes
  • Pfam Domain: PF00014 (Kunitz-type protease inhibitor)

**2. Acquire the [UniProtKB/Swiss-Prot] (https://www.uniprot.org/help/downloads) database file.**

**3. Obtain [PDB structures] (https://www.rcsb.org) with:**
  • Pfam Annotation: PF00014
  • Resolution ≤ 3.5 Å
  • Sequence Length: 45–80 amino acids

**4. Use CD-HIT to cluster sequences based on sequence identity:**

This is the command-line tool to cluster sequences based on sequence identity:

`cd-hit -i pdb_kunitz.fasta -o pdb_kunitz.clst`

✅ -i pdb_kunitz.fasta
  • Input file: a FASTA file (pdb_kunitz.fasta) containing protein sequences for Kunitz domains extracted from the PDB.

✅ -o pdb_kunitz.clst
  • Output file prefix.
  • CD-HIT will generate:
  • pdb_kunitz.clst: a FASTA file with the representative sequences.
  • pdb_kunitz.clst.clstr: a text file showing the cluster composition (which sequences belong to which cluster).

### 📏 Multiple Structural Alignment

**1. Upload the IDs of the representatives of the clusters to [PDBeFold] (https://www.ebi.ac.uk/msd-srv/ssm/cgi-bin/ssmserver), and select:** 
  • Submission form/3D alignment: multiple
  • Source: List of PDB codes
  • Submit your query

### 🔦 Construction, Validation and Evaluation of the Profile Hidden Markov Model

**1. Build the HMM Profile**

Build a profile Hidden Markov Model (HMM) from a multiple sequence alignment using HMMER with the following command:

`hmmbuild pdb_kunitz_nr_clean.hmm pdb_kunitz_nr_clean.ali`

✅ hmmbuild
  • This is an HMMER tool that builds a profile HMM from a multiple sequence alignment in FASTA or Stockholm format.

✅ pdb_kunitz_nr_clean.hmm
  • This is the output file — the HMM profile that gets created.

✅ pdb_kunitz_nr_clean.ali
  • This is the input alignment file in FASTA format.

**2. Search for Kunitz Domains in positive and negative sequences against the profile HMM**

These two commands are running HMMER’s hmmsearch tool on  positive and negative datasets to identify sequences that match your Kunitz domain HMM profile.

`hmmsearch -Z 1000 --max --tblout neg_#.out pdb_kunitz_nr_clean.hmm neg_#.fasta` 
`hmmsearch -Z 1000 --max --tblout pos_#.out pdb_kunitz_nr_clean.hmm pos_#.fasta

✅ -Z 1000
  • Sets the database size used for E-value calculations to 1000. This standardizes E-values across different queries/datasets.

✅ --max
  • Turns off all HMM heuristics and performs a full exhaustive search.
 
✅ --tblout neg_#.out or --tblout pos_#.out 
  • Outputs the result in a concise, table format to neg_#.out or pos_#.out. Useful for automated parsing.

✅ pdb_kunitz_nr_clean.hmm
  • The profile HMM you’re using for the search (in this case, representing Kunitz domain).

✅ neg_#.fasta or pos_#.fasta
  • The FASTA file of negative sequences (not expected to have the Kunitz domain) or positive sequences (expected to have the Kunitz domain).

**3. Evaluate performance**

Run the performance.py script with a treshold sweep from 1e-1 to 1e-12 on both test sets to evaluate how performance metrics vary with different e-value cutoffs.

`for i in `seq 1 12`; do python3 performance.py set_1.class 1e-$i; done`

This runs the performance.py script using 1e-$i as the e-value threshold to classify predictions:
  • If e-value <= 1e-$i  → predict positive.
  • If e-value > 1e-$i  → predict negative.

The script then computes:
  • Confusion matrix
  • Q2 (accuracy)
  • MCC (Matthews Correlation Coefficient)
  • TPR (True Positive Rate / Sensitivity)
  • PPV (Positive Predictive Value / Precision)

### 📊 Results

The following graph illustrates the bahavior of the MACC value of the Set 1 (cyan) and Set 2 (fuchsia) for different e_values:

![Picture1](https://github.com/user-attachments/assets/684c0878-4654-4174-8b32-d77698c7a319)

Final remarks:
• Matthews Correlation Coefficient (MCC): Up to 0.99
• Optimal threshold: E-value of 1e-05-1e-06 showed the best balance between sensitivity and specificity.
• Structural conservation: Disulfide-bridging cysteines were highly conserved, supporting domain specificity.
• See the full report in docs/report/LB1ProjectPCV.pdf.

⸻


## 📧 Contact

For questions or feedback, please contact:

### Priscilla Castro-Vargas
Department of Pharmacy and Biotechnology
Alma Mater Studiorum – Università di Bologna
**Email:** priscilla.castro@studio.unibo.it
**ORCID:** 0000-0001-6333-0419

_This project was conducted as part of the Laboratory of Bioinformatics I course at the University of Bologna, 2025._

