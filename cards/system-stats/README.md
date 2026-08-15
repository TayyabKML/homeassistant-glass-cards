# System stats card

Host health for the Home Assistant machine: six stat tiles over two 24-hour load graphs,
in a single glass shell.

The unavailable-entity count is the one nobody thinks to add. It's an early warning that
something has silently dropped off — a Zigbee device with a flat battery, an integration
that failed to reload after a restart.

## Replace these

**Nothing.** This is the only card in the repo with no placeholders, because every entity
it reads is created by an integration under a fixed name that's identical on any install:

| Entity | From |
|---|---|
| `sensor.system_monitor_disk_free` / `_disk_usage` | System Monitor |
| `sensor.system_monitor_processor_use` / `_processor_temperature` | System Monitor |
| `sensor.system_monitor_memory_usage` | System Monitor |
| `sensor.system_monitor_last_boot` | System Monitor |
| `sensor.backup_last_successful_automatic_backup` | Backup |

If yours are named differently they were likely created under an older Home Assistant
version — check **Developer Tools → States** and adjust.

## The tiles

| Tile | Green | Amber | Red | Tap |
|---|---|---|---|---|
| Disk Free | < 75% used | 75–85% | > 85% | — |
| CPU Temp | < 60 °C | 60–75 °C | > 75 °C | — |
| Uptime | — | — | — | — |
| Last Backup | < 2 days | 2–7 days | > 7 days | — |
| Updates | 0 pending | 1–3 | 4+ | `/config/updates` |
| Unavailable | < 45 | 45–60 | 60+ | `/config/entities` |

Adjust the CPU band to suit your hardware — alarming for one board is unremarkable for
another. The unavailable-entity thresholds are calibrated to a large install; if you have
a few dozen entities rather than a few hundred, bring 45/60 right down or the tile will
never leave green.

## Two implementation notes

**Two tiles have no sensor of their own.** Updates and Unavailable compute their values
inline from the entity registry:

```jinja
{{ states | selectattr('state','eq','unavailable')
   | rejectattr('domain','in',['button','update']) | list | count }}
```

`button` and `update` are rejected because both are routinely unavailable in normal
operation and would swamp the count. No template sensor is needed for this — resist the
urge to add one.

Because a mushroom-template-card still wants an `entity` for its tap target, both tiles
name `sensor.system_monitor_processor_use` as a placeholder entity. It's never read. That
looks like a mistake and isn't.

**Icon tint comes from two places.** `ha-state-icon { color: ... }` sets the glyph, and the
`mushroom-shape-icon$` shadow-root selector sets `--shape-color` for the disc behind it.
Change one without the other and the tile looks half-dressed.

## Requirements

- `card-mod`
- `stack-in-card`
- `mushroom`
- `mini-graph-card`
- The **System Monitor** integration — enable the processor, memory, disk and last-boot
  sensors
- The **Backup** integration for the Last Backup tile

## Setup

1. Enable System Monitor via **Settings → Devices & Services → Add Integration → System
   Monitor**, selecting processor, memory, disk and last boot.
2. Paste `system-card.yaml` into a view.
