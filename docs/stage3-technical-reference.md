# Stage 3 Technical Reference

## Scope

Stage 3 calculates local-background methane anomaly and standardized
enhancement products from monthly Sentinel-5P XCH4 fields. It then tests how
Carbon Mapper plume-event points overlap binary enhancement masks across
multiple local-background kernel widths.

The implementation is:

```text
stage3/local_anomaly_enhancement.ipynb
```

## Data Flow

```text
Earth Engine Sentinel-5P monthly XCH4
        |
        v
Monthly mean XCH4 GeoTIFFs with CONUS support buffer
        |
        v
Local tiled anomaly and sigma processing
        |
        v
Anomaly, sigma, enhancement Z, and Z >= 2 masks
        |
        v
Carbon Mapper point sampling and kernel-sensitivity summaries
```

## Domain and Support Buffer

The strict study domain is CONUS, built from `TIGER/2018/States` after
excluding Alaska, Hawaii, Puerto Rico, Guam, the Virgin Islands, American
Samoa, and the Northern Mariana Islands.

The notebook also creates:

```text
conus_support_geom = CONUS buffered by 2.5 degrees * 111,320 m/degree
```

Monthly XCH4 rasters are exported over this support geometry. The buffer
provides external pixels for local-background calculations near the CONUS
boundary and reduces edge artifacts.

## Sentinel-5P Source

Earth Engine collection:

```text
COPERNICUS/S5P/OFFL/L3_CH4
```

Preferred band:

```text
CH4_column_volume_mixing_ratio_dry_air_bias_corrected
```

Fallback band:

```text
CH4_column_volume_mixing_ratio_dry_air
```

The notebook checks the first image's band names and chooses the preferred
band when available. Monthly means are calculated with `ImageCollection.mean()`
for each configured year and month.

When a month has no imagery, the notebook creates a fully masked empty image
so export logic remains consistent.

## Earth Engine Export

Monthly mean exports use:

- Drive folder `stage3_XCH4_mean`;
- output prefix `XCH4_mean_YYYY_MM_CONUSbuf2p5deg`;
- region `conus_support_geom`;
- nominal scale from the selected Sentinel-5P band projection;
- GeoTIFF format;
- `maxPixels = 1e13`.

The export function starts asynchronous Drive tasks. It does not poll for
completion, retry failures, or verify downloaded files.

The notebook also provides an optional direct Earth Engine product exporter
for anomaly, sigma, continuous enhancement, and binary enhancement masks.
This optional route is useful for small tests. The main reproducible branch is
the local tiled processor.

## Local-background Definitions

For a monthly XCH4 raster and kernel width `k`, the notebook calculates a
local background around each pixel:

```text
B_k(x, y) = median of XCH4 values in the local window
```

The center pixel is excluded from the local window statistics.

The local anomaly is:

```text
Delta XCH4_k(x, y) = XCH4(x, y) - B_k(x, y)
```

The local sigma is calculated after capping local-window values above the
local 95th percentile. The standardized enhancement is:

```text
Z_k(x, y) = Delta XCH4_k(x, y) / max(sigma_k(x, y), 0.1)
```

The binary enhancement mask is:

```text
enhancement_ge2 = 1 when Z_k >= 2.0, else 0
```

## Kernel and Pixel Geometry

Configured kernel widths:

```text
1, 2, 3, 4, 5, 6 degrees
```

The local processor assumes:

```text
PIXEL_DEG = 0.01
radius_px = round((kernel_deg / 2) / PIXEL_DEG)
window size = 2 * radius_px + 1
```

Thus a 5-degree kernel corresponds to an approximate radius of 250 pixels and
a 501-by-501 local window. The input raster grid should be consistent with
the assumed 0.01-degree pixel spacing for the kernel labels to match their
intended spatial scale.

## Validity Rules

The local processor uses:

```text
MIN_VALID_FRAC = 0.25
MIN_KEPT_FRAC_AFTER_TRIM = 0.10
SIGMA_MIN = 0.1
```

For each output pixel:

- the center XCH4 value must be finite;
- at least 25 percent of non-center local-window pixels must be finite;
- after p95 trimming, at least 10 percent must remain finite;
- background and sigma must be finite.

Pixels failing these tests become NoData in float outputs and zero in the
binary mask.

