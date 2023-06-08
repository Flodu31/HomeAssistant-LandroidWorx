# HomeAssistant-LandroidWorx
Here is a small description to do the integration between your home assistant, and your Mower Landroid Worx, to have all information in a dashboard:

![image](https://user-images.githubusercontent.com/15648175/230577605-2d7adc00-b85e-4696-85fc-7d6b2e9c89d2.png)

## Configuration
Edit the file **/config/command_line.yaml** and add the following code, by replacing the username, password and serial number with your own:

```yaml

# Landroid Worx
  - sensor:
      name: rest_token
      scan_interval: 3600 # every 1 hour
      command: 'json=$(curl --location --request POST ''https://id.eu.worx.com/oauth/token'' --header ''Content-Type: application/x-www-form-urlencoded'' --data-urlencode ''client_id=150da4d2-bb44-433b-9429-3773adc70a2a'' --data-urlencode ''scope=*'' --data-urlencode ''client_secret=nCH3A0WvMYn66vGorjSrnGZ2YtjQWDiCvjg7jNxK'' --data-urlencode ''grant_type=password'' --data-urlencode ''username=YourUserName'' --data-urlencode ''password=YourPassword'' | jq -r ''.access_token'') && echo $json > /config/.landroidToken'
    
  - sensor:
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
        - name
  ```
  
Edit the file **/config/sensor.yaml** and add the following code:

```yaml

# Landroid Worx
  - platform: template
    sensors:
      landroid_blade_work_time:
        friendly_name: Blade Work Time
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
      landroid_rain_delay:
        friendly_name: Rain Delay
        icon_template: mdi:weather-rainy
        value_template: >-
            {% if (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['rain']['s'] == 1) %}
              {{ state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['rain']['cnt'] }} min
            {% else %}
              N/A
            {% endif %}
      landroid_status:
        # https://github.com/nibi79/worxlandroid/blob/master/src/main/java/org/openhab/binding/worxlandroid/internal/codes/WorxLandroidStatusCodes.java
        friendly_name: Status
        icon_template: mdi:information-outline
        value_template: >-
            {% if (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == -1) %}
              UNKNOWN
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 0) %}
              IDLE
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 1) %}
              Home
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 2) %}
              Start sequence
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 3) %}
              Leaving home
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 4) %}
              Follow wire
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 5) %}
              Searching home
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 6) %}
              Searching wire
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 7) %}
              Mowing
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 8) %}
              Lifted
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 9) %}
              Trapped
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 10) %}
              Blade blocked
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 11) %}
              Debug
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 12) %}
              Remote control
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 30) %}
              Going home
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 31) %}
              Zone training
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 32) %}
              Border cut
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 33) %}
              Border cut
            {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['ls'] == 34) %}
              Pause
            {% endif %}
      landroid_error:
        # https://github.com/nibi79/worxlandroid/blob/master/src/main/java/org/openhab/binding/worxlandroid/internal/codes/WorxLandroidErrorCodes.java
        friendly_name: Error
        icon_template: mdi:information-outline
        value_template: >-
          {% if (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == -1) %}
            UNKNOWN
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 0) %}
            No error!
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 1) %}
            Trapped
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 2) %}
            Lifted
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 3) %}
            Wire missing
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 4) %}
            Wire missing
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 5) %}
            Raining
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 6) %}
            Close door to mow
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 7) %}
            Close door to go home
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 8) %}
            Blade motor blocked
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 9) %}
            Wheel motor blocked
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 10) %}
            Trapped timeout
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 11) %}
            Upside down
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 12) %}
            Battery low
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 13) %}
            Reverse wire
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 14) %}
            Charge error
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 15) %}
            Timeout finding home
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 16) %}
            Mower locked
          {% elif (state_attr('sensor.landroid_rest', 'last_status')['payload']['dat']['le'] == 16) %}
            Battery over temperature
          {% endif %}

  ```
Create a new manual card and use the following code:

```yaml

title: FALA Mower
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
  - entity: sensor.landroid_error
  - entity: sensor.landroid_rain_delay
  - entity: sensor.landroid_battery
  - entity: sensor.landroid_distance_covered
  - entity: sensor.landroid_mower_work_time
  - entity: sensor.landroid_blade_work_time
  - entity: sensor.landroid_rsi
  - entity: sensor.landroid_battery_charge_cycles
  - entity: sensor.landroid_firmware_version
  - entity: sensor.landroid_last_status_update

```
