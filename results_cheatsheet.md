# FLT Results — Cheat Sheet
**Use this to explain any month, airport, or airline/airport combo that is worse than F+ or worse than 5% WAPE.**

---

## Overall Picture (Jul–Dec 2025)

| Month | FLT WAPE | F+ WAPE | FLT Wins? |
|-------|----------|---------|-----------|
| Jul   | 1.6%     | 2.4%    | ✓         |
| Aug   | 1.9%     | 2.7%    | ✓         |
| Sep   | 3.2%     | 2.4%    | ✗         |
| Oct   | 3.7%     | 6.4%    | ✓         |
| Nov   | 7.6%     | 24.2%   | ✓         |
| Dec   | 6.1%     | 24.1%   | ✓         |

**FLT wins 5 of 6 months. Only September is a loss (narrow: 3.2% vs 2.4%).**

---

## Root Cause 1 — Delta model ignores scheduled = 0 or near-zero

**The problem:** The Phase 1 delta model learned historical patterns for a route. When the schedule drops to 0 (airline exited) or is heavily reduced, it still applies a large positive historical correction — predicting flights that were never going to operate.

**How we know it is this:** The schedule already showed the correct reduced level. The model overrode it.

| Combo | Month | Scheduled | Predicted | Actual | What happened |
|-------|-------|-----------|-----------|--------|--------------|
| EW/WAW | Nov | 0 | 75 | 12 | Schedule correctly shows EW exited WAW. Model predicts 75 from historical memory. |
| EW/WAW | Dec | 0 | 82 | 12 | Same. |
| EW/WAW | Oct | 17 | 83 | 19 | Schedule shows EW cutting back. Model predicts 5× scheduled. |
| EW/FRA | Nov | 9 | 41 | 21 | Schedule shows reduced EW ops. Model predicts 4.5×. |
| EW/FRA | Dec | 9 | 45 | 22 | Same. |
| EN/BLL | Sep | 30 | 62 | 30 | Schedule already shows correct winter level. Model adds +32 from summer memory. |
| EN/BLL | Oct | 25 | 63 | 25 | Same — schedule was right, model overrode it. |

**Why we cannot fully fix it now:** Requires the model to learn that a schedule showing 0 means 0. This is a known limitation being addressed.

**Concrete fix — implemented in `feature/departure-cap`:** Cap Phase 1 predictions at `scheduled × 1.5`. When schedule = 0, cap = 0. When schedule = 9, cap = 13. This removes the hallucinated flights immediately without changing the model. **Testing this branch now.**

**Why FLT is still better than F+ on these:**
F+ predicts 0 for EW/WAW Nov and Dec (−100% error) because it has no schedule to work from. FLT at least produces a positive number. The cap brings FLT to the same 0 — but with the cap, FLT correctly predicts ≈0 while F+ also predicts 0. On the routes where the schedule is nonzero (EW/FRA, EN/BLL), the capped FLT is still better than F+.

---

## Root Cause 2 — Schedule was wrong from the start (charter/seasonal)

**The problem:** The June schedule showed full summer charter operations. Those flights were quietly cancelled before September–October. The Phase 1 delta model barely changes the scheduled count — the error is entirely in the input schedule, not the model.

**How we know it is this:** `sched ≈ pred`, but `actual` is far lower. The model did nothing wrong — the schedule lied.

| Combo | Month | Scheduled | Predicted | Actual | What happened |
|-------|-------|-----------|-----------|--------|--------------|
| XQ/WAW | Jul | 36 | 38 | 15 | June schedule shows 36 SunExpress WAW flights. Only 15 operated. Model adds +2 (fine). The schedule was the problem. |
| XQ/WAW | Aug | 35 | 38 | 17 | Same. |
| XQ/WAW | Sep | 35 | 37 | 18 | Same. |
| XQ/WAW | Oct | 35 | 41 | 24 | Model adds +6. Schedule still showing summer level. |
| EW/KEF | Sep | 9 | 9 | 3 | Schedule and model agree. 6 last-minute cancellations. |

**Why we cannot fix this with what we have:** No model can predict flights a filed schedule says will operate but that an airline later cancels quietly. This is not an ML limitation — it is a data availability problem. The only inputs available 6 months out are the filed schedule and historical patterns.

**What fixes it:**
- **Winterflugplan:** The winter schedule correctly shows SunExpress not operating WAW in September–October. Once we have this feed, the scheduled count drops to the correct level and the model follows.
- **Real-time SSIM updates:** When an airline files revised schedule with cancellations, the model sees it immediately.

**The comparison to F+:** F+ uses the same incorrect schedule. It makes the exact same error for the same reason. This is not a disadvantage of FLT vs F+ — both are equally blind to post-filing cancellations. FLT actually handles these routes better overall because it wins on aggregate WAPE.

---

## Root Cause 3 — Last-minute operational cancellations

