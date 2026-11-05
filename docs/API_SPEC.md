# API Specification — KARSA-FIRE Backend Service

> **Specification Standard:** OpenAPI 3.1.0  
> **Base URL:** `/api/v1`  
> **Content-Type Default:** `application/json` (Spatial: `application/geo+json` or `application/x-protobuf`)  
> **Status:** Proposal Under Review (Phase 0)

---

## 1. Global Conventions & Error Schema

### 1.1 Standard Error Envelope
All error responses return a standardized RFC-7807 compatible JSON structure:

```json
{
  "type": "https://karsa-fire.org/errors/invalid-parameter",
  "title": "Invalid Spatial Bounding Box",
  "status": 400,
  "detail": "The requested bounding box area exceeds the maximum allowed limit of 250,000 sq km.",
  "instance": "/api/v1/hotspots/points?bbox=...",
  "timestamp": "2026-10-26T14:30:00Z"
}
```

### 1.2 Common Query Parameters
- `bbox`: Comma-delimited coordinates `min_lon,min_lat,max_lon,max_lat` (EPSG:4326). Example: `100.5,-1.2,104.8,2.1`.
- `start_date`: ISO 8601 date string (`YYYY-MM-DD`). Default: 30 days prior to current date.
- `end_date`: ISO 8601 date string (`YYYY-MM-DD`). Default: current date.
- `confidence`: Array or comma-delimited strings: `low,nominal,high`. Default: `nominal,high`.
- `satellite`: Comma-delimited list: `VIIRS_SNPP,VIIRS_NOAA20,MODIS`. Default: all.

---

## 2. Core Endpoint Specifications

### 2.1 System Health & Metadata
#### `GET /api/v1/health`
- **Purpose:** Service liveness and dependency status probe.
- **Response (200 OK):**
```json
{
  "status": "healthy",
  "timestamp": "2026-10-26T14:30:00Z",
  "version": "0.1.0-alpha.0",
  "database": {
    "status": "connected",
    "postgis_version": "3.4.0",
    "latest_hotspot_date": "2026-10-25"
  }
}
```

---

### 2.2 Hotspot Observation Endpoints

#### `GET /api/v1/hotspots/points`
- **Purpose:** Retrieve individual thermal anomaly points within a restricted spatial-temporal window.
- **Query Parameters:**
  - `bbox` (required): `min_lon,min_lat,max_lon,max_lat`
  - `start_date` (optional): `YYYY-MM-DD`
  - `end_date` (optional): `YYYY-MM-DD`
  - `min_frp` (optional, float): Minimum Fire Radiative Power (MW)
  - `confidence` (optional): `nominal,high`
  - `limit` (optional, integer, default: 5000, max: 25000)
  - `offset` (optional, integer, default: 0)
- **Response (200 OK):** `application/geo+json`
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": 1042918,
      "geometry": {
        "type": "Point",
        "coordinates": [101.4421, 0.5184]
      },
      "properties": {
        "source": "VIIRS_SNPP",
        "acq_datetime": "2026-10-25T06:18:00Z",
        "confidence": "high",
        "frp": 14.8,
        "brightness_ti4": 348.2,
        "is_peatland": true,
        "h3_res7": "8764a5209ffffff"
      }
    }
  ],
  "properties": {
    "count": 1,
    "limit": 5000,
    "offset": 0,
    "total_matched": 1
  }
}
```

---

#### `GET /api/v1/hotspots/tiles/{z}/{x}/{y}.pbf`
- **Purpose:** High-performance vector tile service delivering clipped Mapbox Vector Tiles (`ST_AsMVT`).
- **Path Parameters:**
  - `z`: Zoom level ($0 \le z \le 16$)
  - `x`: Tile column index
  - `y`: Tile row index
- **Query Parameters:**
  - `start_date`: `YYYY-MM-DD`
  - `end_date`: `YYYY-MM-DD`
  - `confidence`: `nominal,high`
- **Response Headers:** `Content-Type: application/x-protobuf`, `Cache-Control: public, max-age=3600`

---

### 2.3 Spatial & Temporal Aggregations

#### `GET /api/v1/hotspots/aggregates/time-series`
- **Purpose:** Historical hotspot count and FRP trend over time for charts and statistical breakdowns.
- **Query Parameters:**
  - `admin_id` (optional): Administrative region code (e.g. `'ID-RI'` for Riau)
  - `interval` (optional, default: `'day'`): `'day'`, `'week'`, `'month'`, `'year'`
  - `start_date`: `YYYY-MM-DD`
  - `end_date`: `YYYY-MM-DD`
- **Response (200 OK):**
```json
{
  "interval": "day",
  "start_date": "2026-10-01",
  "end_date": "2026-10-25",
  "data": [
    {
      "date": "2026-10-01",
      "hotspot_count": 42,
      "high_confidence_count": 29,
      "total_frp": 624.5,
      "peatland_hotspots": 31
    }
  ]
}
```

#### `GET /api/v1/hotspots/aggregates/h3`
- **Purpose:** Hexagonal grid aggregation for density heatmaps and spatio-temporal binning.
- **Query Parameters:**
  - `bbox`: `min_lon,min_lat,max_lon,max_lat`
  - `resolution` (default: 7): H3 resolution (6, 7, or 8)
  - `start_date`: `YYYY-MM-DD`
  - `end_date`: `YYYY-MM-DD`
- **Response (200 OK):**
```json
{
  "resolution": 7,
  "cells": [
    {
      "h3_index": "8764a5209ffffff",
      "count": 18,
      "sum_frp": 240.2,
      "max_frp": 55.1,
      "dominant_confidence": "high"
    }
  ]
}
```

---

### 2.4 Predictive Fire-Risk Endpoints

#### `GET /api/v1/risk/predictions`
- **Purpose:** Fetch spatial fire-risk predictions for a designated target date and forecast horizon.
- **Query Parameters:**
  - `date` (required): Target prediction date (`YYYY-MM-DD`)
  - `horizon_hours` (required): `24`, `72`, or `168`
  - `bbox` (optional): Bounding box
  - `admin_id` (optional): Administrative boundary filter
  - `min_risk` (optional, float): Filter cells above minimum threshold (e.g., $0.3$)
- **Response (200 OK):**
```json
{
  "model_version": "v0.1.0-baseline-lgbm",
  "target_date": "2026-10-27",
  "horizon_hours": 24,
  "predictions": [
    {
      "h3_index": "8764a5209ffffff",
      "risk_score": 0.84,
      "risk_category": "EXTREME",
      "top_drivers": [
        {"factor": "consecutive_dry_days", "contribution": 0.35},
        {"factor": "topsoil_moisture_swvl1", "contribution": -0.28},
        {"factor": "peatland_indicator", "contribution": 0.22}
      ]
    }
  ]
}
```

---

## 3. Rate Limiting & Guardrails

| Constraint | Limit | Trigger Action |
|---|---|---|
| **Max Bounding Box Area** | $250,000\text{ km}^2$ | Returns `400 Bad Request` |
| **Max Query Limit** | 25,000 records | Clamped automatically with warning in header |
| **Rate Limit (Public Tier)** | 60 requests / minute | Returns `429 Too Many Requests` |
