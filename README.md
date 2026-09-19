# Temperature-Based Light Color

A Home Assistant blueprint that automatically changes the color of one or more lights according to the value of a temperature sensor.

The blueprint supports both:

* **RGB color interpolation** — smoothly transitions between two configurable colors.
* **Color-temperature interpolation** — smoothly transitions between two configurable Kelvin values.

For example, with a temperature range of `0 °C` to `30 °C`, you can configure:

* `0 °C` → blue
* `30 °C` → red

The blueprint calculates the intermediate color automatically.

## Features

* 🌡️ Configurable temperature sensor
* 💡 One or more configurable lights
* 🎨 RGB color interpolation
* 💡 Color-temperature interpolation in Kelvin
* 🌡️ Configurable minimum and maximum temperatures
* 🎨 Configurable colors for the minimum and maximum temperatures
* ⏱️ Configurable transition duration
* 🔄 Optional minimum temperature-change threshold
* 💡 Optional "only update lights that are already on" mode
* 🔆 Automatically synchronizes a light when it is turned on
* 🚀 Synchronizes lights after Home Assistant starts
* 🛡️ Handles unavailable and unknown temperature sensors
* 📈 Temperatures outside the configured range are clamped to the configured endpoint colors

## Installation

### Import directly into Home Assistant

Use the following URL to import the blueprint:

<https://raw.githubusercontent.com/tobus3000/hass-temperature-based-light-color-blueprint/main/blueprints/automation/tobus3000/temperature_based_light_color.yaml>

Alternatively, in Home Assistant:

1. Go to **Settings → Automations & scenes → Blueprints**.
2. Select **Import Blueprint**.
3. Enter the raw GitHub URL above.
4. Select **Preview**.
5. Import the blueprint.

### Manual installation

Copy:

```text
blueprints/automation/tobus3000/temperature_based_light_color.yaml
```

from this repository into the corresponding directory in your Home Assistant configuration:

```text
/config/blueprints/automation/tobus3000/temperature_based_light_color.yaml
```

After copying the file, reload automations or restart Home Assistant if necessary.

## Configuration

Create an automation from the **Temperature-based light color** blueprint.

### Temperature sensor

Select the temperature sensor that should control the lights.

The sensor should expose a numeric temperature state and use the `temperature` device class.

### Lights

Select one or more light entities.

The blueprint evaluates each selected light independently when deciding which lights should be updated.

### Color mode

Choose one of:

#### RGB color

The blueprint interpolates between two RGB colors.

For example:

```text
0 °C   → blue
15 °C  → purple
30 °C  → red
```

The interpolation is continuous, so intermediate temperatures produce intermediate colors.

#### Color temperature

The blueprint interpolates between two Kelvin values.

For example:

```text
0 °C   → 6500 K
15 °C  → 4600 K
30 °C  → 2700 K
```

Note that Kelvin works in the opposite direction from the temperature sensor:

* Higher Kelvin = cooler/bluer light
* Lower Kelvin = warmer/yellower light

Not every light supports color temperature. The selected lights must provide a compatible color mode.

## Temperature range

Configure:

* **Minimum temperature**
* **Maximum temperature**

Temperatures outside this range are clamped.

For example:

```text
Minimum: 0 °C
Maximum: 30 °C
```

means:

```text
-10 °C → minimum color
  0 °C → minimum color
 15 °C → interpolated color
 30 °C → maximum color
 40 °C → maximum color
```

The minimum temperature must be lower than the maximum temperature.

## Transition

The transition controls how quickly the light changes from its current color to the new color.

For example:

```text
0 seconds → immediate
2 seconds → smooth 2-second transition
10 seconds → slow 10-second transition
```

The actual transition behavior depends on whether the selected light supports transitions.

## Minimum temperature change

Temperature sensors can report very small changes frequently.

For example:

```text
20.01
20.02
20.01
20.03
20.04
```

If **Minimum temperature change** is set to `0.5 °C`, these small changes will not cause the lights to be updated.

Set it to `0` if every sensor state change should be processed.

## Only update lights that are already on

This option is enabled by default.

When enabled:

* Temperature changes only affect lights that are currently on.
* An off light is never switched on by this blueprint.
* When an individual light is subsequently switched on, its color is immediately synchronized with the current temperature.

Example:

```text
Light 1: ON
Light 2: OFF
Light 3: ON

Temperature changes
        ↓
Light 1 updated
Light 2 unchanged
Light 3 updated
```

Then:

```text
Light 2 switched ON
        ↓
Light 2 receives the current temperature color
```

This is generally the recommended configuration.

### Disabling this option

If the option is disabled, all configured lights are updated whenever the temperature changes.

Because Home Assistant uses `light.turn_on` to set the color, an off light may be turned on when this option is disabled.

## Startup behavior

When Home Assistant starts, the blueprint evaluates the current temperature and synchronizes the configured lights.

When "Only update lights that are already on" is enabled, only lights that are already on are changed.

This prevents Home Assistant startup from unexpectedly turning lights on.

## Example

A typical outdoor-temperature setup could use:

```text
Temperature sensor:
  sensor.outdoor_temperature

Lights:
  light.living_room
  light.dining_room

Color mode:
  RGB

Minimum temperature:
  0 °C

Maximum temperature:
  30 °C

Minimum color:
  Blue

Maximum color:
  Red

Transition:
  2 seconds

Minimum temperature change:
  0.5 °C

Only update lights that are already on:
  Enabled
```

This results in a continuously changing color that represents the current outdoor temperature whenever the selected lights are on.

## Limitations

### Light color capabilities

The blueprint sends either `rgb_color` or `color_temp_kelvin` to Home Assistant.

The selected lights therefore need to support the corresponding color mode.

RGB behavior depends on the capabilities and color representation of the individual light. Color-temperature behavior depends on the supported Kelvin range of the light.

### Multiple lights

Multiple lights can be selected. The blueprint determines which lights should be updated and sends the calculated color to those lights.

If different lights have substantially different color capabilities, their resulting colors may not look identical even though they receive the same requested color.

## Troubleshooting

### The light does not change color

Check that:

1. The temperature sensor has a numeric state.
2. The light supports the selected color mode.
3. The configured temperature range is valid.
4. The light is on when "Only update lights that are already on" is enabled.
5. The transition is supported by the light.

### The light turns on unexpectedly

Check **Only update lights that are already on**.

This should normally be enabled. If it is disabled, the blueprint is allowed to update all configured lights, including lights that are currently off.

### The color changes too frequently

Increase **Minimum temperature change**.

For example:

```text
0.1 °C → frequent updates
0.5 °C → moderate updates
1.0 °C → fewer updates
```

## License

See [LICENSE](LICENSE) for the license applicable to this project.

## Repository

Source code and issue tracking:

<https://github.com/tobus3000/hass-temperature-based-light-color-blueprint>
