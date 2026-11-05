# Data Dictionary & Schema Specification — KARSA-FIRE

> **Document Version:** 1.0.0-draft  
> **Status:** Specification (Phase 0)  
> **Coordinate Reference System Default:** WGS 84 (`EPSG:4326`) for geometries; Web Mercator (`EPSG:3857`) for map tile reprojection.

---

## 1. Primary Remote Sensing Datasets

### 1.1 NASA FIRMS Active Fire Data (Thermal Anomalies)

NASA Fire Information for Resource Management System (FIRMS) delivers near-real-time and archived active fire pixel detections.

| Dataset Key | Sensor & Satellite | Spatial Resolution | Temporal Resolution / Pass | Primary Strengths & Limitations |
|---|---|---|---|---|
| `firms_viirs_snpp` | VIIRS (Suomi-NPP) | $375\text{ m}$ (I-Band) | Twice daily (ascending & descending) | High spatial resolution; detects small/cool fires; orbital drift observed late in satellite lifecycle. |
| `firms_viirs_noaa20` | VIIRS (NOAA-20 / JPSS-1) | $375\text{ m}$ (I-Band) | Twice daily (alternates with S-NPP ~50 min gap) | Modern calibration; consistent orbit; complement to S-NPP. |
| `firms_viirs_noaa21` | VIIRS (NOAA-21 / JPSS-2) | $375\text{ m}$ (I-Band) | Twice daily | Newest sensor; useful for future data continuity. |
| `firms_modis` | MODIS (Terra & Aqua) | $1\text{ km}$ | Terra (~10:30 AM/PM) & Aqua (~1:30 AM/PM) | 20+ year baseline (from 2000/2002); coarse spatial resolution; saturation on large fires. |

#### Attribute Dictionary: VIIRS Active Fire Detection (`375m`)
| Field Name | Type | Unit | Nullable | Description & Domain Rules |
|---|---|---|---|---|
| `latitude` | `DOUBLE PRECISION` | Degrees | No | Latitude of active fire pixel center (WGS 84, $-90.0$ to $+90.0$). |
| `longitude` | `DOUBLE PRECISION` | Degrees | No | Longitude of active fire pixel center (WGS 84, $-180.0$ to $+180.0$). |
| `bright_ti4` | `DOUBLE PRECISION` | Kelvin (K) | No | VIIRS I-4 thermal brightness temperature ($3.74\ \mu\text{m}$ channel). Saturated at ~367 K. |
| `scan` | `DOUBLE PRECISION` | Kilometers | No | Along-scan pixel dimension ($0.32$ to $0.80\text{ km}$). |
| `track` | `DOUBLE PRECISION` | Kilometers | No | Along-track pixel dimension ($0.32$ to $0.80\text{ km}$). |
| `acq_date` | `DATE` | YYYY-MM-DD | No | Acquisition date in UTC. |
| `acq_time` | `VARCHAR(4)` | HHMM | No | Acquisition time in UTC (e.g., `'0615'`). |
| `satellite` | `VARCHAR(10)` | Code | No | Satellite identifier: `'N'` (Suomi-NPP), `'1'` (NOAA-20), `'2'` (NOAA-21). |
| `confidence` | `VARCHAR(10)` | Category | No | Categorical confidence assessment: `'l'` (low), `'n'` (nominal), `'h'` (high). |
| `version` | `VARCHAR(10)` | String | No | Processing algorithm collection version (e.g., `'2.0NRT'`, `'C1'`). |
| `bright_ti5` | `DOUBLE PRECISION` | Kelvin (K) | No | VIIRS I-5 thermal brightness temperature ($11.45\ \mu\text{m}$ channel). |
| `frp` | `DOUBLE PRECISION` | Megawatts (MW) | Yes | Fire Radiative Power calculated from MWIR radiance. Measures instantaneous radiant heat release. |
| `daynight` | `CHAR(1)` | Code | No | Acquisition condition: `'D'` (Daytime), `'N'` (Nighttime). |

> [!WARNING]
> **Hotspot Fact vs. Ground Reality:**  
> Detections in `confidence = 'l'` contain significant sun glint, bright sand, and sensor noise artifacts. For operational fire monitoring, standard practice filters for `confidence IN ('n', 'h')` unless conducting ultra-sensitive smoldering detection.

---

### 1.2 ECMWF ERA5-Land Meteorological Reanalysis

ERA5-Land provides hourly and daily global atmospheric and land-surface reanalysis at $\sim 9\text{ km}$ ($0.1^\circ \times 0.1^\circ$) spatial resolution.

| Variable Name | NetCDF Parameter | Physical Unit | Description & Role in Fire Modeling |
|---|---|---|---|
| `2m_temperature` | `t2m` | Kelvin (K) | Air temperature at 2 meters. Higher temperatures correlate with elevated fuel drying. |
| `2m_dewpoint_temperature` | `d2m` | Kelvin (K) | Dewpoint temperature at 2 meters. Used in conjunction with `t2m` to compute Relative Humidity ($RH$). |
| `10m_u_component_of_wind` | `u10` | $\text{m/s}$ | Zonal wind speed component (east-west). |
| `10m_v_component_of_wind` | `v10` | $\text{m/s}$ | Meridional wind speed component (north-south). Combined with `u10` to get wind magnitude and direction. |
| `volumetric_soil_water_layer_1` | `swvl1` | $\text{m}^3/\text{m}^3$ | Topsoil moisture ($0 - 7\text{ cm}$). Rapid indicator of surface organic layer drought. |
| `volumetric_soil_water_layer_2` | `swvl2` | $\text{m}^3/\text{m}^3$ | Subsurface moisture ($7 - 28\text{ cm}$). Critical for deep peat smoldering vulnerability. |
| `surface_solar_radiation_downwards` | `ssrd` | $\text{J/m}^2$ | Solar insolation driving fuel evaporation and atmospheric instability. |

