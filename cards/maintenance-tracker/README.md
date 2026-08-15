# Maintenance tracker card

A "last done / days since" card for recurring household jobs. Tap it to stamp the task as
done today; the border and glow shift from green through amber to red as the task comes
due.

![placeholder — drop a screenshot here as screenshot.png]

## How it works

Three pieces per task:

1. An `input_datetime` helper storing the date it was last done.
2. A `script` that sets that helper to today.
3. A card reading the helper and calling the script on tap.

Splitting the stamp into a script rather than calling `input_datetime.set_datetime`
straight from the card means the same action is reachable from an automation, a voice
command or an NFC tag.

It's a plain manual card, not a Streamline template — paste it once per task.

## Replace these

Same `<task>` value across all three files, or they won't find each other.

| Placeholder | In | Replace with |
|---|---|---|
| `<task>` | all three files | snake_case task name, e.g. `filter_clean` |
| `<Task>` | `helpers.yaml`, `scripts.yaml`, card | Display label, e.g. `Filter Clean` |
| `<icon>` | card | mdi icon name, without the `mdi:` prefix |
| `<amber_days>` | card | Days after which it goes amber |
| `<red_days>` | card | Days after which it goes red |

The naming pattern is `input_datetime.last_<task>` and `script.set_<task>_today`.

### Suggested thresholds

Intervals vary a lot by task, and getting them wrong makes the card useless — everything
either permanently green or permanently red. Some starting points:

| Task | `<task>` | Icon | Amber | Red |
|---|---|---|---|---|
| Filter clean | `filter_clean` | `filter` | 60 | 100 |
| Fish tank water change | `fish_tank_water_change` | `water-sync` | 10 | 21 |
| Plants watered | `plants_water` | `sprout` | 7 | 14 |

Set amber at roughly the point you'd want a nudge, and red at the point it's genuinely
overdue.

## Requirements

- `card-mod`
- `mushroom`

## Setup

1. Create an `input_datetime` helper per task — either via **Settings → Devices &
   Services → Helpers**, or with the YAML in `helpers.yaml`.
2. Add a stamp script per task from `scripts.yaml`.
3. Paste `maintenance-card.yaml` into a view, once per task, substituting the
   placeholders each time.

## Things worth knowing

**The helpers are date-only.** `has_time: false`, because these tasks are tracked to the
day and the card only ever compares dates. The stamp script sets `date` alone to match —
if you turn time on, the script has to supply both halves or the call fails.

**Thresholds appear twice per card.** Once in `icon_color`, once in the `card_mod` glow
block. Change both, or the icon and the border will disagree with each other.

**A task that's never been stamped reads red.** The card treats a missing value as 999
days and shows `Never recorded`, which is usually what you want on a fresh install.

**`action:` vs `service:` in the script.** `action:` is the current key for calling a
service. On Home Assistant 2024.7 and older, use `service:` instead.

## Adapting it

- **Reminders** — the state is a datetime helper, so a template sensor or automation can
  notify you when something goes overdue. Not included here; that's config, not a card.
- **Stamping from elsewhere** — because the stamp is a script, an NFC tag on the fish tank
  or a voice command can call the same service the card does.