## Tiled PyTorch Processor

The local processor reads the monthly raster in tiles with a halo equal to
the kernel radius. It uses PyTorch `unfold()` to build local-window patches.

Output arrays are assembled at full raster size and written as GeoTIFFs:

```text
anomaly_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
sigma_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
enhancementZ_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
enhancement_ge2_YYYY_MM_Kdeg_CONUSbuf2p5deg.tif
```

Float outputs use:

```text
nodata = -9999.0
dtype = float32
compression = deflate
```

Binary mask outputs use:

```text
nodata = 0
dtype = uint8
1 = Z >= threshold
0 = NoData or not enhanced
```

Because `0` combines NoData and non-enhanced cells in the binary mask, use the
continuous `enhancementZ` raster and QA outputs when the distinction matters.

## Carbon Mapper Case Filtering

The notebook reads a Carbon Mapper plume-event CSV and searches for common
longitude and latitude column names. Timestamp is optional if the file is
already pre-filtered. If a timestamp exists, records are filtered to the case
year and month. If a `gas` column exists, records are filtered to CH4.

The case bounding box is applied after timestamp and gas filtering:

```text
[min_lon, min_lat, max_lon, max_lat]
```

The default case is Visalia, October 2024:

```text
[-120.1, 35.9, -118.8, 36.9]
```

The filtered coordinates are stored as `_lon` and `_lat`.

## Point Sampling

For every configured kernel, the notebook samples:

- `enhancement_ge2` binary mask;
- continuous `enhancementZ`.

Sampling uses the raster index for each plume-event longitude/latitude. It
does not buffer the point, search neighboring cells, or sample the full plume
geometry.

Kernel summary fields:

| Field | Meaning |
| --- | --- |
| `kernel_deg` | Local-background window width. |
| `intersections` | Number of plume-event points falling on mask value 1. |
| `total_records` | Number of case plume-event points. |
| `share_percent` | `intersections / total_records * 100`. |
| `z_min_at_intersections` | Minimum continuous Z at intersecting points. |
| `z_max_at_intersections` | Maximum continuous Z at intersecting points. |
| `z_range_at_intersections` | Text representation of the Z range. |

## Figures

The notebook provides:

- six-panel kernel mask maps with the same plume-event points overlaid;
- intermediate-product panels showing monthly mean XCH4, anomaly, Z, and the
  threshold mask for one kernel.

The figures are diagnostic and publication-style, but they should be checked
against raster bounds, coordinate ordering, and QA outputs before use.

## QA Checks

Provided helpers inspect:

- CRS;
- affine transform;
- width and height;
- data type;
- nodata value;
- raster bounds;
- valid cell count;
- min/max values.

`compare_raster_grid()` confirms whether selected rasters share the same CRS,
transform, and dimensions.

Recommended QA before reporting results:

- all sampled rasters have identical grids;
- all plume-event coordinates lie inside raster bounds;
- the same point set is sampled for every kernel;
- table counts match plotted points;
- NoData handling is documented for float and binary outputs;
- monthly mean export dates and product versions are recorded.

## Reproducibility Checklist

Record:

- repository commit hash;
- Sentinel-5P collection and selected band;
- Earth Engine project and export task IDs;
- years, months, CONUS definition, and support buffer;
- monthly mean raster filenames and checksums;
- kernel list, `PIXEL_DEG`, tile size, validity thresholds, and `SIGMA_MIN`;
- PyTorch version, CPU/GPU runtime, and memory constraints;
- plume CSV filename and checksum;
- case year, month, and bounding box;
- number of plume-event records sampled;
- kernel overlap summary outputs;
- QA status for raster grids and bounds.

## Known Limitations

- Monthly mean XCH4 removes sub-monthly timing information.
- Local anomaly and Z depend strongly on kernel width and data availability.
- The 0.01-degree pixel assumption should match the input raster grid.
- The support buffer is an approximation based on metres per degree.
- Local median and p95-trimmed sigma are methodological choices.
- PyTorch `unfold()` can be memory intensive for large kernels.
- Binary mask value `0` represents both non-enhanced and NoData cells.
- Point sampling does not represent plume footprint overlap.
- Carbon Mapper plume events are not a continuous source inventory.
- Point-to-mask overlap is not a Sentinel-5P plume detection rate.
