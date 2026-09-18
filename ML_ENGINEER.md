# ML Engineer Guide

**Owns**: `src/model/`, `notebooks/`
**Deliverable**: a trained segmentation model with a documented evaluation report.

[← Back to main README](../../README.md)

---

## Objective

Train a U-Net model that accurately segments cloud (and optionally land cover classes) from Sentinel-2 patches, and prove its performance with proper segmentation metrics — not accuracy alone.

## Scope

- Model architecture and backbone selection
- Training loop, augmentation, regularization
- Evaluation methodology
- Experiment tracking and reproducibility

## Responsibilities

### 1. Model architecture
- Use `segmentation-models-pytorch` for a ready-made U-Net rather than implementing from scratch
- Start with a binary task (cloud vs. not-cloud) before attempting multi-class land cover
- Compare at least two encoder backbones (e.g. ResNet34 vs. MobileNetV2) for an accuracy/speed tradeoff discussion

### 2. Training
- Input: patches + masks from `data_loader.py`
- Loss: combine BCE/Cross-Entropy with Dice loss — pure pixel accuracy is misleading when cloud coverage is a small fraction of the image
- Augmentation via `albumentations`: horizontal/vertical flips, rotation, brightness/contrast jitter — satellite orientation is arbitrary, so flips are safe and effective
- Use a learning rate scheduler and early stopping to avoid overfitting on small datasets

### 3. Evaluation
- Primary metrics: **IoU (Intersection over Union)** and **Dice coefficient**, computed per class and averaged
- Report a confusion matrix for multi-class runs
- Visualize failure cases (thin cirrus clouds, cloud shadows are the hardest cases — call these out explicitly)

### 4. Experiment tracking
- Log hyperparameters, metrics, and sample predictions per run (Weights & Biases free tier, or a structured CSV/JSON log if avoiding external dependencies)
- Keep a `notebooks/03_results_analysis.ipynb` summarizing the best runs with example predictions

### 5. Export
- Export the final model to ONNX or TorchScript for the backend to consume — decouples the serving layer from PyTorch training internals

## File structure

```
src/model/
├── unet.py           # model definition (via smp)
├── train.py           # training loop, checkpointing
├── evaluate.py         # IoU / Dice computation
└── infer.py            # single-patch inference, used for export testing

notebooks/
├── 01_eda.ipynb              # band distributions, cloud coverage stats
├── 02_training.ipynb         # Colab training runs
└── 03_results_analysis.ipynb  # final metrics, visualizations
```

## Interfaces provided to other roles

| Consumer | Interface |
|---|---|
| Backend Engineer | exported model file (`models/unet.onnx`) + `infer.py::predict(patch) → mask` |

## Definition of done

- [ ] Model trains end-to-end on the patched dataset without errors
- [ ] IoU / Dice reported on a held-out test set, not just training loss
- [ ] At least one backbone comparison documented
- [ ] Model exported in a format the backend can load without PyTorch training dependencies
- [ ] Sample predictions (good and bad cases) visualized in a notebook

## Suggested tools

`segmentation-models-pytorch`, `torch`, `albumentations`, `wandb` (optional), `onnx`
