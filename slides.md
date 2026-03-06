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

### Our Mission

```{=html}
<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; margin-top:2em; text-align:center; gap:1.6em;">

  <div style="font-size:1.6em; font-weight:bold; color:#e0e0e0; line-height:1.3;">
    Build a fuel uplift forecast that<br>
    <span style="color:#00d4aa;">outperforms the current standard.</span>
  </div>

  <div style="width:60px; height:3px; background:#00d4aa; border-radius:2px;"></div>

  <div style="font-size:1em; color:#aaa; max-width:880px; line-height:1.8;">
    F+ is the benchmark.<br>We want to beat it — consistently, measurably, and in production.
  </div>

</div>
```

---

### How F+ Works

<div style="font-size: 0.8em; margin-bottom: 0.7em;">

For each event in the published schedule, F+ looks up the **average uplift from the last three full calendar months** on that City Pair and Aircraft Type — then sums everything up.

</div>

```{=html}
<div style="zoom: 1.35;">
<div class="mermaid">
%%{init: {'theme': 'dark', 'flowchart': {'nodeSpacing': 40, 'rankSpacing': 50}}}%%
flowchart LR
    S["Published Schedule"]
    H["Last 3 months history\nCity Pair + Aircraft Type"]
    A["Average uplift\nper flight"]
    P["Predicted Uplift"]
    S --> A
    H --> A
    A --> P
    style S fill:#2a3a5a,stroke:#5a7aaa,color:#fff
    style H fill:#2a3a5a,stroke:#5a7aaa,color:#fff
    style A fill:#2a2a4a,stroke:#888,color:#fff
    style P fill:#1a4a3a,stroke:#00d4aa,color:#fff
</div>
</div>
```

---

### The Limits of F+

<div style="font-size:0.85em; color:#aaa; margin-bottom:0.5em;">Five structural weaknesses — FLT was built to address them.</div>

::: incremental

- **Schedule drift** — the published schedule is never what actually flies: cancellations, swaps and additions happen constantly
- **Horizon degradation** — the further out the forecast, the worse it gets: schedule diverges from reality over time
- **No route history** — new and seasonal routes have no city-pair data, so F+ returns no estimate
- **Station disruptions** — if an airport can't fuel, aircraft tank elsewhere; F+ doesn't detect or adjust
- **Aircraft swaps** — a 180-seat aircraft replacing a 250-seat one carries very different fuel requirements

:::

---

### Our Hypothesis

<div style="font-size: 0.9em; margin: 0.6em 0;">

**Don't rely on city pairs or fixed routes.**

</div>

::: incremental

- City-pair history breaks down for new routes, swaps and disruptions
- Instead: use **flight volume** as the signal — encoded as **departure count** and **flight minutes**
- These are measurable, forecastable quantities that reflect what will actually operate
- Aircraft size (**seat bin**) captures fuel efficiency without needing route-specific history
- The model learns: *given this many flights of this size from this airport — how much fuel?*

:::

---

### What We Tried — Uplift Model

```{=html}
<div style="margin-top:0.3em; font-size:0.78em;">
    <div style="font-size:0.75em; color:#aaa; margin-bottom:0.25em;">Models</div>
    <div style="display:flex; flex-wrap:wrap; gap:5px; margin-bottom:0.6em;">
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Dummy</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Linear Regression</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Ridge</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">XGBoost</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">GradientBoosting</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">HistGBM</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">LightGBM</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Direct target</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Ratio target</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Quantile target</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Blend (deps + mins)</span>
    </div>
    <div style="font-size:0.75em; color:#aaa; margin-bottom:0.25em;">Features</div>
    <div style="display:flex; flex-wrap:wrap; gap:5px;">
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">no_of_deps · flight_mins</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">days_to_ops</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">airline · airport · seat_bin</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">airline × airport</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">month · quarter · day_of_year sin/cos</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">schedule_season</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">hub · charter · long_haul · leisure</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">mins/dep · mins×hub · mins×charter</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">route_mean/dep · route_mean/min</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">seasonal route mean (SS/WS)</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">outlier removal</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">feature scaling</span>
    </div>
</div>
```

