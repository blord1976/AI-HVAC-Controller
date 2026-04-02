# AI HVAC Advisor Blueprint - Complete Installation Guide

## 🎯 Overview

The AI HVAC Advisor Blueprint provides intelligent, AI-powered HVAC optimization with:
- **Multi-thermostat support** (deploy across multiple zones)
- **ECO mode** with automatic occupancy detection
- **Actionable notifications** with approve/reject buttons
- **Configurable temperature bounds** and safety limits
- **Full transparency** with logged reasoning for every suggestion

Built for Home Assistant using Gemini AI (or compatible conversation agents).

---

## 📋 Prerequisites

### Required
1. **Home Assistant** 2024.1 or later
2. **Gemini AI integration** (or compatible conversation agent)
   - Settings → Voice Assistants → Add Assistant
   - Configure with your Gemini API key
3. **Mobile app** configured on iPhone/Android
   - Companion app installed and logged in
   - Notifications enabled

### Recommended
- Outdoor temperature sensor
- Weather integration
- Door/window sensors
- Humidity sensors

---

## 🚀 Quick Start Installation

### Step 1: Import the Blueprint (5 minutes)

**Option A: Via UI** (easiest)
1. Download `ai_hvac_advisor_eco_blueprint.yaml`
2. Go to **Settings → Automations & Scenes → Blueprints**
3. Click **Import Blueprint**
4. Click **Upload** and select the file

**Option B: Manual**
1. Copy `ai_hvac_advisor_eco_blueprint.yaml` to:
   `/config/blueprints/automation/blord1976/ai_hvac_advisor_eco.yaml`
2. Restart Home Assistant or reload automations

### Step 2: Create Input Helpers (10 minutes per zone)

**For Living Room Zone:**

1. Go to **Settings → Devices & Services → Helpers**
2. Click **+ CREATE HELPER** and create these entities:

**Toggle Helpers:**
- `input_boolean.ai_hvac_lr_enabled` - AI HVAC LR Enabled
- `input_boolean.ai_hvac_lr_eco_mode` - AI HVAC LR ECO Mode (turn ON)

**Text Helpers:**
- `input_text.ai_hvac_lr_pending_action` (max 50 chars)
- `input_text.ai_hvac_lr_pending_temp` (max 10 chars)
- `input_text.ai_hvac_lr_pending_mode` (max 20 chars)
- `input_text.ai_hvac_lr_pending_reason` (max 255 chars)

**Counter Helpers:**
- `counter.ai_hvac_lr_approvals`
- `counter.ai_hvac_lr_rejections`

**Number Helpers:**
- `input_number.ai_hvac_lr_eco_cooling_offset` (min: 0, max: 10, step: 1, default: 5, unit: °F)
- `input_number.ai_hvac_lr_eco_heating_offset` (min: 0, max: 10, step: 1, default: 5, unit: °F)

**Tip:** You can paste the contents of `ai_hvac_helpers_all_zones.yaml` into your `configuration.yaml` instead of creating via UI!

### Step 3: Create Automation from Blueprint (5 minutes)

1. Go to **Settings → Automations & Scenes → Automations**
2. Click **+ CREATE AUTOMATION → Create new automation**
3. Select **AI HVAC Advisor with ECO Mode**

**Fill in configuration:**

**Required Entities:**
- **Thermostat**: `climate.lr_stat`
- **Indoor Temperature Sensor**: `sensor.lr_stat_air_temperature`
- **AI Conversation Agent**: `conversation.google_generative_ai`
- **Notification Service**: `mobile_app_brandons_iphone_16_pro`
- **Occupancy Sensor**: `binary_sensor.smart_home_occupancy`

**Optional Entities:**
- **Indoor Humidity**: `sensor.lr_stat_humidity`
- **Outdoor Temperature**: `sensor.outdoor_temperature`
- **Outdoor Humidity**: `sensor.temp_humidity_sensor_humidity`
- **Weather Entity**: `weather.forecast_home`
- **Door Sensor**: `sensor.doors`

**Input Helpers:**
- **Enable Boolean**: `input_boolean.ai_hvac_lr_enabled`
- **ECO Mode Boolean**: `input_boolean.ai_hvac_lr_eco_mode`
- **Pending Action Text**: `input_text.ai_hvac_lr_pending_action`
- **Pending Temp Text**: `input_text.ai_hvac_lr_pending_temp`
- **Pending Mode Text**: `input_text.ai_hvac_lr_pending_mode`
- **Pending Reason Text**: `input_text.ai_hvac_lr_pending_reason`
- **Approval Counter**: `counter.ai_hvac_lr_approvals`
- **Rejection Counter**: `counter.ai_hvac_lr_rejections`
- **ECO Cooling Offset**: `input_number.ai_hvac_lr_eco_cooling_offset`
- **ECO Heating Offset**: `input_number.ai_hvac_lr_eco_heating_offset`

