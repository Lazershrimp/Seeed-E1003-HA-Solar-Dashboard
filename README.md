# Seeed-E1003-HA-Solar-Dashboard
A custom solar dashboard for the Seeed re-Terminal E1003 e-ink display, connected to Home Assistant by ESPHome 
An ESPHome configuration for the Seeed reTerminal E1003 10.3-inch e-ink display, showing solar production data from Home Assistant.
![Display photo](images/display.jpg)
<!-- Replace with your own photo -->
---
Based on
This project builds on the ESPHome driver by koosoli, which provides the custom `it8951_reterminal_e1003` display component. The driver handles low-level communication with the IT8951 e-ink controller. This repository adds a complete solar dashboard layout with Home Assistant sensor integration.
---
Features
Section 1 — Solar energy today: Four boxes showing produced, exported, imported and total consumed energy
Section 2 — Last 7 days: Rolling display of the last 6 days with yesterday rightmost, updates daily
Section 3 — Monthly overview: All 12 months in two rows; months without data show `---`
Section 4 — Per year + Lifetime totals: Yearly production (current year auto-summed from monthly helpers) alongside lifetime produced, self-consumed, imported and exported totals
Battery and WiFi status icons in top-right corner
Physical refresh button (KEY0/GPIO3) triggers immediate display update
HA button entity for remote refresh from Home Assistant dashboard
Night mode: No display updates between 22:00 and 06:00 to save battery
Hourly updates during daytime (06:00–22:00)
WiFi power save mode enabled
---
Hardware
Seeed reTerminal E1003 (ESP32-S3, IT8951E/DX, 10.3" e-ink ED103TC2)
Built-in 3000mAh Li-ion battery
USB-C charging
---
Requirements
ESPHome driver
Clone or download the custom component from koosoli's repository and place the `custom_components` folder in your ESPHome configuration directory.
Home Assistant sensors
The following sensors must exist in your Home Assistant instance. These are typically provided by your inverter integration (e.g. Solis, SolarEdge, Huawei):
Sensor	Description
`sensor.daily_pv_generation`	Solar energy produced today
`sensor.daily_exported_energy`	Solar energy exported to grid today
`sensor.daily_imported_energy`	Grid energy imported today
`sensor.daily_consumed_energy`	Total energy consumed today
`sensor.total_pv_generation`	Lifetime solar production
`sensor.total_imported_energy`	Lifetime grid import
`sensor.total_direct_energy_consumption`	Lifetime self-consumed solar
`sensor.total_exported_energy`	Lifetime solar export
`sensor.monthly_pv_generation`	Current month solar production (from inverter)
Home Assistant helpers (input_number)
Create the following helpers in HA under Settings → Devices & Services → Helpers → Number. Set minimum = -1 (used as "no data" sentinel), maximum = 99999, step = 0.1, unit = kWh.
Weekdays (updated nightly by automation):
`input_number.solar_mandag`, `solar_tisdag`, `solar_onsdag`, `solar_torsdag`, `solar_fredag`, `solar_lordag`, `solar_sondag`
Months (updated nightly by automation):
`input_number.solar_januari`, `solar_februari`, `solar_mars`, `solar_april`, `solar_maj`, `solar_juni`, `solar_juli`, `solar_augusti`, `solar_september`, `solar_oktober`, `solar_november`, `solar_december`
Years (enter manually):
`input_number.solar_2023`, `input_number.solar_2024`, `input_number.solar_2025`
> **Note:** Helpers with value `-1` display as `---` on the screen. Set helpers for months/days not yet started to `-1`.
---
Automations
Daily — update current weekday
Runs at 23:59 each night and writes `sensor.daily_pv_generation` to the correct weekday helper. Values are overwritten at the start of each new week.
```yaml
alias: Spara daglig solenergi per veckodag
description: Saves today's solar production to the correct weekday helper at midnight
triggers:
  - at: "23:59:00"
    trigger: time
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ now().weekday() == 0 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_mandag
            data:
              value: "{{ states('sensor.daily_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().weekday() == 1 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_tisdag
            data:
              value: "{{ states('sensor.daily_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().weekday() == 2 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_onsdag
            data:
              value: "{{ states('sensor.daily_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().weekday() == 3 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_torsdag
            data:
              value: "{{ states('sensor.daily_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().weekday() == 4 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_fredag
            data:
              value: "{{ states('sensor.daily_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().weekday() == 5 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_lordag
            data:
              value: "{{ states('sensor.daily_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().weekday() == 6 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_sondag
            data:
              value: "{{ states('sensor.daily_pv_generation') | float(0) }}"
mode: single
```
Daily — update current month
Runs at 23:50 each night and writes `sensor.monthly_pv_generation` to the current month helper. At month-end the final value is saved; the next month starts from the inverter's own monthly counter.
```yaml
alias: Uppdatera pågående månads solenergi
description: Updates the current month helper nightly with the running monthly total
triggers:
  - at: "23:50:00"
    trigger: time
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ now().month == 1 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_januari
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 2 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_februari
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 3 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_mars
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 4 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_april
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 5 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_maj
            data:
              value: >
                {{ (states('sensor.monthly_pv_generation') | float(0)) +
                   (states('input_number.solar_maj_offset') | float(0)) }}
      - conditions:
          - condition: template
            value_template: "{{ now().month == 6 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_juni
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 7 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_juli
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 8 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_augusti
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 9 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_september
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 10 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_oktober
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 11 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_november
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
      - conditions:
          - condition: template
            value_template: "{{ now().month == 12 }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.solar_december
            data:
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
mode: single
```
> **Note on `solar_maj_offset`:** The May 2026 automation includes an offset helper (`input_number.solar_maj_offset`) to compensate for production data that predates the automation being set up. Set this to `0` from June 2026 onwards by removing the offset line from the May action.
---
Installation
Install the koosoli ESPHome driver by placing `custom_components` in your ESPHome config directory
Copy `seeed-e1003-solar-dashboard.yaml` to your ESPHome config directory
Create all required `input_number` helpers in Home Assistant
Add the two automations in Home Assistant
Set your `secrets.yaml` with `wifi_ssid`, `wifi_password`, and `ota_password`
Flash the device via ESPHome dashboard
Set yearly production values manually in `input_number.solar_2023` etc.
---
Customisation
Timezone: Change `timezone: "Europe/Stockholm"` in the `time` block
Sensor names: Update entity IDs in the `sensor` block to match your inverter integration
Update interval: Currently 3600s (hourly). Change `update_interval` in the display block
Active hours: Currently 06:00–22:00. Adjust the condition `now.hour < 6 || now.hour >= 22`
VCOM value: The `vcom: 1400` value is specific to this display. Check your display's FPC cable for the correct value if yours differs
---
License
MIT