---

### What We Use — Uplift Model

```{=html}
<div style="margin-top:0.5em;">
  <div style="font-size:0.7em; color:#aaa; margin-bottom:0.4em; text-transform:uppercase; letter-spacing:0.08em;">Approach</div>
  <div style="display:flex; flex-wrap:wrap; gap:8px; margin-bottom:1.2em;">
    <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:4px 12px;border-radius:20px;font-size:0.72em;">Formula blend</span>
    <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:4px 12px;border-radius:20px;font-size:0.72em;">Recency-weighted rates</span>
    <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:4px 12px;border-radius:20px;font-size:0.72em;">HistGBM correction ratio</span>
  </div>
  <div style="font-size:0.7em; color:#aaa; margin-bottom:0.4em; text-transform:uppercase; letter-spacing:0.08em;">Features (correction model)</div>
  <div style="display:flex; flex-wrap:wrap; gap:8px;">
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">airline</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">airport</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">airline × airport</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">seat_bin</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">no_of_deps</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">flight_mins</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">flight_mins / dep</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">flight_mins × deps</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">month</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">schedule_season</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">hub_indicator</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">charter_airline</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">long_haul</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">leisure_route</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">mins × hub</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">mins × charter</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">route_mean uplift / flight</span>
  </div>
</div>
```

---

### How It Works — The Equation

```{=html}
<div style="margin-top:0.3em; font-size:0.82em;">

  <!-- Base formula -->
  <div style="text-align:center; margin-bottom:0.6em;">
    <div style="font-size:0.72em; color:#aaa; text-transform:uppercase; letter-spacing:0.08em; margin-bottom:0.3em;">Base estimate</div>
    <div style="display:inline-block; background:#1a1a2e; border:1px solid #5a6a2a; border-radius:8px; padding:0.4em 1.4em; font-family:monospace; font-size:1.15em; color:#fcc419;">
      base &nbsp;=&nbsp; ½ (rate<sub style="font-size:0.75em;">dep</sub> × deps &nbsp;+&nbsp; rate<sub style="font-size:0.75em;">min</sub> × mins)
    </div>
    <div style="font-size:0.78em; color:#aaa; margin-top:0.3em;">Recency-weighted kg/dep and kg/min rates blended equally</div>
  </div>

  <!-- Train / Predict columns -->
  <div style="display:flex; gap:1em;">

    <div style="flex:1; background:#1a1a2e; border:1px solid #3a4a6a; border-radius:8px; padding:0.5em 0.9em;">
      <div style="font-size:0.7em; color:#9ecae1; text-transform:uppercase; letter-spacing:0.08em; margin-bottom:0.3em;">① Training — learning the ratio</div>
      <div style="font-family:monospace; font-size:1em; color:#e0e0e0; margin-bottom:0.25em;">
        target &nbsp;=&nbsp; <span style="color:#00d4aa;">actual uplift</span> &nbsp;÷&nbsp; <span style="color:#fcc419;">base</span>
      </div>
      <div style="font-size:0.78em; color:#aaa; line-height:1.4;">
        For every historical month where we know the outcome, we compute this ratio and train HistGBM to predict it from route and schedule features.
      </div>
    </div>

    <div style="flex:1; background:#1a1a2e; border:1px solid #2a5a2a; border-radius:8px; padding:0.5em 0.9em;">
      <div style="font-size:0.7em; color:#74c476; text-transform:uppercase; letter-spacing:0.08em; margin-bottom:0.3em;">② Prediction — applying the ratio</div>
      <div style="font-family:monospace; font-size:1em; color:#e0e0e0; margin-bottom:0.25em;">
        result &nbsp;=&nbsp; <span style="color:#fcc419;">base</span> &nbsp;×&nbsp; <span style="color:#00d4aa;">predicted ratio</span>
      </div>
      <div style="font-size:0.78em; color:#aaa; line-height:1.4;">
        The ratio clusters close to 1.0 — the formula carries the load. The model corrects where the formula systematically over- or underestimates.
      </div>
    </div>

  </div>
</div>
```

---

### Uplift Model Architecture

