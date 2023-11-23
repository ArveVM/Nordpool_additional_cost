[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/custom-components/hacs)
![Version](https://img.shields.io/github/v/release/ArveVM/nordpool_additional_cost)

# Nordpool 'additional_cost'
Tired of calculating/changing templates for Nordpool-spot price additional cost? Look no further. This Template extension for home assistant makes additional_cost calculations easy!

By updating the correct data in this repository and updating to your HA through HACS, my goal is that we don't have to individually update addtional_cost in our HA-instances -> one should only use HACAS to update to the last version of this template!

### Inspiration
Petro31's [easy-time-jinja](https://github.com/Petro31/easy-time-jinja)  
Kemon - github-wizardry :)  
Erlend Sellie, both for the Priceanalyzer-integration but also a lot of help on templating and structure :)
<br />

### Disclaimer
This is a community concept,, under no cercomstanceas can I take responsability for not updated templates or wrong calculations. If you are worried - make a fork and create your own - or just close the browser and find anonther solution ;)

<br />

## Concept; add HACS-downloadable re-usable templates/macros
Two HA-integrations fetch Nordpool-spot prices ('current_price'); [Nordpool](https://github.com/custom-components/nordpool) and [Priceanalyzer](https://github.com/erlendsellie/priceanalyzer). On top of that 'current_price' there come additional costs, which contains many components that constantly seem to change :(
So when a number of state fees and power subsidy constantly change, power transport company have night/day tarrifs that often change, and power-broker-company change their markup,, how do you keep the actual total power-cost pr hour in your future planning?

We use this repo to gather and create good re-usable template(s) which is updated with: 
- country level fees/subsidies
- power distributor fees/subsidies
Then the template provides some inputs (including power-broker-markup), and out you get the full price you have to pay each hour (including next-day price)

<br />

## Dictionary:
- Spot-price = the actual cost of power,, decided hourly on the Nordpool spot market. Prices provided for this and next day
- Broker = the company that sell you power. They buy from Nordpool (perhaps via hedges/futures, long term contracts or spot), but sell to you on spot + broker_fee
- BrokerFee = the markup your broker takes in addition to the spot price for selling you that actual power 
- Transport = cost-element for transportation of power to your location. Separate company/billing-features (at least in norway
- Transporter = The company that transport the power to you house. 
- Norway State fees = elavgift, enova-avgift, mva
- Norway subsidy = governemnt subsidy to private individuals when the spot price is above certain level. Only applicable for an individuals main residence
- re-usable template https://www.home-assistant.io/docs/configuration/templating/#reusing-templates

<br />

# Installation
- Install this in HACS (you need to manually add this repository,, this is a custom-level HACS-template)
- or download the nordpool_additional_cost.jinja from this repository and place the files into your config\custom_templates directory.

PS: you need to enable [HACS-experimental](https://experimental.hacs.xyz/docs/faq/experimental_features/) features (to enable downloading templates) and then select "template" when adding repository.

### Updates 
.. new versions will show up in HA as updates (requirement= HACS-install),, just click update and latest version will be downloaded. 
Please remember to refresh template after downloading new version ??  
<img width="417" alt="image" src="https://github.com/ArveVM/nordpool_additional_cost/assets/96014323/20773a3d-4e97-4f2f-ac67-508323c840f1">

  
<br />

# Re-usable Templates:
## `nac(config) `

add_cost returns the additional cost given the right requested function. For example, if you request price with broker fee and state fee, select function-X 

Argument | Type | Default | Example | Description
:-:|:-:|:-:|:-:|---
function     | string | none | transport_broker_add | (Required) Selected function to present. See list of functions below.
transporter  | ?? | - | `no_linja_m` | (Required) Select transporter for transport-cost. Aee list of functions below.
broker_fee | float | none | `0.029|float/1.25` | (Required) Additional fee for power broker. 
timeslot   | datetime | none | `now().hour` | (Required) The hour of when add_cost should be calculated.
timeslot_price | string | from nordpool/ priceanalyzer | `curr_price` | (Required) Must be price for your area without VAT.
price_in_cents| boolean | `False` | `True` | (required) If your `integration` is set up with prices in cents, mark True so the template can calculate right additional cost.
output_vat| boolean | `False` | `True` | (required) If you want output to be including VAT, set true. 

### Examples
```jinja
{%- set curr_price = current_price %}
{%- set config = {
"function"       : "price_subz_tb_sf_vat_pure",
"transporter" 	 : "no_linja_m",
"brokerfee"      : 0.029|float/1.25,
"timeslot"       : now(),
"timeslot_price" : curr_price,
"price_in_cents" : true,
"output_vat"     : true
} -%}

{% from 'nordpool_additional_cost.jinja' import nac %} 
{{ nac(config)}}
```

### Function
Functions come in two types; 
- 'add' is to add the actual calculation as 'additional_cost' in Nordpool-/Priceanalyzer-sensor. Use this also for template sensors or other calculations,, will give the price according to cost the current hour
- while 'pure' is to use NP/PA to return the price but for specific calculation only - not concidering the price in Nordpool-/Priceanalyzer-sensor for the next day. (use 'pure' in test or buildup of graph showing levels/cost-structure. 

Function | Type | Description
:-:|:-:|---
transport | add | Return Transport for given timeslot/hour
transport_broker | add | Return Transport and BrokerFee for given timeslot/hour
transport_broker_pure | pure | Reduce NP/PA-spot to show only the fulctuations in cost on transport+broker during the day ahead
subsidy | add | Return subsidy for given timeslot/hour
price_subsidized | add | Return subsidized price for given timeslot/hour
price_subz_tb | add | Return subsidized price with transport and brookerfee for given timeslot/hour
price_subz_tb_sf_vat_pure | pure | calculate addition to NP/PA-spot to show full electricity-price the day ahead
<br />

### Transporter
Transport-companies: Areas have different companies delivering power, and each of them have a different price structure. Prices are higly regulated given theyr monopoly-situation, so they have to change their prices according to their cost-base etc etc quite often.
This is the list of added 'Transporters', with link to the actual price-book used when adding the price(s). If price is wrong/not updated,, please raise an issue/PR
Transporter | Description | Pricebook
:-:|:-:|---
no_linja_m | Norway Mørenett | [Linja-Mørenett](https://static1.squarespace.com/static/61438478feca7941581468f1/t/64c773640c3b884d34be8a58/1690792804308/Tariffark+1.+juli+2023+-+hushald.pdf)
no_tensio_s | Norway Tensio south | [Tensio-sør](https://ts.tensio.no/kunde/nettleie-priser-og-avtaler)
no_elvia    | Norway Elvia        | [Elvia](https://www.elvia.no/nettleie/alt-om-nettleiepriser/nettleiepriser-for-privatkunder/)

<br />

### Broker_fee
Add your broker's fee: Add a float representing the markup upon the spot-price you have in your deal with you broker. Ideally this would be a reference to a template sensor/input that you can change outside templates 

```yaml
input_number:
  # Set value for your power-brokers markup-fee upon Nordpool-spot.
  # It should be added to a card to be edited from GUI
  nordpool_additional_cost_broker_fee:
    name: Nordpool Additional Cost broker_fee
    initial: 0.0232
    min: -20
    max: 35
```

then you need to point the main-macro to that sensor 
```jinja
"brokerfee"      : states('input_number.nordpool_additional_cost_broker_fee'),
```



<br />

# Example for test in HA DeveloperTools / Template:

1. Create a Priceanalyzer or a Nordpool-sensor with nothing as additional cost
2. Then adjust first line in sample-template to the name of your sensor
3. add this code in SDevelopment tools / template, and you will get the return-value of the function you have specified. (select the last variable as the "function" you want to return
```
{% set current_price = (states('sensor.nordpool')|float) %}
{%- set curr_price = current_price %}
{%- set config = {
  "function"       : "price_subz_tb_sf_vat_pure",
  "transporter" 	 : "no_linja_m",
  "brokerfee"      : states('input_number.nordpool_additional_cost_broker_fee'),
  "timeslot"       : now(),
  "timeslot_price" : curr_price,
  "price_in_cents" : true,
  "output_vat"     : true
} -%}

{% from 'nordpool_additional_cost.jinja' import nac %} 
{{ nac(config)}}

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
          {%- set curr_price = current_price %}
          {%- set config = {
            "function"       : "price_subz_tb_sf_vat_pure",
            "transporter" 	 : "no_linja_m",
            "brokerfee"      : states('input_number.nordpool_additional_cost_broker_fee'),
            "timeslot"       : now(),
            "timeslot_price" : curr_price,
            "price_in_cents" : true,
            "output_vat"     : true
          } -%}
          
          {% from 'nordpool_additional_cost.jinja' import nac %} 
          {{ nac(config)}}
```
------------------------

<br />

# Requirements
1. Must have Nordpool or Priceanalyzer installed, both use same method of adding additional_cost so they can piggyback on the cost-calculation this template (hopfully) can provide
2. HACS to get updates, set to experimental mode to enable add/download templates
3. HA-core >= 2023.4 (to enable reusable-templates function released there)
4. Community focus on updating
   - There are many different countrys with separate cost-structures, and

<br />


# FAQ
Q: Why is a change in the macro/template not updated/refleceted in the dev/tools or sensors? 
A: Have you remembered to refresh the jinja2-templates?  
<img width="281" alt="image" src="https://github.com/ArveVM/nordpool_additional_cost/assets/96014323/b0bbcdb9-7168-474c-8682-b1e92863deb5">
Q: Why is a my supplier not listed ?
A: All in due time,,, Have you raised an issue with a link to your suppliers name/pricebook?  - or perhaps create a PR with the additional config for your supplier ;)  
Q: Why is only Norway functioning ?
A: All in due time,,, Have not got any data on fee-structure for other contries,, Please raise an issue with descriptions  a link to your suppliers name/pricebook?  - or perhaps create a PR with the additional config for your supplier ;)  


<br />

# Tested combination of Country/Power-distributor:

see 'Transporters'
<br />

draft:
- https://gis3.nve.no/ferdigkart/omraedekonsesjon_a0_2021.pdf
- https://www.nve.no/reguleringsmyndigheten/kunde/nett/nettleie/
- https://github.com/martinju/stromstotte/blob/master/data/database_nettleie_simple.csv
- https://github.com/erlendsellie/priceanalyzer/wiki/Additional-Costs
- tensio-Nord: https://tn.tensio.no/nettleie-og-tilknytningsavtaler
- Agder Energi: https://www.aenett.no/nettleie/tariffer/
- lnett (lyse nett)  https://www.l-nett.no/nettleie/priser-og-vilkar-privat/

# History:
2023-08- started working on re-usable templats - which was new functionality from HA 2023.4
2023-10 - 
