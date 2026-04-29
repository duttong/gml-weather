# Weather Reanalysis Comparison

Compares observed wind data from [NOAA GML](https://gml.noaa.gov/) flask measurement sites against reanalysis wind data from the [Open-Meteo Archive API](https://open-meteo.com/). Validates historical wind observations at globally distributed monitoring stations.

## Overview

The project fetches HCFC-142b flask measurements (which include coincident wind observations) from the NOAA GML network, then retrieves hourly reanalysis wind data for each observation timestamp via Open-Meteo. Results are saved as CSVs and visualized as time series comparison plots.

Five sites are analyzed by default:

| Code | Location | Lat | Lon |
|------|----------|-----|-----|
| ALT | Alert, Northwest Territories | 82.452°N | 62.517°W |
| BRW | Barrow, Alaska | 71.323°N | 156.611°W |
| CGO | Cape Grim, Tasmania | 40.683°S | 144.690°E |
| HFM | Harvard Forest, Massachusetts | 42.500°N | 72.200°W |
| NWR | Niwot Ridge, Colorado | 40.050°N | 105.580°W |

## Scripts

### `met_comp.py` — Main pipeline

Runs the full comparison workflow: loads GML data, fetches reanalysis wind for each observation, writes enriched CSVs, and generates plots.

```bash
python met_comp.py
```

Outputs go to `results/`:
- `{site}_gml_comparison.csv` — observed + reanalysis wind data
- `wind_{site}_comparison_speed.png` — speed time series with difference panel
- `wind_{site}_comparison_direction.png` — direction time series with difference panel

Caches API responses at the day level to avoid redundant requests. If a CSV already exists for a site, it is loaded from disk rather than re-fetched.

---

### `met_api.py` — Open-Meteo API client

Fetches and interpolates reanalysis wind at a specific site and time. Uses linear interpolation for wind speed and circular interpolation for wind direction (handles the 0°/360° wraparound correctly).

```bash
# Query by site code
python met_api.py --site ALT --datetime "2020-01-15 12:30"

# Query by coordinates
python met_api.py --lat 82.452 --lon -62.517 --datetime "2020-01-15 12:30"
```

Supported datetime formats: `YYYYMMDDHHMMSS`, `YYYYMMDDHHMM`, `YYYY-MM-DD HH:MM:SS`, `YYYY-MM-DD HH:MM`.

Site codes and coordinates are read from `noaa-sites.yaml`.

---

### `flask_data.py` — NOAA GML data loader

Parses NOAA GML flask measurement files (tab-delimited). Handles both legacy and current column formats.

```bash
# Preview data from default HCFC-142b URL
python flask_data.py

# Load from a local file and save to CSV
python flask_data.py --source data/HCFC142B_GCMS_flask.txt --save-csv output.csv
```

---

### `met_comparisons.ipynb` — Exploratory notebook

Jupyter notebook with examples for loading data, querying the API, and generating plots.

## Configuration

`noaa-sites.yaml` contains coordinates and metadata for 100+ NOAA monitoring stations. Each entry includes:

```yaml
- code: BRW
  name: Barrow, Alaska
  latitude: 71.323
  longitude: -156.611
  elevation_m: 11
  gmt_offset: -9
```

## Dependencies

- Python 3.8+
- `pandas`
- `numpy`
- `matplotlib`
- `typer`
- `PyYAML`

Install with:

```bash
pip install pandas numpy matplotlib typer pyyaml
```

## Data Sources

- **Observations**: [NOAA GML HCFC-142b flask data](https://gml.noaa.gov/aftp/data/hats/hcfcs/hcfc142b/flasks/HCFC142B_GCMS_flask.txt)
- **Reanalysis**: [Open-Meteo Archive API](https://archive-api.open-meteo.com/v1/archive) (free, no key required)