**Statistical base + ML correction ratio**

```{=html}
<div style="zoom: 1.2;">
<div class="mermaid">
%%{init: {'theme': 'dark', 'flowchart': {'nodeSpacing': 35, 'rankSpacing': 55}}}%%
flowchart LR
    D["Actual departures"]
    M["Actual flight minutes"]
    R["Route mean rates\nkg/dep · kg/min\n(recency-weighted)"]
    B["Base estimate\n½(rate/dep × deps\n+ rate/min × mins)"]
    ML["ML correction ratio\nLightGBM\n≈ 1.0 nudge"]
    P["Predicted uplift"]

    D --> B
    M --> B
    R --> B
    B --> ML
    ML -->|"base × ratio"| P

    style D fill:#1a3a5a,stroke:#5a8aaa,color:#fff
    style M fill:#1a3a5a,stroke:#5a8aaa,color:#fff
    style R fill:#3a2a1a,stroke:#aa8a5a,color:#fff
    style B fill:#2a2a4a,stroke:#888,color:#fff
    style ML fill:#2a4a2a,stroke:#5aaa5a,color:#fff
    style P fill:#1a4a3a,stroke:#00d4aa,color:#fff
    click D showNodeDetail
    click M showNodeDetail
    click R showNodeDetail
    click B showNodeDetail
    click ML showNodeDetail
    click P showNodeDetail
</div>
</div>
```

<div style="font-size: 0.78em; margin-top: 0.8em; color: #aaa; line-height: 1.6;">

Predicting raw uplift kg directly is hard — a busy hub route can carry 10× more fuel than a thin leisure route.
By predicting a **ratio near 1.0** instead, the model sees a normalised target that is consistent across all routes and airline types.
This makes learning easier and generalisation to new routes much more reliable.
<span style="color:#555; font-size:0.88em;">↑ Click any node for details</span>

</div>



---

### Training and Test Data — Uplift Model

```{=html}
<div style="display:flex; gap:1.5em; margin-top:0.2em; font-size:0.8em;">
  <div style="flex:1; background:#1a2a1a; border:1px solid #3a6a4a; border-radius:8px; padding:0.5em 0.9em;">
    <div style="color:#74c476; font-weight:bold; margin-bottom:0.3em; text-transform:uppercase; font-size:0.85em; letter-spacing:0.06em;">Training Data</div>
    <table style="width:100%; border-collapse:collapse; color:#ccc;">
      <tr><td style="padding:2px 0; color:#aaa;">Period</td><td style="padding:2px 0;">Jan 2024 – Jun 2025 <span style="color:#aaa;">(17 months)</span></td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Airports</td><td style="padding:2px 0;">BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Airlines</td><td style="padding:2px 0;">11 airlines · 64 airline–airport pairs</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Target</td><td style="padding:2px 0;">Actual uplift kg · per day · per airport · per airline · per seat_bin</td></tr>
    </table>
  </div>
  <div style="flex:1; background:#1a1a2a; border:1px solid #3a5a8a; border-radius:8px; padding:0.5em 0.9em;">
    <div style="color:#9ecae1; font-weight:bold; margin-bottom:0.3em; text-transform:uppercase; font-size:0.85em; letter-spacing:0.06em;">Test Vector — Perfect Schedule</div>
    <table style="width:100%; border-collapse:collapse; color:#ccc;">
      <tr><td style="padding:2px 0; color:#aaa;">Forecast date</td><td style="padding:2px 0;">12 June 2025</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Test window</td><td style="padding:2px 0;">Jul – Dec 2025 <span style="color:#aaa;">(6 months)</span></td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Departures</td><td style="padding:2px 0;"><span style="color:#00d4aa; font-weight:bold;">Actual departures</span> — what really flew</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Flight minutes</td><td style="padding:2px 0;"><span style="color:#00d4aa; font-weight:bold;">Actual flight minutes</span> — what really flew</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Purpose</td><td style="padding:2px 0;">Isolate uplift model quality from schedule error</td></tr>
    </table>
  </div>
</div>
<div style="margin-top:0.4em; font-size:0.76em; color:#aaa; text-align:center;">
  Using actual inputs removes schedule noise — this measures the ceiling of our approach.
</div>
```

