# Zone Rules → Home Assistant Implementation Map

Brandon's Living Room and Upstairs HVAC rules, mapped to **enforceable YAML** vs **prompt-only**, with live entity IDs from your HA instance.

## Recommended entity wiring (per automation)

| Blueprint input | Living Room | Upstairs |
|-----------------|-------------|----------|
| Thermostat | `climate.lr_stat` | `climate.upstairs_stat` |
| Occupancy (home) | `input_boolean.anyone_home` | `input_boolean.anyone_home` |
| Zone occupancy (optional) | — | `binary_sensor.upstairs_occupancy_status` |
| Indoor temp | LR zone temp sensor | `sensor.upstairs_air_temperature` |
| Feels-like | `sensor.lr_thermal_comfort_heat_index` | `sensor.upstairs_thermal_comfort_heat_index` |
| Dew point | `sensor.lr_thermal_comfort_dew_point` | `sensor.upstairs_thermal_comfort_dew_point` |
| Outdoor temp | your outdoor temp entity | same |
| Weather | your forecast entity | same |

**Ezlo note:** “Auto” in your rules = HA mode `heat_cool` (not a separate `auto` mode).

---

## Rule layer model

| Layer | What it does |
|-------|----------------|
| **L1 – Template policy** | Mode, setpoint floors/ceilings, skip-if-comfortable, ECO — runs *before* AI |
| **L2 – AI prompt** | Judgment inside L1 bounds (forecast nuance, shoulder season) |
| **L3 – Post-parse clamp** | Enforces L1 on whatever AI returned — runs *before* `climate.set_*` |

Your pasted RULES blocks are **L2**. They worked in automatic mode because you tuned the model — **L1 + L3** make that behavior survive model drift.

---

## Living Room — rule-by-rule

### 1. Mode control

| Rule | Enforce in L1 (template) | Notes |
|------|--------------------------|-------|
| Occupied: Heat or Cool only | `required_mode` ∈ `{heat, cool}` | Block `change_mode` → `heat_cool` when `anyone_home` on |
| Summer outdoor >75°F → cool only | `if outdoor > 75: required_mode = cool` | Use outdoor temp sensor |
| Winter outdoor <55°F → heat only | `if outdoor < 55: required_mode = heat` | |
| Shoulder: AI picks | L2 prompt + forecast high | |
| Unoccupied ECO: Auto OK | `if not anyone_home: required_mode = heat_cool` | Ezlo `heat_cool` |

### 2. Mode switching logic

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Forecast high >70 → cool | Morning run (e.g. 8am trigger) sets mode | Add `input_datetime.ai_hvac_lr_last_mode_change` |
| Forecast high <60 → heat | same | |
| ≤1 mode switch / day | Compare `now().date()` to last helper date | Emergency override: indoor >86 or <58 |

### 3. Nighttime (after 11pm)

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Min setpoint 74°F | `eff_min_temp = max(min_temperature, 74)` when `hour >= 23` | “Never cooler” = floor |
| Less aggressive AI | L2: “prefer maintain unless feels-like outside 68–72” | |
| Keep mode unless extreme | L1: don’t allow `change_mode` at night unless emergency | |

### 4. Occupancy

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Anyone Home drives ECO | `eco = eco_enabled and is_state('input_boolean.anyone_home','off')` | Already matches blueprint if entity wired |
| Prefer occupied | Already implicit via Anyone Home aggregation | |

### 5. Thermal comfort

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Feels-like over dry-bulb | Pass `sensor.lr_thermal_comfort_heat_index` into context | |
| Dew point >65 → dehumidify priority | L1: if dew>65 and cooling, cap setpoint 72; L2 prompt | Outdoor dew now ~67°F — relevant |
| Target feels 68–72 | `skip_ai` if heat_index in band and idle | |

### 6. Decision framework

| Rule | Implementation |
|------|----------------|
| Morning forecast / mode | Separate trigger `time at 08:00` or first hourly after 8am |
| Occupied restore comfort | Trigger on `input_boolean.anyone_home` → `on` (optional automation) |
| Unoccupied ECO + Auto | L1 when `anyone_home` off |
| Emergency override | Template bypass when temp/feels-like outside safe band |

