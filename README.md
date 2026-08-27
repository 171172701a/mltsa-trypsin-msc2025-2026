# Reproducibility-aware MLTSA — trypsin–benzamidine

Analysis code for the MSc dissertation *Reproducibility-aware machine-learning
transition state analysis of benzamidine unbinding from trypsin*
(UCL, MSc Scientific and Data Intensive Computing, 2026).

**Yucheng Zhao** · Supervisors: Prof. Edina Rosta, Dr. Pedro J. Buigues

`DESIGN.md` documents the software itself: architecture, correctness, performance
and reproducibility.

---

## What this does

173 unbiased molecular dynamics trajectories of benzamidine dissociating from
trypsin are described by 274 ligand-distance collective variables. A classifier
is trained to predict each trajectory's committed outcome — rebind or release —
and the coordinates it relies on are taken as candidate mechanistic
determinants. The pipeline additionally measures how reproducible those
feature-importance rankings are before interpreting them.

Headline results, all out-of-fold under trajectory-grouped cross-validation:

| | |
|---|---|
| Balanced accuracy | 0.737 (95% CI 0.668–0.800) |
| Label-permutation null | 0.497 ± 0.040 |
| Windows chosen by the automated search | all five outer folds selected frames > 2,000 |
| Permutation-importance reproducibility, late window | top-15 Jaccard 0.034 |
| Water share of top-50 features, transition → settled | 26% → 4% (odds ratio 8.4) |

---

## Layout

| Path | Contents |
|---|---|
| `notebooks/run_improved_pipeline_v4.ipynb` | Nested window and hyperparameter search; ≈ 12.2 h |
| `notebooks/window_fi_contrast.ipynb` | Two fixed windows, three importance measures |
| `notebooks/window_fi_figures.ipynb` | Figure generation from stored artefacts |
| `notebooks/block_permutation.ipynb` | Block permutation importance; 73 blocks, ≈ 2.1 h |
| `notebooks/verify_fi.ipynb` | Recomputes every reported importance statistic from `fi_per_seed.npz` |
| `results/` | All numerical outputs; every figure in the dissertation regenerates from these |
| `figures/` | Figures as submitted |

Notebooks are committed with their outputs intact, so the analysis can be
inspected without access to the trajectory data.

### Key result files

| File | Contents |
|---|---|
| `run_metadata.json` | Full argument set, dataset summary, and a `candidate_grid_provenance_caveat` field recording that the candidate grid was narrowed using earlier exploratory runs |
| `summary.json` | Every reported accuracy, confidence interval, selected window, null baseline and feature ranking |
| `reproducibility.json` | Per-seed accuracies and split-half reproducibility floors |
| `fi_per_seed.npz` | Per-seed importance matrices, three measures × two windows (0.13 MB) |
| `fi_table_TS.csv`, `fi_table_late.csv` | Per-feature importances with residue and water identities |
| `block_perm_summary.json`, `block_perm_table.csv` | Block-level floors, cross-window agreement, water enrichment |
| `feature_names.csv` | Collective-variable index → residue or water rank |

---

## Data

The trajectory ensemble — 173 trajectories × 2,500 frames × 274 collective
variables, stored as HDF5 — was generated within the Rosta group and is **not
redistributed here**. Contact the supervisors for access. The pipeline expects it
at the path recorded in `results/run_metadata.json`.

---

## Reproducing the results

Every number and figure in the dissertation can be regenerated from `results/`
**without** rerunning the 12.2-hour search:

```bash
pip install -r requirements.txt
jupyter notebook notebooks/verify_fi.ipynb
```

That notebook reloads the stored per-seed arrays and recomputes the
reproducibility floors, cross-window correlations, method-agreement matrix,
feature composition counts and leading-feature lists from scratch.

Seeds are fixed and recorded: root seed 42 for the nested search, seeds 0–9 for
the two-window analysis.

---

## Environment

Python 3.12. Key packages: scikit-learn, SHAP 0.50.0, h5py, MDTraj, NumPy,
pandas, SciPy, matplotlib. Full listing in `requirements.txt`.
