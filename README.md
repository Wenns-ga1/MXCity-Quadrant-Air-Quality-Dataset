# Mexico City Quadrant Air Quality Dataset for Anomaly Detection
### SINAICA via OpenAQ, 2016

**Version:** 1.0.0 &nbsp;|&nbsp; **Date:** 2026-05-08 &nbsp;|&nbsp; **License:** Creative Commons Zero v1.0 Universal (CC0)

---

## 1. Overview

This dataset contains hourly air quality sensor readings from 16 government-grade monitoring stations distributed across the Mexico City Metropolitan Area (MCMA), covering the period March 10 – October 2, 2016.

Stations are organized into four geographic quadrants defined by urban land-use characteristics, enabling spatial comparison across industrial, transport-corridor, urban-core, and peri-urban reference zones.

It provides labeled telemetry suitable for:

- Benchmarking fault classification (spike detection, communication loss)
- Study of spatial pollution gradients across heterogeneous urban zones
- Seasonal and diurnal pattern analysis over a seven-month observation window

---

## 2. Scientific Context and Motivation

In practice, distributed IoT sensor networks are subject to failure modes — spikes, dropouts, communication loss — that corrupt a data stream before it reaches the Analyze and Plan stages in a MAPE-K system. A robust Monitor component must identify and flag these anomalies in real time.

Mexico City was selected as the study environment given the following conditions:

- One of the largest metropolitan areas in the world (~22 million people), with one of the densest air quality sensor networks in Latin America
- Extreme pollution gradients between industrial zones (NW), transport corridors (NE), urban cores (SW), and peri-urban fringes (SE)
- Well-documented seasonal patterns including dry-season pollution peaks (March–May) that generate natural anomalous spikes useful for anomaly detection benchmarking
- Government-grade instrumentation with established calibration protocols, making data suitable for academic citation

**Air quality reference values:**
World Health Organization. (2021). WHO global air quality guidelines.
https://www.who.int/publications/i/item/9789240034228

---

## 3. Data Source

### 3.1 Primary Source

