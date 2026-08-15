# Glassmorphic Lovelace Cards for Home Assistant

A set of reusable Lovelace dashboard cards built with `card-mod`, `mushroom` and
`streamline-card` — translucent glass surfaces, heavy backdrop blur, per-room accent
colours and state-reactive glows.

Every card is parameterised. Point it at your own entities and it works; there are no
entity IDs from my house baked in, and each card's README lists exactly which values you
need to change.

This is deliberately **not** a config dump — no automations, no floor plan, no entity
registry. Just the parts that are genuinely reusable.

The shared design tokens live in [`theme/`](theme/), which is what keeps the cards looking
like one system rather than eight unrelated boxes.

## What's in here

| Path | What it is |
|---|---|
| [`cards/room-card`](cards/room-card/) | Room overview tile — climate, light/media/switch chips, occupancy bar, taps through to the room's view |
| [`cards/room-description`](cards/room-description/) | Room status bar for a view header — occupancy pill, time since, temperature |
| [`cards/bubble-cards`](cards/bubble-cards/) | Light, switch, media player and climate entity cards sharing one glass recipe and variable set |
| [`cards/network-switch`](cards/network-switch/) | Table-style card showing per-port status, speed and link activity for a managed switch |
| [`cards/camera-snapshot`](cards/camera-snapshot/) | Streamline template for detection snapshots, with an age badge and a border that reacts to freshness |
| [`cards/bambu-printer`](cards/bambu-printer/) | Bambu Lab P1S card — SVG progress ring, remaining time, temperatures, chamber light, pause/resume |
| [`cards/maintenance-tracker`](cards/maintenance-tracker/) | "Last done / days since" tracker cards with tap-to-stamp |
| [`cards/system-stats`](cards/system-stats/) | Host system card — disk, CPU temp, uptime, backups, load graphs |
| [`theme/`](theme/) | Shared design tokens: colours, radii, blur values, glass surface recipe |
| [`templates/`](templates/) | Jinja templates, including an LLM house-state summary prompt |
| [`docs/`](docs/) | Hard-won notes: card_mod limitations, Frigate/go2rtc/Scrypted, Jinja traps |

## Requirements

All installed via [HACS](https://hacs.xyz/):

| Component | Needed for |
|---|---|
| [`card-mod`](https://github.com/thomasloven/lovelace-card-mod) | The glass treatment. Used by every card here. |
| [`streamline-card`](https://github.com/brunosabot/streamline-card) | Card templating — define once, instantiate many |
| [`mushroom`](https://github.com/piitaya/lovelace-mushroom) | Most cards |
| [`button-card`](https://github.com/custom-cards/button-card) | Room card, printer card |
| [`stack-in-card`](https://github.com/custom-cards/stack-in-card) | Network switch, system stats |
| [`mini-graph-card`](https://github.com/kalkih/mini-graph-card) | System stats graphs |

Each card's README lists the specific subset it needs — nothing requires all six.

Note the cards in [`cards/bubble-cards`](cards/bubble-cards/) are named for their shape,
not for the [Bubble Card](https://github.com/Clooos/Bubble-Card) component. They're built
on mushroom, and you don't need Bubble Card installed.

## How to use these

Cards are written as [Streamline](https://github.com/brunosabot/streamline-card) templates
wherever they're repeated. Streamline templates are defined once under
`streamline_templates:` in your dashboard's raw YAML editor, then instantiated with
different variables:

```yaml
streamline_templates:
  room_card:
    card:
      type: custom:button-card
      # ... template body, referring to [[variables]]

views:
  - cards:
      - type: custom:streamline-card
        template: room_card
        variables:
          name: Kitchen
          entity_id_temperature: sensor.kitchen_temperature
          entity_id_lights: light.kitchen_lights
```

Where a card isn't repeated, it's a plain manual card you can paste straight into a view.

Placeholders you need to replace are written in angle brackets — `sensor.<switch>_port_1_status`
— and every card README has a **Replace these** table listing them. A card with no
placeholders says so.

## Licence

MIT — see [LICENSE](LICENSE). Use them, change them, no attribution needed.
