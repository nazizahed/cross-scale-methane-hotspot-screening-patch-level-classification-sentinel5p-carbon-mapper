# Stage 3: Local Anomaly and Enhancement Workflow

Stage 3 computes local-background methane anomaly and standardized
enhancement products from monthly Sentinel-5P XCH4 rasters, then samples
Carbon Mapper plume-event coordinates against the resulting enhancement masks.

The workflow is a kernel-sensitivity and contextual-overlap analysis. It is
not a Sentinel-5P plume-detection-rate estimate and does not replace
high-resolution Carbon Mapper/Tanager plume observations.
For the thesis-level research framing and literature context, see
[../docs/thesis-context.md](../docs/thesis-context.md).

## Main Notebook

```text
local_anomaly_enhancement.ipynb
```

## Workflow

1. Authenticate Google Earth Engine.
2. Build strict CONUS geometry from `TIGER/2018/States`.
3. Add a 2.5-degree support buffer around CONUS.
4. Export monthly Sentinel-5P XCH4 mean GeoTIFFs.
5. Compute local-background anomaly for 1-6 degree windows.
6. Estimate local sigma after local p95 capping.
7. Calculate standardized enhancement `Z`.
8. Create binary masks where `Z >= 2`.
9. Filter Carbon Mapper plume events to a case area and month.
10. Sample plume-event coordinates against each kernel mask and `Z` raster.
11. Export kernel-overlap tables and diagnostic figures.

The equations, raster-processing details, and assumptions are documented in
[../docs/stage3-technical-reference.md](../docs/stage3-technical-reference.md).

## Thesis Result Anchor

In the October 2024 Visalia case, the number of six Carbon Mapper plume-event
records intersecting the `Z >= 2` enhancement mask varied with the
local-background window: two or three records intersected the smaller and
intermediate 1-4 degree masks, while all six intersected the 5 and 6 degree
masks.

This is a processing-scale sensitivity result, not a Sentinel-5P plume
detection rate.

## Requirements

- Python 3.10-3.12 is recommended.
- Packages in the repository-level `requirements.txt`.
- Google Earth Engine access.
- An Earth Engine export destination for monthly mean XCH4 rasters.
- A Carbon Mapper plume-event CSV with longitude, latitude, and preferably
  timestamp/gas columns.
- GPU runtime is recommended for large kernels or full-archive local
  processing.

Install dependencies from the repository root:

```bash
python -m pip install -r requirements.txt
```

PyTorch installation can vary by CPU/GPU/CUDA platform. If the shared
requirements install a CPU-only version but GPU acceleration is required, install
the PyTorch build recommended for your runtime before running the notebook.

## Required Input

### Sentinel-5P monthly mean rasters

The local processor expects monthly mean GeoTIFFs named:

```text
XCH4_mean_YYYY_MM_CONUSbuf2p5deg.tif
```

These can be exported by the notebook's Earth Engine section. Large rasters
should remain outside Git. The thesis data release is stored on Zenodo; Google
Drive may still be used as a temporary Earth Engine export destination during
notebook execution.

### Carbon Mapper plume-event CSV

The case comparison expects a plume-event table with coordinate columns.
Recognized longitude examples:

```text
plume_longitude
plume_lon
longitude
lon
source_longitude
point_source_longitude
```

Recognized latitude examples:

```text
plume_latitude
plume_lat
latitude
lat
source_latitude
point_source_latitude
```

Recognized timestamp examples:

```text
scene_timestamp
datetime
timestamp
observation_time
date
```

If a `gas` column exists, the notebook keeps CH4 records.

## Configuration

Edit the `USER SETTINGS` cell before running.

### Earth Engine

```python
EE_PROJECT = None
S5P_COLLECTION = "COPERNICUS/S5P/OFFL/L3_CH4"
PREFERRED_BAND = "CH4_column_volume_mixing_ratio_dry_air_bias_corrected"
FALLBACK_BAND = "CH4_column_volume_mixing_ratio_dry_air"
```

Set `EE_PROJECT` if your Earth Engine account requires a Google Cloud project.

### Time range and kernels

