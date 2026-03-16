---
title: "FLT — Smarter Fuel Uplift Forecasting"
subtitle: datatactics GmbH
date: March 2026
theme: moon
transition: slide
highlight-style: breezeDark
progress: true
slideNumber: true
hash: true
navigationMode: linear
css: styles.css
---

### Our Mission

```{=html}
<div style="display:flex; flex-direction:column; align-items:center; justify-content:center; margin-top:2em; text-align:center; gap:1.6em;">

  <div style="font-size:1.6em; font-weight:bold; color:#FBF9F5; line-height:1.3;">
    Build a fuel uplift forecast that<br>
    <span style="color:#5BA3C9;">outperforms the current standard.</span>
  </div>

  <div style="width:60px; height:3px; background:#5BA3C9; border-radius:2px;"></div>

  <div style="font-size:1em; color:#aaa; max-width:880px; line-height:1.8;">
    F+ is the benchmark.<br>We want to beat it — consistently, measurably, and in production.
  </div>

</div>
```

---

### The Limits of F+

```{=html}
<div style="display:flex;flex-direction:column;gap:0.6em;margin-top:0.5em;font-size:0.8em;">
  <div style="background:#1a1a2e;border-left:4px solid #F7BB40;border-radius:6px;padding:0.4em 0.9em;color:#FBF9F5;line-height:1.35;">
    F+ multiplies a <strong>published schedule</strong> by <strong>historical average kg-per-flight rates</strong>.
  </div>
  <div style="display:flex;gap:0.7em;">
    <div style="flex:1;background:#2a1a1a;border:1px solid #6a3a3a;border-radius:8px;padding:0.45em 0.8em;">
      <div style="color:#ff6b6b;font-weight:bold;margin-bottom:0.15em;">❌ Blind to cancellations</div>
      <div style="color:#ccc;font-size:0.88em;line-height:1.35;">Schedule shows 60 flights. 30 operate. F+ forecasts all 60.</div>
    </div>
    <div style="flex:1;background:#2a1a1a;border:1px solid #6a3a3a;border-radius:8px;padding:0.45em 0.8em;">
      <div style="color:#ff6b6b;font-weight:bold;margin-bottom:0.15em;">❌ Static averages</div>
      <div style="color:#ccc;font-size:0.88em;line-height:1.35;">3-month lookback misses seasonal shifts and route exits.</div>
    </div>
    <div style="flex:1;background:#2a1a1a;border:1px solid #6a3a3a;border-radius:8px;padding:0.45em 0.8em;">
      <div style="color:#ff6b6b;font-weight:bold;margin-bottom:0.15em;">❌ Rates go stale</div>
      <div style="color:#ccc;font-size:0.88em;line-height:1.35;">kg-per-flight averages are updated manually. When operations change, F+ stays wrong.</div>
    </div>
  </div>
</div>
```

---

### Our Hypothesis

```{=html}
<div style="display:flex;flex-direction:column;align-items:center;margin-top:0.5em;text-align:center;gap:0.7em;">
  <div style="font-size:1.2em;font-weight:bold;color:#FBF9F5;line-height:1.4;max-width:880px;">
    <span style="color:#5BA3C9;">How many flights actually depart</span>
    and <span style="color:#5BA3C9;">how long they actually fly</span>
    predicts fuel uplift better than a static schedule.
  </div>
  <div style="width:60px;height:3px;background:#5BA3C9;border-radius:2px;"></div>
  <div style="display:flex;gap:1em;font-size:0.8em;color:#ccc;width:100%;">
    <div style="flex:1;text-align:left;background:#1a1a2e;border-radius:8px;padding:0.45em 0.75em;line-height:1.4;">
      <strong style="color:#F7BB40;">The challenge:</strong> At forecast time we only have the published schedule — not what will actually fly.
    </div>
    <div style="flex:1;text-align:left;background:#0d1a0d;border-radius:8px;padding:0.45em 0.75em;line-height:1.4;">
      <strong style="color:#10B981;">Our answer:</strong> First predict what will actually operate. Then predict fuel uplift from those corrected numbers.
    </div>
  </div>
</div>
```

---

### The FLT Pipeline

