# Architecture Decision Records (ADR) — KARSA-FIRE

> **Repository:** KARSA-FIRE  
> **Status:** Active  
> **Format:** Modified Michael Nygard ADR Template

---

## Index of Decisions

- [ADR-001: Strict Separation of Foundation Phase (Phase 0) from Feature Implementation](#adr-001-strict-separation-of-foundation-phase-phase-0-from-feature-implementation)
- [ADR-002: Next.js (TypeScript) and Deck.gl / MapLibre GL for Geospatial Client](#adr-002-nextjs-typescript-and-deckgl--maplibre-gl-for-geospatial-client)
- [ADR-003: FastAPI & PostgreSQL/PostGIS for Spatial Vector & Analytics Backend](#adr-003-fastapi--postgresqlpostgis-for-spatial-vector--analytics-backend)
- [ADR-004: Adoption of Uber H3 Hexagonal Discrete Global Grid System (DGGS)](#adr-004-adoption-of-uber-h3-hexagonal-discrete-global-grid-system-dggs)
- [ADR-005: Strict Blocked Spatio-Temporal Validation to Prevent Data Leakage](#adr-005-strict-blocked-spatio-temporal-validation-to-prevent-data-leakage)
- [ADR-006: Explicit Semantic Distinction Between Satellite Thermal Anomalies and Confirmed Wildfires](#adr-006-explicit-semantic-distinction-between-satellite-thermal-anomalies-and-confirmed-wildfires)
- [ADR-007: Open Architectural Decisions Pending User Clarification](#adr-007-open-architectural-decisions-pending-user-clarification)

---

## ADR-001: Strict Separation of Foundation Phase (Phase 0) from Feature Implementation

### Status
Accepted

### Context
Geospatial and machine learning systems for natural hazard modeling carry significant risk of premature optimization, data contamination, and ungrounded architectural complexity if development begins before boundaries, validation methodologies, and data contracts are formalized.

### Decision
Enforce a strict Phase 0 (Foundation Only) checkpoint. No production code, mock models, or simulated predictions are implemented until all foundational specifications (PRD, Architecture, Data Dictionary, API Spec, ML Spec, UX Spec, and Roadmap) are authored and reviewed by the project lead.

### Consequences
- **Positive:** Eliminates technical debt, prevents hallucinated metrics, ensures clean workspace preservation.
- **Negative:** Requires initial upfront documentation effort before visual feature demonstration.

---

## ADR-002: Next.js (TypeScript) and Deck.gl / MapLibre GL for Geospatial Client

### Status
Proposed (Awaiting Phase 0 signoff)

### Context
Rendering tens of thousands of historical thermal anomaly points alongside dynamic hexagonal density grids on a browser map requires hardware-accelerated graphics. Traditional DOM-based mapping libraries (such as Leaflet) choke when rendering $>5,000$ vector elements.

### Decision
Adopt Next.js 14+ (App Router, TypeScript) for the web application shell and Deck.gl paired with MapLibre GL for the mapping canvas.

### Consequences
- **Positive:** Deck.gl utilizes WebGL2 GPU buffers, rendering 100,000+ points smoothly at 60 FPS. MapLibre GL provides open-source, vector-tiled basemaps without vendor lock-in or proprietary API fees. Next.js delivers type-safe routing, SSR for landing/analytics summaries, and modular bundling.
- **Negative:** Steeper learning curve for WebGL buffer management and custom Deck.gl layer shaders compared to simple Leaflet setups.

---

## ADR-003: FastAPI & PostgreSQL/PostGIS for Spatial Vector & Analytics Backend

### Status
Proposed (Awaiting Phase 0 signoff)

### Context
The backend must handle high-volume spatio-temporal queries, bounding-box filtering, vector tile generation (`ST_AsMVT`), and seamless integration with Python-based geospatial and machine learning workflows (GeoPandas, Xarray, LightGBM).

### Decision
Select FastAPI (Python 3.11+) as the REST/Tile API service and PostgreSQL 16 + PostGIS 3.4 as the spatial persistence engine.

### Consequences
- **Positive:** Native PostGIS spatial indexing (`GIST`, `BRIN`) and MVT generation; Python enables direct sharing of schemas and data structures between the API and ML pipeline; FastAPI offers high-throughput asynchronous request handling and automated OpenAPI generation.
- **Negative:** Requires managing a relational database with PostGIS extensions rather than a simple static file server.

---

## ADR-004: Adoption of Uber H3 Hexagonal Discrete Global Grid System (DGGS)

### Status
Proposed (Awaiting Phase 0 signoff)

### Context
Raw satellite active-fire observations are point events distributed irregularly in space and time based on orbital swaths. Aggregating these points and harmonizing them with gridded climate reanalysis (ERA5-Land $\approx 9\text{ km}$, CHIRPS $\approx 5.5\text{ km}$) requires a standardized spatial tessellation.

### Decision
Adopt Uber H3 hexagonal discrete global grid system (Resolution 7 $\approx 5.16\text{ km}^2$ or Resolution 8 $\approx 0.74\text{ km}^2$) as the core analytical and modeling cell unit.

### Consequences
- **Positive:** Hexagonal cells feature equidistant neighbors (all 6 adjacent cells share equal distances), eliminating diagonal neighbor artifacts inherent to rectangular grids; hierarchical indexing allows fast integer-based spatial lookups and multiscale aggregation.
- **Negative:** Requires coordinate-to-H3 conversion on ingestion; spatial boundaries do not align directly with rectangular raster pixels without spatial interpolation.

---

## ADR-005: Strict Blocked Spatio-Temporal Validation to Prevent Data Leakage

### Status
Accepted

### Context
Standard random K-fold cross-validation on spatial-temporal data produces severe data leakage due to spatial and temporal autocorrelation, resulting in artificially high model accuracy metrics that fail in real-world deployment.

### Decision
Mandate Purged Blocked Spatio-Temporal Cross Validation:
1. Temporal partitioning: Models must be trained exclusively on historical years and tested on subsequent unseen future years (e.g. Train: 2018–2022, Validate: 2023, Test: 2024).
2. Spatial partitioning: Geographic regions must be blocked and buffered (no training cells within 50 km of validation cells when assessing regional generalization).
3. 7-day temporal embargo buffer between train and test windows.

### Consequences
- **Positive:** Guarantees honest, realistic generalization metrics; protects against deploying dangerously misleading hazard models.
- **Negative:** Validation scores will be lower than naive random splits; requires rigorous pipeline orchestration.

---

## ADR-006: Explicit Semantic Distinction Between Satellite Thermal Anomalies and Confirmed Wildfires

### Status
Accepted

### Context
Non-specialist users and naive implementations often conflate satellite hotspot detections with confirmed, active ground wildfires. Satellite sensors detect mid-infrared radiation spikes, which include gas flares, volcanic activity, controlled agricultural burns, and solar glint reflections.

### Decision
1. System UI, data dictionaries, and APIs must label raw NASA FIRMS data explicitly as **Thermal Anomalies / Hotspots**, never as "Confirmed Fires".
2. Confidence levels (Low, Nominal, High) must be visibly surfaced.
3. Fire-risk predictions must be explicitly framed as **Fire Hazard / Occurrence Probability**, not certainty.

### Consequences
- **Positive:** Scientific accuracy and credibility; prevents operational misallocation of firefighting resources.
- **Negative:** Requires clearer UI tooltips and explanatory disclaimers.

---

## ADR-007: Open Architectural Decisions Pending User Clarification

### Status
Open (Awaiting User Input)

### Pending Questions:
1. **Pilot Study Area:** What is the approved pilot region for initial data ingestion and model benchmarking?
   - *Option A:* Riau Province (Sumatra) — High peatland fire frequency, dense historical record.
   - *Option B:* South Sumatra Province — Extensive peat and plantation landscape.
   - *Option C:* Central Kalimantan (Borneo) — Major peat dome region.
   - *Option D:* National Indonesia Extent.
2. **Machine Learning Target & Temporal Horizon:**
   - What is the operational forecast horizon: 24 Hours, 72 Hours, or 7 Days?
   - Is binary occurrence in cell $c$ acceptable for the MVP baseline?
3. **Deployment Strategy:**
   - Docker Compose on local workstation vs. cloud-hosted managed services (AWS / GCP).