```python
YEARS = [2023, 2024, 2025]
MONTHS = list(range(1, 13))
BUFFER_DEG = 2.5
KERNEL_DEG_LIST = [1, 2, 3, 4, 5, 6]
SIGMA_MIN = 0.1
ENH_THRESHOLD = 2.0
```

Kernel values are full local-background window widths in degrees. The code
converts each width to a pixel radius using `PIXEL_DEG = 0.01`.

### Paths

The default paths target Google Colab and mounted Google Drive as a runtime
export workspace:

```python
ROOT = Path("/content/drive/MyDrive/stage3_local_anomaly")
XCH4_MEAN_DIR = ROOT / "XCH4_mean"
LOCAL_PRODUCTS_DIR = ROOT / "local_products"
SUMMARY_DIR = ROOT / "summary_tables"
FIGURE_DIR = ROOT / "figures"
PLUMES_CSV = "/content/drive/MyDrive/stage3_local_anomaly/carbonmapper_CH4_plumes_CONUS_2023_2025.csv"
```

Update these paths for your local or Drive layout.

### Case study

The default case is the Visalia area in October 2024:

```python
CASE_YEAR = 2024
CASE_MONTH = 10
CASE_NAME = "Visalia_October_2024"
CASE_BBOX = [-120.1, 35.9, -118.8, 36.9]
```

Change these values to analyze a different month or region.

## Running

From the repository root:

```bash
jupyter lab stage3/local_anomaly_enhancement.ipynb
```

Recommended sequence:

1. Install dependencies and mount Drive if using Colab.
2. Configure Earth Engine, folders, years/months, and case settings.
3. Run Earth Engine initialization.
4. Export monthly mean XCH4 rasters if they are not already available.
5. Wait for Drive exports to complete.
6. Start with one month and one kernel to test memory and paths.
7. Run the full kernel set for the case month.
8. Load case plume events.
9. Sample masks and continuous `Z` rasters.
10. Generate kernel-sensitivity and intermediate-product figures.
11. Run QA checks before using results.

Full archive processing is intentionally disabled by default:

```python
RUN_FULL_ARCHIVE = False
```

Set it to `True` only after testing because the 2023-2025, 12-month,
six-kernel run is computationally expensive.

## Outputs

The notebook can generate:

```text
XCH4_mean_YYYY_MM_CONUSbuf2p5deg.tif
anomaly_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
sigma_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
enhancementZ_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
enhancement_ge2_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
Visalia_October_2024_sampled_kernel_values.csv
Visalia_October_2024_kernel_overlap_summary.csv
Visalia_October_2024_kernel_masks.png
Visalia_October_2024_intermediate_products_5deg.png
```

Large GeoTIFF products and generated summaries are ignored by Git.

## Interpretation

- `XCH4_mean` is a monthly Sentinel-5P methane field in ppb.
- Local anomaly is `XCH4 - local background`, also in ppb.
- Standardized enhancement `Z` is dimensionless.
- `Z >= 2` is a thresholded enhancement mask for sensitivity analysis.
- Carbon Mapper records are point samples of plume events, not a continuous
  source inventory.
- Point-to-mask overlap does not prove Sentinel-5P detected an individual
  plume.
- Kernel width controls the local background scale and can strongly affect
  overlap counts.

## Troubleshooting

**Earth Engine initialization fails**

Confirm Earth Engine access and set `EE_PROJECT` if your account requires a
Cloud project.

**Monthly mean raster missing**

Run the Earth Engine monthly mean export cell, wait for Drive completion, and
confirm the file is in `XCH4_MEAN_DIR` with the expected filename.

**PyTorch is unavailable**

Install `torch` for the current runtime. For GPU, use the PyTorch command
matching the CUDA version of the environment.

**GPU out of memory**

Reduce `TILE`, process a smaller kernel/month batch, or use a larger GPU
runtime.

**No case plume events**

Check `PLUMES_CSV`, coordinate column names, timestamp parsing, gas filtering,
month/year settings, and `CASE_BBOX`.

**Mask sampling fails**

Confirm all expected `enhancement_ge2_*` and `enhancementZ_*` files exist for
the selected year, month, and kernel list.

**Plots look empty**

Check that the case bounding box intersects valid raster cells and that
`Z >= 2` actually occurs for the selected kernel/month.