```{=html}
<div style="display:flex;gap:0.9em;margin-top:0.5em;font-size:0.8em;align-items:stretch;">
  <div style="flex:1;background:#102238;border:2px solid #5BA3C9;border-radius:10px;padding:0.6em 0.85em;">
    <div style="color:#5BA3C9;font-weight:bold;margin-bottom:0.35em;text-transform:uppercase;letter-spacing:0.05em;">Phase 1 — Schedule Correction</div>
    <div style="color:#FBF9F5;line-height:1.5;">
      <div style="margin-bottom:0.2em;">📥 <strong>Input:</strong> Published schedule (SSIM)</div>
      <div style="margin-bottom:0.2em;">⚙️ <strong>Model:</strong> LightGBM — predicts scheduled vs actual gap</div>
      <div>📤 <strong>Output:</strong> Corrected departures + flight minutes</div>
    </div>
    <div style="margin-top:0.45em;background:#0a1828;border-radius:5px;padding:0.25em 0.6em;font-size:0.8em;color:#9ecae1;">
      Learns how schedules drift from reality — per airline, airport, horizon, and season
    </div>
  </div>
  <div style="display:flex;align-items:center;font-size:1.5em;color:#F7BB40;padding:0 0.2em;">→</div>
  <div style="flex:1;background:#102238;border:2px solid #10B981;border-radius:10px;padding:0.6em 0.85em;">
    <div style="color:#10B981;font-weight:bold;margin-bottom:0.35em;text-transform:uppercase;letter-spacing:0.05em;">Phase 2 — Uplift Prediction</div>
    <div style="color:#FBF9F5;line-height:1.5;">
      <div style="margin-bottom:0.2em;">📥 <strong>Input:</strong> Corrected departures + flight minutes</div>
      <div style="margin-bottom:0.2em;">⚙️ <strong>Model:</strong> Formula base × ML correction ratio</div>
      <div style="margin-bottom:0.2em;font-size:0.85em;color:#9ecae1;font-style:italic;">base = corrected deps × kg/dep + corrected mins × kg/min</div>
      <div>📤 <strong>Output:</strong> Predicted fuel uplift (kg)</div>
    </div>
    <div style="margin-top:0.45em;background:#0a1828;border-radius:5px;padding:0.25em 0.6em;font-size:0.8em;color:#74c476;">
      ML corrects where route-average rates over- or underestimate
    </div>
  </div>
</div>
<div style="margin-top:0.5em;text-align:center;font-size:0.78em;color:#aaa;font-style:italic;">
  Better schedule inputs → better uplift predictions. The hypothesis drives both phases.
</div>
```

---

### FLT vs F+ — Month by Month

```{=html}
<div style="margin-top:0.2em;font-size:0.83em;">
  <div style="text-align:center;margin-bottom:0.4em;color:#aaa;font-size:0.76em;">
    WAPE = Weighted Absolute % Error &nbsp;·&nbsp; Lower is better &nbsp;·&nbsp; Jul–Dec 2025
  </div>
  <table style="width:100%;border-collapse:collapse;text-align:center;">
    <thead>
      <tr style="color:#aaa;font-size:0.76em;text-transform:uppercase;letter-spacing:0.05em;border-bottom:1px solid #333;">
        <th style="padding:4px 6px;text-align:left;">Month</th>
        <th style="padding:4px 6px;">F+</th>
        <th style="padding:4px 6px;">FLT</th>
        <th style="padding:4px 6px;">Result</th>
        <th style="padding:4px 6px;text-align:left;">Context</th>
      </tr>
    </thead>
    <tbody style="color:#FBF9F5;">
      <tr style="background:#0d1a0d;">
        <td style="padding:5px 6px;text-align:left;font-weight:bold;">July</td>
        <td style="padding:5px 6px;color:#e67e22;">2.4%</td>
        <td style="padding:5px 6px;color:#10B981;font-weight:bold;">1.6%</td>
        <td style="padding:5px 6px;">✅</td>
        <td style="padding:5px 6px;text-align:left;color:#aaa;font-size:0.82em;">Summer peak — both accurate, FLT tighter</td>
      </tr>
      <tr>
        <td style="padding:5px 6px;text-align:left;font-weight:bold;">August</td>
        <td style="padding:5px 6px;color:#e67e22;">2.7%</td>
        <td style="padding:5px 6px;color:#10B981;font-weight:bold;">1.9%</td>
        <td style="padding:5px 6px;">✅</td>
        <td style="padding:5px 6px;text-align:left;color:#aaa;font-size:0.82em;">Peak summer — FLT wins</td>
      </tr>
      <tr style="background:#1a1210;">
        <td style="padding:5px 6px;text-align:left;font-weight:bold;">September</td>
        <td style="padding:5px 6px;color:#10B981;font-weight:bold;">2.4%</td>
        <td style="padding:5px 6px;color:#e67e22;">3.2%</td>
        <td style="padding:5px 6px;">❌</td>
        <td style="padding:5px 6px;text-align:left;color:#aaa;font-size:0.82em;">Closest loss of the 6 months — Due to bad Phase 1 predictions</td>
      </tr>
      <tr style="background:#0d1a0d;">
        <td style="padding:5px 6px;text-align:left;font-weight:bold;">October</td>
        <td style="padding:5px 6px;color:#e67e22;">6.4%</td>
        <td style="padding:5px 6px;color:#10B981;font-weight:bold;">3.7%</td>
        <td style="padding:5px 6px;">✅</td>
        <td style="padding:5px 6px;text-align:left;color:#aaa;font-size:0.82em;">F+ degrades at season transition — FLT holds</td>
      </tr>
      <tr style="background:#0d1a0d;">
        <td style="padding:5px 6px;text-align:left;font-weight:bold;">November</td>
        <td style="padding:5px 6px;color:#e67e22;font-weight:bold;">24.2%</td>
        <td style="padding:5px 6px;color:#10B981;font-weight:bold;">7.6%</td>
        <td style="padding:5px 6px;">✅</td>
        <td style="padding:5px 6px;text-align:left;color:#aaa;font-size:0.82em;">Winter — F+ blind to route exits, FLT adapts</td>
      </tr>
      <tr style="background:#0d1a0d;">
        <td style="padding:5px 6px;text-align:left;font-weight:bold;">December</td>
        <td style="padding:5px 6px;color:#e67e22;font-weight:bold;">24.1%</td>
        <td style="padding:5px 6px;color:#10B981;font-weight:bold;">6.1%</td>
        <td style="padding:5px 6px;">✅</td>
        <td style="padding:5px 6px;text-align:left;color:#aaa;font-size:0.82em;">Winter — FLT is 4× more accurate than F+</td>
      </tr>
    </tbody>
  </table>
  <div style="margin-top:0.4em;text-align:center;background:#102238;border-radius:6px;padding:0.3em;font-size:0.82em;">
    <strong style="color:#F7BB40;">FLT wins 5 out of 6 months.</strong>
  </div>
  <div style="margin-top:0.3em;text-align:center;font-size:0.75em;color:#aaa;">
    Even excluding routes F+ cannot predict (3S &amp; YF — 5.8% of volume): <strong style="color:#10B981;">FLT 3.1%</strong> vs <strong style="color:#e67e22;">F+ 3.9%</strong> — FLT still wins!
  </div>
</div>
```

