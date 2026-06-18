# RetinaSafe

Patient-disjoint OCT classification, cross-dataset transfer, and fundus
diabetic-retinopathy severity baselines — supporting work for spatial-decomposition
adapter research on retinal foundation models.

## Results

| Experiment | Dataset | Metric | Value | 95% CI |
|---|---|---|---|---|
| OCT 4-class classification | Kermany OCT2017 | macro AUROC | **0.9877** | [0.986, 0.989] |
| OCT cross-dataset transfer (binary referable) | Kermany → OCTDL | AUROC | **0.9850** | [0.978, 0.990] |
| Fundus DR 5-class severity | APTOS 2019 | Quadratic Weighted Kappa | **0.9046** | [0.871, 0.932] |
| Fundus multilabel (12 diseases) | BRSET | macro AUROC | **0.9211** | per-label CIs in `results/` |

All numbers use **patient-disjoint splits** (where the dataset supports it) and
**1,000-resample bootstrap** confidence intervals.

## BRSET fairness audit (statistically significant findings only)

Subgroup gaps where 95% CIs do not overlap:

- **Camera bias on `drusens`** — AUROC 0.890 [0.867, 0.912] on Canon CR vs.
  0.808 [0.760, 0.849] on Nikon NF5050. ~8 percentage point gap that persists
  after bootstrap testing.
- **Image quality on `increased_cup_disc`** — AUROC 0.916 [0.902, 0.930] on
  Adequate-quality images vs. 0.845 [0.784, 0.898] on Inadequate-quality.

No statistically significant subgroup differences detected by sex or age band.

All numbers use **patient-disjoint splits** (where the dataset supports it) and
**1,000-resample bootstrap** confidence intervals.

## Methodology notes

- Kermany splits resplit patient-disjointly (released splits leak 566 patients
  across train/test; we restratify by modal class per patient, seed 42).
- OCTDL transfer evaluated under the **binary referable** scheme (Normal = 0, all
  6 disease classes = 1) to align the 4-class Kermany head with OCTDL's 7-class
  label space.
- APTOS lacks patient IDs in the released metadata, so splits are
  image-disjoint, not patient-disjoint. This is a known dataset limitation.
- Training: ResNet-50 ImageNet-pretrained, AdamW, cosine LR with 2-epoch
  warmup, AMP, class-balanced sampling, class-weighted CE.

## Reproducing the results

Each notebook in `notebooks/` is self-contained and runs on a Kaggle T4 GPU
with the datasets attached as Kaggle Inputs:

- `kermany_baseline.ipynb` → attach `paultimothymooney/kermany2018`
- `kermany_to_octdl_transfer_v3.ipynb` → attach Kermany, OCTDL (`orvile/octdl-...`), and the kermany-baseline notebook output
- `aptos_baseline.ipynb` + `aptos_eval_fix.ipynb` → attach `mariaherrerot/aptos-2019-dataset`

Result JSONs from each run are committed in `results/<experiment>/`.

## Roadmap

- BRSET multilabel fundus classification with demographic fairness audit
- mBRSET cross-camera external validation
- Spatial-decomposition adapter on RETFound (the actual paper contribution)

## Citation

This work is in progress; a citation block will be added once a preprint exists.

## License

MIT. Datasets used here are governed by their own licenses — see each dataset's
source page.
