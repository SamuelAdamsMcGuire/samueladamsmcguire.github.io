---
title: "FLT: Smarter Fuel Uplift Forecasting"
subtitle: DataTactics GmbH
date: February 2026
theme: moon
transition: slide
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
| **ML Calibration** | Correction ratio (≈ 1.0) nudges the estimate |

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

---

### Appendix: Phase 1 — Schedule Correction Detail

<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; height:55vh; gap:16px;">
<p style="color:#8899aa; font-size:0.8em;">Departures &amp; flight minutes by month — filter by airline, airport, seat bin — Volumes / Abs Error / WAPE % toggle</p>
<a href="assets/phase1_comparison.html" target="_blank"
   style="display:inline-block; padding:14px 40px; background:#00d4aa; color:#16213e;
          font-size:1.1em; font-weight:bold; border-radius:6px; text-decoration:none;">
  Open Phase 1 Chart ↗
</a>
</div>

---

### Appendix: Worst Case Root Cause Navigator

<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; height:55vh; gap:16px;">
<p style="color:#8899aa; font-size:0.8em;">6 worst routes · Uplift → Departures → Training history · Category badges · Root cause per case</p>
<a href="assets/defense.html" target="_blank"
   style="display:inline-block; padding:14px 40px; background:#00d4aa; color:#16213e;
          font-size:1.1em; font-weight:bold; border-radius:6px; text-decoration:none;">
  Open Root Cause Navigator ↗
</a>
</div>

---

### Appendix: Phase 1 — What We Tested

<div style="font-size:0.84em;">

- **Delta model** — predict the *correction* (`actual − scheduled`); final = `scheduled + Δ`
- **Direct model** — predict the actual value outright, ignoring the schedule
- **Horizon split** — separate models for short (1–90d) and mid (91–220d) horizons
- **No-split** — one model for all horizons (tested for minutes; departures already uses this)
- **No lags** — removes all lag features; shows how much historical patterns contribute
- **Core only** — airline + airport + seat bin + scheduled + month + days_to_ops only

</div>

---

### Appendix: Phase 1 — Results

<div style="font-size:0.78em;">

**Departures** — WAPE vs actual departures, Jul–Dec 2025:

| Variant | Overall | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---|---|---|---|---|---|---|
| Delta split short+mid | 6.1% | 2.6 | 3.6 | 2.6 | 3.8 | 14.4 | 20.0 |
| **Delta single (current)** | **6.4%** | 3.0 | 4.0 | 3.2 | 4.3 | 13.8 | 20.8 |
| Delta no lags | 7.2% | 3.3 | 4.3 | 3.6 | 4.4 | 15.3 | 21.4 |

**Minutes** — WAPE vs actual flight minutes, Jul–Dec 2025:

| Variant | Overall | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---|---|---|---|---|---|---|
| **Delta split (current)** | **5.5%** | 4.1 | 5.3 | 4.7 | 7.7 | 11.1 | 15.2 |
| Delta split no lags | 6.5% | 3.6 | 5.5 | 5.6 | 5.9 | 9.2 | 19.0 |
| Delta no split | 7.4% | 4.8 | 6.2 | 6.3 | 6.6 | 9.4 | 18.2 |

</div>

---

### Appendix: Departures — Predict the Correction

<div style="font-size:0.84em;">

The schedule is our best starting point. We don't predict departures from scratch — we predict **how far off the schedule will be**, then add that correction on top.

</div>

<div style="background:#0d1f2d; border-left:4px solid #00d4aa; padding:10px 16px; margin:14px 0; font-family:monospace; font-size:0.88em;">
predicted departures = scheduled departures + Δ
</div>

<div style="font-size:0.84em;">

- Schedule says **14 flights**, model expects **2 cancellations** → predict **12**
- Route chronically over-promises → delta is consistently negative; model learns this
- Extra charters regularly get added → delta is positive; model learns this too

</div>

---

