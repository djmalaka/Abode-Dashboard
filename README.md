### CLOCK WEATHER CARD

**Purpose**

Glassmorphic header block for the Home dashboard that:

- Hides the HA top bar when kiosk mode is enabled (via a `kiosk-mode="on"` attribute on the view).
- Shows a live front door camera when the doorbell view is active.
- Otherwise shows:
  - A “Welcome Abode” title.
  - A large animated weather + time header (condition, temperature, current time, date).
  - A dynamic sun arc / lunar phase visual using `sun.sun` and `sensor.moon_phase`.
  - A compact 5-day forecast bar card with min/max temps, precipitation probability, and animated SVG icons.
  - A 12-hour temperature line chart with glowing gradient and animated path.

---

**Required custom cards / plugins**

Install via HACS (or equivalent):

- `custom:mod-card` (and `card-mod` for styling)
- `custom:layout-card`
- `custom:button-card`
- `custom:stack-in-card`

---

**Required entities**

- Doorbell toggle and camera  
  - `input_boolean.doorbell_camera_active` – turns the doorbell live view on/off.  
  - `camera.front_door_bell_live_view` – your front doorbell / camera entity.

- Time + sun + moon  
  - `sensor.time` – standard HA time sensor (Time & Date integration).  
  - `sun.sun` – core Sun integration.  
  - `sensor.moon_phase` – moon phase sensor (e.g. Moon integration).

- Weather  
  - `weather.forecast_abode` – main weather entity used for current condition + temperature.  
  - `sensor.tomorrow_raw_hourly` – raw hourly forecast sensor from Tomorrow.io (must expose
    `attributes.timelines.hourly` with `time` and `values.temperature`, `values.weatherCode`,
    `values.precipitationProbability`).

---

**Required assets**

Animated SVG weather icons in `/config/www` (exposed as `/local/...`):

- Line icons for the big header:
  - `/local/weather-icons/weather-icons-master/design/line/animation-ready/<icon>.svg`
- Fill icons for the 5-day forecast:
  - `/local/weather-icons/weather-icons-master/design/fill/animation-ready/<icon>.svg`

You can use any icon set as long as you update the paths in the JavaScript blocks.

---

**Kiosk mode behaviour**

The top-level `mod-card` includes CSS:

- Hides `app-header` and `app-toolbar` when the active `hui-view` has `kiosk-mode="on"`.

To use this:

- Either add `kiosk-mode="on"` to your view via a custom theme or custom panel, or  
- Remove that CSS block if you don’t need kiosk mode.

---

**How to adapt for your setup**

- Replace the following entities with your own:
  - `input_boolean.doorbell_camera_active` → your boolean controlling when to show the doorbell.
  - `camera.front_door_bell_live_view` → your door camera entity.
  - `weather.forecast_abode` → your main weather entity.
  - `sensor.tomorrow_raw_hourly` → your hourly forecast sensor (structure must match Tomorrow.io style).
  - `sensor.moon_phase` → your moon phase sensor (or adjust the code to remove lunar phase if not used).
- Ensure `sensor.time` and `sun.sun` are available (core integrations).
- Update icon file paths if your `/local` weather icons live in a different folder or if you use another icon pack.
- If you don’t want the header text “Welcome Abode 🙏”, change the markdown card content to whatever title you prefer.
- If you’re not using kiosk mode, you can safely remove the `/* Hide top bar when kiosk-mode is ON */` CSS section from the `style` block at the top.

---

**Notes**

- The 5-day forecast bars compute per-day min/max temperature and precipitation probability by aggregating the raw hourly timeline (no template sensors needed).
- The rain bar uses a glassmorphic gradient with a small 💧 icon and percentage label, and animates in a short time after page load.
- The 12-hour chart is an inline SVG rendered inside a `custom:button-card` using the first 12 entries of `sensor.tomorrow_raw_hourly.attributes.timelines.hourly`.

---
---

### ENERGY CARD

**Purpose**

A glassmorphic energy summary card showing:

- Electricity and Gas daily spend (from Bright / Hildebrand Glow integration).
- A horizontal-pill layout with animated pulses (blue for electricity, orange for gas).
- A smooth 24-hour mini-graph of cost trends (using `mini-graph-card`).
- Beautiful gradient background, rounded edges, and subtle drop-shadows.

