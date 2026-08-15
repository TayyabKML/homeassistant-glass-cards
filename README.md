# Home Assistant — Reusable Cards & Tooling

A collection of reusable Lovelace card templates, Jinja templates and helper tooling
extracted from my Home Assistant setup.

This is deliberately **not** a config dump. There are no entity IDs from my house, no
secrets, no automations and no floor plan — just the parts that are genuinely reusable,
parameterised so you can point them at your own entities.

Everything here targets a dark, glassmorphic dashboard: translucent surfaces, heavy
backdrop blur, per-room accent colours and state-reactive glows. The shared design tokens
live in [`theme/`](theme/) so cards stay visually consistent.

## What's in here

| Path | What it is |
|---|---|
| [`cards/network-switch`](cards/network-switch/) | Table-style card showing per-port status, speed and link activity for a managed switch |
| [`cards/camera-snapshot`](cards/camera-snapshot/) | Streamline template for detection snapshots, with an age badge and a border that reacts to freshness |
| [`cards/bambu-printer`](cards/bambu-printer/) | Bambu Lab P1S card — SVG progress ring, remaining time, temperatures, chamber light, pause/resume |
| [`cards/maintenance-tracker`](cards/maintenance-tracker/) | "Last done / days since" tracker cards with tap-to-stamp |
| [`cards/system-stats`](cards/system-stats/) | Host system card — disk, CPU temp, uptime, backups, load graphs |
| [`theme/`](theme/) | Shared design tokens: colours, radii, blur values, glass surface recipe |
| [`templates/`](templates/) | Jinja templates, including an LLM house-state summary prompt |
| [`docs/`](docs/) | Hard-won notes: card_mod limitations, Frigate/go2rtc/Scrypted, Jinja traps |

## Requirements

Most cards depend on custom components installed via [HACS](https://hacs.xyz/):

- [`card-mod`](https://github.com/thomasloven/lovelace-card-mod) — CSS injection, used everywhere
- [`button-card`](https://github.com/custom-cards/button-card)
- [`mushroom`](https://github.com/piitaya/lovelace-mushroom)
- [`bubble-card`](https://github.com/Clooos/Bubble-Card)
- [`streamline-card`](https://github.com/brunosabot/streamline-card) — card templating
- [`stack-in-card`](https://github.com/custom-cards/stack-in-card)
- [`mini-graph-card`](https://github.com/kalkih/mini-graph-card)

Each card's README lists the specific subset it needs.

## How to use these

Cards are written as [Streamline](https://github.com/brunosabot/streamline-card) templates
wherever they're repeated. Streamline templates are defined once under
`streamline_templates:` in your dashboard's raw YAML editor, then instantiated with
different variables:

```yaml
streamline_templates:
  port_card:
    card:
      type: custom:mushroom-template-card
      # ... template body, using [[variables]]

views:
  - cards:
      - type: custom:streamline-card
        template: port_card
        variables:
          port: 1
```

Where a card isn't repeated, it's given as a plain manual card you can paste directly.

## Licence

MIT — see [LICENSE](LICENSE). Use them, change them, no attribution needed.
