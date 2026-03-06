# FLT Speaker Notes

---

## Our Mission

The whole project in one sentence. We built an alternative to F+, and we want to beat it cleanly.

Keep it short here — don't over-explain. Let the mission land, then move on.

---

## How F+ Works

F+ is simple: take the published schedule, look up the average uplift for each city pair and aircraft type over the last three months, and sum it up. That's it.

The simplicity is both its strength and its weakness. It works well when routes are stable and history is available — but it falls apart quickly when either of those isn't true.

---

## The Limits of F+

Walk through each point and let it build. Each one is a real failure mode you can explain in one sentence:

- **Schedule drift** — the published schedule and what actually flies diverge constantly, especially further out.
- **Horizon degradation** — the further out the forecast, the more schedule drift accumulates.
- **New/seasonal routes** — no history, no estimate. F+ is blind.
- **Station disruptions** — if a station can't fuel, aircraft tank elsewhere. F+ doesn't know.
- **Aircraft swaps** — a narrow-body replaced by a wide-body carries very different fuel. F+ treats them the same.

These aren't edge cases. They happen regularly across 400 airports.

Tell the audience these aren't just observations — FLT was designed specifically around solving them. The next slide shows how.

---

## Our Hypothesis

This is the conceptual pivot. We're not throwing out route history — we're reframing what we use.

Instead of "what did this city pair historically uplift?", we ask: "given how much flying is actually happening here, how much fuel does that require?"

Flight volume — departures and minutes — is a proxy for fuel demand that works even for new routes, swaps, and disruptions. Seat bin captures aircraft size without needing route-level history.

---

## What We Tried — Uplift Model

This is a credibility slide. We didn't jump straight to the answer — we tested widely.

Point out that we tried everything from a dummy baseline up to LightGBM, and experimented with how we frame the target (direct kg, ratio, quantile) and how we blend departures vs minutes.

Don't spend too long here — the takeaway is just: we were thorough.

---

## What We Use — Uplift Model

The winning approach is a formula blend with a learned correction ratio.

- The formula computes a base estimate from recency-weighted kg/dep and kg/min rates.
- A HistGBM model learns to predict how much the formula is off — expressed as a ratio near 1.0.
- The features for the correction model are everything the formula can't capture on its own: airline identity, airport, seat bin, route type, seasonality.

The correction model doesn't replace the formula. It improves it.

---

## How It Works — The Equation

The base estimate is a simple blend of two rate-based estimates:
- kg/dep rate × departures
- kg/min rate × flight minutes

These rates are recency-weighted from historical data.

The model then predicts a ratio: `actual / base`. In training this ratio clusters around 1.0 — the formula already does most of the work. The model just corrects the systematic over/underestimates.

At prediction time: `base × predicted ratio = final forecast`.

---

## Uplift Model Architecture

Walk through the diagram left to right.

Actual departures and actual flight minutes feed into the base estimate, together with recency-weighted route rates. That base estimate is then multiplied by the ML correction ratio.

The key design choice is predicting a ratio near 1.0 rather than raw kg. A busy hub might carry 10× more fuel than a thin leisure route — the ratio normalises that, making the model much easier to train and generalise.

---

## Training and Test Data — Uplift Model

Important: the test vector uses **actual departures and actual flight minutes** — not the published schedule. This is the "perfect schedule" test.

Why? Because we want to isolate the quality of the uplift model itself, separate from any schedule prediction error. If the inputs are perfect, how good is the model?

17 months of training data across 10 airports, 11 airlines, 64 airline–airport pairs.

---

## Uplift Model Results — Using Perfect Schedule

These are the headline numbers for the uplift model in isolation.

If someone asks: this is the ceiling — what FLT can achieve when it has perfect inputs. The schedule correction models (Phase 1) are what bring us closer to this ceiling in production.

Use the filters to show specific airlines or airports if there's interest.

---

## The Missing Piece — Schedule Reality

This is the bridge slide. The previous results are great — but they assumed we knew exactly what would fly.

In reality, we only have the published schedule, which diverges from actual operations. The closer our schedule predictions are to reality, the closer our uplift forecast gets to those ceiling numbers.

Phase 1 is the schedule correction layer — its job is to take the published SSIM and predict what will actually operate.

---

## What We Tried — Schedule Correction

Same credibility slide as for the uplift model. We tried the full range: linear models, SVR, KNN, tree ensembles, joint vs separate targets, delta vs direct, horizon splits.

The feature set here is richer because schedule prediction depends heavily on timing: lag features at 7, 14, 28, 90, 180, 365 days. The model needs to know how far ahead you are and what the historical pattern looks like.

---

## What We Use — Schedule Models

Four models in production: deps 1–90, deps 91–220, mins 1–90, mins 91–220.

LightGBM with a delta formulation — predict the difference between scheduled and actual, not the absolute value.

Splitting by horizon (near-term vs long-range) is important: near-term schedules are more reliable, long-range schedules drift more. Each model specialises.

---

## How It Works — Schedule Equation

The target is simply: `actual − scheduled`.

In training, for every historical day we know both the scheduled and actual values, so we can compute this delta directly.

At prediction time: `scheduled + predicted delta = predicted actual`.

The delta approach is better than predicting absolute values because the schedule is already a strong baseline — the model only needs to correct its known biases, not predict from scratch.

---

## Schedule Correction Model Architecture

Four models run in parallel, two per metric (deps, mins), split by horizon (1–90 and 91–220 days).

The outputs are merged: predicted actual departures and predicted actual flight minutes. These then feed into the uplift model as if they were the real schedule.

