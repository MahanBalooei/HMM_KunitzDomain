
# HMM_KunitzDomain  
**Structure-Based Profile Hidden Markov Modeling for Kunitz-Type Protease Inhibitor Domains**

This repository contains the code, data references, and evaluation scripts for a bioinformatics project that models the BPTI/Kunitz domain using profile Hidden Markov Models (HMMs). The project combines both **sequence-based** and **structure-informed** approaches for domain detection and annotation in UniProt protein sequences.

---

## 📁 Repository Structure

```
HMM_KunitzDomain/
├── docs/
│   └── report.pdf           # Final project report (PDF)
├── results/
│   ├── evaluation/          # Classification metrics, logs, confusion matrices
│   └── hmm_profile/         # Final profile HMMs
├── scripts/
│   └── performance.py       # Script for evaluating HMM classification performance
└── README.md
```

---

## 🎯 Project Goals

- ✅ Build profile HMMs from both sequence and structure-based alignments of Kunitz domains
- ✅ Evaluate model performance on curated positive and negative datasets
- ✅ Determine optimal detection thresholds using Matthews Correlation Coefficient (MCC)
- ✅ Demonstrate improved performance of structure-informed HMMs on distant homologs

---

## 🧪 Tools & Dependencies

Make sure the following are installed:

- Python ≥ 3.8  
- [HMMER 3.4+](http://hmmer.org)  
- [CD-HIT 4.8.1+](https://github.com/weizhongli/cdhit)  
- [BLAST+ 2.16.0+](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download)  
- [UCSF Chimera](https://www.cgl.ucsf.edu/chimera/) (for structural inspection)  
- [Git LFS](https://git-lfs.github.com) (for large datasets)

---

## ⚙️ Installation Example

```bash
# Clone the repository
git clone https://github.com/MahanBalooei/HMM_KunitzDomain.git
cd HMM_KunitzDomain

# Install Git LFS and fetch large FASTA files
git lfs install
git lfs pull

```

---

## 📖 Method Summary

### 1. Data Collection
- Protein sequences were retrieved from [UniProtKB/Swiss-Prot](https://www.uniprot.org) (Taxonomy ID: 9606, Pfam: PF00014).
- Structural representatives were extracted from [PDB](https://www.rcsb.org) with resolution ≤ 3.5Å and clustered using CD-HIT at 90% identity.

### 2. Structure-Based Alignment
- Representative PDB structures were aligned using [PDBeFold](https://www.ebi.ac.uk/msd-srv/ssm/).
- Outliers were filtered using RMSD and Q-score thresholds.

### 3. HMM Construction
```bash
hmmbuild pdb_kunitz_struct.hmm pdb_kunitz_strali.fasta
```

### 4. Domain Search
```bash
hmmsearch -Z 1000 --max --tblout pos_1.out pdb_kunitz_struct.hmm pos_1.fasta
hmmsearch -Z 1000 --max --tblout neg_1.out pdb_kunitz_struct.hmm neg_1.fasta
```

### 5. Evaluation
```bash
for i in `seq 1 12`; do python3 scripts/performance.py results/evaluation/set_1.class 1e-$i; done
```
- Outputs: MCC, Q2 (Accuracy), TPR (Sensitivity), PPV (Precision), confusion matrix

---

## 📊 Key Results

- ✅ **Structure-based HMM MCC**: up to **0.997**
- ✅ **Sequence-based HMM MCC**: up to **0.991**
- 🧠 Optimal E-value threshold: **1e-05 to 1e-06**
- 🧬 Conserved Cysteines (Cys) modeled accurately across alignments
- 🧩 Identified true Kunitz domains even when Pfam annotations were missing

For full discussion and evaluation figures, see: [`docs/report.pdf`](docs/report.pdf)

---

## 📫 Contact

**Mahan Balooei**  
Bioinformatics MSc, University of Bologna  
📧 mahan.balooei@studio.unibo.it  

_This project was developed for the Laboratory of Bioinformatics I, 2025._

---

## 📄 License
This project is licensed under the [MIT License](LICENSE).

---

## 🔗 Acknowledgments
- Dr. Capriotti (supervisor)
- UniProt, Pfam, InterPro
- UCSF Chimera, HMMER, CD-HIT, PDBeFold tools
