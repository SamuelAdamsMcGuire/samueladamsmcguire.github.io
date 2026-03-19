# FLT Results — Cheat Sheet

**Ctrl+F the airline/airport combo (e.g. "EW/WAW") to find the explanation.**
WAPE = Σ|pred−actual| / Σactual. Lower is better.

---

## Overall Monthly Results

| Month | FLT  | F+    | FLT Wins? |
|-------|------|-------|-----------|
| Jul   | 1.6% | 2.4%  | ✓         |
| Aug   | 1.9% | 2.7%  | ✓         |
| Sep   | 3.2% | 2.4%  | ✗         |
| Oct   | 3.7% | 6.4%  | ✓         |
| Nov   | 7.6% | 24.2% | ✓         |
| Dec   | 6.1% | 24.1% | ✓         |

**FLT wins 5 of 6 months. September loss is narrow (3.2% vs 2.4%).**
Even excluding 3S & YF (routes F+ cannot predict — 5.8% of volume): FLT 3.1% vs F+ 3.9%.

---

## September — Why FLT Loses This Month

FLT loses September by 0.8pp (3.2% vs 2.4%). Both models forecast from June 12. Three drivers:

1. **EN/BLL** — Largest contributor. Model learned EN/BLL always operates 1.5–2.5× the schedule (true in 2024 training). Applied the same correction in Sep 2025 when actual exactly matched the schedule. F+ wins here because it uses the scheduled count directly without modifying it — and the schedule was correct.
2. **EW/HAM** — Eurowings in-period contraction. Model trained on ratios ≈1.0 (correct for all of 2024). EW reduced Hamburg ops mid-summer 2025, after the forecast date. FLT trusted the schedule; F+ applied a slightly lower Mar/Apr/May spring rate per flight, which compensated marginally.
3. **XQ routes** — Charters operated at 40–50% of scheduled levels. Model had the right seasonal signal from training but under-applied it at 220-day horizon. F+ doesn't correct flight counts either — it predicts 0 or inflates (65.6% WAPE vs FLT 50.7%).

**Why F+ wins September overall:** F+ applies its Mar/Apr/May 2025 rate to the scheduled count without modification. When the schedule is accurate (EN/BLL) this works perfectly. When the schedule is wrong (XQ), F+ is even worse than FLT. The narrow September win is almost entirely due to EN/BLL, where not modifying a correct schedule was the right call.

**From October onwards:** F+'s spring rate goes completely stale for winter operations. Airlines contract, routes close, charter frequencies drop. F+'s May rate has no information about November. FLT recovers strongly — 6.4% vs 3.7% in October, then 24% vs 7.6% in November.

**Fix:** EN/BLL resolves with more training data covering periods when it followed the schedule accurately. EW/HAM resolves once post-2025 EW contraction data enters the training window. XQ improves as the seasonal signal strengthens with more training cycles.

---

## EW/WAW — Eurowings Warsaw

**FLT WAPE: 200% overall. F+ WAPE: 34.7%**

| Month | Sched | Pred | Actual | Note |
|-------|-------|------|--------|------|
| Jul   | 14    | 15   | 14     | Fine |
| Aug   | 10    | 12   | 10     | Fine |
| Sep   | 21    | 44   | 21     | Model adds +23 on a correct schedule |
| Oct   | 17    | 83   | 19     | Model predicts 5× the schedule |
| Nov   | 0     | 75   | 12     | Schedule shows EW exited. Model predicts 75 from historical memory. |
| Dec   | 0     | 82   | 12     | Same. |

**Root cause — what the training data shows:** EW/WAW only appears in the training data for **2 months** (May and June 2025). In both months, actual departures were *above* scheduled (ratio 1.28 and 1.15). The model learned from just 2 data points: "EW/WAW always flies more than scheduled." When it sees sched=0 for Nov/Dec, it applies that same positive pattern and predicts 75+. The model has never seen EW/WAW wind down — because that only happened in the test period.

**Why F+ looks better here:** F+ predicted 0 for Nov/Dec (−100% error) because it had no scheduled flights to multiply. Zero happens to be closer to 12 than 75 — but a zero forecast is useless to a planner.

**Fix:** More training data. Once post-2025 data is included in training — showing EW/WAW with sched=0 and actual≈0–12 — the model learns the route is winding down and stops applying the positive correction. This resolves itself automatically as the training window extends.

---

## EW/FRA — Eurowings Frankfurt

**FLT WAPE: 73%. F+ WAPE: 13.3%**

| Month | Sched | Pred | Actual | Note |
|-------|-------|------|--------|------|
| Jul–Sep | ~12 | ~13 | ~12 | Fine |
| Oct   | 9    | 19   | 10     | Model doubles the schedule |
| Nov   | 9    | 41   | 21     | Model predicts 4.5× schedule |
| Dec   | 9    | 45   | 22     | Model predicts 5× schedule |