---

### 1.3 CHIRPS (Climate Hazards Group InfraRed Precipitation with Station data)

High-resolution precipitation dataset combining satellite thermal infrared imagery with ground station rain gauges.

| Field | Type | Unit | Description |
|---|---|---|---|
| `precip` | `FLOAT4` | $\text{mm/day}$ | Daily total precipitation at $0.05^\circ \times 0.05^\circ$ ($\sim 5.5\text{ km}$) resolution. |
| `dry_spell_days` | `INT2` | Days | Derived counter: consecutive days with $\text{precip} < 1.0\text{ mm}$. Key covariate for fire ignition risk. |
| `spi_30` | `FLOAT4` | Z-Score | Derived Standardized Precipitation Index over 30-day rolling accumulation. |

---

## 2. System Database Tables (PostgreSQL / PostGIS)

### 2.1 Table: `hotspots` (Partitioned by Year/Month)
```sql
CREATE TABLE hotspots (
    id BIGSERIAL,
    source_sensor VARCHAR(16) NOT NULL, -- 'VIIRS_SNPP', 'VIIRS_NOAA20', 'MODIS'
    geom GEOMETRY(Point, 4326) NOT NULL,
    latitude DOUBLE PRECISION NOT NULL,
    longitude DOUBLE PRECISION NOT NULL,
    acq_date DATE NOT NULL,
    acq_time VARCHAR(4) NOT NULL,
    confidence_level VARCHAR(10) NOT NULL, -- 'low', 'nominal', 'high'
    confidence_numeric SMALLINT,           -- 0-100 for MODIS
    bright_ti4 DOUBLE PRECISION,
    bright_ti5 DOUBLE PRECISION,
    frp DOUBLE PRECISION,
    daynight CHAR(1) NOT NULL,            -- 'D', 'N'
    h3_res7 VARCHAR(15) NOT NULL,          -- H3 Index Resolution 7
    h3_res8 VARCHAR(15) NOT NULL,          -- H3 Index Resolution 8
    admin_id VARCHAR(32),                 -- Administrative Boundary Code (e.g. BPS code)
    is_peatland BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (id, acq_date)
) PARTITION BY RANGE (acq_date);
```

### 2.2 Table: `h3_daily_aggregates`
Pre-aggregated rollups computed daily to serve real-time dashboard statistics and map tiles without scanning raw point partitions.

```sql
CREATE TABLE h3_daily_aggregates (
    h3_index VARCHAR(15) NOT NULL,
    date DATE NOT NULL,
    hotspot_count INTEGER NOT NULL,
    high_conf_count INTEGER NOT NULL,
    sum_frp DOUBLE PRECISION NOT NULL,
    max_frp DOUBLE PRECISION NOT NULL,
    avg_brightness DOUBLE PRECISION NOT NULL,
    day_count SMALLINT NOT NULL,
    night_count SMALLINT NOT NULL,
    PRIMARY KEY (h3_index, date)
);
```

### 2.3 Table: `fire_risk_predictions`
Stores spatial-temporal model outputs and contributing feature attributions.

```sql
CREATE TABLE fire_risk_predictions (
    id BIGSERIAL PRIMARY KEY,
    model_version VARCHAR(32) NOT NULL,
    prediction_date DATE NOT NULL,       -- Target forecast date
    horizon_hours SMALLINT NOT NULL,     -- 24, 72, 168
    h3_index VARCHAR(15) NOT NULL,
    risk_score FLOAT NOT NULL,           -- Calibrated probability [0.0 - 1.0]
    risk_category VARCHAR(12) NOT NULL,  -- 'LOW', 'MODERATE', 'HIGH', 'EXTREME'
    feature_attributions JSONB,          -- Key SHAP values or covariate contributions
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 3. Data Ingestion & Transformation Validation Rules

1. **Coordinate Bounding Box Sanity:** All ingested points must fall within valid coordinate bounds: $\text{lat} \in [-90, +90]$, $\text{lon} \in [-180, +180]$. Points outside the defined pilot region (e.g., Indonesia: $\text{lat} \in [-11.0, 6.0]$, $\text{lon} \in [95.0, 141.0]$) are tagged or filtered out.
2. **Missing FRP Imputation:** In certain satellite passes, FRP can be null or negative due to sensor scan geometry. Raw null values are preserved in storage, but imputed to $0.0$ or handled explicitly in statistical aggregation routines.
3. **Time Normalization:** All acquisition timestamps are ingested in UTC and indexed with explicit UTC date-times. Local time conversion (e.g., UTC+7 for Western Indonesia Time / WIB) is performed as a presentation-layer transform.
