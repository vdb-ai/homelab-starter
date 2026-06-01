# Homelab — Home Assistant Automations & Scripts

> **Source of truth:** Individual YAML files in [`ha-automations/`](ha-automations/) and [`ha-scripts/`](ha-scripts/).
> This file is a human-readable index. To edit an automation, edit the YAML file and run `deploy-ha-automations.sh`.

## Deploy Workflow
1. Edit the relevant `ha-automations/<slug>.yaml` or `ha-scripts/<slug>.yaml` file
2. Run: `./deploy-ha-automations.sh` (Touch ID prompt, then auto)
3. Verify in HA Settings → Automations
4. Commit: `git add ha-automations/ ha-scripts/ && git commit -m "feat: ..."`

---

## Summary

| Name | Type | Trigger | Purpose |
| :--- | :--- | :--- | :--- |
| UPS — On Battery | Automation | State (ups_status_data OL→OB) | Slack alert when UPS transitions from line to battery power |
| UPS — Shutdown Imminent | Automation | Numeric state (battery_runtime < 900s while OB) | Early warning Slack + mobile push when runtime < 15 min |
| UPS — Power Restored | Automation | State (ups_status_data →OL from OB) | Slack alert when power restored with outage duration |
| Workday TV Schedule | Automation | Time | Turns a monitor TV on/off on workdays, skips PTO/holidays |
| Doorbell – Visitor Alert | Automation | Doorbell visitor sensor | Sends rich push notification with snapshot when someone rings the doorbell |
| Doorbell – Package Detection | Automation | Doorbell package sensor | Sends rich push notification with snapshot when a package is detected |
| Doorbell – Person Detected | Automation | Doorbell person sensor (24/7) | Sends rich push notification with snapshot when a person is detected at the doorbell |
| Outdoor Zone – Person Detected | Automation | Outdoor person sensor (HH PM – HH AM only) | Sends rich push notification with snapshot when a person is detected outdoors overnight |
| Bedside Lamps — Scene Button 1 Toggle | Automation | Scene controller Button 1 single tap | Toggles both bedside lamps in sync |
| Bedroom Fans — Scene Button 2 Toggle | Automation | Scene controller Button 2 single tap | Toggles both bedroom fan plugs in sync |
| Air Purifiers — Scene Button 3 Toggle | Automation | Scene controller Button 3 single tap | Toggles both air purifier plugs in sync |
| Security Status — Scene Button 4 LED | Automation | Door lock / garage door state changes | Updates Button 4 LED: green=all secure, red=any open/unlocked |
| Security Lock — Scene Button 4 Press | Automation | Scene controller Button 4 single tap | Locks front door if unlocked; never unlocks |
| Pet Feeder Evening Auto Feed | Automation | Time (HH:MM AM / HH:MM PM daily) | Triggers the Feed Pet script twice daily |
| Watchdog — Uptime Kuma Reachable | Automation | Time pattern (every 5 min) | Pings Uptime Kuma via `rest_command`; push notification on 2 consecutive failures |
| Air Purifiers Off | Automation | Manual / unknown trigger | Turns off both air purifier plugs |
| All Air Purifiers On | Automation | Manual / unknown trigger | Turns on all air purifiers in the house |
| Toggle Monitor TV | Script | Manual / called by automation | WoL on or turns off the monitor TV |
| Toggle Living Room TV | Script | Manual | WoL on or turns off the living room TV |
| Feed Pet | Script | Manual / called by automation | Triggers one manual feed portion on the pet feeder |

---

## Automations

### Workday TV Schedule

Automatically wakes a monitor TV at HH:MM AM and turns it off at HH:MM PM, Monday–Friday. Checks a work calendar for PTO or holiday events and skips those days. Sends a push notification to the primary phone on both actions.

**Key entities:**
- `media_player.tv_1`
- `calendar.work_calendar`
- `notify.mobile_app_phone_1`
- WoL MAC: `aa:bb:cc:dd:ee:05` (Monitor TV)

**File:** [`ha-automations/workday-tv-schedule.yaml`](ha-automations/workday-tv-schedule.yaml)

---

### Doorbell – Visitor Alert

Triggers when the doorbell detects a visitor press. Takes a snapshot from the doorbell camera, waits 1 second for the file to write, then sends a rich push notification to the household with the image attached and a deep-link button to open the camera live view. 3-minute cooldown after firing.

**Key entities:**
- `binary_sensor.doorbell_visitor`
- `camera.doorbell_snapshot`
- `notify.household`
- Snapshot path: `/config/www/snapshots/doorbell_visitor.jpg`

**File:** [`ha-automations/doorbell-visitor-alert.yaml`](ha-automations/doorbell-visitor-alert.yaml)

---

### Doorbell – Package Detection

Triggers when the doorbell's package detection sensor activates. Same flow as the visitor alert — snapshot, 1-second wait, rich push notification with image and camera deep-link. 3-minute cooldown after firing.

