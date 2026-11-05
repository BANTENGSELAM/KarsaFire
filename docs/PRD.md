# Product Requirements Document (PRD) — KARSA-FIRE

> **Document Version:** 1.0.0-draft  
> **Status:** Under Review (Phase 0)  
> **Target System:** KARSA-FIRE (Geospatial Wildfire & Hotspot Predictive Intelligence)

---

## 1. Problem Statement & Mission

Forest and land fires (karhutla) cause catastrophic ecological, economic, and respiratory health damage across tropical ecosystems and fire-prone landscapes. Existing public monitoring tools frequently present isolated thermal anomalies without comprehensive temporal aggregation, environmental context (soil moisture, rainfall deficits, wind vectors), or calibrated forward-looking predictive risk assessments.

**KARSA-FIRE** provides an integrated geospatial analytics and decision-support platform designed to:
1. Provide intuitive, high-performance exploration of historical thermal anomalies (hotspots) from NASA FIRMS.
2. Deliver spatial-temporal aggregations across administrative boundaries and standardized discrete hexagonal grids (Uber H3).
3. Compute and visualize forward-looking fire-risk scores derived from multi-source environmental covariates.

---

## 2. Requirements Categorization Framework

To prevent premature implementation and hallucinations, all statements in this document are strictly categorized:

| Category | Definition |
|---|---|
| **FACT** | Established truth verified against physical/technical constraints or official datasets. |
| **PROPOSAL** | Architecturally reasoned recommendation pending formal user validation. |
| **ASSUMPTION** | Working premise used to design baseline specifications; subject to revision upon user input. |
| **OPEN DECISION** | Unresolved parameter or policy requiring explicit stakeholder determination. |

### 2.1 Verified Facts
- **FACT 1:** NASA FIRMS detects thermal anomalies (active fire pixels) using satellite sensors (MODIS on Terra/Aqua; VIIRS on Suomi-NPP, NOAA-20, NOAA-21).
- **FACT 2:** Satellite thermal anomalies represent radiant heat signatures and do not guarantee an active, ground-confirmed uncontrolled wildfire (can be industrial flares, geothermal activity, or prescribed burning).
- **FACT 3:** VIIRS 375m spatial resolution provides significantly higher hotspot spatial fidelity than MODIS 1km resolution.
- **FACT 4:** ERA5-Land reanalysis products have an inherent latency (typically 5 days to 2–3 months for final quality-controlled data, or ERA5T near-real-time with ~5 days latency), meaning zero-day real-time weather requires short-term NWP forecasts (e.g., GFS or ECMWF IFS).
- **FACT 5:** CHIRPS rainfall estimates have a 1-2 day to semi-monthly release cycle depending on the preliminary vs. final product tier.

### 2.2 Proposals
- **PROPOSAL 1:** Use Uber H3 resolution 7 (~5.16 km² hexagon area, edge length ~1.22 km) or resolution 8 (~0.74 km², edge length ~461 m) as the standard predictive and aggregation grid cell.
- **PROPOSAL 2:** Construct the client application with Next.js 14 (App Router, TypeScript) and Deck.gl / MapLibre GL for client-side spatial rendering.
- **PROPOSAL 3:** Implement the backend as a modular FastAPI microservice exposing GeoJSON, Mapbox Vector Tiles (MVT), and tabular aggregates backed by PostgreSQL/PostGIS.

### 2.3 Working Assumptions
- **ASSUMPTION 1:** The primary geographic focus is tropical Southeast Asia / Indonesia (specifically high-risk peatland provinces such as Riau, South Sumatra, Central Kalimantan, or West Kalimantan).
- **ASSUMPTION 2:** The target user base possesses basic GIS literacy (understands bounding boxes, time ranges, confidence thresholds, and risk score heatmaps).
- **ASSUMPTION 3:** The user environment supports modern WebGL2-compatible desktop browsers (Chrome, Firefox, Safari, Edge).

### 2.4 Open Decisions
- **DECISION 1:** What is the approved pilot study area (Specific province, e.g., Riau vs. South Sumatra vs. all of Sumatra/Kalimantan vs. national scope)?
- **DECISION 2:** What is the prediction target definition for ML (Binary classification of hotspot occurrence in cell $c$ over window $t+\Delta t$, or continuous Fire Radiative Power (FRP) density)?
- **DECISION 3:** What is the operational forecast horizon ($\Delta t = 24\text{ hours}$, $72\text{ hours}$, or $7\text{ days}$)?
- **DECISION 4:** What is the production deployment target (Self-hosted on-premise Docker, AWS, GCP, or Azure)?

