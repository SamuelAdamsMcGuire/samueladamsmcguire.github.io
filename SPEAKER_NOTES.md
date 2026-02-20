# Speaker Notes — FLT Presentation

---

## Slide 1: The Problem

**Key message:** F+ is simple and outdated.

- F+ takes the last 3 months of actual fuel uplift for each route and averages them. That's the prediction.
- Problem 1: If flights get cancelled or added between now and the forecast month, F+ doesn't know. It still uses the old numbers.
- Problem 2: If a route didn't exist 3 months ago — like a new seasonal route — F+ has nothing to average. It's blind.
- Problem 3: The further out you forecast, the more the schedule changes. F+ can't adapt to that. Its accuracy degrades over time.
- "Can we do better?" — rhetorical, the answer is yes, and the next slide proves it.

---

## Slide 2: FLT Beats F+ in 5 out of 6 Months

**Key message:** The headline result. Hook the audience immediately.

- 5 wins out of 6 months. The one loss is September, and we know why (explained later if asked).
- Overall error: FLT 3.3% vs F+ 3.9%. That's volume-weighted — meaning it accounts for the fact that bigger airports matter more.
- 6-month misorder: FLT was off by 53,800 tonnes total, F+ was off by 63,500 tonnes. That's ~9,700 tonnes of better ordering.
- "Misorder" means the absolute difference between what was predicted and what actually happened — whether it's over-ordering or under-ordering.

**If asked "why did you lose September?":**
- F+'s June/July/August average happens to be a near-perfect predictor for September — there's low seasonal variance between summer and early autumn.
- We lose by less than 1 percentage point. And we win the other 5 months by larger margins.

---

## Slide 3: Month-by-Month Head-to-Head

**Key message:** Walk through each month.

- July and August: FLT wins clearly. These are the nearest months (forecast made ~6 months in advance), and our schedule correction already adds value here.
- September: The one F+ win. Small margin. F+'s summer average is coincidentally accurate for Sep.
- October, November, December: FLT wins, and the gap grows. This is where schedule changes accumulate — cancellations, new winter routes, aircraft swaps — and F+ can't keep up.
- Point to the trend: "Notice how F+'s error climbs from left to right, but FLT stays relatively flat."

---

## Slide 4: The Further Out We Forecast, the Bigger Our Edge

**Key message:** This is the "why it matters" chart.

- Same data as the previous slide but visualized as trend lines.
- F+ error climbs steeply from October onward. FLT stays more stable.
- Why? Because the further out you forecast, the more the schedule will change before the actual month arrives. F+ doesn't know that. It's still using a 3-month average that assumes the world stays the same.
- FLT corrects the schedule first, so even at 6 months out, we're predicting based on what we think will actually fly, not what was published.
- **This is the core value proposition of FLT: schedule intelligence.**

---

## Slide 5: Why F+ Struggles at Long Horizons

**Key message:** Explain the "why" in plain language.

- F+ depends on two things: the published schedule and the last 3 months of actual uplift. It multiplies them together.
- When the schedule changes — and it always does — F+ doesn't adjust. It still applies the old average to the old schedule.
- FLT takes a different approach: we first predict what will actually happen (how many flights, how many minutes), THEN we estimate fuel based on that corrected picture.
- That means seasonal shifts (summer→winter transition), cancellations, and brand new routes are all captured in our prediction.

---

## Slide 6: The F+ Pipeline

**Key message:** Show how simple (and limited) F+ is.

- Four boxes, top to bottom: Schedule → Historical uplift (last 3 months) → Simple average per route → Prediction.
- That's it. No intelligence, no correction, no learning.
- "This is basically a spreadsheet formula. It works when nothing changes, but the aviation world always changes."

---

## Slide 7: The FLT Pipeline

**Key message:** Show our approach is more sophisticated but still explainable.

Walk through the flowchart:

**Input (blue):** We start with the published schedule — the SSIM file that airlines publish months in advance.

**Phase 1 — Schedule Correction (green):** Two ML models (LightGBM) that predict what will actually happen:
- **1a: Predict actual departures.** The schedule says 30 flights this month on this route, but historically 10% get cancelled. Our model learns that pattern and predicts 27.
- **1b: Predict actual flight minutes.** Airlines often swap aircraft — a route scheduled with a wide-body might actually fly narrow-body. Flight minutes capture this because different aircraft fly different durations. This also catches frequency changes.

