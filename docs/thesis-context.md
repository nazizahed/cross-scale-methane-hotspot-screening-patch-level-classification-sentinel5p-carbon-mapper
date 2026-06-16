# Thesis Context And Literature Alignment

This repository accompanies the thesis:

**Machine Learning Fusion of Hyperspectral-derived and Sentinel-5P Data for
Greenhouse Gas and Air Pollution Mapping**
Sadra Zahed Kachaee, M.Sc. Geoinformatics Engineering, Academic Year
2025-2026.

The repository is the executable companion to the thesis. It does not replace
the written methodology, discussion, or bibliography; it exposes the notebooks
and scripts used to produce the thesis workflows.

## Research Question

The thesis asks how regional Sentinel-5P/TROPOMI methane rasters and
plume-resolving Carbon Mapper observations can be integrated in a technically
transparent geospatial workflow without obscuring their different meanings, and
whether their combined monthly spatial patterns can support source-associated
patch classification.

The main research question is:

```text
To what extent can monthly Sentinel-5P/TROPOMI concentration indicators provide
meaningful regional context for Carbon Mapper plume-event records, which
geospatial processing choices control that interpretation, and how effectively
can multi-scale raster patches distinguish months and locations associated with
Carbon Mapper sources observed during those months?
```

## Scientific Position

The thesis treats methane observation as a geospatial data-integration problem.
The central distinction is measurement meaning:

- Sentinel-5P/TROPOMI provides repeated regional XCH4 concentration fields in
  ppb.
- Carbon Mapper provides discrete plume-event records, source coordinates,
  plume masks, column-enhancement rasters, and reported emission-rate estimates.
- Sentinel-5P monthly products provide context and screening information, not
  facility-level plume detection by themselves.
- Carbon Mapper plume-event records are observations of specific acquisition
  events, not a complete spatial or temporal inventory.
- CNN predictions are source-associated patch probabilities under constructed
  labels, not plume segmentation, source localization, or emission-rate
  retrieval.

This distinction should be preserved in all downstream reuse of the repository.

## Literature Framing

The thesis literature review is organized around six themes.

### Measurement Concepts

Methane emission rate, atmospheric concentration, enhancement, anomaly,
exceedance, plume, hotspot, and source label are different objects. The code
therefore keeps units, raster masks, timestamps, coordinate reference systems,
and label provenance explicit.

### Sentinel-5P Regional Screening

Sentinel-5P/TROPOMI provides broad and repeated methane observation. The
thesis uses the Earth Engine `COPERNICUS/S5P/OFFL/L3_CH4` collection and the
bias-corrected dry-air methane column mixing ratio band. The Earth Engine
Level 3 grid spacing is 0.01 degrees, but the underlying methane retrieval
support is much coarser; the grid must not be described as independent 1 km
methane sensing.

Related literature includes regional anomaly and persistence methods, model-
referenced anomaly detection, spectral time-series anomaly detection, and
automated plume screening from satellite scenes.

### Super-Emitters And Plume-Resolving Systems

High-emitting point sources can contribute a disproportionate share of observed
methane emissions, but detectability depends on emission rate, wind, surface
brightness, retrieval precision, acquisition geometry, and source activity.
Plume-resolving systems such as AVIRIS-3, Global Airborne Observatory, EMIT,
Tanager, GHGSat, and PRISMA provide higher-resolution evidence for plume
geometry, source attribution, and emission-rate estimation.

### Carbon Mapper Data Architecture

Carbon Mapper provides a public catalogue with plume-event attributes and
links to raster assets. The thesis distinguishes:

- a plume-event record: one acquisition-specific observation;
- a named source: an attributed geographic source that may have multiple
  plume-event records;
- a plume mask or concentration raster: linked geospatial raster evidence;
- a reported emission-rate estimate: a source-rate product with uncertainty,
  not time-integrated emitted mass.

### Raster-Vector Interoperability

The work explicitly records CRS, affine transforms, NoData semantics, reducer
scale, temporal support, and class definitions. Point-to-raster sampling is a
contextual association between a plume/source coordinate and a monthly raster
cell; it is not proof of same-overpass detection.

