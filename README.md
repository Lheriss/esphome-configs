# esphome-configs

Shared [ESPHome](https://esphome.io) packages for my devices. Each device keeps a short local file (name, calibration, secrets), and everything common lives here, versioned with Git tags.

## Layout

| Path | Content |
|---|---|
| `common/base.yaml` | Shared by every node: logger (level selectable from Home Assistant), encrypted OTA, fallback AP + captive portal, web server v3, diagnostics (status, Wi-Fi dBm/%, IP, uptime, reset reason), restart / safe-mode buttons |
| `common/ble-proxy-bk72xx.yaml` | *Optional.* Passive Bluetooth proxy for BK72xx chips with BLE 5.x (BK7231N / CB2S…), ESPHome ≥ 2026.8: forwards advertisements (BTHome, ATC thermometers, beacons), no GATT connections |
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
    ref: lspa8-v1.1.0
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
2. **Reference load** — use the *active power* increase on that phase when the load switches on, `ΔP_meter`, **not the current increase**. Active power adds up; current magnitudes do not: with a reactive background on the phase (PF ≈ 0.6, 1–1.5 A here), `ΔI` underestimated the kettle current by about 5 %.
   - kettle current: `I_ref = ΔP_meter / V_meter_load` (meter voltage under load, PF = 1);
   - wiring losses between meter and plug: `P_loss = (V_meter_load − V_plug_load) × I_ref`, with `V_plug_load` already corrected by step 1;
   - power at the plug: `P_ref = ΔP_meter − P_loss`.
3. **Power, then current** — power scales as `voltage_divider / current_resistor`, current as `current_multiply / current_resistor`:
   - `current_resistor_new = current_resistor × (P_plug / P_ref) × (voltage_divider_new / voltage_divider)`;
   - `current_multiply_new = current_multiply × (I_ref / I_plug) × (current_resistor_new / current_resistor)`.
   - Check: after correction, `P ≈ V × I` at the plug (PF ≈ 1.00).
4. Run 2 or 3 on/off cycles of ~30 s, average the steady samples (skip the first one after switching), and discard any cycle where another appliance switched on.

Notes: compare voltage at no load only — under load, wiring between the meter and the plug drops several volts. Set the plug's log level to DEBUG from Home Assistant during the test (no restart needed) and keep its log: plug voltage and current are usually not in the recorder.

## Hardware notes — LSPA8 (A8TYV1.3 / CB2S)

| CB2S pin | Function |
|---|---|
| P26 | Relay + red LED (same pin) |
| P8 | Blue LED (independent → status LED) |
| P10 | Button (also RX1) |
| P6 / P7 / P24 | BL0937 CF1 / CF / SEL |

Flashing (UART, `ltchiptool`): firmware 1.1.17 is patched against tuya-cloudcutter. Use a 3.3 V source able to supply the inrush current. A PL2303 adapter was not enough; an ESP32 DevKit held in reset (EN→GND), used as USB-serial bridge (TX→TX1, RX→RX1, no crossover) and 3.3 V supply, worked. Back up the 2 MB original firmware first (`ltchiptool flash read … BK7231N`).

Reference: <https://devices.esphome.io/devices/tuya-smart-plug-16a-il/>
