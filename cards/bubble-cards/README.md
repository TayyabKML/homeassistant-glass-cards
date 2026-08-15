# Bubble cards

Four entity cards — light, switch, media player, climate — sharing one glass recipe and a
consistent variable set, so a view built from them reads as one system rather than four
components that happen to sit together.

| File | Card type | Accent |
|---|---|---|
| `light-bubble-card.yaml` | `mushroom-light-card` | Amber, reactive |
| `switch-bubble-card.yaml` | `mushroom-template-card` | Your choice, reactive |
| `media-player-bubble-card.yaml` | `mushroom-media-player-card` | Purple, static |
| `climate-bubble-card.yaml` | `mushroom-climate-card` | Purple, static |

Take all four or just the one you need — they're independent files with no shared
dependency beyond the design tokens in [`theme/`](../../theme/).

"Bubble" describes the shape, not the component. These are built on
[mushroom](https://github.com/piitaya/lovelace-mushroom); you do **not** need the
[Bubble Card](https://github.com/Clooos/Bubble-Card) integration installed.

## Replace these

**Nothing.** Every entity arrives as a variable, so there's nothing in any of the four
template bodies to edit.

## Usage

All four take the same shape:

```yaml
- type: custom:streamline-card
  template: light_bubble_card
  variables:
    entity: light.<room>_lamp
    name: <Room> Lamp
    icon: mdi:desk-lamp-on
```

The switch card adds an accent colour:

```yaml
- type: custom:streamline-card
  template: switch_bubble_card
  variables:
    entity: switch.<room>_fan
    name: Fan
    icon: mdi:fan
    icon_color: "#5BE1A6"
```

## The one that will catch you out

**`icon_color` on the switch card must be a 6-digit hex, not a colour name.**

The card builds its border and glow by appending a two-digit alpha channel to whatever you
pass:

```
[[icon_color]]55   ->   #5BE1A655
[[icon_color]]40   ->   #5BE1A640
```

Pass `red` or `amber` and you get `red55`, which isn't a colour — so the border and glow
silently disappear. The icon itself *will* still tint, because that value is used raw, and
that combination is what makes this hard to diagnose: the card looks like it's working,
just flatter than the others.

Always quote it, too. Unquoted `#5BE1A6` starts a YAML comment.

## Which cards react to state

| Card | Reacts |
|---|---|
| Light | Border, glow and icon shift amber when on |
| Switch | Same, in your chosen accent |
| Media player | No — always purple |
| Climate | No — always purple |

The media and climate cards are deliberately static. Both spend most of their time idle,
and a card that only looks alive while active reads as broken the rest of the day.

## Ignored variables

Several variables are accepted and then ignored: `size`, `rows` and `button_type` on the
light and switch cards, `tint` on the switch card, and `rows` and `cover_background` on
the media player and climate cards.

They're harmless — pass them or don't. Listed here only so you don't spend time working
out why setting `rows: "2"` changes nothing.

## Requirements

- `card-mod`
- `mushroom`
- `streamline-card`

## Setup

1. Paste whichever cards you want under `streamline_templates:` in your dashboard's raw
   configuration editor.
2. Instantiate them in a view.

## Slider theming

Each card tints its slider through a different shadow-root selector. If you restyle one,
this is the mapping you need:

| Card | Selector |
|---|---|
| Light | `mushroom-light-brightness-control$` |
| Media player | `mushroom-media-player-volume-control$` |
| Climate | `mushroom-climate-temperature-control$` |

Inside each, the slider takes `--main-color` and `--bg-color` — except the light card,
which uses `--slider-color` and `--slider-bg-color`. That inconsistency is mushroom's, not
this repo's, and it's the usual reason a slider refuses to change colour.

The trailing `$` is card_mod's shadow-root piercing syntax and is required. Without it the
selector matches nothing and fails silently — see [`docs/gotchas.md`](../../docs/gotchas.md).

## Adapting it

- **Different accent on the light card** — amber `255,181,71` appears in the gradient,
  border, glow, icon colour and slider. Five places, all in `light-bubble-card.yaml`.
- **Colour-temperature or colour controls** — the light card sets
  `show_color_temp_control: false` and `show_color_control: false`. Turn either on and
  set `collapsible_controls: true` so the card doesn't grow permanently taller.
- **Fewer media controls** — trim the `media_controls` list. `volume_set` is what draws
  the slider; drop it and you're left with the buttons.
