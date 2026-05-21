# Urban Analysis Framework

A Google Colab-based framework for urban remote sensing analysis using satellite imagery from Google Earth Engine. The framework performs **Land Surface Temperature (LST)**, **Urban Heat Island (UHI)**, **spectral change detection**, and **deep learning-based urban change detection** over user-defined study areas.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step-by-Step Setup](#step-by-step-setup)
4. [Folder Structure](#folder-structure)
5. [Running the Pipelines](#running-the-pipelines)
6. [Change Detection Validation Pipeline](#change-detection-validation-pipeline)
7. [Output Files](#output-files)
8. [Troubleshooting](#troubleshooting)
9. [Credits](#credits)

---

## Overview

This framework analyzes urban areas across two time periods (T1 and T2) using Landsat and Sentinel satellite data. It is organized into three spatial zones of a study area:

| Folder | Description |
|---|---|
| `A_Zone/` | Analysis for Zone A |
| `B_Zone/` | Analysis for Zone B |
| `Sobral_Perimeter/` | Full perimeter of Sobral, CE, Brazil |
| `Change_Detection_Validation/` | Deep learning urban change detection (Sentinel-1 + Sentinel-2) |
| `Heigths_Neural_Network_Change_Detection/` | Pre-trained U-Net model weights |

**What each pipeline computes:**
- Land Surface Temperature (LST) at 30 m resolution
- Urban Heat Island (UHI) intensity (classified in 5 levels)
- Spectral indices: SAVI (vegetation), DBSI (bare soil), NBAI (built-up area)
- Delta maps (change between T1 and T2) and Change Detection Index (CDI)
- Deep learning change detection map from multi-modal Sentinel data

---

## Prerequisites

Before you start, make sure you have the following:

### 1. Google Account
You need a standard Google account (Gmail). This gives you access to:
- **Google Drive** — where the framework folder will live
- **Google Colab** — the cloud environment where the notebooks run (free, no installation required)

> If you don't have one, create a Google account at [accounts.google.com](https://accounts.google.com).

### 2. Google Earth Engine Account
Google Earth Engine (GEE) is a satellite data platform. You need to:

1. Go to [earthengine.google.com](https://earthengine.google.com) and click **Get Started**.
2. Sign in with your Google account.
3. Request access by filling out the registration form (select "Academic / Research" as the purpose). Approval usually takes a few minutes to a few hours.
4. Once approved, go to the [GEE Cloud Console](https://console.cloud.google.com/earth-engine) and **create a project**. Copy the **Project ID** — you will need it inside the notebooks. It looks like `ee-yourname` or a custom name you choose.

> The GEE project ID is the string you pass to `ee.Initialize(project="your-project-id")` in the notebooks. Locate all cells where this appears and replace the example value with your own ID.

---

## Step-by-Step Setup

### Step 1 — Place the Framework folder in Google Drive

The notebooks are configured to read files from Google Drive. For the paths to work correctly, **the `Framework` folder must be placed at the root of your Google Drive**, not inside any subfolder.

Your Google Drive should look like this after uploading:
```
My Drive/
└── Framework/
    ├── A_Zone/
    ├── B_Zone/
    ├── Sobral_Perimeter/
    ├── Change_Detection_Validation/
    ├── Heigths_Neural_Network_Change_Detection/
    └── README.md
```

**How to upload:**
1. Go to [drive.google.com](https://drive.google.com).
2. Make sure you are at the root of "My Drive" (not inside any folder).
3. Drag and drop the entire `Framework` folder into the browser window.
4. Wait for the upload to complete — this may take several minutes depending on your internet connection.

### Step 2 — Open a notebook in Google Colab

1. In Google Drive, navigate to the notebook you want to run (e.g., `A_Zone/Pipeline_A_Zone.ipynb`).
2. Double-click the `.ipynb` file. It will open in **Google Colab** automatically.
3. If it does not open automatically, right-click the file → **Open with** → **Google Colaboratory**.

### Step 3 — Mount Google Drive

The first cell of every notebook mounts your Google Drive. When you run it, a pop-up will ask for permission to access your Drive.

```python
from google.colab import drive
drive.mount('/content/drive')
```

Click **Connect to Google Drive** and select your Google account. After mounting, the `Framework` folder will be accessible at `/content/drive/MyDrive/Framework/`.

### Step 4 — Authenticate Google Earth Engine

Every notebook has a cell like:

```python
try:
    ee.Initialize(project="ee-joaovictornh01")  # <-- replace with your project ID
except:
    ee.Authenticate()
    ee.Initialize(project="ee-joaovictornh01")  # <-- replace with your project ID
```

**Replace `"ee-joaovictornh01"` with your own GEE project ID** before running this cell.

On the first run, `ee.Authenticate()` will open a browser tab asking you to authorize access. Log in with the same Google account used for GEE, copy the authorization code, and paste it back into the Colab input box.

### Step 5 — Install dependencies

Each notebook includes a cell that installs the required Python packages automatically:

```python
!pip install localtileserver geemap eemont geopandas rasterio folium -q
```

Run this cell and wait for the installation to finish (usually 1–3 minutes). You only need to do this once per Colab session.

### Step 6 — Run the notebook cells in order

After completing the steps above, run each cell **from top to bottom** using the play button (▶) or by pressing `Shift + Enter`. Do not skip cells.

---

## Folder Structure

```
Framework/
│
├── A_Zone/
│   ├── Pipeline_A_Zone.ipynb          ← Main analysis notebook for Zone A
│   ├── A_Zone.kmz                     ← Zone A boundary (import into GEE or QGIS)
│   ├── Data_A_Zone/                   ← Output GeoTIFF files generated by the pipeline
│   └── modulos/                       ← Python modules (do not edit unless you know what you're doing)
│       ├── config.py                  ← Color palettes, scales, and constants
│       ├── gee_setup.py               ← GEE authentication and Drive mounting helpers
│       ├── image_processing.py        ← Landsat/Sentinel image processing
│       ├── lst_analysis.py            ← LST and UHI computation
│       ├── spectral_analysis.py       ← SAVI, DBSI, NBAI indices and CDI
│       ├── change_detection.py        ← Change detection logic
│       ├── downscaling.py             ← LST downscaling to 10 m
│       ├── water_mask.py              ← Water body masking
│       └── export_utils.py            ← Export to Google Drive helpers
│
├── B_Zone/                            ← Same structure as A_Zone
│
├── Sobral_Perimeter/
│   ├── Pipeline_Sobral_Perimeter.ipynb
│   ├── Sobral_Perimeter.kml           ← Full city boundary
│   └── modulos/                       ← Same modules as above
│
├── Change_Detection_Validation/
│   ├── 01.Search_and_Export_Sentinel_Data.ipynb
│   ├── 02.U-net_extraction_Change_Detection_Map_for_Validation.ipynb
│   ├── 03.Validation_CD_Model.ipynb
│   └── urban_cd_app/                  ← Input/output GeoTIFFs for this pipeline
│
└── Heigths_Neural_Network_Change_Detection/
    ├── mmcr_train100.pt               ← Pre-trained U-Net weights (included)
    └── Link_Acess.txt                 ← Download link and model citation
```

---

## Running the Pipelines

### Main Analysis Pipeline (A_Zone, B_Zone, Sobral_Perimeter)

Each zone has a single self-contained notebook (`Pipeline_*.ipynb`). Open and run it top to bottom.

**What you may need to customize:**
- Your GEE project ID (see Step 5).
- The time periods T1 and T2 (search for `start_date` and `end_date` in the notebook).
- The region of interest (ROI), if you want to analyze a different area.

The pipeline will:
1. Retrieve Landsat imagery from GEE.
2. Compute LST, UHI, and spectral indices for both time periods.
3. Generate delta maps showing change between T1 and T2.
4. Export all results as GeoTIFF files to the corresponding `Data_*/` folder on your Google Drive.

### Change Detection Validation Pipeline

This pipeline uses a deep learning U-Net model trained on Sentinel-1 SAR and Sentinel-2 MSI data to detect urban changes. It must be run **in sequence**:

| Step | Notebook | What it does |
|---|---|---|
| 1 | `01.Search_and_Export_Sentinel_Data.ipynb` | Searches GEE for Sentinel images, lets you preview them, and exports T1/T2 image pairs to Drive |
| 2 | `02.U-net_extraction_Change_Detection_Map_for_Validation.ipynb` | Loads exported images and the pre-trained model, runs inference, and saves the change detection map |
| 3 | `03.Validation_CD_Model.ipynb` | Compares the model output against a ground truth mask and computes accuracy metrics |

> **Important:** Complete notebook 01 and wait for the GEE export tasks to finish before opening notebook 02. You can monitor export progress at [code.earthengine.google.com/tasks](https://code.earthengine.google.com/tasks).

---

## Output Files

All outputs are GeoTIFF files (`.tif`) saved to your Google Drive. Key outputs per zone:

| File | Description |
|---|---|
| `sentinel2_*_t1.tif` / `*_t2.tif` | Sentinel-2 multispectral image (bands B2–B8, B11, B12) |
| `sentinel1_*_t1.tif` / `*_t2.tif` | Sentinel-1 SAR composite (VV, VH, normalized 0–1) |
| `LST_30m_*_t1.tif` / `*_t2.tif` | Land Surface Temperature at 30 m |
| `UHI_30m_*_t1.tif` / `*_t2.tif` | Urban Heat Island map |
| `UHI_Intensity_30m_*_t1.tif` / `*_t2.tif` | UHI classified in 5 intensity levels |
| `Sentinel2_SAVI_*_T1.tif` / `*_T2.tif` | Soil-Adjusted Vegetation Index |
| `Sentinel2_DBSI_*_T1.tif` / `*_T2.tif` | Dry Bare Soil Index |
| `Sentinel2_NBAI_*_T1.tif` / `*_T2.tif` | Normalized Built-up Area Index |
| `Delta_LST_30m_*.tif` | LST change between T1 and T2 |
| `Delta_UHI_30m_*.tif` | UHI change between T1 and T2 |
| `Delta_UHI_Intensity_30m_*.tif` | UHI intensity change |
| `Delta_S2_SAVI_*.tif` | SAVI change |
| `Delta_S2_DBSI_*.tif` | DBSI change |
| `Delta_S2_NBAI_*.tif` | NBAI change |
| `CDI_S2_SAVI_*.tif` | Change Detection Index — SAVI |
| `CDI_S2_DBSI_*.tif` | Change Detection Index — DBSI |
| `CDI_S2_NBAI_*.tif` | Change Detection Index — NBAI |

All output files use **EPSG:32724** (UTM Zone 24S) as the coordinate reference system, suitable for the Sobral region.

---

## Troubleshooting

**"Module not found" or import errors**
Make sure Google Drive is mounted and the `Framework` folder is at the root of your Drive (not inside a subfolder). Check that the path `/content/drive/MyDrive/Framework/` exists in Colab by running:
```python
import os
os.listdir('/content/drive/MyDrive/Framework')
```

**GEE authentication fails or keeps asking for login**
Run the authentication cell again. If it fails repeatedly, go to [earthengine.google.com](https://earthengine.google.com), sign in, and ensure your account and project are active.

**GEE export tasks are not appearing on Drive**
Export tasks run asynchronously on Google's servers. Go to [code.earthengine.google.com/tasks](https://code.earthengine.google.com/tasks) to see their status. Large exports (full city perimeter) may take 10–30 minutes.

**Colab session disconnects mid-run**
Colab free tier sessions can time out. If this happens, remount Drive, re-authenticate GEE, and re-run only the cells from where the session stopped. Output files already exported to Drive are preserved.

**Model file not found (`mmcr_train100.pt`)**
The file is included in the `Heigths_Neural_Network_Change_Detection/` folder. If it is missing after uploading to Drive, re-upload it from the original `Framework` folder. The download link and citation can be found in `Heigths_Neural_Network_Change_Detection/Link_Acess.txt`.

---

## Credits

**Neural network model for change detection:**

> Hafner, S., Ban, Y. and Nascetti, A., 2023. *Semi-Supervised Urban Change Detection Using Multi-Modal Sentinel-1 SAR and Sentinel-2 MSI Data.* Remote Sensing, 15(21), p.5135.

Original repository: [github.com/SebastianHafner/SemiSupervisedMultiModalCD](https://github.com/SebastianHafner/SemiSupervisedMultiModalCD)

**Satellite data:** Copernicus Sentinel-1 and Sentinel-2 (ESA) | Landsat (USGS/NASA), accessed via Google Earth Engine.
