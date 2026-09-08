# Solar-Aware Energy Cluster

Seven solar-aware behaviours in one blueprint, sharing cooldowns and
once-a-day gates so they don't fight each other or spam you.

## What it does

1. **Appliance auto-start** — if grid export stays above a threshold for a
   few minutes, starts the washer and/or dishwasher (once a day) if they're
   loaded and ready.
2. **Nightly forecast announcement** — at a set time, notifies (and
   optionally announces) a summary of tomorrow's forecasted solar
   production.
3/4. **Overnight fallback starts** — if tomorrow's forecast is low, and the
   washer/dishwasher are still loaded and ready by a set time, starts them
   after a delay (so today's solar gets first refusal).
5. **Peak-rate appliance warning** — if the washer, dryer or dishwasher
   starts while the electricity price and grid import are both high and
   someone's home, sends a warning.
6. **High export utilisation announcement** — if export stays high for a
   minute or so and someone's home, announces it (and whether the
   washer/dishwasher are ready to go), optionally nudging a heating boost if
   it's on the cool side.
7. **Direct heating boost** — if export/generation stays high for a while,
   boosts heating directly (independent of #6, own cooldown).

All seven run from one automation in `parallel` mode so they don't block
each other.

## Before you import

- **Three `input_datetime` helpers**, one each for: appliance auto-start
  (once-a-day gate), high-export announcements (cooldown) and heating boost
  (cooldown). Settings → Devices & Services → Helpers → Create Helper →
  Date and/or time. One set per instance of this blueprint.
- **`select` entities** for your washer/dishwasher with a "run" (or your
  own) option value, if you want the auto-start and fallback-start
  behaviours.
- Everything else — the announcement script, heating-boost script and
  forecast rest_command — is optional. Leave the defaults ("none") and
  those parts of the automation quietly skip themselves.

## Importing

1. Copy the raw GitHub URL for `solar_aware_energy_cluster.yaml`.
2. Settings → Automations & Scenes → Blueprints → Import Blueprint → paste
   the URL.
3. Create an automation from the blueprint and fill in your entities.
   Thresholds, timing and cooldowns are grouped into collapsed sections
   with sensible defaults — expand them if you want to tune anything.