**Root cause — what the training data shows:** Same underlying issue as EW/WAW. EW/FRA has limited training history and the model learned a positive delta pattern (EW historically flies close to or above schedule at FRA). When EW reduced to 9 flights/month in winter 2025, the model kept predicting at historical summer levels. The model has never seen EW/FRA at reduced winter operations — that data only exists in the test period.

**Fix:** More training data. Once winter 2025 EW/FRA operations enter the training window, the model sees the reduced pattern and corrects itself.

---

## EW/HAM — Eurowings Hamburg

**FLT WAPE: 5.2%. F+ WAPE: 4.9% — F+ wins narrowly**

FLT and F+ are essentially tied. The main issue is September: FLT predicts ~7.2M kg, actual was ~6.0M kg.

**Root cause — what the training data shows:** EW/HAM training ratios (actual/scheduled) hold close to 1.0 from August 2024 onwards — the model correctly learned that EW/HAM is a reliable, well-scheduled route. The September 2025 shortfall was not a schedule filing failure; it was an in-period contraction: EW reduced Hamburg operations mid-summer 2025, after the June 12 forecast date. The model had no signal of this — the contraction only shows up in post-June actuals that were not yet available at forecast time.

**Why F+ wins narrowly:** F+ applies its Mar/Apr/May 2025 spring rate to the scheduled count. Spring EW/HAM rates are slightly lower per flight than summer peaks (lighter loads, shorter legs in spring mix). This gives F+ a marginal edge in September on volume — essentially by accident. Both models have the same input: the (slightly overstated) June schedule. The 0.3pp difference is noise at this scale.

**Fix:** More training data. Once post-2025 EW/HAM operations enter the training window, the model sees EW in contraction mode and learns to apply a smaller positive correction. This resolves itself as the training window extends.

---

## EW/KEF — Eurowings Iceland

| Month | Sched | Pred | Actual | Note |
|-------|-------|------|--------|------|
| Jul   | 8     | 8    | 8      | Fine |
| Aug   | 9     | 9    | 9      | Fine |
| Sep   | 9     | 9    | 3      | 6 last-minute cancellations |

**Root cause:** Schedule and model both correct. 6 flights cancelled at short notice — weather, crew, or ATC. No 6-month forecast can predict operational day-of cancellations. F+ also wrong.

**Fix:** Nothing at 6-month horizon. Short-range (1–7 day) forecasts could use ops/NOTAMs data.

---

## EN/BLL — Air Dolomiti Billund

**FLT WAPE: 64.8%. F+ WAPE: 3.0%**

| Month | Sched | Pred | Actual | Note |
|-------|-------|------|--------|------|
| Jul   | 31    | 30   | 30     | Fine |
| Aug   | 31    | 30   | 31     | Fine |
| Sep   | 30    | 62   | 30     | Schedule correct. Model adds +32 from training memory. |
| Oct   | 25    | 63   | 25     | Schedule correct. Model adds +38 from training memory. |

**Root cause — what the training data shows:** Throughout 2024, EN/BLL actual departures were consistently 1.5–2.5× the scheduled count (October 2024 ratio: 2.56, November 2024: 2.09). Air Dolomiti was routinely adding extra capacity at Billund beyond what it filed in advance. The model learned from this history that EN/BLL always flies far more than scheduled and applied a strong positive correction — even in Sep/Oct 2025 when the schedule was perfectly accurate and no extra flights operated.

**Why F+ does well here:** F+ applies a fixed kg/departure rate (from Mar/Apr/May 2025) to the **scheduled** departure count — it does not modify the flight count at all. The schedule was correct (25–30 flights). FLT's Phase 1 inflated the count. F+ wins simply by not touching a correct input.

**Fix:** More training data covering periods where EN/BLL followed the schedule accurately. Once the model sees enough examples of EN/BLL with ratio ≈ 1.0, it stops applying the strong positive correction. With a full seasonal cycle in training the model learns the difference between summer extra-capacity and autumn scheduled-only operations.

---

## XQ/WAW — SunExpress Warsaw

**FLT WAPE: 50.7%. F+ WAPE: 65.6% — FLT wins**

| Month | Sched | Pred | Actual | Note |
|-------|-------|------|--------|------|
| Jul   | 36    | 38   | 15     | Schedule already wrong. Model barely changes it. |
| Aug   | 35    | 38   | 17     | Same. |
| Sep   | 35    | 37   | 18     | Same. |
| Oct   | 35    | 41   | 24     | Same. |
| Nov   | 26    | 26   | 24     | Both fine. |
| Dec   | 18    | 18   | 17     | Both fine. |

