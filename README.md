# AI HVAC Override + Learning System

**Phase 1 MVP** - User override commands with AI-powered natural language parsing and basic learning system.

---

## 🎯 What This Does

Adds a third button to your AI HVAC notifications that lets you **override** the AI's suggestion with your own command using natural language:

**Before (2 buttons):**
```
AI: "Turn off HVAC - doors are open"
You: [Approve] [Reject]
```

**After (3 buttons):**
```
AI: "Turn off HVAC - doors are open"  
You: [Approve] [Reject] [Override]
     ↓ (tap Override)
Text Input: "No, set it to 78° instead"
     ↓
System: ✅ Set to 78°F
        🧠 Learned your preference
```

**The AI learns from your overrides** and will incorporate these preferences into future suggestions!

---

## 📦 What's Included

### **1. Updated Blueprint** (`ai_hvac_blueprint_unified.yaml`)
- Adds "Override" button with text input to notifications
- Only available in MONITORED mode
- Works with existing automation configurations

### **2. Universal Override Handler** (`ai_hvac_override_handler.yaml`)
- Single automation that handles ALL zones
- Uses Gemini AI to parse natural language commands
- Executes commands on thermostats
- Stores learning data
- Sends confirmation notifications

### **3. This README**
- Complete setup instructions
- Helper creation guide
- Command examples
- Troubleshooting

---

## 🚀 Installation (30 minutes)

### **Step 1: Update Existing Blueprint** (5 minutes)

1. **Delete old blueprint:**
   - Settings → Automations & Scenes → Blueprints
   - Find "AI HVAC Advisor"
   - Click ⋮ → Delete Blueprint

2. **Import updated blueprint:**
   - Click "Import Blueprint"
   - Paste URL: `https://raw.githubusercontent.com/blord1976/ai-hvac-blueprint/main/ai_hvac_blueprint_unified.yaml`
   - (Or manually upload the file)

3. **Verify automations still work:**
   - Settings → Automations & Scenes
   - Check both AI HVAC automations are still enabled
   - They should continue working normally

---

### **Step 2: Create Learning Storage Helpers** (10 minutes)

You need to create input_text helpers to store override learning data **for each zone**.

**For Living Room:**

1. **Settings → Devices & Services → Helpers**
2. Click **"+ Create Helper"**
3. Choose **"Text"**
4. Configure:
   - **Name:** `AI HVAC LR Learning Log`
   - **Entity ID:** `input_text.ai_hvac_lr_learning_log`
   - **Maximum length:** 5000
   - **Icon:** `mdi:school`
5. Click **"Create"**

6. Create another helper:
   - **Name:** `AI HVAC LR Pending Thermostat`
   - **Entity ID:** `input_text.ai_hvac_lr_pending_thermostat`
   - **Maximum length:** 100
   - Click **"Create"**

**For Upstairs (repeat above):**
- `AI HVAC Upstairs Learning Log` → `input_text.ai_hvac_upstairs_learning_log` (max 5000)
- `AI HVAC Upstairs Pending Thermostat` → `input_text.ai_hvac_upstairs_pending_thermostat` (max 100)

**For future zones:** Follow same pattern replacing zone name.

---

### **Step 3: Install Override Handler Automation** (5 minutes)

**Option A: Via UI (Recommended)**

1. **Settings → Automations & Scenes**
2. Click **"+ Create Automation"**
3. Click **"⋮" menu** (top right) → **"Edit in YAML"**
4. **Delete everything** in the editor
5. **Copy the entire contents** of `ai_hvac_override_handler.yaml`
6. **Paste** into the editor
7. Click **"Save"**
8. Name it: `AI HVAC - Universal Override Handler`

**Option B: Via automations.yaml**

1. Open your `/config/automations.yaml` file
2. Copy the contents of `ai_hvac_override_handler.yaml`
3. Paste at the bottom
4. Reload automations:
   - Developer Tools → YAML → Reload Automations

---

### **Step 4: Update Blueprint to Store Thermostat** (10 minutes)

The override handler needs to know which thermostat to control. Update each AI HVAC automation:

**For Living Room automation:**

1. Open the automation in edit mode
2. Click **"⋮" menu** → **"Edit in YAML"**
3. Find the section that starts with `# Store pending action data`
4. Add this action **before** the event action:

```yaml
          # Store thermostat entity for override handler
          - service: input_text.set_value
            target:
              entity_id: input_text.ai_hvac_lr_pending_thermostat
            data:
              value: "{{ thermostat }}"
```

5. Click **"Save"**

**Repeat for Upstairs automation** (change `lr` to `upstairs` in entity ID).

---

## ✅ Testing

### **Test 1: Verify Override Button Appears**

1. Wait for next hourly AI HVAC check (or manually run automation)
2. Check notification - should have **3 buttons** now:
   - ✅ Approve
   - ❌ Reject
   - 🔧 Override

### **Test 2: Send an Override Command**

1. Tap **"Override"** button
2. Type: `set to 72`
3. Tap **"Send"**
4. Check:
   - Thermostat should change to 72°F
   - You get confirmation notification
   - Logbook shows the override
   - Learning log helper contains the event

### **Test 3: Check Learning Data**

