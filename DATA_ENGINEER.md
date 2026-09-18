# Data Engineer Guide

**Owns**: `src/data/`, `data/`
**Deliverable**: a reproducible, versioned dataset pipeline that any model can consume without modification.

[← Back to main README](../../README.md)

---

## Objective

Turn raw Sentinel-2 satellite imagery and public labeled datasets into clean, patched, model-ready tensors — with a pipeline that can be re-run on demand to pull fresh live imagery for inference.

## Scope

- Sentinel Hub / Copernicus API integration
- Band selection and normalization
- Tiling large scenes into fixed-size patches
- Dataset versioning and documentation
- A shared `data_loader.py` interface used by the ML layer

## Responsibilities

### 1. API access
- Register for a Sentinel Hub account and OAuth client credentials
- Store credentials in `.env` (`SENTINEL_HUB_CLIENT_ID`, `SENTINEL_HUB_CLIENT_SECRET`) — never commit these
- Implement token refresh handling in `fetch_sentinel.py`

### 2. Data acquisition
- Pull Sentinel-2 L2A tiles for defined regions and date ranges
- Select relevant bands: RGB (B4, B3, B2), NIR (B8), SWIR (B11, B12) — these carry most of the signal for cloud and vegetation discrimination
- Support both historical batch pulls (for training) and live single-tile pulls (for inference demo)

### 3. Preprocessing
- Normalize reflectance values (typically scaled 0–10000 → 0–1)
- Handle nodata / masked pixels at scene edges
- Optionally compute derived indices (NDVI, NDWI) as additional input channels

### 4. Patching
- Split full scenes (often 10,000×10,000 px) into fixed patches, e.g. 256×256
- Discard or flag patches that are mostly nodata
- Maintain a consistent train/val/test split at the patch level to avoid data leakage from overlapping tiles

### 5. Dataset management
- Store processed patches as `.npy` or `.tif` under `data/patches/`
- Document dataset version, source, date range, and band configuration in `data/README.md`
- Keep raw data out of version control; use `.gitignore` and, if needed, Git LFS or cloud storage for large files

## File structure

```
src/data/
├── fetch_sentinel.py     # API client, live tile retrieval
├── preprocess.py         # normalization, band selection, index computation
├── patchify.py           # tiling logic, train/val/test split
└── data_loader.py         # PyTorch Dataset/DataLoader used downstream

data/
├── raw/                  # untouched downloads
├── processed/            # normalized scenes
└── patches/               # model-ready patches
```

## Interfaces provided to other roles

| Consumer | Interface |
|---|---|
| ML Engineer | `data_loader.py` — returns `torch.utils.data.Dataset` yielding (image, mask) pairs |
| Backend Engineer | `fetch_sentinel.py::get_live_tile(lat, lon, date)` — used at inference time |

## Definition of done

- [ ] Live tile fetch works for an arbitrary lat/lon and returns normalized bands
- [ ] Training dataset is patched, split, and documented
- [ ] `data_loader.py` is tested and consumable independently of the fetch scripts
- [ ] No credentials or raw imagery committed to the repo

## Suggested tools

`sentinelsat`, `rasterio`, `GDAL`, `numpy`, `python-dotenv`
