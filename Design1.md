# Design 1: Temporal Dashboard — WA Data Breaches

**File:** `index.html`
**Dataset:** Data Breach Notifications Affecting Washington Residents (WA Attorney General's Office)

---

## Overview

Design 1 is a time-series area chart that shows the number of Washington State residents affected by reported data breaches, aggregated by month. The view supports both a high-level overview of the full dataset timeline and a zoomed-in focus on any user-selected time window.

---

## Tasks Addressed

| Task | Description |
|------|-------------|
| **Trend detection** | Are data breaches worsening over time? |
| **Peak identification** | Which months saw the largest spikes in affected residents? |
| **Cause breakdown** | What types of breaches (Cyberattack, Unauthorized Access, etc.) drive each spike? |
| **Event drill-down** | Which specific organizations caused the largest exposures in a given month? |

---

## Visual Encoding

| Channel | Variable | Justification |
|---------|----------|---------------|
| **X position** (time axis) | Month of breach awareness date | Position on a common scale is the most accurate channel for ordered quantitative data |
| **Y position** (linear axis) | Total Washingtonians affected per month | Enables direct magnitude comparison across months |
| **Area mark** (filled under line) | Cumulative "weight" of exposure | Reinforces the sense of accumulated harm better than a bare line |
| **Gradient fill** (opaque at peak → transparent at baseline) | Visual salience | Draws the eye upward to spikes without obscuring the x-axis |
| **Line stroke** | Month-over-month trend | Provides precise shape cues independent of fill opacity |
| **Dots on hover** | Individual monthly data points | Surfaces exact values on demand without permanent clutter |
| **Color (legend)** | Breach cause category | Hue is used in tooltips to identify the dominant cause for each month |

---

## Interaction Design

### Focus + Context (Brush)
A small **context chart** below the main chart shows the full temporal range at reduced height. Users drag a brush selection over it to zoom the main (focus) chart into any sub-range. This is the standard "overview + detail" idiom for dense time series:

- The context chart preserves the global shape at all times.
- The focus chart rescales its Y-axis dynamically to the selected window, maximizing resolution for the chosen period.
- The selected date range is labeled above the chart and in the "Selected Window" stat card.

### Hover Tooltip
Moving the cursor over the focus chart bisects to the nearest monthly data point and shows:
- Month and year
- Total residents affected
- Number of breach events
- Dominant breach cause
- Top 3 largest individual incidents (organization name + count)

A vertical crosshair tracks the cursor to link pointer position to chart position.

### Stat Cards
Four summary cards update in real time:
1. **Total Residents Affected** — all years in the dataset
2. **Breach Events** — unique incident count (deduplicated by breach ID)
3. **Worst Single Month** — peak month and its resident count
4. **Selected Window** — total for the brushed date range

---

## Data Transformations

- **Deduplication:** Each breach `Id` appears once per `InformationType` row in the CSV; rows are deduplicated by `Id` before aggregation so each incident is counted once.
- **Monthly rollup:** Breach awareness dates (`DateAware`) are floored to the first of their month using `d3.timeMonth`. Resident counts are summed per month.
- **Cause normalization:** `DataBreachCause` values are mapped to four categories: `Cyberattack`, `Unauthorized Access`, `Theft or Mistake`, and `Unknown`.

---

## Design Alternatives Considered

| Alternative | Reason Not Chosen |
|-------------|-------------------|
| Bar chart per month | At ~8 years × 12 months ≈ 96 bars, individual bars become too thin; area preserves shape better |
| Stacked area by cause | Cause proportions are noisy month-to-month; stacking obscures the total trend which is the primary question |
| Scatter plot (one dot per breach) | Overplotting at large counts; hard to see the aggregate trend |
| Log Y-axis | Distorts the perceived magnitude of large spikes, which are the most important signal |

---

## Screenshot Description

```
┌──────────────────────────────────────────────────────────┐
│  Design 1: The Temporal Dashboard                        │
│  [Total Affected] [Breach Events] [Worst Month] [Window] │
│                                                          │
│  Washingtonians Affected per Month                       │
│  ┌──────────────────────────────────────────────────┐   │
│  │  ^                                               │   │
│  │  │        ████                                   │   │
│  │  │       █████                                   │   │
│  │  │  ██  ████████  ██                             │   │
│  │  │ ████████████████████████                      │   │
│  │  └──────────────────────────────────────────►    │   │
│  │       2017   2019   2021   2023   2025            │   │
│  └──────────────────────────────────────────────────┘   │
│  Context (drag to zoom):                                 │
│  ┌──────────────────────────────────────────────────┐   │
│  │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │   │
│  └──────────────────────────────────────────────────┘   │
│  [● Cyberattack] [● Unauthorized Access] [● Theft] ...  │
└──────────────────────────────────────────────────────────┘
```

---

## Files

| File | Role |
|------|------|
| `index.html` | D3.js implementation of Design 1 |
| `Design1.md` | This design document |
| `Data_Breach_Notifications_...csv` | Source dataset |
