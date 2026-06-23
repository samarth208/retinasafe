# RetinaSafe

Comparative evaluation of retinal foundation models for multilabel fundus
classification, with patient-disjoint splits, bootstrap-CI fairness audits,
and per-label calibration analysis.

**Primary contribution:** on the Brazilian Multilabel Ophthalmological Dataset
(BRSET), a general-purpose vision foundation model (DINOv2) outperforms a
domain-specific retinal foundation model (RETFound) on both raw performance
and demographic fairness — and the choice between linear probing and full
fine-tuning is itself a performance-vs-fairness trade-off.

See **[docs/paper_outline.md](docs/paper_outline.md)** for the in-progress
paper structure.

---

## Headline results — BRSET multilabel (12 disease labels)

| Method | Macro AUROC | Macro ECE | Significant subgroup gaps |
|---|---|---|---|
| ResNet-50 (supervised, fine-tuned) | 0.921 | — | 2 |
| RETFound (linear probe) | 0.784 | 0.304 | **7** |
| DINOv2 (linear probe) | 0.897 | 0.156 | **0** |
| RETFound (fine-tuned) | 0.896 | 0.070 | 3 |
| **DINOv2 (fine-tuned, LR=1e-5)** | **0.936** | **0.066** | 2 |

All numbers use patient-disjoint stratified splits (8,524 patients → 5,967
train / 1,278 val / 1,279 test) and 1,000-resample nonparametric bootstrap
95% confidence intervals. Per-disease AUROC and CI tables in
`results/{condition}/test_metrics.json`.

### Per-label performance

![Per-label AUROC](figures/figure1_per_label_auroc.png)

### Per-label calibration

![Per-label ECE](figures/figure2_per_label_ece.png)

### Fairness audit summary

![Fairness summary](figures/figure3_fairness_summary.png)

### Drusens camera-bias case study

![Drusens camera bias](figures/figure5_drusens_camera_bias.png)

---

## Headline results — OCT (previous work, included for completeness)

| Experiment | Dataset | Metric | Value | 95% CI |
|---|---|---|---|---|
| OCT 4-class classification | Kermany OCT2017 | macro AUROC | 0.988 | [0.986, 0.989] |
| OCT cross-dataset transfer (binary referable) | Kermany → OCTDL | AUROC | 0.985 | [0.978, 0.990] |
| Fundus DR 5-class severity | APTOS 2019 | Quadratic Weighted Kappa | 0.905 | [0.871, 0.932] |

These three experiments demonstrate the patient-disjoint-split methodology
and the cross-dataset transfer pipeline; the BRSET-foundation-model
comparison above is the paper's primary contribution.

---

## Methodology highlights

- **Patient-disjoint splits.** Kermany OCT2017's released splits leak 566
  patients across train/test; we restratify by modal class per patient
  (seed 42, 70/15/15). BRSET splits stratified by maximum DR_ICDR severity
  per patient.
- **Bootstrap CIs on every reported metric.** 1,000 resamples, alpha=0.05.
- **Statistically significant fairness gaps** are reported only when the
  95% CIs of two subgroup AUROCs do not overlap. Counts in the fairness
  summary table reflect every such pairwise CI separation.
- **Contamination-aware dataset selection.** EyePACS (used in RETFound's
  pretraining) is excluded from any downstream evaluation. RETFound is the
  Nature 2023 CFP variant.

## Reproducing the results

Each notebook in `notebooks/` is self-contained and runs on a Kaggle T4 GPU
with the relevant datasets attached as Kaggle Inputs.

### OCT arm (notebooks 01–04)

- `01_kermany_baseline.ipynb` → attach `paultimothymooney/kermany2018`
- `02_kermany_to_octdl_transfer.ipynb` → attach Kermany, `orvile/octdl-...`, and the Kermany-baseline notebook output
- `03_aptos_baseline.ipynb` and `04_aptos_eval_fix.ipynb` → attach `mariaherrerot/aptos-2019-dataset`

