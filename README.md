# # Whirlpool Control Card (Aquard fork)

This is a fork of [Aquard](https://github.com/sorz/aquard) — a profile-driven
water monitoring card for Home Assistant — adapted for a Bestway/LayZSpa
whirlpool with added time-based scheduling: automatic pre-heating ahead of
calendar events, filtration tracking, and dosing recommendation badges.

I've uploaded the card under the link above. I wasn't quite sure how to do
it any differently from my phone, so consider this a provisional upload for
now — screenshots of the redesigned card will follow shortly, and I'm still
restructuring parts of it.

## What's different from the original Aquard card

- Configurable pH tolerance zones
- Dosing recommendation badges
- Animated heating visuals
- Calendar-based pre-heating logic
- Filtration time tracking, decoupled from the heating logic
- A visual editor for the card

## Required Helpers

The scheduling features above rely on a set of Home Assistant helpers that
are **not created automatically** by installing the card — a Lovelace card
only adds a frontend resource, it can't create backend entities on its own.
You need to set these up once before the card will show correct data.

### Option A: YAML package (recommended)

1. Make sure packages are enabled in your `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

2. Copy [`packages/aquard_whirlpool.yaml`](./packages/aquard_whirlpool.yaml)
   from this repo into your own `packages/` folder.
3. Open the file and adjust the entity references at the top to match your
   own setup (your calendar entity, your spa's climate entity, your pH/ORP
   sensors, and your spa's water volume).
4. Restart Home Assistant (packages require a restart, not just a reload).

### Option B: Manual setup

If you don't use packages, create these helpers manually under
**Settings → Devices & Services → Helpers**, or via
**Settings → Automations & Scenes → Template** for the template entities.

| Entity | Type | Purpose |
|---|---|---|
| `input_boolean.whirlpool_auto_aktiv` | Boolean | Enables full automatic mode |
| `input_boolean.whirlpool_manuell_heizen` | Boolean | Manual heating override |
| `input_boolean.whirlpool_kalender_vorheizen_aktiv` | Boolean | Enables calendar-based pre-heating |
| `input_number.whirlpool_filter_soll_minuten` | Number | Target daily filter runtime (min) |
| `input_number.whirlpool_filter_credit_heute` | Number | Filter minutes already completed today |
| `input_number.whirlpool_vorheiz_reserve_minuten` | Number | Safety buffer before a calendar event (min) |
| `input_number.whirlpool_heizrate_grad_pro_stunde` | Number | Heating rate used for ETA calculations (°C/h) |
| `input_number.whirlpool_heizen_auf_zeit_dauer` | Number | Duration for "heat until fixed time" mode (min) |
| `timer.whirlpool_filtern_timer` | Timer | Tracks active filtration runtime |
| `timer.whirlpool_heizen_auf_zeit_timer` | Timer | Tracks the scheduled heat-by-time countdown |
| `sensor.whirlpool_filter_rest_minuten` | Template sensor | Remaining filter minutes today |
| `sensor.whirlpool_naechster_termin_start` | Template sensor | Start time of next calendar event |
| `sensor.whirlpool_vorheiz_startzeit` | Template sensor | Calculated pre-heat start time |
| `sensor.whirlpool_naechste_aktion` | Template sensor | Next scheduled action (human-readable) |
| `sensor.whirlpool_zeit_bis_fertig` | Template sensor | Time remaining until target temperature (h) |
| `binary_sensor.whirlpool_vorheizen_noetig` | Template binary sensor | Whether pre-heating should be active right now |
| `binary_sensor.whirlpool_filter_nachholen_noetig` | Template binary sensor | Whether filtering still needs to run today |
| `binary_sensor.whirlpool_wasserpflege_noetig` | Template binary sensor | Whether pH/chlorine dosing is due |
| `sensor.whirlpool_ph_geglättet` | Filter sensor (1-min moving average) | Smoothed pH value |
| `sensor.whirlpool_orp_geglättet` | Filter sensor (1-min moving average) | Smoothed ORP value |

Exact template logic and default values are documented inline in
[`packages/aquard_whirlpool.yaml`](./packages/aquard_whirlpool.yaml) — the
manual-setup values in the table above should match that file exactly.

### After setup

Once the helpers exist, restart (or reload templates via
**Developer Tools → YAML → Templates**, plus a full restart for packages),
add the card to your dashboard, and point its configuration at the entities
above.

## Screenshots

Coming soon — I'll add screenshots of the current card here once the
redesign is finished.
