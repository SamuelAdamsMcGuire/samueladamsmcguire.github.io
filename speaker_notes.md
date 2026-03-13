# Speaker Notes — FLT: Smarter Fuel Uplift Forecasting

Presentation date: March 17, 2026
Audience: Technical/business stakeholders familiar with airline fuel planning

---

## Our Mission

The one-sentence framing for everything that follows. F+ is what's in production today — it's the bar we need to beat. This slide sets up the entire conversation as a head-to-head comparison. Don't linger. Move quickly to what F+ actually does.

---

## How F+ Works

Stress the simplicity: published schedule → look up the last 3 months of history on that City Pair and Aircraft Type → average it → done. That's the whole system. It's not a dumb approach — it works reasonably well for stable, well-historied routes. The problem is what it can't see, which is the next slide.

---

## The Limits of F+

Walk through the five points one by one, letting each one land before moving on (incremental animation).

- **Schedule drift**: airlines cancel, swap, and add flights constantly. F+ trusts the published schedule without question.
- **Horizon degradation**: a forecast 6 months out is based on a schedule that's much less reliable than one made next week. F+ doesn't know that.
- **No route history**: new routes, seasonal one-offs — F+ can't price them at all. No city pair = no estimate.
- **Station disruptions**: if Frankfurt can't fuel, planes tank at the previous stop. F+ doesn't model this. Note the "(FLT roadmap: station disruption flag)" callout — if asked, explain that FLT's architecture supports a per-station binary flag; this is designed but not yet implemented.
- **Aircraft swaps**: a 180-seat narrowbody and a 250-seat widebody are very different fuel consumers. A swap breaks F+'s city-pair average.

These aren't edge cases — they happen constantly across a network of 400 airports.

---

## Our Hypothesis

The key insight: **stop thinking in city pairs, start thinking in flight volume**.

Departures and flight minutes are what actually drive fuel consumption. They're measurable and forecastable. Seat bin captures aircraft size without needing route-specific history. The model generalises: "given this many flights of this size from this airport" — that's all it needs.

This approach is route-agnostic by design. That's what makes it scalable.

---

## The FLT Pipeline — Two Stages

High-level architecture overview — two phases working in sequence. Use the Mermaid diagram to walk through the flow before going into detail.

Phase 1 (left, blue/green): takes the published schedule and corrects it — departure and flight minutes delta models predict what will actually operate, splitting by forecast horizon (1–90 days vs 91–220 days).

Phase 2 (right, amber/purple): takes the corrected schedule and converts it to predicted fuel uplift — statistical base (recency-weighted kg/dep and kg/min rates) multiplied by an ML correction ratio.

The clickable nodes are available if the audience wants to drill into any component. The key message: the two stages are independent but designed to connect — the output of Phase 1 is exactly what Phase 2 expects as input.

---

## What We Tried — Uplift Model

Context slide showing the breadth of the experimentation. Don't read every tag. The point: we tested everything from linear models to gradient boosting, tried multiple target formulations, dozens of feature combinations. The tags are evidence of thoroughness. Move on to what we kept.

---

## The Key Insight — Why a Ratio?

This slide explains the core design choice before showing the equation. Spend a moment here — it makes the equation slide much easier to follow.

**The problem**: raw fuel kg varies by an order of magnitude across routes. Frankfurt LH might uplift 500 t/day; a thin charter might do 15 t. A model trying to predict raw kg has to simultaneously understand both ends of that scale — it's a very hard learning problem.

**The solution**: normalise. Compute a formula-based estimate first, then predict how far off that estimate will be — expressed as a ratio around 1.0. Every route now looks similar to the model regardless of its size. The ratio for a thin charter and the ratio for LH-FRA both cluster near 1.0 — that's a consistent, learnable target.

If asked: the formula estimate uses recency-weighted kg/departure and kg/flight-minute rates. The ratio is actual ÷ formula. The model learns to predict ratio values slightly above or below 1.0 depending on route characteristics.

---

## What We Use — Uplift Model

Three things survived the experimentation: a formula blend, recency-weighted rates, and a HistGBM correction ratio. The correction model's feature set is worth noting — it includes computed interaction terms (mins × hub, mins × charter) and a route mean uplift per flight as a prior. This is what gives it the ability to generalise to thin or new routes.

---

## How It Works — The Equation

Two-step process.

**Base estimate**: blend two fuel rate signals — kg per departure and kg per flight minute — weighted by recency. This gives a formula-based estimate that's already a strong predictor.

**Correction ratio**: HistGBM predicts how far the formula is likely to be off, expressed as a ratio around 1.0. Multiplying base × ratio gives the final prediction.

Why a ratio? Because raw fuel kg varies enormously across routes — Frankfurt LH volumes are orders of magnitude above a thin charter. Normalising to a ratio near 1.0 gives the model a consistent learning target. That's what makes generalisation work.

---

## Uplift Model Architecture

Walk through the flow: actual departures and minutes, combined with recency-weighted rates, produce a base estimate. HistGBM then nudges it via a correction ratio. The base does the heavy lifting; ML handles the systematic gaps.

