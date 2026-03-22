# Oref Red Alert Automation System for Home Assistant

A comprehensive Home Assistant automation system for Israel's Pikud HaOref (Home Front Command) rocket alert system. Provides multi-sensory alerts through PA announcements with pre-recorded audio, RGB light color changes, light flickering, and siren activation.

These automations have been tested in production during real alert events.

## What This Does

When a red alert is triggered via the [Oref Alert integration](https://github.com/amitfin/oref_alert), these automations:

1. **Pause any currently playing media** on your PA/announcement system, disable repeat mode, and clear the playlist to prevent unwanted audio resumption
2. **Play an airport-style chime** followed by a **pre-recorded voice announcement**
3. **Change all RGB lights** to the alert color (red, orange, or green)
4. **Flicker lights** and **activate sirens** for the duration of the alert phase
5. **Hold the alert color** for 3 minutes so you know the alert state at a glance
6. **Restore all lights** to their pre-alert state using scene snapshots
7. **Resume media playback** on the PA system

## Automations

### Real Alerts (4 automations)

| File | Trigger | Color | Sirens | Duration |
|------|---------|-------|--------|----------|
| `red_alert_early_warning.yaml` | Oref "next few minutes" preemptive warning | Orange | Yes | 30s flicker, 3 min hold |
| `red_alert_active.yaml` | `binary_sensor.oref_alert` turns on | Red | Yes | 30s flicker, 3 min hold |
| `red_alert_all_clear.yaml` | `binary_sensor.oref_alert` turns off | Green | No | 3 min hold |
| `red_alert_100_plus.yaml` | 100+ simultaneous alerts across Israel | N/A | No | PA announcement only |

### Test Automations (3 automations)

| File | Color | Sirens | Duration |
|------|-------|--------|----------|
| `test_early_warning.yaml` | Orange | Yes | 15s countdown, 10s flicker, immediate restore |
| `test_active_red_alert.yaml` | Red | Yes | 15s countdown, 10s flicker, immediate restore |
| `test_all_clear.yaml` | Green | No | 15s countdown, 10s hold, immediate restore |

Test automations have no trigger -- run them manually from the Home Assistant UI to verify your setup works without waiting for a real alert.

## How the Oref Alert Entity Works

The [Oref Alert integration](https://github.com/amitfin/oref_alert) provides several entities. The key ones used by these automations:

| Entity | States | Description |
|---|---|---|
| `binary_sensor.oref_alert` | `on` / `off` | Safety binary sensor; `on` = active alert for your area |

The `binary_sensor.oref_alert` also has:
- A `country_active_alerts` attribute containing all currently active alerts across Israel (used by the mass alert automation)
- A `selected_areas_updates` attribute containing preemptive warnings with "בדקות הקרובות" (Hebrew for "in the coming minutes")

### Alert Lifecycle

```
Normal → Pre-warning → Active Alert → All Clear
                │                        ▲
                └────────────────────────┘  (if alert doesn't reach your area)
```

## Requirements

- **[Oref Alert integration](https://github.com/amitfin/oref_alert)** -- provides `binary_sensor.oref_alert` and area-specific alert data
- **Multi-room audio system** -- tested with [Snapcast](https://github.com/badaix/snapcast) via [Music Assistant](https://music-assistant.io/), but any media_player entities that support `announce: true` should work
- **RGB-capable smart lights** -- grouped into a single light group for color changes
- **Smart sirens** (optional) -- Zigbee/Z-Wave sirens for audible alerts
- **Pre-recorded audio files** -- included in the `audio/` directory

## Audio Files

All PA announcement audio files are in `audio/pa-messages/`. These were generated using **OpenAI TTS** with the **onyx** voice.

The airport chime sound effect (`pa_chime_2.mp3`) is not included in this repository. The original chime used is by **Gustavo Rezende** and can be found on [Pixabay](https://pixabay.com/) -- search for "airport call" or use a similar chime sound. Place it in your Home Assistant media directory at the path referenced in the automations.

### Audio file list

| File | Used In | Description |
|------|---------|-------------|
| `pa_early_warning.mp3` | Early warning | Preemptive warning announcement |
| `pa_active_alert.mp3` | Active alert | Active red alert announcement |
| `pa_all_clear.mp3` | All clear | Alert over announcement |
| `pa_mass_alert.mp3` | 100+ alerts | Mass alert announcement |
| `pa_test_early_warning_countdown.mp3` | Test early warning | 15-second countdown before test |
| `pa_test_early_warning_alert.mp3` | Test early warning | Test alert announcement |
| `pa_test_early_warning_concluded.mp3` | Test early warning | Test concluded announcement |
| `pa_test_active_alert_countdown.mp3` | Test active alert | 15-second countdown before test |
| `pa_test_active_alert_alert.mp3` | Test active alert | Test alert announcement |
| `pa_test_active_alert_concluded.mp3` | Test active alert | Test concluded announcement |
| `pa_test_all_clear_countdown.mp3` | Test all clear | 15-second countdown before test |
| `pa_test_all_clear_alert.mp3` | Test all clear | Test alert announcement |
| `pa_test_all_clear_concluded.mp3` | Test all clear | Test concluded announcement |

## How to Customize

Every automation file includes `# CUSTOMIZE` comments at the points where you need to update entity IDs.

### Entity IDs to Replace

1. **Light entities** -- Replace `light.living_room`, `light.bedroom`, etc. with your actual light entity IDs in:
   - Scene snapshot lists (all lights you want to save/restore)
   - Flicker target lists (lights that turn on/off rapidly during alerts)

2. **Light group** -- Replace `light.rgb_lights_group` with your own light group containing all RGB-capable lights

3. **Media player entities** -- Replace `media_player.speaker_1_snapcast`, `media_player.pa_system`, etc. with your actual media player entities

4. **Siren entities** -- Replace `siren.siren_1`, `siren.siren_2`, `siren.siren_3` with your actual siren entity IDs (or remove siren actions if you don't have smart sirens)

### Media Paths

The automations reference audio files at:
- Chime: `media-source://media_source/local/audio/soundfx/pa_chime_2.mp3`
- Messages: `media-source://media_source/local/audio/pa-messages/pa_*.mp3`

Upload the audio files from this repo's `audio/pa-messages/` directory to your Home Assistant's media directory, then update the paths in the automations if your directory structure differs.

### Optional Integrations

The real alert automations include `# TODO` comments where you can add:
- **MQTT publishing** for alert state tracking on other systems
- **REST commands / webhooks** for external notifications (e.g., push notifications, logging)
- **Custom scripts** (e.g., a dramatic siren ramp-up script)

### About the Mass Alert Threshold (100+)

The mass alert automation triggers at 100+ simultaneous alerts across Israel. This threshold has been validated through real-world observation: from experience watching alert patterns, 100 is a reliable yardstick for a ballistic missile attack, even if it's not happening in your immediate area. Fewer than 50 simultaneous alerts tends to indicate a localized event (e.g., Hezbollah rockets targeting a specific region), whereas 100+ almost always indicates a ballistic missile strike (e.g., from Iran or Yemen). This distinction matters because ballistic missiles have a wider threat radius and longer flight time, giving you more reason to take shelter even if the alert isn't for your specific area.

To adjust this threshold, edit the template trigger in `red_alert_100_plus.yaml`:

```yaml
value_template: "{{ state_attr('binary_sensor.oref_alert', 'country_active_alerts') | length > 50 }}"
```

## Installation

1. Copy the YAML files from `automations/` into your Home Assistant automations directory (or paste them into the automation editor in YAML mode)
2. Upload the audio files from `audio/pa-messages/` to your Home Assistant media directory
3. Source an airport chime sound and place it at the chime path referenced in the automations
4. Update all `# CUSTOMIZE` entity IDs to match your setup
5. Reload automations in Home Assistant (Developer Tools > YAML > Reload Automations)
6. Run each test automation manually to verify everything works

## How It Works in Practice

During an alert cycle, the automations fire in sequence:

1. **Pre-warning fires first** -- lights turn orange, chime + PA announcement plays, sirens activate, lights flicker for 30 seconds
2. **If the alert reaches your area** -- lights turn red, chime + PA announcement plays, sirens activate, lights flicker for 30 seconds
3. **When the alert ends** -- lights turn green, chime + PA announcement plays, green holds for 3 minutes
4. **After 3 minutes** -- all lights restore to their pre-alert state, media playback resumes

The active alert and early warning automations use `mode: restart`, so if a new alert arrives while the 3-minute hold timer is running, it resets.

## License

MIT

## Credits

- **[Oref Alert integration](https://github.com/amitfin/oref_alert)** by Amit Finkelstein
- **Airport chime sound** by [Gustavo Rezende](https://pixabay.com/) via Pixabay
- **PA voice announcements** generated with OpenAI TTS (onyx voice)
- **Automations** by [Daniel Rosehill](https://danielrosehill.com)
