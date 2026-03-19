# Oref Alert Payload Reference

This document shows the exact data structures from both the **raw Pikud HaOref API** (as scraped from `oref.org.il`) and the **Home Assistant Oref Alert integration** entities. All examples are from real alert events.

---

## 1. Raw Pikud HaOref API

### Active Alerts Endpoint

**URL:** `https://www.oref.org.il/WarningMessages/alert/alerts.json`

**Headers required:**
```
Referer: https://www.oref.org.il/
X-Requested-With: XMLHttpRequest
```

**Response when alerts are active** — returns a JSON array of alert objects:

```json
[
  {
    "data": "כרמיאל",
    "category": 1,
    "title": "ירי רקטות וטילים",
    "desc": "היכנסו למרחב המוגן",
    "alertDate": "",
    "presumed": true,
    "alertStartTime": 1773933140.9630408
  },
  {
    "data": "סכנין",
    "category": 1,
    "title": "ירי רקטות וטילים",
    "desc": "היכנסו למרחב המוגן",
    "alertDate": "",
    "presumed": true,
    "alertStartTime": 1773933134.8368363
  }
]
```

**Response when no alerts are active** — returns an empty string (not `[]`):

```
(empty response body)
```

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `data` | string | Area/city name (Hebrew) |
| `category` | integer | Alert type (see category table below) |
| `title` | string | Alert title (Hebrew), e.g., "ירי רקטות וטילים" = Rocket and missile fire |
| `desc` | string | Instruction (Hebrew), e.g., "היכנסו למרחב המוגן" = Enter the protected space |
| `alertDate` | string | Alert date (may be empty for live alerts) |
| `presumed` | boolean | Whether this is a presumed/inferred alert |
| `alertStartTime` | float | Unix timestamp when the alert started |

### Alert History Endpoint

**URL:** `https://www.oref.org.il/warningMessages/alert/History/AlertsHistory.json`

**Headers required:** Same as above.

**Response** — returns a JSON array of historical alert objects:

```json
[
  {
    "alertDate": "2026-03-19 17:12:48",
    "title": "ירי רקטות וטילים",
    "data": "יובלים",
    "category": 1
  },
  {
    "alertDate": "2026-03-19 17:12:48",
    "title": "ירי רקטות וטילים",
    "data": "מג'דל כרום",
    "category": 1
  },
  {
    "alertDate": "2026-03-19 17:12:08",
    "title": "ירי רקטות וטילים",
    "data": "סכנין",
    "category": 1
  }
]
```

**Note:** The history endpoint has fewer fields than the active alerts endpoint — no `desc`, `presumed`, or `alertStartTime`.

### Alert Categories

| Category | Meaning (Hebrew) | Meaning (English) |
|----------|-------------------|-------------------|
| 1 | ירי רקטות וטילים | Rocket and missile fire |
| 2 | חדירת כלי טיס עוין | Hostile aircraft intrusion |
| 3 | רעידת אדמה | Earthquake |
| 4 | צונאמי | Tsunami |
| 5 | חומרים מסוכנים | Hazardous materials |
| 6 | חדירת מחבלים | Terrorist infiltration |
| 13 | האירוע הסתיים | Event over (all clear) |

---

## 2. Home Assistant Oref Alert Integration