---

### Uplift Model Results — Using Perfect Schedule

<iframe scrolling="no" style="border:none;" seamless="seamless"
  data-src="assets/oracle_comparison.html" height="615" width="100%"></iframe>

---

### The Missing Piece — Schedule Reality

<div style="font-size: 0.9em; margin: 0.8em 0 1em 0;">

The uplift model works. The bottleneck is the **inputs**.

</div>

::: incremental

- The previous results used **actual departures and flight minutes** — what really flew
- At prediction time we only have the **published schedule** — which can differ from actually flies
- The closer our departure and flight-minute estimates are to reality, the closer our uplift forecast gets to those results
- So we built models to **predict what will actually operate** — not what was planned
- **Closing the gap between scheduled and actual is Phase 1 — the schedule correction models.**


:::

---

### What We Tried — Schedule Correction

```{=html}
<div style="margin-top:0.3em; font-size:0.78em;">
    <div style="font-size:0.75em; color:#aaa; margin-bottom:0.25em;">Models</div>
    <div style="display:flex; flex-wrap:wrap; gap:5px; margin-bottom:0.6em;">
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Linear · Lasso · Ridge · ElasticNet</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">SVR</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">KNeighbors</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Random Forest</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">XGBoost</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">CatBoost</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">HistGBM</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">LightGBM</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">MultiOutputRegressor (joint)</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Separate models per target</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Delta target</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Direct target</span>
      <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:3px 9px;border-radius:20px;font-size:0.78em;">Horizon split 1–90 / 91–220</span>
    </div>
    <div style="font-size:0.75em; color:#aaa; margin-bottom:0.25em;">Features</div>
    <div style="display:flex; flex-wrap:wrap; gap:5px;">
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">sched_deps · sched_mins</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">days_to_ops</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">airline · airport · seat_bin</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">airline × airport</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">month · quarter · day_of_week</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">day_of_year sin/cos · schedule_season</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">short/long haul · charter · leisure month</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">lag Δdeps / Δmins — 7/14/28/90/180/365d</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">rolling mean (7d)</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">German holidays</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">route historical bias</span>
      <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:3px 9px;border-radius:20px;font-size:0.78em;">schedule reliability score</span>
    </div>
</div>
```

---

### What We Use — Schedule Models

```{=html}
<div style="margin-top:0.5em;">
  <div style="font-size:0.7em; color:#aaa; margin-bottom:0.4em; text-transform:uppercase; letter-spacing:0.08em;">Approach</div>
  <div style="display:flex; flex-wrap:wrap; gap:8px; margin-bottom:1.2em;">
    <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:4px 12px;border-radius:20px;font-size:0.72em;">LightGBM</span>
    <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:4px 12px;border-radius:20px;font-size:0.72em;">Delta model — predict the change</span>
    <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:4px 12px;border-radius:20px;font-size:0.72em;">Horizon split: 1–90 / 91–220 days</span>
    <span style="background:#1a3a5a;border:1px solid #3a6a9a;color:#9ecae1;padding:4px 12px;border-radius:20px;font-size:0.72em;">Separate models for deps &amp; mins</span>
  </div>
  <div style="font-size:0.7em; color:#aaa; margin-bottom:0.4em; text-transform:uppercase; letter-spacing:0.08em;">Features</div>
  <div style="display:flex; flex-wrap:wrap; gap:8px;">
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">airline</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">airport</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">airline × airport</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">seat_bin</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">sched_departures</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">sched_flight_mins</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">days_to_ops</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">month</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">day_of_week</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">quarter · day_of_year sin/cos</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">schedule_season</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">is_short_haul · is_long_haul</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">is_charter · is_leisure_month</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">lag Δdeps 7/14/28/90/180/365d</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:4px 12px;border-radius:20px;font-size:0.72em;">lag Δmins 7/14/28/90/180/365d</span>
  </div>
</div>
```

---

### How It Works — Schedule Equation