The key point: a direct ML model predicting raw kg on a new route has nothing to anchor to. The formula provides that anchor. That's why this architecture outperforms pure ML by a wide margin.

---

## Training and Test Data — Uplift Model

Two things to highlight:

1. **Actual inputs test**: the test vector uses actual departures and flight minutes — not the published schedule. This intentionally isolates the uplift model from schedule error. It answers: "how good can this get if we knew exactly what would fly?"

2. **17 months of training**: January 2024 – June 2025. One full summer and one winter season.

The results on the next slide are the *ceiling* of the approach — what the uplift model achieves when inputs are perfect.

---

## Uplift Model Results — Actual Inputs

Interactive chart — walk through it. Filters for airline, airport, month are live.

Key message: when we give the model perfect inputs, it's extremely accurate across all months and routes. This establishes that the uplift model is not the bottleneck. The challenge is that in production we have the published schedule, which drifts from reality. That's what Phase 1 addresses.

---

## The Missing Piece — Schedule Reality

The pivot point of the presentation. At prediction time — June 12, 2025 — we only have what was published in the schedule. That diverges from what actually flew, especially at long horizons. The further out the forecast, the less reliable the scheduled departures and flight minutes are as inputs.

The solution: build models to predict what will actually operate. That's Phase 1 — schedule correction.

---

## What We Tried — Schedule Correction

Same pattern as the uplift model slide — broad experimentation. Notable here is the horizon split (1–90 / 91–220 days) and the lag features (7, 14, 28, 90, 180, 365 days). Schedule behaviour is very different near-term vs. long-range, which is why we split. The lags capture seasonal patterns and route-specific biases.

---

## What We Use — Schedule Models

Four LightGBM models: departures delta and minutes delta, each split by horizon (short: 1–90d, long: 91–220d). Delta target because the schedule is already a strong baseline — predicting a correction near zero is a much easier learning problem than predicting absolute flight counts from scratch.

---

## How It Works — Schedule Equation

Same two-panel structure as the uplift equation slide.

Training: for every historical day we know what was scheduled and what actually flew. The model learns the delta.

Prediction: take the published schedule, add the predicted delta. The model corrects systematic over- and under-scheduling by route, airline, and horizon.

---

## Schedule Correction Model Architecture

Four models running in parallel. The key design choice: splitting by horizon (1–90 and 91–220 days). Near-term schedules are relatively stable; long-range schedules diverge substantially. Separate specialisation outperforms a single model handling both.

Each model produces predicted actual departures and predicted actual flight minutes, which then feed directly into the uplift model.

---

## Training and Test Data — Schedule Models

Training: 20 months of schedule vs. actuals pairs (October 2023 – June 2025). Longer training window than the uplift model because schedule drift is a longer-horizon signal that benefits from more seasonal coverage.

Test: the published schedule as of June 12, 2025 — horizon 1 through 220 days — compared against what actually flew July through December 2025. Models see only what was available at forecast time.

---

## Schedule Model Results — Departures

Interactive chart. The correction model substantially improves departure estimates across the board. The improvement is especially visible at longer horizons where schedule drift is largest. Walk through a few airline or airport examples if the audience wants depth.

---

## Schedule Model Results — Flight Minutes

Same story for flight minutes. The model handles it separately because aircraft swaps and route changes affect total block time independently from departure counts. Both corrections feed into the uplift model's base estimate.

---

## Putting It All Together

The connective tissue slide. Walk through it in sequence:

1. Phase 1 takes the published schedule and corrects it — producing predicted actual departures and flight minutes.
2. Those corrected values feed into the uplift model exactly as if they were the real thing.
3. Recency-weighted rates convert corrected flight volume into a base estimate.
4. The correction ratio fine-tunes for route-specific drift.
5. Result: a forecast that tracks what will actually fly, not just what was planned.

---

## The Complete FLT Pipeline

The full architecture in one diagram. Published schedule goes in at the top; predicted uplift comes out at the bottom. Phase 1 (green, schedule correction) feeds Phase 2 (amber, uplift formula + purple, ML correction).

**The fallback path** (dotted line) — important if asked about sparse routes:

The model uses a three-level fallback hierarchy for routes with little or no history:

- **Level 0 (exact match)**: airline + airport + seat_bin — uses a recency-weighted kg/departure rate. Normal case for any route with history.
- **Level 1 (airport + seat_bin)**: cross-airline average for that airport and aircraft size. Switches to kg/flight-minute rate to avoid a short-haul route inheriting a long-haul number.
- **Level 2 (seat_bin only)**: global average for that aircraft size class across all airports and airlines. Same kg/minute approach.

The key design point: fallback levels 1 and 2 use kg/minute not kg/departure, because minutes encode flight length. One departure means very different things for a 45-minute hop vs. a 10-hour wide-body flight. No route is unforecastable — something always fires.

---

## Validation Setup

Test conditions, clearly stated:

- 10 airports, 11 airlines, 67 airline–airport routes
- Forecast date: June 12, 2025 (single fixed point — not a rolling window)
- Test window: July – December 2025 (6 months)
- Same inputs and forecast date for both FLT and F+ — genuine apples-to-apples
- WAPE as the primary metric; Misorder (absolute tonnes of wrong ordering) as the operational measure