The [Oref Alert integration](https://github.com/amitfin/oref_alert) creates several entities. Here are the key ones with their full state and attribute structures.

### `binary_sensor.oref_alert`

The primary entity for triggering automations. Goes `on` when an active alert is declared for your configured area(s).

**State during active alert:**

```json
{
  "entity_id": "binary_sensor.oref_alert",
  "state": "on",
  "attributes": {
    "areas": ["ירושלים - מרכז"],
    "selected_areas_active_alerts": [
      {
        "data": "ירושלים - מרכז",
        "category": 1,
        "channel": "tzevaadom",
        "alertDate": "2026-03-19 16:50:57",
        "title": "ירי רקטות וטילים"
      }
    ],
    "selected_areas_updates": [
      {
        "data": "ירושלים - מרכז",
        "category": 1,
        "channel": "mobile",
        "alertDate": "2026-03-19 16:39:19",
        "title": "בדקות הקרובות צפויות התרעות באזורך"
      }
    ],
    "country_active_alerts": [
      {
        "data": "אשבל",
        "category": 1,
        "channel": "tzevaadom",
        "alertDate": "2026-03-19 17:12:48",
        "title": "ירי רקטות וטילים"
      },
      {
        "data": "כרמיאל",
        "category": 1,
        "channel": "tzevaadom",
        "alertDate": "2026-03-19 17:12:12",
        "title": "ירי רקטות וטילים"
      }
    ],
    "country_updates": [],
    "device_class": "safety",
    "friendly_name": "Oref Alert"
  },
  "last_changed": "2026-03-19T14:50:58.260745+00:00",
  "last_updated": "2026-03-19T15:12:49.219231+00:00"
}
```

**State when all clear:**

```json
{
  "entity_id": "binary_sensor.oref_alert",
  "state": "off",
  "attributes": {
    "areas": ["ירושלים - מרכז"],
    "selected_areas_active_alerts": [],
    "selected_areas_updates": [
      {
        "data": "ירושלים - מרכז",
        "category": 13,
        "channel": "mobile",
        "alertDate": "2026-03-19 17:01:05",
        "title": "האירוע הסתיים"
      }
    ],
    "country_active_alerts": [],
    "country_updates": [],
    "device_class": "safety",
    "friendly_name": "Oref Alert"
  }
}
```

**Key attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| `areas` | array of strings | Your configured area(s) |
| `selected_areas_active_alerts` | array of objects | Active alerts for your area(s) only |
| `selected_areas_updates` | array of objects | Updates for your area(s) — includes pre-warnings and "event over" messages |
| `country_active_alerts` | array of objects | All active alerts across Israel (used by the 100+ mass alert automation) |
| `country_updates` | array of objects | Updates for all of Israel |

**Alert object fields (integration format):**

| Field | Type | Description |
|-------|------|-------------|
| `data` | string | Area/city name (Hebrew) |
| `category` | integer | Alert type (same categories as raw API) |
| `channel` | string | Source channel: `"tzevaadom"` (Tzeva Adom / Red Color) or `"mobile"` |
| `alertDate` | string | Alert date/time as `"YYYY-MM-DD HH:MM:SS"` |
| `title` | string | Alert title (Hebrew) |

### `sensor.oref_alert`

Enum sensor with three states for your configured area.

```json
{
  "entity_id": "sensor.oref_alert",
  "state": "ok",
  "attributes": {
    "options": ["ok", "pre_alert", "alert"],
    "area": "ירושלים - מרכז",
    "record": {
      "data": "ירושלים - מרכז",
      "category": 13,
      "channel": "mobile",
      "alertDate": "2026-03-19 17:01:05",
      "title": "האירוע הסתיים"
    },
    "device_class": "enum",
    "friendly_name": "Oref Alert"
  }
}
```

**States:**

| State | Meaning | Triggers when |
|-------|---------|---------------|
| `ok` | No alert | Alert ends, or no activity |
| `pre_alert` | Incoming warning | `selected_areas_updates` contains "בדקות הקרובות" (="in the coming minutes") |
| `alert` | Active alert | `binary_sensor.oref_alert` goes `on` |

### `event.oref_alert`

Event entity that fires on each state transition.

```json
{
  "entity_id": "event.oref_alert",
  "state": "2026-03-19T15:01:08.817+00:00",
  "attributes": {
    "event_types": ["pre_alert", "end", "alert"],
    "event_type": "end",
    "record": {
      "data": "ירושלים - מרכז",
      "category": 13,
      "channel": "mobile",
      "alertDate": "2026-03-19 17:01:05",
      "title": "האירוע הסתיים"
    },
    "friendly_name": "Oref Alert"
  }
}
```

**Event types:**

| Event Type | Meaning |
|-----------|---------|
| `pre_alert` | Warning — alerts expected in your area soon |
| `alert` | Active alert declared for your area |
| `end` | Alert over / all clear |

---

## 3. Comparison: Raw API vs. Integration

| Aspect | Raw API (`oref.org.il`) | HA Integration (`oref_alert`) |
|--------|------------------------|-------------------------------|
| **Data format** | Raw JSON array from HTTP endpoint | HA entity state + attributes |
| **Empty response** | Empty string (not `[]`) | `state: "off"`, empty arrays |
| **Alert object fields** | `data`, `category`, `title`, `desc`, `alertDate`, `presumed`, `alertStartTime` | `data`, `category`, `title`, `channel`, `alertDate` |
| **Extra fields (raw only)** | `desc` (instruction text), `presumed`, `alertStartTime` (unix float) | — |
| **Extra fields (integration only)** | — | `channel` (`tzevaadom` / `mobile`) |
| **Area filtering** | None — returns all active alerts in Israel | `selected_areas_*` for your area, `country_*` for all Israel |
| **Pre-alert detection** | Must poll and infer | Built-in `pre_alert` state + `selected_areas_updates` with "בדקות הקרובות" |
| **All clear** | Must detect when array becomes empty | `binary_sensor` goes `off`, category 13 "האירוע הסתיים", `event_type: "end"` |
| **History** | Separate endpoint (`AlertsHistory.json`) | HA recorder history |
| **Update frequency** | As fast as you poll (typically 1–3s) | Integration polls every few seconds |
| **BOM** | Response may have UTF-8 BOM (`\uFEFF`) — strip before parsing | Handled by integration |

---

## 4. Key Hebrew Strings for Template Conditions

| Hebrew | Transliteration | English | Used for |
|--------|----------------|---------|----------|
| בדקות הקרובות | b'dakot ha'krovot | In the coming minutes | Pre-alert detection in `selected_areas_updates` |
| ירי רקטות וטילים | yeri raketot v'tilim | Rocket and missile fire | Category 1 alert title |
| היכנסו למרחב המוגן | hikkansu la'merhav ha'mugan | Enter the protected space | Alert instruction (raw API `desc`) |
| האירוע הסתיים | ha'iru'a histayem | The event is over | All clear (category 13) |
| חדירת כלי טיס עוין | hadirat kli tis oyen | Hostile aircraft intrusion | Category 2 alert title |
| חדירת מחבלים | hadirat mekhablim | Terrorist infiltration | Category 6 alert title |