---

## 3. User Personas & Use Cases

### Persona 1: Disaster Management / Environmental Analyst
- **Profile:** GIS specialist or operational analyst at a regional disaster mitigation agency (e.g., BPBD/BNPB or environmental NGO).
- **Goals:** Monitor active hotspots, identify clusters crossing conservation or peatland boundaries, generate weekly historical trend reports, and anticipate high-risk zones for ground patrol deployment.
- **Pain Points:** Slow map rendering on large datasets; lack of integrated weather and soil moisture context; difficulty exporting standardized summaries.

### Persona 2: Field Patrol Coordinator / Forestry Officer
- **Profile:** Operational team lead allocating patrol and suppression teams in the field.
- **Goals:** Rapidly check which sub-districts or grid cells have high fire-risk probability in the next 24 to 72 hours to stage firefighting equipment proactively.
- **Pain Points:** Cluttered user interfaces with confusing terminology; non-transparent "black-box" risk numbers without explanatory factors (e.g., dryness, lack of rain).

### Persona 3: Data Science / Climate Researcher
- **Profile:** Quantitative researcher assessing climate trends and model fidelity.
- **Goals:** Access documented feature dictionaries, verify spatio-temporal validation metrics, inspect spatial cross-validation folds, and export clean historical data tables.
- **Pain Points:** Unclear label definitions, spatial-temporal data leakage in published models, lack of confidence intervals.

---

## 4. Scope & System Boundaries

### 4.1 In-Scope for Minimum Viable Product (MVP)
1. **Historical Hotspot Explorer:**
   - Ingestion of NASA FIRMS active fire data (VIIRS S-NPP / NOAA-20 and MODIS) for the designated study area.
   - Spatial filtering (Administrative boundary hierarchy: Province -> Regency/Kabupaten -> District/Kecamatan; or Bounding Box).
   - Temporal filtering (Date-range selector, monthly/annual views, animation scrubber over historical timeline).
   - Confidence and sensor filtering (VIIRS: Low, Nominal, High; MODIS: 0–100%).
2. **Hexagonal Spatial Aggregation (H3):**
   - On-the-fly and pre-aggregated count/density and cumulative Fire Radiative Power (FRP) per H3 hexagon.
   - Interactive drill-down from macro-region to individual grid cells.
3. **Fire-Risk Visualization (Baseline & Pilot ML):**
   - Display of fire-hazard risk scores (0.0 to 1.0 or categorized Low / Moderate / High / Extreme).
   - Inspection panel detailing driving factors for selected cells (e.g., cumulative dry days, precipitation deficit, peatland presence).
4. **Summary Analytics Dashboard:**
   - Time-series chart of hotspot counts over time.
   - Top vulnerable administrative regions ranking table.
   - Data export in standardized formats (GeoJSON, CSV).

### 4.2 Explicitly Out-of-Scope for MVP
- Real-time satellite downlinking or direct ground station processing.
- Autonomous dispatch or automated IoT siren/alarm integration.
- Smoke dispersion and atmospheric particulate ($PM_{2.5}$) plume dynamics modeling.
- Native mobile applications (iOS/Android) — mobile will be supported via responsive web design only.
- Multi-tenant enterprise RBAC with complex organizational permission trees (MVP will feature single-tier authenticated access or public read-only mode).
- Drone flight path routing or direct telemetry streaming.

---

## 5. Non-Functional Requirements (NFRs)

| NFR Metric | Target Specification | Validation Method |
|---|---|---|
| **Map Vector Render Latency** | $\le 100\text{ ms}$ frame time when rendering $\le 50,000$ points via Deck.gl / WebGL. | Browser performance profiler |
| **Spatial API Response Time** | $\le 400\text{ ms}$ for typical regional aggregation queries ($< 1\text{ year}$ range). | Automated load test (k6 / locust) |
| **Data Integrity & Traceability** | 100% of displayed points must trace back to raw sensor acquisition timestamp, satellite ID, and coordinates. | Automated schema & audit validation |
| **Accessibility & Design** | WCAG 2.1 Level AA compliance, including colorblind-safe color maps (Viridis / Plasma / ColorBrewer palettes for risk/hotspots). | Axe-core accessibility audit |
| **System Availability** | 99.5% uptime during operational testing. | Health check endpoint monitoring |
