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

::: incremental

- The published schedule is never what actually flies — cancellations, swaps, additions happen constantly
- New and seasonal routes have no history to rely on
- The further out the forecast, the more the schedule diverges from reality
- **FLT predicts what will actually fly — not what was planned.**

:::

---

### Validation Setup

<div style="font-size: 0.8em;">

| | |
|---|---|
| **Departure Airports** | BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW |
| **Airlines** | LH, OS, LX, SN, EW, EN, WK, 4Y, YF, XQ, 3S |
| **Airline–Airport combinations** | 64 active |
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

### How FLT Works

**From route averages → forecasted capacity**

<div style="font-size: 0.75em; margin-top: 0.6em;">

| Step | What | Why |
|---:|---|---|
| **1a** | Departure delta model | The schedule is a baseline — correct it, don't replace it |
| **1b** | Flight-minutes delta model | Captures aircraft swaps and frequency changes |
| **2** | Recency-weighted rates + correction ratio | kg/flight-min by airline + aircraft type + airport; ratio nudges at long horizons |

</div>

<div style="margin-top: 0.8em; font-size: 0.82em;">

F+ asks: *"How much did this route historically burn?"*

FLT asks: *"How much will actually fly — and what will those aircraft burn?"*

</div>

---

### How the Model Learns — Phase 1

**Schedule Correction: Two Delta Models**

The schedule is the baseline — the models learn **to adjust it.**

<div style="text-align: center; font-size: 1.1em; margin: 0.8em 0;">

`predicted = scheduled + Δ`

</div>

<div style="font-size: 0.85em; margin-top: 0.6em;">

| | Departures | Flight Minutes |
|---|---|---|
| **Target (Δ)** | actual − scheduled | actual − scheduled |

</div>

<div style="margin-top: 0.8em; font-size: 0.82em; color: #aaa;">

Δ driven by: `days_to_ops` · `seat_bin` · `airline` · `station` · historical lags

</div>

---

### How the Model Learns — Phase 2

**Capacity → Uplift**

<div style="font-size: 0.85em; margin-top: 0.8em;">

| What | How |
|---|---|
| **Fuel rates** | kg/dep · kg/min, recency-weighted by airline, actype, airport |
| **Estimate** | Blend of per-flight and per-minute signals |
| **Sparse routes** | Fallback: route → airport → global |
| **Long horizon** | Correction ratio (≈ 1.0) nudges the estimate |

</div>

<div style="font-size: 0.78em; margin-top: 1.2em; color: #aaa;">

Uplift model trained on **2025 data** · Schedule correction trained on **Jan 2024 – Jun 2025** · Grouped by airline + aircraft type + departure airport

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
        P1A["Departure Delta Model"]
        P1B["Flight Minutes Delta Model"]
    end
    subgraph phase2["Phase 2: Uplift Prediction"]
        direction TB
        RM["Recency-Weighted Rates"]
        BLEND["Blend kg/dep + kg/min"]
        FB["Fallback: route / airport / global"]
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
    style phase2 fill:#3a2a1a,stroke:#fcc419,color:#fff
    style RM fill:#4a3a1a,stroke:#fcc419,color:#fff
    style BLEND fill:#4a3a1a,stroke:#fcc419,color:#fff
    style FB fill:#4a3a1a,stroke:#fcc419,color:#fff
    style CR fill:#3a1a4a,stroke:#cc5de8,color:#fff
    style output fill:#1a3a3a,stroke:#20c997,color:#fff
    style FO fill:#1a4a4a,stroke:#20c997,color:#fff
</div>
```

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

### Month-by-Month Head-to-Head

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/10_drilldown_bars.html" height="520" width="100%"></iframe>

---

### Built to Improve

<div style="font-size: 0.88em; margin-top: 0.6em;">

- **17 months** of data — one summer, one winter season
- Every new month improves accuracy at every forecast horizon
- Uplift model trained on **2025 data only** due to tankering regulations
- More airports = stronger shared patterns

</div>

<div style="margin-top: 1em; font-size: 0.85em;">

**These results are our floor, not our ceiling.**

</div>

::: notes
- days_to_ops: each new month adds training examples at every horizon from 1 to 180+ days out. The model learns more precisely how schedule accuracy degrades with distance.
- seat_bin: more routes and airports means richer aircraft-type coverage — fewer fallbacks on new or seasonal routes where history is sparse.
- Tankering regulations changed how airlines order fuel in 2025, making pre-2025 uplift data less relevant as a training signal. Every additional month under the new rules directly improves the uplift model.
- Adding airports strengthens shared patterns across the network without diluting route-level accuracy — the model scales cleanly.
:::

---

### Roadmap

```{=html}
<div class="mermaid">
%%{init: {'theme': 'dark', 'timeline': {'useMaxWidth': true}}}%%
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

### The Ask

<div style="font-size: 0.85em;">

::: incremental

- **5 out of 6 months, 15% less misorder**
- 10 airports, 11 airlines, 64 airline–airport combinations — 6 months validated
- Airline-agnostic: adding airlines = adding data
- Continue the project and expand coverage

:::

</div>

```{=html}
<!-- build: 2026-02-25 -->
```
