# Nordpool additional_costs
Tired of updating additional_costs every time your power-company or government changes some minor parameters?

By updating the correct data in this repository and updating it through HACS, my goal is that we don't have to individually update addtional_cost in our HA-instances


# Requirements
1. Must have Nordpool or Priceanalyzer installed, both use same method of adding additional_Cost so they can piggyback on the cost-calculation this template (hopfully) can provide
2. HACS to get updates
3. Community focus on updating
   - There are many different countrys with separate cost-structures, and
  

# How to use:

Create a Priceanalyzer or a Nordpool-sensor with the following as additional cost
(select the last variable as the "function" you want to return
```
{% set curr_hour = now().hour %}
{% set curr_month = now().month %}
{% set curr_price = current_price %}
{% from 'nordpool_additional_cost.jinja' import SpotTest %} 
{{ SpotTest(
    curr_hour,
    curr_month,
    0.029|float/1.25,
    curr_price,
    true,
    2
    )}}
```



  # Tested combination of Country/Power-distributor:
  
  ## Norway
  1. Linja-Mørenett
