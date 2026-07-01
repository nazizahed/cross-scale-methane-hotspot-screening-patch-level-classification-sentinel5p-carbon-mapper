# Data Availability

Large thesis data products are distributed outside Git through Zenodo. The
repository tracks only code, notebooks, documentation, and a lightweight file
manifest.

## Dataset Location

The official thesis dataset is archived on Zenodo:

[https://doi.org/10.5281/zenodo.21080361](https://doi.org/10.5281/zenodo.21080361)

Dataset citation:

```text
Zahed Kachaee, S., Yordanov, V., & Brovelli, M. A. (2026).
Cross-Scale Methane Hotspot Screening and Patch-Level Source-Associated
Classification with Sentinel-5P and Carbon Mapper (v1.0.0). Zenodo.
https://doi.org/10.5281/zenodo.21080361
```

## Local Folder Layout

After downloading the Zenodo dataset files, place or extract the dataset at
the repository root as:

```text
Data/
`-- Thesis_Data/
    |-- 1-Event-level cross-sensor inspection/
    |-- 2-Monthly bivariate raster workflow/
    |-- 3-Local anomaly and enhancement workflow/
    `-- 4-CNN patch-level source classifcation/
```

The local `Data/` folder is intentionally ignored by Git because it is about
2.08 GB and consists mainly of GeoTIFF rasters.

Approximate local folder sizes:

| Folder | Size |
| --- | ---: |
| `1-Event-level cross-sensor inspection/` | 1.83 MB |
| `2-Monthly bivariate raster workflow/` | 190.87 MB |
| `3-Local anomaly and enhancement workflow/` | 1832.05 MB |
| `4-CNN patch-level source classifcation/` | 108.35 MB |

## Manifest

The file [data-manifest.csv](data-manifest.csv) lists the expected dataset
files with:

- relative path inside `Data/`;
- byte size;
- SHA-256 checksum.

Use the manifest to verify that a downloaded Zenodo copy matches the dataset
used by the thesis repository.

## Verify A Download

From the repository root in Windows PowerShell:

```powershell
$manifest = Import-Csv docs/data-manifest.csv
$missing = @()
$changed = @()

foreach ($row in $manifest) {
    $path = Join-Path "Data" $row.path
    if (-not (Test-Path -LiteralPath $path)) {
        $missing += $row.path
        continue
    }

    $file = Get-Item -LiteralPath $path
    $hash = (Get-FileHash -LiteralPath $path -Algorithm SHA256).Hash.ToLowerInvariant()

    if ($file.Length -ne [int64]$row.bytes -or $hash -ne $row.sha256) {
        $changed += $row.path
    }
}

"Missing files: $($missing.Count)"
"Changed files: $($changed.Count)"
```

Both counts should be zero for an exact copy.

## Why Data Are Not In Git

GitHub blocks regular Git files larger than 100 MiB and warns for files larger
than 50 MiB. Although the largest file in this dataset is below 100 MiB, the
full folder is over 2 GB, which would make the repository slow to clone and
would permanently inflate Git history.

For reproducibility, keep large rasters and generated model products in the
Zenodo thesis dataset or another citable data repository, and keep only
manifest files and access instructions in the Git repository.
