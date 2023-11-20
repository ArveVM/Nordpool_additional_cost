# Nordpool additional_costs
Tired of updating additional_costs every time your power-company or government changes some minor parameters?

By updating the correct data in this repository and updating it through HACS, my goal is that we don't have to individually update addtional_cost in our HA-instances


# Requirements
1. Must have Nordpool or Priceanalyzer installed, both use same method of adding additional_Cost so they can piggyback on the cost-calculation this template (hopfully) can provide
2. HACS to get updates
3. Community focus on updating
   - There are many different countrys with separate cost-structures, and
  

# How to use:
```
{% from 'easy_time.jinja' import clock_icon %}

{# Return the current time icon #}
{{ clock_icon() }}

{# Return midnight or noon's current time icon #}
{{ clock_icon(0) }}
{{ clock_icon(12) }}
```



  # Tested combination of Country/Power-distributor:
  
  ## Norway
  1. Linja-Mørenett
