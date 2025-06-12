---
author: Samuel McGuire 
title: Datatactics GmbH
subtitle: The Surprising Complexity of Fueling an Aircraft
date: 17-06-2025
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

#### Has Your Flight Ever Been Delayed?

![](assets/airport_delays.png){ width=60% }

**Delays happen. But we always blame the weather?**

---

#### Why Planes Are Late

::: incremental

- Late incoming aircraft
- Weather disruptions
- Technical or maintenance issues
- Crew availability
- Catering or baggage loading delays
- Fueling took too long  -> REALLY?? 

:::
--- 



#### Costs of late planes

According to Eurocontrol in 2023 27% of delays were due to maintanence, baggage handling, boarding and fueling issues. This cost airlines 275 million € in fuel burn, crew time, lost opportunity and almost 650 million € in compensation given to customers from EU261.

---

#### How Much to Fuel

::: incremental
- Where the plane is flying to (with reserves taken into account)
- How much fuel is already in the plane
- Weight Matters
- Tankering (Used to Be Common Practice)
- New EU Tankering Regulations (2024)
:::


### When Will Fueling Actually Finish?

1. Pilot places fuel order  
2. Fueling starts → Prediction kicks in here  
3. Fueling ends → Flight departs  

---


### Pipeline

![](assets/prediction_flow.png){ width=80% }

---


### Uplift Prediction


::: incremental
- Uplift = Block Fuel - FOB 
- This is what the pilot orders
- the prediction is made using the FOB, aircraft type and minutes to take off
:::

---

### Techstack

- 🧱 PostgreSQL – Raw flight & fueling data
- 🔁 Rahla – ETL - not in the classic sense
- 🧠 Linear Regression model – Fueling duration prediction
- 🧪 MLflow – Model versioning / Serving
- ☸️ Kubernetes – Deployment platform
- 🔁 ArgoCD – Orchestration
- 🛠️ Theia – Continual development environment

---

### Model comparison

![](assets/model_comparison)

---

### Current Results


<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/absolute_error_quantiles.html" height="450" width="100%"></iframe>

---


### What is this prediction even for?

    We asked: “What’s the real point of the prediction?”

    The answer: to improve visibility and coordination

    Before departure:
    🛫 Fueling must be done
    🍽 Catering must be finished
    👨‍✈️ Crew needs a clear status

    Simple monitoring → Smoother operations → Fewer delays

    Currently prediciton is made after the fuel is ordered
---

### Future Plans

- Currently prediciton is made after the fuel is ordered
- Long term goal is use the preiction to trigger the fueling
- This will ensure the fueling is done before planned take off


### Links

- [These sides](https://samueladamsmcguire.github.io)
- [datatactics website](https://www.datatactics.de/)
- [datatactics LinkedIn](https://www.linkedin.com/company/datatactics-gmbh)
- [My Personal LinkedIn](https://www.linkedin.com/in/samuel-mcguire/)



