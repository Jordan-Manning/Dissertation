# Rare Variant Annotation and Structural Analysis Pipeline

MSc Health Data Science dissertation project — University of St Andrews (2026)

## Overview

This pipeline annotates rare genetic variants identified from whole-exome sequencing data and maps candidate variants onto 3D protein structures to assess likely functional impact. It was built to support dissertation analysis of a dyslexia cohort and a twin-study cohort.

The pipeline combines three independent AI variant-effect predictors with structural biology tools:

- **PrimateAI** — matches variants against a locally downloaded PrimateAI score database
- **AlphaMissense** — matches variants against a locally downloaded, compressed AlphaMissense score database
- **EVE** — searches locally downloaded, gene-specific EVE CSV files
- **UniProt REST API** — identifies the reviewed human UniProt entry for each target gene
- **Ensembl VEP REST API** — translates genomic variant coordinates into amino-acid positions
- **AlphaFold Protein Structure Database** — supplies the predicted protein structure for each gene
- **PyMOL** — generates scripts to visualise and highlight the mapped variant residues

> PrimateAI, AlphaMissense and EVE are annotated from **local score databases**, not live APIs. Only the structural workflow (Section 4) queries external services at runtime (UniProt, Ensembl, AlphaFold).

## Pipeline structure

1. **WES Study (DYS)** — PrimateAI and AlphaMissense annotation of the whole-exome sequencing dataset
2. **Twin Study (TWIN)** — the same two-tool annotation applied to the twin-study dataset
3. **EVE** — gene-specific lookup applied to both datasets
4. **3D structural analysis** — for a targeted subset of variants: UniProt lookup → VEP coordinate translation → AlphaFold structure download → PyMOL visualisation script generation
5. **Outputs** — annotated CSVs per tool/dataset, plus PDB structures and PyMOL scripts for the structural subset

## ⚠️ Data and privacy

This repository contains **code only** — no participant data is included or should ever be committed.

- `ExtractedDYS.xlsx`, `ExtractedTWIN.xlsx` and `Targeted_Variants.csv` are cohort-derived variant tables from human research participants and are **not included in this repository**. They are not shared here due to participant confidentiality and university data governance requirements.
- If you want to demonstrate the pipeline running, consider adding a small synthetic/dummy input file that mimics the expected columns, rather than any real data.
- Make sure `.gitignore` excludes any of these filenames if you ever work with real copies locally (see below).

## Requirements

- Python 3.9+
- Packages: `pandas`, `requests` (see `requirements.txt`)
- [PyMOL](https://pymol.org/) (open-source or commercial build) to run the generated `.pml` visualisation scripts

Install packages with:

```bash
pip install -r requirements.txt
```

## Before running

You will need your own copies of:

**User-specific input files** (place in the working directory):
- `ExtractedDYS.xlsx` — WES/dyslexia variants
- `ExtractedTWIN.xlsx` — twin-study variants
- `Targeted_Variants.csv` — variants selected for 3D structural analysis

**External prediction databases** (separate local downloads, not included):
- PrimateAI: `PrimateAI_scores_v0.2_hg38.tsv` — [Illumina BaseSpace](https://basespace.illumina.com/s/yYGFdGih1rXL) · [project info](https://github.com/Illumina/PrimateAI)
- AlphaMissense: `AlphaMissense_hg38.tsv.gz` — [official score file](https://storage.googleapis.com/dm_alphamissense/AlphaMissense_hg38.tsv.gz) · [repository](https://github.com/google-deepmind/alphamissense)
- EVE: bulk gene/protein CSV files — [EVE bulk download](https://evemodel.org/download/bulk)

The 3D structural workflow needs no pre-downloaded database — it queries [UniProt](https://rest.uniprot.org/) and [Ensembl VEP](https://rest.ensembl.org/) at runtime and downloads structures from [AlphaFold DB](https://alphafold.ebi.ac.uk/).

> **Path configuration:** the EVE lookup path is built from your home directory (`~/Documents/University/Dissertation/...`). Update this in the EVE section to point at wherever you extract the EVE `variant_files` directory on your own machine.

## Outputs

Annotation CSVs:
- `DYS_with_PrimateAI.csv`, `DYS_AlphaMissense_Annotated.csv`, `DYS_EVE_Annotated.csv`
- `TWIN_with_PrimateAI.csv`, `TWIN_AlphaMissense_Annotated.csv`, `TWIN_EVE_Annotated.csv`

Structural workflow:
- `pdb_files/<UniProt accession>.pdb` — downloaded AlphaFold structures
- `pymol_scripts/<gene>_visualization.pml` — PyMOL scripts that highlight the mapped variant residues; run directly in PyMOL to reproduce each visualisation

Each annotation step reports how many variants were successfully matched, as a basic sanity check on the process.

## Limitations

- Match rates depend on coordinate/allele formatting consistency between the input tables and each prediction database.
- The structural workflow uses the first VEP transcript consequence with a `protein_start` value, which may not always correspond to the canonical transcript.
- Findings are prediction-based and intended to prioritise candidates for further investigation, not to provide clinical interpretation.

## License

MIT — see [LICENSE](LICENSE).
