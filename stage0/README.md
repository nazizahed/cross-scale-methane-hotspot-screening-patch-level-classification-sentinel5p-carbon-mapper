# Stage 0: Carbon Mapper Interactive Dashboard

Stage 0 is an interactive Jupyter dashboard for querying, exploring,
summarizing, mapping, and exporting Carbon Mapper methane (CH4) and carbon
dioxide (CO2) plume observations.

It is the data-access and exploratory-analysis entry point for the thesis
pipeline. The exported plume CSV can be used as input to Stage 1 and Stage 2.
For the thesis-level research framing and literature context, see
[../docs/thesis-context.md](../docs/thesis-context.md).

## Main Files

```text
carbon_mapper_dashboard.py
carbon_mapper_dashboard.ipynb
```

- The Python module implements the API workflow, widgets, statistics, charts,
  maps, and optional plume-area estimation.
- The notebook installs/imports the module, requests the API token, and
  launches the dashboard.

Implementation details are documented in
[../docs/stage0-technical-reference.md](../docs/stage0-technical-reference.md).

## Features

- Query Carbon Mapper plume metadata by country, state/province, date range,
  and gas.
- Preserve the raw API response as CSV.
- Assign plume coordinates to first-level administrative units.
- Summarize plume counts, exploratory plume areas, and available emission
  values by administrative unit.
- Display bar charts, histograms, choropleths, plume points, clusters, and
  heatmaps.
- Export summary tables and interactive Folium maps.

Stage 0 does not perform Sentinel-5P processing or formal emission
validation.

## Requirements

- Python 3.10-3.12 is recommended.
- Packages in the repository-level `requirements.txt`.
- JupyterLab, Jupyter Notebook, or Google Colab.
- A Carbon Mapper API token.
- Internet access to Carbon Mapper, Natural Earth, and linked raster products
  when area estimation is enabled.

Install the shared dependencies from the repository root:

```bash
python -m pip install -r requirements.txt
```

## API Token

No token is stored in the repository. The notebook requests it through a
hidden prompt and keeps it only in the current Python process.

Alternatively, set `CM_TOKEN` before starting Jupyter.

Windows PowerShell:

```powershell
$env:CM_TOKEN = "your-token"
jupyter lab stage0/carbon_mapper_dashboard.ipynb
```

macOS or Linux:

```bash
export CM_TOKEN="your-token"
jupyter lab stage0/carbon_mapper_dashboard.ipynb
```

Do not place a real token in a notebook cell, Python source file, committed
environment file, screenshot, or issue report.

## Running

From the repository root:

```bash
jupyter lab stage0/carbon_mapper_dashboard.ipynb
```

Run the notebook cells from top to bottom through:

```python
app = CMApp(cm_token=os.environ["CM_TOKEN"])
app.display()
```

The installation cell inside the notebook is intended mainly for Colab and
can be skipped when `requirements.txt` is already installed locally.

## Using the Dashboard

1. Select a country.
2. Optionally select one or more states/provinces. Leave the list empty to
   use the whole country.
3. Choose a start and end date.
4. Select CH4, CO2, or both.
5. Keep **Max plumes** small for the first test, such as 500.
6. Leave **Estimate plume areas** disabled for the fastest first run.
7. Click **Fetch & Analyze**.
8. Review the status output, charts, table, and map.

A practical first test is the United States, CH4, a one-month interval, no
state selection, and a 500-row processing limit.

## Controls

| Control | Behavior |
| --- | --- |
| Country | Loads the country and its ADM1 boundaries. |
| States/Prov | Restricts the bounding query and summaries; empty means all. |
| Start / End | Defines the API datetime interval. |
| Gases | Filters downloaded records to CH4 and/or CO2. |
| Max plumes | Caps rows processed after download; it does not limit API transfer. |
| Estimate plume areas | Downloads raster products and calculates exploratory areas. |
| Estimate areas now | Runs area estimation after metadata is loaded. |
| Top N | Controls rows shown in charts and tables. |
| Show points | Displays individual plume markers. |
| Cluster points | Groups nearby markers for map responsiveness. |
| Heatmap | Adds a plume-density layer, weighted by area when available. |
| Max points | Caps rendered markers without changing statistics. |

For multi-select widgets, use `Ctrl` on Windows/Linux or `Command` on macOS
to select or deselect multiple entries.

## Outputs

Raw API responses are written to:

```text
stage0_outputs/plumes_<bbox>_<start>_<end>.csv
```

The output directory is ignored by Git. In Colab, the **Raw CSV** panel can
download a selected file. Locally, it displays the saved absolute path.

After a successful query:

```python
app.export_state_stats(
    path="carbon_mapper_state_stats_ch4.csv",
    gas="CH4",
)

app.save_map(
    path="carbon_mapper_ch4_map.html",
    gas="CH4",
    metric="plume_count",
)
```

Available map metrics depend on the returned data and may include:

```text
plume_count
area_mean_km2
area_sum_km2
emission_total
```

## Stage Handoff

The raw plume CSV can feed both later stages.

- Stage 1 needs plume coordinates, time, gas, platform, `plume_tif`, and
  `con_tif`.
- Stage 2 needs plume coordinates, time, and gas; emission, sector, and
  instrument columns enable additional summaries.

Preserve the original raw CSV and record its checksum for reproducibility.

## Scientific Interpretation

- Plume count means returned catalogue records, not necessarily unique
  facilities or independent events.
- Optional area calculation is a threshold-based exploratory estimate, not an
  official validated Carbon Mapper plume geometry.
- `emission_auto` is aggregated only when available and is not unit-converted.
- Administrative assignment uses the reported plume point and a `within`
  spatial join.
- The API query uses a rectangular bounding polygon, so the raw CSV can
  include records outside an irregular administrative boundary.

## Troubleshooting

**401 Unauthorized**

The token is missing, expired, rejected, or lacks access. Enter a fresh token
and restart the affected notebook cells.

**No plumes for this selection**

Use a wider date interval, another gas, or fewer geographic restrictions, and
check catalogue coverage.

**Could not load ADM1 list**

Check internet access to Natural Earth and verify the country name.

**Downloaded CSV is missing coordinates**

The current Carbon Mapper response schema does not provide the expected
`plume_latitude` and `plume_longitude` columns.

**Area values are empty**

The record may lack a usable `con_tif` or `plume_tif`, the raster may be
unavailable, or raster processing may have failed.

**Widgets do not appear**

Restart the kernel, verify that `ipywidgets` is installed, and rerun the
widget-manager setup cell when using Colab.
