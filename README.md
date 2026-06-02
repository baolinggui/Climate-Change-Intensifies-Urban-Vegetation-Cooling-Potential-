Steps 1-3 pertain to data construction, which is relatively complex. I first constructed a collection of ERA5 re-climate data and other variables (a dataset format suitable for training) (GlobalUrbanCooling_dataset_v2.parquet). 
Then, based on this dataset, I aggregated the MERRA-2 data into the aforementioned dataset to form GlobalUrbanCooling_dataset_v2_ERA5_plus_MERRA2hybrid.parquet, which can be used for comparative analysis. 
This data construction is relatively complicated; you can also try to construct it yourself. Alternatively, you can directly obtain these two datasets and perform the processing described in steps 4-6 below to obtain the results of this project. Below is the link to obtain the data:
https://figshare.com/s/b3eba93a87f51c558a39
# Climate Change Intensifies Urban Vegetation Cooling Potential

Code and data workflow for the study of urban vegetation cooling across global cities from 2002 to 2025.

This repository is a research-code archive. It is organized as ordered notebook-style code collections rather than as an installable Python package. Some files contain Google Earth Engine (GEE) JavaScript, while others contain Python cells intended for Google Colab. Run the code in the order described below and update the paths for your own Google Drive and GEE account.

## Quick Start

If your goal is to reproduce the main modeling and analysis workflow, start from the processed datasets rather than rebuilding every satellite export.

