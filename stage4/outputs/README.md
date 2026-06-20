# Stage 4 Output Products

Generated output files are stored outside Git because of their size.

The notebook can generate:

- monthly source-label rasters;
- known-source exclusion rasters;
- prepared patch catalogues;
- channel normalization tables;
- PyTorch model checkpoints;
- validation and test prediction CSVs;
- overall and monthly metrics tables;
- held-out monthly confusion maps;
- optional wall-to-wall source-probability maps.

These outputs can be large. Store them outside GitHub and document their
location, checksum, and software commit hash.
