# Jinja templates

## `house-summary-prompt.jinja`

Builds a compact natural-language prompt describing the current state of the house —
who's home, presence sensors, what's playing, lights and power, indoor and outdoor
temperatures — for feeding to an LLM.

The output is written into an `input_text` helper and rendered on the dashboard, giving a
one-line "what's going on right now" summary instead of a wall of tiles.

### Replace these

| Placeholder | Replace with |
|---|---|
| `<indoor_temp>` | An indoor temperature sensor |
| `<outdoor_temp>` | An outdoor temperature sensor |
| `<house_power>` | Whole-house power draw, in watts |
| `<house_summary>` | The `input_text` helper you're writing the summary into |

Everything else iterates over domains and device classes rather than naming entities, so
it works unmodified on any install. That's deliberate — the sections that matter most
(who's home, what's on, what's open) shouldn't need a per-house entity list.

### The 255-character trap

`input_text` helpers are capped at **255 characters**. An LLM will cheerfully return
400 and the `set_value` call will fail, sometimes silently. Two defences, both worth
having:

1. Tell the model the limit explicitly in the prompt (this template does).
2. Truncate on the way in anyway: `{{ response | truncate(250, true, '…') }}`.

Never trust the first without the second.

### Update cadence

Every 30 minutes is a sensible default. Every state change would hammer whatever API is
behind it for no perceptible benefit — nobody reads the summary that often.

### Usage sketch

```yaml
automation:
  - alias: House summary
    trigger:
      - platform: time_pattern
        minutes: "/30"
    action:
      - service: conversation.process   # or your LLM integration of choice
        data:
          text: !include house-summary-prompt.jinja
        response_variable: result
      - service: input_text.set_value
        target:
          entity_id: input_text.<house_summary>
        data:
          value: "{{ result.response.speech.plain.speech | truncate(250, true, '…') }}"
```

Adjust entity names to match your setup — this template is the shape, not a drop-in.
