# Thesis Context and Literature Alignment

This repository accompanies the final revised thesis:

**Cross-Scale Methane Hotspot Screening and Patch-Level Source-Associated
Classification with Sentinel-5P and Carbon Mapper**

Sadra Zahed Kachaee, M.Sc. Geoinformatics Engineering, Academic Year
2025-2026.

## Research Scope

The thesis develops a reproducible geospatial framework for integrating two
observation systems with different measurement supports:

- Sentinel-5P/TROPOMI supplies repeated regional fields of column-averaged
  dry-air methane mole fraction (XCH4).
- Carbon Mapper supplies plume-event catalogue records and linked
  high-resolution plume and concentration-enhancement products.

The comparison is contextual rather than one-to-one. A monthly Sentinel-5P
cell aggregates retrievals over broad spatial and temporal support, whereas a
Carbon Mapper record describes a discrete plume observation. The integration
therefore preserves differences in units, acquisition time, spatial support,
data model, and retrieval purpose.

## Research Question

> To what extent can monthly Sentinel-5P/TROPOMI concentration indicators
> provide meaningful regional context for Carbon Mapper plume-event records,
> which geospatial processing choices control that interpretation, and how
> effectively can multi-scale raster patches distinguish months and locations
> associated with Carbon Mapper sources observed during those months?

## Objectives

1. Characterize the relevant Sentinel-5P and Carbon Mapper products,
   including their sensors, processing levels, spatial and temporal
   properties, units, coordinate systems, data structures, and limitations.
2. Implement a documented workflow for querying, filtering, harmonizing,
   mapping, and exporting observations through the Carbon Mapper REST API and
   Google Earth Engine.
3. Evaluate whether regional Sentinel-5P concentration patterns provide useful
   context for independent Carbon Mapper plume-event records.
4. Quantify the effects of observation support, temporal aggregation,
   fixed-threshold choices, point-to-pixel sampling, and local-background
   scale.
5. Construct and evaluate a CNN that classifies monthly
   Sentinel-5P-derived patches with temporally separated training, validation,
   and test periods.
6. Distinguish regional screening and patch classification from plume
   detection, source localization, and emission-rate estimation.

## Literature Alignment

### Regional screening and plume-resolving observation

Sentinel-5P provides broad, repeated methane coverage suitable for regional
screening. During the 2023-2025 study period, its nominal Level 2 methane
footprint was approximately 5.5 km across track by 7 km along track. Earth
Engine bins those retrievals to a 0.01 degree Level 3 grid; the finer grid is a
processing lattice and does not create independent 1 km measurements.

Regional anomaly methods identify departures from a local or modeled
background. Their output depends on retrieval support, background definition,
meteorology, and filter scale. Automated TROPOMI screening can identify broad
plume-like structures or persistent enhancements, but it does not provide the
facility-scale detail of an imaging spectrometer.

Carbon Mapper observations complement regional screening with localized plume
geometry, source-oriented catalogue attributes, and reported emission-rate
estimates. Their targeted acquisition pattern is not a complete source census,
and repeated catalogue rows may refer to the same named source.

### Multi-channel raster fusion and imperfect labels

Stage 4 is patch classification, not semantic segmentation. Its five aligned
channels are transformations of the same monthly Sentinel-5P field: mean XCH4,
3-degree anomaly, 3-degree standardized enhancement, 5-degree anomaly, and
5-degree standardized enhancement. The channels provide multiple spatial
views rather than independent sensor measurements.

The Carbon Mapper-derived labels are catalogue-supported and incomplete.
Positive labels identify source-associated monthly patches; conservative
negative labels indicate patches that pass explicit exclusion rules, not proof
that no methane source exists. The natural class imbalance is retained.
Training uses a weighted loss, and evaluation emphasizes precision, recall,
F1, AUPRC, and AUROC rather than accuracy alone. AUPRC is compared with the
positive-class prevalence baseline.

The chronological split tests future months. It is not a strict unseen-source
test because a physical source may occur in more than one month. The thesis
therefore identifies source-grouped, spatially blocked, and external-region
evaluation as future work.

## Repository Workflow Mapping

