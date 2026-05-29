# Example configurations

Copy these values into the **AI HVAC Advisor** blueprint when creating automations. All settings are per-zone — no helpers required.

---

## Living Room (example — warm climate home)

| Blueprint input | Example value |
|-----------------|---------------|
| Zone name | Living Room |
| Action ID prefix | `LR` |
| Thermostat | `climate.living_room` |
| Home occupancy | `input_boolean.anyone_home` or `binary_sensor.occupancy` |
| Feels-like | `sensor.living_room_heat_index` |
| Dew point | `sensor.living_room_dew_point` |
| Automation mode | AUTOMATIC |
| **Custom rules** | See block below |
| Summer outdoor threshold | `75` |
| Summer required mode | cool |
| Winter outdoor threshold | `55` |
| Winter required mode | heat |
| Forecast cool if high above | `70` |
| Forecast heat if high below | `60` |
| Uncertain mode | Keep current |
| Night start hour | `23` |
| Night min setpoint | `74` |
| Night max setpoint | `0` (disabled) |
| Comfort feels min / max | `68` / `72` |
| ECO cooling / heating offset | `5` / `5` |

**Custom rules (Living Room):**

```
- Occupied: use heat OR cool only (never auto/heat_cool).
- Summer: cool mode when outdoor above threshold.
- Winter: heat mode when outdoor below threshold.
- After 11pm: never set below 74°F; prefer maintain unless extreme.
- When away: ECO setbacks apply; heat_cool OK.
- Prefer feels-like temperature over dry-bulb for comfort decisions.
- High dew point: prioritize dehumidification / cooling.
```

---

## Upstairs / bedroom zone (example)

| Blueprint input | Example value |
|-----------------|---------------|
| Zone name | Upstairs |
| Action ID prefix | `UPSTAIRS` |
| Thermostat | `climate.upstairs` |
| Zone occupancy | `binary_sensor.upstairs_occupancy` |
| Feels-like | `sensor.upstairs_heat_index` |
| Automation mode | AUTOMATIC |
| Summer outdoor threshold | `65` |
| Winter outdoor threshold | `50` |
| Forecast cool if high above | `65` |
| Forecast heat if high below | `50` |
| Uncertain mode | Prefer cool |
| Night start hour | `22` |
| Night end hour | `6` |
| Night max setpoint | `71` |
| Comfort feels min / max | `68` / `74` |
| **Custom rules** | See block below |

**Custom rules (Upstairs):**

```
- Occupied: heat or cool only. Prefer cool in spring/summer/fall.
- After 10pm: never above 71°F — sleep comfort priority.
- Weekday daytime: often unoccupied; allow 72–74°F when empty.
- Weekend: use zone occupancy sensor (motion/lights) before aggressive cooling.
- When away: ECO + heat_cool for basic protection.
```

---

## Minimal / generic (new user)

Leave seasonal thresholds at **0** (disabled). Use default custom rules. Set only:

- Thermostat, indoor temp, AI agent, notification service, occupancy sensor
- Min/max temperature and max change per run
- Automation mode: **AUTOMATIC**

The AI will follow the default custom rules text; add your own thresholds as you tune the system.

---

## MONITORED mode (optional)

1. Set zone automation to **MONITORED**
2. Create **one** automation from **AI HVAC Notification Handler** blueprint
3. Same AI agent and notification service as your zones
4. No helpers — approve/reject/override use data embedded in the notification

Requires the **Home Assistant Companion app** on your phone.

---

## Thermostat mode names

Many integrations use `heat_cool` for “auto”. Map your vendor terminology in custom rules:

| Vendor term | Home Assistant mode |
|-------------|---------------------|
| Auto | `heat_cool` |
| Heat | `heat` |
| Cool | `cool` |
| Off | `off` |