```{=html}
<div style="margin-top:0.3em; font-size:0.82em;">

  <!-- Base formula -->
  <div style="text-align:center; margin-bottom:0.6em;">
    <div style="font-size:0.72em; color:#aaa; text-transform:uppercase; letter-spacing:0.08em; margin-bottom:0.3em;">Correction target</div>
    <div style="display:inline-block; background:#1a1a2e; border:1px solid #5a6a2a; border-radius:8px; padding:0.4em 1.4em; font-family:monospace; font-size:1.15em; color:#fcc419;">
      Δ &nbsp;=&nbsp; actual &nbsp;−&nbsp; scheduled
    </div>
    <div style="font-size:0.78em; color:#aaa; margin-top:0.3em;">One delta model for departures, one for flight minutes</div>
  </div>

  <!-- Train / Predict columns -->
  <div style="display:flex; gap:1em;">

    <div style="flex:1; background:#1a1a2e; border:1px solid #3a4a6a; border-radius:8px; padding:0.5em 0.9em;">
      <div style="font-size:0.7em; color:#9ecae1; text-transform:uppercase; letter-spacing:0.08em; margin-bottom:0.3em;">① Training — learning the delta</div>
      <div style="font-family:monospace; font-size:1em; color:#e0e0e0; margin-bottom:0.25em;">
        target &nbsp;=&nbsp; <span style="color:#00d4aa;">actual</span> &nbsp;−&nbsp; <span style="color:#fcc419;">scheduled</span>
      </div>
      <div style="font-size:0.78em; color:#aaa; line-height:1.4;">
        For every historical day we know what was scheduled and what actually flew. LightGBM learns to predict the gap from route, horizon, and lag features.
      </div>
    </div>

    <div style="flex:1; background:#1a1a2e; border:1px solid #2a5a2a; border-radius:8px; padding:0.5em 0.9em;">
      <div style="font-size:0.7em; color:#74c476; text-transform:uppercase; letter-spacing:0.08em; margin-bottom:0.3em;">② Prediction — applying the delta</div>
      <div style="font-family:monospace; font-size:1em; color:#e0e0e0; margin-bottom:0.25em;">
        result &nbsp;=&nbsp; <span style="color:#fcc419;">scheduled</span> &nbsp;+&nbsp; <span style="color:#00d4aa;">predicted Δ</span>
      </div>
      <div style="font-size:0.78em; color:#aaa; line-height:1.4;">
        Predicting a delta near zero is easier than predicting an absolute count from scratch — the schedule is already a strong baseline, and the model just corrects its known biases.
      </div>
    </div>

  </div>
</div>
```

---

### Schedule Correction Model Architecture

**Four models run in parallel — split by horizon**

```{=html}
<div style="zoom: 1.2;">
<div class="mermaid">
%%{init: {'theme': 'dark', 'flowchart': {'nodeSpacing': 30, 'rankSpacing': 55}}}%%
flowchart LR
    S["Published Schedule\n(SSIM)"]

    S --> H1["days_to_ops\n1 – 90"]
    S --> H2["days_to_ops\n91 – 220"]

    H1 -->|"Δ deps"| D1["Departures Model\nHistGBM"]
    H1 -->|"Δ mins"| M1["Minutes Model\nHistGBM"]
    H2 -->|"Δ deps"| D2["Departures Model\nHistGBM"]
    H2 -->|"Δ mins"| M2["Minutes Model\nHistGBM"]

    D1 --> PD["Predicted\nActual Departures"]
    D2 --> PD
    M1 --> PM["Predicted\nActual Flight Minutes"]
    M2 --> PM

    style S fill:#2a2a3a,stroke:#888,color:#fff
    style H1 fill:#2a2a3a,stroke:#666,color:#bbb
    style H2 fill:#2a2a3a,stroke:#666,color:#bbb
    style D1 fill:#1a3a5a,stroke:#5a8aaa,color:#fff
    style M1 fill:#1a3a5a,stroke:#5a8aaa,color:#fff
    style D2 fill:#1a3050,stroke:#4a7a9a,color:#ddd
    style M2 fill:#1a3050,stroke:#4a7a9a,color:#ddd
    style PD fill:#1a4a3a,stroke:#00d4aa,color:#fff
    style PM fill:#1a4a3a,stroke:#00d4aa,color:#fff
    click S showNodeDetail
    click H1 showNodeDetail
    click H2 showNodeDetail
    click D1 showNodeDetail
    click M1 showNodeDetail
    click D2 showNodeDetail
    click M2 showNodeDetail
    click PD showNodeDetail
    click PM showNodeDetail
</div>
</div>
```

