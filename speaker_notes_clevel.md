# Speaker Notes — FLT Presentation

---

## Slide 1 — Our Mission

The core problem: fuel planners work from a published schedule that can be weeks or months out of date by the time flights actually operate. Airlines cancel flights, swap aircraft, add extra legs — and the planning system sees none of it. FLT is built to close that gap.

F+ is the current production benchmark. Our goal is simple: beat it consistently and measurably, across airports, airlines, and seasons.

---

## Slide 2 — The Limits of F+

F+ assigns each scheduled event the average uplift from the last three full calendar months for that city pair and scheduled aircraft type, then aggregates. Walk through the four failure modes:

**Blind to schedule changes:** The June schedule might show Air Dolomiti flying 60 times to BLL in October. By October, 35 of those are cancelled. F+ has no way to know — it forecasts all 60. We saw this exact pattern in our test data.

**Last 3 full calendar months:** The rate flips at month boundaries, not day by day. When operations change — airline restructuring, seasonal shifts, route exits — the rate is slow to adapt, updating only when a new full calendar month closes. Same rate whether forecasting tomorrow or 6 months out.

**No history = no forecast:** If there is no uplift history for that city pair + aircraft type in the last 3 months, F+ returns nothing. Charter and cargo operators only publish short-term schedules — so there is no mid- to long-term forecast for those routes. In our test, these routes account for 5.8% of total volume.

**Scheduled type, not actual:** F+ does use aircraft type — but the scheduled type, not what actually operates. If a 777 flies instead of a scheduled A320, F+ applies the A320 rate and is systematically wrong on fuel volume.

---

## Slide 3 — Our Hypothesis

The key insight: instead of trusting the published schedule (which lies), we ask — what will actually fly?

Departures and flight minutes are the two signals that carry almost all the information about how much fuel will be uplifted. More flights = more fuel. Longer flights = more fuel. These signals are stable, learnable, and far more predictive than the static schedule.

The challenge is that at forecast time (6 months out) we only have the schedule, not the actuals. So we build a first model to predict what will actually operate, then feed that into the uplift model.

**On the cargo/charter stat:** Even when we remove 3S and YF entirely (routes F+ literally cannot predict — 5.8% of volume), FLT still wins: 3.1% vs F+ 3.9%. Our advantage is not just from covering routes F+ cannot see — we genuinely outperform on the routes F+ can predict too.

---

## Slide 4 — The FLT Pipeline

Two stages, one pipeline:

**Phase 1 — Schedule Correction:** A LightGBM model has learned, from 20 months of history, how scheduled departures and flight minutes typically drift from what actually operates — broken down by airline, airport, days to operations, and season. It outputs corrected departure counts and flight minute estimates.

**Phase 2 — Uplift Prediction:** Takes Phase 1's corrected numbers as input. The model predicts a correction ratio applied to a formula base (corrected deps × kg/dep + corrected mins × kg/min). Predicting a ratio near 1.0 is far easier to learn than predicting raw kg directly — the target is consistent across all routes regardless of size.

The two stages compound: better schedule correction = better uplift inputs = better uplift prediction.

---

## Slide 5 — Training Data & Test Setup

Three clean boxes showing exactly what went in and what we tested against.

**Phase 1** trained on 20 months (Oct 2023 – Jun 2025) — the longer window is needed because we're learning schedule drift patterns across seasons and horizons up to 220 days.

**Phase 2** trained on 17 months (Jan 2024 – Jun 2025) — uplift actuals started accumulating later.

**Test Setup:** Forecast made on 12 June 2025 for Jul–Dec 2025. The only input was the published schedule as of that date — no future information, no actuals. Ground truth was actual departures, flight minutes, and uplift kg. F+ forecast for the same window is the benchmark.

---

## Slide 6 — FLT vs F+ — Month by Month

Walk through the table:

**July and August:** Both models are accurate in summer peak. Schedule is reliable, routes operating as expected. FLT's edge is modest — 1.6% vs 2.4% and 1.9% vs 2.7%.

**September — the one loss (3.2% vs F+ 2.4%):** June schedule still shows full summer ops. By September, airlines have quietly cancelled seasonal and charter routes without filing it. FLT's Phase 1 model trusts the (wrong) schedule and overcounts departures. F+ survives this accidentally — its 3-month rate was built from July/August actuals which already reflect the summer wind-down in kg-per-flight terms, so even with the wrong flight count the rate compensates. October onwards that trick stops working and F+ collapses while FLT recovers. Fix: Winterflugplan.

**October:** F+ starts to break down as more winter cancellations kick in. FLT 3.7% vs F+ 6.4%.

**November and December:** F+ collapses — 24.2% and 24.1%. Eurowings contracted significantly, charter routes stopped, long-haul frequency dropped. F+ kept forecasting on stale summer patterns. FLT: 7.6% and 6.1%.

---

## Slide 7 — End-to-End Results

This is the full pipeline result — Phase 1 predicts the schedule correction, Phase 2 predicts uplift from those corrected inputs. Use the filters to drill into specific airlines or airports.

The month-by-month table gives the headline numbers. This chart lets you explore the detail behind them.

---

## Slide 8 — Closing the Gap

Walk through the rows in the same order as the Limits slide:

**Schedule drift — Solved:** Phase 1. The delta model is trained on 20 months of history and learns the correction per airline, airport, and horizon.

**Fixed 3-month average / horizon degradation — Solved:** FLT uses recency-weighted rates (not a fixed window) and trains separate models per horizon band (1–90 / 91–220 days), so each specialises on its range.

**No schedule = no forecast — Solved:** Fallback hierarchy: route historical rate → airport average → global mean. FLT can predict any route from day one.

**Aircraft swaps — Partial:** Within-bin swaps (A319 for A320) are handled well via seat_bin + kg/min rates. Cross-bin swaps (A320 for 777) are the gap. For short-range, confirmed aircraft type from ops systems closes this. For long-range, aircraft assignment is not yet decided 6 months out — no system can solve that.

**Station disruptions — Roadmap:** A per-station binary flag (`station_disrupted = True/False`) would zero out uplift for that station and redistribute to alternates. Pre/post processing step — no retraining needed. Requires ops/NOC integration and agreement on redistribution logic. Short-horizon only (1–30 days).

---

## Slide 9 — Roadmap

Five phases, no fixed dates:

**Now — Proof of Concept:** Full pipeline validated on 10 airports, 6 months, 11 airlines.

**Phase 1 — Multi-Scenario Validation:** Winterflugplan integration (closes September gap). More training data. Additional test scenarios including edge cases.

**Phase 2 — Full Airport Expansion:** Scale beyond 10 airports. Real-time SSIM feed. Station disruption flag.

**Phase 3–4 — Production + Extended Horizon:** Live deployment at 4–6 month horizon, then extend to 15–18 months for capacity and budget planning.

**Phase 5 — Full System Launch:** ~400 airports. Continuous retraining. Fully integrated.

Today's 10 airports already represent 96,993 tonnes less misorder across the 6-month test. Every airport added compounds that directly.
