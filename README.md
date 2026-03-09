#  Structure-Based HMM Profiling for Kunitz-Type Protease Inhibitor Domains
##  Table of Contents

- Overview
- Repository Structure
- Objectives
- Installation
- Methodology
- Results
- Contact

###  Overview
This repository contains the code, datasets, and results for a project focused on developing structure-informed Profile Hidden Markov Models (HMMs) for the detection and annotation of Kunitz/BPTI-type protease inhibitor domains. Two HMMs were built: one based on sequence alignment (MUSCLE) and one on structural alignment (PDBeFold). The models were evaluated using curated datasets from Swiss-Prot, with performance assessed via 2-fold cross-validation, highlighting the superiority of the structure-based approach in capturing remote homologs.

###  Repository Structure
```text
HMM_KunitzDomain/
│
├── Data/                      # Input datasets
│   ├── Raw/                   # Original downloaded sequences (e.g., Swiss-Prot, PDB structures)
│   └── Processed/             # Curated datasets, alignments, HMM models, and evaluation files
│
├── Docs/                      # Project documentation
│   └── report/                # Final project report (PDF)
│
├── Script/                    # Custom scripts for analysis
│   └── performance.py         # Script to evaluate HMM performance (e.g., MCC, TPR, PPV)
│
├── .gitignore                 # Files to ignore in Git
├── LICENSE                    # MIT License
└── README.md                  # This file
```

###  Objectives

- Construct profile HMMs from sequence-based and structure-based alignments of Kunitz domains.
- Evaluate model performance on positive and negative datasets using classification metrics, addressing annotation incompleteness in databases like Pfam.
- Demonstrate the advantage of incorporating structural information for improved sensitivity to remote homologs.

###  Installation
To ensure reproducibility, use a conda environment with the following dependencies:

- Python >= 3.10
- HMMER >= 3.3.2
- MMseqs2 (for sequence clustering)
- UCSF ChimeraX (for structural visualization)
- Biopython, NumPy, Pandas, Matplotlib (for data processing and plotting)
- Optional: CD-HIT >= 4.8.1 (for redundancy reduction), MUSCLE v5 (for sequence alignment)

The environment specification is available in the repository for easy recreation.

### Methodology
 Data Preparation (in Data/)

1- Retrieve protein sequences from UniProtKB/Swiss-Prot (release 2025_03):
- Positive set: Annotated with Pfam PF00014, InterPro IPR036880, or Prosite (PS00280, PS50279).
- Resulted in 368 high-confidence Kunitz proteins after filtering, split into two folds (pos_1, pos_2).
- Negative set: All other Swiss-Prot proteins lacking Kunitz annotations.

2- Acquire PDB structures:
- Pfam PF00014 annotation, resolution ≤ 3.5 Å, length 45–80 amino acids.
- Clustered with CD-HIT at 90% identity for non-redundancy.

3- Remove training-test contamination:
- Use BLASTp to filter sequences ≥95% identical to structural seeds.


#### Multiple Structural Alignment (MStA)

- Align representative structures using PDBeFold with secondary structure matching (SSM).
- Export in FASTA format for HMM input.
- All structures showed canonical Kunitz fold: two β-strands, central α-helix, inhibitory loop, three disulfide bridges.

#### Multiple Sequence Alignment (MSA)

- Generate using MUSCLE v5 on the same representative sequences for fair comparison.

#### HMM Construction, Validation, and Evaluation

1- Build the HMM Profiles
Use HMMER's hmmbuild:
```text
hmmbuild sequence_based.hmm muscle_alignment.ali
hmmbuild structure_based.hmm pdbe_alignment.ali
```
(Files in Data/Processed/)

2- Search for Kunitz Domains
Run hmmsearch on positive/negative folds:
```text
hmmsearch -Z 1000 --max --tblout pos_1.out structure_based.hmm pos_1.fasta
hmmsearch -Z 1000 --max --tblout neg_1.out structure_based.hmm neg_1.fasta
```
Repeat for sequence-based model and folds. 

3- Evaluate Performance
Use performance.py with E-value threshold sweep (e.g., 10^{-1} to 10^{-12}):
```text
for i in `seq 1 12`; do python performance.py set_1.class 1e-$i; done
``` 
Computes confusion matrix, Q2 (accuracy), MCC, TPR (sensitivity), PPV (precision).
Annotation corrections applied using InterPro/Prosite to reassign mislabeled positives.

### Results
- Conservation Analysis: Structural alignment showed clearer positional correspondence, especially around the inhibitory loop.
- Threshold Optimization: Structure-based HMM achieved peak MCC of 0.997 at E-values ~10^{-5}–10^{-6}; sequence-based at 0.991.
- Cross-Validation: Near-perfect performance with 0 false positives and 0–2 false negatives per fold.
- E-value Distribution: Strong separation; positives at 10^{-36} to 10^{-60}, negatives at 1–10.
- Full-Sequence vs. Domain Scoring: Domain-level scoring better for multidomain proteins.
- Misclassification Analysis: Apparent false positives were genuine Kunitz domains missed by Pfam; false negatives had atypical features.

The structure-based model outperformed the sequence-based one, highlighting its value for remote homolog detection. See the full report in Docs/report/Mahan_Balooei_LAB1_Report.pdf for figures (e.g., MCC curves, confusion matrices).

### 📧 Contact
For questions or feedback, please contact:
Mahan Balooei
- Department of Pharmacy and Biotechnology
Alma Mater Studiorum – Università di Bologna
- Email: mahan.balooei@studio.unibo.it
- ORCID: 0009-0006-5358-0784