Emphasise: same forecast date, same inputs. F+ isn't penalised unfairly. It sees everything it would see in production.

---

## FLT Beats F+ in 5 out of 6 Months

The headline result. Five wins, one loss (September). Overall WAPE: 3.84% vs 9.51%. Total misorder over 6 months: 65,548 t vs 162,541 t — FLT has 96,993 t less misorder.

A win = our forecast is closer to actual fuel uplift for that month.

September is the one loss. Worth acknowledging rather than glossing over — transition months (summer → winter schedule) are harder to predict because schedule structures shift. The model has one such transition in training; more data will improve this.

---

## Why the Formula Approach Works

The table tells the story. Direct ML (LR, Boosting) comes in at 11–19% WAPE — worse than F+. Pure formula approaches sit at 4.4–7.5%. The formula with ML correction lands at 3.84%.

The ML correction needs the formula's anchor to work well. Neither component alone is enough. The combination is what wins.

---

## Where We Win Big — Unscheduled Flights

XQ-KEF, July 2025: zero scheduled flights, two actual flights, two predicted by FLT. F+ had nothing — empty schedule, no city-pair history, no estimate.

FLT caught it using historical patterns: XQ has operated at KEF before. The model seeds historically observed routes even when the current schedule shows nothing.

Uplift: FLT predicted 18.9 t against 15.9 t actual. Not perfect, but close. F+ predicts zero.

This is what route-agnostic means in practice.

---

## The Schedule Blind Spot

3S and YF are cargo and charter operators. They don't publish long-range schedules — F+ sees nothing for these airlines beyond the near term. For November and December, F+ made no predictions at all for these routes. FLT forecasted from historical patterns and was close.

The full-period result — 3.84% vs 9.51% — reflects this advantage. But even on the routes F+ can actually see, FLT wins: 3.1% vs 3.9%.

The blind spot isn't what makes FLT better. FLT is better everywhere. The blind spot is an additional structural capability.

3S and YF together represent 5.8% of the full 6-month test volume — meaningful, not a rounding error.

---

## FLT End-to-End Results

The full drilldown. Interactive — filters for airline, airport, month.

Key things to point out:
- FLT wins across most airline–airport combinations
- Largest absolute wins tend to be the highest-volume routes (biggest misorder reductions)
- A few outlier months/routes where F+ wins — these are systematic improvement opportunities as more data accumulates

---

## How FLT Closes the Gap

Map the five F+ limitations from the opening slide to outcomes:

- **Schedule drift** → Solved. Phase 1 corrects for it.
- **Horizon degradation** → Solved. Horizon-split models specialise by range.
- **No route history** → Solved. Three-level fallback: exact route → airport+aircraft size → global aircraft size. No route returns zero.
- **Station disruptions** → Roadmap. The architecture supports a per-station binary disruption flag. When a station cannot fuel, predicted volume would redistribute to alternate stations (configured per airline). Short-horizon only — meaningful for 1–30 day forecasts. Not yet implemented.
- **Aircraft swaps** → Partial. seat_bin + kg/min rates capture aircraft size; within-bin swaps handled well; cross-bin swaps are partially addressed.

Be honest about the partial and roadmap items — it shows maturity and gives a clear direction for future phases.

---

## Built to Improve

These results come from 17 months of data — one summer and one winter. The model improves structurally as more data accumulates:

- Every new month adds data at every forecast horizon — `days_to_ops` is a feature, so the model learns how accuracy degrades with distance. More months = better calibration across the 1–220 day range.
- `seat_bin` means aircraft-type patterns accumulate across routes and airports — more coverage reduces fallback errors on new or thin routes.
- Charter and cargo routes with sparse published schedules improve as historical patterns accumulate.
- New airlines onboard faster — seat-bin patterns transfer across carriers, so new entrants benefit from existing network knowledge immediately.
- More airports strengthen the shared patterns for hub, leisure, and charter behaviour.

The 3.84% is the floor, not the ceiling.

---

## Roadmap

Five phases from proof of concept to full production:

- **Now**: 10 airports validated, 5-1 result, proof of concept done.
- **Phase 1–2**: multi-scenario validation, expand airport coverage.
- **Phase 3**: production deployment, 4–6 month horizon.
- **Phase 4**: extend to 15–18 month horizon for longer-range planning.
- **Phase 5**: full network, ~400 airports.

Each phase is incremental — the core model doesn't change, only its scope.

---

## The Opportunity

10 airports in the test period = 96,993 t less misorder in 6 months. Scale that across a full network of ~400 airports and the impact is substantial.

The model is airline-agnostic — each new airline adds data that benefits all others through shared seat-bin and airport patterns.

The per-airport marginal cost of adding to the model is low. The incremental value scales with network size.

---

## The Ask

Summary: 5 out of 6 months, 60% less misorder, across 67 airline–airport route combinations.

The ask is to continue and expand. The proof of concept is done. The question is how far to take it.

Keep it short — the slides have done the work. End with the number and the invitation to continue the conversation.