---

### Closing the Gap

```{=html}
<div style="margin-top:0.4em;font-size:0.76em;">
  <table style="width:100%;border-collapse:collapse;">
    <thead>
      <tr style="border-bottom:1px solid #3a3a4a;">
        <th style="text-align:left;padding:0.25em 0.6em;color:#aaa;font-weight:normal;font-size:0.85em;text-transform:uppercase;letter-spacing:0.06em;width:26%;">F+ Limitation</th>
        <th style="text-align:left;padding:0.25em 0.6em;color:#aaa;font-weight:normal;font-size:0.85em;text-transform:uppercase;letter-spacing:0.06em;">How FLT Addresses It</th>
        <th style="text-align:center;padding:0.25em 0.6em;color:#aaa;font-weight:normal;font-size:0.85em;text-transform:uppercase;letter-spacing:0.06em;width:11%;">Status</th>
      </tr>
    </thead>
    <tbody>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.3em 0.6em;color:#FBF9F5;">Schedule drift</td>
        <td style="padding:0.3em 0.6em;color:#ccc;">Phase 1 models predict actual departures &amp; flight minutes from the published schedule</td>
        <td style="padding:0.3em 0.6em;text-align:center;"><span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#10B981;padding:2px 10px;border-radius:20px;">Solved</span></td>
      </tr>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.3em 0.6em;color:#FBF9F5;">Horizon degradation</td>
        <td style="padding:0.3em 0.6em;color:#ccc;">Horizon-split models (1–90 / 91–220 days) each specialise on their forecast range</td>
        <td style="padding:0.3em 0.6em;text-align:center;"><span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#10B981;padding:2px 10px;border-radius:20px;">Solved</span></td>
      </tr>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.3em 0.6em;color:#FBF9F5;">No route history</td>
        <td style="padding:0.3em 0.6em;color:#ccc;">Fallback hierarchy: route rates → airport means → global mean. No city-pair history required.</td>
        <td style="padding:0.3em 0.6em;text-align:center;"><span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#10B981;padding:2px 10px;border-radius:20px;">Solved</span></td>
      </tr>
      <tr style="border-bottom:1px solid #2a2a3a;">
        <td style="padding:0.3em 0.6em;color:#FBF9F5;">Station disruptions</td>
        <td style="padding:0.3em 0.6em;color:#ccc;">Architecture supports a per-station disruption flag — volume redistributes to alternate stations when a station cannot fuel.</td>
        <td style="padding:0.3em 0.6em;text-align:center;"><span style="background:#3a2a1a;border:1px solid #7a5a2a;color:#F7BB40;padding:2px 10px;border-radius:20px;">Roadmap</span></td>
      </tr>
      <tr>
        <td style="padding:0.3em 0.6em;color:#FBF9F5;">Aircraft swaps</td>
        <td style="padding:0.3em 0.6em;color:#ccc;">seat_bin + kg/min rates capture aircraft size; within-bin swaps handled well</td>
        <td style="padding:0.3em 0.6em;text-align:center;"><span style="background:#2a2a1a;border:1px solid #5a5a3a;color:#ffd966;padding:2px 10px;border-radius:20px;">Partial</span></td>
      </tr>
    </tbody>
  </table>
  <div style="margin-top:0.5em;font-size:0.88em;color:#aaa;line-height:1.5;">
    17 months of training data — every new month improves accuracy at every horizon.
    More airports = stronger shared patterns. <strong style="color:#FBF9F5;">These results are our floor, not our ceiling.</strong>
  </div>
</div>
```

