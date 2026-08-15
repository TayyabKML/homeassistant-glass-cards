# Room card

A 150px overview tile per room: name, temperature and humidity, a column of status chips
for lights, media and a switch, and an occupancy bar along the bottom. The whole tile
tints and glows in the colour of whatever's active, and taps through to that room's view.

![placeholder — drop a screenshot here as screenshot.png]

Two-up in a `horizontal-stack` is the layout it's designed for.

## The thing that makes it worth using

**Every variable is optional, and the tile reshapes around what you give it.** Leave one
blank and that part simply isn't drawn — no gap, no `unknown`, no `unavailable`. So the
same template covers a living room with a full sensor stack:

```yaml
- type: custom:streamline-card
  template: room_card
  variables:
    name: <Room>
    navigation_path: /<dashboard>/<room>
    entity_id_temperature: sensor.<room>_temperature
    entity_id_humidity: sensor.<room>_humidity
    entity_id_lights: light.<room>_lights
    lights_icon: mdi:lightbulb-group-outline
    entity_id_media_player: media_player.<room>_tv
    entity_id_motion: binary_sensor.<room>_motion
    entity_id_switch: switch.<room>_desk
    switch_icon: mdi:desktop-tower
```

…and a spare room with nothing but a lamp:

```yaml
- type: custom:streamline-card
  template: room_card
  variables:
    name: <Room>
    navigation_path: /<dashboard>/<room>
    entity_id_lights: switch.<room>_lamp
    lights_icon: mdi:desk-lamp-on
```

Both render as a clean tile. That's the whole point — you don't need a second template for
rooms with less kit in them.

## Replace these

Nothing in the template body, with one exception to check.

| Placeholder | Where | Replace with |
|---|---|---|
| `weather.forecast_home` | `top_left` field, twice | Your weather entity, **if yours differs** |

Every other entity arrives as a variable, so all the wiring happens where you instantiate
the card.

`weather.forecast_home` is what Home Assistant creates when you accept the default Met.no
integration during onboarding, so on most installs it's already correct. It's read for the
temperature unit, and for outdoor conditions when you enable `weather`. Check it in
**Developer Tools → States** before assuming.

## Variables

| Variable | Does |
|---|---|
| `name` | Room label |
| `navigation_path` | View to open on tap |
| `entity_id_temperature` | Temperature, shown to one decimal |
| `entity_id_humidity` | Humidity, shown to one decimal |
| `entity_id_lights` | Light or switch — amber chip, amber room tint |
| `lights_icon` | Icon for that chip, default `mdi:lightbulb` |
| `entity_id_media_player` | Green chip and tint when `playing` |
| `entity_id_switch` | Blue chip and tint when `on` |
| `switch_icon` | Icon for that chip |
| `entity_id_motion` | Occupancy bar along the bottom |
| `weather` | Anything but `"false"` swaps the sensor line for outdoor conditions |
| `icon` | Accepted but unused; kept so existing cards don't need editing |

`entity_id_lights` takes a `switch.` entity perfectly happily — it only ever tests for
state `on`, so a lamp on a smart plug works without a light entity.

## Requirements

- `card-mod`
- `button-card`
- `streamline-card`

## Setup

1. Paste `room-card.yaml` under `streamline_templates:` in your dashboard's raw
   configuration editor.
2. Add one instance per room.

## How the tint works

Three entities are checked in priority order — lights, then media, then switch — and the
first one active picks the colour:

| Active | Colour |
|---|---|
| Lights on | Amber `255,181,71` |
| Media playing | Green `91,225,166` |
| Switch on | Blue `122,167,255` |
| Nothing | Neutral glass |

That colour then drives three separate `styles.card` properties: a radial gradient in the
top-right corner, the border, and the glow. Each one re-runs the same lookup because
button-card evaluates each property independently — there's nowhere to cache it. If you
add a fourth state, remember to change all three.

## Adapting it

- **Tile height** — `height: 150px` in `styles.card`. The three custom fields are
  absolutely positioned with a 58px right margin to clear the chip column, so if you make
  the tile much narrower, widen that gap first.
- **Occupancy wording** — the bottom bar says `Occupied` / `Clear` with a relative
  timestamp from `last_updated`. Both strings are in the `bottom_bar` field.
- **More chips** — `controls` builds them from a `chip(active, color, icon)` helper;
  adding a fourth is a one-line call, though the column starts to crowd past three.
