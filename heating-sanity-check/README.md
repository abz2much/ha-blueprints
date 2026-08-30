# Heating Sanity Check

Catches heating running when it probably shouldn't be, mild outside, or a
window open, and asks before touching anything. Two variants, pick whichever
suits your setup.

## Variants

| File | Dependencies | How it decides |
| --- | --- | --- |
| [`heating_sanity_check_rule_based.yaml`](./heating_sanity_check_rule_based.yaml) | None | Local rule: warm outside OR a window open = flag it. Everyone can use this one. |
| [`heating_sanity_check_ai.yaml`](./heating_sanity_check_ai.yaml) | A `rest_command` you build yourself, pointing at an advisor endpoint (e.g. n8n calling an LLM) | Calls your advisor for a judgement call, only when the same cheap pre-filter (warm outside or window open) is met |

Start with the rule-based one. Move to the AI version later if you want more
nuanced reasoning (e.g. weighing forecast trend, how long the window's been
open, whether anyone's home) than a flat threshold gives you.

## What both variants do

1. Trigger when a thermostat starts actively heating (debounced 2 minutes to
   ignore brief cycling).
2. Decide whether the heating looks wrong (locally, or via your advisor).
3. If flagged, notify up to two people with Approve/Dismiss actions.
4. If nobody responds within a timeout, fall back to a spoken announcement.
5. A cooldown (via an `input_datetime` helper) stops it re-asking on every
   heating cycle.

## Before you import (rule-based version)

1. An `input_datetime` helper for the cooldown timestamp (Settings > Devices
   & Services > Helpers > Create Helper > Date and/or time). One per
   instance of the blueprint.
2. `mobile_app` set up for anyone who should get the approve/dismiss push.

That's it, no external services.

## Before you import (AI advisor version)

Everything above, plus a `rest_command` in your `configuration.yaml` that
POSTs to your own advisor endpoint:

```yaml
rest_command:
  heating_sanity_check:
    url: "https://your-advisor-endpoint.example.com/heating-check"
    method: POST
    headers:
      Content-Type: application/json
    payload: >
      {
        "zone_label": "{{ zone_label }}",
        "current_temp": {{ current_temp }},
        "target_temp": {{ target_temp }},
        "outdoor_temp": {{ outdoor_temp }},
        "weather_condition": "{{ weather_condition }}",
        "window_open": {{ window_open }}
      }
    timeout: 15
```

Your endpoint must return JSON shaped like:

```json
{ "decision": "TURN_OFF", "reason": "It's 17°C outside and a window's open." }
```

or:

```json
{ "decision": "OK", "reason": "Cold enough outside, nothing wrong here." }
```

`decision` is the only field checked (`TURN_OFF` vs anything else). `reason`
is shown in the notification and spoken in the fallback announcement.

## Importing

1. Copy the raw GitHub URL for whichever `.yaml` file you want.
2. Settings → Automations & Scenes → Blueprints → Import Blueprint → paste
   the URL.
3. Create an automation from the blueprint, fill in your entities and notify
   targets.
