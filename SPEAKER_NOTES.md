# Speaker Notes — FLT Presentation

---

## Slide 1: The Challenge

**Key message:** Schedule-based forecasting has fundamental blind spots — FLT addresses them.

- F+ takes the last 3 months of actual fuel uplift per route and averages them. That's the prediction.
- Problem 1: Schedules change. Cancellations, additions, and aircraft swaps happen after publication. F+ doesn't see any of that.
- Problem 2: New and seasonal routes have no history. F+ has nothing to average — it's blind.
- Problem 3: The further out you forecast, the more the schedule will diverge from reality. F+ accuracy degrades over time.
- FLT addresses this by predicting what will actually fly — correcting for the schedule gap before estimating fuel.

---

## Slide 2: Validation Setup

**Key message:** Diverse, real-world data. Not a toy test.

- 10 departure airports: BLL, FRA, VIE, PMI, ORD, HRG, KEF, HAM, HKG, WAW — a deliberate mix of hubs, leisure, long-haul, and regional.
- 11 airlines: the full Lufthansa Group portfolio.
- ~64 active airline–airport combinations across the 6-month window.
- Forecast date: June 12, 2025. As if today is June 12th and we're predicting July through December 2025.
- Total validated uplift: 1.61M tonnes. This is a meaningful sample.
- "We designed this to stress-test the model across different airport types, not just pick easy routes."

---

## Slide 3: FLT Beats F+ in 5 out of 6 Months

**Key message:** The headline result — lead with it.

- 5-1. FLT wins five months, F+ wins one (September — explained in Q&A if asked).
- Overall WAPE: FLT 3.3% vs F+ 3.9%. Volume-weighted, so larger airports count more.
- 6-month total misorder: FLT 53,800 t vs F+ 63,500 t. That's ~9,700 tonnes of better ordering.
- "Misorder" = absolute gap between prediction and actual, regardless of direction. Both over-ordering and under-ordering are costly.

---

## Slide 4: Month-by-Month Head-to-Head

**Key message:** Walk through the chart — the trend is the story.

- July and August: FLT wins clearly. These are nearest months; schedule correction already adds value.
- September: F+'s one win. Small margin. F+'s summer average coincidentally aligns with September demand.
- October, November, December: FLT wins, gaps grow. Schedule changes accumulate — winter routes, cancellations, aircraft swaps — and F+ can't respond.
- "Notice how F+'s error tends to climb further out. FLT stays more stable. That's schedule intelligence at work."

---

## Slide 5: The F+ Approach

**Key message:** Simple, static, and limited by design.

- Four steps: Published Schedule → Last 3 months average → Route average → Predicted uplift.
- That's it. No learning, no correction.
- "This is essentially a spreadsheet formula. It works when nothing changes, but aviation always changes."
- Point out: when the schedule changes after publication, F+'s prediction is anchored to the old numbers and can't self-correct.

---

## Slide 6: The FLT Pipeline

**Key message:** More sophisticated but still explainable — walk through each phase.

**Input (blue):** Published schedule (SSIM). Same starting point as F+.

**Phase 1 — Schedule Correction (green):**
- Two ML models that predict what will actually fly vs what was scheduled.
- **1a Departures:** The schedule says 30 flights, but historically 10% cancel on this route. The model predicts 27.
- **1b Flight minutes:** Captures aircraft swaps. If a wide-body is replaced by a narrow-body, the flight duration and fuel profile changes. Flight minutes capture that.

**Phase 2 — Statistical Uplift (yellow):**
- NOT machine learning — a weighted formula. For each airline + aircraft type + airport, we compute recency-weighted averages: kg per flight and kg per flight-minute. More recent months count more.
- We calculate two estimates (per-flight and per-minute) and blend them. Per-minute adjusts for aircraft swaps; per-flight adjusts for frequency changes.
- Fallback (dashed): no history for a route? Fall back to same airport + aircraft size, then global average. This is how FLT handles brand-new routes.

