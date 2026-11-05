# Project State — KARSA-FIRE

> **Current Sprint / Phase:** Phase 0 — Foundation & Architecture Governance  
> **Status:** Foundation Complete — Awaiting User Review & Signoff  
> **Last Synchronized:** 2026-10-26

---

## 1. Executive Summary

The KARSA-FIRE technical foundation has been established from a clean workspace state. Complete architectural, data engineering, machine learning validation, API, and user experience specifications have been authored and placed under version control. No application code or simulated ML models have been written, preserving full scientific and architectural integrity prior to stakeholder review.

---

## 2. Fact Sheet & Current Baseline

- **Repository Root:** `c:\Users\kenyi\OneDrive\Desktop\KarsaCipta`
- **Git State:** Clean initialized repository on branch `main` (0 commits prior to Phase 0).
- **Existing Files Preserved:** Workspace had no legacy code files (`.dist` directory was empty).
- **Core Technology Decisions Proposed:**
  - Frontend: Next.js 14+ (TypeScript), MapLibre GL, Deck.gl
  - Backend: FastAPI (Python 3.11+), Pydantic v2
  - Spatial Store: PostgreSQL 16 + PostGIS 3.4 + H3-pg extension
  - ML Pipeline: LightGBM / XGBoost with Purged Spatio-Temporal CV

---

## 3. Completed Actions Log (Phase 0)

| Timestamp | Document / Milestone | Summary of Action |
|---|---|---|
| 2026-10-26 | Workspace Assessment | Inspected repository structure; identified empty `.dist` and clean git state. Verified absence of existing files requiring preservation. |
| 2026-10-26 | `AGENTS.md` | Defined 6 agent skills (`karsa-planner`, `karsa-geo`, `karsa-ml`, `karsa-frontend`, `karsa-qa`, `karsa-docs`), non-negotiable engineering laws, and standard 6-stage task workflow. |
| 2026-10-26 | `README.md` | Created project overview, architecture diagram, tech stack matrix, and documentation navigation index. |
| 2026-10-26 | `docs/PRD.md` | Formulated Product Requirements Document; separated facts, proposals, assumptions, and open decisions; detailed user personas and MVP boundaries. |
| 2026-10-26 | `docs/ARCHITECTURE.md` | Designed decoupled 4-tier architecture; completed comprehensive trade-off evaluations (Deck.gl vs. Leaflet, H3 vs. Quadtree, PostGIS MVT vs. GeoJSON). |
| 2026-10-26 | `docs/DATA_DICTIONARY.md` | Documented schemas for NASA FIRMS (VIIRS 375m / MODIS 1km), ERA5-Land reanalysis, CHIRPS rainfall, and PostGIS partitioned table structures. |
| 2026-10-26 | `docs/API_SPEC.md` | Authored OpenAPI 3.1 REST & Vector Tile endpoints, rate limits, error schemas, and bounding box constraints. |
| 2026-10-26 | `docs/ML_SPEC.md` | Formalized prediction targets, candidate horizons (24h/72h/7d), feature dictionary, blocked spatio-temporal cross-validation, and baseline benchmarks. |
| 2026-10-26 | `docs/UX_SPEC.md` | Specified map layout, timeline scrubber, cell inspection drawer, dark-mode design tokens, and colorblind-safe hazard palettes. |
| 2026-10-26 | `docs/ROADMAP.md` | Defined Phase 0 through Phase 4 milestones, entry/exit criteria, and quality gates. |
| 2026-10-26 | `docs/DECISIONS.md` | Initialized Architectural Decision Records (ADR-001 through ADR-007). |
| 2026-10-26 | `CHANGELOG.md` | Initialized semantic version changelog under `0.1.0-alpha.0`. |

---

## 4. Active Blockers & Decisions Requiring User Input

1. **Pilot Study Area Selection:**
   - *Status:* Pending User Input.
   - *Options:* (A) Riau Province, (B) South Sumatra, (C) Central Kalimantan, (D) Custom Region.
2. **ML Prediction Target Formalization:**
   - *Status:* Pending User Input.
   - *Options:* (A) Binary Hotspot Occurrence in Cell $c$ over $24\text{h}$, (B) High-Confidence Severe Fire ($FRP \ge 20\text{ MW}$), (C) Continuous FRP Density.
3. **Primary Forecast Horizon:**
   - *Status:* Pending User Input.
   - *Options:* (A) 24 Hours, (B) 72 Hours, (C) 7 Days.
4. **Local Development Container Strategy:**
   - *Status:* Pending User Input.
   - *Options:* Docker Compose for PostGIS & backend, vs. external cloud-hosted database.

---

## 5. Single Recommended Next Task

**Task:** User review of the Phase 0 Foundation deliverables and selection of the Pilot Study Area and Target Horizon to unblock Phase 1 (Data Ingestion & PostGIS Setup).
