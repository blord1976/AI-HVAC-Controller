# AI HVAC Advisor

Universal AI-powered climate automation for **any** Home Assistant installation. Configure your own rules per zone, pick any conversation agent, and run in **AUTOMATIC** mode with no helpers or input entities.

Install once via **HACS**, create automations from bundled blueprints, done.

[Examples for specific homes (including multi-zone setups)](EXAMPLES.md)

---

## What you get

- **Per-zone AI advisor** — one automation per thermostat
- **Your rules** — free-form custom rules text + numeric thresholds (season, night, comfort band)
- **Template safety** — required mode, setpoint clamp, max change, skip-when-comfortable (not prompt-only)
- **Any conversation agent** — Google Generative AI, OpenAI, Ollama, etc.
- **Any occupancy entity** — person, group, binary_sensor, input_boolean, device_tracker-derived
- **Optional MONITORED mode** — Approve / Reject / Override via Companion app (no helpers)
- **No helpers required** — zero `input_text` / `input_boolean` setup for core functionality

---

## Requirements

- Home Assistant **2024.6+**
- [HACS](https://hacs.xyz/)
- A configured **conversation agent** (Settings → Voice assistants)
- A **mobile notification** service if you want phone alerts (e.g. `notify.mobile_app_…`)

---

## Installation

### 1. HACS

1. HACS → **Integrations** → **⋮** → **Custom repositories**
2. Add `https://github.com/blord1976/AI-HVAC-Controller` → category **Integration**
3. Search **AI HVAC Advisor** → **Download**
4. **Restart Home Assistant**

No configuration flow — the integration registers bundled blueprints automatically.

### 2. Create zone automations

1. **Settings → Automations & Scenes → Blueprints**
2. Open **AI HVAC Advisor** (under `ai_hvac`)
3. **Create automation** — repeat for each thermostat
4. Required inputs:
   - Thermostat (`climate.*`)
   - Indoor temperature sensor
   - Conversation agent
   - Notification service
   - Home occupancy entity
5. Optional: outdoor temp, weather, feels-like, dew point, zone occupancy
6. Set **Custom zone rules** (multiline) — describe how *your* home should behave
7. Tune thresholds (set **0** to disable any seasonal/night rule you do not need)
8. **Automation mode:** **AUTOMATIC** (recommended)

Unique **Action ID prefix** per zone (e.g. `LR`, `BED`, `UPSTAIRS`).

### 3. Optional — MONITORED mode

Only if you want manual approval instead of auto-execute:

1. Set zone blueprint **Automation mode** to **MONITORED**
2. Blueprints → **AI HVAC Notification Handler** → create **one** automation
3. Select the same AI agent and notification service
4. Requires **Home Assistant Companion app** (notification actions pass thermostat data back)

No helpers. Approve/Reject/Override read `ai_hvac_*` fields embedded in the notification.

---

## Configuring your rules

Two layers work together:

| Layer | Where | Purpose |
|-------|--------|---------|
| **Custom rules** | Multiline blueprint input | Your preferences, schedules, comfort philosophy — sent to the AI |
| **Thresholds** | Numeric blueprint inputs | Enforced in templates before/after AI (mode, night min/max, comfort skip) |

### Threshold inputs (all optional — use 0 to disable)

| Input | Purpose |
|-------|---------|
| Summer / winter outdoor thresholds | Force cool or heat mode by outdoor temp |
| Forecast cool / heat thresholds | Force mode from today's forecast high |
| Uncertain mode | Default when no seasonal rule matches |
| Night start / end hour | When night setpoint limits apply |
| Night min / max setpoint | Floor or ceiling during night period |
| Comfort feels min / max | Skip AI call when in band and HVAC idle |
| Dew point high threshold | Flag humid conditions in AI prompt |
| ECO offsets | Setback when away |
| Min / max temp, max change | Hard safety limits |

See [EXAMPLES.md](EXAMPLES.md) for full Living Room / Upstairs examples you can copy.

---

## How it works

```
Hourly (or your interval) → Gather sensors → Compute policy (templates)
  → Comfortable & idle? → Skip AI
  → Else → Call conversation agent with your custom rules
  → Clamp result → AUTOMATIC: execute | MONITORED: notify
```

Logs: **Settings → System → Logs** or **Logbook** — search `AI HVAC`.

---

## Repository layout

```
custom_components/ai_hvac/          ← HACS integration
  manifest.json
  blueprints/automation/ai_hvac/
    ai_hvac_advisor.yaml            ← Main zone blueprint
    ai_hvac_notification_handler.yaml ← Optional MONITORED handler
EXAMPLES.md                         ← Sample configs (not defaults)
README.md
hacs.json
```

Legacy root-level YAML files are removed — use HACS blueprints only.

---

## Troubleshooting

| Issue | Check |
|-------|--------|
| Blueprint not listed | Restart HA after HACS install; look under `ai_hvac` domain |
| No AI response | Test conversation agent; check logs |
| No changes executed | AUTOMATIC mode? `maintain` in logs? Comfort skip? |
| MONITORED buttons fail | Notification handler automation created? Companion app? |
| Wrong mode on thermostat | Thermostat may use `heat_cool` for auto — say so in custom rules |
| Thresholds seem ignored | Values are in *your* unit scale (°F or °C label is prompt-only) |

After updating from GitHub: HACS → Redownload → Restart → **Update from blueprint** on each automation.

---

## Contributing

[Issues & PRs](https://github.com/blord1976/AI-HVAC-Controller/issues)

---

## License

MIT — Brandon Lord