---

## Training and Test Data — Schedule Models

Training goes back to October 2023 — 20 months — because schedule prediction benefits from more seasonal history.

The test vector uses the schedule as it was published on June 12, 2025. Ground truth is actual departures and flight minutes for July–December 2025.

Metric is WAPE on predicted vs actual deps and mins — how close are our schedule corrections to reality?

---

## Schedule Model Results — Departures

Use the filters to explore by airline or airport. The mode toggle shows raw volumes (Volumes), absolute error, or WAPE %.

The key story: our predicted departures track actual departures significantly better than the raw published schedule. That improvement flows directly into uplift accuracy.

---

## Schedule Model Results — Flight Minutes

Same as departures — use filters as needed.

Flight minutes is a proxy for block time, which drives fuel burn per flight. Getting this right matters as much as departure counts.

---

## Putting It All Together

This is the integration slide — how the two phases connect.

Phase 1 takes the raw SSIM schedule and outputs corrected departure and minute estimates. Those estimates feed straight into the uplift model as inputs. The recency-weighted rates convert volume to a base estimate, and the ML correction ratio fine-tunes it.

The whole system is end-to-end: schedule in, fuel forecast out.

---

## The Complete FLT Pipeline

Walk through the diagram top to bottom.

The published SSIM schedule enters Phase 1, which outputs predicted actual departures and flight minutes. These flow into Phase 2: recency-weighted rates compute a base estimate, with route/airport/global fallbacks if no history exists. A correction ratio model adjusts the base. Final output is predicted uplift.

If someone asks about the fallback: if a route has no history (new route, charter, unscheduled), the system falls back to airport-level or global means — it never returns null.

---

## Validation Setup

Same forecast date for both FLT and F+: June 12, 2025. Neither model had access to anything after that.

64 active airline–airport pairs across 10 airports and 11 airlines. 1.61 million tonnes of actual uplift across the 6-month window. This is a real-world test, not a toy dataset.

---

## FLT Beats F+ in 5 out of 6 Months

The headline result. 5–1. Overall WAPE 3.1% vs 3.9% — a 21% reduction in error.

The one month FLT lost (September — the X) is worth acknowledging: unusual schedule behaviour that month skewed the model's corrections slightly high. Investigating.

13,303 tonnes less misorder over the 6-month window. That's the real number.

---

## Why the Formula Approach Works

The lesson from all the experimentation: pure ML approaches struggled. The formula carries most of the signal — it just needs correction at the edges.

The table shows this clearly. ML direct (no formula) got 6.7–7.5% WAPE. Formula alone got 4.3–5.2%. Formula + ML correction got 3.1% — better than everything else and better than F+.

The correction model only needs to be right about the residuals. That's a much easier learning problem.

---

## Where We Win Big — Unscheduled Flights

This is a concrete example of something F+ literally cannot do.

XQ-KEF, July 2025: zero scheduled flights. F+ sees zero, predicts nothing. FLT predicted 2 flights and 18,909 kg — close to the actual 15,854 kg.

How? The historical patterns for that route were seeded into the test vector. Even with zero scheduled, the model picked up that flights typically operate on this route and gave an estimate.

This is a structural advantage. F+ will always miss unscheduled operations.

---

## FLT End-to-End Results

Full results table: all airlines, airports, months, actual vs FLT predicted vs F+ predicted.

Use the filters to drill into specific combinations. Winner column shows who won each route.

Good to use during Q&A when someone asks about a specific airline or airport.

---

## How FLT Closes the Gap

Walk the table row by row — it maps directly to the five limits on the previous slide.

- **Schedule drift** and **Horizon degradation** → both solved by Phase 1. Two different problems, one model family.
- **No route history** → the fallback hierarchy is the key insight. We never need city-pair data. Route → airport → global.
- **Station disruptions** → honest answer: roadmap. We can detect anomalies via uplift ratio monitoring, but proactive redistribution isn't built yet. Good to acknowledge — shows we know the gaps.
- **Aircraft swaps** → partial. If the swap is within the same seat bin (e.g. A319 → A320), FLT handles it seamlessly. Cross-bin late swaps (narrow → wide body after schedule publication) are still a gap — but the kg/min rate softens the error compared to F+.

The overall message: 3 fully solved, 1 on the roadmap, 1 partially addressed. That's a strong story.

---

## Built to Improve

We have 17 months of training data — one full summer and one winter season. That's enough to validate, but not a lot of history.

Every new month of data improves accuracy at every forecast horizon. More airports means richer shared patterns — the model sees more variation in airline and aircraft types.

These results are the floor. The system gets better automatically as it runs in production.

---

## Roadmap

Start at Now: 10 airports validated, 5–1 vs F+.

Phase 1: multi-scenario validation — more airlines, more conditions.
Phase 2: expand to more airports.
Phase 3: production deployment, 4–6 month horizon.
Phase 4: extend to 15–18 month long-term model.
Phase 5: ~400 airports, full system.

The architecture doesn't change — it scales.

---

## The Opportunity

13,303 tonnes of misorder reduction from 10 airports in 6 months.

The model is airline-agnostic. Adding a new airline means adding their data — same model, more signal. Scaling to 400 airports with the same architecture is the path.

The economics scale linearly with coverage.

---

## The Ask

Summarise: 5 out of 6 months, 21% less misorder, 10 airports, 11 airlines, 64 combinations. The validation is done.

The ask is simple: continue the project and expand coverage. Every airport added improves the model and increases the savings.

If there are questions about timeline, cost, or integration — defer to the team for specifics.