### CNN Patch Classification

The CNN stage is framed as patch-level prioritization for imbalanced data. It
uses precision, recall, F1, AUPRC, and AUROC rather than accuracy alone. AUPRC
is interpreted relative to the positive-class prevalence baseline.

## Repository Workflow Mapping

| Thesis component | Repository stage | Purpose |
| --- | --- | --- |
| Carbon Mapper data discovery and export | Stage 0 | Query, filter, map, summarize, and export Carbon Mapper plume-event records. |
| Event-level Tanager/Sentinel-5P inspection | Stage 1 | Rank strong Tanager events, associate nearby events, and inspect daily Sentinel-5P context. |
| Monthly bivariate Sentinel-5P products | Stage 2 | Build 36 monthly valid-count/exceedance-count rasters and sample Carbon Mapper plume-event records. |
| Local anomaly and enhancement analysis | Stage 3 | Estimate local background and standardized enhancement at multiple spatial scales. |
| CNN patch-level source classification | Stage 4 | Train and test a five-channel CNN for source-associated monthly patch probability. |

## Key Thesis Parameters

| Topic | Thesis value |
| --- | --- |
| Study area | Conterminous United States, 48 states plus District of Columbia |
| Main monthly archive | January 2023 through December 2025 |
| Sentinel-5P product | `COPERNICUS/S5P/OFFL/L3_CH4` |
| Sentinel-5P methane band | `CH4_column_volume_mixing_ratio_dry_air_bias_corrected` |
| Monthly export grid | EPSG:4326, 0.01 degree spacing |
| Bivariate threshold | XCH4 > 1900 ppb |
| Bivariate NoData code | 0 |
| Bivariate NoEx codes | 21-24 |
| Local anomaly kernels | 1 through 6 degree moving windows |
| Enhancement criterion | Z >= 2 |
| CNN patch size | 64 x 64 cells |
| CNN channels | mean XCH4, 3deg anomaly, 3deg enhancement, 5deg anomaly, 5deg enhancement |
| CNN temporal split | train Jan 2023-May 2025, validation Jun-Aug 2025, test Sep-Dec 2025 |
| CNN threshold | validation-selected probability threshold 0.38 |

## Key Thesis Results

The thesis reports the following main results:

- 5,261 Carbon Mapper methane plume-event records were used in the monthly
  raster-vector analysis.
- 4,869 records had classified monthly raster support.
- 165 classified records fell in valid NoEx cells and represented 3.5% of the
  classified rate-estimate sum.
- 4,704 classified records fell in cells with at least one monthly threshold
  exceedance and represented 96.5% of the classified rate-estimate sum.
- In the October 2024 Visalia kernel test, the number of six Carbon Mapper
  plume-event records intersecting the Z >= 2 mask varied from two or three
  for smaller/intermediate windows to all six for the 5deg and 6deg windows.
- On the independent September-December 2025 CNN test set, AUPRC was 0.648
  versus a prevalence baseline of 0.173, and AUROC was 0.814.
- At threshold 0.38, the CNN reached precision 0.643, recall 0.526, and
  F1 0.579.

These results support regional context and patch prioritization. They should
not be reported as facility detection probabilities, causal source attribution,
or regional emitted mass estimates.

## Interpretation Boundaries

Use the following language when describing this repository:

- Prefer "regional methane screening" over "facility detection" for
  Sentinel-5P products.
- Prefer "plume-event record" over "source" when referring to individual
  Carbon Mapper catalogue rows.
- Prefer "source-associated patch probability" over "plume probability" for
  Stage 4 CNN outputs.
- Describe rate sums as sums of reported rate estimates, not integrated
  emissions.
- Keep NoData separate from valid zero-exceedance cells.
- Treat the 0.01 degree grid as a processing lattice, not native TROPOMI
  spatial resolution.

## Future Work From The Thesis

The thesis identifies the following extensions:

