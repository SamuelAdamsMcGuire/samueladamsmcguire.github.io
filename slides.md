---
title: "FLT: Smarter Fuel Uplift Forecasting"
subtitle: DataTactics GmbH
date: February 2026
theme: moon
transition: concave
highlight-style: breezeDark
progress: true
slideNumber: true
hash: true
navigationMode: linear
autoPlayMedia: true
css: styles.css
---

### The Problem

F+ forecasts fuel uplift by averaging the **last 3 months** of actual uplift per route.

- Ignores schedule changes (cancellations, added flights)
- Can't predict new or seasonal routes
- Gets worse the further into the future it forecasts

**Can we do better?**

---

### Validation Setup

<div style="font-size: 0.8em;">

| | |
|---|---|
| **Departure Airports** | BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW |
| **Airlines** | LH, OS, LX, SN, EW, EN, WK, 4Y, YF, XQ, 3S |
| **Routes** | ~64 active |
| **Forecast Horizon** | 6 months (Jul–Dec 2025) |
| **Total Volume** | 1,610M kg fuel |

</div>

---

### FLT Beats F+ in 5 out of 6 Months {.highlight-title}

<div style="font-size: 3em; text-align: center; margin: 0.5em 0;">
**5 — 1**
</div>

<div style="display: flex; justify-content: center; gap: 0.3em; font-size: 2em;">
<span style="color: #00d4aa;">&#10003;</span>
<span style="color: #00d4aa;">&#10003;</span>
<span style="color: #ff6b6b;">&#10007;</span>
<span style="color: #00d4aa;">&#10003;</span>
<span style="color: #00d4aa;">&#10003;</span>
<span style="color: #00d4aa;">&#10003;</span>
</div>

<div style="margin-top: 1em; font-size: 0.9em;">

| | FLT | F+ |
|---|---|---|
| **Overall Error** | **3.3%** | 3.9% |
| **6-Month Misorder** | 53,800t | 63,500t |

</div>

---

### Month-by-Month Head-to-Head

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/01_monthly_bars.html" height="500" width="100%"></iframe>

---

### The Further Out We Forecast, the Bigger Our Edge

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/02_error_trend.html" height="500" width="100%"></iframe>

---

### Why F+ Struggles at Long Horizons

::: incremental

- F+ depends on the schedule and the previous 3 months uplift average
- When the schedule changes, F+ doesn't know
- FLT **corrects the schedule first**, then predicts fuel using ML
- Seasonal shifts, cancellations, and new routes are captured

:::

---

### The F+ Pipeline

```{=html}
<div class="mermaid">
flowchart TB
    A1["Schedule departures"]
    A2["Historical Uplift<br/>(last 3 months)"]
    A3["Simple Average<br/>per Route"]
    A4["Fuel Order<br/>Prediction"]
    A1 --> A2 --> A3 --> A4
    style A1 fill:#662222,stroke:#ff6b6b,color:#fff
    style A2 fill:#662222,stroke:#ff6b6b,color:#fff
    style A3 fill:#662222,stroke:#ff6b6b,color:#fff
    style A4 fill:#662222,stroke:#ff6b6b,color:#fff
</div>
```

---

### The FLT Pipeline

```{=html}
<div class="mermaid">
flowchart TB
    subgraph input["Inputs"]
        S["Published Schedule<br/>(SSIM)"]
    end
    subgraph phase1["Phase 1: Schedule Correction"]
        direction LR
        P1A["1a: Predict<br/>Actual Departures"]
        P1B["1b: Predict<br/>Actual Flight Minutes"]
    end
    subgraph phase3["Phase 2: Statistical Uplift"]
        direction TB
        RM["Recency-Weighted<br/>Route Means"]
        BLEND["Blend Formula<br/>(per-flight + per-minute) / 2"]
        FB["3-Level Fallback<br/>route → airport → global"]
    end
    subgraph phase4["Phase 3: ML Correction"]
        CR["Correction Ratio<br/>(HistGradientBoosting)"]
    end
    subgraph output["Output"]
        FO["Fuel Order<br/>Prediction"]
    end
    S --> phase1
    P1A --> BLEND
    P1B --> BLEND
    RM --> BLEND
    FB -.->|"no history?"| BLEND
    BLEND --> CR
    CR --> FO
    style input fill:#1a2a4a,stroke:#4dabf7,color:#fff
    style S fill:#1a3a5a,stroke:#4dabf7,color:#fff
    style phase1 fill:#1a3a1a,stroke:#51cf66,color:#fff
    style P1A fill:#1a4a1a,stroke:#51cf66,color:#fff
    style P1B fill:#1a4a1a,stroke:#51cf66,color:#fff
    style phase3 fill:#3a2a1a,stroke:#fcc419,color:#fff
    style RM fill:#4a3a1a,stroke:#fcc419,color:#fff
    style BLEND fill:#4a3a1a,stroke:#fcc419,color:#fff
    style FB fill:#4a3a1a,stroke:#fcc419,color:#fff
    style phase4 fill:#2a1a3a,stroke:#cc5de8,color:#fff
    style CR fill:#3a1a4a,stroke:#cc5de8,color:#fff
    style output fill:#1a3a3a,stroke:#20c997,color:#fff
    style FO fill:#1a4a4a,stroke:#20c997,color:#fff
</div>
```