<div style="font-size: 0.78em; margin-top: 0.6em; color: #aaa;">

Near-term (1–90 days) and long-range (91–220 days) schedules behave differently — splitting by horizon lets each model specialise.
<span style="color:#555; font-size:0.88em;">↑ Click any node for details</span>

</div>
---

### Training and Test Data — Schedule Models

```{=html}
<div style="display:flex; gap:1.5em; margin-top:0.2em; font-size:0.8em;">
  <div style="flex:1; background:#1a2a1a; border:1px solid #3a6a4a; border-radius:8px; padding:0.5em 0.9em;">
    <div style="color:#74c476; font-weight:bold; margin-bottom:0.3em; text-transform:uppercase; font-size:0.85em; letter-spacing:0.06em;">Training Data</div>
    <table style="width:100%; border-collapse:collapse; color:#ccc;">
      <tr><td style="padding:2px 0; color:#aaa;">Period</td><td style="padding:2px 0;">Oct 2023 – Jun 2025 <span style="color:#aaa;">(20 months)</span></td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Airports</td><td style="padding:2px 0;">BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Airlines</td><td style="padding:2px 0;">11 airlines · 64 airline–airport pairs</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Horizon</td><td style="padding:2px 0;">1 – 220 days to operations</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Target</td><td style="padding:2px 0;">Δ departures · Δ flight minutes · per day · per airport · per airline · per seat_bin</td></tr>
    </table>
  </div>
  <div style="flex:1; background:#1a1a2a; border:1px solid #3a5a8a; border-radius:8px; padding:0.5em 0.9em;">
    <div style="color:#9ecae1; font-weight:bold; margin-bottom:0.3em; text-transform:uppercase; font-size:0.85em; letter-spacing:0.06em;">Test Vector</div>
    <table style="width:100%; border-collapse:collapse; color:#ccc;">
      <tr><td style="padding:2px 0; color:#aaa;">Forecast date</td><td style="padding:2px 0;">12 June 2025</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Test window</td><td style="padding:2px 0;">Jul – Dec 2025 <span style="color:#aaa;">(6 months)</span></td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Input</td><td style="padding:2px 0;">Published schedule as of Jun 12 — scheduled deps + mins</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Ground truth</td><td style="padding:2px 0;"><span style="color:#00d4aa; font-weight:bold;">Actual departures</span> + <span style="color:#00d4aa; font-weight:bold;">actual flight minutes</span></td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Metric</td><td style="padding:2px 0;">WAPE on predicted vs actual deps / mins</td></tr>
    </table>
  </div>
</div>
<div style="margin-top:0.4em; font-size:0.76em; color:#aaa; text-align:center;">
  Models see only the schedule available at forecast time — no future information.
</div>
```

---

### Schedule Model Results — Departures

<iframe scrolling="no" style="border:none;" seamless="seamless"
  data-src="assets/phase1_comparison_deps.html" height="615" width="100%"></iframe>

---

### Schedule Model Results — Flight Minutes

<iframe scrolling="no" style="border:none;" seamless="seamless"
  data-src="assets/phase1_comparison_mins.html" height="615" width="100%"></iframe>

---

### Putting It All Together

::: incremental

- Phase 1 corrects the published schedule — predicted departures and flight minutes replace the raw SSIM inputs
- Those corrected values feed directly into the uplift model as if they were the real schedule
- Recency-weighted fuel rates (kg/dep · kg/min) convert the corrected flight volume into a base estimate
- A learned correction ratio fine-tunes where the formula systematically drifts
- The result: a forecast that adapts to what will actually fly — not just what was planned

:::

---

### The Complete FLT Pipeline

