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

**Delays happen. But have you ever thought about why?**

---

#### Why Planes Are Late

::: incremental

- Late incoming aircraft
- Weather disruptions
- Air traffic congestion
- Technical or maintenance issues
- Crew availability
- Catering or baggage loading delays
- Fueling took too long   

:::
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



🧱 PostgreSQL – Raw flight & fueling data
🔁 Rahla – ETL
🧠 Linear Regression model – Fueling duration prediction
🧪 MLflow – Model versioning / Serving
🔁 ArgoCD – Orchestration
🛠️ Theia – Continual development environment

---

### Current Results




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

- [odd documentation](https://docs.opendatadiscovery.org/)
- [Internal INT ODD URL](https://odd-fraalliance-platform.int.k8s.lsyesp.lhgroup.de)
- [Internal PROD ODD URL](https://odd-fraalliance-platform.prod.k8s.lsyesp.lhgroup.de)


