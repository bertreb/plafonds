# Dutch energy plafonds
Home-assistant python script for the Dutch energy plafonds in 2023.

This script can generate 6 todays plafonds for gas and electricity. 
- gas, today
- gas, today - cummulative for the month
- gas, today - cummulatief for the year
- electricity, today
- electricity, today - cummulative for the month
- electricity, today - cummulatief for the year
You can select which of the plafond you want to use.

Instructions for using this script
1. put the script 'plafond.py' in the HA directory /config/python_scripts (create, if it doesn't excist)
2. add the content of 'services.yaml' to the file 'services.yaml' in the HA directory /config/python_scripts
3. Reload the 'python_script' in HA (development tool->yaml-configuration -> reload python scripts)
4. Create the plafond sensors you want. I used a template sensor with the following structure for the daily gas plafond (template.yaml)
```
- sensor:
  - name: gasplafond dag
    unit_of_measurement: "m³"
    icon: "mdi:fire"
    device_class: gas
    state: >
      {% if states('sensor.gasplafond_dag')|float(0) > 0 %}
        {{ '{:.2f}'.format(states('sensor.gasplafond_dag')|float(0)) }}
      {% endif %}
```
This way you can format the presentation in the dashboard (2 decimals in the case) and prevent 0's in the history due to unknown values on startup.

5. Create and automation the runs every day just after midnight, runs on reload of the template sensors or automations and restart of HA.
```
alias: plafonds update
description: ""
trigger:
  - platform: event
    event_type: event_template_reloaded
    event_data: {}
  - platform: event
    event_type: automation_reloaded
    event_data: {}
  - platform: homeassistant
    event: start
  - platform: time
    at: "00:01:00"
condition: []
action:
  - service: python_script.plafonds
    data:
      gasplafond_dag: sensor.gasplafond_dag
      gasplafond_maand_cummulatief: sensor.gasplafond_maand
      gasplafond_jaar_cummulatief: sensor.gasplafond_dag_cummulatief
      electriciteitplafond_jaar_cummulatief: sensor.electriciteitsplafond_maand_cummulatief
      electriciteitplafond_maand_cummulatief: sensor.electriciteitsplafond_maand
      electriciteitplafond_dag: sensor.electriciteitsplafond_dag
mode: restart
```

6. Reload the automation (development tool->yaml-configuration -> reload automations)
7. The sensor values should be updated to todays plafond values