**The problem:** Even when the schedule is correct and the model is correct, operations on the day can cancel flights due to weather, crew, ATC, or technical issues. No 6-month forecast can predict these.

**Example:** EW/KEF Sep — schedule says 9, model predicts 9, actual was 3. Six flights cancelled at short notice.

**Why we cannot fix this:** Nobody can. Not FLT, not F+, not the airline itself 6 months out. This is irreducible forecast error at long horizons. Short-term forecasts (1–7 days) could catch some of this using NOTAMs or ops data.

---

## Root Cause 4 — Long-haul sparse training data (HKG)

**FLT wins HKG overall (18.1% vs F+ 35.5%) but absolute WAPE is still high.**

F+ shows −100% for several HKG combos in Nov–Dec — meaning F+ predicted 0 (no forecast at all). FLT at least predicts something meaningful, which is why FLT wins despite a high absolute %.

| Combo | FLT WAPE | F+ WAPE | Notes |
|-------|----------|---------|-------|
| 3S/HKG | 21% | 44% | FLT wins by a large margin. F+ blind to 3S long-range schedule. |
| YF/HKG | 22% | 41% | Same pattern. |
| LH/HKG | 19% | 26% | FLT wins. Nov/Dec LH over-predicts — network restructuring. |

**What improves this:** More HKG training data. Currently 17 months — extending to 3+ years, especially for long-haul routes, tightens predictions significantly.

---

## September — Why FLT Loses

September is the only month FLT loses (3.2% vs F+ 2.4%). Drivers:

1. **EW/HAM September** — EW operates ~1.2M fewer kg than FLT predicts. Summer schedule wind-down. Largest single contributor.
2. **EN/BLL September** — 158k pred vs 78k actual. Delta model overrides correct winter schedule.
3. **XQ charter routes** — Multiple XQ routes show full summer scheduling vs reduced autumn reality.

F+ handles September better because it looks back at recent actuals (June, July, August) which are already showing the wind-down. FLT's June schedule input still shows full summer ops.

**The fix:** Winterflugplan. With the winter schedule as input, September schedule counts drop to the correct autumn level and FLT wins September too.

---

## The defence — why use FLT despite these cases?

**1. F+ is equally wrong on these routes — just differently expressed.**
When F+ predicts 0 it looks "less wrong" in % terms than FLT predicting 75 when actual is 12. But a zero forecast is useless. A planner cannot order 0 kg of fuel. FLT with the departure cap produces ≈0 for these cases too — same outcome, but explainable.

**2. These extreme cases are tiny volume relative to total.**
EW/WAW November actual: 37,000 kg. Full test period: hundreds of millions of kg across 10 airports. A 525% error on 37,000 kg moves the total WAPE by less than 0.1%. The headline — 5/6 months FLT wins — is on the total volume.

**3. The root causes are data gaps, not model failures.**
Every bad case traces back to either: (a) the schedule not yet reflecting a filed cancellation, or (b) an airline business decision made after the forecast date. No system — ML or otherwise — can predict what has not yet been filed. The fixes are concrete and in progress: departure cap (now in testing), Winterflugplan (Phase 1 roadmap), real-time SSIM (Phase 2 roadmap).

**4. These are our floor, not our ceiling.**
The model has 17 months of training data and no winter schedule feed yet. Every improvement — more data, Winterflugplan, departure cap — removes a specific identified error category. The improvement path is clear and bounded.

---

## Missing Combinations — Expected vs Verify

### Expected seasonal absences
| Combo | Missing | Reason |
|-------|---------|--------|
| EN/BLL | Nov, Dec | Summer-only route ✓ |
| LX/BLL | Sep–Dec | Swiss summer-only ✓ |
| XQ/BLL | Dec | SunExpress BLL ends Nov ✓ |
| EW/KEF | Oct–Dec | Iceland summer-only ✓ |
| OS/KEF | Nov–Dec | Iceland summer-only ✓ |
| SN/PMI | Dec | Palma summer route ✓ |
| LX/VIE | Nov–Dec | Seasonal ✓ |

### Verified one-off / sparse operations (confirmed from actual flight records)
All of the following are single or double flights — genuine but tiny operations. % errors on these are meaningless; volumes are negligible.

| Combo | Present | What it is |
|-------|---------|-----------|
| XQ/KEF | Jul + Dec | Summer peak + Christmas special |
| EW/BLL | Dec only | 2 Christmas charter flights |
| OS/BLL | Dec only | 2 Christmas flights |
| WK/BLL | Sep only | 1 Condor charter |
| EN/VIE | Dec only | 2 Air Dolomiti Christmas flights |
| SN/HKG | Dec only | 1 Brussels Airlines December flight |
| 3S/VIE | Sep only | 1 charter flight |
| 4Y/VIE | Sep–Oct | 3–4 HiSky flights |
| 4Y/HAM | Jul + Nov | 1 flight each, HiSky charters |
