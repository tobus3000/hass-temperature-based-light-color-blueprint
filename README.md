# Temperature-Based Light Color

A Home Assistant blueprint that automatically changes the color of one or more lights according to the value of a temperature sensor.

The blueprint supports both:

* **RGB color interpolation** — smoothly transitions through three configurable colors.
* **Color-temperature interpolation** — smoothly transitions through three configurable Kelvin values.

The temperature range is divided into two phases:

```text
Cold ──────────► Normal ──────────► Hot
       Phase 1                 Phase 2
```

For example, with:

```text
Minimum temperature: 0 °C
Normal temperature: 20 °C
Maximum temperature: 30 °C

Cold color:   Blue
Normal color: Green
Hot color:    Red
```

the resulting color curve is:

```text
Blue ──────────► Green ──────────► Red
  0 °C             20 °C            30 °C
                   ▲
                 Normal
```

The intermediate colors are calculated automatically. With blue → green → red, the transition naturally passes through colors such as cyan, yellow, and orange.

---

## Features

* 🌡️ Configurable temperature sensor
* 💡 One or more configurable lights
* 🎨 RGB color interpolation
* 💡 Color-temperature interpolation in Kelvin
* 🌡️ Configurable minimum, normal, and maximum temperatures
* 🎨 Configurable cold, normal, and hot colors
* ⏱️ Configurable transition duration
* 🔄 Optional minimum temperature-change threshold
* 💡 Optional "only update lights that are already on" mode
* 🔆 Automatically synchronizes a light when it is turned on
* 🚀 Synchronizes lights after Home Assistant starts
* 🛡️ Handles unavailable and unknown temperature sensors
* 📈 Temperatures outside the configured range are clamped to the cold or hot color
* 🔀 Uses separate interpolation phases for cold → normal and normal → hot

---

## How the temperature curve works

The blueprint uses three configurable temperature points:

1. **Minimum / Cold**
2. **Normal**
3. **Maximum / Hot**

The temperature range is split into two independent linear interpolation phases.

### Phase 1 — Cold to Normal

From the minimum temperature up to the normal temperature, the color is interpolated between the configured cold and normal colors.

### Phase 2 — Normal to Hot

From the normal temperature up to the maximum temperature, the color is interpolated between the configured normal and hot colors.

This makes the normal temperature an actual color anchor rather than simply a point somewhere along a single cold-to-hot gradient.

### Example

With the default RGB colors:

```text
Cold:   Blue
Normal: Green
Hot:    Red
```

and:

```text
Minimum: 0 °C
Normal:  20 °C
Maximum: 30 °C
```

the curve behaves approximately like this:

| Temperature | Result |
|-------------|--------|
| ≤ 0 °C | Cold color: blue |
| 5 °C | Blue → cyan |
| 10 °C | Intermediate blue/green |
| 15 °C | Greenward transition |
| 20 °C | Normal color: green |
| 22 °C | Green → yellow |
| 25 °C | Yellow → orange |
| 28 °C | Orange → red |
| ≥ 30 °C | Hot color: red |

The exact intermediate colors depend on the three colors selected by the user.

The intermediate colors are calculated mathematically rather than hard-coded, so any three RGB colors can be used.

---

## Installation

### Import directly into Home Assistant

Use the following URL to import the blueprint:

<https://raw.githubusercontent.com/tobus3000/hass-temperature-based-light-color-blueprint/refs/heads/main/blueprints/automation/tobus3000/temperature_based_light_color.yaml>

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

---

## Configuration

Create an automation from the **Temperature-based light color** blueprint.

### Temperature sensor

Select the temperature sensor that should control the lights.

The sensor should expose a numeric temperature state and use the `temperature` device class.

---

## Lights

Select one or more light entities.

When multiple lights are selected, the blueprint determines which lights should be updated based on their current state and the **Only update lights that are already on** setting.

---

## Color mode

Choose one of:

* **RGB color**
* **Color temperature**

### RGB color

RGB mode interpolates through three configurable RGB colors:

```text
Cold → Normal → Hot
```

For example:

```text
0 °C   → Blue
20 °C  → Green
30 °C  → Red
```

The resulting gradient is:

```text
Blue → Cyan → Green → Yellow → Orange → Red
```

The exact colors depend on the three configured RGB values.

The normal color is guaranteed to be reached at the configured normal temperature.

### Color temperature

Color-temperature mode interpolates through three configurable Kelvin values:

```text
Cold → Normal → Hot
```

For example:

```text
0 °C   → 6500 K
20 °C  → 4000 K
30 °C  → 2700 K
```

The Kelvin value is interpolated separately across the two temperature phases.

Remember that Kelvin behaves opposite to the usual visual temperature scale:

* Higher Kelvin = cooler/bluer light
* Lower Kelvin = warmer/yellower light

Not every light supports color temperature. The selected lights must provide a compatible color mode and supported Kelvin range.

---

## Temperature range

Configure three temperature points:

* **Minimum temperature**
* **Normal temperature**
* **Maximum temperature**

The following relationship must be true:

```text
Minimum < Normal < Maximum
```

For example:

```text
Minimum: 0 °C
Normal:  20 °C
Maximum: 30 °C
```

The resulting behavior is:

```text
≤ 0 °C     → Cold color
0–20 °C    → Cold → Normal interpolation
20 °C      → Normal color
20–30 °C   → Normal → Hot interpolation
≥ 30 °C    → Hot color
```

Temperatures outside the configured range are clamped to the corresponding endpoint color.