```{=html}
<div style="zoom: 1.1;">
<div class="mermaid">
%%{init: {'theme': 'dark', 'flowchart': {'useMaxWidth': false, 'nodeSpacing': 35, 'rankSpacing': 45}}}%%
flowchart TB
    subgraph input["Input"]
        S["Published Schedule (SSIM)"]
    end
    subgraph phase1["Phase 1: Schedule Correction"]
        direction LR
        P1A["Departure Delta Model\n1–90 days · 91–220 days"]
        P1B["Flight Minutes Delta Model\n1–90 days · 91–220 days"]
    end
    subgraph phase2["Phase 2: Uplift Prediction"]
        direction TB
        RM["Recency-Weighted Rates\nkg/dep · kg/min"]
        BLEND["Base Estimate"]
        FB["Fallback: route / airport / global"]
        CR["ML Correction Ratio"]
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
</div>
```

---



### Validation Setup

```{=html}
<div style="display:flex; gap:1.5em; margin-top:0.4em; font-size:0.8em;">
  <div style="flex:1; background:#1a2a1a; border:1px solid #3a6a4a; border-radius:8px; padding:0.7em 0.9em;">
    <div style="color:#74c476; font-weight:bold; margin-bottom:0.4em; text-transform:uppercase; font-size:0.85em; letter-spacing:0.06em;">Scope</div>
    <table style="width:100%; border-collapse:collapse; color:#ccc;">
      <tr><td style="padding:2px 0; color:#aaa;">Airports</td><td style="padding:2px 0;">BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Airlines</td><td style="padding:2px 0;">LH, OS, LX, SN, EW, EN, WK, 4Y, YF, XQ, 3S</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Combinations</td><td style="padding:2px 0;">64 active airline–airport pairs</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Total volume</td><td style="padding:2px 0;">1.61M t uplift</td></tr>
    </table>
  </div>
  <div style="flex:1; background:#1a1a2a; border:1px solid #3a5a8a; border-radius:8px; padding:0.7em 0.9em;">
    <div style="color:#9ecae1; font-weight:bold; margin-bottom:0.4em; text-transform:uppercase; font-size:0.85em; letter-spacing:0.06em;">Test Conditions</div>
    <table style="width:100%; border-collapse:collapse; color:#ccc;">
      <tr><td style="padding:2px 0; color:#aaa;">Forecast date</td><td style="padding:2px 0;">12 June 2025</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Test window</td><td style="padding:2px 0;">Jul – Dec 2025 <span style="color:#aaa;">(6 months)</span></td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Benchmark</td><td style="padding:2px 0;">F+ — current production system</td></tr>
      <tr><td style="padding:2px 0; color:#aaa;">Metric</td><td style="padding:2px 0;">WAPE % · Misorder (t)</td></tr>
    </table>
  </div>
</div>
<div style="margin-top:0.7em; font-size:0.76em; color:#aaa; text-align:center;">
  Same forecast date and inputs for both FLT and F+ — apples-to-apples comparison.
</div>
```

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
| **Overall Error** | **3.1%** | 3.9% |
| **6-Month Total Misorder** | 50,149 t | 63,452 t |

</div>


---

### Why the Formula Approach Works

<div style="font-size:0.84em; margin-bottom:0.6em;">

**The formula does the heavy lifting. The ML correction handles the edges.** 

</div>

<div style="font-size:0.8em;">

| Approach | Best WAPE |
|---|---|
| **Current FLT** (formula + ML correction) | **3.1%** |
| F+ benchmark | 3.9% |
| Formula HL (blend, deps, mins) | 4.3–4.7% |
| ML ratio HL | 5.1% |
| Formula simple | 5.2% |
| ML ratio simple | 6.0–6.3% |
| ML direct | 6.7–7.5% |
| Baselines | 9–120% |

</div>

---

### Where We Win Big — Unscheduled Flights

**XQ-KEF, July 2025**

<div style="font-size: 0.85em;">

| | Scheduled | Actual | FLT Predicted |
|---|---|---|---|
| Flights | 0 | 2 | 2 |
| Uplift | — | 15,854 kg | 18,909 kg |

</div>

