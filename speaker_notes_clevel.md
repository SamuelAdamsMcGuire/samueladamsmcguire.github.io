# Speaker Notes — FLT C-Level Deck

---

## Slide 1 — Our Mission

The core problem: fuel planners work from a published schedule that can be weeks or months out of date by the time flights actually operate. Airlines cancel flights, swap aircraft, add extra legs — and the planning system sees none of it. FLT is built to close that gap.

F+ is the current production benchmark. Our goal is simple: beat it consistently and measurably, across airports, airlines, and seasons.

---

## Slide 2 — The Limits of F+

Walk through the three failure modes:

**Blind to cancellations:** The June schedule might show Air Dolomiti flying 60 times to BLL in October. By October, 35 of those are cancelled. F+ has no way to know — it forecasts all 60 and is off by nearly 2×. We saw this exact pattern in our test data.

**Static averages:** F+ uses the average uplift per flight from the last three calendar months. It cannot distinguish between a July summer peak and a November winter pattern. It cannot react to airlines restructuring their network mid-year.

**No learning:** F+ cannot improve. Every month it makes the same class of errors. FLT gets better with every new month of data.

The November and December numbers are the headline: F+ was off by 24% in both months. Winter schedule changes — airlines cutting routes, reducing frequencies — blindsided F+ completely. FLT stayed at 7.6% and 6.1%.

---

## Slide 3 — Our Hypothesis

The key insight: instead of trusting the published schedule (which lies), we ask — what will actually fly?

Departures and flight minutes are the two signals that carry almost all the information about how much fuel will be uplifted. More flights = more fuel. Longer flights = more fuel. These signals are stable, learnable, and far more predictive than the static schedule.

The challenge is that at forecast time (6 months out) we only have the schedule, not the actuals. So we build a first model to predict what will actually operate, then feed that into the uplift model.

**On the cargo/charter stat at the bottom:** 3S (Sun Express) and YF are cargo and charter operators who don't publish long-range schedules. F+ cannot predict them at all — it shows zero and is 100% wrong. Even when we remove these routes entirely from the comparison (levelling the playing field), FLT still wins: 3.1% vs F+ 3.9%. This is important because it means our advantage is not just from routes F+ cannot see — we genuinely outperform on the routes F+ can predict too.

---

## Slide 4 — The FLT Pipeline

Two stages, one pipeline:

**Phase 1 — Schedule Correction:** Takes the published schedule as input. A LightGBM model has learned, from 17 months of history, how scheduled departures and flight minutes typically drift from what actually operates — broken down by airline, airport, days to operations, and season. It outputs corrected departure counts and flight minute estimates.

**Phase 2 — Uplift Prediction:** Takes Phase 1's corrected numbers as input. Instead of predicting raw kg directly (hard — a hub route carries 10× more than a thin leisure route), the model predicts a ratio: how much should the route-average formula estimate be adjusted? The ratio clusters near 1.0 for all routes regardless of size, which makes it much easier to learn.

The two stages compound: better schedule correction = better uplift inputs = better uplift prediction.

---

## Slide 5 — FLT vs F+ — Month by Month

Walk through the table:

**July and August:** Both models are accurate in summer peak. The schedule is reliable, routes are operating as expected. FLT's edge is modest — 1.6% vs 2.4% and 1.9% vs 2.7%.

**September — the one loss (3.2% vs F+ 2.4%):** June schedule still shows full summer ops. By September, airlines have quietly cancelled seasonal and charter routes without filing it. FLT's Phase 1 model trusts the (wrong) schedule and overcounts departures. F+ survives this accidentally — its 3-month rate was built from July/August actuals which already reflect the summer wind-down in kg-per-flight terms, so even with the wrong flight count the rate compensates. October onwards that trick stops working (summer rate goes stale) and F+ collapses while FLT recovers. Fix: Winterflugplan gives us the correct September schedule.

**October:** The gap widens — F+ starts to break down as more winter cancellations kick in. FLT 3.7% vs F+ 6.4%.

**November and December:** F+ collapses — 24.2% and 24.1%. Eurowings contracted significantly, several charter routes stopped, long-haul frequency dropped. F+ kept forecasting based on stale summer patterns. FLT adapted: 7.6% and 6.1%.