**Root cause — what the training data shows:** The training data does contain the right seasonal signal: XQ/WAW summer 2024 actual/scheduled ratios were 0.47–0.64 — SunExpress consistently operated far fewer Warsaw flights than it filed in summer 2024. The model has seen this pattern. However, at a 220-day forecast horizon the signal is not applied strongly enough: the model nudges only slightly below schedule rather than cutting by half. The June 2025 ratio was 0.39, reinforcing the same pattern. There are two compounding issues:

1. **Model under-applies the seasonal signal at long horizons** — at 220 days the model is more conservative about large corrections. More training data strengthens this signal.
2. **Partially irreducible:** The July 2025 batch of cancellations had not been filed with schedulers by June 12. No model can predict cancellations that were not yet submitted. August and September were partially foreseeable from the seasonal pattern; July was not.

**Why F+ is worse:** F+ applies the June/July/August 2024 average kg/flight rate to all scheduled departures — it doesn't reduce the flight count at all, it just under-weights kg per flight. The flight count is the dominant driver at this scale, so F+ over-predicts even more than FLT.

**Fix:** More training data deepens the seasonal signal so the model applies larger corrections at long horizons when it has seen the pattern repeat across multiple years. The July irreducible component cannot be recovered without a post-June schedule amendment feed.

---

## XQ/BLL — SunExpress Billund

Small volumes. November: sched=4, pred=7, actual=4 — model adds +3. Both negligible in aggregate.

---

## LH/BLL — Lufthansa Billund

**FLT WAPE: 13.7%. F+ WAPE: 20.9% — FLT wins**

FLT consistently under-predicts LH/BLL (actual higher than predicted) while F+ over-predicts strongly (24–28% above actual). FLT is closer in every month. The absolute % is inflated by small volumes at BLL.

---

## BLL Overall

**FLT WAPE: 18.9%. F+ WAPE: 17.7% — F+ wins narrowly**

F+ wins BLL overall because EN/BLL Sep/Oct (where FLT is very wrong) outweighs LH/BLL (where FLT wins). Once EN/BLL is in the training window long enough to show the schedule-following pattern, BLL flips to a FLT win.

---

## HRG — Hurghada (FLT consistently loses)

**FLT WAPE: 12.2%. F+ WAPE: 8.4% — F+ wins all 6 months**

HRG is a leisure/holiday destination with strong charter operations. The key routes:

**EW/HRG:** FLT 8.4%, F+ 1.0%. F+ handles EW/HRG well — stable charter schedule, consistent kg/flight rate. FLT's model slightly over-predicts.

**WK/HRG (Condor):** FLT 20.2%, F+ 17.1%. Condor HRG is highly seasonal. Both models over-predict; FLT more so.

**SN/HRG (Brussels Airlines):** FLT 16.5%, F+ 11.3%. Similar pattern — both over-predict, FLT more.

**Root cause:** HRG charter routes have volatile actual uplifts relative to schedule (holiday demand spikes and troughs). F+'s 3-month rate happens to be well-calibrated for these stable charter patterns. FLT's model learns a more complex correction that adds noise on these routes.

**Fix:** More HRG training data. Currently only 17 months. With 3+ years the model learns seasonal HRG patterns properly.

---

## VIE — Vienna (FLT loses every month but margins are tiny)

**FLT WAPE: 2.1%. F+ WAPE: 1.2% — F+ wins all 6 months**

Both models are very accurate. VIE is dominated by OS/VIE (Austrian home hub) — a massive, stable, well-scheduled route. F+'s 3-month rate is almost perfectly calibrated for OS/VIE.

**Monthly margins are 0.2–1.8pp.** In absolute terms the difference is negligible for planning purposes. This is not a meaningful loss — both forecasts are excellent.

---

## WAW — Warsaw (FLT loses overall)

**FLT WAPE: 20.1%. F+ WAPE: 16.0% — F+ wins**

WAW is heavily distorted by two issues:

1. **EW/WAW** (see above) — Eurowings exit. Dominates the WAW bad result.
2. **XQ/WAW** (see above) — schedule over-stated in June. Both models wrong.

Fix EW/WAW and EW/FRA with the departure cap, fix XQ/WAW with a more recent schedule snapshot, and WAW flips to a FLT win.

**SN/WAW and LX/WAW** also over-predict in Nov/Dec but by smaller margins — winter schedule transition.

---

## HKG — Hong Kong (FLT wins but absolute WAPE is high)

**FLT WAPE: 18.1%. F+ WAPE: 35.5% — FLT wins by a large margin**

