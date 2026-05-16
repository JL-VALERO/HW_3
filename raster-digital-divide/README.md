# Territorial Digital Divide – Geospatial Raster Analysis

Geospatial pipeline that measures digital inequality in Cusco, Peru using NASA nighttime radiance (VNL) and mobile network coverage density as proxies for urbanization and internet access.

## Data

Place the following files in `data/` (not committed — see `.gitignore`):

- `VNL_cusco_2025.tif` — NASA Black Marble nighttime radiance (EPSG:4326)
- `kernel_cobmovil2019_50m.tif` — Mobile coverage kernel density (EPSG:32719)

## Setup

```bash
pip install -r requirements.txt
```

## Execution

Open and run all cells in `notebooks/digital_divide_cusco.ipynb` in order.

## Outputs

All processed rasters and the composite dashboard are saved in `output/`:

| File | Description |
|------|-------------|
| `vnl_normalized.tif` | VNL normalized to [0, 1] |
| `connectivity_normalized.tif` | Connectivity normalized to [0, 1] |
| `ibd_index.tif` | Digital Divide Index (IBD) |
| `territorial_classification.tif` | 2×2 territorial classification (4 classes) |
| `dashboard.png` | Composite map dashboard at 150 dpi |

## Pipeline Summary

| Step | Description |
|------|-------------|
| 0–1 | Load rasters, inspect CRS, dimensions, NoData, bounds, resolution |
| 2 | Reproject connectivity EPSG:32719 → 4326, align to VNL grid |
| 3 | Percentile normalization (p2–p98) to [0, 1] with NoData handling |
| 4 | Visualize raw vs. normalized VNL (inferno colormap) |
| 5 | Compute IBD and EDT indices |
| 6 | Classify intervention priority (3 levels) |
| 7 | Social exclusion risk with Gaussian smoothing (σ=5) |
| 8 | Territorial 2×2 classification + GeoTIFF export |
| 9 | Statistical analysis: correlation, KDE, Welch's t-test |
| 10 | Export 4 rasters + composite dashboard |
