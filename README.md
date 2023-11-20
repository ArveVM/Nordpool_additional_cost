# Nordpool 'additional_costs'
Templating tool to assist integrations: Nordpool or Priceanalyzer

## Tired of updating additional_costs every time your power-company or government changes some minor parameters?

Nordpool spot-prices can be fetched to HA with two integrations; Nordpool and Priceanalyzer.
Both of them have the ability to add 'additional_cost' to the spot price

So when a number of state fees and power subsidy constantly cange, power transport company have night/day tarrifs, and power-broker-company change their markup,, how do you keep the actual total power-cost pr hour in your future planning?


By updating the correct data in this repository and updating it through HACS, my goal is that we don't have to individually update addtional_cost in our HA-instances


## Concept
The entire community gather and create a good template which is updated with 
- country level fees/subsidies
- power distributor fees/subsidies
Then the template provides some inputs (including power-broker-markup), and out you get the full price you have to pay each hour (including next-day price)

### Functions
Selected outputs for additional_cost (adjustment to net-spot (no vat) which will have to be set in nordpool/priceanalyzer
1. price for transport and broker
2. price for tarnsport, broker and subsidized spot (net)
3. price for transport, broker and subsidized spot including state fees

outputs for test/verification or to use in other sensors/calculations
11. state fees

31. transport and broker
32. price for tarnsport, broker and subsidized spot


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


# How to test:

Create a Priceanalyzer or a Nordpool-sensor with nothing as additional cost
Then adjust first line in sample-template to the name of your sensor
(select the last variable as the "function" you want to return
add this code in SDevelopment tools / template, and you will get the return-value of the function you have specified.
```
{% set current_price = (states('sensor.nordpool')|float) %}
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
