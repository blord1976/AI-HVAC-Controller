# AI HVAC Advisor - Universal Blueprint

AI-powered HVAC optimization using Gemini AI with occupancy-based ECO mode and flexible automation modes.

## 🎯 Features

- 🤖 **AI-Powered** - Gemini AI analyzes conditions and suggests/executes HVAC changes
- 🏠 **Multi-Thermostat** - Deploy once per thermostat for complete home coverage
- 🌿 **ECO Mode** - Automatic setbacks when unoccupied (configurable offsets)
- ⚙️ **Three Automation Modes:**
  - **OFF**: Completely disabled
  - **AUTOMATIC**: AI executes changes with notifications (recommended for most users)
  - **MONITORED**: Requires approval via notification buttons (advanced setup)
- 🛡️ **Safety Bounds** - Temperature limits, max change limits
- 📊 **Full Transparency** - All decisions logged with AI reasoning

## 🚀 Quick Start (10 minutes per thermostat)

### Step 1: Import Blueprint

**Import URL:**
```
https://raw.githubusercontent.com/blord1976/ai-hvac-blueprint/main/ai_hvac_blueprint_unified.yaml
```

**How to import:**
1. Go to **Settings → Automations & Scenes → Blueprints**
2. Click **Import Blueprint**
3. Paste the URL above
4. Click **Preview** → **Import Blueprint**

### Step 2: Create Automation from Blueprint

1. **Settings → Automations → Create Automation**
2. Select **AI HVAC Advisor**
3. Configure:

**Required Entities:**
- **Thermostat**: Your climate entity (e.g., `climate.living_room`)
- **Indoor Temp Sensor**: Temperature sensor for this zone
- **AI Agent**: `conversation.google_generative_ai` (or your AI agent)
- **Notification Service**: `notify.mobile_app_your_phone`
- **Occupancy Sensor**: Binary sensor for home/away

**Automation Mode:**
- **AUTOMATIC** (recommended): AI executes changes, sends notification after
- **MONITORED** (advanced): Requires approval - see Advanced Setup below

**Zone Configuration:**
- **Zone Name**: Display name (e.g., "Living Room")
- **Action ID Prefix**: Unique ID for this zone (e.g., "LR", "MASTER")

**ECO Mode:**
- **Enabled**: Turn ON for automatic setbacks when away
- **Cooling Offset**: +5°F warmer when away (adjust as needed)
- **Heating Offset**: -5°F cooler when away

**Safety Limits:**
- Min/Max Temperature: 60-85°F (adjust for your comfort)
- Max Change: 4°F per hour

4. **Save** automation

### Step 3: Test

**Automatic Mode:**
1. Wait for next hour OR manually run the automation
2. If AI suggests a change, it executes immediately
3. You receive a notification showing what happened
4. Check **Settings → System → Logs** → search "AI HVAC"

**You're done!** 🎉

---

## 📱 For Multiple Thermostats

**Living Room + Upstairs example:**

1. Create automation from blueprint for Living Room:
   - Zone Name: `Living Room`
   - Action ID: `LR`
   - Thermostat: `climate.living_room`

2. Create another automation from same blueprint for Upstairs:
   - Zone Name: `Upstairs`
   - Action ID: `UPSTAIRS`
   - Thermostat: `climate.upstairs`

Each zone operates independently!

---

## 🌿 ECO Mode

**How it works:**

**When Home Occupied** (normal mode):
- AI optimizes for comfort + efficiency
- Suggests changes based on weather, humidity, outdoor conditions

**When Home Unoccupied** (ECO mode active):
- AI prioritizes energy savings
- **Cooling**: Allows temperature to rise (target + offset)
  - Example: Normal 72°F → ECO 77°F (with +5°F offset)
- **Heating**: Allows temperature to drop (target - offset)
  - Example: Normal 68°F → ECO 63°F (with -5°F offset)

**Configure Offsets:**
Adjust in blueprint configuration:
- **ECO Cooling Offset**: 0-15°F (default 5°F)
- **ECO Heating Offset**: 0-15°F (default 5°F)

---

## ⚙️ Advanced: MONITORED Mode

