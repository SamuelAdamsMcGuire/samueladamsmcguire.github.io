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

### Appendix: Phase 2 — Approaches Tested

<div style="font-size: 0.8em; margin-bottom: 0.6em;">

- **Baselines** — seasonal mean (Dummy) and simple Ridge regression to anchor the scale
- **ML direct** — HistGradientBoosting predicts kg uplift directly from Phase 1 schedule outputs
- **ML ratio** — model predicts a *correction ratio* on top of a route-mean base; HL variants use exponentially-weighted means (half-life 200 days)
- **Formula (no ML)** — pure `route_mean × predicted deps/mins`; no model trained; simple vs HL-weighted

Key finding: **formula-only variants beat all ML ratio models** — Phase 1 already captures most of the signal.

</div>

<div style="font-size: 0.72em;">

| Approach | Variants | Best WAPE |
|---|---|---|
| Production FLT | Phase 3+4, HL blend + correction | **3.43%** |
| F+ benchmark | 3-month route avg | 3.9% |
| Formula HL | blend HL, deps HL, mins HL | 4.3–4.7% |
| ML ratio HL | Ratio Blend HL | 5.1% |
| Formula simple | blend, mins, deps | 5.2% |
| ML ratio simple | Ratio Blend, Log Ratio, Ratio (dep) | 6.0–6.3% |
| ML direct | Hist Simple, Hist Full | 6.7–7.5% |
| Baselines | Ridge Simple, Ridge Full, Dummy | 9–120% |

</div>

---

### Appendix: Phase 2 Model Comparison

<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; height:42vh; gap:16px;">
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

---

### Appendix: Phase 1 — Approaches Tested

<div style="font-size: 0.8em; margin-bottom: 0.6em;">

- **Delta model** — predicts the *correction* to the schedule (`actual − scheduled`); final prediction = `scheduled + Δ`. Both models use this approach.
- **Direct model** — predicts the actual value outright, ignoring the schedule as a baseline
- **Horizon split** — separate models for short (1–90d) and mid (91–220d) forecasts; captures that near-term and long-range predictions have different dynamics
- **No-split** — single unified model across all horizons (minutes only, since departures currently uses single)
- **No lags** — removes all lag/rolling features; isolates how much historical patterns help
- **Core only** — airline + airport + seat_bin + scheduled + month + days_to_ops; the absolute minimum

</div>

<div style="font-size: 0.72em;">

**Departures** — WAPE on predicted vs actual departures (Jul–Dec 2025):

| Variant | Overall | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---|---|---|---|---|---|---|
| **Delta split short+mid** | **6.1%** | 2.6 | 3.6 | 2.6 | 3.8 | 14.4 | 20.0 |
| Delta full (current — single) | 6.4% | 3.0 | 4.0 | 3.2 | 4.3 | 13.8 | 20.8 |
| Delta no lags | 7.2% | 3.3 | 4.3 | 3.6 | 4.4 | 15.3 | 21.4 |

**Minutes** — WAPE on predicted vs actual flight minutes:

| Variant | Overall | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---|---|---|---|---|---|---|
| **Delta split full (current)** | **5.5%** | 4.1 | 5.3 | 4.7 | 7.7 | 11.1 | 15.2 |
| Delta no split | 7.4% | 4.8 | 6.2 | 6.3 | 6.6 | 9.4 | 18.2 |
| Delta split no lags | 6.5% | 3.6 | 5.5 | 5.6 | 5.9 | 9.2 | 19.0 |

</div>

---

### Appendix: Phase 1 Model Comparison

<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; height:42vh; gap:16px;">
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

### Appendix: Departures — Predict the Correction, Not the Count

<div style="font-size:0.82em; margin-top:0.3em;">

The published schedule is our best starting point. Rather than predicting how many flights will depart from scratch, we predict *how far off the schedule will be* — then apply that correction on top.

</div>