---

**Required custom cards / plugins**

Install via HACS:

- `custom:stack-in-card`
- `custom:button-card`
- `custom:mini-graph-card`
- `card-mod`

---

**Required entities**

Electricity + gas cost sensors:
- `sensor.electricity` — *Displays the current electricity cost*  
- `sensor.gas` — *Displays the current gas cost*  
- `sensor.electricity_cost_today` — *Used for the 24h electricity graph*  
- `sensor.gas_cost_today` — *Used for the 24h gas graph*

> These typically come from the **Bright / Hildebrand Glow DCC** integration.

---

**Visual behaviour**

- The two pill-style header buttons show:
  - ⚡ Electricity price with blue theme
  - 🔥 Gas price with orange theme
- The bottom section displays a **mini-graph-card** line chart only when cost sensors are not `unknown`.
- Includes delayed animation for the graph path (smooth, glowing line).

---

**How to adapt for your setup**

- Replace the following entities with yours if named differently:
  - `sensor.electricity`
  - `sensor.gas`
  - `sensor.electricity_cost_today`
  - `sensor.gas_cost_today`
- Change colors by adjusting:
  - `#47C1E8` for electricity  
  - `#ffaa00` for gas
- You can remove the pulse animations if not needed by deleting the `@keyframes` blocks.
- Background gradient can be customised in:
  ```yaml
  background: linear-gradient(135deg, rgba(151, 87, 167, 0.25), rgba(75, 124, 132, 0.25));

---
---

### MEDIA CENTRE CARD

**Purpose**

A full-featured **Media Centre** for an Alexa-based home, combining:

- A dynamic **"Now Playing" header** that follows the active Echo (`sensor.active_echo_player`) and shows a `bubble-card` media player.
- Three round glassmorphic action buttons:
  - 📢 **Announcements** – type or dictate a message, select rooms, and broadcast via Alexa.
  - 🎵 **Music Request** – request music by text/voice, pick target Echo or group, and start playback.
  - 📞 **Drop-In Intercom** – start Alexa Drop-In calls between rooms using source/target selects.
  - ⏰ **Alarms / Timers / Reminders** – create Alexa alarms, timers, and reminders from one UI.
- A scrolling **“Top News + Upcoming reminders/alarms/timers”** ticker driven by helper sensors.
- A full list of all Echo devices and TVs as `bubble-card` media players.

---

**Required custom cards / plugins**

Install via HACS (or manually):

- `custom:mod-card`
- `custom:bubble-card`
- `custom:button-card`
- `custom:state-switch`
- `custom:mushroom-*` cards:
  - `mushroom-select-card`
  - `mushroom-number-card`
  - `mushroom-title-card`
- `custom:text-input-row`
- `card-mod`
- `browser_mod` (for popups + javascript actions)

---

**Core entities & helpers**

**1. Active player & media devices**

- `sensor.active_echo_player`  
  → State is one of your media_player IDs (e.g. `media_player.kitchen_echo`). Drives the header `state-switch`.
- Echo media players:
  - `media_player.hallway_echo`
  - `media_player.kitchen_echo`
  - `media_player.lounge_echo`
  - `media_player.master_bedroom_echo`
  - `media_player.upstairs_echo`
  - `media_player.room_2_echo`
  - `media_player.room_3_echo`
  - `media_player.room_4_echo`
  - `media_player.study_echo`
  - `media_player.everywhere` (optional Echo group)
- TVs:
  - `media_player.lg_webos_tv`
  - `media_player.kitchen_tv`

Replace these with your own media_player entities if different.

---

**2. Announcement system (popup 1 – 📢)**

Helpers:

- `input_text.announcement_message` – holds the announcement text.
- `input_boolean.mic_recording` – UI flag for mic animation + JS SpeechRecognition toggle.
- Per-room booleans to choose speakers:
  - `input_boolean.echo_hallway`
  - `input_boolean.echo_kitchen`
  - `input_boolean.echo_lounge`
  - `input_boolean.echo_master_bedroom`
  - `input_boolean.echo_upstairs`
  - `input_boolean.echo_room_2`
  - `input_boolean.echo_room_3`
  - `input_boolean.echo_room_4`
  - `input_boolean.echo_study`
  - `input_boolean.all_echoes`
- Script:
  - `script.make_an_announcement` – sends the message to the selected Echo devices.

Features:

- Text input for announcement.
- Mic button uses **browser_mod.javascript** + browser **SpeechRecognition API** (Chrome/Edge) and writes text to `input_text.announcement_message`.
- Quick preset buttons: “Food is ready”, “Stop fighting”, “Keep quiet please!”.
- Grid of Echo selection buttons with glowing “ON” state.

---

**3. Music Request Station (popup 2 – 🎵)**

Helpers:

- `input_text.music_request_query` – current music request text.
- `input_text.last_music_request` – last successful request (used in scrolling chip).
- `input_boolean.mic_recording` – reused for mic animation.
- `input_select.music_target_speaker` – where to play (e.g. Hallway, Lounge, Everywhere, etc).
- `input_select.music_transfer_targets` – drop-down for transfer target.
- Script:
  - `script.play_requested_music` – handles TTS/command to Alexa to play the requested music.
  - `script.transfer_music_to_echo` – transfers current music from source Echo to selected target.

Features:

- Text box + mic input for request.
- Preset chips (Jim Reeves, George Ezra, etc).
- “Last Requested” chip with animated scrolling text.
- `bubble-card` select for target speaker.
- “🎶 Play Now” button to trigger `script.play_requested_music`.
- Conditional **“Transfer music from X → [target]”** bar that:
  - Detects which Echo is playing.
  - Lets you choose a target from `input_select.music_transfer_targets`.
  - Shows a 🎵 Transfer button when a valid room is selected.

---

**4. Drop-In Intercom (popup 3 – 📞)**

Helpers:

- `input_select.intercom_source` – “Calling from” Echo.
- `input_select.intercom_target` – “To” Echo.

Script:

- `script.initiate_intercom` – reads source/target selects and sends a custom command to Alexa to start a Drop-In call.

UI:

- Two `mushroom-select-card` dropdowns (source/target).
- Single “🤙 Drop in Now” button.

---

**5. Alexa Alarms / Timers / Reminders (popup 4 – ⏰)**

Tab selector:

- `input_select.alexa_popup_tab` – values: `Alarm`, `Timer`, `Reminder`.

Alarm helpers:

- `input_select.alexa_alarm_device` – Echo to set alarm on.
- `input_select.alarm_hour` – hour (e.g. 1–12).
- `input_number.alarm_minute` – minute.
- `input_select.alarm_period` – AM/PM.
- Script: `script.set_alexa_alarm`.

Timer helpers:

- `input_select.alexa_timer_device`
- `input_number.alexa_timer_value`
- `input_select.alexa_timer_unit` (e.g. minutes/hours)
- Script: `script.set_alexa_timer`.

Reminder helpers:

- `input_select.alexa_reminder_device`
- `input_text.alexa_reminder_text`
- Script: `script.set_alexa_reminder`.

Each tab shows the relevant controls and a single “Set Alarm / Set Timer / Set Reminder” button.

---

**6. News + Upcoming alarms/timers/reminders ticker**

Entities:

- `event.top_stories_google_news` – used for headline + link.
- A 5-second ticking helper:
  - `input_boolean.5_second_timer` – used just to re-render and keep the scrolling content fresh.
- Per-echo “next event” sensors:
  - For each Echo (hallway, kitchen, lounge, master_bedroom, upstairs, room_2, room_3, room_4, study):
    - `sensor.<echo>_echo_next_reminder`
    - `sensor.<echo>_echo_next_alarm`
    - `sensor.<echo>_echo_next_timer`

Behaviour:

- Builds a JS array of upcoming events across all devices.
- Calculates time remaining in minutes / hours or shows date/time if > 24 hours away.
- Displays a vertically scrolling list:
  - Top: **Top News** headline (if available).
  - Then: **“Upcoming”** with all timers/alarms/reminders.
- You can tap the ticker to open the top news article link.

> Note: The card has two versions of this ticker – one when `sensor.active_echo_player != none` (shorter height) and one when it is `none` (taller height).

---

**Styling / Design**

- Entire card sits in a glassmorphic container with:
  - Gradient background (`red → blue` tone).
  - Blur, borders, and soft shadows.
- All main controls use rounded, glowing buttons and subtle tap animations (`tapPulse`, `pressEffect`).
- Designed for Storage Mode: **all logic is inline**, no YAML includes or Jinja templates.

---

**How to adapt for your setup**

- Replace all `media_player.*_echo` and TV entities with your own device names.
- Adjust helper entity IDs (`input_text.*`, `input_boolean.*`, `input_select.*`, `input_number.*`) to match your defined helpers, or create new helpers with these names.
- Hook the scripts (`script.make_an_announcement`, `script.play_requested_music`, `script.transfer_music_to_echo`, `script.initiate_intercom`, `script.set_alexa_alarm`, `script.set_alexa_timer`, `script.set_alexa_reminder`) to your Alexa integration (e.g. using `media_player.play_media` with `media_content_type: custom` commands).
- If you don’t use Google News, remove or replace `event.top_stories_google_news` and the `tap_action: url`.

---

**Prerequisites**

- A working **Alexa Media** or similar integration that exposes Echo devices as `media_player.*`.
- **browser_mod** configured in Home Assistant.
- Helper entities and scripts set up to match the ones listed above.

---
---

### ROOM CARDS

This file defines the complete set of glassmorphic **room summary cards** used across the home dashboard.  
They provide presence, temperature, automation, lighting, fans, blind controls, and door-state indicators.

---

## Dependencies

- `custom:button-card`
- `custom:stack-in-card`
- `custom:card-mod`

**Scripts**
- `script.control_blinds`
- `script.master_bedroom_toggle_all_lights`
- `script.master_bedroom_blinds_boolean_toggle`
- `script.room_2_toggle_all_lights`
- `script.room_2_blinds_boolean_toggle`
- `script.room_3_blinds_boolean_toggle`
- `script.room_4_blinds_boolean_toggle`
- `script.study_toggle_all_lights`
- `script.study_blinds_boolean_toggle`

---

## Entities Used (grouped by room)

---

### **Lounge**
- **Presence**
  - `binary_sensor.lounge_fp2_presence`
- **Temperature / Climate**
  - `climate.lounge_thermostat`
  - `sensor.lounge_current_temperature`
- **Automation**
  - `input_boolean.lounge_light_control`
- **Lights**
  - `binary_sensor.lounge_any_light_on`
- **Fan**
  - `fan.kitchen_fan`
- **Blinds**
  - `cover.lounge_shutter`
  - `input_boolean.lounge_blinds_expanded`
  - `input_boolean.lounge_blinds_collapsed`
  - `input_select.lounge_blinds_last_command`
  - `script.lounge_blinds_boolean_toggle`
  - `script.control_blinds`

---

### **Hallway**
- **Presence**
  - `binary_sensor.presence_sensor_fp2_b046_presence_sensor_1`
- **Temperature / Climate**
  - `climate.hallway_thermostat`
  - `sensor.hallway_current_temperature`
- **Automation**
  - `input_boolean.upstairs_stairway_and_hallway_light_automation`
- **Lights**
  - `light.hallway_lights`

---

### **Kitchen / Dining / Living**
- **Presence**
  - `binary_sensor.kitchen_dining_living_fp2_overallpresence`
- **Temperature / Climate**
  - `climate.kitchen_thermostat`
  - `sensor.kitchen_current_temperature`
- **Automation**
  - `input_boolean.kitchen_automation`
- **Lights**
  - `binary_sensor.kitchen_any_light_on`
  - `script.kitchen_toggle_all_lights`
- **Fan**
  - `fan.kitchen_fan`

---

### **WC**
- **Presence**
  - `binary_sensor.wc_motion_sensor_2`
- **Temperature / Climate**
  - `climate.wc_thermostat`
  - `sensor.wc_current_temperature`
- **Automation**
  - `input_boolean.wc_automation`
- **Lights**
  - `light.wc_light`
- **Door**
  - `binary_sensor.wc_door_sensor1`

---

### **Garage**
- **Automation**
  - `input_boolean.garage_automation`
- **Lights**
  - `light.garage_light`

---

### **Stairs to Foyer / Landing**
- **Presence**
  - `binary_sensor.upstairs_fp2_overallpresence`
- **Automation**
  - `input_boolean.upstairs_stairway_and_hallway_light_automation`
- **Lights**
  - `switch.smart_switch_2211283432205855080548e1e9b1820f_outlet`
  - `light.stairway_light`

---

### **Master Bedroom**
- **Presence**
  - `binary_sensor.master_bedroom_fp2_presence_sensor_1`
- **Temperature / Climate**
  - `sensor.linknlink_e04b41006845_temperature`
- **Automation**
  - `input_boolean.master_bedroom_automation`
- **Lights**
  - `binary_sensor.master_bedroom_any_light_on`
  - `script.master_bedroom_toggle_all_lights`
- **Fan**
  - `input_boolean.master_bedroom`
- **Blinds**
  - `cover.master_bedroom_shutter`
  - `input_boolean.master_bedroom_blinds_expanded`
  - `input_boolean.master_bedroom_blinds_collapsed`
  - `input_select.master_bedroom_blinds_last_command`
  - `script.master_bedroom_blinds_boolean_toggle`
  - `script.control_blinds`

---

### **Ensuite**
- **Presence**
  - `binary_sensor.ensuite_presence`
- **Temperature / Climate**
  - `climate.ensuite_thermostat`
  - `sensor.ensuite_current_temperature`
- **Automation**
  - `input_boolean.ensuite_automation`
- **Lights**
  - `light.ensuite_light`
- **Door**
  - `binary_sensor.ensuite_door_sensor`

---

### **Room 2**
- **Presence**
  - `binary_sensor.room_2_presence`
- **Temperature / Climate**
  - `climate.room_2_thermostat`
  - `sensor.room_2_current_temperature`
- **Automation**
  - `input_boolean.room_2_automation`
- **Lights**
  - `binary_sensor.room_2_any_light_on`
  - `script.room_2_toggle_all_lights`
  - `light.room2_light`
- **Blinds**
  - `cover.room_2_shutter`
  - `input_boolean.room_2_blinds_expanded`
  - `input_boolean.room_2_blinds_collapsed`
  - `input_select.room_2_blinds_last_command`
  - `script.room_2_blinds_boolean_toggle`
  - `script.control_blinds`

---

### **Room 3**
- **Presence**
  - `binary_sensor.room_3_presence`
- **Temperature / Climate**
  - `climate.room_3_thermostat`
  - `sensor.room_3_current_temperature`
- **Automation**
  - `input_boolean.room_3_automation`
- **Lights**
  - `light.room3_light`
- **Fan**
  - `fan.room_3_fan`
- **Blinds**
  - `cover.room_3_shutter`
  - `input_boolean.room_3_blinds_expanded`
  - `input_boolean.room_3_blinds_collapsed`
  - `input_select.room_3_blinds_last_command`
  - `script.room_3_blinds_boolean_toggle`
  - `script.control_blinds`

---

### **Room 4**
- **Presence**
  - `light.room_4_light`
- **Temperature / Climate**
  - `climate.room_4_thermostat`
  - `sensor.room_4_current_temperature`
- **Automation**
  - `input_boolean.room_4_automation`
- **Lights**
  - `light.room_4_light`
- **Fan**
  - `fan.room_3_fan`
- **Blinds**
  - `cover.room_4_shutter`
  - `input_boolean.room_4_blinds_expanded`
  - `input_boolean.room_4_blinds_collapsed`
  - `input_select.room_4_blinds_last_command`
  - `script.room_4_blinds_boolean_toggle`
  - `script.control_blinds`

---

### **Study**
- **Presence**
  - `binary_sensor.study_fp2_presence_sensor_2`
- **Temperature / Climate**
  - `climate.study_thermostat`
  - `sensor.study_current_temperature`
- **Automation**
  - `input_boolean.study_automation`
- **Lights**
  - `binary_sensor.study_any_light_on`
  - `script.study_toggle_all_lights`
- **Fan**
  - `fan.study_fan`
- **Blinds**
  - `cover.study_shutter`
  - `input_boolean.study_blinds_expanded`
  - `input_boolean.study_blinds_collapsed`
  - `input_select.study_blinds_last_command`
  - `script.study_blinds_boolean_toggle`
  - `script.control_blinds`

---

### **Washroom**
- **Presence**
  - `binary_sensor.washroom_presence`
- **Temperature / Climate**
  - `climate.washroom_thermostat`
  - `sensor.washroom_current_temperature`
- **Automation**
  - `input_boolean.washroom_automation`
- **Lights**
  - `light.washroom_light`
- **Door**
  - `binary_sensor.contact_sensor`

---