| Thesis component | Repository stage | Purpose |
| --- | --- | --- |
| Carbon Mapper data discovery and export | Stage 0 | Query, filter, map, summarize, and export plume-event records. |
| Event-level Tanager/Sentinel-5P inspection | Stage 1 | Rank Tanager events, associate nearby observations, and inspect daily Sentinel-5P context. |
| Monthly bivariate Sentinel-5P products | Stage 2 | Build monthly valid-count and exceedance-count rasters and sample plume events. |
| Local anomaly and enhancement analysis | Stage 3 | Estimate local background and standardized enhancement at multiple scales. |
| CNN patch-level classification | Stage 4 | Evaluate source-associated monthly patch probability from five raster channels. |

## Key Parameters

| Topic | Thesis value |
| --- | --- |
| Study area | CONUS: 48 states plus the District of Columbia |
| Main monthly archive | January 2023 through December 2025 |
| Sentinel-5P product | `COPERNICUS/S5P/OFFL/L3_CH4` |
| Methane band | `CH4_column_volume_mixing_ratio_dry_air_bias_corrected` |
| Native retrieval support | Approximately 5.5 km across track by 7 km along track |
| Monthly export grid | EPSG:4326, 0.01 degree spacing |
| Bivariate threshold | XCH4 > 1900 ppb |
| Bivariate NoData code | 0 |
| Bivariate NoEx codes | 21-24 |
| Local-background kernels | 1-6 degree moving windows |
| Enhancement criterion | Z >= 2 |
| CNN patch size | 64 x 64 cells |
| CNN channels | Mean XCH4; 3/5 degree anomaly and enhancement |
| CNN temporal split | Train Jan 2023-May 2025; validation Jun-Aug 2025; test Sep-Dec 2025 |
| CNN threshold | Validation-selected probability threshold 0.38 |

## Result Anchors

- The monthly analysis contained 5,261 Carbon Mapper methane plume-event
  records; 4,869 had classified raster support.
- Valid cells with no threshold exceedance contained 165 records and 3.5% of
  the classified reported rate-estimate sum.
- Cells with at least one threshold exceedance contained 4,704 records and
  96.5% of the classified reported rate-estimate sum.
- In the October 2024 Visalia case, overlap of six plume-event records with
  the Z >= 2 mask varied from two or three at smaller/intermediate windows to
  all six at 5 and 6 degrees.
- The independent September-December 2025 test set contained 2,202 patches.
  AUPRC was 0.648 against prevalence 0.173, and AUROC was 0.814.
- At threshold 0.38, precision was 0.643, recall 0.526, and F1 0.579.

These results support regional context and patch prioritization. They are not
facility-detection probabilities, causal source attribution, or regional
emitted-mass estimates.

## Interpretation Boundaries

- Use **regional methane screening** for Sentinel-5P-derived products.
- Use **plume-event record** for an individual Carbon Mapper catalogue row.
- Use **source-associated patch probability** for the Stage 4 output.
- Describe rate totals as sums of reported rate estimates, not integrated
  emissions.
- Keep NoData separate from valid zero-exceedance cells.
- Treat the 0.01 degree grid as a processing lattice, not native TROPOMI
  resolution.

## Future Work

The final thesis identifies adaptive thresholds, meteorological and
transport-aware matching, larger event-level co-visibility studies, broader
regional and sectoral anomaly tests, source-grouped and spatially blocked CNN
evaluation, external validation, probability calibration, model and channel
ablation, metric-grid patch construction, expanded GIS exports, and extension
to additional plume-resolving instruments.

## Thesis Bibliography

1. Audebert, Le Saux, and Lefevre (2018). Beyond RGB: Very high resolution
   urban remote sensing with multimodal deep networks.
   DOI: `10.1016/j.isprsjprs.2017.11.011`.
2. Barre et al. (2021). Systematic detection of local CH4 anomalies by
   combining satellite measurements with high-resolution forecasts.
   DOI: `10.5194/acp-21-5117-2021`.
3. Bhardwaj et al. (2022). Evaluating the detectability of methane point
   sources from satellite observing systems using microscale modeling.
   DOI: `10.1038/s41598-022-20567-z`.