**Key entities:**
- `binary_sensor.doorbell_package`
- `camera.doorbell_snapshot`
- `notify.household`
- Snapshot path: `/config/www/snapshots/doorbell_package_drop.jpg`

**File:** [`ha-automations/doorbell-package-detection.yaml`](ha-automations/doorbell-package-detection.yaml)

---

### Doorbell – Person Detected

Triggers when the doorbell's person detection sensor activates. Runs 24/7 with no time restriction. Takes a snapshot, waits 1 second for the file to write, then sends a rich push notification to the household with the image attached and a deep-link button to open the camera live view. 3-minute cooldown after firing.

**Key entities:**
- `binary_sensor.doorbell_person`
- `camera.doorbell_snapshot`
- `notify.household`
- Snapshot path: `/config/www/snapshots/doorbell_person.jpg`

**File:** [`ha-automations/doorbell-person-detected.yaml`](ha-automations/doorbell-person-detected.yaml)

---

### Outdoor Zone – Person Detected

Triggers when an outdoor camera's person detection sensor activates, but only between HH PM and HH AM. Takes a snapshot, waits 1 second for the file to write, then sends a rich push notification to the household with the image attached and a deep-link button to open the camera live view. 3-minute cooldown after firing.

**Key entities:**
- `binary_sensor.outdoor_zone_person`
- `camera.outdoor_zone_snapshot`
- `notify.household`
- Snapshot path: `/config/www/snapshots/outdoor_zone_person.jpg`

**File:** [`ha-automations/outdoor-zone-person-detected.yaml`](ha-automations/outdoor-zone-person-detected.yaml)

---

### Bedside Lamps — Scene Button 1 Toggle

Triggers on a single tap of Button 1 (lamp icon) on the scene controller in the bedroom. If either bedside lamp is on, turns both off. If both are off, turns both on. This prevents mismatched lamp states — any light on means "pressing turns everything off."

**Key entities:**
- `event.scene_controller_scene_001` (trigger — Button 1 single tap)
- `switch.bedside_lamp_1` (bedside lamp plug)
- `switch.bedside_lamp_2` (bedside lamp plug)

**File:** [`ha-automations/bedside-lamps-button-1-toggle.yaml`](ha-automations/bedside-lamps-button-1-toggle.yaml)

---

### Bedroom Fans — Scene Button 2 Toggle

Triggers on a single tap of Button 2 on the scene controller in the bedroom. If either fan plug is on, turns both off. If both are off, turns both on. LED color indicates state (cyan = on, white = off).

**Prerequisites:** Set `select.scene_controller_led_indicator_button_2` to "Always on" in HA.

**Key entities:**
- `event.scene_controller_scene_002` (trigger — Button 2 single tap)
- `switch.fan_plug_1`
- `switch.fan_plug_2`
- `select.scene_controller_led_indicator_color_button_2` (Button 2 LED color — cyan=on, white=off)

**File:** [`ha-automations/bedroom-fans-button-2-toggle.yaml`](ha-automations/bedroom-fans-button-2-toggle.yaml)

---

### Air Purifiers — Scene Button 3 Toggle

Triggers on a single tap of Button 3 (fan icon) on the scene controller in the bedroom. If either plug is on, turns both off. If both are off, turns both on.

**Key entities:**
- `event.scene_controller_scene_003` (trigger — Button 3 single tap)
- `switch.air_purifier_1`
- `switch.ac_plug`

**File:** [`ha-automations/air-purifiers-button-3-toggle.yaml`](ha-automations/air-purifiers-button-3-toggle.yaml)

---

### Security Status — Scene Button 4 LED

Monitors all exterior doors, windows, and locks across the house. Updates the Button 4 LED on the scene controller to green when everything is secure (all doors/windows closed, all locks locked), or red if any one is open or unlocked. Fires automatically whenever any monitored entity changes state.

**Prerequisites:** Set `select.scene_controller_led_indicator_button_4` to "Always on" in HA.

**Key entities:**
- `lock.front_door`, `cover.garage_door_1`, `cover.garage_door_2`
- All exterior door and window `binary_sensor` entities
- `select.scene_controller_led_indicator_color_button_4` (green=secure, red=insecure)

**File:** [`ha-automations/security-status-button-4-led.yaml`](ha-automations/security-status-button-4-led.yaml)

---

### Security Lock — Scene Button 4 Press

Triggers on a single tap of Button 4 (lock icon) on the scene controller. If the front door is unlocked, locks it. Does nothing if already locked — this button never unlocks the door. After a 5-second Z-Wave settle delay, checks all exterior doors, windows, and locks and sends a push notification: green if everything is secure, red with an itemized list if anything needs attention.

**Key entities:**
- `event.scene_controller_scene_004` (trigger — Button 4 single tap)
- `lock.front_door`
- All exterior door, window, and lock `binary_sensor` entities
- `notify.household`

**File:** [`ha-automations/security-lock-button-4-press.yaml`](ha-automations/security-lock-button-4-press.yaml)

---

### Pet Feeder Evening Auto Feed