1. **Developer Tools → States**
2. Search: `input_text.ai_hvac_lr_learning_log`
3. Should see: `2026-04-02T20:30:00|set to 72|adjust_temp|72|null;`

---

## 💬 Command Examples

### **Temperature Commands:**
- `set to 78`
- `78 degrees`
- `make it 72°F`
- `change to 70`
- `set temp to 75`

### **Mode Commands:**
- `switch to heat mode`
- `use auto mode`
- `change to cooling`
- `heat mode please`
- `put it in auto`

### **Combined Commands:**
- `set to 72 in auto mode`
- `78 degrees cool mode`
- `heat at 68`
- `make it 70 and use auto`

### **Turn Off:**
- `turn it off`
- `turn off the HVAC`
- `shut it down`
- `off`

### **Complex Examples:**
- `No, don't turn it off, set to 78 instead`
- `ignore that, just make it 72`
- `actually keep it on at 75`

---

## 🧠 How Learning Works

### **Phase 1 (Current): Basic Learning**

**What gets stored:**
- Timestamp (for time-of-day patterns)
- Your command text
- Parsed action (adjust_temp, change_mode, turn_off)
- Temperature value (if specified)
- Mode value (if specified)

**Example log entry:**
```
2026-04-02T19:30:00|set to 78 instead|adjust_temp|78|cool;
2026-04-02T20:15:00|switch to auto mode|change_mode|null|heat_cool;
```

### **Phase 2 (Future): Advanced Learning**

**Pattern recognition:**
- Evening overrides → Learn evening preferences
- When doors open → Learn your response
- High outdoor temp → Learn preferred settings

**Auto-incorporation into AI prompts:**
```
Based on past overrides:
- Evenings: User prefers 78°F not off when doors open
- Hot days (>85°F): User prefers 76°F not 72°F
```

**Confidence-based suggestions:**
```
AI: "Based on 3 similar situations, suggest 78°F (confidence: 90%)"
```

---

## 🐛 Troubleshooting

### **Override button doesn't appear**

**Check:**
1. Blueprint was updated (delete old, import new)
2. Automation mode is **MONITORED** (not AUTOMATIC)
3. Automation has run at least once since update

### **"No thermostat found" error**

**Fix:**
1. Make sure you added the `input_text.set_value` action to store thermostat
2. Check helper exists: `input_text.ai_hvac_[zone]_pending_thermostat`
3. Manually trigger automation to populate helper

### **"Command unclear" - Low confidence**

**Try:**
- More specific commands: "set to 72" instead of "make it cooler"
- Include units: "78°F" or "78 degrees"
- Use standard mode names: "heat", "cool", "auto"

### **Command executes wrong action**

**Examples of AI parsing:**
- "warmer" → May not parse (be specific: "75°F")
- "auto" → Changes mode to auto
- "off" → Turns off HVAC

**Solution:** Check logbook to see what was parsed, then use clearer commands.

### **Learning data not saving**

**Check:**
1. Helper exists with correct entity ID
2. Helper max length is 5000
3. Check Developer Tools → States to see current value
4. If full (5000 chars), clear it or increase max length

---

## 📊 Viewing Learning Data

### **Via Developer Tools:**

1. **Developer Tools → States**
2. Search: `input_text.ai_hvac_lr_learning_log`
3. See all overrides as semicolon-separated entries

### **Via Logbook:**

1. **Settings → System → Logbook**
2. Filter: `AI HVAC Override`
3. See all parsed commands with timestamps

### **Decode Log Format:**

```
timestamp|user_command|action|temperature|mode;
```

Example:
```
2026-04-02T19:30:00|set to 78|adjust_temp|78|cool;
```
= On April 2 at 7:30 PM, user overrode with "set to 78", system parsed as adjust_temp to 78°F in cool mode

---

## 🔮 Future Enhancements (Phase 2)

**Planned features:**
- ✅ Pattern recognition engine
- ✅ Automatic preference profiles
- ✅ Multi-condition learning (time + weather + occupancy)
- ✅ "Stop suggesting this" learning
- ✅ Cross-zone preference correlation
- ✅ ML-based prediction refinement
- ✅ Dashboard showing learned preferences
- ✅ Export/import preference profiles

**Timeline:** Phase 2 development after Phase 1 validation (few weeks of real-world usage)

---

## 📁 File Locations

**Blueprint:** `/config/blueprints/automation/blord1976/ai_hvac_blueprint_unified.yaml`  
**Override Handler:** `/config/automations.yaml` (or separate file)  
**Learning Logs:** Stored in input_text helpers (accessible via Developer Tools)

---

## 🤝 Support

**Issues:** https://github.com/blord1976/ai-hvac-blueprint/issues  
**Discussions:** https://github.com/blord1976/ai-hvac-blueprint/discussions  
**Documentation:** https://github.com/blord1976/ai-hvac-blueprint

---

## ⚖️ License

MIT License - Free to use, modify, and distribute

---

## 👨‍💻 Author

Brandon Lord

Built for Home Assistant enthusiasts who want smarter, adaptive HVAC control with AI learning.

---

**Enjoy your AI-powered, learning HVAC system!** 🎉🌡️🧠
