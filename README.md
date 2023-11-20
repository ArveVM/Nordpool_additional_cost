[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/custom-components/hacs)
![Version](https://img.shields.io/github/v/release/ArveVM/nordpool_additional_cost)

# Nordpool 'additional_cost'
Tired of calculating/changing templates for Nordpool-spot price additional cost? Look no further. This Template extension for home assistant makes additional_cost calculations easy!

By updating the correct data in this repository and updating to your HA through HACS, my goal is that we don't have to individually update addtional_cost in our HA-instances -> one should only use HACAS to update to the last version of this template!

### Inspiration
Petro31's [easy-time-jinja](https://github.com/Petro31/easy-time-jinja)  
Kemon - github-wizardry :)

<br />

## Concept
Two HA-integrations fetch Nordpool-spot prices; [Nordpool](https://github.com/custom-components/nordpool) and [Priceanalyzer](https://github.com/erlendsellie/priceanalyzer). On top of that price there come additional costs, which contains many components that constantly seem to change :(
So when a number of state fees and power subsidy constantly cange, power transport company have night/day tarrifs that often change, and power-broker-company change their markup,, how do you keep the actual total power-cost pr hour in your future planning?

We use this repo to gather and create a good template which is updated with: 
- country level fees/subsidies
- power distributor fees/subsidies
Then the template provides some inputs (including power-broker-markup), and out you get the full price you have to pay each hour (including next-day price)

<br />

## Dictionary:
- Spot-price = the actual cost of power,, decided hourly on the Nordpool spot market. Prices provided for this and next day
- Broker = the company that sell you power. They buy from Nordpool (perhaps via hedges/futures, long term contracts or spot), but sell to you on spot + broker_fee
- broker-fee = the markup your broker takes in addition to the spot price for selling you that actual power 
- Transport = cost-element for transportation of power to your location. Separate company/billing-features (at least in norway
- Transport company = areas have different companies delivering power, and each of them have a different price structure. Prices are higly regulated given theyr monopoly-situation, so they have to change their prices according to their cost-base etc etc quite often.
- Norway State fees = elavgift, enova-avgift, mva
- Norway power subsidy = governemnt subsidy to private individuals when the spot price is above certain level. Only applicable for an individuals main residence

<br />

# Installation
Install this in HACS or download the nordpool_additional_cost.jinja from this repository and place the files into your config\custom_templates directory.

PS: you need to enable HACS-experimental features (to enable downloading templates) and then select "template" when adding repository.
  
<br />

# Template definition:
------------------------------
## `add_cost(curr_hour, curr_month, broker_fee_43, current_price, price_in_cents, output) `

add_cost returns the additional cost given the right requested function. For example, if you request price with broker fee and state fee, select function-X 

Argument | Type | Default | Example | Description
:-:|:-:|:-:|:-:|---
curr_hour| ?? | - | `now().hour` | (Required) The hour of when add_cost should be calculated.
curr_month | ?? | No | `now().month` | (Required) The month of when add_cost should be calculated.
broker_fee | float | none | `0.029|float/1.25` | (Required) Additional fee for power broker. 
current_price| string | from nordpool/ priceanalyzer | `curr_price` | (Required) Must be price for your area without VAT.
price_in_cents| boolean | `False` | `True` | (required) If your `integration` is set up with prices in cents, mark True so the template can calculate right additional cost.
output | Int | - | `1` | (Required) Selected function to present.

<br />

### Functions
Selected outputs for additional_cost (adjustment to net-spot (no vat) which will have to be set in nordpool/priceanalyzer
1. price for transport and broker
2. price for tarnsport, broker and subsidized spot (net)
3. price for transport, broker and subsidized spot including state fees

outputs for test/verification or to use in other sensors/calculations
11. state fees

31. transport and broker
32. price for tarnsport, broker and subsidized spot

<br />


# How to use the template:

Create a Priceanalyzer or a Nordpool-sensor with the new template calculating additional cost


### Example for additional cost
Create a Priceanalyzer or a Nordpool-sensor with the following as additional cost
(select the last variable as the "function" you want to return

```jinja

{% set curr_hour = now().hour %}
{% set curr_month = now().month %}
{% set curr_price = current_price %}
{% from 'nordpool_additional_cost.jinja' import SpotTest %} 
{{ add_cost(
    curr_hour,
    curr_month,
    0.029|float/1.25,
    curr_price,
    true,
    2
    )}}
```

<br />

# Example for test in HA DeveloperTools / Template:

1. Create a Priceanalyzer or a Nordpool-sensor with nothing as additional cost
2. Then adjust first line in sample-template to the name of your sensor
3. add this code in SDevelopment tools / template, and you will get the return-value of the function you have specified. (select the last variable as the "function" you want to return
```
{% set current_price = (states('sensor.nordpool')|float) %}
{% set curr_hour = now().hour %}
{% set curr_month = now().month %}
{% set curr_price = current_price %}
{% from 'nordpool_additional_cost.jinja' import SpotTest %} 
{{ add_cost(
    curr_hour,
    curr_month,
    0.029|float/1.25,
    curr_price,
    true,
    2
    )}}
```

<br />

# Example for creation of a template sensor:

1. Create a Priceanalyzer or a Nordpool-sensor with nothing as additional cost
2. Then adjust first line in sample-template to the name of your sensor
3. add this code in 'configuration.yaml', packages-folder, gui-template-creator or whatever tool you have to create template-sensor
```
template:
  - sensor:
      - name: Nordpool Additional Cost
        state: >
          {% set current_price = (states('sensor.nordpool')|float) %}
          {% set curr_hour = now().hour %}
          {% set curr_month = now().month %}
          {% set curr_price = current_price %}
          {% from 'nordpool_additional_cost.jinja' import add_cost %} 
          {{ SpotTest(
              curr_hour,
              curr_month,
              0.029|float/1.25,
              curr_price,
              true,
              1
              )}}
```
------------------------

<br />

# Requirements
1. Must have Nordpool or Priceanalyzer installed, both use same method of adding additional_cost so they can piggyback on the cost-calculation this template (hopfully) can provide
2. HACS to get updates
3. Community focus on updating
   - There are many different countrys with separate cost-structures, and

<br />







  # Tested combination of Country/Power-distributor:
  
  ## Norway
  1. [Linja-Mørenett](https://static1.squarespace.com/static/61438478feca7941581468f1/t/64c773640c3b884d34be8a58/1690792804308/Tariffark+1.+juli+2023+-+hushald.pdf)
