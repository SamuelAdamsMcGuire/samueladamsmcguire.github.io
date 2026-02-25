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

### The Challenge

Predicting fuel uplift accurately, months ahead, across a constantly changing schedule.

::: incremental

- Schedules change — cancellations, additions, and aircraft swaps happen after publication
- New and seasonal routes have limited history to rely on
- The further ahead the forecast, the more the schedule diverges from reality
- FLT addresses this by predicting **what will actually fly**, not what was planned

:::

---

### Validation Setup

<div style="font-size: 0.8em;">

| | |
|---|---|
| **Departure Airports** | BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW |
| **Airlines** | LH, OS, LX, SN, EW, EN, WK, 4Y, YF, XQ, 3S |
| **Airline–Airport combinations** | ~64 active |
| **Forecast date** | 12 June 2025 — as if today is June 12th and we forecast Jul–Dec 2025 |
| **Total Uplift Volume** | 1.61M t |

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
| **6-Month Total Misorder** | 53,800 t | 63,500 t |

</div>

---

### Month-by-Month Head-to-Head

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/01_monthly_bars.html" height="500" width="100%"></iframe>

---

### The F+ Approach

```{=html}
<div class="mermaid">
%%{init: {'theme': 'dark', 'flowchart': {'nodeSpacing': 40, 'rankSpacing': 50}}}%%
flowchart LR
    A1["Published Schedule"] --> A2["Last 3 months avg"] --> A3["Route Average"] --> A4["Predicted Uplift"]
    style A1 fill:#2a2a2a,stroke:#888,color:#fff
    style A2 fill:#2a2a2a,stroke:#888,color:#fff
    style A3 fill:#2a2a2a,stroke:#888,color:#fff
    style A4 fill:#2a2a2a,stroke:#888,color:#fff
</div>
```

<div style="font-size: 0.8em; margin-top: 1em;">

When the schedule changes, the historical average doesn't adjust — leading to growing errors the further out the forecast.

</div>

---

### The FLT Pipeline

```{=html}
<div class="mermaid">
%%{init: {'theme': 'dark', 'flowchart': {'useMaxWidth': false, 'nodeSpacing': 35, 'rankSpacing': 45}}}%%
flowchart TB
    subgraph input["Input"]
        S["Schedule (SSIM)"]
    end
    subgraph phase1["Phase 1: Schedule Correction"]
        direction LR
        P1A["Predict Departures"]
        P1B["Predict Flight Minutes"]
    end
    subgraph phase3["Phase 2: Statistical Uplift"]
        direction TB
        RM["Recency-Weighted Means"]
        BLEND["Blend per-flight + per-minute"]
        FB["Fallback: route / airport / global"]
    end
    subgraph phase4["Phase 3: ML Correction"]
        CR["Correction Ratio"]
    end
    subgraph output["Output"]
        FO["Predicted Uplift"]
    end
    S --> phase1
    P1A --> BLEND
    P1B --> BLEND
    RM --> BLEND
    FB -.->|"no history"| BLEND
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
| **2** | Statistical uplift formula | Recency-weighted route means × predicted volume |
| **3** | ML correction | Fine-tunes the formula at far horizons |

</div>

<div style="margin-top: 0.8em; font-size: 0.8em;">

**Key insight:** We predict by airline, aircraft type, and airport — shifting from fixed route averages to forecasted capacity.

</div>

---

### Fuel Ordering Impact

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/08_fuel_savings.html" height="550" width="100%"></iframe>

<div style="font-size: 0.7em;">

**9,700 t less misorder** than F+ over 6 months across 10 airports

</div>

---

### Where We Win Big — LH-FRA

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/06_lh_fra_deep_dive.html" height="500" width="100%"></iframe>

<div style="font-size: 0.7em;">

LH-FRA alone: **12,700 t less misorder** with FLT vs F+ over 6 months

</div>

---

### Where We Win Big — Unscheduled Flights

**XQ-KEF, July 2025**

<div style="font-size: 0.85em;">

| | Scheduled | Actual | FLT Predicted |
|---|---|---|---|
| Flights | 0 | 2 | 2 |
| Uplift | — | 15,854 kg | 18,915 kg |

</div>

<div style="margin-top: 1em;">

F+ has **no prediction** — it can't see what wasn't scheduled.

FLT caught the unscheduled flights using historical patterns.

</div>

---

### Roadmap

```{=html}
<div class="mermaid">
%%{init: {'theme': 'dark'}}%%
timeline
    Now : Proof of Concept : 10 airports validated : 5-1 vs F+
    Phase 1 : Multi-scenario validation
    Phase 2 : Full airport expansion
    Phase 3 : Production : 4–6 month horizon
    Phase 4 : Long-term model : 15–18 month horizon
    Phase 5 : Full system launch : ~400 airports
</div>
```

---

### The Opportunity

<div style="font-size: 0.85em;">

- **Today:** 10 departure airports → **9,700 t** less misorder in 6 months
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

- **5 out of 6 months, 15% less uplift misorder**
- 10 airports, 11 airlines, ~64 airline–airport combinations — 6 months validated
- Airline-agnostic: adding airlines = adding data
- Continue the project and expand coverage

:::

</div>