### BRSET arm (notebooks 05–12)

- `05_brset_baseline.ipynb` → attach `samarthmishra208/brset-...` (your private BRSET upload)
- `06_brset_fairness_audit.ipynb` → attach BRSET + 05's notebook output
- `07_retfound_smoke_test.ipynb` → attach BRSET + `samarthmishra208/retfound-mae-nature-cfp-weights` (your private RETFound upload)
- `08_retfound_linear_probe_brset.ipynb` → attach all 3 inputs above
- `09_dinov2_linear_probe_brset.ipynb` → attach BRSET + 05's notebook output (Internet ON for torch.hub)
- `10_retfound_finetune_brset.ipynb` → same as `08`
- `11_dinov2_finetune_brset_v2.ipynb` → same as `09`, LR=1e-5 with 5-epoch warmup
- `12_paper_figures.ipynb` → attach all 6 BRSET-era notebook outputs

Result JSONs from each run are committed under `results/{condition}/`.

## Roadmap

- [x] OCT baseline + cross-dataset transfer (Kermany / OCTDL)
- [x] Fundus DR severity baseline (APTOS)
- [x] BRSET multilabel baseline (ResNet-50)
- [x] BRSET fairness audit with bootstrap CIs
- [x] RETFound vs DINOv2 head-to-head (linear probe + fine-tuning)
- [x] Per-label calibration analysis
- [x] Paper outline
- [ ] First-draft Methods + Results sections
- [ ] First-draft Introduction + Discussion
- [ ] arXiv preprint
- [ ] Submission

## Datasets used

- **Kermany OCT2017** ([Kaggle](https://www.kaggle.com/datasets/paultimothymooney/kermany2018))
- **OCTDL** (Kulyabin et al. 2024, [Kaggle mirror](https://www.kaggle.com/datasets/orvile/octdl-optical-coherence-tomography-dataset))
- **APTOS 2019 Blindness Detection** ([Kaggle mirror](https://www.kaggle.com/datasets/mariaherrerot/aptos2019))
- **BRSET v1.0.1** (Nakayama et al. 2024, [PhysioNet](https://physionet.org/content/brazilian-ophthalmological/1.0.1/) — credentialed access required)

## Foundation models used

- **RETFound CFP** (Zhou et al. 2023, *Nature*, [HuggingFace](https://huggingface.co/YukunZhou/RETFound_mae_natureCFP) — gated access)
- **DINOv2 ViT-L/14** (Oquab et al. 2023, [torch.hub](https://github.com/facebookresearch/dinov2))

## Closest prior work

- **Hou et al. 2025**, *Ophthalmology Science*, ["Can a Natural Image-Based Foundation Model Outperform a Retina-Specific Model in Detecting Ocular and Systemic Diseases?"](https://arxiv.org/abs/2502.06289) — concurrent DINOv2-vs-RETFound comparison on different (non-BRSET) datasets, fine-tuning only, no fairness audit. This repository confirms their direction and extends with multilabel framing, fairness audit, and per-label calibration on BRSET.
- **Restrepo et al. 2026** ([PhysioNet](https://physionet.org/content/embedding-brset-mbrset/1.0.0/)) — BRSET DINOv3 embeddings release. Concurrent BRSET work using a different model and without RETFound comparison.
- **FairMedFM (Jin et al. 2024)** — fairness benchmarking precedent for medical foundation models.

## License

Code released under MIT (see `LICENSE`). All datasets are governed by their
own licenses — see each dataset's source page. Foundation model weights
(RETFound, DINOv2) are governed by their respective licenses (CC BY-NC 4.0
for RETFound, Apache 2.0 for DINOv2). Results in `results/` and figures in
`figures/` are released under CC BY 4.0.

## Contact

[Your name] — [your email] — independent research

## Citation

Work in progress; a citation block will be added once a preprint exists.
