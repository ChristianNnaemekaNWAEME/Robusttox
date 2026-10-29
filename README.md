# RobustTox

A research library and benchmark for **contamination-robust, interpretable toxicity prediction** from molecular structure.

Research question: *when does robust, sparse estimation beat standard models on real toxicity data, and can it be made accurate and explainable at the same time?*

This repository extends the earlier `toxicity-estimator-benchmark` (which showed that a trimmed L1-logistic estimator was the most stable under label noise but not the most accurate) by testing three candidate fixes, adding structured contamination, and mapping the selected features back to substructures.

## Layout
```
robusttox/        features.py (RDKit descriptors + Morgan counts)  split.py (randomised scaffold splits)
                  methods.py (L1-Logit, SLTS-linear (J-5), TrimLogit, AdaTrim, StabilitySelection, RobustTox)
                  alerts.py (Brenk / PAINS / NIH via RDKit FilterCatalog)
scripts/          prepare_data.py  run_benchmark.py  run_alerts_baseline.py  run_simulation.py  analyze.py  interpret.py
tests/            pytest suite (stratified trimming, monotone C-steps, split disjointness, support recovery)
docs/             THEORY.md  SKILLS.md  PAPER_OUTLINE.md
results/          tables/*.csv, figures
```

## Reproduce
```bash
pip install numpy pandas scikit-learn scipy rdkit matplotlib pytest
# put tox21.csv clintox.csv sider.csv toxcast_data.csv (MoleculeNet) in one folder, then:
python scripts/prepare_data.py /path/to/csvs
export OMP_NUM_THREADS=1
python scripts/run_benchmark.py 0 2 3 & python scripts/run_benchmark.py 1 2 3   # two workers, three scaffold splits
python scripts/run_alerts_baseline.py
python scripts/run_simulation.py 30
python scripts/analyze.py
python scripts/interpret.py RobustTox
pytest tests
```

## Data (all public, via MoleculeNet)
Tox21 (12 tasks), ClinTox (CT_TOX), SIDER (Hepatobiliary disorders), and three ToxCast hepatocyte high-content-screening endpoints (HepG2 cell loss 72 h, mitochondrial membrane potential 24 h, oxidative stress 24 h). **The ToxCast endpoints are in-vitro hepatocyte readouts, not clinical drug-induced liver injury**; a DILI set (e.g. DILIrank) should be added when it can be downloaded.

## Contamination models
`random`: each training label flipped with probability eps. `margin`: the eps-fraction of training compounds that a reference model, fitted on the clean labels, classifies most confidently are flipped (a worst case for the direction of the fit). Test labels are never altered.

## Caveats
* Only the training labels are corrupted; the test set is clean.
* Scaffold splits are randomised per seed; three splits per endpoint is a minimum, not a recommendation for the final paper (use 10).
* Sparse-model feature identity is only meaningful up to the correlation among Morgan bits and descriptors.
