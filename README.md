# Profile HMMs for the Kunitz/BPTI Domain: Sequence Alignment vs Structural Alignment

MSc Bioinformatics, University of Bologna. Project for the Laboratory of Bioinformatics 1 course (Prof. Emidio Capriotti).

The Kunitz/BPTI domain is a small protease-inhibitor fold of 50 to 70 residues held together by three disulfide bridges. Its structure is well conserved while its sequence is not, which makes it a good case for asking a simple question: if you build a profile HMM from a structural alignment instead of a sequence alignment of the same proteins, does it find the domain any better?

I built both models from the same set of representative PDB structures and tested them on all of Swiss-Prot.

## Result in short

| Model | Alignment | Best MCC | E-value threshold at best MCC |
|---|---|---:|---|
| Structure-based | PDBeFold | 0.997 | 1e-5 to 1e-6 |
| Sequence-based | MUSCLE | 0.991 | 1e-5 to 1e-6 |

Both models separate Kunitz proteins from everything else almost perfectly. The structure-based model scores slightly higher, but the gap is a handful of proteins out of more than 573,000, and I have no variance estimate for it, so I would not call it a meaningful difference. For a domain this well conserved, either alignment gives a model that works.

The more interesting finding was about the labels, not the models (see "What the errors turned out to be").

## Data

**Positives.** Swiss-Prot release 2025_03, proteins annotated with Pfam PF00014, InterPro IPR036880 or the PROSITE Kunitz signatures (PS00280, PS50279). The union gives 398 proteins; 368 remain after removing truncated and low-quality sequences.

**Negatives.** Every other Swiss-Prot protein.

**Structures for the models.** PDB entries annotated with PF00014, resolution of 3.5 Å or better, chain length 45 to 80 residues, clustered with CD-HIT at 90% identity to remove near-duplicates.

**Keeping model and test data apart.** The HMMs are built only from the PDB seed structures. Any Swiss-Prot sequence at least 95% identical to a seed over 50 or more aligned residues was removed from the evaluation set with BLASTp.

## Method

1. **Structural alignment.** The seed structures were aligned with PDBeFold and the alignment exported as FASTA. All of them show the canonical fold: two antiparallel strands, a central helix, the inhibitory loop and the three disulfides.
2. **Sequence alignment.** The same seed sequences were aligned with MUSCLE v5, so the two models differ only in how the alignment was made.
3. **Model building.**
   ```bash
   hmmbuild structure_based.hmm pdb_kunitz_nr_clean.ali
   hmmbuild sequence_based.hmm muscle_alignment.ali
   ```
4. **Search.** Positives and negatives were each split in two halves, giving two evaluation sets. Each was searched with `--max` (heuristic filters off) and `-Z 1000` so that E-values are comparable between runs.
   ```bash
   for i in 1 2; do
     hmmsearch -Z 1000 --max --tblout pos_$i.out structure_based.hmm pos_$i.fasta
     hmmsearch -Z 1000 --max --tblout neg_$i.out structure_based.hmm neg_$i.fasta
   done
   ```
   Sequences that HMMER does not report at all are given a placeholder E-value so that every protein is counted.
5. **Scoring.** The hits are collected into one file per set (`set_1.class`, `set_2.class`) with three whitespace-separated columns: sequence ID, true label (1 or 0) and E-value. `Script/performance.py` then reports the confusion matrix, accuracy, MCC, sensitivity and precision at a given threshold:
   ```bash
   for i in $(seq 1 12); do
     python Script/performance.py set_1.class 1e-$i
     python Script/performance.py set_2.class 1e-$i
   done
   ```
   The threshold that works best on one set can be checked on the other. MCC is the main metric because the classes are extremely unbalanced.

## What the errors turned out to be

At the best thresholds the structure-based model makes no false positives and at most two false negatives per set.

The false negatives (for example D3GGZ8, O62247, A0A1Q1NL17 and Q8WPG5) have unusual loop lengths or cysteine spacing.

The apparent false positives were the instructive part. When I first labelled positives using Pfam PF00014 alone, several "wrong" hits turned out to be real Kunitz proteins that simply lack the Pfam cross-reference in Swiss-Prot and are annotated only through InterPro or PROSITE. That is why the final positive set uses all three sources. It also means the labels in this kind of benchmark are not perfectly clean, and scores this close to 1 depend partly on how the positives were defined.

Full-sequence and best-domain E-values agree for most proteins. They diverge for a few multidomain proteins, where the full-sequence score adds up weak matches; for those the domain-level score is the one to trust.

## Limitations

- The two evaluation sets were not checked for redundancy against each other, and Kunitz proteins in Swiss-Prot include many close homologs, so very low E-values for positives are partly expected.
- I did not test the models separately on distant homologs, so this project cannot say whether structural alignment helps more where sequence identity is low. That is the case where one would expect it to matter.
- A single split, no confidence intervals.

## What is in the repository

```
Data/Raw/         Swiss-Prot FASTA files used to build the positive and negative sets
Data/Processed/   Structural alignment (pdb_kunitz_nr_clean.ali) and the HMM built from it
Script/           performance.py and the notebooks that draw the figures
Docs/             Project report (PDF) with all figures
environment.yml   Conda environment
requirements.txt  Python packages
```

Only the structure-based alignment and model are included. The sequence-based model can be rebuilt by aligning the same seed sequences with MUSCLE.

## Installation

```bash
git clone https://github.com/MahanBalooei/HMM_KunitzDomain.git
cd HMM_KunitzDomain
conda env create -f environment.yml
conda activate hmm-kunitz
```

The conda environment installs HMMER, MUSCLE, CD-HIT and MMseqs2. BLAST+ is also needed for the contamination filter and is not in the environment file (`conda install -c bioconda blast`). The structural alignment was made on the PDBeFold web server, and the structure figure in the report with UCSF ChimeraX.

## Contact

Mahan Balooei, Department of Pharmacy and Biotechnology, University of Bologna
mahan.balooei@studio.unibo.it · ORCID [0009-0006-5358-0784](https://orcid.org/0009-0006-5358-0784)

## License

MIT. See `LICENSE`.
