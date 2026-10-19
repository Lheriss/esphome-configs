# Changelog

## lspa8-v1.0.0 — 2026-10-19

First release.

- `common/base.yaml`: logger DEBUG / initial INFO with HA level selector, OTA encrypted with the API key, fallback AP + captive portal, web server v3, diagnostics (status, Wi-Fi dBm and %, IP, uptime, reset reason), restart and safe-mode buttons, explicit icons on all entities.
- `devices/lspa8-cb2s.yaml`: LSPA8 (A8TYV1.3 / CB2S): relay with configurable restore mode, button toggle, blue LED as status LED, BL0937 with per-device calibration substitutions, over-current cut-off (16 A default), "power-on state" selector.
