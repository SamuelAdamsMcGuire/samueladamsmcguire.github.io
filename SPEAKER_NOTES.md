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
- 64 active airline–airport combinations across the 6-month window.
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

## Slide 4: The F+ Approach

**Key message:** Simple, static, and limited by design.

- Four steps: Published Schedule → Last 3 months average → Route average → Predicted uplift.
- That's it. No learning, no correction.
- "This is essentially a spreadsheet formula. It works when nothing changes, but aviation always changes."
- Point out: when the schedule changes after publication, F+'s prediction is anchored to the old numbers and can't self-correct.

---

## Slide 5: How FLT Works

**Key message:** The shift from route averages to forecasted capacity.

- F+ asks: "How much did this route historically burn?" Static, backward-looking.
- FLT asks: "How much will actually fly — and what will those aircraft burn?" Dynamic, forward-looking.
- Step 1a: The schedule says 30 flights — the model corrects it to what will actually operate, using historical deviation patterns.
- Step 1b: Flight minutes capture aircraft swaps. A wide-body replaced by a narrow-body burns less fuel and flies different minutes. FLT sees that; F+ doesn't.
- Step 2: We're not applying a historical route mean to scheduled flights. We're applying a kg/flight-minute rate (by airline + aircraft type + airport) to our *predicted* capacity. If the aircraft changes, the prediction adjusts. If frequency changes, the prediction adjusts. The correction ratio fine-tunes at long horizons.

---

## Slide 6: How the Model Learns — Phase 1

**Key message:** Two delta models that correct the schedule — not replace it.

- The key framing: the schedule is a baseline, not the prediction. We learn how wrong it will be.
- Departures model: learns the gap between scheduled and actual departure count. `days_to_ops` is critical — the model learns that 5 days out, the schedule is mostly right; 180 days out, it drifts significantly.
- Minutes model: learns the gap between scheduled and actual flight minutes. This captures aircraft swaps that don't change departure count but do change the fuel profile.
- `seat_bin` groups aircraft by size category. When a wide-body is swapped for a narrow-body, the seat_bin changes — it's a feature, not noise.
- Historical lag deltas: how has this route's schedule deviated in the past at 7, 14, 28, 90, 180, and 365 days out? This encodes the route's reliability history.

---

## Slide 7: How the Model Learns — Phase 2

**Key message:** Statistical rates applied to corrected capacity, with ML fine-tuning at long horizons.

- Phase 1 gives us corrected departures and flight minutes. Phase 2 converts those to kg.
- Fuel rates: recency-weighted averages of kg/departure and kg/flight-minute, grouped by airline + aircraft type + airport. More recent months count more — this handles gradual changes in aircraft fleet or fuel efficiency.
- We blend per-flight and per-minute signals. Per-minute adjusts for aircraft swaps (same flight, different aircraft = different minutes); per-flight adjusts for frequency changes.
- Fallback chain: no history for a specific airline + aircraft + airport combination? Fall back to same airport + aircraft size, then to the global airline mean. This is how FLT handles new and seasonal routes.
- Correction ratio (≈ 1.0): a learned nudge for when the statistical formula drifts at long horizons. Keeps the estimate calibrated without replacing the statistical backbone.
- Uplift model trained only on 2025 data — tankering regulations changed fuel ordering behaviour in 2025, making earlier data unreliable as a training signal.

---

## Slide 8: The FLT Pipeline

**Key message:** Here's the full picture — two phases, clear colour coding.

- Blue (input): Published schedule (SSIM). Same starting point as F+.
- Green (Phase 1): Two delta models — departure and flight minutes. Both correct the schedule, not replace it.
- Yellow (Phase 2): Statistical rates (recency-weighted kg/dep and kg/min), blended and with fallback for sparse routes. Correction ratio fine-tunes at long horizons.
- Teal (output): Predicted uplift per airline–airport per month.
- "The statistical formula does the heavy lifting. ML corrects the schedule and fine-tunes the edges."

---

## Slide 9: Where We Win Big — Unscheduled Flights

**Key message:** FLT can predict what F+ literally cannot see.

- XQ (SunExpress) to KEF, July 2025. Schedule: 0 flights. F+ prediction: 0.
- Actual: 2 flights, 15,854 kg uplift.
- FLT predicted: 2 flights, 18,915 kg — within 19% of actual. Not perfect, but far better than zero.
- "How? XQ flew to KEF in previous summers. That pattern was in our training data. F+ is blind to anything not on the current schedule. FLT uses historical intelligence."
- This matters most for charter airlines and seasonal routes where the schedule is always incomplete.

---

## Slide 10: Month-by-Month Head-to-Head

**Key message:** Walk through the chart — the trend is the story.

- July and August: FLT wins clearly. These are nearest months; schedule correction already adds value.
- September: F+'s one win. Small margin. F+'s summer average coincidentally aligns with September demand.
- October, November, December: FLT wins, gaps grow. Schedule changes accumulate — winter routes, cancellations, aircraft swaps — and F+ can't respond.
- "Notice how F+'s error tends to climb further out. FLT stays more stable. That's schedule intelligence at work."
- Use the dropdowns to drill by airline or airport. LH-FRA shows the effect at scale — the biggest route, the biggest gap.

---

## Slide 11: Built to Improve

**Key message:** 17 months of data is our starting point, not our ceiling.

- The model has seen one summer and one winter. Seasonal patterns are real but still thin.
- `days_to_ops`: each new month adds training examples at every forecast horizon — 1 day out through 180+ days. The model learns more precisely how accuracy changes with distance.
- `seat_bin`: more routes and airports means richer aircraft-type coverage — fewer fallbacks on new or seasonal routes where history is sparse.
- Tankering regulations changed how airlines order fuel in 2025, making pre-2025 uplift data less relevant as a training signal. Every additional month under the new rules directly improves the uplift model.
- More airports strengthen shared patterns across the network without diluting route-level accuracy.
- "These results are our floor, not our ceiling."

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

## Slide 14: The Ask

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
- Training: ~30–60 minutes on a compute cluster for all model phases.
- Inference: under a minute locally for all 10 airports and 6 months.
- No special infrastructure needed in PoC stage.

**"What if the schedule correction is wrong?"**
- The statistical formula (Phase 2) is anchored to historical route means, which are very stable. Even if Phase 1 over- or under-corrects, the route mean provides a strong baseline. The correction ratio further corrects systematic bias. No single component can cause catastrophic errors — the system is designed to be robust.

**"Why not just use ML for everything?"**
- Pure ML at long horizons tends to drift from historical patterns. By anchoring to statistical route means and using ML only to correct the edges, we get stability + adaptability. The statistical formula captures ~90% of the signal; ML handles the remaining variance at the extremes.

**"What's the schedule correction doing exactly?"**
- Published schedules (SSIM) are what airlines file months in advance. Reality always differs — cancellations, additions, aircraft swaps. Our Phase 1 models learn these patterns from 17 months of schedule-vs-actuals data. Example: if a route historically cancels 10% of flights in winter, the model predicts fewer departures than scheduled. Flight minutes capture aircraft swaps — a wider-body replaced by a narrow-body shows up as fewer minutes even if departure count is the same.

**"Why is the uplift model trained only on 2025 data?"**
- Tankering regulations changed fuel ordering behaviour significantly in 2025. Pre-2025 uplift data reflects different ordering patterns — using it would introduce systematic bias. We train only on data that reflects how airlines are actually ordering today. Every additional month of 2025+ data improves the model directly.
