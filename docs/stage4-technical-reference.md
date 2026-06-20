# Stage 4 Technical Reference: CNN Patch-Level Source Classification

This document describes the technical workflow implemented in
`stage4/cnn_patch_classification.ipynb`.

## Objective

Stage 4 estimates whether a monthly raster patch is associated with a known
Carbon Mapper source label. It transforms monthly Sentinel-5P-derived products
into fixed-size patches, trains a PyTorch CNN, and evaluates source-associated
patch probabilities on a held-out temporal test period.

The output should be interpreted as a patch-level classification score under
the constructed labels. It is not direct plume detection, facility
localization, or emission quantification.

## Inputs

The notebook expects a project directory with the following folders:

```text
CNN_point_source/
|-- sources_plumes/
|-- mean_xch4/
|-- anomaly_3deg/
|-- enhancement_3deg/
|-- anomaly_5deg/
|-- enhancement_5deg/
|-- bivariate/
|-- prepared_data/
`-- models/
```

Required source tables:

- `sources_plumes/plumes_with_point_source_name.csv`
- `sources_plumes/matched_point_sources_with_matched_plumes.csv` when
  available

Required raster families:

- monthly mean XCH4 from Stage 3;
- monthly 3deg anomaly rasters from Stage 3;
- monthly 3deg continuous standardized-enhancement rasters from Stage 3;
- monthly 5deg anomaly rasters from Stage 3;
- monthly 5deg continuous standardized-enhancement rasters from Stage 3;
- monthly bivariate rasters from Stage 2.

The notebook discovers rasters from filename patterns containing year and
month tokens. Keep the original Stage 2 and Stage 3 naming conventions unless
the discovery cells are updated at the same time.

## Predictor Channels

Each patch contains five image channels:

```text
mean_xch4
anomaly_3deg
enhancement_3deg
anomaly_5deg
enhancement_5deg
```

The bivariate raster is not passed to the CNN as a predictor. It is used to
screen strict negative examples.

## Label Construction

The notebook converts Carbon Mapper plume/source records into monthly source
labels. Records are filtered and grouped by named source and observation month.
Monthly source locations are rasterized onto the Sentinel-5P grid, and a
Gaussian target surface is created around source locations.

Patch labels are derived from the overlap between non-overlapping 64 x 64
patches and the monthly source target:

- positive patches are source-associated under the monthly target raster;
- negative patches must pass all strict background-screening rules.

## Strict Negative Screening

Strict negative patches are intended to avoid ambiguous background labels.
Default rules include:

- no source-label target in the patch;
- outside known-source exclusion areas;
- bivariate center class equal to 1;
- standardized-enhancement area fraction at or below the configured maximum.

The default enhancement-area threshold is 2.0 standard deviations, and the
default maximum enhancement-area fraction is 0.20.

## Temporal Split

The default split is temporal rather than random:

```text
Training:   2023-01 through 2025-05
Validation: 2025-06 through 2025-08
Testing:    2025-09 through 2025-12
```

This reduces leakage from nearby monthly observations and provides a clearer
held-out evaluation period.

The thesis patch catalogue has the following class distribution:

| Split | Source patches | No-source patches | Total |
| --- | ---: | ---: | ---: |
| Training | 1,058 | 10,872 | 11,930 |
| Validation | 283 | 1,292 | 1,575 |
| Test | 380 | 1,822 | 2,202 |

## Model And Training

The notebook trains a five-branch PyTorch CNN using the five raster channels.
Each channel is processed by an independent convolutional branch, the five
64-element branch outputs are concatenated into a 320-element feature vector,
and a fully connected classifier head produces one source-associated patch
logit. Default training settings include:

- patch size: 64 x 64 pixels;
- batch size: 32;
- maximum epochs: 100;
- early-stopping patience: 8 epochs;
- optimizer: AdamW;
- learning rate: 1e-4;
- weight decay: 2e-2;
- base channels: 16;
- dropout: 0.4;
- positive-class loss weight: 2.5;
- random seed: 42.

Training writes checkpoints under `models/<experiment>/<run_id>/`. For a new
training run, clear or change `run_id` in the configuration cell so the
notebook does not unintentionally reuse a previous run folder.

## Evaluation

The workflow evaluates validation and test predictions using thresholded
binary metrics and probability-based diagnostics. The thesis run selected a
validation probability threshold of 0.38.

On the independent September-December 2025 test set, the thesis reports:

| Metric | Value |
| --- | ---: |
| Positive-class prevalence baseline | 0.173 |
| AUPRC | 0.648 |
| AUROC | 0.814 |
| Precision at threshold 0.38 | 0.643 |
| Recall at threshold 0.38 | 0.526 |
| F1 at threshold 0.38 | 0.579 |

Main evaluation products include:

- prediction CSVs;
- overall metrics;
- monthly metrics;
- confusion matrices;
- monthly confusion maps for held-out test patches;
- optional wall-to-wall source-probability maps.

## Reproducibility Checklist

Record the following with any reported result:

- repository commit hash;
- source CSV filenames and checksums;
- raster filenames and checksums;
- date range and temporal split;
- patch size and grid alignment;
- source-label construction rules;
- strict negative-screening rules;
- channel normalization statistics;
- model architecture and training settings;
- selected checkpoint and probability threshold;
- software versions and hardware type;
- missing months, skipped rasters, or excluded records.

## Limitations

- Carbon Mapper source labels are not a complete census of emission sources.
- Sentinel-5P pixels are much coarser than Carbon Mapper plume observations.
- Monthly aggregation can mix transport, persistence, and sampling effects.
- Negative labels are screened, not guaranteed source-free.
- The model estimates source-associated patch probability under the training
  labels and should not be interpreted as direct facility identification.