**Phase 3 — ML Correction (purple):**
- Learns when the formula is systematically off. Predicts a correction ratio (near 1.0). Final prediction = Phase 2 × ratio.
- Mainly helps at far horizons (5-6 months out) where formula drift accumulates.

**Output (teal):** Predicted uplift per airline–airport per month.

"The statistical formula does 90% of the work. ML corrects the schedule and fine-tunes the edges."

---

## Slide 7: What We Train On

**Key message:** Real data, common window, granular grouping.

- All models trained on the same 17-month window: January 2024 – June 2025.
- Departure and flight minutes models: schedule vs actual data — the model learns which routes over- or under-deliver vs the published plan.
- Uplift rate model: actual uplift data by airline + aircraft type + departure airport. Not just route — aircraft type matters because different planes burn fuel differently.
- ML correction: predicted vs actual uplift ratios over the same 17 months.
- "This is why we group by airline, aircraft type, and airport — not just route. The same airline at the same airport with a different aircraft has a different fuel profile."

---

## Slide 8: How FLT Works

**Key message:** The shift from route averages to forecasted capacity.

- F+ asks: "How much did this route historically burn?" Static, backward-looking.
- FLT asks: "How much will actually fly — and what will those aircraft burn?" Dynamic, forward-looking.
- Step 2 is the key: we're not applying a historical route mean to the scheduled flights. We're applying a kg/flight-minute rate (by airline + aircraft type + airport) to our *predicted* flight minutes.
- If the aircraft changes, the prediction adjusts. If frequency changes, the prediction adjusts. F+ does neither.

---

## Slide 9: Fuel Ordering Impact

**Key message:** Tonnes matter more than percentages to fuel buyers.

- Two bars per month: FLT misorder vs F+ misorder. Smaller = better.
- FLT smaller in 5 of 6 months.
- Total: 53,800 t (FLT) vs 63,500 t (F+). Delta: ~9,700 t.
- "9,700 tonnes is fuel that was either over-ordered — capital locked in inventory — or under-ordered — forcing expensive spot purchases."
- At ~€700/tonne Jet-A1, this represents ~€6.8M in fuel value more accurately predicted. Actual financial benefit depends on the cost structure of over vs under-ordering.

---

## Slide 10: Where We Win Big — LH-FRA

**Key message:** High-volume routes amplify every percentage point.

- LH-FRA is the highest-volume route in our validation set.
- On this one route alone, FLT saves ~12,700 t of misorder vs F+ over 6 months.
- "A 1% improvement on a route this large is worth more than a 10% improvement on a small regional route. Scale is where our advantage compounds."

---

## Slide 11: Where We Win Big — Unscheduled Flights

**Key message:** FLT can predict what F+ literally cannot see.

- XQ (SunExpress) to KEF, July 2025. Schedule: 0 flights. F+ prediction: 0.
- Actual: 2 flights, 15,854 kg uplift.
- FLT predicted: 2 flights, 18,915 kg — within 19% of actual. Not perfect, but far better than zero.
- "How? XQ flew to KEF in previous summers. That pattern was in our training data. F+ is blind to anything not on the current schedule. FLT uses historical intelligence."
- This matters most for charter airlines and seasonal routes where the schedule is always incomplete.

---

## Slide 12: Roadmap

**Key message:** Clear path from proof of concept to production.

- **Now:** PoC — 10 airports validated, 5-1 vs F+.
- **Phase 1:** Multi-scenario validation — test on different time periods, confirm robustness across seasons.
- **Phase 2:** Full airport expansion — scale to all Lufthansa Group airports.
- **Phase 3:** Production — run FLT alongside F+ for real decisions, 4–6 month horizon.
- **Phase 4:** Long-term model — extend to 15–18 month horizon for procurement planning.
- **Phase 5:** Full system launch — ~400 airports, all horizons, fuel procurement tooling.

---

## Slide 13: The Opportunity

**Key message:** Today's results are just 10 airports — the opportunity scales.

