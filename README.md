# Kunitz Domain HMM Pipeline

## Index

- [Introduction](#introduction)
- [Recommended Packages](#recommended-packages)
- [Required Files](#required-files)
- [Workflow](#workflow)
- [Author](#Author)

---

## Introduction

This repository presents the implementation of a computational workflow designed to construct and evaluate a Profile Hidden Markov Model (HMM) focused on a specific protein domain of interest. The project involves generating a structure-based multiple sequence alignment of representative domain-containing proteins, from which an HMM is built and calibrated. The resulting model is tested against both positive (domain-containing) and negative (domain-absent) protein sequences to assess its classification performance. The project was carried out as part of a Laboratory in Bioinformatics course during my MSc in Bioinformatics, and integrates principles of structural bioinformatics, statistical modeling, and sequence analysis. 

---

## Recommended Packages

The pipeline depends on the following bioinformatics tools and libraries, which must be installed before running the workflow. Most of them are available through the conda package manager (via the Bioconda or Conda-Forge channels):

- **CD-HIT**: Groups highly similar protein sequences into clusters to reduce redundancy in input datasets. This ensures that the training set for the HMM is non-redundant.  
  `bash conda install -c bioconda cd-hit`

- **HMMER**: Core tool for building Profile Hidden Markov Models and scanning sequence databases for domain detection. Essential for training and testing the structural HMM.  
  `bash conda install -c bioconda hmmer`

- **BLAST+**: Performs similarity searches (via blastp) to retrieve known homologous sequences or verify hits. Useful for validating HMM predictions.  
  `conda install -c bioconda blast`

- **MUSCLE** (Optional — only needed for sequence-based HMM comparison): Generates multiple sequence alignments from sequence data, used to build the baseline sequence-based HMM for comparison against the structure-based model.  
  `conda install -c bioconda muscle`

- **Biopython**: Required to run `get_seq.py`, a script that extracts specific sequences from FASTA files based on identifiers. Enables automation and parsing of biological data formats.  
  `conda install -c conda-forge biopython`

---

## Required Files

- [`rcsb_pdb_custom_report_20250407015737.csv`](./rcsb_pdb_custom_report_20250407015737.csv) – Custom report downloaded from RCSB PDB, filtered for Kunitz domain structures (identifier = 'PF00014' AND annotation type = 'Pfam' AND polymer entity sequence length <= 80 AND polymer entity sequence length >= 45). Custom the report with these flags: Entry ID, PDB ID, Entity ID, Auth Asym ID, Sequence, Annotation Identifier, Data collection resolution.
- [`script_recover_representative_kunitz.sh`](./script_recover_representative_kunitz.sh) – Script to extract representative PDB IDs of Kunitz domain structures.
- [`create_hmm_str.sh`](./create_hmm_str.sh) – Script to build the structural HMM model from the structural alignment.
- [`create_testing_sets.sh`](./create_testing_sets.sh) – Script to generate the test sets and compute performance metrics.
- [`performance.py`](./performance.py) – Python script to compute evaluation metrics (MCC, precision, TPR, etc.) for model predictions.
- [`getseq.py`](./getseq.py) – Python script to extract specific sequences from a FASTA file based on a list of accession IDs.
- It is also essential to download all protein sequences available in the Swiss-Prot database, as they will be used to construct the negative dataset.


---

## Workflow

### Step 1: Run Representative ID Extraction Script
Execute the following script:

```bash script_recover_representative_kunitz.sh```
This will generate the file [`tmp_pdb_efold_ids.txt`](./tmp_pdb_efold_ids.txt).

Note: Before submitting to PDBeFold, manually review the sequences to ensure they are of appropriate length and free of unstructured regions. This ensures alignment and HMM quality.

### Step 2: Perform Structure-Based Multiple Alignment
Go to the PDBeFold Multi Alignment Tool.

Set the following options:

- Mode: Multiple

- Source: List of PDB codes

- Upload the [`tmp_pdb_efold_ids.txt`](./tmp_pdb_efold_ids.txt) file and download the FASTA alignment.

Paste the downloaded content into the [`pdb_kunitz_rp.ali`](./pdb_kunitz_rp.ali) file.

### Step 3: Build and Test the HMM Model

Execute the following script:

```bash create_hmm_str.sh```

Then execute:

```bash create_testing_sets.sh```

This pipeline will:

- Build the structural HMM from the PDBeFold alignment.

- Create training and test sets by separating positive (Kunitz) and negative (non-Kunitz) sequences.

- Perform 2-fold cross-validation to identify the optimal E-value thresholds using MCC.

- Evaluate the model on Set 1 (set_1_strali.class), Set 2 (set_2_strali.class), and the combined dataset.

- Report MCC, precision, recall, false positives, and false negatives.

The results will be saved in the [`hmm_results_strali.txt`](./hmm_results_strali.txt) file.

## Author

Valerio Piersanti

MSc Bioinformatics - University of Bologna

Laboratory of Bioinformatics 1 (LB1)

Further details can be found in the article [`Lab1_project_paper`](./Lab1_project_paper.pdf)

Fell free to contact me on valerio.piersanti3@unibo.studio.it 