### Appendix: Departures — Why Delta Works

<div style="font-size:0.84em;">

**Why predict the gap rather than the absolute number?**

The published schedule already encodes route structure, airline intent, and seasonality.
We only need to learn the *correction* — a much smaller and more stable signal.
This generalises far better across routes with sparse history.

</div>

<div style="font-size:0.84em; margin-top:1em;">

**One model or two?**

We tested a short/mid horizon split identical to the minutes approach.
Result: **6.1% vs 6.4% WAPE** — marginal improvement, not worth the added complexity.
A single model covering 1–220 days ahead works well for departures.

</div>

---

### Appendix: Departures — Features

<div style="font-size:0.84em;">

| Feature | Role |
|---|---|
| Scheduled departures | Starting baseline |
| Airline | Each airline over/under-delivers on its schedule differently |
| Airport | Disruption levels vary by airport |
| Seat bin | Wide-body vs narrow-body have different cancellation rates |
| `days_to_ops` | Accuracy degrades with forecast distance; model adjusts |
| Month | Seasonal patterns — summer over-promises more than winter |
| Lag corrections — 7 / 14 / 28 / 90 / 180 / 365 days | Correction patterns at this route and horizon in history |

Without lag features, WAPE rises from **6.4% → 7.2%**.
The historical pattern of corrections is real, repeating signal.

</div>

---

### Appendix: Flight Minutes — Why Not Just Departures

<div style="font-size:0.84em;">

Predicted departures alone cannot estimate fuel — **duration matters**.

An A380 long-haul sector burns more fuel than five A320 short hops, even with the same departure count.
We need to know *how long the aircraft is in the air*.

</div>

<div style="background:#0d1f2d; border-left:4px solid #756bb1; padding:10px 16px; margin:14px 0; font-family:monospace; font-size:0.88em;">
predicted flight minutes = scheduled flight minutes + Δ
</div>

<div style="font-size:0.84em;">

Same delta approach — predict the correction to the schedule, not the absolute value.

</div>

---

### Appendix: Flight Minutes — What Minutes Adds

<div style="font-size:0.84em;">

Minutes captures two things that departure count cannot:

- **Aircraft size** — seat bin identifies the family; minutes tell us how far it actually flew
- **Routing changes** — a swap from direct to connecting changes flight time even if the number of departures stays the same

Together, predicted departures and predicted minutes feed the fuel rate formula in Phase 2:

</div>

<div style="background:#0d1f2d; border-left:4px solid #f59e0b; padding:10px 16px; margin:14px 0; font-family:monospace; font-size:0.88em;">
base_fuel = (kg_per_dep × deps) + (kg_per_min × mins)
</div>

<div style="font-size:0.84em;">

Without the minutes signal, this formula collapses to a simple per-departure rate — far less accurate.

</div>

---

### Appendix: Flight Minutes — Short vs Mid Horizon Split

<div style="font-size:0.84em; margin-bottom:0.6em;">

Unlike departures, minutes uses **two separate models** — the single biggest accuracy gain in Phase 1.

WAPE improvement: **7.4% → 5.5%** on unseen Jul–Dec 2025 data.

</div>

<div style="display:flex; gap:18px; font-size:0.82em;">

<div style="flex:1; background:#1a2a3a; border-radius:8px; padding:14px 16px;">

**Short horizon — 1 to 90 days**

- Operational decisions mostly finalised
- Swaps and cancellations partially confirmed
- Corrections are small and predictable
- Model learns tight, low-variance adjustments

</div>

<div style="flex:1; background:#1a2a3a; border-radius:8px; padding:14px 16px;">

**Mid horizon — 91 to 220 days**

- Schedule is still highly speculative
- Airline planning changes are common
- Corrections are larger and more volatile
- Model learns to discount the schedule more

</div>

</div>

---

### Appendix: Flight Minutes — Features

<div style="font-size:0.84em;">