If you want **approval-required** mode where AI asks before making changes:

### Additional Requirements

**Per-zone input helpers** (create via Settings → Helpers):

```
input_text.ai_hvac_{zone}_pending_action (max 50)
input_text.ai_hvac_{zone}_pending_temp (max 10)
input_text.ai_hvac_{zone}_pending_mode (max 20)
input_text.ai_hvac_{zone}_pending_reason (max 255)
```

Replace `{zone}` with your zone ID (e.g., `lr`, `upstairs`)

### Setup Process

1. Create 4 input_text helpers per zone (see above)
2. Set blueprint **Automation Mode** to `MONITORED`
3. Add universal action handler automation (see repository for YAML)
4. Test by long-pressing notification to see approve/reject buttons

**Note:** Most users should use AUTOMATIC mode - it's simpler and equally safe with proper temperature bounds configured.

---

## 🔍 Monitoring

**View AI Decisions:**
- **Settings → System → Logs** → search "AI HVAC"
- See all suggestions, reasoning, confidence scores

**Check Execution:**
- **Settings → System → Logbook** → filter by zone name
- See execution history and ECO mode activations

---

## 🐛 Troubleshooting

### No Suggestions Being Made

1. Check automation is enabled
2. Verify AI agent works: **Settings → Voice Assistants** → test it
3. Check logs for errors
4. Ensure sensors are reporting valid values

### AI Suggestions Seem Wrong

1. Review AI reasoning in logs
2. Check if sensors are accurate
3. Adjust temperature bounds if too restrictive
4. Adjust ECO offsets if too aggressive
5. Add more optional sensors (outdoor temp, weather, etc.)

### ECO Mode Not Working

1. Verify occupancy sensor changes state when you leave
2. Check **ECO Mode Enabled** is ON in blueprint config
3. Review automation trace to see if ECO logic triggered
4. Look for "🌿 ECO" in notifications/logs

---

## 📊 Sample Dashboard Card

```yaml
type: entities
title: AI HVAC - Living Room
entities:
  - climate.living_room
  - sensor.living_room_temperature
  - entity: automation.ai_hvac_living_room
    name: AI Advisor
  - binary_sensor.anyone_home
  - entity: input_boolean.eco_mode_enabled
    name: ECO Mode
```

---

## ⚠️ Safety Features

✅ Temperature bounds prevent extreme suggestions  
✅ Max change limit prevents drastic swings  
✅ All AI decisions logged with full reasoning  
✅ Can disable per-zone anytime  
✅ Manual thermostat control always works  
✅ ECO mode optional and configurable  

---

## 🔬 How It Works

**Every Hour:**
1. AI analyzes current conditions (temp, humidity, weather, occupancy)
2. Considers ECO mode if away from home
3. Suggests optimal HVAC setting
4. **AUTOMATIC mode**: Executes + notifies
5. **MONITORED mode**: Sends notification for approval
6. Logs decision with full AI reasoning

**AI considers:**
- Indoor temperature vs target
- Indoor/outdoor humidity
- Weather forecast
- Doors/windows open
- Home occupancy
- Current HVAC mode
- ECO mode setbacks

---

## 📝 Configuration Examples

**Conservative (minimal changes):**
- Max Change: 2°F
- ECO Offsets: 3°F
- Check Interval: 2 hours

**Aggressive (maximum efficiency):**
- Max Change: 6°F
- ECO Offsets: 8°F
- Check Interval: 1 hour

**Balanced (recommended):**
- Max Change: 4°F
- ECO Offsets: 5°F
- Check Interval: 1 hour

---

## 🤝 Contributing

Contributions welcome! Open issues or PRs on GitHub.

---

## 📄 License

MIT License

---

## 👨‍💻 Author

Brandon Lord

Built for Home Assistant enthusiasts who want smarter climate control with AI.

---

## 🔗 Links

- [GitHub Repository](https://github.com/blord1976/ai-hvac-blueprint)
- [Home Assistant Community](https://community.home-assistant.io/)
- [Report Issues](https://github.com/blord1976/ai-hvac-blueprint/issues)
