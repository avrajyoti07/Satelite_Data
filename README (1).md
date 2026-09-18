<div align="center">

# SatSeg

### Cloud & Land Cover Segmentation from Live Satellite Imagery

Pixel-level semantic segmentation of Sentinel-2 imagery using U-Net — served through a live API and mapped in real time.

![Python](https://img.shields.io/badge/python-3.10+-blue.svg)
![PyTorch](https://img.shields.io/badge/PyTorch-2.x-ee4c2c.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-backend-009688.svg)
![License](https://img.shields.io/badge/license-MIT-lightgrey.svg)
![Status](https://img.shields.io/badge/status-in%20development-yellow.svg)

[Overview](#overview) · [Architecture](#architecture) · [Roles](#roles) · [Roadmap](#roadmap) · [Setup](#setup) · [Docs](#documentation)

</div>

---

## Overview

Raw Sentinel-2 imagery spans 13 spectral bands and needs two things before it's useful for downstream analysis: **cloud masking** and **land cover classification**. This repository implements that pipeline end-to-end, using it as a learning vehicle for production-style ML systems design:

- **Data ingestion** — near-real-time Sentinel-2 tiles via the Sentinel Hub API
- **Model** — U-Net with a pretrained encoder, trained on labeled cloud/land-cover datasets
- **Serving** — FastAPI inference service with async job handling and caching
- **Visualization** — interactive map where a user picks a location and sees the segmentation overlay live

This mirrors preprocessing pipelines run in production by ESA, USGS, and commercial providers like Planet Labs — scoped down to something buildable and demonstrable by a single contributor.

---

## Architecture

```
 ┌────────────────────────────────────────────────────────────────┐
 │  DATA LAYER                                                     │
 │  Sentinel Hub API   →   38-Cloud / EuroSAT   →   GDAL / rasterio │
 └────────────────────────────────┬─────────────────────────────────┘
                                   ▼
 ┌────────────────────────────────────────────────────────────────┐
 │  ML LAYER                                                        │
 │  Preprocessing   →   U-Net (train / fine-tune)   →   Evaluation │
 └────────────────────────────────┬─────────────────────────────────┘
                                   ▼
 ┌────────────────────────────────────────────────────────────────┐
 │  SERVING LAYER                                                    │
 │  Model export (ONNX)  →  FastAPI backend  →  Async job queue    │
 └────────────────────────────────┬─────────────────────────────────┘
                                   ▼
 ┌────────────────────────────────────────────────────────────────┐
 │  FRONTEND LAYER                                                   │
 │  Map UI (Leaflet)  →  Overlay viewer (mask vs. original)        │
 └────────────────────────────────────────────────────────────────┘
```

**Request flow**: user selects a location on the map → backend resolves and fetches the latest Sentinel-2 tile → tile is preprocessed and passed through the trained U-Net → predicted mask is returned and rendered as a map overlay.

---

## Tech stack

| Layer | Tools |
|---|---|
| Data | Sentinel Hub API, `sentinelsat`, GDAL, rasterio |
| Modeling | PyTorch, `segmentation-models-pytorch`, albumentations |
| Experiment tracking | Weights & Biases / CSV logs |
| Backend | FastAPI, Celery, Redis |
| Frontend | React, Leaflet.js |
| Infrastructure | Docker, Render / Railway / Hugging Face Spaces |
| Dev tooling | Google Colab (GPU training), GitHub Actions |

---

## Repository structure

```
satseg/
├── data/                     # raw, processed, and patched imagery
├── src/
│   ├── data/                 # ingestion & preprocessing
│   ├── model/                # U-Net, training, evaluation
│   ├── api/                  # FastAPI service
│   └── utils/
├── frontend/                 # map UI (React + Leaflet)
├── notebooks/                # EDA, training runs, analysis
├── models/                   # saved weights (git-ignored / LFS)
├── tests/
├── docs/
│   └── roles/                # role-specific deep-dive docs (see below)
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── README.md
```

---

## Roles

This project is built solo but structured as if staffed by a small team — each role owns a layer of the architecture, has its own scope, and ships an independent, verifiable deliverable. Full responsibilities, workflows, and technical detail for each role live in dedicated docs:

| Role | Owns | Deliverable | Docs |
|---|---|---|---|
| Data Engineer | `src/data/`, `data/` | Reproducible dataset pipeline | [docs/roles/DATA_ENGINEER.md](docs/roles/DATA_ENGINEER.md) |
| ML Engineer | `src/model/`, `notebooks/` | Trained model + evaluation report | [docs/roles/ML_ENGINEER.md](docs/roles/ML_ENGINEER.md) |
| Backend Engineer | `src/api/` | Inference API | [docs/roles/BACKEND_ENGINEER.md](docs/roles/BACKEND_ENGINEER.md) |
| Frontend Engineer | `frontend/` | Interactive map demo | [docs/roles/FRONTEND_ENGINEER.md](docs/roles/FRONTEND_ENGINEER.md) |
| MLOps Engineer | `Dockerfile`, CI/CD | Live deployment | [docs/roles/MLOPS_ENGINEER.md](docs/roles/MLOPS_ENGINEER.md) |

---

## Roadmap

- [ ] **Phase 1 — Data pipeline**: Sentinel Hub integration, dataset acquisition, patching
- [ ] **Phase 2 — Model training**: U-Net baseline, augmentation, metric tracking
- [ ] **Phase 3 — Backend API**: inference endpoint, async queue, caching
- [ ] **Phase 4 — Frontend**: map UI, overlay visualization
- [ ] **Phase 5 — Deployment**: containerization, hosting, monitoring
- [ ] **Phase 6 — Stretch goals**: multi-class land cover, time-series change detection

Development is not time-boxed — each phase is completed and verified before the next begins.

---

## Datasets

| Dataset | Purpose | Source |
|---|---|---|
| 38-Cloud | Binary cloud segmentation, ready-made masks | Kaggle / GitHub |
| CloudSEN12 | Large-scale cloud segmentation | Hugging Face |
| EuroSAT | Multi-class land cover | TensorFlow Datasets |
| DeepGlobe | Land cover classification | Kaggle |
| Sentinel Hub | Live imagery for inference | sentinel-hub.com |

---

## Setup

```bash
git clone https://github.com/<your-org>/satseg.git
cd satseg

python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# populate SENTINEL_HUB_CLIENT_ID and SENTINEL_HUB_CLIENT_SECRET

uvicorn src.api.main:app --reload      # backend
cd frontend && npm install && npm run dev   # frontend
```

---

## Documentation

Full documentation lives in [`docs/`](docs/README.md):

- [Architecture](docs/ARCHITECTURE.md) — full system design and data flow
- [Getting Started](docs/GETTING_STARTED.md) — local setup
- [Data Pipeline](docs/DATA_PIPELINE.md)
- [Model](docs/MODEL.md)
- [API Reference](docs/API_REFERENCE.md)
- [Frontend](docs/FRONTEND.md)
- [Deployment](docs/DEPLOYMENT.md)
- [Roadmap](docs/ROADMAP.md)
- [FAQ](docs/FAQ.md)

Role guides:
- [Data Engineer](docs/roles/DATA_ENGINEER.md)
- [ML Engineer](docs/roles/ML_ENGINEER.md)
- [Backend Engineer](docs/roles/BACKEND_ENGINEER.md)
- [Frontend Engineer](docs/roles/FRONTEND_ENGINEER.md)
- [MLOps Engineer](docs/roles/MLOPS_ENGINEER.md)

See also: [CONTRIBUTING.md](CONTRIBUTING.md), [CHANGELOG.md](CHANGELOG.md)

## References

- Ronneberger, O., Fischer, P., Brox, T. — *U-Net: Convolutional Networks for Biomedical Image Segmentation* (2015)
- ESA Sentinel-2 documentation — sentinel.esa.int
- `segmentation-models-pytorch` — github.com/qubvel/segmentation_models.pytorch

## License

MIT — see [LICENSE](LICENSE).