<div style="background:#0d1f2d; border-left:4px solid #00d4aa; padding:12px 18px; margin:12px 0; font-family:monospace; font-size:0.9em;">
predicted departures &nbsp;=&nbsp; scheduled departures &nbsp;+&nbsp; Δ<br>
<span style="color:#8899aa; font-size:0.9em;">where Δ = the correction learned from historical schedule vs actual patterns</span>
</div>

<div style="font-size:0.82em;">

If the schedule says 14 flights but historically this route cancels 2 at this forecast horizon, we predict 12.
If the schedule chronically over-promises on a route, the model learns a consistently negative delta.
If it under-promises (e.g. extra charters get added), the delta is positive.

**Why this works better than predicting from scratch:** The schedule already encodes route structure, airline intent, and seasonality. We only need to learn the *gap* — a much smaller signal that generalises better across routes and time.

**One model, all horizons (1–220 days).** Tested against a short/mid split: 6.4% vs 6.1% WAPE — not enough difference to justify the added complexity.

</div>

---

### Appendix: Departures — What the Model Learns

<div style="font-size:0.82em;">

| Feature | What it captures |
|---|---|
| Scheduled departures | The starting baseline |
| Airline | Each airline has a different pattern of over/under-delivering on its schedule |
| Departure airport | Some airports have more disruption than others |
| Seat bin (aircraft type) | Wide-body vs narrow-body — different swap and cancellation rates |
| `days_to_ops` | How far ahead we are — accuracy degrades with distance; the model adjusts |
| Month | Seasonal patterns: summer schedules over-promise more than winter |
| Lag corrections at 7 / 14 / 28 / 90 / 180 / 365 days | What the actual vs scheduled gap was the last time we were this far out from this route — the model finds repeating correction patterns |

The lag features are the key differentiator. Without them, WAPE rises from 6.4% to 7.2% — the historical pattern of corrections carries real signal.

</div>

---

### Appendix: Flight Minutes — Why We Need a Second Prediction

<div style="font-size:0.82em; margin-bottom:0.4em;">

Predicted departures alone are not enough to estimate fuel. A single long-haul A380 sector burns more than five short A320 hops. We need to know *how long the aircraft will be in the air*.

</div>

<div style="background:#0d1f2d; border-left:4px solid #756bb1; padding:12px 18px; margin:10px 0; font-family:monospace; font-size:0.9em;">
predicted flight minutes &nbsp;=&nbsp; scheduled flight minutes &nbsp;+&nbsp; Δ
</div>

<div style="font-size:0.82em;">

Same delta concept as departures — predict the correction to the schedule, not the absolute value.

Flight minutes captures two things departures cannot:

- **Aircraft type** — a seat bin tells us the aircraft family, but minutes tell us how far it actually flew
- **Routing changes** — a swap from a direct to a connecting route changes minutes even if departure count stays the same

Together, predicted departures and predicted flight minutes feed directly into the fuel rate formula in Phase 2.

</div>

---

### Appendix: Flight Minutes — Two Models for Two Different Problems

<div style="font-size:0.82em; margin-bottom:0.5em;">

Unlike departures (one model), minutes uses **two separate models** — one for the short horizon, one for the mid horizon. This was the single biggest accuracy improvement in Phase 1.

</div>

<div style="display:flex; gap:20px; font-size:0.8em;">

<div style="flex:1; background:#1a2a3a; border-radius:8px; padding:14px 16px;">

**Short horizon (1–90 days)**

Operational decisions are mostly made. Aircraft swaps, known cancellations, and slot changes are partially confirmed. Corrections are smaller and more predictable.

The model learns tight, low-variance corrections.

</div>

<div style="flex:1; background:#1a2a3a; border-radius:8px; padding:14px 16px;">

**Mid horizon (91–220 days)**

The schedule is still highly speculative. Airline planning changes are common. Corrections are larger and more volatile.

The model learns to discount the schedule more aggressively.

</div>

</div>

<div style="font-size:0.82em; margin-top:14px;">

Mixing both horizons into one model forces it to average between two very different regimes — it handles neither well.
Splitting them reduced WAPE from **7.4% → 5.5%** on unseen Jul–Dec 2025 data.