1. Download the processed datasets from [Figshare](https://figshare.com/s/b3eba93a87f51c558a39).
2. Create the Google Drive folders shown in [Expected Drive Layout](#expected-drive-layout).
3. Place `GlobalUrbanCooling_dataset_v2.parquet` in:

   ```text
   /content/drive/MyDrive/Second_research/derived_datasets/
   ```

4. Open Google Colab, mount Google Drive, and install the Python packages listed below.
5. Run the code cells in `5. Training and CE Calculation` in order.
6. Run the required analysis cells from `6. Nonlinear Analysis of Different Variables`.
7. To reproduce the ERA5 versus MERRA-2 comparison, also prepare the dual-source parquet file and run the relevant cells in `7. Comparative analysis of two reanalysis-forced data sets`.

Steps 1 to 4 of the repository are mainly for rebuilding the input datasets. They are not required if you use the processed Figshare data.

## Repository Files

| File | Environment | Purpose |
| --- | --- | --- |
| `1. GEE data obtian` | GEE Code Editor | Export ERA5-Land, land cover, DEM, EVI, LAI, LST, nighttime light, and optional MERRA-2 GeoTIFF stacks for one city at a time. |
| `2. Based on ERA5 location data, the merra2 data is summarized into a tabular data.` | Google Colab | Prepare MERRA-2 point tables, merge yearly CSV exports, validate ERA5 versus MERRA-2 variables, and create hybrid MERRA-2 forcing datasets. |
| `3. (GEE) MERRA2 data was obtained based on ERA5 point data and summarized into tabular data. data was obtained based on ERA5 point data` | GEE Code Editor | Export yearly MERRA-2 wide CSV tables from a GEE point asset. |
| `4. Dataset construction for training (ERA5)` | Google Colab | Read city GeoTIFF stacks and build `GlobalUrbanCooling_dataset_v2.parquet`. |
| `5. Training and CE Calculation` | Google Colab | Train the LST model, estimate vegetation-attributable cooling, save the derived dataset, and merge city metadata. |
| `6. Nonlinear Analysis of Different Variables` | Google Colab | Run nonlinear response, climate-window, probability, vulnerability, and mechanism analyses. This file contains multiple alternative analysis blocks; run only the blocks needed for your figure or table. |
| `7. Comparative analysis of two reanalysis-forced data sets` | Google Colab | Build ERA5 versus MERRA-2 comparison figures and the revised main-text outputs. This file also contains multiple figure-specific blocks. |
| `city information1.xlsx` | Spreadsheet | Metadata for 103 retained cities. |

The long filenames are preserved to match the original archive. For easier use, you may rename the code files locally to `01_gee_exports.js`, `02_merra_processing.py`, and so on.

## Requirements

### Accounts

- A [Google Earth Engine](https://code.earthengine.google.com/) account
- A Google Drive account
- A Google Colab notebook, or a Python environment with enough memory for the parquet datasets

### Python Packages

In Google Colab, run:

```python
from google.colab import drive
drive.mount("/content/drive")

!pip -q install numpy pandas matplotlib scipy tqdm pyarrow openpyxl
!pip -q install rasterio rioxarray xarray
!pip -q install scikit-learn xgboost shap statsmodels joblib catboost
```

`catboost` and `shap` are only needed for analysis blocks that import them.

## Expected Drive Layout

Use this folder structure unless you update the path constants inside the code cells:

```text
/content/drive/MyDrive/Second_research/
  data/
    Guangzhou/
      <exported GeoTIFF stacks>
    Shanghai/
      <exported GeoTIFF stacks>
    ...
  derived_datasets/
    GlobalUrbanCooling_dataset_v2.parquet
    GlobalUrbanCooling_dataset_v3.parquet
    merra2_match_points.csv
    merra2_match_points_dedup_for_gee.csv
  MERRA/
    CSV/
      MERRA2_wide_2002.csv
      ...
      MERRA2_wide_2025.csv
    processed/
    hybrid/
  city_information1.xlsx
```

The code currently refers to a differently named metadata spreadsheet. Upload the included `city information1.xlsx` file and edit `CITY_META_PATH` in `5. Training and CE Calculation`:

```python
CITY_META_PATH = "/content/drive/MyDrive/Second_research/city_information1.xlsx"
```

## Route A: Reproduce Modeling from Processed Data

This is the recommended route for most users.

### A1. Prepare the ERA5 Dataset

Download `GlobalUrbanCooling_dataset_v2.parquet` from [Figshare](https://figshare.com/s/b3eba93a87f51c558a39) and place it in:

```text
/content/drive/MyDrive/Second_research/derived_datasets/
```

### A2. Train the LST Model

Open `5. Training and CE Calculation` in Colab and run the cells sequentially:

1. Load `GlobalUrbanCooling_dataset_v2.parquet`.
2. Convert dates and inspect missing-value ratios.
3. Fill missing features by pixel and retain finite rows.
4. Apply winsorization and the time split.
5. Train the XGBoost LST model with spatial cross-validation.
6. Calculate cooling strength and cooling efficiency.
7. Save `GlobalUrbanCooling_dataset_v3.parquet`.
8. Merge latitude from the city metadata spreadsheet if needed by later analyses.

The main in-memory objects used by later cells are:

```text
dataset
dataset_fe
feature_cols
final
```

Keep the Colab session active when moving from training to downstream analysis. The current training file creates `final` in memory but does not save it automatically.

### A3. Run the Nonlinear Analyses

Open `6. Nonlinear Analysis of Different Variables` in the same Colab session.

This file is a collection of figure- and question-specific analysis blocks. It is not intended to be executed as one Python script from top to bottom. Each major block restarts with its own imports, configuration, and expected input columns.

Before running a block:

1. Confirm that `dataset` exists in memory.
2. Check the block header and `assert` statements.
3. Update `OUTDIR` or `OUT_DIR`.
4. Run the block as a complete Colab cell.
5. Inspect the generated CSV, JSON, and PNG files.

Typical outputs include nonlinear response curves, VPD window metrics, `Pr(HighCE)` curves, city vulnerability tables, marginal-effects summaries, and conceptual figures.

## Route B: Rebuild the ERA5 Dataset from GEE Exports

Use this route only if you need to reconstruct the input parquet dataset.

### B1. Export Per-City GeoTIFF Stacks

Open `1. GEE data obtian` in the GEE Code Editor.

The file contains separate JavaScript blocks for:

1. ERA5-Land daily aggregates converted to 8-day stacks
2. MODIS annual land cover
3. SRTM DEM
4. MODIS EVI
5. MODIS LAI
6. MODIS LST
7. VIIRS nighttime light
8. Optional MERRA-2 segmented raster export

Run one block at a time. At the top of each block, update:

```javascript
var CITY_NAME = "Guangzhou";
var CITY_LON = 113.2644;
var CITY_LAT = 23.1291;
var CITY_BUFFER_KM = 80;
```

Start the generated tasks from the GEE **Tasks** panel and download or move the GeoTIFF outputs into:

```text
/content/drive/MyDrive/Second_research/data/<CityName>/
```

Repeat this process for each city you intend to include.

### B2. Build the Training Parquet

Open `4. Dataset construction for training (ERA5)` in Colab.

Update:

```python
ROOT_DATA = "/content/drive/MyDrive/Second_research/data"
N_CITIES = 106
CITY_LIST = None
OUT_DIR = "/content/drive/MyDrive/Second_research/derived_datasets"
```

Run the cells in order. The code:

1. Finds city folders.
2. Reads and aligns raster stacks.
3. Builds the nighttime-light urban-core mask.
4. Samples time steps and pixels.
5. Constructs the multi-city table.
6. Saves:

   ```text
   /content/drive/MyDrive/Second_research/derived_datasets/GlobalUrbanCooling_dataset_v2.parquet
   ```

The spreadsheet contains 103 retained cities, while the reconstruction code uses `N_CITIES = 106` as an upper limit before downstream filtering. Set `CITY_LIST` explicitly if you need an exact city set.

## Optional: ERA5 Versus MERRA-2 Comparison

The dual-source comparison requires additional MERRA-2 processing.

### C1. Prepare MERRA-2 Match Points

The MERRA workflow expects:

```text
/content/drive/MyDrive/Second_research/derived_datasets/merra2_match_points.csv
```

The repository does not currently include the code that creates this file from `GlobalUrbanCooling_dataset_v2.parquet`. Obtain it from the shared processed data, or generate a table with at least the expected date, longitude, latitude, city, and pixel keys before continuing.

Run the first Python block in:

```text
2. Based on ERA5 location data, the merra2 data is summarized into a tabular data.
```

This creates:

```text
merra2_match_points_dedup_for_gee.csv
```

### C2. Upload the Point Asset to GEE

Upload `merra2_match_points_dedup_for_gee.csv` as a GEE table asset.

Open file `3. (GEE) MERRA2 data was obtained based on ERA5 point data and summarized into tabular data. data was obtained based on ERA5 point data` and replace:

```javascript
var POINTS_ASSET = "projects/ee-abao9474/assets/merra2_match_points_dedup_for_gee";
```

with your own asset path.

Run the script and start the yearly CSV export tasks. Place the exported CSV files in:

```text
/content/drive/MyDrive/Second_research/MERRA/CSV/
```

### C3. Create the Hybrid MERRA-2 Dataset

Return to file `2. Based on ERA5 location data, the merra2 data is summarized into a tabular data.` and run the later Python blocks in order.

The workflow:

1. Merges yearly MERRA-2 CSV files.
2. Reshapes the wide tables to long format.
3. Matches ERA5 and MERRA-2 samples.
4. Validates variable relationships.
5. Fits ERA5-to-MERRA-2 mapping models.
6. Saves the dual-source parquet:

   ```text
   /content/drive/MyDrive/Second_research/MERRA/hybrid/GlobalUrbanCooling_dataset_v2_ERA5_plus_MERRA2hybrid.parquet
   ```

### C4. Run the Comparative Figures

Open `7. Comparative analysis of two reanalysis-forced data sets`.

Run one figure block at a time. Most blocks expect one or more of:

```text
final
feature_cols
GlobalUrbanCooling_dataset_v2_ERA5_plus_MERRA2hybrid.parquet
GlobalUrbanCooling_dataset_v2_dualCE_full.parquet
```

The file contains separate blocks for revised main-text figures and supporting summaries. Update each `DATA_FP`, `FULL_FP`, `RAW_FP`, `SAVE_FP`, and `OUTDIR` before running.

## Important Reproducibility Notes

The archived code is sufficient to understand and rerun most of the workflow, but it is not yet a complete, exact reproduction package. Confirm the following points before using it for a formal reproduction:

| Topic | Manuscript description | Current archived code | Action needed |
| --- | --- | --- | --- |
| Counterfactual vegetation reference | `EVI_ref = 0.05` | Core training and comparative cells use `EVI -> 0` | Update the counterfactual cells if reproducing the manuscript definition. |
| Cooling-efficiency denominator | Normalize by vegetation contrast relative to `EVI_ref`; exclude contrast `< 0.02` | Several cells use `max(EVI, 0.02)` or divide by EVI directly | Use one manuscript-consistent formula throughout. |
| Spatial grid | Common `0.1 degree` grid | Most GEE export blocks set `DEG_PER_PIXEL = 0.005`, although comments often say `0.1 degree`; the optional MERRA raster block uses `0.1` | Confirm the intended grid and edit all GEE blocks consistently. |
| Main model | Manuscript describes histogram-based gradient boosting | Training file uses `xgboost.XGBRegressor`; MERRA mapping uses `HistGradientBoostingRegressor` for selected variables | Confirm which model produced the reported manuscript results. |
| City count | Final manuscript dataset contains 103 cities | Builder uses `N_CITIES = 106`; spreadsheet contains 103 city rows | Set an explicit `CITY_LIST` for exact reproduction. |
| MERRA match points | Required intermediate table | Generator for `merra2_match_points.csv` is not included | Provide the table or add a generation cell. |
| Model persistence | Comparative file can load a saved model | Training file creates `final` in memory but does not save it | Keep the same Colab session or add a `joblib.dump(final, ...)` step. |

## Recommended Cleanup for a Future Release

For a cleaner public release:

1. Split each code collection into separate `.js`, `.py`, or `.ipynb` files.
2. Rename files with short ordered names such as `01_gee_exports.js`.
3. Add the missing `merra2_match_points.csv` generation step.
4. Save the trained model and `feature_cols` after training.
5. Standardize the grid resolution and manuscript counterfactual formula.
6. Add a small sample dataset for smoke testing.
7. Add a license and citation section.

## Citation

Please cite the associated manuscript when using this workflow. Add the final journal citation and DOI here after publication.