4. Bruno et al. (2024). U-Plume: Automated algorithm for plume detection and
   source quantification by satellite point-source imagers.
   DOI: `10.5194/amt-17-2625-2024`.
5. Buda, Maki, and Mazurowski (2018). A systematic study of the class
   imbalance problem in convolutional neural networks.
   DOI: `10.1016/j.neunet.2018.07.011`.
6. Carbon Mapper (2025). Product guide: Data definition and specification,
   version 1.1.6.
7. Carbon Mapper (2026). Level 3 and level 4 algorithm theoretical basis
   document.
8. Copernicus Space Component (2026). Sentinel-5P mission and TROPOMI
   instrument.
9. Crosman (2021). Meteorological drivers of Permian Basin methane anomalies
   derived from TROPOMI. DOI: `10.3390/rs13050896`.
10. Cusworth et al. (2019). Potential of next-generation imaging
    spectrometers to detect and quantify methane point sources from space.
    DOI: `10.5194/amt-12-5655-2019`.
11. Cusworth et al. (2022). Strong methane point sources contribute a
    disproportionate fraction of total emissions across multiple U.S. basins.
    DOI: `10.1073/pnas.2202338119`.
12. Duren et al. (2025). The Carbon Mapper emissions monitoring system.
    DOI: `10.5194/amt-18-6933-2025`.
13. Google Earth Engine Data Catalog (2026). Sentinel-5P OFFL CH4: Offline
    methane.
14. Gorelick et al. (2017). Google Earth Engine: Planetary-scale geospatial
    analysis for everyone. DOI: `10.1016/j.rse.2017.06.031`.
15. Jongaramrungruang et al. (2019). Towards accurate methane point-source
    quantification from high-resolution 2-D plume imagery.
    DOI: `10.5194/amt-12-6667-2019`.
16. Kaiser et al. (2017). Learning aerial image segmentation from online
    maps. DOI: `10.1109/TGRS.2017.2719738`.
17. Kussul et al. (2017). Deep learning classification of land cover and crop
    types using remote sensing data. DOI: `10.1109/LGRS.2017.2681128`.
18. Lauvaux et al. (2022). Global assessment of oil and gas methane
    ultra-emitters. DOI: `10.1126/science.abj4351`.
19. LeCun et al. (1998). Gradient-based learning applied to document
    recognition. DOI: `10.1109/5.726791`.
20. Loshchilov and Hutter (2019). Decoupled weight decay regularization.
21. Ouerghi et al. (2022). Automatic methane plumes detection in time series
    of Sentinel-5P L1B images.
    DOI: `10.5194/isprs-annals-V-3-2022-147-2022`.
22. Pandey et al. (2025). Relating multi-scale plume detection and area
    estimates of methane emissions. DOI: `10.1021/acs.est.4c07415`.
23. Paszke et al. (2019). PyTorch: An imperative style, high-performance deep
    learning library.
24. Pedregosa et al. (2011). Scikit-learn: Machine learning in Python.
25. Roberts et al. (2017). Cross-validation strategies for data with
    temporal, spatial, hierarchical, or phylogenetic structure.
    DOI: `10.1111/ecog.02881`.
26. Saito and Rehmsmeier (2015). The precision-recall plot is more
    informative than the ROC plot for imbalanced binary classifiers.
    DOI: `10.1371/journal.pone.0118432`.
27. Schuit et al. (2023). Automated detection and monitoring of methane
    super-emitters using satellite data. DOI: `10.5194/acp-23-9071-2023`.
28. Vanselow et al. (2024). Automated detection of regions with persistently
    enhanced methane concentrations using Sentinel-5 Precursor satellite
    data. DOI: `10.5194/acp-24-10441-2024`.
29. Varon et al. (2018). Quantifying methane point sources from fine-scale
    satellite observations of atmospheric methane plumes.
    DOI: `10.5194/amt-11-5673-2018`.
30. Wang et al. (2023). Toward a versatile spaceborne architecture for
    immediate monitoring of the global methane pledge.
    DOI: `10.5194/acp-23-5233-2023`.