For example:

```text
-10 °C → Cold color
  0 °C → Cold color
 10 °C → Interpolated Cold → Normal color
 20 °C → Normal color
 25 °C → Interpolated Normal → Hot color
 30 °C → Hot color
 40 °C → Hot color
```

---

## Cold, Normal and Hot colors

### Cold color

The color used at or below the configured minimum temperature.

For example:

```text
Blue
```

### Normal color

The color used exactly at the configured normal temperature.

For example:

```text
Green
```

The normal color is the transition point between the two interpolation phases.

### Hot color

The color used at or above the configured maximum temperature.

For example:

```text
Red
```

---

## Transition

The **Transition** setting controls how quickly the light changes from its current color to the newly calculated color.

For example:

```text
0 seconds  → immediate
2 seconds  → smooth 2-second transition
10 seconds → slow 10-second transition
```

The actual transition behavior depends on whether the selected light supports transitions.

The transition duration applies whenever the blueprint updates a light.

---

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

For example:

```text
Minimum temperature change: 0.5 °C

20.0 → 20.2 °C   ignored
20.2 → 20.4 °C   ignored
20.4 → 21.0 °C   update
```

Set it to `0` if every sensor state change should be processed.

This setting can be useful for reducing unnecessary light updates when a temperature sensor reports frequent small fluctuations.

---

## Only update lights that are already on

This option is enabled by default.

When enabled:

* Temperature changes only affect lights that are currently on.
* An off light is never switched on by this blueprint.
* When an individual light is subsequently switched on, its color is synchronized with the current temperature.

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

This allows the blueprint to control the color of active lights without unexpectedly turning other lights on.

### Disabling this option

If the option is disabled, all configured lights are updated whenever the blueprint runs.

Because the blueprint uses `light.turn_on` to apply the requested color, an off light may be turned on when this option is disabled.

---

## Startup behavior

When Home Assistant starts, the blueprint evaluates the current temperature and synchronizes the configured lights.

When **Only update lights that are already on** is enabled, only lights that are already on are changed.

This prevents Home Assistant startup from unexpectedly turning lights on.

If a configured light is off during startup, it remains off. When it is subsequently turned on, the blueprint applies the color corresponding to the current temperature.

---

## Example configuration

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

Normal temperature:
  20 °C

Maximum temperature:
  30 °C

Cold color:
  Blue

Normal color:
  Green

Hot color:
  Red

Transition:
  2 seconds

Minimum temperature change:
  0.5 °C

Only update lights that are already on:
  Enabled
```

This produces a temperature-dependent color scale approximately like:

```text
Cold                         Normal                         Hot
 │                              │                            │
 ▼                              ▼                            ▼
Blue ─── Cyan ─── Green ─── Yellow ─── Orange ─── Red
 0 °C                           20 °C                        30 °C
```

The selected lights continuously represent the current temperature whenever they are on.

---

## Limitations

### Light color capabilities

The blueprint sends either:

```text
rgb_color
```

or:

```text
color_temp_kelvin
```

to Home Assistant.

The selected lights therefore need to support the corresponding color mode.

RGB behavior depends on the capabilities and color representation of the individual light.

Color-temperature behavior depends on the supported Kelvin range of the light.

### Multiple lights

Multiple lights can be selected.

The blueprint calculates one color based on the current temperature and applies that requested color to the selected lights that are eligible for an update.

If different lights have substantially different color capabilities, their resulting colors may not look identical even though they receive the same requested color.

### Color temperature support

Color-temperature mode should only be used with lights that support color temperature.

If a light does not support the requested color mode, Home Assistant or the light integration may reject or ignore the color command.

### Temperature sensor values

The temperature sensor should provide a numeric state.

`unknown` and `unavailable` sensor states are ignored.

---

## Troubleshooting

### The light does not change color

Check that:

1. The temperature sensor has a numeric state.
2. The temperature sensor uses the `temperature` device class.
3. The light supports the selected color mode.
4. **Minimum temperature < Normal temperature < Maximum temperature**.
5. The light is on when **Only update lights that are already on** is enabled.
6. The configured minimum temperature change is not too large.
7. The transition is supported by the light.

### The light turns on unexpectedly

Check **Only update lights that are already on**.

This should normally be enabled if the blueprint should only control lights that are already active.

If it is disabled, the blueprint is allowed to update all configured lights, including lights that are currently off.

### The color changes too frequently

Increase **Minimum temperature change**.

For example:

```text
0.1 °C → frequent updates
0.5 °C → moderate updates
1.0 °C → fewer updates
```

The appropriate value depends on how frequently the temperature sensor changes and how quickly the light should respond.

### The normal color is never reached

Check the configured temperatures.

The relationship must be:

```text
Minimum < Normal < Maximum
```

For example:

```text
Minimum: 0 °C
Normal:  20 °C
Maximum: 30 °C
```

At exactly `20 °C`, the configured normal color should be used.

### The color does not look like the expected gradient

RGB interpolation is performed independently for the red, green, and blue channels.

The actual intermediate colors therefore depend on the three selected RGB colors.

For example, selecting:

```text
Cold:   Blue
Normal: Green
Hot:    Red
```

produces a different gradient from:

```text
Cold:   Blue
Normal: White
Hot:    Red
```

The blueprint does not use a fixed list of named intermediate colors.

---

## License

See [LICENSE](LICENSE) for the license applicable to this project.

## Repository

Source code and issue tracking:

<https://github.com/tobus3000/hass-temperature-based-light-color-blueprint>
