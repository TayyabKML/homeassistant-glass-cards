# Cards

Each subdirectory is a self-contained card with its own README covering requirements,
setup and the things that will trip you up.

| Card | What it does |
|---|---|
| [`room-card`](room-card/) | Per-room overview tile: climate, status chips, occupancy, tap-through |
| [`room-description`](room-description/) | Room status bar for a view header: occupancy, time since, temperature |
| [`bubble-cards`](bubble-cards/) | Light, switch, media player and climate cards sharing one glass recipe |
| [`network-switch`](network-switch/) | Per-port status table for a managed switch, with a live up-count |
| [`camera-snapshot`](camera-snapshot/) | Detection snapshots with an age badge and freshness-reactive border |
| [`bambu-printer`](bambu-printer/) | 3D printer status: SVG progress ring, remaining time, chamber light, pause/resume |
| [`maintenance-tracker`](maintenance-tracker/) | Recurring household tasks with tap-to-stamp |
| [`system-stats`](system-stats/) | Host health tiles and 24-hour load graphs |

Each card is self-contained: the YAML you paste, plus a README covering what it needs,
how to install it, and which values to change.

All of them assume the design tokens in [`../theme/`](../theme/). Read that first — it's
short, and it's what keeps the dashboard coherent rather than a collection of unrelated
boxes.

Two conventions used throughout:

- Repeated cards are **Streamline templates**, defined once and instantiated with
  variables. One-off cards are plain manual cards you can paste directly.
- Entity IDs are always built by string interpolation from a predictable naming pattern.
  If your entities aren't named consistently, fix that first — it makes everything else
  simpler.
