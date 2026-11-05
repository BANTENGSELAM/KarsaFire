# User Experience & Interface Specification (UX-SPEC) — KARSA-FIRE

> **Document Version:** 1.0.0-draft  
> **Status:** Specification (Phase 0)  
> **Target Platform:** Desktop-first Responsive Web Application (Chrome, Edge, Firefox, Safari)

---

## 1. UX Design Philosophy

KARSA-FIRE is an operational situational awareness and analytical workstation. The design language must balance:
1. **Immediate Visual Clarity:** High-contrast map layers that differentiate raw satellite thermal anomalies from aggregated risk surfaces instantly.
2. **Temporal Fluency:** Intuitive, buttery-smooth temporal playback and scrubbing through historical fire seasons.
3. **Scientific Credibility & Explainability:** Providing contextual explanations for risk scores rather than opaque ratings.
4. **Accessible Aesthetics:** Modern, dark-mode-first glassmorphic UI using perceptually uniform, colorblind-safe palettes (avoiding red-green confusion).

---

## 2. Core Layout Architecture

```text
+---------------------------------------------------------------------------------------+
|  [LOGO] KARSA-FIRE    | Region: [ Riau Province v ] | Range: [ Last 30 Days v ] | User|
+---------------------------------------------------------------------------------------+
|  LAYER CONTROLS  |                                                      |  INSPECTION |
|  [v] Active Hotspots                                                    |  DRAWER     |
|      - VIIRS (375m)                                                     |             |
|      - MODIS (1km)                                                      |  H3 Cell:   |
|      Conf: [Nom / High]                                                 |  #8764a520  |
|  [v] H3 Density Grid                                                    |             |
|  [ ] Fire Risk Forecast                                                 |  Risk: 84%  |
|      Horizon: [ 24h v ]                                                 |  [EXTREME]  |
|  -----------------                                                      |             |
|  STATISTICS                                                             |  Drivers:   |
|  Total Hotspots: 1,420                                                  |  - Dryness  |
|  High Conf: 1,104                                                       |  - Peatland |
|  Peatland: 68%                                                          |  - Wind     |
|                                                                         |  [Export]   |
+-------------------------------------------------------------------------+-------------+
|  [ < Play > ] |====o====================================================| Oct 26 2026 |
|  Time-Series Scrubber & Fire Radiative Power Histogram (UTC+7)                        |
+---------------------------------------------------------------------------------------+
```

---

## 3. UI Component Specifications

### 3.1 Primary Header & Global Bar
- **Region Selector (`#control-region-select`):** Hierarchical dropdown allowing instant zoom to predefined hotspots provinces (e.g., Riau, South Sumatra, Central Kalimantan, West Kalimantan, Jambi).
- **Date Range Picker (`#control-date-range`):** Quick presets (`Last 7 Days`, `Last 30 Days`, `Peak Dry Season 2023`, `El Niño 2019`, `Custom Range`).
- **System Health Indicator (`#status-pill`):** Displays data freshness timestamp (e.g. `FIRMS: Updated 2h ago`).

### 3.2 Floating Layer & Filter Drawer (Left)
- **Sensor Toggles:** Independent switches for VIIRS (S-NPP / NOAA-20) and MODIS (Terra / Aqua).
- **Confidence Slider / Filter:** Radio-checkbox group (`High only`, `Nominal + High`, `All detections`).
- **Layer Stacking Manager:**
  - Layer 1: Base Map (Dark Matter / Satellite Imagery / Topo).
  - Layer 2: Peatland / Forest Concession Vector Outlines.
  - Layer 3: Risk Prediction Heatmap (H3 hexagonal cells).
  - Layer 4: Raw Active Hotspot Points with glowing animated halos.

### 3.3 Interactive Map Canvas (Center)
- **Rendering Engine:** WebGL2 via Deck.gl and MapLibre GL.
- **Hover Micro-interaction:** Hovering over a hotspot or H3 cell presents a glassmorphic tooltip with:
  - Latitude, Longitude, and H3 index.
  - Acquisition timestamp (UTC and Local Time).
  - Fire Radiative Power (MW) and Brightness (K).
  - Peatland status badge (`PEATLAND` / `MINERAL SOIL`).
- **Click Behavior:** Pins the selected cell and expands the right-side Inspection Drawer.

### 3.4 Temporal Scrubber & Playback Engine (Bottom)
- **Histogram-integrated Timeline (`#timeline-scrubber`):**
  - Displays daily hotspot counts as micro-vertical bars along the temporal axis.
  - Draggable time slider with playback controls (`Play`, `Pause`, `Speed: 1x, 2x, 5x`).
  - Animates the map canvas by filtering point coordinates matching the sliding temporal window.

### 3.5 Detailed Cell Inspection Drawer (Right)
- **Cell Metadata:** H3 Hexagon ID, Administrative District (Kabupaten, Kecamatan, Desa).
- **Risk Attribution Breakdown:** Waterfall chart or ranked bar chart displaying SHAP feature contributions (e.g., Consecutive Dry Days: $+35\%$, Peat Subsurface: $+22\%$).
- **Local Time-Series Trend:** 14-day historical trend of soil moisture and rainfall for the selected cell.

---

## 4. Visual Design Tokens & Palette (Colorblind Accessibility)

### 4.1 Theme Tokens (Sleek Dark Mode)
```css
:root {
  --bg-primary: #0a0e17;        /* Ultra-dark slate canvas */
  --bg-surface: #121826;        /* Surface container */
  --bg-glass: rgba(18, 24, 38, 0.78); /* Frosted glass with backdrop-filter */
  --border-subtle: rgba(255, 255, 255, 0.08);
  --border-focus: #ff5722;      /* Vivid fire orange */
  --text-primary: #f8fafc;
  --text-secondary: #94a3b8;
  --text-muted: #64748b;
}
```

### 4.2 Fire & Hazard Color Scales (Colorblind-Safe)
Standard red-green scales fail for deuteranopia and protanopia. KARSA-FIRE uses luminance-monotonic, perceptually uniform palettes:

| Hazard Level | Color Hex | Color Name | Luminance / Visual Appearance |
|---|---|---|---|
| **Low** | `#2cba00` / `#3b82f6` | Calm Blue-Green / Cool Azure | High legibility against dark slate basemap |
| **Moderate** | `#f59e0b` | Amber / Gold | Distinct intermediate threshold |
| **High** | `#ef4444` | Vivid Crimson | Sharp contrast against amber |
| **Extreme** | `#a855f7` | Ultraviolet / Neon Purple | Highest optical luminance, denotes peak emergency |

---

## 5. Automated Testing Attributes (`data-testid`)

All interactive elements must feature distinct, stable test attributes for end-to-end automation:
- `data-testid="map-canvas"`
- `data-testid="filter-sensor-viirs"`
- `data-testid="filter-sensor-modis"`
- `data-testid="filter-confidence-high"`
- `data-testid="timeline-play-btn"`
- `data-testid="timeline-slider"`
- `data-testid="cell-inspection-drawer"`
- `data-testid="risk-score-badge"`
