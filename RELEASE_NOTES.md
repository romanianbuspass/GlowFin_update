# GlowFin v1.1.0 — Release Notes (draft)

**Govee BLE LED control for Flipper Zero — modernised for current firmware,
now with a real BLE central path via an ESP32-C3 bridge.**

## Highlights

- **Runs clean on current firmware** — Official 1.0+ (API 87.x), Momentum,
  and Unleashed (API 88.x). Zero-warning builds, no deprecated Furi APIs.
- **Real device control** — new ESP32-C3 bridge (NimBLE GATT client) on the
  GPIO UART. Flipper firmware alone cannot act as a BLE central on any
  current channel (verified against stock, Momentum and Unleashed symbol
  tables — see `BLE_CENTRAL_NOTES.md`); the bridge closes that gap.
- **Crash-free** — four on-device faults fixed: Back-key bus fault, NULL
  custom-event context, exit trap, scanner stop race.
- **Complete UI** — scan → device list → control view: power, brightness,
  8 colour presets, 5 white temperatures, 2 s keepalive.
- **Simulation mode** — evaluate the full UI with no hardware
  (`cdefines=["GOVEE_TRANSPORT_SIM"]`); every packet logged in wire format.

## Assets

| File | Use |
|------|-----|
| `govee_control_stable_api87_1.fap` | Official release firmware (1.4.x, API 87) |
| `govee_control_dev_api88_2.fap` | Dev/Momentum/Unleashed builds (API 88) |
| `govee_control_SIM_api88_2.fap` | Simulation backend, no bridge needed |
| `firmware.bin` | ESP32-C3 bridge image (flash via esptool/ESPWebTool) |

## Install

App: copy the `.fap` matching your firmware to `SD Card/apps/Bluetooth/`.
Bridge: flash `firmware.bin` to an ESP32-C3, wire GPIO4→14, GPIO5→13, 3V3,
GND (see `bridge/README.md`).

## Known limitations

- Single-device connection (multi-device pooling is on `TODO.md`).
- Bridge filters advertisements to `ihoment`/`Govee`/`H6006` names.
- GATT writes are unacknowledged by the bulb beyond ATT success — the UI
  mirrors assumed state.

**Full changelog:** [`CHANGELOG.md`](CHANGELOG.md)