**Bottom line on cargo/charter:** Even if you remove the routes F+ literally cannot predict (3S and YF — 5.8% of total volume), FLT still wins 3.1% vs 3.9%.

---

## Slide 6 — Closing the Gap

This table shows how FLT addresses each known limitation of F+. Walk through the rows:

**Schedule drift — Solved:** This is Phase 1. The delta model (predicted actual minus scheduled) is trained on all historical routes and learns the correction per airline, airport, and horizon.

**Horizon degradation — Solved:** Long-range forecasts (91–220 days) behave very differently from short-range (1–90 days). Mixing them in one model produces a poor compromise. We train separate models for each horizon band, so each specialises on the range it sees.

**No route history — Solved:** New routes or thin routes with few historical flights are handled by a fallback hierarchy: first try the specific route's historical rate; if insufficient data, fall back to the airport average; if still insufficient, use the global mean. This means FLT can predict any route from day one of operation.

**Station disruptions — Roadmap:**

The problem: if a fueling station cannot operate (infrastructure failure, strike, severe weather), F+ has no way to know and keeps forecasting normal volumes. This leads to a major over-order at a station that cannot fuel, and a corresponding under-order at nearby alternatives.

The solution: a binary per-station flag — `station_disrupted = True/False`. When set to True for a station, FLT zeros out the predicted uplift for that station and redistributes the volume to the configured alternate stations for each airline route. For example, if FRA cannot fuel, affected aircraft may need to tanker from VIE or MUC.

Implementation: a small lookup table (`{station, disrupted_until}`) fed by ops or NOC teams. This is a pre/post processing step — the model itself does not change and does not need retraining. Only meaningful for short-horizon forecasts (1–30 days), because disruptions beyond that cannot be predicted.

This is on the roadmap because it requires agreement with operations teams on the redistribution logic and an integration point for the disruption signal.

**Aircraft swaps — Partial:**

The problem: F+ uses City Pair + Aircraft Type. FLT uses seat_bin (a grouping: small / medium / large / wide-body) plus kg/min rates. Within a seat_bin, swaps are well handled — a 737-800 for a 737-700 barely changes fuel consumption per minute, and the rate absorbs the difference.

The gap is cross-bin swaps: if a scheduled A320 (medium) is operated by a 777 (wide-body) — perhaps due to late demand surge or ACMI wet-lease — the model won't catch this because it was trained on the A320 pattern and the schedule shows A320.

The fix: for short-range forecasts (1–14 days), actual aircraft assignments are often already known via ACARS or ops systems. Feeding confirmed aircraft type as a feature for near-term predictions would close this gap. For long-range, there is no reliable fix — aircraft assignment at 6 months is not yet decided.

This is marked Partial because within-bin swaps are handled well and represent the large majority of swaps. Cross-bin swaps are relatively rare but can cause large errors on high-volume routes.

---

## Slide 7 — End-to-End Results

This chart shows Phase 2 performance using actual departures and flight minutes — this is the ceiling: how well can we predict uplift when the schedule correction is perfect? The answer is strong across all months and airlines.

In production, Phase 1 feeds Phase 2. The end-to-end result will be slightly weaker than this ceiling — but the month-by-month slide shows the production-representative comparison, and FLT still wins 5 of 6 months.

Use the filters in the chart to drill into specific airlines or airports if needed.

---

## Slide 8 — Roadmap

Five phases, no fixed dates:

**Now — Proof of Concept:** We have validated the full pipeline on 10 airports, 6 months of live data, 11 airlines. The models are trained, tested, and the results are consistent.

**Phase 1 — Multi-Scenario Validation:** Winterflugplan integration to close the September gap. Extended training data (17 months → 3+ years). Additional test scenarios including edge cases (route exits, airline restructuring).

**Phase 2 — Full Airport Expansion:** Scale beyond the initial 10 airports. Real-time SSIM feed so the model reacts to schedule changes immediately. Station disruption flag implementation.

**Phase 3–4 — Production + Extended Horizon:** Live production deployment with 4–6 month forecast horizon. Then extend to a 15–18 month long-range model for capacity and budget planning cycles.

**Phase 5 — Full System Launch:** ~400 airports. Continuous automated retraining on new data. Fully integrated with the fuel planning workflow.

Today's 10 airports already represent 96,993 tonnes less misorder across the 6-month test period. Every airport added compounds that impact directly.
