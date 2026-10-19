# esphome-configs

Shared [ESPHome](https://esphome.io) packages for my devices. Each device keeps a short local file (name, calibration, secrets), and everything common lives here, versioned with Git tags.

## Layout

| Path | Content |
|---|---|
| `common/base.yaml` | Shared by every node: logger (level selectable from Home Assistant), encrypted OTA, fallback AP + captive portal, web server v3, diagnostics (status, Wi-Fi dBm/%, IP, uptime, reset reason), restart / safe-mode buttons |
| `devices/lspa8-cb2s.yaml` | Tuya LSPA8 smart plug (board A8TYV1.3, CB2S / BK7231N): relay, button, status LED, BL0937 power metering, over-current cut-off, power-on state selector |
| `examples/` | Local device files to copy into ESPHome Builder |
| `tests/` | CI configs with dummy secrets |

## Usage

Copy `examples/lspa8-cb2s.yaml` into `/config/esphome/`, rename it and adapt the substitutions:

```yaml
substitutions:
  device_name: prise-bouilloire
  friendly_name: PriseBouilloire
  voltage_divider: "780"
  current_resistor: "0.00113 ohm"
  current_multiply: "0.436"

packages:
  lspa8:
    url: https://github.com/Lheriss/esphome-configs
    ref: lspa8-v1.0.0
    files: [common/base.yaml, devices/lspa8-cb2s.yaml]
    refresh: 1d

api:
  encryption:
    key: !secret api_encryption_key
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

### Rules

- **No secrets here.** ESPHome refuses `!secret` in remote packages. `api:` and `wifi:` credentials stay in the local file.
- **Local substitutions win** over the package defaults: a device only declares what differs.
- **Pin a tag** (`ref:`), never `main`. A change reaches a device only when its `ref` is bumped and it is recompiled.

## Versioning

One repository, several device types, so tags are **prefixed by type**: `lspa8-v1.0.0`, later e.g. `<type>-v1.0.0`. A device pins the tag of its own type. A change to `common/base.yaml` is released as a new version of each type when it is adopted. `esphome.project.version` in the device package matches the tag, so Home Assistant shows which version each device runs.

See [CHANGELOG.md](CHANGELOG.md).

## Calibration (BL0937)

Each BL0937 has its own offsets; calibrate every plug against a trusted reference, ideally the utility meter (here a gPlugM reading the Landis+Gyr E450 per phase), with a purely resistive load (kettle):

1. **Voltage** — compare no-load voltage on the plug's phase: `voltage_divider_new = voltage_divider × V_ref / V_plug`.
2. **Current** — compare the current *increase* on that phase when the load switches on with the plug reading. Power scales as `voltage_divider / current_resistor`, current as `current_multiply / current_resistor`:
   - fix power first: `current_resistor_new = current_resistor × (P_plug / P_ref) × (voltage_divider_new / voltage_divider)` (with `P_ref = V_plug_ref × I_ref`, PF = 1, `V_plug_ref` being the corrected voltage at the plug under load);
   - then current: `current_multiply_new = current_multiply × (I_ref / I_plug) × (current_resistor_new / current_resistor)`.
3. Repeat 2–3 on/off cycles of ~30 s and discard any cycle where another appliance switched on.

Note: compare voltage at no load only. Under load, wiring between the meter and the plug drops several volts.

## Hardware notes — LSPA8 (A8TYV1.3 / CB2S)

| CB2S pin | Function |
|---|---|
| P26 | Relay + red LED (same pin) |
| P8 | Blue LED (independent → status LED) |
| P10 | Button (also RX1) |
| P6 / P7 / P24 | BL0937 CF1 / CF / SEL |

Flashing (UART, `ltchiptool`): firmware 1.1.17 is patched against tuya-cloudcutter. Use a 3.3 V source able to supply the inrush current. A PL2303 adapter was not enough; an ESP32 DevKit held in reset (EN→GND), used as USB-serial bridge (TX→TX1, RX→RX1, no crossover) and 3.3 V supply, worked. Back up the 2 MB original firmware first (`ltchiptool flash read … BK7231N`).

Reference: <https://devices.esphome.io/devices/tuya-smart-plug-16a-il/>
