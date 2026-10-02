# ConformalLadder

**Layer-resolved error budgets with distribution-free guarantees for routing quantum-chemical calculations.**

[![Open in Kaggle](https://kaggle.com/static/images/open-in-kaggle.svg)](https://www.kaggle.com/kernels/welcome?src=https://github.com/alikhairbek/ConformalLadder/blob/main/ConformalLadder.ipynb)
[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/alikhairbek/ConformalLadder/blob/main/ConformalLadder.ipynb)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)

ConformalLadder turns **one cheap electronic-structure calculation** into

* an **exact, layer-resolved error budget** — basis set (β), exchange–correlation functional (φ), dispersion (δ) and remainder (ρ) — with the non-additive coupling between layers *measured*, not assumed;
* **distribution-free conformal intervals** for every layer (guaranteed 90 % coverage under exchangeability);
* a **certificate** that the corrected cheap result is within 1 kcal/mol of the reference, with a controlled false-discovery rate (conformal p-values + Benjamini–Hochberg);
* a **router** that keeps certified items at the cheap level, escalates the rest one rung, and *abstains* — escalating ~50 items to the reference level for recalibration — whenever a new batch is not exchangeable with the calibration data.

It is evaluated on 130,258 QM9 molecules with G4MP2 references, 21,468 MultiXC-QM9 isomerization energies (76 functionals × 3 basis sets), 7,211 molecules of the QM7b HF/MP2/CCSD(T) grid, and a new 3,440-point functional × basis grid on W4-17 computed with PySCF (8 functionals × def2-SVP/def2-TZVP + D3/D4 + FOD).

---

## Everything runs with one click: `ConformalLadder.ipynb`

The single notebook at the root of this repository **downloads the raw data, processes it, runs all four studies, and produces every figure, table and number of the paper** — no manual steps, no configuration.

| Where | How |
|---|---|
| **Kaggle** (recommended) | Click the *Open in Kaggle* badge above (or *File → Import Notebook → GitHub* and paste the notebook URL). In the right panel set **Settings → Internet → On**. Press **Run All**. CPU is enough (4 cores). |
| **Google Colab** | Click the *Open in Colab* badge and press *Runtime → Run all*. |
| **Locally** | `git clone https://github.com/alikhairbek/ConformalLadder && cd ConformalLadder && pip install -r requirements.txt && jupyter lab ConformalLadder.ipynb`, then *Run All*. |

What happens during the run:

1. installs PySCF, dftd3, dftd4 and RDKit (everything else is in the standard images);
2. writes the library (`src/conformalladder`) and the analysis scripts (`notebooks/`) to disk — they are embedded in the notebook, so the notebook is self-contained;
3. downloads QM9-G4MP2 (78 MB, GitHub), ACCDB (60 MB, GitHub) and MultiXC-QM9 (533 MB, DTU Data/Figshare); the precomputed W4-17 grid (`data/processed/w4-17_grid.jsonl`) is taken from this repository, so the 2–3 h PySCF campaign is **not** repeated unless you ask for it;
4. runs pilot 0 (QM9-G4MP2), pilot 1 (MultiXC-QM9 isomerization ladder), pilot 2 (QM7b correlation × basis non-additivity), the abstention gates and the escalate-to-recalibrate loop, and the W4-17 rung budget with its learned layers;
5. renders the six main figures, the SI figure (Wong palette, 300 dpi PNG + PDF), all tables (`results/tables/*.csv`, `SUMMARY.md`) and packs everything into `ConformalLadder_results.zip`.

**Runtime:** ≈ 40 min on a 4-core Kaggle CPU session with the precomputed grid (≈ 15 min of it is the MultiXC download), ≈ 3 h if the PySCF campaign is recomputed (`RUN_W4_CAMPAIGN = True` and no grid present).

**The only thing that cannot be downloaded automatically** is the QM7b multilevel grid of Zaspel *et al.* (JCTC 2019, **15**, 1546): it is ACS Supporting Information (`ct8b00832_si_001.zip`) and the ACS server refuses scripted downloads. Download it once by hand from the article page and attach it as a (private) Kaggle dataset, drop it into `/content/drive/MyDrive` on Colab, or place it under `data/raw/qm7b_multilevel/` locally — the notebook finds it (also when Kaggle has auto-extracted it). Without it, pilot 2 and its three shift scenarios are skipped and everything else runs unchanged.

Settings live in the first code cell (`RUN_W4_CAMPAIGN`, `W4_FUNCTIONALS`, `RUN_RECALIBRATION`, `CLEAN_RAW_CSV`, `GITHUB_REPO`). All random operations are seeded (`SEED = 20260920`).

## What you get

```
results/pilot0/pilot0_summary.png            Fig. 2  QM9-G4MP2: intervals, FDP vs q, routing under shift
results/pilot1_multixc/pilot1_summary.png    Fig. 3  MultiXC-QM9: four-layer ladder, non-additivity, certification
results/pilot2_qm7b/pilot2_summary.png       Fig. 4  QM7b: correlation × basis non-additivity of composite schemes
results/w4-17/w4_17_rung_comparison.png      Fig. 5  W4-17: layer budget for seven target rungs
results/gates/gates_and_recalibration.png    Fig. 6  abstention gates + escalate-to-recalibrate
results/figures/fig1_schematic.png           Fig. 1  framework schematic
results/w4-17/w4_17_budget_PBE_def2-SVP.png  Fig. S1 learned layers with conformal intervals on W4-17
results/tables/T1…T8*.csv, SUMMARY.md        every headline number (Tables 1–3 and S1–S11 of the paper are built from these)
results/**/*.json                            full per-experiment results
```

## Reproducibility

A fresh run of the notebook on Kaggle reproduces every number of the three large studies and of the measured W4-17 budget **exactly** (bit-identical tables). The learned-layer statistics on W4-17, whose training set holds only 417 reactions, can differ between machines by a few reactions per split (up to 0.04 in the coverage of a single split); they are therefore reported as means ± s.d. over ten random splits, which are stable to ~0.01.

## Repository layout

```
ConformalLadder.ipynb      the one-click notebook (built from the sources by kaggle/build_notebook.py)
src/conformalladder/       library: conformal.py (intervals, weighted conformal, conformal p-values + BH), layers.py (exact
                           layer identity, interactions, features), gates.py (abstention gates), data_*.py (data sets),
                           compute_grid.py (PySCF campaign), kaggle_data.py (downloads), palette.py (figure style)
notebooks/                 the analysis scripts run by the notebook (pilots, gates, recalibration, W4-17, figures, tables)
scripts/                   stand-alone downloader for a workstation, packing helpers, campaign polling
data/processed/w4-17_grid.jsonl   the W4-17 functional × basis grid computed in this work (3,440 DFT single points,
                           215 FOD calculations, 5,160 dispersion evaluations) — the only data file in the repository
results/                   figures (PNG + PDF), tables and JSON results of the published run
manuscript/                manuscript sources: main_text.md, references.py, build_docx.py (house-format Word builder),
                           numbers.json (every number quoted in the text, extracted from results/), out/ (DOCX + PDF)
kaggle/                    notebook builder, an executed copy of the notebook, Kaggle notes
docs/                      method notes and development log
```

## Data sources

| Data set | Reference | Licence / access |
|---|---|---|
| QM9 | Ramakrishnan *et al.*, *Sci. Data* 2014, 1, 140022 | open |
| QM9-G4MP2 | Ward *et al.*, *MRS Commun.* 2019, 9, 891 — github.com/globus-labs/g4mp2-atomization-energy | open |
| MultiXC-QM9 | Nandi, Vegge & Bhowmik, *Sci. Data* 2023, 10, 783 — DTU Data | open (CC BY) |
| QM7b multilevel | Zaspel *et al.*, *J. Chem. Theory Comput.* 2019, 15, 1546 — ACS Supporting Information | download by hand (see above) |
| W4-17 / ACCDB | Karton *et al.*, *J. Comput. Chem.* 2017, 38, 2063; Morgante & Peverati, *J. Comput. Chem.* 2019, 40, 839 — github.com/peverati/ACCDB | open |
| W4-17 DFT grid | this work (`data/processed/w4-17_grid.jsonl`) | MIT (this repository) |

## Building the manuscript

```
PYTHONPATH=src python manuscript/extract_numbers.py   # results/ -> manuscript/numbers.json
PYTHONPATH=src python manuscript/build_docx.py        # -> manuscript/out/ConformalLadder_main.docx + ConformalLadder_SI.docx
```

Every number in the text is substituted from `numbers.json`, so a re-run of the notebook followed by these two commands regenerates a manuscript whose numbers match the new results.

## Citation

If you use this code or the W4-17 grid, please cite the repository (see `CITATION.cff`) and the manuscript:

> Khairbek, A. A. *ConformalLadder: layer-resolved error budgets with distribution-free guarantees for routing quantum-chemical calculations.* (2026, manuscript in preparation).

## Licence

Code, notebook and the W4-17 grid: MIT (see `LICENSE`). The external data sets keep their own licences and must be obtained from their original sources, as the notebook does.