**Phase 2 — Statistical Uplift (yellow):** This is NOT machine learning — it's a weighted average formula.
- **Recency-weighted route means:** For each route (airline + airport + aircraft size), we compute the historical average fuel uplift per departure and per flight minute. Recent months count more — we use exponential decay weighting, so last month's data matters more than data from a year ago.
- **Blend formula:** We calculate two estimates: (route_mean_per_flight × predicted_departures) and (route_mean_per_minute × predicted_minutes). Then we average both. Using both per-flight and per-minute gives stability — if aircraft get swapped, per-minute adjusts; if frequency changes, per-flight adjusts.
- **3-level fallback (dashed line):** If a route has no history at all — a brand new route — we fall back: first try other airlines at the same airport with similar aircraft, then a global average by aircraft size. This is why FLT can predict unscheduled flights that F+ completely misses.

**Phase 3 — ML Correction (purple):** A gradient boosting model (HistGradientBoosting) that learns when the statistical formula is wrong.
- It predicts a correction ratio near 1.0. If the formula tends to over-predict charter routes in winter, the ratio pulls it down. If it under-predicts hubs in December, the ratio pushes it up.
- Final prediction = Phase 2 output × correction ratio.
- This mainly helps at far horizons (months 11-12) where the formula alone starts to drift.

**Output (teal):** The final fuel order prediction per route per month.

**Key talking point:** "The statistical formula does the heavy lifting. The ML models correct the schedule and fine-tune the result. It's not a black box — each step is explainable."

---

## Slide 8: How FLT Works

**Key message:** Simplified text version of the pipeline for reference.

- This is a summary table for people who want to see the steps listed out.
- Step 1a: Departures prediction — because the schedule is never perfectly right.
- Step 1b: Flight minutes prediction — captures aircraft swaps and frequency changes that raw departure counts miss.
- Step 2: Statistical uplift formula — recency-weighted route means multiplied by predicted volume. This is the backbone of the prediction.
- Step 3: ML correction — fine-tunes at far horizons where the formula alone isn't enough.
- Key insight to emphasize: "We don't trust the schedule. We predict what will actually fly." This is the sentence they should remember.

---

## Slide 9: Fuel Ordering Impact

**Key message:** Translate percentages into tonnes — tangible impact.

- Two bars per month: green (FLT misorder) vs red (F+ misorder). Smaller bar = better prediction.
- In 5 of 6 months, FLT's bar is smaller. The one exception is September.
- Total: FLT misorders 53,800 tonnes, F+ misorders 63,500 tonnes. That's ~9,700 tonnes of better ordering.
- "9,700 tonnes is fuel that was either over-ordered — meaning capital tied up in inventory — or under-ordered — meaning emergency spot purchases at a premium."
- At current Jet-A1 prices (~€700/tonne), 9,700 tonnes represents roughly €6.8M worth of fuel that was more accurately ordered. The actual financial benefit depends on the cost of over-ordering vs under-ordering, but the direction is clear.

---

## Slide 10: Where We Win Big — LH-FRA

**Key message:** High-volume route, big absolute impact.

- LH-FRA is the single highest-volume route in our dataset — over 824,000 tonnes of actual uplift across 6 months.
- On this one route alone, FLT saves ~12,700 tonnes of misorder vs F+.
- That's more than the entire net improvement across all other routes combined.
- "Even small percentage improvements on routes this large translate to huge tonnage differences. A 1% improvement on LH-FRA is worth more than a 10% improvement on a small regional route."

---

## Slide 11: Where We Win Big — Unscheduled Flights

**Key message:** FLT can predict what F+ literally cannot see.

- XQ (SunExpress) to KEF (Reykjavik), July 2025.
- The schedule showed 0 flights. Zero. F+ has nothing to work with — it predicts nothing.
- In reality, 2 flights operated carrying 15,854 kg of fuel.
- FLT predicted 2 flights and 18,915 kg of uplift — within 19% of actual.
- How? Our model looks at historical patterns. XQ flew to KEF in previous summers. Even though this year's schedule didn't show it yet, the model caught the pattern.
- "This is the kind of thing F+ will never be able to do. If it's not on the schedule, F+ is blind. FLT uses historical intelligence."

---

## Slide 12: Current Scope

**Key message:** This is a proof of concept, and it already works.

- 10 airports: BLL (Billund), FRA (Frankfurt), VIE (Vienna), PMI (Palma), ORD (Chicago), HRG (Hurghada), KEF (Reykjavik), HAM (Hamburg), HKG (Hong Kong), WAW (Warsaw).
- Mix of hubs (FRA, VIE), leisure (PMI, HRG), long-haul (ORD, HKG), and regional (BLL, HAM, WAW, KEF).
- 11 airlines across the Lufthansa Group: LH, OS, LX, SN, EW, EN, WK, 4Y, YF, XQ, 3S.
- 6-month forecast horizon validated: July through December 2025.
- ~64 active routes across these airports.
- 1.61 billion kg (1,610,000 tonnes) of fuel over the validation period — this is not a toy dataset.
- "This is our proof of concept. We picked a diverse set of airports to stress-test the model. It's ready to scale."

