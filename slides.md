---
title: Uplift Forecast
subtitle: DataTactics GmbH
date: 26-08-2025
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

### Staus Quo 

The current tool assigns the average uplift from the last three full calendar months for the city pair and aircraft type to each event in the schedule and sums everything up.

---

### Goal: More efficient fuel Uplift forecast

Taking a different approach we will try to model the changes in the plan flight from schedule to actual in order to have a more precise flight plan that can be used to calculate the uplift needed.

<!-- <img src="assets/dT-blue.svg" alt="Logo" width="60" style="position: absolute; bottom: 1rem; right: 1rem;" /> -->
<!-- <img src="assets/dT-blue.svg" alt="Logo" width="60" style="position: absolute; top: 1rem; left: 1rem;" /> -->


---

### Assumptions

1. Using a seasonal avg of mins flown will be better than the scheduled min
2. Minutes flown and total departures have a direct relationship with uplift
3. Due to scheduling flux modeling schedule changes will lead to more accurate forecasts



---

### Problems that could hurt forecast

1. Uplift values to train model 2 can only be used from 2025 due to new tankering regulations 
2. Fueling is still adjusting to the tankering regulations
3. Anolomies such as weather events and econmic changes could affect the forecast


--- 



### Schedule behavior analysis

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/schedule_change_analysis_by_airline_designator.html" height="450" width="100%"></iframe>

---

### Our Idea Model 1

::: incremental

- Scheduled and actual flight plan aree rarely the same
- Model difference between scheduled and actual flights
- Forecast: Number of departures and minutes flown

:::

---

### Model 1 input features

![](assets/model_1_features.png)

---

### Our Idea Model 2

::: incremental

- Use output of model 1 as input to predict total uplift
- Instead of using avg try other ML based models 
- Compare the results to status quo as well as actual burnoff

:::


---

### Model 2 input features

![](assets/model_2_input.png)

---

### Model 2 General results

![](assets/lightgbm_uplift_error_distribution.png)

--- 


### Model Flow Chart


![](assets/pipeline.png)

---


### General Results Model 1


![](assets/july_preds.png)

---

### General Results Model 2


<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/absolute_error_quantiles.html" height="450" width="100%"></iframe>

---

### Current Status

The model pipeline has been set. However the comparison with the actual uplift is still not matching up. 
We plan on improving model 1 while getting model 2 to a place where the uplift forecast is more accurate and eventually better than the status quo.


### Next Steps

- Model 2: get reasonable results from the forecast
- Model 1: improve, add more features such as enconmic impact, waeather and so on
- Set up in test environment for rigorous testing 
- comparing with current forecast and actual uplift


