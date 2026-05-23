# Home Assistant Blueprints for Energi Data Service

Automate devices based on the cheapest electricity hours using the [Energi Data Service](https://github.com/MTrab/energidataservice) integration.

## Prerequisites

- Home Assistant with the [Energi Data Service](https://github.com/MTrab/energidataservice) integration installed and configured
- HACS (Home Assistant Community Store)

---

## Installation

### Via HACS (recommended)

1. Open HACS → **Automations & Scenes** → **⋮** → **Custom repositories**
2. Add `https://github.com/runesoeknudsen/home-assistant-blueprints` as category **Blueprint**
3. Click **Explore & Download Blueprints**, search for "Energi Data Service" and download

### Direct import

| Blueprint | Import link |
|---|---|
| Cheapest Consecutive Hours | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Frunesoeknudsen%2Fhome-assistant-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fenergidata service_cheapest_hours_consecutive.yaml) |
| Cheapest Distributed Hours | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Frunesoeknudsen%2Fhome-assistant-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fenergidata service_cheapest_hours_distributed.yaml) |

---

## Available Blueprints

### 1. Cheapest Consecutive Hours

Activates a device during the X cheapest **consecutive** hours (must run back-to-back).

**Best for:** Water heaters, pool pumps, EV chargers — devices that benefit from longer uninterrupted operation.

**Example:** 4 cheapest consecutive hours are 13:00–16:00 → device turns on at 13:00, off at 17:00.

---

### 2. Cheapest Distributed Hours

Activates a device during the X cheapest hours **distributed freely** throughout the day.

**Best for:** Heat pumps, flexible loads — devices that can run at any time.

**Example:** 4 cheapest hours are 02:00, 06:00, 13:00 and 22:00 → device turns on/off individually at each hour.

---

## Configuration Options

Both blueprints share the same inputs:

| Input | Description | Default |
|---|---|---|
| **Price Sensor** | Your Energi Data Service sensor | `sensor.energi_data_service` |
| **Number of Hours** | How many cheap hours to activate (1–24) | 4 |
| **Action Target** | Device / scene / light to control | — |
| **Action Type** | `Turn On`, `Turn Off`, or `Toggle` | Turn On |
| **Enable Time Window** | Restrict search to a specific time range | Off |
| **Time Window Start** | Search window start (24h format) | 00:00 |
| **Time Window End** | Search window end (24h format) | 23:59 |
| **Override Mode** | `Auto`, `Force Always On`, or `Force Always Off` | Auto |

> **Note:** Time window start must be earlier than end (e.g. 06:00–22:00). Overnight windows spanning midnight (e.g. 22:00–06:00) are not supported.

---

## How It Works

The automation triggers **every hour at XX:00** and runs the following logic:

```
Override = Force On?  → turn_on
Override = Force Off? → turn_off
Current hour in cheapest N hours? → action_type (e.g. turn_on)
                               else → inverse action (e.g. turn_off)
```

This means the device is automatically turned **on at the start of each cheap hour** and **off at the start of each non-cheap hour** — no separate "turn off" automation needed.

---

## Visualising the Schedule

To see today's prices and expected on/off hours, paste this into **Developer Tools → Template**:

```jinja2
{% set price_sensor = 'sensor.energi_data_service' %}
{% set num_hours = 4 %}
{% set prices = state_attr(price_sensor, 'raw_today') or [] %}
{% set cheapest_hours = (prices | sort(attribute='price') | list)[:num_hours]
                      | map(attribute='hour.hour') | list %}

Hour  | Price (DKK) | Status
------|-------------|-------
{% for item in prices | sort(attribute='hour.hour') %}
{%- set h = item.hour.hour %}
{%- set bar = '█' * (item.price | float * 8) | int %}
{{ '%02d' | format(h) }}:00 | {{ '%6.3f' | format(item.price) }}     | {{ 'ON ✓  ' if h in cheapest_hours else 'off   ' }} {{ bar }}
{% endfor %}
ON hours: {{ cheapest_hours | sort }}
```

Change `num_hours` to match your automation. For a graphical bar chart, see the [ApexCharts card](#apexcharts-dashboard-card) section below.

### ApexCharts Dashboard Card

Requires [ApexCharts Card](https://github.com/RomRider/apexcharts-card) from HACS. Change `numHours` to match your automation.

```yaml
type: custom:apexcharts-card
header:
  show: true
  title: Electricity Price & Schedule
graph_span: 24h
span:
  start: day
apex_config:
  chart:
    type: bar
    height: 250
  plotOptions:
    bar:
      columnWidth: 90%
  xaxis:
    type: datetime
    labels:
      datetimeFormatter:
        hour: "HH:mm"
  yaxis:
    title:
      text: DKK/kWh
  tooltip:
    x:
      format: "HH:mm"
  legend:
    show: false
series:
  - entity: sensor.energi_data_service
    name: Price
    type: column
    data_generator: |
      const prices = entity.attributes.raw_today || [];
      const numHours = 4;
      const sorted = [...prices].sort((a, b) => a.price - b.price);
      const cheapSet = new Set(
        sorted.slice(0, numHours).map(p => new Date(p.hour).getHours())
      );
      return prices.map(item => {
        const dt = new Date(item.hour);
        return {
          x: dt.getTime(),
          y: parseFloat(item.price.toFixed(3)),
          fillColor: cheapSet.has(dt.getHours()) ? '#00b894' : '#e17055'
        };
      });
```

Green bars = device ON · Red bars = device OFF · Height = price

---

## Examples

### Water heater (consecutive, 4 hours)

| Setting | Value |
|---|---|
| Blueprint | Cheapest Consecutive Hours |
| Number of Hours | 4 |
| Action Target | `switch.water_heater` |
| Action Type | Turn On |

### Pool pump (distributed, 12 hours)

| Setting | Value |
|---|---|
| Blueprint | Cheapest Distributed Hours |
| Number of Hours | 12 |
| Action Target | `switch.pool_pump` |
| Action Type | Turn On |

### EV charger (distributed, daytime only)

| Setting | Value |
|---|---|
| Blueprint | Cheapest Distributed Hours |
| Number of Hours | 6 |
| Action Target | `switch.ev_charger` |
| Action Type | Turn On |
| Enable Time Window | Yes |
| Time Window Start | 06:00 |
| Time Window End | 22:00 |

### Using Override Mode

To temporarily keep a device off (e.g. during maintenance):

1. Open the automation → change **Override Mode** to `Force Always Off` → Save
2. Change back to `Auto` when done

---

## Troubleshooting

**Blueprint not showing after import**
- Reload blueprints: Developer Tools → YAML → Reload Automations
- Clear browser cache (Ctrl+F5)

**Device not turning on/off**
- Check that the entity ID is correct and the device responds to manual control
- Open the automation → **Traces** to see exactly which branch ran and what `in_cheap_hour` evaluated to
- Verify the price sensor has data: `{{ state_attr('sensor.energi_data_service', 'raw_today') | length }}`

**Time window not working**
- Make sure **Enable Time Window** is toggled ON in the blueprint UI
- Use 24-hour format (e.g. `06:00`, not `6:00 AM`)
- Start time must be before end time — overnight ranges are not supported

---

## Support

- Energi Data Service integration issues: https://github.com/MTrab/energidataservice
- Blueprint issues: open an issue in this repository

## License

See [LICENSE](LICENSE).