---

### End-to-End Results

<iframe scrolling="no" style="border:none;" seamless="seamless"
  data-src="assets/oracle_comparison.html" height="615" width="100%"></iframe>

---

### Roadmap

```{=html}
<div style="display:flex;align-items:stretch;gap:0;margin-top:0.7em;font-size:0.78em;">
  <div style="flex:1;background:#1a2a1a;border:1px solid #3a6a4a;border-radius:8px 0 0 8px;padding:0.6em 0.8em;border-right:none;">
    <div style="color:#10B981;font-weight:bold;font-size:0.88em;text-transform:uppercase;margin-bottom:0.25em;">✅ Now</div>
    <div style="color:#F7BB40;font-size:0.8em;margin-bottom:0.2em;font-weight:bold;">Proof of Concept</div>
    <div style="color:#ccc;font-size:0.82em;line-height:1.4;">10 airports · 5/6 months FLT wins · End-to-end pipeline validated</div>
  </div>
  <div style="flex:1;background:#121e12;border:1px solid #3a5a3a;padding:0.6em 0.8em;border-left:none;border-right:none;">
    <div style="color:#74c476;font-weight:bold;font-size:0.88em;text-transform:uppercase;margin-bottom:0.25em;">Phase 1</div>
    <div style="color:#F7BB40;font-size:0.8em;margin-bottom:0.2em;font-weight:bold;">Multi-Scenario Validation</div>
    <div style="color:#ccc;font-size:0.82em;line-height:1.4;">· More data · Extend scenarios into the future</div>
  </div>
  <div style="flex:1;background:#0d1a18;border:1px solid #2a5a4a;padding:0.6em 0.8em;border-left:none;border-right:none;">
    <div style="color:#5BA3C9;font-weight:bold;font-size:0.88em;text-transform:uppercase;margin-bottom:0.25em;">Phase 2</div>
    <div style="color:#F7BB40;font-size:0.8em;margin-bottom:0.2em;font-weight:bold;">Full Airport Expansion</div>
    <div style="color:#ccc;font-size:0.82em;line-height:1.4;">Scale beyond 10 airports · Develope Disruption flag</div>
  </div>
  <div style="flex:1;background:#0d1520;border:1px solid #2a4a6a;padding:0.6em 0.8em;border-left:none;border-right:none;">
    <div style="color:#9ecae1;font-weight:bold;font-size:0.88em;text-transform:uppercase;margin-bottom:0.25em;">Phase 3–4</div>
    <div style="color:#F7BB40;font-size:0.8em;margin-bottom:0.2em;font-weight:bold;">Production · 4–18 Month Horizon</div>
    <div style="color:#ccc;font-size:0.82em;line-height:1.4;">Long range horizon → 15–18 month</div>
  </div>
  <div style="flex:1;background:#1a1a2e;border:2px solid #F7BB40;border-radius:0 8px 8px 0;padding:0.6em 0.8em;border-left:none;">
    <div style="color:#F7BB40;font-weight:bold;font-size:0.88em;text-transform:uppercase;margin-bottom:0.25em;">Phase 5</div>
    <div style="color:#F7BB40;font-size:0.8em;margin-bottom:0.2em;font-weight:bold;">Full System Launch</div>
    <div style="color:#ccc;font-size:0.82em;line-height:1.4;"><strong style="color:#F7BB40;">~400 airports</strong> · Continuous retraining · Fully integrated</div>
  </div>
</div>
<div style="margin-top:0.5em;text-align:center;font-size:0.78em;color:#aaa;">
  Today: 10 airports · <strong style="color:#FBF9F5;">96,993 t</strong> less misorder in 6 months · Every airport added compounds the impact.
</div>
```