Both models (short + mid) use the same feature set, trained and applied within their `days_to_ops` range.

| Feature | Role |
|---|---|
| Scheduled flight minutes | Starting baseline |
| Airline + airport + seat bin | Route and aircraft identity |
| `days_to_ops` | Horizon within the band (1–90 or 91–220) |
| Month | Seasonal routing shifts |
| Lag departure corrections — 7 / 14 / 28 / 90 / 180 / 365 days | Past departure accuracy at this route + horizon |
| Lag minute corrections — 7 / 14 / 28 / 90 / 180 / 365 days | Past minute accuracy at this route + horizon |

Lags are grouped by route **and** `days_to_ops` band — the model learns horizon-specific patterns.
Without lags: WAPE rises **5.5% → 6.5%**.

</div>

---

### Appendix: Phase 1 Model Comparison

<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; height:68vh; gap:16px;">
<p style="color:#8899aa; font-size:0.8em;">5 departures variants · 5 minutes variants · delta vs direct · single vs split · full vs core · Jul–Dec 2025</p>
<div style="display:flex; gap: 20px;">
<a href="assets/phase1_comparison.html" target="_blank"
   style="display:inline-block; padding:14px 36px; background:#4e9af1; color:#fff;
          font-size:1.05em; font-weight:bold; border-radius:6px; text-decoration:none;">
  Open Combined Chart ↗
</a>
<a href="assets/phase1_comparison_deps.html" target="_blank"
   style="display:inline-block; padding:14px 36px; background:#00d4aa; color:#16213e;
          font-size:1.05em; font-weight:bold; border-radius:6px; text-decoration:none;">
  Departures Only ↗
</a>
<a href="assets/phase1_comparison_mins.html" target="_blank"
   style="display:inline-block; padding:14px 36px; background:#756bb1; color:#fff;
          font-size:1.05em; font-weight:bold; border-radius:6px; text-decoration:none;">
  Minutes Only ↗
</a>
</div>
</div>

---

### Appendix: Phase 2 — What We Tested

<div style="font-size:0.84em;">

- **Baselines** — seasonal mean (Dummy) and Ridge regression; anchor for scale
- **ML direct** — HistGradientBoosting predicts kg uplift directly from Phase 1 outputs
- **ML ratio** — model predicts a *correction ratio* on a route-mean base; HL variants use exponentially-weighted means (half-life 200 days)
- **Formula only** — pure `route_mean × predicted deps/mins`; no model trained; simple vs HL-weighted

</div>

---

### Appendix: Phase 2 — Key Finding and Results

<div style="font-size:0.84em; margin-bottom:0.6em;">

**Formula-only variants beat all pure ML ratio models** — Phase 1 already captures most of the signal.
Production FLT beats both by combining the formula with a learned correction.

</div>

<div style="font-size:0.8em;">

| Approach | Best WAPE |
|---|---|
| **Production FLT** (formula + ML correction) | **3.43%** |
| F+ benchmark | 3.9% |
| Formula HL (blend, deps, mins) | 4.3–4.7% |
| ML ratio HL | 5.1% |
| Formula simple | 5.2% |
| ML ratio simple | 6.0–6.3% |
| ML direct | 6.7–7.5% |
| Baselines | 9–120% |

</div>

---

### Appendix: Fuel Uplift — Step 1: The Base Formula

<div style="font-size:0.84em; margin-bottom:0.4em;">

With corrected departures and minutes in hand, we apply a statistical base formula.
We know historically, per route and aircraft type, how much fuel was uplifted per departure and per flight minute.

</div>

<div style="background:#0d1f2d; border-left:4px solid #f59e0b; padding:12px 16px; margin:10px 0; font-family:monospace; font-size:0.88em;">
base = (kg_per_dep × predicted_deps)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+ (kg_per_min × predicted_mins)
</div>

<div style="font-size:0.84em;">

