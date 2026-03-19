# Home Assistant Oref Alert Automations

A set of Home Assistant automations for the [Oref Alert](https://github.com/amitfin/oref_alert) integration that provide TTS (text-to-speech) announcements, light control, and notifications during red alert events in Israel.

These automations have been tested in production during real alert events.

## What's Included

| Automation | Trigger | What It Does |
|---|---|---|
| **Preemptive Warning** | `pre_alert` state (incoming alert for nearby areas) | Turns on lights, TTS: "Red Alert. Warning.", activates sirens |
| **Active Alert** | `binary_sensor.oref_alert` turns `on` | Snapshots light state, TTS: "Red Alert. Active Alert. Seek shelter immediately.", plays alert audio, restores lights after 5 min |
| **All Clear** | `binary_sensor.oref_alert` turns `off` | TTS: "All clear. The alert is over.", plays all-clear audio |
| **Mass Alert** | 100+ simultaneous alerts nationwide | TTS: "More than 100 alerts active across Israel", persistent notification |

## How the Oref Alert Entity Works

The [Oref Alert integration](https://github.com/amitfin/oref_alert) provides several entities. The key ones used by these automations:

| Entity | States | Description |
|---|---|---|
| `sensor.oref_alert` | `ok`, `pre_alert`, `alert` | Enum sensor for your configured area |
| `binary_sensor.oref_alert` | `on` / `off` | Safety binary sensor; `on` = active alert for your area |

The `binary_sensor.oref_alert` also has a `country_active_alerts` attribute containing a list of all currently active alerts across Israel, which the mass alert automation uses.

### Alert Lifecycle

```
ok → pre_alert → alert → ok
         │                 ▲
         └─────────────────┘  (if alert doesn't reach your area)
```

- **`pre_alert`**: Alerts are active in nearby areas; yours may be next. The `selected_areas_updates` attribute contains updates with "בדקות הקרובות" (Hebrew for "in the coming minutes") in the title or text.
- **`alert`**: Active alert declared for your configured area. Take shelter.
- **`ok`**: All clear.

## Recommended Hardware

A minimal setup for whole-home TTS alert coverage:

| Item | Purpose | Notes |
|------|---------|-------|
| **1x Raspberry Pi** (3B+ or newer) | Runs Home Assistant | Pi 4 or Pi 5 recommended for better performance. Use an SSD instead of SD card for reliability. |
| **1x Speaker** connected to the Pi | TTS announcements | Any powered speaker connected via 3.5mm audio jack or USB. For better coverage, use a Bluetooth speaker or a network speaker (e.g., Google Home / Nest) as the media player target. |
| **Zigbee smart light bulbs** | Visual alert indicators | Any Zigbee-compatible color bulbs (e.g., IKEA TRADFRI, Philips Hue, Sonoff). The automations turn lights on during pre-alerts and can flash colors during active alerts. |
| **Zigbee coordinator** | Connects Zigbee devices to HA | USB stick such as SONOFF Zigbee 3.0 Dongle Plus (ZBDongle-E) or ConBee II. Required for Zigbee bulbs. |

**Optional additions:**

- **Zigbee sirens** (e.g., HEIMAN HS2WD-E) — for audible siren alerts in addition to TTS
- **Additional speakers** in different rooms — use HA speaker groups for simultaneous announcements
- **UPS / battery backup** — keeps the alert system running during power outages (common during escalations)

## Prerequisites

1. **Home Assistant** (2024.1+)
2. **[Oref Alert integration](https://github.com/amitfin/oref_alert)** installed and configured with your area(s)
3. **A TTS platform** configured (e.g., Google Translate TTS, Google Cloud TTS, Piper, etc.)
4. **A media player** for announcements (e.g., Google Home, Nest speaker, or any HA media player)

## Installation

### Option 1: Copy YAML into your automations

1. Open each YAML file in the `automations/` directory
2. Copy the contents into your `automations.yaml` file (or your split automation files)
3. Replace placeholder entities with your own (see [Customization](#customization))
4. Reload automations in Home Assistant (Developer Tools > YAML > Reload Automations)

### Option 2: Use the Home Assistant UI

1. Go to **Settings > Automations & Scenes > Create Automation**
2. Switch to YAML mode (three-dot menu > Edit in YAML)
3. Paste the contents of each automation file
4. Replace placeholder entities with your own
5. Save

### Option 3: Use the automation directory (advanced)

If you use split configuration, place the files in your automations directory and reference them from `configuration.yaml`:

```yaml
automation: !include_dir_list automations/
```

## Customization

Each automation file contains placeholder entities that you need to replace with your own:

| Placeholder | Replace With |
|---|---|
| `media_player.your_speaker` | Your media player entity (e.g., `media_player.living_room_speaker`) |
| `tts.google_en_com` | Your TTS entity (e.g., `tts.piper`, `tts.google_cloud`) |
| `light.your_*` | Your light entities |
| `siren.your_siren` | Your siren entities (or remove the siren action) |
| `your-alert-sound.mp3` | Path to your alert audio file in HA media |
| `your-all-clear-sound.mp3` | Path to your all-clear audio file in HA media |

### Optional Features

These actions can be removed if not needed:

- **Siren activation** (preemptive warning) — remove if you don't have smart sirens
- **Audio file playback** (active alert & all clear) — remove if you only want TTS
- **Light snapshot/restore** (active alert) — remove if you don't want light control
- **MQTT publish** — removed in the template versions; add back if you use MQTT for external integrations

### Adjusting the Mass Alert Threshold

The mass alert automation triggers at 100+ simultaneous alerts. To change this, edit the template trigger in `mass_alert.yaml`:

```yaml
value_template: >-
  {{ state_attr('binary_sensor.oref_alert', 'country_active_alerts') | length > 50 }}
```

## TTS Messages

| Automation | Message |
|---|---|
| Preemptive Warning | "Red Alert. Warning. Red Alert. Warning." |
| Active Alert | "Red Alert. Active Alert. Red Alert. Active Alert. Seek shelter immediately." |
| All Clear | "All clear. The alert is over. All clear." |
| Mass Alert | "Attention. More than 100 alerts are currently active across Israel." |

You can customize these messages in each automation's `tts.speak` action.

## How It Works in Practice

During an alert cycle, the automations fire in sequence:

1. **Pre-alert fires first** — lights turn on, TTS warns "Red Alert. Warning.", sirens activate
2. **If the alert reaches your area** — active alert fires with "Seek shelter immediately" and plays the alert audio
3. **When the alert ends** — all clear fires with "The alert is over" and plays the all-clear audio

The active alert automation uses `mode: restart`, so if a new alert arrives while the 5-minute light restore timer is running, it resets the timer.

## License

MIT

## Credits

- [Oref Alert integration](https://github.com/amitfin/oref_alert) by Amit Finkelstein
- Automations by [Daniel Rosehill](https://danielrosehill.com)
