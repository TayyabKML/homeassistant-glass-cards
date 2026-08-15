# Network switch card

A table-style card for a managed switch: one row per port, showing the port number, what's
plugged in, the negotiated link speed, and a pulsing LED for link state. A live `up/16`
count sits in the header.

Sixteen ports in a single glass shell at 50px per row. Readable at a glance, and short
enough not to dominate a view.

![placeholder — drop a screenshot here as screenshot.png]

## Files

| File | What it is |
|---|---|
| `port-row.yaml` | Streamline template for one 50px row. This is the one `switch-card.yaml` uses. |
| `port-card.yaml` | Streamline template for the same data as a vertical tile, for grid layouts. |
| `switch-card.yaml` | The assembled card: glass shell, header with up-count, sixteen rows. |

## Replace these

| Placeholder | In | Replace with |
|---|---|---|
| `<switch>` | `switch-card.yaml` | Your switch's entity prefix |

That's the only substitution. **Neither template contains an entity ID** — `port_row` and
`port_card` take `status_entity`, `speed_entity` and `description_entity` as Streamline
variables, so all the wiring happens in `switch-card.yaml` where they're instantiated.

`[[port_number]]` and friends are Streamline variables, not placeholders. Leave them.

## Requirements

- `card-mod`
- `stack-in-card`
- `mushroom`
- `streamline-card`
- Three sensors per port from your switch integration, named consistently:
  `sensor.<switch>_port_<n>_status`, `_speed`, `_description`

## Setup

1. Paste `port-row.yaml` under `streamline_templates:` in your dashboard's raw
   configuration editor.
2. Paste `switch-card.yaml` into the view where you want it.
3. Find-and-replace `<switch>` with your entity prefix.

## The two things that will catch you out

**Link state is tested as the string `'1'`, not `'up'` or `'on'`.** That's what the source
switch integration reports. Every state test in both templates is
`is_state('[[status_entity]]', '1')` — there are four in `port_row` and four in
`port_card`. If your integration reports something else, change all of them or the card
will render every port as dead.

**Speed is formatted, not passed through.** `ha-card::before` reads the speed sensor as an
integer of Mbps and renders 1000-and-above as `1G`, anything lower as e.g. `100M`, and a
dead link as an em dash. If your sensor reports `"1 Gbps"` as a string rather than a
number, that `| int(0)` yields 0 and every live port shows `0M`.

## How the row is drawn

The port number and speed are a **single text run** in `ha-card::before`, pushed to
opposite ends with `text-align-last: justify`. The link LED is `ha-card::after`. The
mushroom card supplies only the icon and the description text; its internal layout is
flattened to a flex row via `mushroom-card` and `mushroom-state-item` rules, and the icon
is resized to 26px through the `mushroom-shape-icon$` shadow-root selector.

The `$` suffix is card_mod's shadow-root piercing syntax, and it is load-bearing here —
the icon cannot be resized without it.

## Adapting it

- **Different port count** — edit the `range(1, 17)` in the header loop, the `/16` in the
  header text, and add or remove instantiations. Past about 24 rows the card gets tall
  enough to want scrolling; two side-by-side stacks read better at that point.
- **Tile layout instead of rows** — swap `port_row` for `port_card` in the
  instantiations and put them in a grid. Note `port_card` carries its own darker glass
  shell, so drop the `stack-in-card` wrapper if you use it standalone.
- **Colours** — green `#00DDB8` for a live link, red `#FF4F72` for a dead one. These are
  the card's own values and differ from the shared palette in [`theme/`](../../theme/);
  see the note there about drift.