</div>

---

### Appendix: Flight Minutes — What the Models Learn

<div style="font-size:0.82em;">

Each model (short and mid) uses the same feature set — the only difference is the `days_to_ops` range each is trained and applied to.

| Feature | What it captures |
|---|---|
| Scheduled flight minutes | The baseline |
| Airline + airport + seat bin | Route-specific and aircraft-specific patterns |
| `days_to_ops` | Forecast horizon within the band (1–90 or 91–220) |
| Month | Seasonal routing shifts |
| Lag departure corrections (7 / 14 / 28 / 90 / 180 / 365 days) | Past departure accuracy at this route and horizon |
| Lag minute corrections (7 / 14 / 28 / 90 / 180 / 365 days) | Past minute accuracy at this route and horizon |

The lag features are computed per route **and** per `days_to_ops` band — so the model separately learns "at 60 days out this route runs short" and "at 150 days out it over-extends". Without lags, WAPE rises from 5.5% to 6.5%.

</div>

---

### Appendix: Fuel Uplift — Step 1: The Statistical Formula

<div style="font-size:0.82em; margin-bottom:0.5em;">

Once we have predicted departures and predicted flight minutes, we apply a simple but powerful formula to estimate fuel uplift. We know historically, per route and aircraft type, how much fuel was uplifted per departure and per flight minute.

</div>

<div style="background:#0d1f2d; border-left:4px solid #f59e0b; padding:14px 18px; margin:10px 0; font-family:monospace; font-size:0.9em;">
base_estimate &nbsp;=&nbsp; (kg_per_dep &nbsp;× predicted_deps)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+ (kg_per_min &nbsp;× predicted_mins)
</div>

<div style="font-size:0.82em;">

The rates (`kg_per_dep`, `kg_per_min`) are computed separately for every airline + airport + aircraft type combination, using an **exponentially-weighted moving average** (half-life ≈ 200 days) so that recent data counts more than old data.

This formula is simple — but it is strong. In our Phase 2 comparison, it outperformed most trained ML models. The reason: Phase 1 already corrects the schedule inputs, and the fuel rates capture most of the remaining variation.

The formula alone achieves WAPE of **4.3–5.2%** on unseen data, depending on the rate weighting method. Production FLT goes further with a second step.

</div>

---

### Appendix: Fuel Uplift — Step 2: The ML Correction

<div style="font-size:0.82em; margin-bottom:0.4em;">

The statistical formula is good but not perfect. It drifts when conditions change — a new aircraft type on a route, a new hub, or a period of unusual disruption. To handle this, we train a model to predict *how wrong the formula is likely to be* and correct it.

</div>

<div style="background:#0d1f2d; border-left:4px solid #00d4aa; padding:14px 18px; margin:10px 0; font-family:monospace; font-size:0.9em;">
correction_ratio &nbsp;=&nbsp; actual_uplift &nbsp;÷&nbsp; base_estimate &nbsp;&nbsp;← learned from history<br><br>
final_uplift &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp; base_estimate &nbsp;×&nbsp; predicted_ratio
</div>

<div style="font-size:0.82em; margin-bottom:0.8em;">

| Ratio | Meaning |
|---|---|
| 1.0 | Formula was exactly right — no change |
| 0.9 | Formula was 10% too high — model pulls us down |
| 1.1 | Formula was 10% too low — model nudges us up |

The model learns from historical mismatches between the formula and actual uplifts. It picks up signals like: "this airline in summer consistently loads more than the formula says" or "this route's rate hasn't updated to reflect the new aircraft yet".

</div>

<div style="font-size:0.82em;">

**Features the correction model uses:** predicted departures, predicted flight minutes, airline + airport + seat bin, `days_to_ops`, month, and the route mean uplift (to anchor the scale).

**Result:** Production FLT with the correction step achieves **3.43% WAPE** vs 3.9% for F+ and 4.3–5.2% for the formula alone on unseen Jul–Dec 2025 data. The ML correction step adds ~0.9 percentage points over formula-only.

</div>
