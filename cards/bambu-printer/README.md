# Bambu Lab P1S card

Printer status: a circular progress ring, remaining time, nozzle/bed/layer chips, the
sliced file name, a cover-image thumbnail, and a chips row for the chamber light and
pause/resume.

![placeholder — drop a screenshot here as screenshot.png]

Built for a P1S, but nothing here is P1S-specific beyond the header text — any printer
integration exposing the same entities will work with the prefix swapped.

## Replace these

| Placeholder | Replace with |
|---|---|
| `<printer>` | Your printer's device prefix |

ha-bambulab names entities `sensor.<model>_<serial>_<thing>`, so yours looks like
`p1s_01p00c590900937` or `x1c_00m09a361800428`. Find it in **Developer Tools → States**.
One find-and-replace covers the JS constant and the direct entity references in the chips
row.

Entities it reads:

| Entity | Used for |
|---|---|
| `sensor.<printer>_print_progress` | Ring fill and the percentage |
| `sensor.<printer>_current_stage` | Status pill, dot colour, pause/resume visibility |
| `sensor.<printer>_remaining_time` | The large `1h 24m` readout |
| `sensor.<printer>_current_file` | Footer filename |
| `sensor.<printer>_nozzle_temperature` / `_bed_temperature` | Temperature chips |
| `sensor.<printer>_current_layer` / `_total_layer_count` | Layer chip |
| `image.<printer>_cover_image` | Thumbnail in the header |
| `light.<printer>_chamber_light` | Chips row toggle |
| `button.<printer>_pause_printing` / `_resume_printing` | Chips row actions |

## Requirements

- `card-mod`
- `button-card` — the entire layout is built in its `content` custom field
- `mushroom` — for the chips row
- A Bambu Lab integration such as [ha-bambulab](https://github.com/greghesp/ha-bambulab)

## Setup

1. Find your device prefix in **Developer Tools → States**.
2. Find-and-replace `<printer>` in `printer-card.yaml`.
3. Paste the card into a view.

## How it's built

The card is a `button-card` whose `content` field is a JavaScript template returning an
HTML string. That's unusual, and it's a deliberate trade: you get real layout control and
a single place to compute everything, at the cost of no Lovelace-level tap actions inside
the rendered markup. Interactive bits live in the separate chips row underneath, which is
why that row exists at all.

Everything downstream builds from one constant:

```js
const p = '<printer>';
const s = (id) => states[`sensor.${p}_${id}`]?.state ?? '';
```

### The progress ring

Inline SVG, rotated `-90deg` so progress starts at twelve o'clock rather than three. It
uses **`stroke-dashoffset`**, not a two-value dasharray:

```js
const R = 44, circ = +(2 * Math.PI * R).toFixed(2);
const offset = +(circ * (1 - progress / 100)).toFixed(2);
```

`stroke-dasharray` is set to the full circumference and the offset shortens the visible
arc. If you change the radius, `circ` recomputes itself — that's the reason it's derived
rather than hardcoded.

### Remaining time

`sensor.<printer>_remaining_time` comes back as a string like `"1h 24m"`, so the card
regex-matches it rather than treating it as a number:

```js
const m = rawTime.match(/(\d+)h\s*(\d+)m/);
```

Under an hour it renders just the minutes. If your integration reports remaining time as
an integer of minutes, this is the block to rewrite — it will otherwise show `--`.

### Stage colours

Three states drive the dot, the pill and the glow: printing is green `#22c55e`, paused is
amber `#f59e0b`, anything else is slate `#64748b`. The orange `#f97316` used for the ring
and temperatures is the printer accent and is independent of stage.

Note these are the card's own colours and don't come from [`theme/`](../../theme/) — see
the drift note there.

## Adapting it

- **Different printer model** — change the `BAMBU P1S` header string and the prefix.
- **No AMS / no cover image** — the thumbnail already degrades gracefully when
  `entity_picture` is absent. There's no AMS section in this card.
- **Pause/resume for a different integration** — the two conditional chips key off
  `current_stage` being exactly `printing` and `pause`. Check what your integration
  reports before assuming those strings match.