F+ shows −100% for several HKG routes in Nov/Dec — meaning F+ predicted 0 (no forecast at all for those combos). FLT at least predicts something meaningful.

**3S/HKG:** FLT 20.6%, F+ 43.6%. 3S doesn't publish long-range schedules; F+ has no forecast. FLT uses historical patterns.

**YF/HKG:** FLT 21.6%, F+ 40.7%. Same pattern.

**LH/HKG:** FLT 18.6%, F+ 25.6%. FLT wins. LH HKG has long-haul volume variance.

**LX/HKG:** FLT 4.2%, F+ 3.7%. Essentially tied — both very accurate.

**Root cause for high absolute %:** HKG routes are long-haul with high kg-per-flight. Small departure count errors translate to large kg errors. Training data is sparser for long-haul routes. More data = tighter predictions.

**Fix:** Extended training data (17 months → 3+ years).

---

## KEF — Keflavik (FLT wins narrowly)

**FLT WAPE: 19.6%. F+ WAPE: 20.8% — FLT wins by 1.2pp**

KEF is a leisure/tourism destination with high summer seasonality. Most routes operate only in summer.

**LH/KEF:** FLT 16.8%, F+ 10.9% — F+ wins. LH over-predicted Jul and Oct.
**OS/KEF:** FLT 27.2%, F+ 22.2% — F+ wins. Austrian KEF is highly seasonal.
**4Y/KEF:** FLT 24.2%, F+ 44.1% — FLT wins large (F+ has no 4Y forecast).
**WK/KEF:** FLT 12.5%, F+ 11.1% — F+ wins narrowly.
**EW/KEF:** FLT 35.8%, F+ 35.9% — essentially tied. Sep cancellations hit both.

Overall FLT edges F+ because it handles the routes F+ cannot predict (4Y, XQ) while being competitive on the rest.

---

## HAM — Hamburg

**FLT WAPE: 7.0%. F+ WAPE: 6.6% — F+ wins narrowly**

HAM is dominated by EW/HAM (Eurowings hub). Both models close. September is the weak month for FLT — EW filed amendments reducing HAM ops after June 12. Fix: more recent schedule snapshot.

---

## PMI — Palma de Mallorca

**FLT WAPE: 8.3%. F+ WAPE: 7.8% — F+ wins narrowly**

PMI is a summer package holiday destination. The main routes (EW/PMI, 4Y/PMI, LX/PMI) are all charter-heavy with strong seasonality. F+'s 3-month rate is well-calibrated for the consistent summer charter patterns. FLT slightly over-predicts on wind-down months (Nov/Dec).

---

## ORD — Chicago O'Hare

**FLT WAPE: 9.8%. F+ WAPE: 12.2% — FLT wins**

**3S/ORD:** FLT 14.9%, F+ 47.9% — FLT wins large. F+ has no 3S long-range forecast.
**LH/ORD:** FLT 11.3%, F+ 3.7% — F+ wins. LH ORD is a major hub route, well-calibrated for F+.
**YF/ORD:** FLT 10.8%, F+ 67.7% — FLT wins large. F+ has no YF long-range forecast.

---

## FRA — Frankfurt

**FLT WAPE: 1.4%. F+ WAPE: 7.9% — FLT wins strongly**

FRA is the largest airport by volume. FLT wins by 6.5pp — the biggest airport win in absolute percentage terms. The dominant routes (LH/FRA, OS/FRA, XQ/FRA) are all well-predicted. EW/FRA is the weak spot (see EW/FRA above) but small volume relative to total FRA.

---

## Defence — Why Use FLT Despite the Bad Cases?

**1. F+ is equally wrong on these routes — just expressed differently.**
When F+ predicts 0 it looks less wrong in % terms than FLT predicting 75 when actual is 12. But zero is useless to a planner. With the departure cap, FLT produces ≈0 for these cases — same number, explainable reason.

**2. The extreme errors are tiny volume relative to total.**
EW/WAW November actual: 37,000 kg. Full test: hundreds of millions of kg across 10 airports. A 525% error on 37,000 kg moves total WAPE by less than 0.1%.

**3. Every bad case traces to a specific, fixable data gap.**
- Delta model overrides correct schedule → departure cap (in testing now)
- Schedule correct but delta model overrides it → departure cap
- Schedule filed optimistically, amendments not yet in June snapshot → more recent schedule pull / real-time SSIM amendments
- Airline business decision after forecast date → real-time SSIM (Phase 2 roadmap)
- Last-minute operational cancellations → irreducible at 6-month horizon, no system solves this

**4. These results are our floor, not our ceiling.**
17 months of training data, no winter schedule feed yet. Every identified fix removes a specific error category. The improvement path is concrete and bounded.
