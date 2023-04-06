# HomeAssistant-LandroidWorx
Integration for Mower Landroid Worx

## Configuration
Edit the file **/config/sensor.yaml** and add the following code, by replacing the username, password and serial number with your own:

```yaml

# Landroid Worx
  - platform: command_line
    name: rest_token
    scan_interval: 3600 # every 1 hour
    command: 'json=$(curl --location --request POST ''https://id.eu.worx.com/oauth/token'' --header ''Content-Type: application/x-www-form-urlencoded'' --data-urlencode ''client_id=150da4d2-bb44-433b-9429-3773adc70a2a'' --data-urlencode ''scope=*'' --data-urlencode ''client_secret=nCH3A0WvMYn66vGorjSrnGZ2YtjQWDiCvjg7jNxK'' --data-urlencode ''grant_type=password'' --data-urlencode ''username=YourUsername'' --data-urlencode ''password=YourPassword'' | jq -r ''.access_token'') && echo $json > /config/.landroidToken'
    
  - platform: command_line
    name: landroid_rest
    scan_interval: 60 # every 1 minute
    command: 'curl -X GET -H "Accept: application/json" -H "Content-Type: application/json" -H "Authorization: Bearer $(cat /config/.landroidToken)" https://api.worxlandroid.com/api/v2/product-items/YourSerialNumber?status=1'
    json_attributes:
      - blade_work_time
      - mower_work_time
      - firmware_version
      - distance_covered
      - battery_charge_cycles
      - last_status
      - dat

  - platform: template
    sensors:
      landroid_blade_work_time:
        friendly_name: Blade Work Time
        unit_of_measurement: 'h'
        icon_template: mdi:robot-mower-outline
        value_template: >-
          {% set t = state_attr("sensor.landroid_rest", "blade_work_time") | int(0) %}
          {% if t != 0 %}
            {{ "%0d J %0.02d H %0.02d M" | format(t // 1440, ((t % 1440) // 60), t % 60) }}
          {%- else -%}
            {{ '0 min' }}
          {% endif %}
      landroid_mower_work_time:
        friendly_name: Mower Total Time
        unit_of_measurement: 'h'
        icon_template: mdi:robot-mower-outline
        value_template: >-
          {% set t = state_attr("sensor.landroid_rest", "mower_work_time") | int(0) %}
          {% if t != 0 %}
            {{ "%0d J %0.02d H %0.02d M" | format(t // 1440, ((t % 1440) // 60), t % 60) }}
          {%- else -%}
            {{ '0 min' }}
          {% endif %}  
      landroid_firmware_version:
        friendly_name: Firmware Version
        value_template: "{{ (state_attr('sensor.landroid_rest', 'firmware_version')) }}"
      landroid_distance_covered:
        friendly_name: Distance Covered
        icon_template: mdi:map-marker-distance
        unit_of_measurement: 'km'
        value_template: "{{ (state_attr('sensor.landroid_rest', 'distance_covered') | float / 1000) | round }}"
      landroid_battery:
        friendly_name: Battery
        unit_of_measurement: '%'
        icon_template: mdi:battery
        value_template: "{{ (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['bt']['p']) | float(0) }}"
      landroid_battery_charge_cycles:
        friendly_name: Battery Charge Cycle
        icon_template: mdi:battery-charging-30
        value_template: "{{ (state_attr('sensor.landroid_rest', 'battery_charge_cycles')) }}"
      landroid_last_status_update:
        friendly_name: Last Update
        icon_template: mdi:update
        value_template: "{{ as_timestamp( state_attr('sensor.landroid_rest', 'last_status')['timestamp']) | timestamp_custom('%d.%m.%Y %H:%M:%S') }}" 
      landroid_rsi:
        friendly_name: Wifi signal
        unit_of_measurement: 'dBm'
        icon_template: mdi:wifi
        value_template: "{{ (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['rsi']) }}"
      landroid_status:
        friendly_name: Status
        icon_template: mdi:information-outline
        value_template: >-
            {% if (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['rain']['s'] == 0) %}
              Ready
            {% else %}
              Rain Delay
            {% endif %}

  ```
Create a new manual card and use the following code:

```yaml

title: FALA Mower Status
header:
  type: picture
  image: /local/images/landroid-worx.jpg
  tap_action:
    action: none
  hold_action:
    action: none
show_header_toggle: false
state_color: false
type: entities
entities:
  - entity: sensor.landroid_status
    name: Status
  - entity: sensor.landroid_battery
  - entity: sensor.landroid_distance_covered
  - entity: sensor.landroid_mower_work_time
  - entity: sensor.landroid_blade_work_time
  - entity: sensor.landroid_rsi
  - entity: sensor.landroid_battery_charge_cycles
  - entity: sensor.landroid_firmware_version
  - entity: sensor.landroid_last_status_update

```
