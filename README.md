# ✨ Abode Dashboard (Home Assistant)

![Screenshot_20251208_223111_Fully Kiosk Browser](https://github.com/user-attachments/assets/0ee04375-9042-4334-9e10-87b6ade4f304)

**Conceptualised & designed by Malaka Jayawardene.**

**Important:** This is a highly user-specific Home Assistant dashboard.  
The YAML in this repository will **not** function out-of-the-box on other systems due to unique helpers, sensors, custom logic, and tailored UI behaviour.

Instead, **use this repo as a guide** to learn and adopt:
- Styling techniques  
- Advanced card constructions  
- Animation & transitions  
- Browser_mod pop-up flows  
- Glassmorphism design  
- Room summary logic  
- Presence mapping  
- Quickfire controls  
- Weather, media, and automation UI patterns  

This repository is best viewed as a **concept library** that demonstrates how the Abode dashboard was built so you can replicate the _patterns_, not the exact implementation.

---

## 🚀 Quick Start — How to Use This Repo

Before diving in, please note that this dashboard is **not** intended to be copied and pasted directly into your Home Assistant setup.  
Nearly every card depends on custom helpers, scripts, sensors, booleans, and UI assets that are **unique** to this installation.

Instead:

---

### **1. Browse the folder structure**

Each folder contains a self-contained part of the UI:

- **Weather Dashboard** — animated icons, sunrise/sunset arc, forecasts
- **Energy Card** - based on Hildebrand integration
- **Media Station** — speaker groups, requests, transfers 
- **Room Cards** — glassmorphic room tiles with presence, lights, climate, blinds
- **Location Card** - Based on a sleek combination of iOS shortcuts and webhooks (will post a separate guide on this in due course as currently only YAML behind the card is included)
- **Surveillance** — live camera streams & motion states
- **Housekeepers Card** — robot vacuums & mower control    
- **Quickfire Controls** — lights, climate, blinds, automations
- **Scripts & Helpers** — logic powering UI behaviour (will be updating this in due course)

Use these as references to understand layout, structure, and design patterns.

---

### **2. Identify ideas you want to borrow**

This repo contains many techniques that are hard to find elsewhere, including:

- Dynamic background effects (gradient fills, presence states, battery/progress bars)  
- Animated icon systems  
- Multi-stage browser_mod pop-ups  
- State-based UI morphing  
- Smooth transitions between collapsed/expanded card modes  
- Layered visibility animations within a single card  
- Full-page modal takeovers  
- RTSP camera cropping + grid layouts  

Pick the concepts you like and adapt them to your own entities.

---

### **3. Replace entities with your own**

Every YAML section uses entities such as:

```
binary_sensor.<custom_presence>
climate.<zone_heating>
cover.<somfy_blinds>
camera.<eufy_custom>
sensor.house_temperature_weighted
input_boolean.<dashboard_flags>
script.<custom_logic>
```

These **must** be replaced with your own Home Assistant entities.

---

### **4. Install required custom components**

Most UI elements rely on:

- **button-card**  v4.3.0
- **browser_mod**  v2.6.1
- **layout-card**  v2.4.6
- **stack-in-card**  v0.2.0
- **card-mod**  v3.4.5
- **slider-entity-row**  v17.4.1
- **state-switch**  v1.9.6
- **bubble card**  v3.0.4 
- **thermostat-card** (HA native)  

Make sure these are installed before attempting to use any examples.

---

### **5. Review scripts & helpers**

Many UI elements depend on:

- Light snapshots  
- Automation snapshots  
- PIN-protected automation kill gate  
- Blinds control helper scripts  
- ETA & presence processing  
- Media control scripts  

You’ll need equivalents adapted to your setup. The separate scripts section will soon be added.

---

## 📜 License

This project is shared for learning and inspiration, not as a turnkey package.
You are free to copy styling techniques, animations, UI behaviour, and card structures.

Please do not repackage or redistribute the entire configuration as your own.

## ❤️ A Note from the Author

This dashboard has been built over hundreds of hours of iteration —
tuning animations, refining behaviour, and making every interaction feel smooth.

If you find ideas here useful, feel free to ⭐ star the repo or share screenshots of what you build.


## 📦 What’s Included in the Abode Dashboard?


<h1 style="color:#8AB4F8; font-weight:700;">CLOCK & WEATHER CARD</h1>

<img width="546" height="409" alt="image" src="https://github.com/user-attachments/assets/f47f4701-2a35-4325-925f-70505247392d" />


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

# Animated SVG weather icons 
in `/config/www` (exposed as `/local/...`) - Provided in the repo

- Line icons for the big header:
  - `/local/weather-icons/weather-icons-master/design/line/animation-ready/<icon>.svg`
- Fill icons for the 5-day forecast:
  - `/local/weather-icons/weather-icons-master/design/fill/animation-ready/<icon>.svg`

You can use any icon set as long as you update the paths in the JavaScript blocks as appropriate to your setup.

# 🧠 Backend Setup (Required)

The clock-weather card uses a few *template sensors* and one *REST sensor* for Tomorrow.io.

Add these to `configuration.yaml` (or your preferred split YAML files).

## Template Sensors (Sun, Moon, Weather)
```yaml
template: 
  - sensor:
      - name: "Weather Abode Temperature"
        state: "{{ state_attr('weather.forecast_abode', 'temperature') }}"
        unit_of_measurement: "°C"

      - name: "Weather Abode Condition"
        state: "{{ states('weather.forecast_abode') }}"

      - name: "Moonrise Time"
        state: >
          {% set lat = state_attr('zone.home','latitude') %}
          {% set lon = state_attr('zone.home','longitude') %}
          {% set times = integration('astral').get_moon_times(now().date(), lat, lon) %}
          {{ times['rise'].astimezone(now().tzinfo) if 'rise' in times else 'unknown' }}

      - name: "Moonset Time"
        state: >
          {% set lat = state_attr('zone.home','latitude') %}
          {% set lon = state_attr('zone.home','longitude') %}
          {% set times = integration('astral').get_moon_times(now().date(), lat, lon) %}
          {{ times['set'].astimezone(now().tzinfo) if 'set' in times else 'unknown' }}

      - name: "Moon is Up"
        state: >
          {% set rise = states('sensor.moonrise_time') %}
          {% set set = states('sensor.moonset_time') %}
          {% set n = now().strftime('%H:%M') %}
          {{ (rise != 'unknown' and set != 'unknown') and (rise < n < set) }}
```
## Tomorrow.io REST Sensor (Hourly Forecast)
```yaml
rest:
  - resource: https://api.tomorrow.io/v4/weather/forecast?location=52.2677,0.3560&timesteps=1h&units=metric&apikey=!secret tomorrow_api_key _(register at tomorrow.io to obtain the API Key)_
    scan_interval: 1800
    sensor:
      - name: "Tomorrow Raw Hourly"
        value_template: "OK"
        json_attributes:
          - timelines
```
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

<h1 style="color:#8AB4F8; font-weight:700;">ENERGY CARD</h1>

<img width="560" height="169" alt="image" src="https://github.com/user-attachments/assets/3013eb69-1f4d-4fda-a3ac-5c3855336338" />

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

<h1 style="color:#8AB4F8; font-weight:700;">MEDIA CENTRE CARD</h1>

<img width="562" height="285" alt="image" src="https://github.com/user-attachments/assets/a31254cf-d2b3-4913-9197-4897d4ca8374" />

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

<img width="709" height="868" alt="image" src="https://github.com/user-attachments/assets/f73d34f5-5a3f-4ec7-930e-a06ade8274c2" />

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

<img width="709" height="1284" alt="image" src="https://github.com/user-attachments/assets/3fc7a2c4-b77b-4f11-9c72-e8dd134a0fa2" />

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

<img width="709" height="783" alt="image" src="https://github.com/user-attachments/assets/33002b3c-57a2-4360-a542-69d91ece0b6a" />

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

<img width="709" height="616" alt="image" src="https://github.com/user-attachments/assets/fa358e0a-b0ee-4f13-8c80-9ef20467525f" />

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

<h1 style="color:#8AB4F8; font-weight:700;">ROOMS CARD</h1>

<img width="571" height="825" alt="image" src="https://github.com/user-attachments/assets/82814fa8-5606-467d-9b99-a27d711f86df" />

This file defines the complete set of glassmorphic **room summary cards** used across the home dashboard.  
They provide presence, temperature, automation, lighting, fans, blind controls, and door-state indicators.

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
---

<h1 style="color:#8AB4F8; font-weight:700;">LOCATION, TRAVEL PROGRESS & SECURITY CARD</h1>

<img width="562" height="221" alt="image" src="https://github.com/user-attachments/assets/c03e8f50-c6b2-40b8-9ca2-601fb19044f3" />


This file defines a glassmorphic **Location & Security strip** that shows:

- Two animated **presence / travel cards**:
  - **Person 1** – travel progress *or* phone battery, ETA, and current place
  - **Person 2** – travel progress *or* phone battery, ETA, and current place
- A compact **Home Security** alarm control panel
- A **Front Door Lock** status/quick-control button

The visual style matches the rest of the dashboard:

- Soft, blurred, glassmorphic background
- Gradient progress bars filling from left to right
- Circular avatar badges with profile photos or silhouettes
- Scrolling subtitle line for ETA / location
- Smooth transitions for progress changes

---

### Behaviour & Logic

#### Progress vs Battery logic

Each person card computes a `progress` variable that controls the gradient fill:

- When **away from home** (or just left):
  - Uses `sensor.home_travel_progress_person_1` / `sensor.home_travel_progress_person_2`
    (0–100%) as the fill amount.
  - The subtitle line shows:  
    `Reaching home in <ETA>` when `input_text.person_1_eta` / `input_text.person_2_eta` is valid.
- When **at home for more than ~2 minutes**:
  - Falls back to **phone battery level (%)** as the fill:
    - `sensor.person_1_phone_battery_level`
    - `sensor.person_2_phone_battery_level`
  - The name line shows a battery icon, for example:  
    `Person 1 ⚡️ 78%`.

There is a short grace period after arriving (travel progress 100% for <2 mins) so the bar can finish animating before switching to battery.

#### Subtitle text

- **Away with ETA available**  
  `Reaching home in <ETA>`
- **Otherwise**  
  `<PlaceLabel> • <last updated X mins/hours ago>`

Where:

- `PlaceLabel` is:
  - `"Home"` when state is `home`
  - `"Work"` when state is `Work`
  - `"Away"` for `not_home/unknown/away`
  - Or a matching `zone.*` friendly name
  - Or a cleaned-up version of the raw state.

The subtitle line scrolls horizontally (`scroll-left` keyframe) inside a masked area.

#### Security & Lock

- **Home Security** uses `mushroom-alarm-control-panel-card` for quick arming:
  - `armed_home`
  - `armed_away`
- **Front Door Lock** uses `mushroom-lock-card` with a minimal, background-less style so it blends into the glassmorphic container.

---

### Required Entities

#### Person 1 card

- **Person / Location**
  - `person.person_1`
- **Travel / ETA**
  - `sensor.home_travel_progress_person_1` – 0–100% distance progress toward home
  - `input_text.person_1_eta` – human-readable ETA text (“12 mins”, etc.)
- **Battery**
  - `sensor.person_1_phone_battery_level`
  - `sensor.person_1_phone_battery_state` – used to detect charging (available if you want a visual hint)
- **Timer / Refresh helper**
  - `input_boolean.person_1_refresh_timer` – used as a polling trigger for smoother updates
- **Avatar image (frontend only)**
  - `/local/person_1.png`

#### Person 2 card

- **Person / Location**
  - `person.person_2`
- **Travel / ETA**
  - `sensor.home_travel_progress_person_2`
  - `input_text.person_2_eta`
- **Battery**
  - `sensor.person_2_phone_battery_level`
  - `sensor.person_2_phone_battery_state`
- **Avatar image (frontend only)**
  - `/local/person_2.png`

#### Security / Lock

- **Alarm**
  - `alarm_control_panel.alarmo`
- **Front door**
  - `lock.front_door_lock_ultra`

---

### Dependencies

This file uses the following custom cards / integrations:

- [`custom:mod-card`](https://github.com/thomasloven/lovelace-card-mod) – outer glassmorphic wrapper
- [`custom:button-card`](https://github.com/custom-cards/button-card) – for the person/location tiles
- [`Mushroom cards`](https://github.com/piitaya/lovelace-mushroom)
  - `custom:mushroom-alarm-control-panel-card`
  - `custom:mushroom-lock-card`
- [`card-mod`](https://github.com/thomasloven/lovelace-card-mod) – under the hood for `mod-card` and `card_mod` styling

Make sure they are installed (e.g. via HACS) and added to your Lovelace `resources:` configuration.

---
---

<h1 style="color:#8AB4F8; font-weight:700;">SURVEILLANCE CARD</h1>

This card defines a glassmorphic **surveillance overview panel** that shows:

- A header row with:
  - **“Surveillance”** label
  - Animated **Active Stream status text**: `Cameras Actively Streaming`
  - A round **arm/disarm button** that blinks red when active
- A main camera row:
  - **Entry camera snapshot** (left)
  - **Outdoor live stream** (right)
- A 4-tile grid of **indoor cameras** that:
  - Show **snapshots** when idle
  - Show live feed on tap when surveillance is armed

All entity IDs below are **anonymised** for safety of publishing. Replace them with your own entities when adapting this card.

---

### Required Entities

#### Surveillance Control

- **Trigger / Arm Boolean**
  - `input_boolean.surveillance_trigger`
- **Arm Script**
  - `script.surveillance_arm_live` – starts / arms your live RTSP streaming

#### Entry / Door Camera

- **Snapshot**
  - `camera.cam_entry_1_snapshot`
- **Live view**
  - `camera.cam_entry_1_live`

#### Outdoor Camera

- **Live view**
  - `camera.cam_outdoor_1_live`

#### Indoor Camera 1

- **Snapshot**
  - `camera.cam_indoor_1_snapshot`
- **Live view**
  - `camera.cam_indoor_1_live`

#### Indoor Camera 2

- **Snapshot**
  - `camera.cam_indoor_2_snapshot`
- **Live view**
  - `camera.cam_indoor_2_live`

#### Indoor Camera 3

- **Snapshot**
  - `camera.cam_indoor_3_snapshot`
- **Live view**
  - `camera.cam_indoor_3_live`

#### Indoor Camera 4

- **Snapshot**
  - `camera.cam_indoor_4_snapshot`
- **Live view**
  - `camera.cam_indoor_4_live`

Each indoor tile uses `input_boolean.surveillance_trigger` via `state-switch`:

- `"off"` → show snapshot only  
- `"on"` → snapshot with **tap to open live** target

---

### Dependencies

You’ll need the following custom cards / integrations:

- **Custom cards**
  - [`custom:mod-card`](https://github.com/thomasloven/lovelace-layout-card)
  - [`custom:button-card`](https://github.com/custom-cards/button-card)
  - [`custom:layout-card`](https://github.com/thomasloven/lovelace-layout-card)
  - [`custom:state-switch`](https://github.com/thomasloven/lovelace-state-switch)
  - [`custom:card-mod`](https://github.com/thomasloven/lovelace-card-mod)

- **Core Lovelace cards**
  - `picture-entity`
  - `vertical-stack`
  - `horizontal-stack`

Make sure these are installed (typically via HACS) and added to your Lovelace `resources:`.

---
---

<h1 style="color:#8AB4F8; font-weight:700;">HOUSEKEEPERS CARD</h1>

<img width="570" height="249" alt="image" src="https://github.com/user-attachments/assets/76938e90-b917-4cd2-9823-b526d186bde6" />

This card defines a glassmorphic **“House Keepers”** panel for three robots (e.g. lawn mower + two vacuums). It shows:

- A **title row**:
  - “House Keepers” label
  - A **scrolling status line** driven by a sensor (e.g. next runs, battery, areas)
  - Subtle title animation when any robot has left the dock or is returning
- A **three-column robot row**:
  - Large robot artwork (one per robot)
  - Under each image: a trio of **round controls**:
    - **Play** (start / resume)
    - **Pause** (pause / play-pause icon)
    - **Home** (send back to dock / returning state)

Each robot uses an `active`, `paused`, and `returning` boolean to control both UI animation and whatever automations you wire up behind them.

All entity IDs below are **anonymised** and safe to publish. Replace them with your own helpers/scripts in your setup.

---

### Required Entities

#### Shared Status / Header

- **Status sensor**
  - `sensor.housekeepers_status`  
    Used for the scrolling status line (e.g. “Lawn mower due at 10:30 • Upstairs vac cleaning bedrooms…”).

- **Activity flags**
  - `input_boolean.hk1_active`
  - `input_boolean.hk2_active`
  - `input_boolean.hk3_active`
  - `input_boolean.hk1_returning`
  - `input_boolean.hk2_returning`
  - `input_boolean.hk3_returning`

The header animation checks whether **any** robot is active or returning and slides the title + status line accordingly.

---

### Housekeeper 1 (e.g. Lawn Mower)

- **Booleans**
  - `input_boolean.hk1_active` – robot is running / away from dock
  - `input_boolean.hk1_paused` – paused vs playing (for icon swap)
  - `input_boolean.hk1_returning` – returning to dock

- **Behaviour**
  - **Play button** → `input_boolean.hk1_active` turned on
  - **Pause button** → toggles `input_boolean.hk1_paused`
  - **Home button** → turns on `input_boolean.hk1_returning`

- **Extras**
  - **App launch** (optional, via Fully Kiosk):  
    `fully.startApplication('com.eco.global.app')`

---

### Housekeeper 2 (e.g. Downstairs Vacuum)

- **Booleans**
  - `input_boolean.hk2_active`
  - `input_boolean.hk2_paused`
  - `input_boolean.hk2_returning`

- **Script**
  - `script.hk2_stop_and_reset_ui` – example script to stop the robot and reset UI flags

- **Behaviour**
  - **Play button** → turns on `input_boolean.hk2_active`
  - **Pause button** → toggles `input_boolean.hk2_paused`
  - **Home button** → calls `script.hk2_stop_and_reset_ui`

- **Extras**
  - App launch example: `fully.startApplication('com.eufylife.smarthome')`

---

### Housekeeper 3 (e.g. Upstairs Vacuum)

- **Booleans**
  - `input_boolean.hk3_active`
  - `input_boolean.hk3_paused`
  - `input_boolean.hk3_returning`

- **Behaviour**
  - **Play button** → turns on `input_boolean.hk3_active`
  - **Pause button** → toggles `input_boolean.hk3_paused`
  - **Home button** → turns on `input_boolean.hk3_returning`

- **Extras**
  - App launch example: `fully.startApplication('com.dreame.smartlife')`

---

### Dependencies

You’ll need:

- **Custom cards**
  - [`custom:mod-card`](https://github.com/thomasloven/lovelace-card-mod)
  - [`custom:button-card`](https://github.com/custom-cards/button-card)
  - [`custom:card-mod`](https://github.com/thomasloven/lovelace-card-mod)

- **Core Lovelace cards**
  - `vertical-stack`
  - `horizontal-stack`

- **Optional**
  - **Fully Kiosk Browser** (for `fully.startApplication(...)` app-launch actions on wall tablets)

Make sure the custom cards are installed (often via HACS) and added to your Lovelace `resources:`.

---
---

<h1 style="color:#8AB4F8; font-weight:700;">QUICKFIRE CARD</h1>

<img width="570" height="146" alt="image" src="https://github.com/user-attachments/assets/4f1a2e80-7060-4c73-9ee9-9d9b996aea6a" />

The **Quickfire** card is a glassmorphic control panel that gives you **fast access** to three common “whole-house” actions:

- **Lights** – snapshot and control all active lights with a smart “restore / replace” flow.
- **Heat** – adjust multiple thermostats (downstairs, upstairs, whole house) from one popup.
- **Blinds** – send **open / stop / my / close** to all blinds at once.
- **Automate** – view system “telemetry” and safely **disable / restore automations** using a PIN-protected keypad.

The card is designed to sit as a **single row** with four large buttons, each opening a full-screen glassy popup.

---

## Dependencies

### Custom Cards / Integrations

You’ll need:

- [`custom:mod-card`](https://github.com/thomasloven/lovelace-card-mod)
- [`custom:button-card`](https://github.com/custom-cards/button-card)
- [`browser_mod`](https://github.com/thomasloven/hass-browser_mod)

Optional core cards used inside popups (bundled with Home Assistant):

- `grid`, `vertical-stack`
- `markdown`
- `thermostat`

Make sure all custom cards + browser_mod are installed (e.g. via HACS) and added in **Settings → Dashboards → Resources**.

---

## High-Level Layout

Quickfire (mod-card)
├─ Title: “Quickfire”
└─ 4-column grid:
   ├─ Lights   → full-screen popup (smart “all lights” control)
   ├─ Heat     → full-screen popup (3 thermostats)
   ├─ Blinds   → full-screen popup (all blinds open/stop/my/close)
   └─ Automate → full-screen popup (telemetry + automation kill/restore with PIN)

The **outer `mod-card`** provides the frosted glass background, rounded corners, and drop shadow.  
Inside it, a **`vertical-stack`** contains:

1. A centered **“Quickfire”** heading.  
2. A **4-column `grid`** – one button per feature.

---

Button 1 – Lights
-----------------

<img width="547" height="291" alt="image" src="https://github.com/user-attachments/assets/00b428bd-f02a-44b4-a290-cedee47b4392" />

### Purpose

- Shows a count of **currently-on lights** (`sensor.on_lights_count`).
- On tap, opens a popup that:
  - Detects if there’s a **saved snapshot** of lights (`sensor.lights_snapshot_storage`).
  - Asks different questions depending on:
    - Are some lights currently on?
    - Is there a stored snapshot?
  - Calls a **single script** with different `command` variables:
    - `save_and_off` – save the current on-lights to snapshot and turn them off.
    - `restore_and_clear` – restore previous lights from snapshot and clear it.
    - `override_and_off` – turn off current lights and replace snapshot.

### Required Entities / Scripts

- `sensor.on_lights_count` – number of lights currently on.
- `sensor.lights_snapshot_storage` – stores CSV list of entity IDs for snapshot.
- `script.smart_lights_control` – main controller script that accepts a `command` variable:
  - `save_and_off`
  - `restore_and_clear`
  - `override_and_off`

### Popup Behaviour

The popup is built with `browser_mod.popup` and nested `mod-card` layers to:

- Make the popup **full screen** and edge-to-edge.
- Show a **title animation** (“Control All Lights”).
- Show conditional Markdown text:
  - All off & no snapshot → “All lights are already off…”
  - No snapshot & some on → “Turn off all current lights?”
  - Snapshot & none on → “Restore your previous lights?”
  - Snapshot & some on → “What would you like to do?” (Restore vs Turn off & replace)
- Show **two main buttons**:
  - Green: “Yes / Restore / Close” (depending on context)
  - Red: “Cancel / Turn off & replace” (hidden entirely if not applicable)
- A final **“Close”** pill button to exit.

The main grid button also:

- Displays a **lightbulb image** (`/local/Quickfire/lighton.png` or `/local/Quickfire/lightoff.png`).
- Shows the **count badge** on top of the image when `on_lights_count > 0`.

---

Button 2 – Heat
---------------

<img width="1223" height="627" alt="image" src="https://github.com/user-attachments/assets/4c97c648-09fa-4d0d-a7b2-687ee8cd006b" />

### Purpose

- Shows a small thermostat icon and **house-average temperature** (`sensor.house_temperature_weighted`).
- On tap, opens a popup to control **3 thermostats**:
  - `climate.downstairs_heating` → “Downstairs”
  - `climate.upstairs_heating` → “Upstairs”
  - `climate.whole_house_heating` → “Whole House”

Each thermostat appears in a floating glass card with a subtle **fade/zoom/blur-in** animation.

### Required Entities

- `sensor.house_temperature_weighted`
- `climate.downstairs_heating`
- `climate.upstairs_heating`
- `climate.whole_house_heating`

Additional climate entities referenced in the icon logic (for “any heating on?” check):

- Downstairs:
  - `climate.hallway_thermostat`
  - `climate.lounge_thermostat`
  - `climate.kitchen_thermostat`
  - `climate.wc_thermostat`
- Upstairs:
  - `climate.ensuite_thermostat`
  - `climate.washroom_thermostat`
  - `climate.room_2_thermostat`
  - `climate.room_3_thermostat`
  - `climate.room_4_thermostat`
  - `climate.study_thermostat`

### Icon Behaviour

- If **any** of the above thermostats have state not equal to `off`, the button uses:
  - `/local/Quickfire/thermostaton.png`
- Otherwise:
  - `/local/Quickfire/thermostatoff.png`
- A badge shows the **weighted house temperature** to one decimal place (e.g. `21.3°`).

---

Button 3 – Blinds
-----------------

<img width="454" height="627" alt="image" src="https://github.com/user-attachments/assets/9fd4a847-5011-4a82-b559-99335e6e76a0" />

### Purpose

- Shows a blinds icon and the **last command** sent to the lounge blinds.
- On tap, opens a popup that sends **global commands** (`cover_entity: all`) to your `script.control_blinds`:
  - **Open** → `command: open`
  - **Stop** → `command: stop`
  - **My** → `command: my`
  - **Close** → `command: close`

Each control is a large, glowing circular button with subtle staggered animations.

### Required Entities / Scripts

- **Somfy control script**
  - `script.control_blinds`
    - Accepts at least:
      - `cover_entity` (here set to `all`)
      - `command` (`open`, `stop`, `my`, `close`)
      - `browser_id` (used to target the calling browser for UI feedback, optional depending on your implementation)
- **State helper (for the small icon badge):**
  - `input_select.lounge_blinds_last_command`  
    Possible values: `open`, `close`, `my`, `stop`.

### Icon Behaviour

- Shows `/local/Quickfire/blinds.png` as the base image.
- Displays a small circular badge with a **mdi icon** based on last command:

| Command | Badge Icon       |
|---------|------------------|
| open    | `mdi:arrow-up`   |
| close   | `mdi:arrow-down` |
| my      | `mdi:star`       |
| stop    | `mdi:pause`      |

---

Button 4 – Automate
-------------------

![Screenshot_20251208_225240_Fully Kiosk Browser](https://github.com/user-attachments/assets/9e940338-1943-4e43-96f6-d6847308aeb7)

### Purpose

This is the **advanced** button – a full “system control” popup that:

1. Shows animated **terminal-style telemetry**:
   - Integrations/components
   - Recent automations & scripts
   - Randomised entity events
2. Lets you **kill all automations** OR **restore** them from a snapshot.
3. Protects the “kill” path with a **PIN keypad**.

### Required Entities / Scripts / Helpers

**Snapshot & counts**

- `sensor.automations_snapshot_storage` (and optional `_2`)
  - Attributes:
    - `entities` (comma-separated list of automations)
    - `count` / `bool_count` (numbers of things to restore)
- `sensor.enabled_automations_count` (and optional `_2`)

**Kill gate & PIN**

![Screenshot_20251208_225255_Fully Kiosk Browser](https://github.com/user-attachments/assets/cde2c373-9d2c-4766-a2a4-291361689f23)

- `input_boolean.automation_kill_gate` – controls whether the keypad is shown.
- `sensor.automation_kill_pin_dots` – displays `••••` style dots in the UI.
- `sensor.automation_kill_pin_status` – text feedback:
  - `Correct code` / `Incorrect code` etc.
- `input_text.automation_kill_pin_entered` – stores the digits as user types.

**PIN scripts**

- `script.append_digit_to_kill_pin` – add a digit `"0"–"9"`.
- `script.backspace_kill_pin` – delete last digit.
- `script.automations_toggle_banner` – main logic to:
  - Enable/disable automations.
  - Validate `code`.
  - Optionally skip PIN when `skip_pin: true`.
- Optional:
  - `sensor.enabled_automations_count_2`
  - `sensor.automations_snapshot_storage_2`

### Behaviour

- First prompt:
  - If **all automations are disabled** but there is a snapshot → primary button becomes **“Restore previous”**, calling `script.automations_toggle_banner` with `skip_pin: true`.
  - Otherwise, primary button opens the **kill gate**:
    - `input_boolean.automation_kill_gate` → `on`
- When kill gate is **on**:
  - Shows a **PIN keypad** (0–9, clear, backspace).
  - “Confirm” button calls `script.automations_toggle_banner` with:
    - `variables.code`: the entered PIN.
  - “Cancel” button turns off `input_boolean.automation_kill_gate`.

The grid icon shows:

- `/local/Quickfire/automationon.png` if any automations are enabled.
- `/local/Quickfire/automationoff.png` if not.
- A numeric badge counting how many automations / scripts have been triggered in the **last 60 seconds**.

