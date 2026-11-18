# Namma SARS

**A real-data satellite change explorer for India.** Optical and Sentinel-1 screening uses Microsoft Planetary Computer. Real NISAR Level-2 GCOV discovery uses NASA CMR through the official `earthaccess` client. The default map is a cloud-penetrating rolling 12-day stack of official NISAR GCOV daily mosaics from NASA GIBS.

The ready-built browser interface is included, so an older MacBook **does not need npm, Node, or a local frontend build** to launch it.

## Run it on macOS older than 13

Requirements: Python **3.10 or 3.11**, internet access, and a browser with WebGL support. On an older Mac, try Python 3.11 if available for your macOS version.

```bash
cd ~/Downloads/Namma-SARS
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements-analysis.txt
python main.py
```

Open `http://127.0.0.1:8765` if a browser window doesn't open. To stop the server press `Ctrl+C`.

> **No key is needed to open maps or search NISAR metadata.** To download selected NISAR GCOV HDF5 science files, copy `.env.example` to `.env` and add a newly generated `EARTHDATA_TOKEN`. Never reuse an exposed token. An optional `PC_SDK_SUBSCRIPTION_KEY` can raise Planetary Computer limits.

### If `pip` cannot install Rasterio

Make sure you are using Python 3.10 or 3.11 and a recent pip. Try `python -m pip install --only-binary=:all: 'rasterio>=1.3.9,<1.4'` to check whether your Python/macOS combination has an existing wheel. Do not try to install large GIS libraries from source on a very old Mac unless you've deliberately set up GDAL. The app itself starts without analysis extras:

```bash
python -m pip install -r requirements.txt
python main.py
```

Satellite **catalog access is distinct from actual raster processing**. Without `rasterio`, `numpy` and `Pillow`, the dashboard opens but analyses report that the dependencies are missing.

## What is working

- Live STAC search with `pystac-client` and `planetary_computer.sign_inplace` against `https://planetarycomputer.microsoft.com/api/stac/v1`.
- Water-like change *screening* from Sentinel-2 L2A green and SWIR imagery (MNDWI), masking invalid/obscured pixels with SCL.
- Dense-vegetation decline/recovery *screening* from Sentinel-2 red + near-infrared (NDVI), **not** verified forest loss.
- Optional historical **tree-cover reference layer** using the Microsoft-hosted Impact Observatory annual 9-class map (`io-lulc-annual-v02`, class 2 = trees; reference through 2023). Overlapping the reference with NDVI decline flags candidate tree-cover change, **not verified deforestation**.
- Optional Sentinel-1 RTC VV backscatter-change overlays (dB). This is **Sentinel-1, not NISAR**.
- Real NISAR L2 GCOV discovery using `earthaccess` and official validated, provisional and beta collection short names.
- Two-stage NISAR workflow: locate one acquisition and draw its footprint first; only after confirmation import historical metadata for the identical AOI.
- Explicit authenticated download of one to four selected GCOV files. The app never silently downloads an entire multi-hundred-file history.
- Keyless MapLibre basemaps: official NASA GIBS NISAR GCOV false-colour radar mosaics stacked across the selected date and previous 11 days, with NASA Blue Marble beneath coverage gaps; NASA VIIRS and OpenStreetMap remain optional references.
- Eight study areas around India, including Bengaluru, Karnataka's Western Ghats and wetlands, plus click-to-select custom small bounding boxes in India.
- Editable before/after dates, progress polling, color-coded image overlays on a MapLibre map, estimated affected hectares, scene provenance and JSON export.
- No simulated measurements, fictitious forest-loss counts, hazard predictions, flood alerts or "safe routes".

## What is *not* working / scope limits

- This release has **not** been validated against live Planetary Computer rasters in this build environment (outbound network is unavailable). The satellite-processing paths are implemented, but real observations must be tested using your own internet connection. Failing asset reads or coverage are shown as errors, not replaced with fake imagery.
- Individual optical scenes are selected within **35 days** of each chosen date; radar scenes within **24 days**. Scenes are single acquisitions, not annual cloud-free composites. The UI displays their actual acquisition dates after analysis.
- One scene covering the entire bounding rectangle is required per date; cross-tile mosaics are not implemented. Heavy cloud cover may leave too few pixels.
- NDVI/MNDWI thresholds are heuristic and location/season dependent, especially across different seasons. The forest reference may be years older than the selected observations. **Vegetation loss is not confirmed deforestation; water expansion is not confirmed flood.** Terrain effects, agriculture, atmospheric variation and alignment may affect results.
- Sentinel-1 power differences are *not* flood detections. Same-pass geometry should be used; this prototype checks orbit direction where metadata permits, but does not fully enforce relative-orbit matching or radiometric normalization.
- Area numbers approximate spherical WGS84 grid hectares for candidate pixels, not ground-survey measurements.
- The ready-built UI bundles React and MapLibre. Internet access is needed for map tiles and remote satellite catalogs, not for JavaScript package CDNs.

## Frontend development (optional)

React/MapLibre source: `dashboard/src/main.jsx`, `dashboard/src/styles.css`.

For modern Node.js systems with the npm registry accessible:

```bash
cd dashboard
npm ci
npm run build
```

The included `dashboard/dist/` is **already available** and works without running these commands. It uses CDN libraries because the build environment could not install npm dependencies. Production deployments should bundle and self-host those libraries and adopt an authenticated, rate-limited backend if exposed to other users. The HTTP server binds to `127.0.0.1` by default; do not expose it to the public internet without security controls.

## Files

```text
main.py                         run the local server
namma_sars/server.py            local API, background jobs, static files
namma_sars/planetary.py         Microsoft STAC query and scene selection
namma_sars/nisar_l2.py          NISAR GCOV probe, history and capped download class
namma_sars/analysis.py          raster reading and NDVI/MNDWI/SAR change screening
namma_sars/regions.py           declared WGS84 study rectangles
dashboard/src/                 editable React frontend
dashboard/dist/                ready-built CDN-backed React frontend
tests/test_namma_sars.py        offline deterministic checks
requirements.txt               STAC packages
requirements-analysis.txt      satellite raster analysis packages
.env.example                   Earthdata download token + optional Microsoft key
```

## Offline tests

```bash
python -m unittest discover -s tests -v
```

## Attribution / provenance

- Planetary Computer STAC: https://planetarycomputer.microsoft.com/docs/
- Sentinel-2 L2A, Sentinel-1 RTC and annual Impact Observatory land-cover: hosted by Microsoft; **original Sentinel missions are part of Europe's Copernicus program**, but this app does **not** connect to the separate Copernicus Data Space service or require Copernicus credentials.
- Default basemap: NASA GIBS / EOSDIS NISAR Level-2 GCOV normalized radar backscatter visualization. NASA Blue Marble fills uncovered pixels. Optional street context: © OpenStreetMap contributors. Basemaps are visual context, not quantitative measurement inputs.

See `docs/METHODOLOGY.md` and `docs/CHANGELOG.md` for details.
