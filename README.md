# biophysical

A Python package for estimating biophysical parameters from Sentinel-2 satellite data, created to supplement the 🌿 **Computing Leaf Area Index Using Sentinel-2 Data** 🛰️ notebook submitted for the [EOPF Zarr Community Notebook Competition](https://github.com/eopf-toolkit/community-notebook-competition/).

The package implements the ESA SNAP S2ToolBox BiophysicalOp neural network algorithm to compute **Leaf Area Index (LAI)** from Sentinel-2 Level-2A bottom-of-atmosphere reflectances. It supports Sentinel-2A, -2B, and -2C at both 10 m and 20 m spatial resolutions, and saves results as CF-1.13 compliant Zarr stores or GeoTIFF files.

---

## Features

- **Neural network inference** via [Modular MAX](https://docs.modular.com/max/) — two-layer tanh neural network with float32 weights bundled from ESA S2ToolBox coefficient files
- **Input validation** — per-band min/max range check and convex-hull domain grid check (two-stage, per the ATBD)
- **Output validation** — extreme-cases thresholding with tolerance bands
- **Viewing angle computation** — per-detector bilinear interpolation of 23×23 geometry grids, merged using detector footprint masks and averaged across input bands
- **CF-1.13 compliant output** — Zarr stores with full global attributes, CRS grid-mapping variable, projected coordinate variables, and provenance `history` tracing the STAC search and processing steps
- **GeoTIFF output** — multi-band with CF-inspired GDAL string tags
- **Supported sensors and resolutions:**

| Enum | Sensor | Resolution | Input bands |
|---|---|---|---|
| `S2A` | Sentinel-2A | 20 m | B03, B04, B05, B06, B07, B8A, B11, B12 |
| `S2A_10m` | Sentinel-2A | 10 m | B03, B04, B08 |
| `S2B` | Sentinel-2B | 20 m | B03, B04, B05, B06, B07, B8A, B11, B12 |
| `S2B_10m` | Sentinel-2B | 10 m | B03, B04, B08 |
| `S2C` | Sentinel-2C | 20 m | B03, B04, B05, B06, B07, B8A, B11, B12 |
| `S2C_10m` | Sentinel-2C | 10 m | B03, B04, B08 |

---

## Requirements

- Python ≥ 3.13
- [Modular MAX](https://docs.modular.com/max/) (nightly) — installed separately from Modular's wheel index (see below)

---

## Installation

### 1. Install [uv](https://docs.astral.sh/uv/) if you don't have it

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Restart your shell after installation so the `uv` command is available.

### 2. Create and activate a virtual environment

```bash
uv venv && source .venv/bin/activate
```

### 3. Install the package and its dependencies

`pyproject.toml` declares all dependencies including `modular`. uv routes `modular` to Modular's nightly wheel index automatically:

```bash
uv sync
```

---

## Usage in a Jupyter notebook (another repository)

### Option A — add as a uv source dependency (recommended)

In the other repo's `pyproject.toml`, declare this package as a git source and configure the Modular nightly index so that `modular` resolves transitively:

```toml
[project]
dependencies = [
    "biophysical",
    # ...
]

[tool.uv]
prerelease = "allow"

[[tool.uv.index]]
name = "modular-nightly"
url = "https://whl.modular.com/nightly/simple/"
explicit = true

[tool.uv.sources]
modular = { index = "modular-nightly" }
biophysical = { git = "https://github.com/<your-username>/biophysicalop.git" }
```

Then run:

```bash
uv sync
```

### Option B — pip install in a notebook cell

If the notebook kernel's environment already has `modular` installed, install the rest of the package directly:

```python
%pip install git+https://github.com/Geomatys/biophysicalop.git
```

---

## Package structure

```
src/biophysical/
├── angles.py        — Detector footprint masking and viewing angle computation
├── constants.py     — Enums, file path constants, and conversion factors
├── exceptions.py    — Custom exception types
├── io.py            — Save results to CF-1.13 Zarr or GeoTIFF
├── models.py        — Data models: SatelliteSensors, BiophysicalVariables, Result
├── normalization.py — Min-max normalisation / denormalisation for NN I/O
├── processing.py    — Neural network inference via MAX graph engine
├── utils.py         — Bilinear interpolation, Sentinel-2 product name parsing
├── validation.py    — Input domain checks and output range validation
├── xaffine.py       — Affine transform helpers for xarray DataArrays
└── resources/       — Bundled ESA S2ToolBox coefficient files (weights, norms, domains)
```

---

## References

Weiss, M., Baret, F. (2016). *S2ToolBox Level 2 products: LAI, FAPAR, FCOVER.* Algorithm Theoretical Basis Document (ATBD), ESA. https://step.esa.int/docs/extra/ATBD_S2ToolBox_V1.1.pdf

---

## License

Copyright © 2026 [Geomatys, SAS](http://www.geomatys.com). Licensed under the [Apache License, Version 2.0](LICENSE).