---

### How FLT Works

<div style="font-size: 0.7em;">

| Step | What | Why |
|---:|---|---|
| **1a** | Predict actual departures | Schedule is never perfectly right |
| **1b** | Predict actual flight minutes | Captures aircraft swaps & frequency changes |
| **2** | Statistical uplift formula | Recency-weighted route means x predicted volume |
| **3** | ML correction | Fine-tunes the formula at far horizons |

</div>

<div style="margin-top: 0.8em; font-size: 0.8em;">

**Key insight:** We don't trust the schedule. We predict what will actually fly.

</div>

---

### Fuel Ordering Impact

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/08_fuel_savings.html" height="550" width="100%"></iframe>

<div style="font-size: 0.7em;">

**~9,700 tonnes less misorder** than F+ over 6 months across 10 airports

</div>

---

### Where We Win Big — LH-FRA

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/06_lh_fra_deep_dive.html" height="500" width="100%"></iframe>

<div style="font-size: 0.7em;">

Even small % improvements on high-volume routes = large kg savings

</div>

---

### Where We Win Big — Unscheduled Flights

**XQ-KEF, July 2025**

<div style="font-size: 0.85em;">

| | Scheduled | Actual | Predicted |
|---|---|---|---|
| Flights | 0 | 2 | 2 |
| Uplift (kg) | — | 15,854 | 18,915 |

</div>

<div style="margin-top: 1em;">

F+ has **no prediction** — it can't see what wasn't scheduled.

FLT caught the unscheduled flights using historical patterns.

</div>

---

### Current Scope

<div style="font-size: 0.85em;">

| | |
|---|---|
| **Airports** | BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW |
| **Airlines** | 11 across Lufthansa Group |
| **Horizon** | 6 months (Jul-Dec 2025 validated) |
| **Routes** | ~64 active |
| **Total Volume** | 1,610M kg fuel over validation period |

</div>

<div style="margin-top: 1em;">

This is our proof of concept. **The model is ready to scale.**

</div>

---

### Roadmap

<div style="font-size: 0.7em;">

| Next Steps | What | Deliverable |
|---:|---|---|
| 1 | Multi-scenario validation | Models validated across diverse periods |
| 2 | Full airport expansion + ML pipeline | Complete coverage, automated training |
| 3 | Short-term production deployment | 4-6 month predictions live, parallel with F+ |
| 4 | Long-term model development | 15-18 month predictions validated |
| 5 | Full system launch | All airports, all horizons, user interface |

</div>


---

### The Opportunity

<div style="font-size: 0.85em;">

- **Today:** 10 departure airports → **9,700 tonnes** less misorder in 6 months
- Every airport added means more routes, more data, more savings
- The model is airline-agnostic and built to scale
- **~400 airports** on the roadmap

</div>

<div style="margin-top: 1em; font-size: 0.8em;">

**More airports. Same model. More impact.**

</div>

---

### Built to Improve

<div style="font-size: 0.85em;">

- Trained on just **17 months** of schedule data (Jan 2024 – Jun 2025)
- The model has only seen **one** summer, **one** winter season
- With each additional month, patterns become stronger
- More airports = stronger shared patterns, without affecting existing route accuracy
- More data = better seasonal correction = fewer misorders

</div>

<div style="margin-top: 1em; font-size: 0.8em;">

**These results are our floor, not our ceiling.**

</div>

---

### The Ask

<div style="font-size: 0.85em;">

::: incremental

- **5 out of 6 months, 15% less fuel misorder**
- 10 airports, 11 airlines, 64 routes — 6 months validated
- Airline-agnostic: adding airlines = adding data
- Continue the project and expand coverage

:::

</div>