**Configuration:**
- **Zone Name**: `Living Room`
- **Approval Action ID**: `APPROVE_LR_HVAC`
- **Rejection Action ID**: `REJECT_LR_HVAC`
- **Min Temperature**: `60` °F
- **Max Temperature**: `85` °F
- **Max Temp Change**: `4` °F

4. Click **Save** and name it: `AI HVAC Advisor - Living Room`

### Step 4: Add Approval/Rejection Handlers (5 minutes)

The blueprint automation analyzes conditions and sends notifications, but you need separate automations to handle button presses.

1. Open your `automations.yaml` file (File Editor or Studio Code Server)
2. Scroll to the **very bottom**
3. Paste the contents of `ai_hvac_approval_rejection_handlers.yaml`
4. **Important:** Update entity IDs if needed:
   - `climate.lr_stat` → your thermostat entity
   - `notify.mobile_app_brandons_iphone_16_pro` → your notification service
5. Save and reload automations

### Step 5: Test the System (10 minutes)

**Test Notification Buttons:**
1. Go to **Settings → Automations**
2. Find **AI HVAC Advisor - Living Room**
3. Click **RUN** button
4. Wait ~10 seconds for AI analysis
5. Check your iPhone for notification
6. **LONG PRESS** the notification (don't tap!)
7. You should see **[✅ Approve] [❌ Reject]** buttons

**Test ECO Mode:**
1. Turn ON: `input_boolean.ai_hvac_lr_eco_mode`
2. Simulate leaving home: turn OFF `binary_sensor.smart_home_occupancy`
3. Trigger the automation again
4. AI should suggest ECO temperature setbacks
5. Notification should show "🌿 ECO Mode Active"

**Verify Execution:**
1. Approve a suggestion
2. Check that thermostat actually changed
3. Check **Settings → System → Logs** → search "AI HVAC"
4. You should see approval logged

---

## 🔧 Multi-Thermostat Setup

### Adding Upstairs Zone

Repeat Steps 2-4 for the Upstairs zone:

**Step 2:** Create helpers with `upstairs` prefix:
- `input_boolean.ai_hvac_upstairs_enabled`
- `input_boolean.ai_hvac_upstairs_eco_mode`
- `input_text.ai_hvac_upstairs_pending_action`
- (etc... see template file)

**Step 3:** Create automation from blueprint:
- **Zone Name**: `Upstairs`
- **Thermostat**: `climate.upstairs_stat`
- **Indoor Temp**: `sensor.upstairs_stat_air_temperature`
- **Approval Action ID**: `APPROVE_UPSTAIRS_HVAC`
- **Rejection Action ID**: `REJECT_UPSTAIRS_HVAC`
- (use same shared entities: occupancy, outdoor temp, weather, AI agent)

**Step 4:** Approval/rejection handlers are already in the YAML file!

---

## 📊 Dashboard Setup

Add this monitoring card to track performance:

```yaml
type: vertical-stack
cards:
  # System Overview
  - type: entities
    title: AI HVAC System
    entities:
      - binary_sensor.smart_home_occupancy
      - input_boolean.ai_hvac_lr_eco_mode
      - input_boolean.ai_hvac_upstairs_eco_mode
  
  # Living Room Zone
  - type: entities
    title: Living Room HVAC
    entities:
      - climate.lr_stat
      - sensor.lr_stat_air_temperature
      - input_boolean.ai_hvac_lr_enabled
      - sensor.ai_hvac_lr_approval_rate
      - counter.ai_hvac_lr_approvals
      - counter.ai_hvac_lr_rejections
  
  # Upstairs Zone
  - type: entities
    title: Upstairs HVAC
    entities:
      - climate.upstairs_stat
      - sensor.upstairs_stat_air_temperature
      - input_boolean.ai_hvac_upstairs_enabled
      - sensor.ai_hvac_upstairs_approval_rate
      - counter.ai_hvac_upstairs_approvals
      - counter.ai_hvac_upstairs_rejections
```

---

## 🌿 ECO Mode Configuration

### How ECO Mode Works

**When Home is Occupied** (normal mode):
- AI optimizes for comfort + efficiency
- Suggests temperature changes based on weather, humidity, outdoor conditions

**When Home is Unoccupied** (ECO mode):
- AI prioritizes energy savings
- **Cooling**: Suggests target + ECO offset (default +5°F)
  - Example: Normal 72°F → ECO 77°F
- **Heating**: Suggests target - ECO offset (default -5°F)
  - Example: Normal 68°F → ECO 63°F

### Configuring ECO Offsets

**Per-Zone Configuration:**
- `input_number.ai_hvac_lr_eco_cooling_offset` - How many °F to add when cooling (away)
- `input_number.ai_hvac_lr_eco_heating_offset` - How many °F to subtract when heating (away)

**Recommended Settings:**
- **Mild climate**: 3-4°F offset
- **Normal climate**: 5-6°F offset (default)
- **Extreme climate**: 7-8°F offset

### Disabling ECO Mode

Turn OFF per zone:
- `input_boolean.ai_hvac_lr_eco_mode`
- `input_boolean.ai_hvac_upstairs_eco_mode`

AI will continue normal optimization when away.

---

## 🔍 Monitoring & Optimization

### Approval Rate Tracking

**Target:** >80% approval rate
- **80-100%**: AI is making excellent suggestions
- **60-80%**: AI is good but could improve
- **<60%**: AI needs tuning (see troubleshooting)

**Check approval rates:**
- `sensor.ai_hvac_lr_approval_rate`
- `sensor.ai_hvac_upstairs_approval_rate`

### Viewing AI Decisions

**All suggestions (approved + rejected):**
1. **Settings → System → Logs**
2. Search: `AI HVAC`
3. See: timestamp, suggestion, reasoning, confidence

**Automation traces:**
1. **Settings → Automations**
2. Find automation → Click **⋮** → **Traces**
3. See: full AI response, template parsing, execution

---

## 🐛 Troubleshooting

### Notifications Not Showing Buttons

**iOS Issue:** You must **LONG PRESS** the notification
- Don't tap - that opens Home Assistant
- Press and hold for 1-2 seconds
- Buttons will appear when notification expands

### AI Suggesting Bad Temperatures

**Too aggressive:**
- Lower `input_number.ai_hvac_*_eco_cooling_offset`
- Lower `input_number.ai_hvac_*_eco_heating_offset`
- Reduce **Max Temp Change** in blueprint config

**Too conservative:**
- Increase ECO offsets
- Check if doors/windows are triggering turn-off

### Low Approval Rate (<60%)

1. **Check AI reasoning** in logbook
2. **Common issues:**
   - ECO offsets too aggressive
   - Missing outdoor temp sensor (AI guessing)
   - Door sensor too sensitive
3. **Tune configuration:**
   - Adjust temperature bounds
   - Adjust ECO offsets
   - Add more optional sensors

### Buttons Not Executing

1. **Check automation exists:**
   - `automation.ai_hvac_lr_approve_handler`
   - `automation.ai_hvac_lr_reject_handler`
2. **Check entity IDs match:**
   - Approval handler uses correct thermostat
   - Notification service matches
3. **Check automation trace:**
   - Did button press trigger automation?
   - Any errors in execution?

### ECO Mode Not Working

1. **Check occupancy sensor:**
   - Is `binary_sensor.smart_home_occupancy` working?
   - Does it actually change when you leave?
2. **Check ECO mode enabled:**
   - `input_boolean.ai_hvac_*_eco_mode` = ON
3. **Check AI prompt:**
   - View automation trace
   - Confirm "ECO MODE ACTIVE" in prompt

---

## 🎯 Next Steps

**After 1-2 Weeks:**
1. Review approval rates for both zones
2. Tune ECO offsets based on comfort vs savings
3. Add more zones if needed
4. Consider bounded autonomous mode (future)

**Future Enhancements:**
- Solar production awareness
- Time-of-use electricity pricing
- Pre-conditioning before schedule events
- Learning loop from manual overrides
- Multi-zone coordination (don't cool LR while heating upstairs)

---

## 📁 File Structure

```
/config/
├── blueprints/automation/blord1976/
│   └── ai_hvac_advisor_eco.yaml           # Main blueprint
├── automations.yaml                        # Add approval/rejection handlers here
└── configuration.yaml                      # Add helpers here (or use UI)
```

---

## 🔗 Support & Resources

**ClickUp Project:** AI HVAC Blueprint folder
**GitHub:** (future) github.com/blord1976/ai-hvac-blueprint
**Home Assistant Community:** (future) Share your setup!

---

## 📝 Credits

Created by Brandon Lord for multi-zone AI-powered HVAC optimization with Gemini integration.

Built with ❤️ for Home Assistant enthusiasts who want smarter, more efficient climate control.
