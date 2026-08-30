# Reload Integration When Entity Gets Stuck Unavailable

Watches an entity and reloads its integration only when it's actually stuck,
not on a blind timer. Built around a recurring Eufy Security camera problem,
but it works for any entity backed by a config entry.

## Why this exists (the Eufy problem)

The Eufy Security integration for Home Assistant
([fuatakgun/eufy_security](https://github.com/fuatakgun/eufy_security))
doesn't talk to an official local API. It goes through a companion add-on,
[eufy-security-ws](https://github.com/bropat/eufy-security-ws), which holds
a persistent WebSocket connection to Eufy's own backend on the integration's
behalf. That connection is known to drop on its own, and when it does,
camera and sensor entities sit "unavailable" (or stop updating while
reporting a stale state) until something reloads the integration.

This isn't a one-off bug. It's been reported against the integration
repeatedly, across several years and Home Assistant versions:

- [fuatakgun/eufy_security#1402](https://github.com/fuatakgun/eufy_security/issues/1402)
  (November 2025, open at time of writing): motion and person sensors stop
  updating entirely after a WebSocket disconnect logged as `code: 1000
  reason: Normal Closure`, even though the add-on still reports a
  successful connection. Confirmed reproducible across multiple integration
  versions by the reporter.
- The long-running [Eufy Security Integration community
  thread](https://community.home-assistant.io/t/eufy-security-integration/318353)
  on the Home Assistant forum has users reporting cameras or the Homebase
  entity going unavailable and staying that way until the add-on and
  integration are manually reloaded, a pattern that shows up across the
  thread's multi-year history rather than one bad release.

Note: this is a connection-stability issue in the unofficial WebSocket
bridge, unrelated to the separate 2024 Eufy cloud video-privacy story that
got press coverage. Different problem entirely, don't conflate the two.

The original version of this automation (not published here) worked around
this by reloading the integration on a blind 6-hour timer. That caused its
own problem: it sometimes reloaded a perfectly healthy integration and
interrupted an in-progress stream or recording. This blueprint only reloads
when the entity is actually unavailable or stuck outside a healthy state,
so a working camera is never touched.

## What it does

1. Triggers when the watched entity goes "unavailable", or sits outside a
   list of healthy states for longer than a threshold you set.
2. Reloads that entity's integration config entry.
3. Waits a configurable delay for the integration to come back.
4. If the entity still isn't healthy, sends a notification instead of
   reloading again or leaving you to notice yourself. It does not retry the
   reload in a loop, if the first attempt doesn't fix it, that's a "go look
   at this" signal, not a "keep hammering it" one.

The "stuck" trigger watches for the main state value to stop changing, not
for the entity to go completely silent. It's set up so an unrelated
attribute ticking over in the background (a timestamp, a counter, anything
an integration updates even while otherwise broken) doesn't quietly reset
the stuck-timer forever and stop this from ever firing.

## Before you import

Nothing beyond stock Home Assistant for the reload/notify logic. If you want
the failure notification, `mobile_app` (or any other `notify` target) needs
to be set up.

## Importing

1. Copy the raw GitHub URL for `reload_entity_when_unavailable.yaml`.
2. Settings → Automations & Scenes → Blueprints → Import Blueprint → paste
   the URL.
3. Create an automation from the blueprint, pick the entity to watch, and
   set a notify target if you want one.

## Adapting it beyond Eufy

The "healthy states" input is the only thing that's camera-shaped by
default (`idle,streaming,recording`). Point this at any entity backed by a
config entry, weather stations, other camera brands, flaky Zigbee/Z-Wave
devices, and set `healthy_states` to whatever that entity's normal states
actually are.