Calls the Feed Pet script at HH:MM AM and HH:MM PM every day of the week. No conditions — fires daily regardless of calendar or presence.

**Key entities:**
- `script.action_feed_pet`

**File:** [`ha-automations/pet-feeder-evening-auto-feed.yaml`](ha-automations/pet-feeder-evening-auto-feed.yaml)

---

### Watchdog — Uptime Kuma Reachable

Pings Uptime Kuma every 5 minutes via the `rest_command.check_uptime_kuma` REST command (targeting `http://198.51.100.30:4040`). Increments `counter.uptime_kuma_failures` on failure. On the second consecutive failure, pushes a warning to the primary phone. On success, resets the counter to 0. Covers the "Uptime Kuma is itself down" blind spot that Slack alerting cannot.

**Key entities:**
- `counter.uptime_kuma_failures` (helper — initial 0, min 0, step 1, restore on startup)
- `rest_command.check_uptime_kuma`
- `notify.mobile_app_phone_1`

**File:** [`ha-automations/watchdog-uptime-kuma-reachable.yaml`](ha-automations/watchdog-uptime-kuma-reachable.yaml)

---

### UPS — On Battery

Fires when `sensor.ups_status_data` transitions away from `OL` and the new state contains `OB`. Records the outage start timestamp in `input_datetime.ups_outage_started`. Sends a Slack message to `#homelab-alerts` with runtime, charge, load, and input voltage.

**Key entities:**
- `sensor.ups_status_data` (raw NUT status — "OL", "OB", "OB LB")
- `input_datetime.ups_outage_started` (helper — stores outage start for duration calc)
- `sensor.ups_battery_runtime`, `sensor.ups_battery_charge`, `sensor.ups_load`, `sensor.ups_input_voltage`
- `notify.slack_homelab_alerts`

**File:** [`ha-automations/ups_state_alerts.yaml`](ha-automations/ups_state_alerts.yaml) (id: `ups-state-on-battery`)

---

### UPS — Shutdown Imminent

Fires when `sensor.ups_battery_runtime` drops below 900 seconds (15 min) while the UPS is on battery. Sends Slack + mobile push. NUT's FSD threshold is 600s (10 min), so this gives ~5 min of human-facing warning ahead of the automated shutdown.

**Key entities:**
- `sensor.ups_battery_runtime`
- `sensor.ups_status_data` (condition guard — only fires if OB)
- `notify.slack_homelab_alerts`, `notify.mobile_app_phone_1`

**File:** [`ha-automations/ups_state_alerts.yaml`](ha-automations/ups_state_alerts.yaml) (id: `ups-state-shutdown-imminent`)

---

### UPS — Power Restored

Fires when `sensor.ups_status_data` transitions to `OL` from a state containing `OB`. Calculates outage duration from `input_datetime.ups_outage_started` and sends a Slack message with duration and current charge.

**Key entities:**
- `sensor.ups_status_data`
- `input_datetime.ups_outage_started` (used to compute outage minutes)
- `notify.slack_homelab_alerts`

**File:** [`ha-automations/ups_state_alerts.yaml`](ha-automations/ups_state_alerts.yaml) (id: `ups-state-power-restored`)

---

### Air Purifiers Off

Turns off both air purifier plugs. Discovered in live HA source during 2026-04-29 migration — not previously documented.

**File:** [`ha-automations/air-purifiers-off.yaml`](ha-automations/air-purifiers-off.yaml)

---

### All Air Purifiers On

Turns on all air purifiers in the house. Discovered in live HA source during 2026-04-29 migration — not previously documented.

**File:** [`ha-automations/air-purifiers-on.yaml`](ha-automations/air-purifiers-on.yaml)

---

## Scripts

### Toggle Monitor TV

Checks if the monitor TV is off or unavailable — if so, sends a Wake-on-LAN magic packet to bring it up. If it's already on, turns it off. Used as a manual dashboard toggle and called by the workday schedule automation.

**Key entities:**
- `media_player.tv_1`
- WoL MAC: `aa:bb:cc:dd:ee:05`

**File:** [`ha-scripts/toggle-monitor-tv.yaml`](ha-scripts/toggle-monitor-tv.yaml)

---

### Toggle Living Room TV

Same pattern as Toggle Monitor TV — WoL on if off/unavailable, turn off if on. Manual dashboard button for the living room TV.

**Key entities:**
- `media_player.tv_2`
- WoL MAC: `aa:bb:cc:dd:ee:06`

**File:** [`ha-scripts/toggle-living-room-tv.yaml`](ha-scripts/toggle-living-room-tv.yaml)

---

### Feed Pet

Triggers one manual feed portion on the automatic pet feeder by setting the manual feed number entity to 2. Called by the Pet Feeder Evening Auto Feed automation and available as a manual dashboard button.

**Key entities:**
- `number.pet_feeder_manual_feed`

**File:** [`ha-scripts/feed-pet.yaml`](ha-scripts/feed-pet.yaml)
