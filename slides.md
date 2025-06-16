---
author: Samuel McGuire - Data Science Lead
title: Datatactics GmbH
subtitle: The Surprising Complexity of Fueling an Aircraft
date: 18-06-2025
theme: moon # https://revealjs.com/themes/
transition: concave # https://revealjs.com/transitions/
# see all the options here: https://revealjs.com/config/
highlight-style: breezeDark
progress: true
slideNumber: true
hash: true
navigationMode: linear
autoPlayMedia: true


---

### Has Your Flight Ever Been Delayed?

![](assets/airport_delays.png){ width=60% }

**Annoying to pax, expensive for all!**

<img src="assets/dT-blue.svg" alt="Logo" width="60" style="position: absolute; bottom: 1rem; right: 1rem;" />
<!-- <img src="assets/dT-blue.svg" alt="Logo" width="60" style="position: absolute; top: 1rem; left: 1rem;" /> -->


--- 

<h3>✈️ Why Planes Get Delayed</h3>

<table style="width: 100%; font-size: 90%;">
  <tr>
    <th style="text-align: left;">Cause</th>
    <th style="text-align: left;">Description</th>
  </tr>
  <tr>
    <td>🚶 Passenger Flow</td>
    <td>Congestion at checkpoints</td>
  </tr>
  <tr>
    <td>🔁 Late Connex</td>
    <td>Planes wait depending</td>
  </tr>
  <tr>
    <td>🧳 Baggage</td>
    <td>Slow or misplaced luggage</td>
  </tr>
  <tr>
    <td>🌧️ Weather</td>
    <td>Storms or low visibility</td>
  </tr>
  <tr>
    <td>⛽ <strong>Fueling Delays</strong></td>
    <td><em>Common — but solvable!</em></td>
  </tr>
</table>

---

### 💸 Costs of Late Planes

::: incremental

- 27% of delays are due to aircraft turnaround (incl. fueling) issues
- €275 million in airline costs (fuel, crew delays, losses)
- €650 million in EU261 passenger compensation


<br>

<span style="font-size:0.7em; color:gray;">Source: <a href="https://www.eurocontrol.int/sites/default/files/2024-06/eurocontrol-performance-review-report-2023.pdf" target="_blank">2023 Eurocontrol Performance Review</a></span>


:::


---


<h3>🛢️ How Much Fuel Is Needed?</h3>

<table style="width: 110%; font-size: 90%;">
  <tr>
    <th style="text-align: left;">Factor</th>
    <th style="text-align: left;">Impact</th>
  </tr>
  <tr>
    <td>🗺️ Route</td>
    <td>Distance and more</td>
  </tr>
  <tr>
    <td>✈️ Aircraft</td>
    <td>Type Specification</td>
  </tr>
  <tr>
    <td>⛽ FOB</td>
    <td>Current tank level</td>
  </tr>
    <tr>
    <td>🌬️ Weather</td>
    <td>Headwinds</td>
  </tr>
</table>

---

### ⛽ Fueling Timeline: When Will It Be Done?

1. 🧑‍✈️ Pilot places fuel order (Uplift = Block Fuel – FOB)  
2. 🚛 Fueling starts — predict how long it will take
3. 🛫 Helps align catering, crew, and departure timing
4. ✅ Fueling ends → Aircraft ready to depart

**Knowing the fueling end time = fewer surprises & smoother handovers**



---

### 🔍 Modeling the Fueling duration

**Which features were relevant?**

::: incremental
- 🛩 Aircraft type captures weight, fuel throughput, engine count — all embedded in a single variable
- ⛽ Remaining uplift = Block Fuel – Fuel On Board → directly tied to fueling time
- ⏱ Minutes until takeoff encodes operational urgency: Time pressure, Likelihood of parallel fueling (more trucks dispatched), Typical ramp behavior
:::


---

### Model comparison

![](assets/model_comparison.png)

---

### Results per aircraft type


<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/absolute_error_quantiles.html" height="450" width="100%"></iframe>

---


### Message to Prediction Pipeline

::: incremental
- 📨 Queue receives messages
- 🧠 Parse messages into structured data
- 🗃️ Insert parsed messages into the database
- 🔁 Duplicate message for prediction route
- 🔍 Extract hashkey and message type
- ❓ Check message type
- 🧩 Join message with context
- 📤 Send features to Mlflow served model
- 📥 Receive predicted fueling duration
- 🧾 Concatenate prediction with original message 
- 🛠️ Update database row with prediction

:::

---

### What is this prediction even for?

    We asked: “What’s the real point of the prediction?”

    The answer: to improve visibility and coordination

    Before departure:
    🛫 Fueling must be done
    🍽 Catering must be finished
    👨‍✈️ Crew needs a clear status

    Simple monitoring → Smoother operations → Fewer delays

---


### Completely Open Source Techstack

- 🧱 PostgreSQL – Raw flight & fueling data
- 🔁 Rahla – Event-driven data processing
- 🧠 SciKit Learn – model
- 🧪 MLflow – Model versioning / Serving
- ☸️ Kubernetes – Deployment platform
- 🔁 ArgoCD – Orchestration
- 🛠️ Theia – Continual development environment

---

### Some dT Projects in the airline industry

![](assets/dT-blue.svg){ width=200px align=right }

- 🔁 Fuel Emission Forecasting 
- 👥 Passenger Flow  
- 🛰️ Flight Positioning 
- ⛽ Fueling Duration 
- 💳 Fuel Purchase Forecasting

--- 

### Links

- [My Github](https://github.com/samueladamsmcguire)
- [datatactics website](https://www.datatactics.de/)
- [datatactics LinkedIn](https://www.linkedin.com/company/datatactics-gmbh)

---

### Let's connect

![](assets/linkedin.jpg){ width=60% }