- 10 airports → 9,700 t improvement in 6 months.
- Every airport added brings more routes, more training data, and stronger shared patterns.
- The model is airline-agnostic. Adding a new airline doesn't require rebuilding — it's more data that improves everything.
- ~400 airports on the roadmap. "More airports. Same model. More impact."
- Avoid giving a specific projected number for 400 airports — extrapolation is non-linear and the mix will differ. The message is: the model is built to scale.

---

## Slide 14: Built to Improve

**Key message:** 17 months of data is our starting point, not our ceiling.

- The model has seen one summer and one winter. Seasonal patterns are real but still thin.
- Each additional month makes the recency weights more accurate, the correction ratios more stable, and the fallback estimates more reliable.
- More airports don't dilute per-route accuracy — they strengthen shared components (fallback means, ML correction patterns).
- "These results are our floor, not our ceiling."

---

## Slide 15: The Ask

**Key message:** Confident, specific, forward-looking. Don't oversell.

- Results: 5 of 6 months, 15% less misorder.
- Coverage: 10 airports, 11 airlines, 64 airline–airport combinations, 6 months.
- Model is airline-agnostic — adding airlines = adding data, not rebuilding.
- Ask: Continue the project and expand coverage toward production.
- Tone: We have evidence. We have a path. We're asking to keep going.

---

## Q&A Preparation

**"Why did you lose September?"**
- F+ uses a rolling recent average (last 3 months). In September 2025, the Jun/Jul/Aug summer average happened to closely predict September's actual demand — seasonal stability between late summer and early autumn.
- FLT was still within 2.4% in September — well within operational accuracy. It's not that we were wrong, F+ was just lucky that month.
- Same pattern that helped F+ in September causes it to lag when the schedule shifts. October, November, December show exactly that.
- Also: we've only seen one September in training. With a second year of data, the model will have seen two September patterns and will model the transition better.

**"How much money does this save?"**
- At ~€700/tonne Jet-A1, 9,700 tonnes of better ordering represents ~€6.8M in fuel value more accurately predicted over 6 months at 10 airports.
- Actual savings depend on the cost structure: over-ordering costs (working capital, storage) vs under-ordering costs (spot purchase premiums). Conservative estimate: €300K–700K net financial benefit over 6 months. Scales with airport count.

**"Can this work for non-Lufthansa airlines?"**
- The model is airline-agnostic. It learns per-route patterns grouped by airline + aircraft type + airport. Adding a new airline is adding training data, not rebuilding the model.
- If we had schedule and actuals data from other airline groups, the model would work for them too.

**"What about real-time updates?"**
- Currently: one-shot prediction from a schedule snapshot taken on the forecast date.
- Roadmap: rolling updates as revised schedules come in, so predictions improve as the actual month approaches.

**"How long does it take to run?"**
- Training: ~30–60 minutes on a compute cluster for all three model phases.
- Inference: under a minute locally for all 10 airports and 6 months.
- No special infrastructure needed in PoC stage.

**"What if the schedule correction is wrong?"**
- The statistical formula (Phase 2) is anchored to historical route means, which are very stable. Even if Phase 1 over- or under-corrects, the route mean provides a strong baseline. Phase 3 further corrects systematic bias. No single component can cause catastrophic errors — the system is designed to be robust.

**"Why not just use ML for everything?"**
- Pure ML at long horizons tends to drift from historical patterns. By anchoring to statistical route means and using ML only to correct the edges, we get stability + adaptability. The statistical formula captures ~90% of the signal; ML handles the remaining variance at the extremes.

**"What's the schedule correction doing exactly?"**
- Published schedules (SSIM) are what airlines file months in advance. Reality always differs — cancellations, additions, aircraft swaps. Our Phase 1 models learn these patterns from 17 months of schedule-vs-actuals data. Example: if a route historically cancels 10% of flights in winter, the model predicts fewer departures than scheduled. Flight minutes capture aircraft swaps — a wider-body replaced by a narrow-body shows up as fewer minutes even if departure count is the same.
