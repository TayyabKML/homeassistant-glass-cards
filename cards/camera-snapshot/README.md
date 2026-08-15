# Camera snapshot card

A Streamline template for detection snapshot images, with a corner badge showing how long
ago the detection fired and a border that reacts to how fresh it is.

![A detection snapshot with a 5h age badge in the top-right corner and a Garden • Bird caption bar across the bottom](screenshot.png)

The snapshot content is blurred in this example image only — the card doesn't blur. The
filters it does apply are dimming and greyscale as a snapshot ages, described below.

## What it does

- A badge in the top-right corner counts the age of the snapshot: `NOW` under a minute,
  then `5m`, `2h`, `3d`.
- Under five minutes old, the card gets a green border and a soft glow — new detections
  pull your eye without any layout change.
- Over an hour it dims slightly. Over a day it goes greyscale at half opacity, so a stale
  camera looks obviously stale rather than looking like nothing happened.
- The caption is picture-entity's own footer, restyled with a blur.

## Replace these

| Placeholder | In | Replace with |
|---|---|---|
| `<camera>` | usage block | The camera's name as configured in Frigate |

The template body needs no edits — `entity` and `name` are both Streamline variables. The
placeholders live only in the usage block at the bottom of the file.

## Requirements

- `card-mod`
- `streamline-card`
- An integration providing snapshot `image.*` entities, e.g. Frigate

## Setup

1. Paste `snapshot-card.yaml` under `streamline_templates:` in your dashboard's raw
   configuration editor.
2. Add one instance per camera/object pair (see the bottom of that file).

## Two things that will waste your afternoon

**Don't force a 16:9 aspect ratio.** Detection snapshots are cropped to the detected
object, so they're usually *taller than they are wide*. Combined with `fit_mode: cover` a
16:9 ratio crops the subject straight out of the frame — you get a card showing someone's
midriff. The template deliberately sets no `aspect_ratio` and lets the image size itself.

**Use `.last_changed`, not `states()`, for the timestamp.** `states('image.x')` returns a
**naive** local datetime while `now()` is timezone-aware, so `now() - as_datetime(state)`
raises a `TypeError`. The template takes the entity object and uses `.last_changed`, which
is aware:

```jinja
{% set s = states['[[entity]]'] %}
{% set age = (now() - s.last_changed).total_seconds() if s else 9999999 %}
```

The `if s else 9999999` matters too — during a restart the entity may not exist yet, and
without the guard the whole style block raises and you get an unstyled card.

## Adapting it

- **Freshness thresholds** — `fresh` is 300 seconds, `stale` is 86400. Both are set as
  Jinja variables at the top of the style block.
- **Badge position** — it's `ha-card::after` at `top: 8px; right: 8px`. Move it to
  `bottom` if you'd rather it sat over the caption.
- **Corner radius** — 16px here rather than the standard 22px; media reads better
  tighter. See [`theme/`](../../theme/).
