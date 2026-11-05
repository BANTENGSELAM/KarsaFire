# KARSA-FIRE (Karsa Cipta Wildfire Intelligence)

> **Geospatial Hotspot Analysis & Fire-Risk Predictive Intelligence System**  
> **Phase:** `Phase 0 — Foundation Only` | **Status:** Technical Foundation Established

---

## 1. Overview

**KARSA-FIRE** is an enterprise-grade geospatial analytics and predictive intelligence system engineered for:
1. **Historical Hotspot Analytics:** Ingesting, aggregating, filtering, and visualizing satellite thermal anomalies from NASA FIRMS (MODIS, VIIRS S-NPP, NOAA-20/21) over long-term temporal horizons.
2. **Predictive Fire-Risk Intelligence:** Integrating multi-source environmental covariates (ERA5-Land reanalysis, CHIRPS precipitation, topographic and land-use indicators) to produce calibrated, spatio-temporal fire-hazard risk scores.

> [!IMPORTANT]
> **Current Project Status (Phase 0):**  
> No production code or mock predictive models are active. This phase establishes architecture specifications, data schemas, verification protocols, ML validation principles, and project roadmaps before feature implementation commences.

---

## 2. Proposed Architecture & Tech Stack

```mermaid
graph TD
    subgraph Data Sources
        FIRMS[NASA FIRMS API / S3<br/>Thermal Anomalies]
        ERA5[ECMWF ERA5-Land<br/>Weather & Soil Moisture]
        CHIRPS[CHIRPS / UCSB<br/>High-Res Precipitation]
        AUX[Land Cover / Peatland<br/>Static GeoTIFF & Shapefiles]
    end

    subgraph Data & ML Pipelines (Python)
        Ingest[Ingestion & Harmonization]
        Features[Spatial Feature Store & H3 Indexing]
        ML[Predictive Fire-Risk Pipeline<br/>Blocked Spatio-Temporal CV]
    end

    subgraph Storage & Serving
        PostGIS[(PostgreSQL 16 + PostGIS 3.4)<br/>Vector & Spatio-temporal Store]
        ObjectStore[(Object Storage / S3 / Zarr)<br/>Gridded Weather & Climatology]
        API[FastAPI Backend Service<br/>REST & Vector Tile Serving]
    end

    subgraph Client Application
        Next[Next.js App Router (TypeScript)<br/>Tailored Vanilla CSS / Tailwind UI]
        Map[Deck.gl / MapLibre GL<br/>Dynamic Vector Tiling & Heatmaps]
    end

    FIRMS --> Ingest
    ERA5 --> Ingest
    CHIRPS --> Ingest
    AUX --> Ingest
    Ingest --> Features
    Features --> PostGIS
    Features --> ObjectStore
    Features --> ML
    ML --> PostGIS
    PostGIS --> API
    ObjectStore --> API
    API --> Next
    Next --> Map
```

| Layer | Proposed Technology | Justification & Role | Status |
|---|---|---|---|
| **Web Frontend** | Next.js 14+ (App Router, TypeScript) | High-performance dashboard, SSR capabilities, rich interactive layout. | Proposed |
| **Geospatial Viz** | Deck.gl + MapLibre GL | GPU-accelerated rendering of tens of thousands of active fire points & raster overlays. | Proposed |
| **API Backend** | FastAPI (Python 3.11+) | Asynchronous spatial endpoints, OpenAPI generation, direct integration with ML models. | Proposed |
| **Spatial Database** | PostgreSQL 16 + PostGIS 3.4 | Standard for spatial indexing (GIST, BRIN), spatial aggregation, polygon clipping. | Proposed |
| **Grid Indexing** | Uber H3 (Discrete Global Grid) | Hexagonal spatial binning for consistent feature extraction and modeling cells. | Proposed |
| **ML Engine** | PyTorch / LightGBM / XGBoost | Tabular and spatio-temporal gradient boosted models; calibrated probabilistic outputs. | Under Evaluation |

---

## 3. Documentation Index

Comprehensive engineering specifications are maintained under the [`docs/`](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs) directory:

- [**docs/PRD.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/PRD.md): Product Requirements, user personas, MVP scope, explicit exclusions, facts vs. assumptions.
- [**docs/ARCHITECTURE.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/ARCHITECTURE.md): Component diagrams, data lifecycle, system trade-offs, and security topology.
- [**docs/DATA_DICTIONARY.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/DATA_DICTIONARY.md): Data schemas for FIRMS, ERA5-Land, CHIRPS, attributes, units, confidence scores, and limitations.
- [**docs/API_SPEC.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/API_SPEC.md): REST and geospatial tile endpoints, query parameters, error contracts, and pagination.
- [**docs/ML_SPEC.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/ML_SPEC.md): Prediction target formalization, temporal horizons, spatial resolution, spatial-temporal data leakage prevention, and metrics.
- [**docs/UX_SPEC.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/UX_SPEC.md): Design system, map controls, time-series scrubbers, colorblind-safe palettes, and layout wireframes.
- [**docs/ROADMAP.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/ROADMAP.md): Milestone definitions from Phase 0 Foundation to Phase 4 Operationalization.
- [**docs/PROJECT_STATE.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/PROJECT_STATE.md): Current sprint state, factual milestones achieved, and active blocker log.
- [**docs/DECISIONS.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/docs/DECISIONS.md): Architectural Decision Records (ADRs) logging key architectural choices and open decisions.
- [**CHANGELOG.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/CHANGELOG.md): Record of all versions, releases, and updates.
- [**AGENTS.md**](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/AGENTS.md): Operating instructions and rules of engagement for AI-assisted workflows.

---

## 4. Getting Started

### Prerequisites
- Node.js 18+ & npm / pnpm
- Python 3.11+ & Poetry or venv
- Docker & Docker Compose (for local PostgreSQL/PostGIS)

### Repository Structure
```text
KarsaCipta/
├── AGENTS.md               # Agent guidelines and operating laws
├── README.md               # Project overview and index
├── CHANGELOG.md            # Version change history
├── docs/                   # Architectural & engineering specifications
│   ├── PRD.md
│   ├── ARCHITECTURE.md
│   ├── DATA_DICTIONARY.md
│   ├── API_SPEC.md
│   ├── ML_SPEC.md
│   ├── UX_SPEC.md
│   ├── ROADMAP.md
│   ├── PROJECT_STATE.md
│   └── DECISIONS.md
```

### Initial Development Guidelines
1. Do not start coding application features before the user reviews and signs off on the Phase 0 foundation.
2. Read [AGENTS.md](file:///c:/Users/kenyi/OneDrive/Desktop/KarsaCipta/AGENTS.md) before executing tasks.
3. Keep all changes testable, scoped, and documented.