<div style="margin-top: 1em;">

F+ has **no prediction** — it can't see what wasn't scheduled.

FLT caught the unscheduled flights using historical patterns.

</div>

---

### FLT End-to-End Results

<iframe scrolling="no" style="border:none;" seamless="seamless"
  data-src="assets/comparison_drilldown.html" height="615" width="100%"></iframe>

---

### How FLT Closes the Gap

```{=html}
<div style="margin-top:0.3em; font-size:0.73em;">
  <table style="width:100%; border-collapse:collapse;">
    <thead>
      <tr style="border-bottom:1px solid #3a3a4a;">
        <th style="text-align:left; padding:0.28em 0.6em; color:#aaa; font-weight:normal; font-size:0.85em; text-transform:uppercase; letter-spacing:0.06em; width:28%;">F+ Limitation</th>
        <th style="text-align:left; padding:0.28em 0.6em; color:#aaa; font-weight:normal; font-size:0.85em; text-transform:uppercase; letter-spacing:0.06em;">How FLT Addresses It</th>
        <th style="text-align:center; padding:0.28em 0.6em; color:#aaa; font-weight:normal; font-size:0.85em; text-transform:uppercase; letter-spacing:0.06em; width:11%;">Status</th>
      </tr>
    </thead>
    <tbody>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.32em 0.6em; color:#e0e0e0;">Schedule drift</td>
        <td style="padding:0.32em 0.6em; color:#ccc;">Phase 1 models predict actual departures &amp; flight minutes from the published schedule</td>
        <td style="padding:0.32em 0.6em; text-align:center;"><span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:2px 10px;border-radius:20px;font-size:0.85em;">Solved</span></td>
      </tr>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.32em 0.6em; color:#e0e0e0;">Horizon degradation</td>
        <td style="padding:0.32em 0.6em; color:#ccc;">Horizon-split models (1–90 / 91–220 days) each specialise on their forecast range</td>
        <td style="padding:0.32em 0.6em; text-align:center;"><span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:2px 10px;border-radius:20px;font-size:0.85em;">Solved</span></td>
      </tr>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.32em 0.6em; color:#e0e0e0;">No route history</td>
        <td style="padding:0.32em 0.6em; color:#ccc;">Fallback hierarchy: route rates → airport means → global mean. No city-pair history required.</td>
        <td style="padding:0.32em 0.6em; text-align:center;"><span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#74c476;padding:2px 10px;border-radius:20px;font-size:0.85em;">Solved</span></td>
      </tr>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.32em 0.6em; color:#e0e0e0;">Station disruptions</td>
        <td style="padding:0.32em 0.6em; color:#ccc;">Uplift ratio monitoring to detect anomalies and flag volume redistribution across stations</td>
        <td style="padding:0.32em 0.6em; text-align:center;"><span style="background:#3a2a1a;border:1px solid #7a5a2a;color:#fcc419;padding:2px 10px;border-radius:20px;font-size:0.85em;">Roadmap</span></td>
      </tr>
      <tr>
        <td style="padding:0.32em 0.6em; color:#e0e0e0;">Aircraft swaps</td>
        <td style="padding:0.32em 0.6em; color:#ccc;">seat_bin + kg/min rates capture aircraft size; within-bin swaps handled well</td>
        <td style="padding:0.32em 0.6em; text-align:center;"><span style="background:#2a2a1a;border:1px solid #5a5a3a;color:#ffd966;padding:2px 10px;border-radius:20px;font-size:0.85em;">Partial</span></td>
      </tr>
    </tbody>
  </table>
</div>
```

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


---

### Roadmap

```{=html}
<div style="zoom: 1.35;">
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
</div>
```

---

### The Opportunity

<div style="font-size: 0.85em;">

- **Today:** 10 departure airports → **13,303 t** less misorder in 6 months
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

- **5 out of 6 months, 21% less misorder**
- 10 airports, 11 airlines, 64 airline–airport combinations — 6 months validated
- Airline-agnostic: adding airlines = adding data
- Continue the project and expand coverage

:::

</div>

```{=html}
<!-- build: 2026-02-25 -->
```

