# Seeed reTerminal E1003 — Solar Dashboard for Home Assistant

An ESPHome configuration for the [Seeed reTerminal E1003](https://www.seeedstudio.com/reTerminal-E1003-p-5811.html) 10.3-inch e-ink display, showing stylish solar production data from Home Assistant.

![Display photo](images/display.jpg)

---

## Based on

This project builds on the ESPHome driver by [koosoli](https://github.com/koosoli/Seeed-10.3-inch-IT8951-ESPHome-Drivers/tree/main/), which provides the custom `it8951_reterminal_e1003` display component. The driver handles low-level communication with the IT8951 e-ink controller. This repository adds a complete solar dashboard layout with Home Assistant sensor integration. Please note that the display is using Swedish language. I have kept this through the yaml, in order to compare code with image. Will translate relevant terms below.

---

## Features

- **Section 1 — Solar energy today (Solenergi idag):** Four boxes showing produced, exported, imported and total consumed energy
- **Section 2 — Last 6 days (Solenergi senaste veckan):** Rolling display with yesterday rightmost, updates daily
- **Section 3 — Monthly overview (Solenergi per månad):** All 12 months in two rows; months without data show `---`
- **Section 4 — Per year (Solenergi per år) + Lifetime totals (Solenergi totalt):** Current year auto-summed from monthly helpers; lifetime produced, self-consumed, imported and exported totals
- **Battery and WiFi status** icons in the top-right corner
- **Physical refresh button** (KEY0/GPIO3) triggers an immediate display update
- **HA button entity** for remote refresh from the Home Assistant dashboard
- **Night mode:** No display updates between 22:00 and 06:00 to save battery
- **Hourly updates** during daytime (06:00–22:00)
- **WiFi power save mode** enabled

---

## Hardware

- Seeed reTerminal E1003 (ESP32-S3, IT8951E/DX, 10.3" e-ink ED103TC2)
- Built-in 3000mAh Li-ion battery
- USB-C charging

---

## Requirements

### ESPHome driver

Clone or download the custom component from [koosoli's repository](https://github.com/koosoli/Seeed-10.3-inch-IT8951-ESPHome-Drivers/tree/main/) and place the `custom_components` folder in your ESPHome configuration directory.

### Home Assistant sensors

The following sensors must exist in your Home Assistant instance. These are typically provided by your inverter integration (e.g. Solis, SolarEdge, Huawei - I am using Sungrow, fetching data through modbus):

| Sensor | Description |
|---|---|
| `sensor.daily_pv_generation` | Solar energy produced today |
| `sensor.daily_exported_energy` | Solar energy exported to grid today |
| `sensor.daily_imported_energy` | Grid energy imported today |
| `sensor.daily_consumed_energy` | Total energy consumed today |
| `sensor.total_pv_generation` | Lifetime solar production |
| `sensor.total_imported_energy` | Lifetime grid import |
| `sensor.total_direct_energy_consumption` | Lifetime self-consumed solar |
| `sensor.total_exported_energy` | Lifetime solar export |
| `sensor.monthly_pv_generation` | Current month solar production (from inverter) |

### Home Assistant helpers

Create the following `input_number` helpers in HA under **Settings → Devices & Services → Helpers → Number**.

Set **minimum = -1** (used as "no data" sentinel displayed as `---`), **maximum = 99999**, **step = 0.1**, **unit = kWh**.

**Weekdays** (updated nightly by automation):

| Entity ID |
|---|
| `input_number.solar_mandag` |
| `input_number.solar_tisdag` |
| `input_number.solar_onsdag` |
| `input_number.solar_torsdag` |
| `input_number.solar_fredag` |
| `input_number.solar_lordag` |
| `input_number.solar_sondag` |

**Months** (updated nightly by automation):

| Entity ID |
|---|
| `input_number.solar_januari` |
| `input_number.solar_februari` |
| `input_number.solar_mars` |
| `input_number.solar_april` |
| `input_number.solar_maj` |
| `input_number.solar_juni` |
| `input_number.solar_juli` |
| `input_number.solar_augusti` |
| `input_number.solar_september` |
| `input_number.solar_oktober` |
| `input_number.solar_november` |
| `input_number.solar_december` |

**Years** (enter manually):

| Entity ID |
|---|
| `input_number.solar_2023` |
| `input_number.solar_2024` |
| `input_number.solar_2025` |

> Set helpers for days/months not yet started to `-1` so the display shows `---` instead of `0`.

---

## Automations

### Daily — update current weekday

Runs at 23:59 each night and writes `sensor.daily_pv_generation` to the correct weekday helper.

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

### Daily — update current month

Runs at 23:50 each night and writes `sensor.monthly_pv_generation` to the current month helper.

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
              value: "{{ states('sensor.monthly_pv_generation') | float(0) }}"
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

---

## Installation

1. Install the [koosoli ESPHome driver](https://github.com/koosoli/Seeed-10.3-inch-IT8951-ESPHome-Drivers/tree/main/) by placing `custom_components` in your ESPHome config directory
2. Copy `seeed-e1003-solar-dashboard.yaml` to your ESPHome config directory
3. Create all required `input_number` helpers in Home Assistant
4. Add the two automations in Home Assistant
5. Add your credentials to `secrets.yaml`: `wifi_ssid`, `wifi_password`, `ota_password`
6. Flash the device via the ESPHome dashboard
7. Enter historical yearly production values manually in `input_number.solar_2023` etc.

---

## Customisation

| Setting | Location | Default |
|---|---|---|
| Timezone | `time` block | `Europe/Stockholm` |
| Sensor entity IDs | `sensor` block | See table above |
| Update interval | `update_interval` in display block | `3600s` |
| Active hours | `now.hour < 6 \|\| now.hour >= 22` | 06:00–22:00 |
| VCOM value | `vcom` in display block | `1400` |

> **VCOM:** The value `1400` is specific to this unit. Check the label on your display's FPC cable if results look washed out or ghosted.

---

## License

MIT