---

## Slide 13: Roadmap

**Key message:** Clear path from proof of concept to production.

- Step 1 — Multi-scenario validation: Test the model on different time periods (not just Jul-Dec 2025) to confirm robustness.
- Step 2 — Full airport expansion + ML pipeline: Expand to all Lufthansa Group airports. Build automated training pipeline so models retrain as new data arrives.
- Step 3 — Short-term production deployment: Run FLT in parallel with F+ for 4-6 month predictions. Let the fuel teams compare in real time.
- Step 4 — Long-term model development: Extend the forecast horizon to 15-18 months. This requires more training data and potentially new model architecture.
- Step 5 — Full system launch: All airports, all horizons, user interface for fuel procurement teams.

---

## Slide 14: Built to Improve

**Key message:** The model will only get better — this is our starting point, not our peak.

- We trained on just 17 months of schedule data (January 2024 to June 2025). That's all we had available.
- The model has only seen one summer and one winter. It's learned seasonal patterns from a single cycle.
- With each additional month of data, seasonal patterns become more robust. After two full years, the model will have seen two summers and two winters — it will know that September behaves differently from August.
- More airports don't hurt accuracy — they help. Each route keeps its own historical mean, so adding new airports doesn't dilute existing predictions. But the shared components (fallback estimates, ML correction patterns) get stronger with more diverse training data.
- "These results are our floor, not our ceiling." — This is the key line. We're already beating F+ in 5 of 6 months with limited data. With more data and more airports, the model will only improve.

**If asked about September specifically:** "September is our one loss, and it's because we've only seen one September in training. With a second year of data, the model will learn that September has its own seasonal pattern distinct from summer. We expect that loss to narrow or flip."

---

## Slide 15: The Ask

**Key message:** Clear, confident, forward-looking.

- We've proven it works: 5 out of 6 months, 15% less fuel misorder.
- We've tested across a diverse set: 10 airports, 11 airlines, 64 routes, 6 months.
- The model is airline-agnostic: adding a new airline doesn't require rebuilding anything. It's just more training data that makes everything better.
- Continue the project and expand coverage: more airports, more time periods, toward production.

**Tone:** Confident but not overselling. We have strong results and a clear path. The ask is to keep going.

---

## Q&A Preparation

**"Why did you lose September?"**
- F+'s June/July/August average is coincidentally a good predictor for September — low seasonal shift between late summer and early autumn. Our model had only one September to learn from. With a second year of data, we expect this to improve.

**"How much money does this save?"**
- At ~€700/tonne Jet-A1, 9,700 tonnes of better ordering represents ~€6.8M in fuel value. The actual savings depend on over-ordering costs (capital, storage) vs under-ordering costs (spot purchase premiums). Conservative estimate: €300K-700K over 6 months at 10 airports. Scales with airport count.

**"Can this work for non-Lufthansa airlines?"**
- The model is airline-agnostic. It learns per-route patterns. Adding airlines means adding training data, not rebuilding. If we had schedule data from other airline groups, the model could predict for them too.

**"What about real-time updates?"**
- Currently: one-shot prediction from a schedule snapshot. Roadmap: rolling updates as schedule revisions come in, so predictions improve as the forecast month approaches.

**"How long does it take to run?"**
- Training: cluster job, runs in about 30-60 minutes for all three models.
- Inference: local script, runs in under a minute for all 10 airports and 6 months.
- No special infrastructure needed — runs on standard compute.

**"What if the schedule correction is wrong?"**
- The statistical formula (Phase 2) is anchored to historical route means, which are very stable. Even if the schedule correction over- or under-corrects, the route mean provides a strong baseline. The ML correction (Phase 3) then fine-tunes. The system is designed to be robust — no single component can cause catastrophic errors.

**"Why not just use ML for everything?"**
- Pure ML models tend to drift from historical patterns, especially at long horizons. By anchoring to statistical route means and only using ML to correct the edges, we get stability + adaptability. The statistical formula captures 90%+ of the signal; ML handles the remaining variance.

**"What's the schedule correction doing exactly?"**
- The published schedule (SSIM) is what airlines plan months in advance. But reality differs — flights get cancelled, added, or swapped to different aircraft. Our Phase 1 models learn these patterns from 17 months of schedule-vs-actuals data. For example, if a route historically has 10% cancellation rate in winter, the model learns to predict fewer departures than scheduled. Similarly, if airlines frequently swap aircraft on a route, the flight minutes prediction captures that.