- adaptive rather than fixed methane exceedance thresholds;
- explicit meteorological filtering and transport-aware matching;
- larger event-level co-visibility experiments;
- local anomaly testing across more regions and sectors;
- source-grouped, spatially blocked, and external CNN evaluation;
- probability calibration and model/channel ablation studies;
- metric-grid reprojection for physically consistent CNN patch dimensions;
- expanded dashboard filters and GIS exports;
- extension to additional plume-resolving instruments and future missions;
- comparison with inventories and mitigation outcomes.

## Bibliography Used By The Thesis

The following references are the thesis bibliography entries most directly tied
to this repository's software workflows.

1. Barre et al. (2021). Systematic detection of local CH4 anomalies by
   combining satellite measurements with high-resolution forecasts.
   `doi:10.5194/acp-21-5117-2021`
2. Bhardwaj et al. (2022). Evaluating the detectability of methane point
   sources from satellite observing systems using microscale modeling.
   `doi:10.1038/s41598-022-20567-z`
3. Bruno et al. (2024). U-Plume: Automated algorithm for plume detection and
   source quantification by satellite point-source imagers.
   `doi:10.5194/amt-17-2625-2024`
4. Carbon Mapper (2025). Product guide: Data definition and specification.
5. Carbon Mapper (2026). Level 3 and level 4 algorithm theoretical basis
   document.
6. Copernicus Space Component (2026). Sentinel-5P mission and TROPOMI
   instrument.
7. Crosman (2021). Meteorological drivers of Permian Basin methane anomalies
   derived from TROPOMI. `doi:10.3390/rs13050896`
8. Cusworth et al. (2019). Potential of next-generation imaging spectrometers
   to detect and quantify methane point sources from space.
   `doi:10.5194/amt-12-5655-2019`
9. Cusworth et al. (2022). Strong methane point sources contribute a
   disproportionate fraction of total emissions across multiple basins in the
   United States. `doi:10.1073/pnas.2202338119`
10. Duren et al. (2025). The Carbon Mapper emissions monitoring system.
    `doi:10.5194/amt-18-6933-2025`
11. Google Earth Engine Data Catalog (2026). Sentinel-5P OFFL CH4: Offline
    methane.
12. Gorelick et al. (2017). Google Earth Engine: Planetary-scale geospatial
    analysis for everyone. `doi:10.1016/j.rse.2017.06.031`
13. Jongaramrungruang et al. (2019). Towards accurate methane point-source
    quantification from high-resolution 2-D plume imagery.
    `doi:10.5194/amt-12-6667-2019`
14. Lauvaux et al. (2022). Global assessment of oil and gas methane
    ultra-emitters. `doi:10.1126/science.abj4351`
15. LeCun et al. (1998). Gradient-based learning applied to document
    recognition. `doi:10.1109/5.726791`
16. Loshchilov and Hutter (2019). Decoupled weight decay regularization.
17. Ouerghi et al. (2022). Automatic methane plumes detection in time series
    of Sentinel-5P L1B images. `doi:10.5194/isprs-annals-V-3-2022-147-2022`
18. Pandey et al. (2025). Relating multi-scale plume detection and area
    estimates of methane emissions. `doi:10.1021/acs.est.4c07415`
19. Paszke et al. (2019). PyTorch: An imperative style, high-performance deep
    learning library.
20. Pedregosa et al. (2011). Scikit-learn: Machine learning in Python.
21. Saito and Rehmsmeier (2015). The precision-recall plot is more informative
    than the ROC plot when evaluating binary classifiers on imbalanced
    datasets. `doi:10.1371/journal.pone.0118432`
22. Schuit et al. (2023). Automated detection and monitoring of methane
    super-emitters using satellite data. `doi:10.5194/acp-23-9071-2023`
23. Vanselow et al. (2024). Automated detection of regions with persistently
    enhanced methane concentrations using Sentinel-5 Precursor satellite data.
    `doi:10.5194/acp-24-10441-2024`
24. Varon et al. (2018). Quantifying methane point sources from fine-scale
    satellite observations of atmospheric methane plumes.
    `doi:10.5194/amt-11-5673-2018`
25. Wang et al. (2023). Toward a versatile spaceborne architecture for
    immediate monitoring of the global methane pledge.
    `doi:10.5194/acp-23-5233-2023`
