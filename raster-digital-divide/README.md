# Territorial Digital Divide – Geospatial Raster Analysis

## Project Description & Research Question

This project builds a geospatial pipeline to measure digital inequality across the Cusco region of Peru. It combines two satellite-derived datasets — NASA Black Marble nighttime radiance (a proxy for urbanization and economic activity) and mobile network coverage kernel density (a proxy for internet access) — to identify where populations face simultaneous deficits in both dimensions.

**Research question:** Which territories in Cusco exhibit the greatest digital divide, and where should policy interventions be prioritized to reduce social exclusion caused by lack of connectivity and urbanization?

## Data

Place the following raster files in `data/` before running the notebook. They are excluded from the repository via `.gitignore` due to their size (>100 MB).

| File | Source | CRS |
|------|--------|-----|
| `VNL_cusco_2025.tif` | NASA Black Marble nighttime radiance | EPSG:4326 |
| `kernel_cobmovil2019_50m.tif` | Mobile coverage kernel density | EPSG:32719 |

## Dependencies & Installation

Python 3.9+ is required. Install all dependencies with:

```bash
pip install -r requirements.txt
```

The `requirements.txt` includes: `rasterio`, `numpy`, `matplotlib`, `scipy`, `seaborn`, and `pandas`.

## How to Run the Notebook

1. Clone the repository and place the two raster files in `data/`.
2. Install dependencies as shown above.
3. Open the notebook:
   ```bash
   jupyter notebook notebooks/digital_divide_cusco.ipynb
   ```
4. Run all cells from top to bottom using **Kernel → Restart & Run All**. No manual intervention is required between steps.
5. All output files will be written to `output/` automatically.

## Output Files

All processed rasters and the composite dashboard are saved in `output/`:

| File | Format | Description |
|------|--------|-------------|
| `vnl_normalized.tif` | GeoTIFF (float32) | NASA nighttime radiance clipped to the 2nd–98th percentile and scaled to [0, 1] |
| `connectivity_normalized.tif` | GeoTIFF (float32) | Mobile coverage density reprojected to EPSG:4326 and normalized to [0, 1] |
| `ibd_index.tif` | GeoTIFF (float32) | Digital Divide Index (IBD = VNL − Connectivity), range [−1, 1]; positive values indicate areas with light but no coverage |
| `territorial_classification.tif` | GeoTIFF (uint8) | 2×2 territorial classification: 1 = Urban Connected, 2 = Urban Divide, 3 = Rural Connected, 4 = Critical Divide |
| `dashboard.png` | PNG (150 dpi) | Composite map showing all six layers: normalized VNL, normalized connectivity, IBD, EDT, social exclusion risk, and territorial classification |

## Pipeline Summary

| Step | Description |
|------|-------------|
| 0–1 | Load rasters; report CRS, dimensions, bands, NoData, bounds, and pixel resolution |
| 2 | Reproject connectivity EPSG:32719 → EPSG:4326 with bilinear resampling; align to VNL grid |
| 3 | Percentile normalization (p2–p98) to [0, 1]; NoData and negatives replaced with 0 |
| 4 | Side-by-side visualization of raw and normalized VNL using the `inferno` colormap |
| 5 | Compute IBD (Digital Divide Index) and EDT (Total Digital Exclusion) indices |
| 6 | Classify intervention priority into 3 levels based on VNL and connectivity thresholds |
| 7 | Compute Social Exclusion Risk; apply Gaussian filter (σ=5); display raw and smoothed maps |
| 8 | Build 2×2 territorial classification (4 classes); export to GeoTIFF; build statistics table |
| 9 | Descriptive stats by class, Pearson correlation (40-pixel subsample), KDE plots, Welch's t-test (Class 1 vs. Class 4) |
| 10 | Export 4 processed rasters and composite dashboard at 150 dpi |

## Key Findings

The analysis reveals that the vast majority of Cusco's territory falls into the **Critical Divide** class (Class 4), where both nighttime radiance and mobile coverage are below the connectivity threshold — confirming that digital exclusion in the region is primarily a rural and geographic phenomenon rather than an urban one. The IBD index shows that areas with high radiance but low connectivity (Urban Divide, Class 2) are spatially concentrated and represent a secondary but policy-relevant gap where infrastructure investment would yield the highest impact. The Welch's t-test comparing Class 1 (Urban Connected) against Class 4 (Critical Divide) yields a statistically significant difference in VNL values (large Cohen's d), confirming that the two classes are clearly separable and that territorial stratification captures a real underlying socioeconomic gradient.
