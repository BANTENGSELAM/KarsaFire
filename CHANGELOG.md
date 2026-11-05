# Changelog — KARSA-FIRE

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.1.0-alpha.0] - 2026-10-26

### Added
- **System Governance & Operating Laws (`AGENTS.md`):**
  - Defined 6 specialized AI agent skills (`karsa-planner`, `karsa-geo`, `karsa-ml`, `karsa-frontend`, `karsa-qa`, `karsa-docs`).
  - Formalized non-negotiable engineering laws regarding empirical data integrity, machine learning validation restraint, and architectural discipline.
  - Established a standard 6-stage task execution workflow.
- **Project Overview & Index (`README.md`):**
  - Authored comprehensive project overview, multi-tier system topology diagram, tech stack evaluation, and documentation index.
- **Product Requirements Document (`docs/PRD.md`):**
  - Structured requirements framework strictly categorizing Facts, Proposals, Assumptions, and Open Decisions.
  - Defined primary user personas (Disaster Management Analyst, Field Patrol Coordinator, Climate Researcher).
  - Explicitly demarcated MVP boundaries and out-of-scope capabilities.
- **System Architecture Specification (`docs/ARCHITECTURE.md`):**
  - Designed decoupled 4-tier architecture (Data Pipeline, Persistence Tier, Backend Service, Client Application).
  - Completed detailed trade-off analyses (Next.js vs. Vite, Deck.gl/MapLibre vs. Leaflet, H3 DGGS vs. Quadtrees, PostGIS MVT vs. GeoJSON).
  - Addressed spatial denial-of-service (Geo-DoS) mitigation, partitioning, and zero-trust security.
- **Data Dictionary & Schema Specification (`docs/DATA_DICTIONARY.md`):**
  - Documented complete schemas for NASA FIRMS (VIIRS 375m & MODIS 1km), ECMWF ERA5-Land reanalysis, and CHIRPS precipitation.
  - Authored DDL schemas for PostgreSQL/PostGIS partitioned tables (`hotspots`, `h3_daily_aggregates`, `fire_risk_predictions`).
  - Defined semantic confidence rules separating sensor artifacts from high-confidence thermal events.
- **Backend API Specification (`docs/API_SPEC.md`):**
  - Specified OpenAPI 3.1 REST endpoints and Mapbox Vector Tile (`.pbf`) routes.
  - Documented query parameters, bounding box limits, rate limiting, and RFC-7807 error envelopes.
- **Machine Learning Specification (`docs/ML_SPEC.md`):**
  - Formalized candidate prediction targets (Binary Occurrence, FRP Exceedance, Poisson Count).
  - Defined candidate forecast horizons (24h, 72h, 7d) and H3 spatial resolutions (Res 7 vs. Res 8).
  - Mandated Purged Blocked Spatio-Temporal Cross Validation to eliminate temporal and spatial data leakage.
  - Formulated baseline heuristics (Persistence and Simple FWI) and evaluation metrics (PR-AUC, Brier score, calibration curves).
- **User Experience & Interface Specification (`docs/UX_SPEC.md`):**
  - Specified layout architecture including map canvas, floating layer manager, timeline playback scrubber, and cell inspection drawer.
  - Established modern dark-mode design tokens and colorblind-safe hazard palettes (avoiding red-green confusion).
  - Defined automated testing identifiers (`data-testid`).
- **Implementation Roadmap (`docs/ROADMAP.md`):**
  - Mapped implementation journey across Phases 0 through 4 with explicit entry/exit criteria and phase gates.
- **Architecture Decision Records (`docs/DECISIONS.md`):**
  - Recorded foundational decisions ADR-001 through ADR-007.
- **Project State Ledger (`docs/PROJECT_STATE.md`):**
  - Established persistent tracking of completed actions, facts, open blockers, and next steps.
