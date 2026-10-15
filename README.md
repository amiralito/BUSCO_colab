# BUSCO Completeness Assessment (Colab)

Batch [BUSCO](https://busco.ezlab.org) on Google Colab: score the completeness of **proteomes, genome
assemblies or transcriptomes** against **one or more OrthoDB lineages**, from a single notebook with
form-field inputs. Point it at a FASTA (or a whole folder of them) and it returns a consolidated score
table, the standard BUSCO stacked-bar plot, a BUSCO × sample status matrix, and every run archived to
Google Drive.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/amiralito/BUSCO_Colab/blob/main/BUSCO_assessment.ipynb)

## Pipeline

```
FASTA(s)          →   BUSCO  (per sample × lineage)                →   busco_summary.tsv   →   plot + status matrix
proteome / genome     proteins: HMMER                                  short summaries          stacked bars per lineage
/ transcriptome       genome:   miniprot (or MetaEuk) → HMMER          full tables              S/D/F/M matrix, shared SCOs
(one or many)         transcriptome: MetaEuk → HMMER                    .tar.gz run archives
```

## What it does

- Runs BUSCO over **one or many inputs** (recursive Drive folder scan, single file, upload, or URLs),
  in `proteins`, `genome` or `transcriptome` mode, against **any number of lineages** (comma-separated).
- Installs BUSCO from source (optionally pinned to a release) with HMMER, **miniprot**, **MetaEuk**,
  BBTools, Prodigal and pigz — no conda.
- **Caches lineage datasets on Drive** (`busco_lineages/<lineage>.tar`) so each is downloaded once;
  can also read pre-downloaded lineage folders (e.g. a copy of an HPC `busco_lineages/` directory).
- Stages a local uncompressed copy of each input (`.gz` accepted) and checks its alphabet against the mode.
- After **every** run: copies the short summary (`.txt` + `.json`) and gzipped `full_table.tsv`,
  archives the BUSCO directory as `.tar.gz`, and rewrites `busco_summary.tsv` — so a disconnect loses at
  most the run in progress.
- **Resumable batches**: `SKIP_DONE` reuses finished runs, `RESUME_RUN` reopens an interrupted run
  folder in a new session, and a failed sample gets an `NA` row while the batch carries on.
- `busco_summary.tsv`: C/S/D/F/M percentages and counts, internal-stop-codon % (miniprot), assembly
  metrics in genome mode (scaffolds, contigs, length, N50s, gaps), BUSCO version, lineage date, runtime.
  Rows are keyed by (sample, mode, lineage), so several modes and lineages accumulate in one table.
- Draws the **BUSCO stacked-bar plot** (one panel per mode × lineage; PNG/PDF/SVG).
- Builds a **BUSCO × sample status matrix** (`S`/`D`/`F`/`M`) and lists BUSCOs that are single-copy in
  at least a chosen fraction of samples (a starting set for phylogenomics).
- Writes `run_config.json` with tool versions, lineage dataset dates, inputs and parameters for each
  invocation.
- Every run gets a **timestamped run folder** so consecutive runs never overwrite each other.

## Requirements

- Google Colab with a **CPU runtime** (BUSCO does not use the GPU). More cores help genome and
  transcriptome mode; proteins mode is quick even on the free 2-core runtime.
- Input FASTA: `.fa/.fasta/.fna/.faa/.fas/.fsa/.pep/.aa/.ffn/.cds`, optionally `.gz`. Each file is one sample.
- `*_odb12.2` lineages need BUSCO ≥ 6.1 (the default `latest`); with 5.8.x / 6.0.x use `*_odb12`.

## Quick start

1. Open the notebook in Colab (CPU runtime is fine).
2. **Cell 1** — runtime check; **cell 2** — install BUSCO + tools (~1–2 min).
3. **Cell 3** — mount Drive, set `WORKDIR`, stamp this run's folder (or resume one).
4. **Cell 4** — pick the mode, lineage(s) and input source.
5. **Cell 5** — fetch lineage datasets; **cell 6** — collect/stage inputs.
6. **Cell 7** — run BUSCO; **cell 8** — table + plot; **cell 9** — status matrix; **cell 10** — download.

To add another lineage or mode to the same table, change cell 4 and re-run cells 4–9.

## Output layout

```
WORKDIR/
├── busco_lineages/                  # cached lineage datasets (.tar), shared by all runs
└── runs/run_<date>-<time>[_tag]/
    ├── busco_summary.tsv
    ├── run_config.json
    ├── summaries/<lineage>/         # short_summary.<run>.txt / .json
    ├── full_tables/<lineage>/       # <run>_full_table.tsv.gz
    ├── archives/<lineage>/          # <run>.tar.gz (whole BUSCO directory)
    ├── logs/                        # <run>.log (full BUSCO output)
    ├── plots/                       # busco_summary.png / .pdf
    └── matrices/                    # busco_matrix_<mode>_<lineage>.tsv, sco_<mode>_<lineage>_min<f>.txt
```

Run names follow `<sample>_busco_<prot|genome|tx>_<lineage>`.

## Notes

- BUSCO writes symlinks, which the Drive mount does not support, so runs execute locally in
  `/content/busco_runs` and only copies/archives go to Drive.
- In proteins mode, use one isoform per gene; alternative transcripts inflate `D%`.
- Compare samples only within the same lineage **and** dataset release.
- Genome/transcriptome mode on whole plant genomes is slow on Colab cores — fine for small genomes and
  drafts, better on HPC at scale.

## Citation

- Manni M, Berkeley MR, Seppey M, Simão FA, Zdobnov EM (2021) BUSCO update. *Mol Biol Evol* 38:4647–4654.
- Tegenfeldt F *et al.* (2025) OrthoDB and BUSCO update. *Nucleic Acids Res* 53:D516–D522.
- Li H (2023) Protein-to-genome alignment with miniprot. *Bioinformatics* 39:btad014.
- Levy Karin E, Mirdita M, Söding J (2020) MetaEuk. *Microbiome* 8:48.