Data was retrieved from the OpenAQ v3 public API (https://openaq.org), an open-source platform that aggregates government air quality monitoring data from national networks worldwide. OpenAQ acts as a data mirror and does not operate sensors directly.

The underlying sensor network is Mexico's SINAICA (Sistema Nacional de Informacion de la Calidad del Aire), operated by INECC (Instituto Nacional de Ecologia y Cambio Climatico). SINAICA is the authoritative source for Mexico City air quality data; OpenAQ ingests a subset of its historical archive.

> **Note for researchers wishing to extend this dataset:**
> OpenAQ's ingestion of SINAICA data is partial and not real-time. As of 2026, the majority of Mexico City SINAICA stations accessible via OpenAQ have data only up to 2016–2022 depending on the station. For current or near-real-time data, researchers should query SINAICA directly at:
> https://sinaica.inecc.gob.mx
>
> SINAICA also covers additional Mexican cities not represented in this dataset. The quadrant methodology described in Section 4 can be directly applied to other cities by adjusting the geographic bounding boxes in Cell 4 of the collection notebook to match the target urban area.

It is worth noting that OpenAQ was chosen for this dataset on the assumption that others who may wish to generate their own dataset can do so with ease. At the time of writing, OpenAQ was the most straightforward platform to work with.

### 3.2 API Access

Data was collected via the OpenAQ v3 REST API using the following endpoint pattern:

```
GET /v3/locations               — station discovery by bounding box
GET /v3/locations/{id}/sensors  — sensor enumeration per station
GET /v3/sensors/{id}/hours      — hourly aggregated measurements
```

A free API key is required and obtainable at:
https://explore.openaq.org/login

The collection notebook (`openaq_dataset_collector.ipynb`) is included in this repository and is fully reproducible. Once you have created an OpenAQ account, go to **Settings → scroll down → copy your API key** and paste it into Cell 2 of the notebook.

### 3.3 Historical Window

Data covers the calendar period 2016-03-10 to 2016-10-02. This window was selected because it represents the most recent period for which the majority of Mexico City SINAICA stations maintained continuous reporting in the OpenAQ archive. The period encompasses:

- The dry season pollution peak (March–May), characterized by elevated PM10 and PM2.5 due to reduced precipitation, high temperatures, and atmospheric thermal inversions
- The transition to the wet season (June–October), showing natural pollutant reduction patterns useful as a contrast baseline

---

## 4. Spatial Methodology — Quadrant Sampling

### 4.1 Design Rationale

Rather than random or convenience sampling, stations were selected using a stratified spatial methodology. The Mexico City Metropolitan Area was divided into four quadrants based on dominant land use, with geographic bounding boxes defined in WGS84 decimal degrees:

| Quadrant | Lat range | Lon range | Land use rationale |
|---|---|---|---|
| NW_Industrial | 19.45–19.70 N | 99.40–99.05 W | Tlalnepantla and Azcapotzalco: heavy industry, oil refining, chemical manufacturing |
| NE_Transport | 19.45–19.70 N | 99.05–98.80 W | Ecatepec corridor: major freight and commuter transport routes, peri-urban expansion |
| SW_UrbanCore | 19.20–19.50 N | 99.40–99.05 W | Historic centre and Iztapalapa: highest population density, mixed commercial and residential emissions |
| SE_Background | 19.20–19.50 N | 99.05–98.80 W | Tlahuac and Chalco: semi-rural fringe, lower anthropogenic activity; serves as spatial reference baseline |

This design enables cross-quadrant validation: a reading that appears anomalous at one station can be compared against contemporaneous readings in adjacent quadrants.

### 4.2 Station Selection

Within each bounding box, up to five candidate stations were retrieved via the OpenAQ bbox query. Candidates were pre-validated to confirm the presence of at least one criteria-pollutant sensor (PM2.5, PM10, NO2, O3, CO, or SO2). Stations with no criteria sensors were excluded.

**Final station roster (16 stations):**

| Quadrant | Stations |
|---|---|
| NW_Industrial | Atizapan, La Presa, Tultitlan, Villa de las Flores, Xalostoc |
| NE_Transport | Acolman, Los Laureles, Montecillo, San Agustin |
| SW_UrbanCore | Hospital General de Mexico, Iztacalco, Pedregal, UAM Xochimilco |
| SE_Background | Chalco, Nezahualcoyotl, Tlahuac |

---

## 5. Pollutants and Measurement Units

Six criteria pollutants are included. PM2.5 and PM10 are reported in micrograms per cubic metre (µg/m³) while gaseous pollutants are reported in parts per million (ppm), as returned by the OpenAQ API for this network:

| Column | Pollutant | Unit | WHO 24h guideline |
|---|---|---|---|
| pm25_ugm3 | Fine particles | µg/m³ | 15 µg/m³ |
| pm10_ugm3 | Coarse particles | µg/m³ | 45 µg/m³ |
| no2_ppm | Nitrogen dioxide | ppm | ~0.013 ppm (25 µg/m³) |
| o3_ppm | Ozone | ppm | ~0.051 ppm (100 µg/m³, 8h) |
| co_ppm | Carbon monoxide | ppm | ~2.09 ppm (4,000 µg/m³) |
| so2_ppm | Sulfur dioxide | ppm | ~0.015 ppm (40 µg/m³) |

> ⚠️ **Important — `who_no2_exceeded` column:**
> This column was computed by comparing NO2 values against the WHO threshold expressed in µg/m³ (25 µg/m³). Because NO2 in this dataset is correctly stored in ppm, this comparison is unit-inconsistent and the column reads 0 for all rows. It should be **disregarded**. Researchers applying a WHO-equivalent NO2 threshold should use: `no2_ppm > 0.013` (equivalent to 25 µg/m³ at standard conditions).

Not every station reports every pollutant. PM2.5 coverage is 23% across the dataset, reflecting that most SINAICA stations in 2016 primarily reported PM10 rather than PM2.5. Missing values (NaN) reflect actual sensor configuration at each location and are not a collection error — this is consistent with the heterogeneous nature of real-world IoT monitoring networks.

---

## 6. Anomaly Detection Methodology

Anomaly labels were computed using a per-station z-score method, standard practice in environmental sensor QA/QC literature:

1. For each station, compute the mean and standard deviation of PM10 readings (the most complete pollutant column at 47% coverage)
2. A reading is flagged `ANOMALOUS_SPIKE` if its z-score exceeds +3.0 standard deviations above the station mean
3. A reading is flagged `DROPOUT_SUSPECTED` if its z-score falls below -3.0 standard deviations
4. A row where all six pollutant columns are NaN simultaneously is flagged `MISSING` (communication loss)
5. A minimum standard deviation threshold of 0.5 is applied — stations with near-zero variance are not flagged, as stable readings reflect consistent sensor behaviour rather than a data-freeze fault

**Anomaly distribution in this dataset:**

| Flag | Count | Share |
|---|---|---|
| VALID | 21,367 | 99.5% |
| ANOMALOUS_SPIKE | 102 | 0.5% |

The 102 spike events are concentrated in the NW_Industrial quadrant and in the April–May dry season peak, consistent with documented thermal inversion episodes in the Mexico City basin during that period.

---

## 7. Dataset Statistics and Notable Events

**Overall:**

| Property | Value |
|---|---|
| Total rows | 21,469 |
| Total columns | 24 |
| Stations | 16 |
| Quadrants | 4 |
| Observation window | 2016-03-10 to 2016-10-02 (~7 months) |
| Temporal resolution | 1 hour |
| Sensor type | Government-grade (all stations) |
| Country | Mexico (MX) |

**Notable spike events recorded in the dataset:**

- **2016-04-05 to 2016-04-11:** Multi-station PM10 spikes across NW and SE quadrants (140–232 µg/m³), consistent with a regional dust and ozone episode in the Mexico City basin
- **2016-05-07:** Chalco (SE_Background) recorded PM10 = 400 µg/m³, the single highest reading in the dataset, with a probable cause being a localized dust storm event from the Texcoco lakebed
- **2016-04-14 to 2016-04-19:** Sustained afternoon spikes at Xalostoc (NW_Industrial, PM10 up to 232 µg/m³) coinciding with elevated CO (up to 3.8 ppm)

---

## 8. Schema Reference

| Column name | Type | Description |
|---|---|---|
| timestamp_utc | string | ISO 8601 UTC timestamp (hourly) |
| station_id | string | Unique station ID (e.g. MX1134) |
| station_name | string | Human-readable station name |
| city | string | City (all rows: Mexico City) |
| country_code | string | ISO 3166-1 alpha-2 (all rows: MX) |
| latitude | float | WGS84 decimal latitude |
| longitude | float | WGS84 decimal longitude |
| sensor_type | string | All rows: "government" |
| quadrant | string | Spatial zone label (NW/NE/SW/SE) |
| openaq_location_id | integer | OpenAQ v3 numeric location ID |
| pm25_ugm3 | float | PM2.5 in µg/m³; NaN = not measured |
| pm10_ugm3 | float | PM10 in µg/m³; NaN = not measured |
| no2_ppm | float | NO2 in ppm; NaN = not measured |
| o3_ppm | float | O3 in ppm; NaN = not measured |
| co_ppm | float | CO in ppm; NaN = not measured |
| so2_ppm | float | SO2 in ppm; NaN = not measured |
| who_pm25_exceeded | integer | 1 if PM2.5 > 15 µg/m³, else 0 |
| who_pm10_exceeded | integer | 1 if PM10 > 45 µg/m³, else 0 |
| who_no2_exceeded | integer | ⚠️ DO NOT USE — unit mismatch (see Section 5) |
| diurnal_period | string | rush_hour / daytime / night |
| is_weekend | integer | 1 if Saturday or Sunday, else 0 |
| anomaly_type | string | none / spike / dropout / missing |
| anomaly_label | integer | 0=none, 1=spike, 2=dropout, 3=missing |
| data_quality_flag | string | VALID / ANOMALOUS_SPIKE / DROPOUT_SUSPECTED / MISSING |

---

## 9. Reproducibility

The collection notebook (`openaq_dataset_collector.ipynb`) is included in this repository and reproduces the dataset with a free OpenAQ API key.

Before running, replace the `API_KEY` value in Cell 2 with your own key. Results may differ marginally from this version if OpenAQ updates its archive, but the methodology, station selection, and quadrant bounding boxes will be identical.

**Python requirements:**

```
Python   >= 3.9
pandas   >= 1.5
numpy    >= 1.23
requests >= 2.28
```

```bash
pip install pandas numpy requests
```

---

## 10. Ethical and Legal Notes

This dataset contains no personally identifiable information. All measurements are from publicly operated government monitoring infrastructure. Data was retrieved via the OpenAQ public API under its open data terms of service.

Released under **Creative Commons Zero v1.0 Universal (CC0)**.
Full license text: https://creativecommons.org/publicdomain/zero/1.0/

---

## 11. Citation

If you use this dataset, it'd be appreciated it if you could please cite it as:

Ramos, K. W. N. (2026). *Mexico City Quadrant Air Quality Dataset for Anomaly Detection — via OpenAQ, 2016* (Version 1.0.0) [Data set]. Zenodo. https://doi.org/10.5281/zenodo.XXXXXXX

**BibTeX:**

```bibtex
@dataset{ramos2026mexicocityaq,
  author    = {Ramos, Karlett Wendy Nathali},
  title     = {Mexico City Quadrant Air Quality Dataset for Anomaly Detection},
  year      = {2026},
  version   = {1.0.0},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.XXXXXXX}
}
```

---

## 12. File Manifest

| File | Description |
|---|---|
| `openaq_mxcity_monitoring_dataset.csv` | Main dataset. UTF-8, comma-separated. 21,469 rows, 24 columns. Missing values as empty fields. |
| `README.md` | This file. |
| `openaq_dataset_collector.ipynb` | Google Colab notebook used to collect and process the data. Requires a free OpenAQ API key (Cell 2). |