---

## Upstairs — rule-by-rule

### 1. Mode control

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Occupied: Heat or Cool only | same as LR | |
| Summer outdoor >**65**°F → cool | stricter than LR (75) | |
| Winter outdoor <**50**°F → heat | stricter than LR (55) | |
| Shoulder + cool bias | L2 prompt | |
| ECO: `heat_cool` | same as LR | |

### 2. Mode switching

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Forecast high >**65** → cool | upstairs thresholds | |
| Forecast high <**50** → heat | | |
| ≤1 switch / day | `input_datetime.ai_hvac_upstairs_last_mode_change` | |
| Prefer cool when uncertain | L2 only | |

### 3. Nighttime (after **10pm**)

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Max setpoint **71°F** | `eff_max_temp = min(max_temperature, 71)` when `hour >= 22` | Confirmed — never above 71°F at night |
| Sleep comfort over savings | L2 | |
| Stable cool at night | L1: block large downward changes after 22:00 | |

### 4. Occupancy

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Home = Anyone Home | `input_boolean.anyone_home` | |
| Bedroom / zone | `binary_sensor.upstairs_occupancy_status` **or** motion + lights | You have `upstairs_occupancy_status`, `upstairs_motion` |
| Night rules when occupied after 10pm | `hour >= 22 and zone_occupied` | |

### 5. Thermal comfort

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Feels-like | `sensor.upstairs_thermal_comfort_heat_index` | |
| Dew point >65 | `sensor.upstairs_thermal_comfort_dew_point` | |
| Daytime feels 68–74 | `eff` band in prompt + skip logic | |
| Nighttime feels 65–70 | stricter band when `hour >= 22` | |

### 6–7. Daytime logic

| Rule | Enforce in L1 | Notes |
|------|---------------|-------|
| Usually unoccupied weekdays | `zone_occupied = upstairs_occupancy_status` | |
| Weekend: check motion/lights | `now().weekday() in [5,6]` + motion/lux | Add lux entity if you have one |
| Day occupied spring/summer/fall → 72°F cool | month in Mar–Nov + zone_occupied → target 72 | |
| Unoccupied 72–74 | `eff_min=72, eff_max=74` when not zone_occupied | |

---

## RoomMind

**Disabled** — not used. AI HVAC targets **`climate.lr_stat`** and **`climate.upstairs_stat`** only.

---

## Helpers to add (once per zone)

```yaml
input_datetime:
  ai_hvac_lr_last_mode_change:
    name: AI HVAC LR Last Mode Change
    has_date: true
    has_time: true
  ai_hvac_upstairs_last_mode_change:
    name: AI HVAC Upstairs Last Mode Change
    has_date: true
    has_time: true
```

Update these whenever L1 executes `climate.set_hvac_mode`.

---

## Triggers to add (beyond hourly)

| Automation | Trigger | Purpose |
|--------------|---------|---------|
| Morning mode set | `time: 08:00` | Forecast-based mode for the day |
| Arrive home | `input_boolean.anyone_home` → `on` | Restore comfort mode/setpoint |
| Leave home | `input_boolean.anyone_home` → `off` | ECO + `heat_cool` |
| Upstairs night | `time: 22:00` + conditions | Cap cool setpoint (71°F) |

These can be separate small automations or blueprint `trigger` list additions.

---

## Blueprint status

Implemented in `ai_hvac_blueprint_unified.yaml` (see [README.md](README.md)):

- `zone_policy`, feels-like / dew-point / zone-occupancy inputs
- `input_boolean` + `binary_sensor` for home occupancy
- L1 policy, skip-before-AI, L3 safe clamp, zone rules in prompt
- Default automation mode: `automatic`

**Still manual / future:** once-per-day mode helpers, 8am forecast trigger, arrive/leave automations, full upstairs weekday logic in templates.

Re-import blueprint in HA and **Update from blueprint** on each zone automation when you pull repo changes.
