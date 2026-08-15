# Gotchas

Things that cost me hours. Written down so they cost you minutes.

## card_mod

**Put pseudo-elements on `ha-card`, not on mushroom's inner elements.** `ha-card::before`
and `ha-card::after` are where every card in this repo draws custom content, and they work
reliably. Attaching them to `mushroom-card` or `mushroom-state-item` instead is not a
pattern any working card here uses — if a pseudo-element seems to be ignored, check what
you attached it to before you check the CSS.

**Shadow-root selectors need the `$` suffix, and they do work.** `mushroom-shape-icon$`
and `mushroom-state-info$` pierce into mushroom's shadow DOM and are load-bearing across
these cards — resizing the icon disc, recolouring `--shape-color`, and turning
`mushroom-state-info`'s stacked layout into a justified row. Without the `$` the selector
matches nothing and fails silently, which is the usual cause of "styling mushroom's
internals doesn't work".

The `.` key is the companion: under `card_mod: style:` a plain string targets the card
itself, but the moment you need a shadow-root selector you must switch to the map form and
put the card's own CSS under `.`:

```yaml
card_mod:
  style:
    .: |
      ha-card { ... }
    mushroom-shape-icon$: |
      .shape { --shape-color: ...; }
```

**A Jinja error inside a card_mod style block discards the entire block.** Silently. You
get a completely unstyled card, no console error, no hint. If styling disappears entirely
after an edit, suspect a Jinja exception before you suspect a CSS mistake. Test the
template in **Developer Tools → Template** first — it'll show you the exception that
card_mod swallows.

**Verify card_mod works on the card type at all** before debugging your styles. Drop a
plain manual card with `border: 2px solid red` on it. If the border appears, card_mod is
wired up and the problem is your CSS or your Jinja. If it doesn't, the problem is
somewhere else entirely.

## Jinja

**`states('image.x')` returns a naive datetime; `now()` is timezone-aware.** So
`now() - as_datetime(states('image.x'))` raises `TypeError: can't subtract offset-naive and
offset-aware datetimes`. Use the entity object's `.last_changed`, which is aware:

```jinja
{{ (now() - states.image.doorbell_cam_person.last_changed).total_seconds() }}
```

The same applies to `input_datetime` helpers — those give you a naive local string, so
attach a tzinfo before subtracting.

**`input_text` helpers cap at 255 characters** and will fail on longer values. If you're
writing LLM output into one, state the limit in the prompt *and* truncate the result. The
model will overrun the limit eventually regardless of what you asked it.

## Lovelace layout

**Don't force `aspect_ratio: 16/9` on Frigate detection snapshots.** They're cropped to
the detected object and are usually taller than wide. With `fit_mode: cover` a 16:9 ratio
crops the subject straight out of frame.

**Only the outer card in a nested stack gets the glass treatment.** Applying
`backdrop-filter` to nested cards stacks the blur and produces progressively lighter,
muddier boxes. Inner cards should be transparent and borderless, separated by hairlines.

**A 4-column grid of tiles becomes unreadable past about a dozen items.** For anything
list-shaped — switch ports, device inventories — a table with one row per item is easier to
scan and takes less vertical space than you'd expect. 50px rows are comfortable.

## Scenes

**Never open a hand-edited `scenes.yaml` in the UI scene editor.** It strips every comment
in the file. If you've documented your tuning inline — and for anything involving colour
temperature and brightness curves you should — that documentation is gone the moment you
click edit. Keep hand-tuned scenes out of the UI entirely.

## Automations

**Notifications behind a TTS call arrive late.** If an automation announces something *and*
sends a phone notification, running them in sequence means the notification waits for the
speech to finish. Use a `parallel:` block so they fire together. For a doorbell that's the
difference between a useful alert and a pointless one.

**Flashing every light in a room is too aggressive.** Scope it to one or two fixtures.
Whole-room flashing reads as an emergency, not a doorbell.

**Disabled automations are worse than deleted ones.** A config with a third of its
automations disabled is a config you can't reason about. Decide, then act on the decision.
