# Changelog

## lspa8-v1.1.0 — 2026-10-20

- New optional package `common/ble-proxy-bk72xx.yaml`: **passive** Bluetooth proxy for BK72xx chips with a BLE 5.x stack (BK7231N / CB2S…), ESPHome ≥ 2026.8. Advertisement forwarding only (no GATT connections); scan runs only while Home Assistant is connected; scan window 30 ms every 320 ms to leave the single CPU core and radio to Wi-Fi. Add it to `files:` on the devices that should act as proxies.
- `common/base.yaml`: firmware upload through the web page disabled (`web_server: ota: false`) — its `/update` endpoint was neither authenticated nor encrypted. Updates go through the encrypted ESPHome OTA.
- CI: also validates `tests/lspa8-cb2s-ble.yaml`.

## lspa8-v1.0.1 — 2026-10-19

- `devices/lspa8-cb2s.yaml`: default over-current cut-off lowered from 16 A to **10 A**, the rating of Swiss T12/T13 plugs and sockets (SN 441011), regardless of the 16 A advertised for the plug. Override `current_limit` locally if needed.

## lspa8-v1.0.0 — 2026-10-19

First release.

- `common/base.yaml`: logger DEBUG / initial INFO with HA level selector, OTA encrypted with the API key, fallback AP + captive portal, web server v3, diagnostics (status, Wi-Fi dBm and %, IP, uptime, reset reason), restart and safe-mode buttons, explicit icons on all entities.
- `devices/lspa8-cb2s.yaml`: LSPA8 (A8TYV1.3 / CB2S): relay with configurable restore mode, button toggle, blue LED as status LED, BL0937 with per-device calibration substitutions, over-current cut-off (16 A default), "power-on state" selector.
