# Cross-Scale Methane Hotspot Screening and Patch-Level Source-Associated Classification with Sentinel-5P and Carbon Mapper

This repository contains the reproducible software workflows developed for a
thesis on integrating regional Sentinel-5P methane fields with plume-resolving
Carbon Mapper observations without conflating their different measurement
supports and product meanings.

It accompanies the thesis **Cross-Scale Methane Hotspot Screening and
Patch-Level Source-Associated Classification with Sentinel-5P and Carbon
Mapper** by Sadra Zahed Kachaee, M.Sc. Geoinformatics Engineering, Academic
Year 2025-2026.

The current implementation focuses on Carbon Mapper/Tanager methane plume
data and Sentinel-5P XCH4. It provides data access, plume prioritization,
cross-sensor temporal analysis, monthly bivariate mapping, local anomaly
enhancement, plume-to-raster matching, and CNN patch-level source
classification.

The central thesis question is whether monthly Sentinel-5P/TROPOMI
concentration indicators can provide meaningful regional context for Carbon
Mapper plume-event records, which geospatial processing choices control that
interpretation, and how effectively multi-scale monthly raster patches can
distinguish locations and months associated with observed Carbon Mapper
sources.

For the literature framing, thesis assumptions, key numerical results, and
interpretation limits, see [Thesis context and literature alignment](docs/thesis-context.md).
Large thesis data products are distributed separately through Google Drive;
see [Data availability](docs/data-availability.md) and
[data-manifest.csv](docs/data-manifest.csv).

## Research Workflow

**Stage 0: Carbon Mapper data access**

Produces the shared Carbon Mapper plume CSV.

**Stage 0 output feeds Stage 1: Cross-sensor visibility**

Produces ranked Tanager plume events and daily Sentinel-5P context.

**Stage 0 output feeds Stage 2: Monthly bivariate matching**

Produces monthly Sentinel-5P classes and plume-level class summaries.

**Stage 0 output feeds Stage 3: Local anomaly and enhancement**

Produces local-background anomaly products, standardized enhancement masks,
and kernel-sensitivity plume-overlap summaries.

**Stage 2 and Stage 3 outputs feed Stage 4: CNN patch classification**

Produces source-associated patch probabilities, model checkpoints, evaluation
metrics, and held-out monthly confusion maps.

## Implemented Stages

| Stage | Purpose | Main entry point | Documentation |
| --- | --- | --- | --- |
| **Stage 0** | Query, inspect, map, and export Carbon Mapper CH4/CO2 plume records. | `stage0/carbon_mapper_dashboard.ipynb` | [User guide](stage0/README.md), [technical reference](docs/stage0-technical-reference.md) |
| **Stage 1** | Rank Tanager CH4 plumes, associate nearby events, and compare them with daily Sentinel-5P context. | `stage1/cross_sensor_visibility.ipynb` | [User guide](stage1/README.md), [technical reference](docs/stage1-technical-reference.md) |
| **Stage 2** | Create monthly Sentinel-5P observation/exceedance classes and match Carbon Mapper plumes to them. | `stage2/monthly_bivariate_mapping.ipynb` | [User guide](stage2/README.md), [technical reference](docs/stage2-technical-reference.md) |
| **Stage 3** | Create local-background anomaly and standardized-enhancement masks, then test plume-event overlap across kernels. | `stage3/local_anomaly_enhancement.ipynb` | [User guide](stage3/README.md), [technical reference](docs/stage3-technical-reference.md) |
| **Stage 4** | Train and evaluate a CNN that classifies monthly raster patches as source-associated or background-like. | `stage4/cnn_patch_classification.ipynb` | [User guide](stage4/README.md), [technical reference](docs/stage4-technical-reference.md) |

## Thesis Chapter Mapping

| Thesis chapter | Repository content |
| --- | --- |
| Chapter 1: Background and literature context | [Thesis context and literature alignment](docs/thesis-context.md) |
| Chapter 2: Data and study design | Stage guides plus stage-specific technical references |
| Chapter 3: Methodology | Stage 0 through Stage 4 notebooks and technical references |
| Chapter 4: Results | Generated CSV, GeoTIFF, HTML, checkpoint, prediction, and figure outputs stored outside Git |
| Chapter 5: Discussion | Interpretation boundaries documented in this README and `docs/thesis-context.md` |
| Chapter 6: Conclusions and future work | Future-work notes in `docs/thesis-context.md` |

## Repository Structure

```text
.
|-- stage0/
|   |-- README.md
|   |-- carbon_mapper_dashboard.py
|   `-- carbon_mapper_dashboard.ipynb
|-- stage1/
|   |-- README.md
|   `-- cross_sensor_visibility.ipynb
|-- stage2/
|   |-- README.md
|   `-- monthly_bivariate_mapping.ipynb
|-- stage3/
|   |-- README.md
|   |-- data/
|   |   `-- README.md
|   `-- local_anomaly_enhancement.ipynb
|-- stage4/
|   |-- README.md
|   |-- data/
|   |   `-- README.md
|   |-- outputs/
|   |   `-- README.md
|   `-- cnn_patch_classification.ipynb
|-- docs/
|   |-- thesis-context.md
|   |-- stage0-technical-reference.md
|   |-- stage1-technical-reference.md
|   |-- stage2-technical-reference.md
|   |-- stage3-technical-reference.md
|   `-- stage4-technical-reference.md
|-- .gitignore
|-- README.md
`-- requirements.txt
```

Generated datasets, rasters, figures, and maps are intentionally excluded
from Git. Each stage README describes its expected inputs and outputs. The
top-level `Data/` folder is ignored and should be restored from the external
Google Drive dataset when needed.

