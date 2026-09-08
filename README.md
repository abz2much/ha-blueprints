# Home Assistant Blueprints

A growing collection of Home Assistant automation blueprints, shared from
real, working setups. Each folder is one automation you can import directly
into your own Home Assistant instance and adapt to your own entities.

## Available blueprints

| Blueprint | Description | Import |
| --- | --- | --- |
| [Heating Sanity Check](./heating-sanity-check/) | Catches heating running when it probably shouldn't be (mild outside, window open), asks for approval before dropping the setpoint, announces if nobody responds. Two variants: rule-based (no dependencies) and AI-advisor (calls out to your own LLM backend). | See folder README |
| [Reload Integration When Entity Gets Stuck Unavailable](./eufy-security-reload/) | Built around Eufy Security's known WebSocket disconnects, reloads an entity's integration only when it's actually stuck unavailable, not on a blind timer, and notifies you if the reload didn't fix it. Works for any entity backed by a config entry. | See folder README |
| [Solar-Aware Energy Cluster](./solar-aware-energy-cluster/) | Seven solar-aware behaviours sharing cooldowns: appliance auto-start on high export, overnight fallback starts on a low forecast, a peak-rate appliance warning, a nightly forecast announcement, a high-export utilisation announcement and a direct heating boost. | See folder README |

More will be added here over time, requests welcome, see below.

## Requesting or suggesting a blueprint

Open an [issue](../../issues) describing the automation you'd like to see.
Include:

- What should trigger it
- What conditions decide whether it runs
- What it should actually do
- Any devices/integrations it depends on (so I know if it's generally
  reusable or specific to unusual hardware)

Not every request will get built, but they're all read.

## Using a blueprint from this repo

1. Open the blueprint's `.yaml` file on GitHub and copy the **raw** file URL
   (click "Raw", copy the address bar).
2. In Home Assistant: Settings → Automations & Scenes → Blueprints →
   Import Blueprint → paste the raw URL.
3. Create a new automation from the imported blueprint and fill in your own
   entities.

## Repo structure

```
ha-blueprints/
├── README.md              <- this file, the index
├── LICENSE
├── TEMPLATE/               <- copy this when adding a new blueprint
└── <blueprint-name>/
    ├── <blueprint-name>.yaml
    └── README.md           <- what it does, prerequisites, any variants
```

## License

MIT, see [LICENSE](./LICENSE). Use, adapt, and share freely, attribution
appreciated but not required.

---
Built and shared by [@automatedhome.ie](https://instagram.com/automatedhome.ie)
