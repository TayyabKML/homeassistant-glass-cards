# Design tokens

The shared visual language. Keeping these values in one place is what stops a dashboard
drifting into six slightly different shades of grey.

> **Read this before copying values out.** These are the tokens the dashboard is
> *designed* around, and every outer card shell follows them. Several individual cards
> don't — see [Where cards deviate](#where-cards-deviate) at the bottom. When a card's
> own values differ from this page, the card is the source of truth for that card, and
> this page is the source of truth for anything new you write.

## Palette

| Token | Value | Used for |
|---|---|---|
| Background | `#0f1117` | Dashboard background |
| Glass surface | `rgba(255, 255, 255, 0.04)` | Card fill |
| Hairline | `rgba(255, 255, 255, 0.08)` | Row separators, card borders |
| Text primary | `rgba(255, 255, 255, 0.92)` | Values, headings |
| Text secondary | `rgba(255, 255, 255, 0.55)` | Labels, units |
| Green | `#5BE1A6` | Healthy / on / connected |
| Amber | `#FFB547` | Warning / attention / living room accent |
| Red | `#FF5E7A` | Fault / offline / critical |
| Blue | `#7AA7FF` | Information / office accent |

Accent colours are assigned **per room** as well as per state — the living room reads
amber, the office blue. That's what makes a glance at the dashboard tell you *where* as
well as *what*.

## Surface recipe

The glass treatment, applied consistently:

```yaml
card_mod:
  style: |
    ha-card {
      background: rgba(255, 255, 255, 0.04);
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: 22px;
      backdrop-filter: blur(22px) saturate(160%);
      -webkit-backdrop-filter: blur(22px) saturate(160%);
      box-shadow: none;
    }
```

| Property | Value |
|---|---|
| Corner radius | `22px` outer shells, `20px` media cards, `14px` inner elements |
| Blur | `blur(22px) saturate(160%)` |
| Border | `1px solid rgba(255, 255, 255, 0.08)` |
| Row height | `50–52px` for table-style rows |

`saturate(160%)` is the part that matters. Blur alone produces flat grey; boosting
saturation lets whatever is behind the card bleed through as colour, which is what makes it
read as glass rather than frosted plastic.

## State-reactive glow

Rather than only changing an icon colour, cards emit a soft glow in the state colour. It's
readable from across a room:

```yaml
card_mod:
  style: |
    ha-card {
      {% if is_state(config.entity, 'on') %}
      box-shadow: 0 0 24px -6px #FFB547;
      border-color: rgba(255, 181, 71, 0.35);
      {% endif %}
      transition: box-shadow 400ms ease, border-color 400ms ease;
    }
```

Keep the transition. Without it, states snap and the dashboard feels mechanical.

## Nesting

Where several rows share one glass shell, only the **outer** card gets the treatment.
Inner cards are made transparent and borderless, separated by hairlines:

```yaml
ha-card {
  background: none;
  border: none;
  box-shadow: none;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
}
```

Applying the glass recipe to nested cards stacks the blur and produces muddy,
progressively lighter boxes.

## Where cards deviate

The tokens above describe the outer glass shells, which are consistent. Individual cards
inside those shells have drifted, and it's worth knowing before you assume a value:

| Card | Deviates by |
|---|---|
| [`network-switch`](../cards/network-switch/) | Link states use `#00DDB8` green and `#FF4F72` red, not the palette's `#5BE1A6` / `#FF5E7A`. `port_card` also carries a darker, more opaque shell: `rgba(11,14,24,0.85)`, `blur(28px) saturate(200%)`, 18px radius. |
| [`camera-snapshot`](../cards/camera-snapshot/) | 16px radius rather than 22px, and `#00DDB8` for the freshness badge. |
| [`bambu-printer`](../cards/bambu-printer/) | An entirely separate orange accent ramp — `#f97316`, `#fb923c`, `#fdba74` — plus its own stage colours (`#22c55e`, `#f59e0b`, `#64748b`). None of these are in the palette. |
| [`maintenance-tracker`](../cards/maintenance-tracker/) | 20px radius. Uses palette colours, but as raw RGB triples (`'91,225,166'`) so they can be interpolated into `rgba()` for the glow. |

Two greens and two reds in circulation is the main thing to fix if you ever consolidate.
`#5BE1A6` / `#00DDB8` and `#FF5E7A` / `#FF4F72` are close enough to look like a rendering
difference rather than a deliberate choice, which is the worst kind of inconsistency.