## Installation

Python 3.10-3.12 is recommended.

### Windows PowerShell

```powershell
git clone https://github.com/nazizahed/thesis-ml-fusion-hyperspectral-sentinel5p-ghg-air-pollution-mapping.git
cd thesis-ml-fusion-hyperspectral-sentinel5p-ghg-air-pollution-mapping
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### macOS or Linux

```bash
git clone https://github.com/nazizahed/thesis-ml-fusion-hyperspectral-sentinel5p-ghg-air-pollution-mapping.git
cd thesis-ml-fusion-hyperspectral-sentinel5p-ghg-air-pollution-mapping
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Geospatial packages such as GeoPandas and Rasterio depend on compiled
libraries. A Conda environment may be easier on systems where `pip`
installation fails.

## External Access

The workflows require some external services:

- **Stage 0:** a Carbon Mapper API token.
- **Stage 1:** Carbon Mapper raster access and Google Earth Engine.
- **Stage 2:** Google Earth Engine and Google Drive for monthly GeoTIFF
  exports.
- **Stage 3:** Google Earth Engine, Google Drive, Carbon Mapper plume-event
  CSVs, and enough CPU/GPU memory for local raster processing.
- **Stage 4:** Stage 2 and Stage 3 raster products, Carbon Mapper source-label
  CSVs, and a PyTorch environment. A GPU is recommended for model training.

No API token, password, or Earth Engine credential is stored in this
repository. Follow the authentication instructions in the relevant stage
guide.

## Running a Stage

Launch the required notebook from the repository root:

```bash
# Stage 0
jupyter lab stage0/carbon_mapper_dashboard.ipynb

# Stage 1
jupyter lab stage1/cross_sensor_visibility.ipynb

# Stage 2
jupyter lab stage2/monthly_bivariate_mapping.ipynb

# Stage 3
jupyter lab stage3/local_anomaly_enhancement.ipynb

# Stage 4
jupyter lab stage4/cnn_patch_classification.ipynb
```

Run only one command at a time. Configure the paths, credentials, Earth
Engine project, dates, and execution switches described in that stage's
README before running all cells.

## Data Handoff

Stage 0 exports Carbon Mapper plume CSV files. These CSVs are the principal
input to Stages 1 and 2.

Typical shared fields include:

```text
plume_latitude
plume_longitude
datetime
gas
platform
plume_tif
con_tif
emission_auto
ipcc_sector
instrument
```

Stage 1 requires the linked plume and concentration rasters for mask-based
statistics. Stage 2 primarily requires coordinates, timestamps, and gas, with
emission, sector, and instrument fields used for optional summaries. Stage 3
uses plume-event coordinates and timestamps for point-to-enhancement-mask
kernel sensitivity tests. Stage 4 uses Stage 2 bivariate rasters, Stage 3
monthly mean XCH4/anomaly/enhancement rasters, and Carbon Mapper named-source
labels to build a patch catalogue for CNN classification.

## Scientific Scope

The workflows support exploratory and reproducible cross-sensor analysis.
They should not be presented as direct validation of an individual
high-resolution plume by Sentinel-5P. The thesis positions Sentinel-5P as a
regional screening and context layer, Carbon Mapper as plume-resolving
source-oriented evidence, and Stage 4 as source-associated patch
prioritization under constructed labels.

Important distinctions:

- Carbon Mapper/Tanager and Sentinel-5P have different spatial resolutions,
  sampling times, retrieval methods, and measurement support.
- The 0.01 degree Sentinel-5P Level 3 grid is a processing lattice and should
  not be described as independent 1 km methane sensing.
- A plume catalogue is not a complete census of emission sources.
- A Carbon Mapper catalogue row is a plume-event record; it is not the same
  object as a named source that may have several event records.
- Plume counts do not necessarily represent unique facilities or independent
  emission events.
- Stage 1 comparisons represent coarse temporal and spatial co-visibility.
- Stage 2 classes represent monthly Sentinel-5P context at plume point
  locations.
- Stage 3 anomaly masks represent local-background enhancement sensitivity,
  not direct plume detection.
- Stage 4 probabilities represent source-associated monthly patches under
  constructed labels, not plume detection, facility localization, or emission
  quantification.
- Any emission values retain the units and limitations of their source
  fields unless a stage explicitly documents a conversion.

## Reproducibility

For thesis results, record:

- repository commit hash;
- input filenames and checksums;
- query or study dates;
- geographic domain;
- algorithm parameters and thresholds;
- Earth Engine collection, band, project, and export settings;
- software package versions;
- missing data, failed raster operations, and excluded records.

The stage-specific technical references provide more detailed checklists and
methodological limitations.

## Thesis Result Anchors

The thesis reports the following headline results for this implementation:

- 4,869 of 5,261 Carbon Mapper methane plume-event records had classified
  monthly Sentinel-5P raster support.
- 96.5% of the classified reported rate-estimate sum occurred in cells with
  at least one monthly threshold exceedance.
- The Stage 4 CNN reached test AUPRC 0.648 versus a prevalence baseline of
  0.173, AUROC 0.814, precision 0.643, recall 0.526, and F1 0.579 at the
  validation-selected threshold of 0.38.

These numbers describe the sampled thesis workflows and constructed labels.
They are not facility detection probabilities or regional emitted-mass
estimates.

## Data Attribution

Users are responsible for complying with the access, licensing, citation, and
attribution requirements of Carbon Mapper, Sentinel-5P/Copernicus, Google
Earth Engine, Natural Earth, TIGER/Line, and any other source data used in an
analysis.
