# Stage 4: CNN Patch-Level Source Classification

Stage 4 trains and evaluates a convolutional neural network that classifies
monthly Sentinel-5P-derived raster patches as source-associated or
background-like. It combines the Stage 2 bivariate context with Stage 3 local
anomaly and enhancement products and Carbon Mapper named-source labels.

The model output is a source-associated patch probability. It is not a plume
detection, facility localization, or emission-rate estimate.

## Entry Point

```bash
jupyter lab stage4/stage4_cnn_patch_classification_pipeline.ipynb
```

The notebook was designed for a Google Colab or Jupyter environment with
PyTorch. A GPU is recommended for training, but the preprocessing and
evaluation cells can run on CPU for smaller checks.

## Expected Project Folder

Keep large rasters, model checkpoints, catalogues, and prediction tables
outside this Git repository. The notebook expects a working project folder
similar to:

```text
CNN_point_source/
|-- sources_plumes/
|   |-- plumes_with_point_source_name.csv
|   `-- matched_point_sources_with_matched_plumes.csv
|-- mean_xch4/
|-- anomaly_3deg/
|-- enhancement_3deg/
|-- anomaly_5deg/
|-- enhancement_5deg/
|-- bivariate/
|-- prepared_data/
`-- models/
```

Set one of the following before running the notebook:

```bash
CNN_PROJECT_DIR=/path/to/CNN_point_source
CNN_LOCAL_PROJECT_DIR=/local/scratch/CNN_point_source
```

In Colab, edit `DRIVE_PROJECT_DIR` in the configuration cell if your Google
Drive folder uses a different path. `CNN_LOCAL_PROJECT_DIR` is optional and is
used only to copy Drive inputs into a faster local runtime directory.

## Inputs

Stage 4 expects monthly rasters from earlier stages:

- Stage 2 monthly bivariate class rasters.
- Stage 3 monthly mean XCH4 rasters.
- Stage 3 monthly 3deg local-anomaly rasters.
- Stage 3 monthly 3deg standardized-enhancement rasters.
- Stage 3 monthly 5deg local-anomaly rasters.
- Stage 3 monthly 5deg standardized-enhancement rasters.
- Carbon Mapper plume/source CSVs with source name, timestamp, gas/type, and
  source or plume coordinates.

The model predictor channels are:

```text
mean_xch4
anomaly_3deg
enhancement_3deg
anomaly_5deg
enhancement_5deg
```

The bivariate raster is used for conservative negative-sample screening, not
as a direct CNN input channel.

## Label And Sampling Rules

The notebook builds monthly source-label rasters from Carbon Mapper named
sources observed in each month. Positive patches are non-overlapping 64 x 64
patches associated with the monthly source target.

Strict negative patches must satisfy all configured rules:

- no monthly source target;
- outside the known-source exclusion raster;
- bivariate center class equal to 1;
- low standardized-enhancement area fraction.

The default temporal split is:

```text
Train:      January 2023 through May 2025
Validation: June 2025 through August 2025
Test:       September 2025 through December 2025
```

## Running The Notebook

1. Place the Stage 2 and Stage 3 raster products in the expected folders.
2. Place Carbon Mapper plume/source CSVs in `sources_plumes/`.
3. Open the notebook and review the configuration cell.
4. For a fresh run, set `run_id` to an empty string or a new run name so the
   notebook creates a new model folder.
5. Run the data-discovery and validation cells before training.
6. Train the CNN, or load an existing checkpoint for evaluation.
7. Export metrics, prediction CSVs, confusion maps, and optional wall-to-wall
   classified maps.

The thesis run used patch size 64, random seed 42, positive-class weight 2.5,
dropout 0.4, and a validation-selected probability threshold of 0.38.

## Outputs

Generated outputs include:

- monthly source-label rasters;
- known-source exclusion raster;
- binary patch catalogue;
- channel normalization statistics;
- PyTorch checkpoint files;
- validation and test prediction CSVs;
- overall and monthly metrics;
- monthly confusion maps for held-out patches;
- optional wall-to-wall source-probability maps.

Do not commit these generated outputs to Git. Store them in Google Drive,
Zenodo, institutional storage, or another documented data repository.

## Technical Details

See [Stage 4 technical reference](../docs/stage4-technical-reference.md) for
the method summary, file-matching rules, reproducibility checklist, and
limitations.