- Rates computed per **airline + airport + aircraft type**
- **Exponentially-weighted average** (half-life ≈ 200 days) — recent months count more
- `kg_per_dep` and `kg_per_min` separately capture fixed and variable fuel costs

</div>

---

### Appendix: Fuel Uplift — Why the Formula Is Strong

<div style="font-size:0.84em;">

This formula looks simple — but in our comparison it outperformed most trained ML models.

**Why?**

Phase 1 already corrects the schedule inputs (departures and minutes).
With accurate inputs, the per-route fuel rates capture most of the remaining variation.
The formula is physics-grounded: more flying always means more fuel.

Formula alone achieves **4.3–5.2% WAPE** on unseen data.
Production FLT adds one more step to close the remaining gap.

</div>

---

### Appendix: Fuel Uplift — Step 2: The ML Correction

<div style="font-size:0.84em; margin-bottom:0.4em;">

The formula drifts when conditions change — a new aircraft type, a new route, or unusual disruption.
We train a model to predict *how wrong the formula is* and apply a correction ratio.

</div>

<div style="background:#0d1f2d; border-left:4px solid #00d4aa; padding:10px 16px; margin:10px 0; font-family:monospace; font-size:0.88em;">
ratio &nbsp;= actual_uplift ÷ formula_base &nbsp;← learned from history<br>
final &nbsp;= formula_base × predicted_ratio
</div>

<div style="font-size:0.84em; margin-top:0.6em;">

| Ratio | Meaning |
|---|---|
| 1.0 | Formula was right — no adjustment |
| 0.9 | Formula 10% too high — model pulls down |
| 1.1 | Formula 10% too low — model nudges up |

</div>

---

### Appendix: Fuel Uplift — Correction Model Features

<div style="font-size:0.84em;">

The correction model learns from historical mismatches between the formula and actual uplifts.
It picks up signals the formula cannot — e.g. "this airline in summer consistently loads heavier than rates suggest".

| Feature | Role |
|---|---|
| Predicted departures | Activity volume |
| Predicted flight minutes | Duration signal |
| Airline + airport + seat bin | Route-specific bias |
| `days_to_ops` | Accuracy degrades further out |
| Month | Seasonal loading differences |
| Route mean uplift | Context on the base estimate's scale |

</div>

---

### Appendix: Fuel Uplift — Results

<div style="font-size:0.84em; margin-bottom:0.6em;">

The ML correction adds ~**0.9 percentage points** over the formula alone.

| Method | WAPE (Jul–Dec 2025) |
|---|---|
| **Production FLT** (formula + ML correction) | **3.43%** |
| F+ benchmark | 3.9% |
| Formula only — HL-weighted | 4.3–4.7% |
| Formula only — simple rates | 5.2% |
| ML models (direct or ratio) | 5.1–7.5% |

</div>

<div style="font-size:0.84em;">

The two-step design separates two kinds of knowledge:

- The **formula** captures stable, physics-based relationships (more flying = more fuel)
- The **ML model** captures context-dependent drift the formula cannot see

Neither alone matches the combination.

</div>

---

### Appendix: Phase 2 Model Comparison

<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; height:68vh; gap:16px;">
<p style="color:#8899aa; font-size:0.8em;">16 model variants · monthly WAPE by approach · formula vs ML vs production · Jul–Dec 2025</p>
<div style="display:flex; gap: 20px;">
<a href="assets/phase2_comparison.html" target="_blank"
   style="display:inline-block; padding:14px 36px; background:#00d4aa; color:#16213e;
          font-size:1.05em; font-weight:bold; border-radius:6px; text-decoration:none;">
  Open Comparison Chart ↗
</a>
<a href="assets/comparison_drilldown.html" target="_blank"
   style="display:inline-block; padding:14px 36px; background:#756bb1; color:#fff;
          font-size:1.05em; font-weight:bold; border-radius:6px; text-decoration:none;">
  Open Per-Route Drilldown ↗
</a>
</div>
</div>
