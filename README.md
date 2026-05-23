# Home Assistant Blueprints for Energi Data Service

This repository contains Home Assistant blueprints for automating devices based on the cheapest electricity hours from the [Energi Data Service](https://github.com/MTrab/energidataservice) integration.

## Installation

### Prerequisites

- Home Assistant with the [Energi Data Service](https://github.com/MTrab/energidataservice) integration installed and configured
- HACS (Home Assistant Community Store) installed

### Install via HACS

1. Open HACS in your Home Assistant
2. Click **Automations & Scenes** → **⋮** → **Custom repositories**
3. Add this repository:
   - **URL**: `https://github.com/YOUR_USERNAME/home-assistant-blueprints`
4. Click **Explore & Download Blueprints**
5. Search for "Energi Data Service" and download the blueprints you want

## Available Blueprints

### 1. Cheapest Consecutive Hours

Activates a device during X consecutive cheapest hours (must be in sequence).

**Use Case:** Water heaters, pool pumps, or other devices that benefit from longer continuous operation during cheap hours.

**Example:** If the 4 cheapest consecutive hours are 14:00-17:00, the device will turn on only during this continuous period.

**Configuration:**
- **Price Sensor**: Your Energi Data Service sensor (default: `sensor.energi_data_service`)
- **Number of Hours**: How many consecutive hours (1-24)
- **Action Target**: The device/scene/light to control
- **Action Type**: Turn On, Turn Off, or Toggle
- **Time Window (Optional)**: Restrict search to specific times (e.g., 06:00-22:00)
- **Override Mode**: 
  - `Auto` (default) - Uses the cheapest hours logic
  - `Force Always On` - Ignore logic, always turn on
  - `Force Always Off` - Ignore logic, always turn off

### 2. Cheapest Distributed Hours

Activates a device during X cheapest hours distributed throughout the day (non-consecutive).

**Use Case:** EV chargers, heat pumps, or flexible loads that can operate at any time during the day.

**Example:** If the 4 cheapest hours are 02:00, 06:00, 14:00, and 22:00, the device will turn on at each of these individual hours.

**Configuration:**
- **Price Sensor**: Your Energi Data Service sensor (default: `sensor.energi_data_service`)
- **Number of Hours**: How many of the cheapest hours (1-24)
- **Action Target**: The device/scene/light to control
- **Action Type**: Turn On, Turn Off, or Toggle
- **Time Window (Optional)**: Restrict search to specific times
- **Override Mode**: 
  - `Auto` (default) - Uses the cheapest hours logic
  - `Force Always On` - Ignore logic, always turn on
  - `Force Always Off` - Ignore logic, always turn off

## How It Works

- **Trigger**: Every hour at minute 0 (XX:00)
- **Override Mode Check**: If override mode is set, it takes precedence:
  - `Force Always On` → Always turn on
  - `Force Always Off` → Always turn off
  - `Auto` → Continue with normal logic below
- **Normal Logic** (when in Auto mode):
  - **Condition**: Checks if the current hour is within the cheapest hours
  - **Action**: 
    - If condition is TRUE → Executes your chosen action (e.g., turn on)
    - If condition is FALSE → Executes the opposite action (e.g., turn off)

## Examples

### Example 1: Water Heater (Consecutive)

1. Create a new automation from the "Cheapest Consecutive Hours" blueprint
2. Configure:
   - **Number of Hours**: 4
   - **Action Target**: `switch.water_heater`
   - **Action Type**: Turn On
3. Save

Result: Your water heater will turn on during the 4 consecutive cheapest hours and turn off automatically after.

### Example 2: EV Charger (Distributed, Night Only)

1. Create a new automation from the "Cheapest Distributed Hours" blueprint
2. Configure:
   - **Number of Hours**: 6
   - **Action Target**: `switch.ev_charger`
   - **Action Type**: Turn On
   - **Enable Time Window**: Yes
   - **Time Window Start**: 22:00
   - **Time Window End**: 06:00
3. Save

Result: Your EV charger will charge for 6 total hours distributed throughout the night's cheapest times (between 22:00-06:00).

### Example 3: Using Override Mode

You can temporarily override the automatic logic for manual control:

**Scenario:** Your water heater needs maintenance, so you want to keep it off even during cheap hours.

1. Open the automation you created
2. Change **Override Mode** from `Auto` to `Force Always Off`
3. Save

The device will now stay off regardless of prices. Switch it back to `Auto` when done with maintenance.

## Troubleshooting

**Blueprint not showing in Home Assistant:**
- Ensure the Energi Data Service integration is installed and configured
- Clear your browser cache (Ctrl+F5 or Cmd+Shift+R)
- Restart Home Assistant

**Device not turning on/off:**
- Verify the entity ID exists and is working (test by manually controlling it)
- Check Home Assistant logs for any template errors
- Ensure the price sensor has valid data (check sensor attributes)

**Time window not working:**
- Make sure to enable the time window toggle in the blueprint configuration
- Use 24-hour format (e.g., 22:00, not 10:00 PM)

## Support

For issues related to the Energi Data Service integration itself, visit: https://github.com/MTrab/energidataservice

For blueprint issues, please open an issue in this repository.

## License

These blueprints are provided as-is. See the LICENSE file for details.
