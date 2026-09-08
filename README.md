# GlowFin

**Govee BLE LED control for the Flipper Zero — modernised for current firmware.**

![Firmware](https://img.shields.io/badge/firmware-Official%201.0%2B%20%7C%20Momentum%20%7C%20Unleashed-blue)
![Platform](https://img.shields.io/badge/platform-Flipper%20Zero-orange)
![Bridge](https://img.shields.io/badge/BLE%20bridge-ESP32--C3-green)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

## Overview

GlowFin discovers and controls Govee Bluetooth LED devices (focused on the
H6006 smart bulb; the same `0x33` packet family covers many H6xxx strips)
directly from a Flipper Zero: power, brightness, RGB colour, white
temperature, with connection keepalive and a live scan list.

This repository is an updated fork of
[`devdotbo/GlowFin`](https://github.com/devdotbo/GlowFin), ported to current
Flipper Zero firmware (Official 1.0+ / API 87–88, Momentum, Unleashed) and
extended with a working BLE central path.

## Important: the ESP32-C3 bridge

No current Flipper firmware — Official, Momentum, or Unleashed — exposes BLE
**central** functions (scanning / GATT client) to external apps; the radio
runs peripheral profiles only. The evidence and options are documented in
[`BLE_CENTRAL_NOTES.md`](BLE_CENTRAL_NOTES.md).

GlowFin therefore controls real bulbs through a small **ESP32-C3 bridge**
(BLE 5) on the GPIO UART. The bridge firmware and protocol live in
[`bridge/`](bridge/) — flash it, wire four pins, and the app talks to real
hardware. Without the bridge, a **simulation backend** (one-line build flag)
lets you run and evaluate the full UI with packets logged over the serial
console.

## Key changes in this release (v1.1)

- Ported to modern SDK standards — clean, warning-free builds against
  Official release (API 87.1) and dev (API 88.2) toolchains.
- Fixed four on-device crashes: Back-key bus fault (module-view context),
  NULL custom-event context, app-exit trap, scanner stop-path race.
- All radio I/O isolated behind a transport seam (`ble_transport.h`) with
  interchangeable backends: UART bridge (default) and simulation.
- New ESP32-C3 bridge firmware (NimBLE GATT client) + framed UART protocol v1.
- Modernised `application.fam` (category, description, author, URL, icon).
- Full UI flow: menu → live scan list → control view (power / brightness /
  8 colour presets / 5 white-temperature presets).

## Hardware & firmware requirements

- Flipper Zero — Official 1.0+, Momentum, or Unleashed (use the `.fap` built
  for your API major: 87.x for release 1.4.x, 88.x for current dev builds).
- For real device control: ESP32-C3 board flashed with
  [`bridge/esp32c3_govee_bridge`](bridge/) and wired to GPIO 13/14 (+3V3/GND).
- A Govee BLE device — H6006 verified by protocol; other `ihoment`/`Govee`
  H6xxx models using the same packet family should work.

## Installation

**Option A — pre-compiled `.fap`:** copy `govee_control.fap` to
`SD Card/apps/Bluetooth/` and launch from *Apps → Bluetooth → GlowFin*.

**Option B — build from source:**

```bash
git clone https://github.com/romanianbuspass/GlowFin_update.git
cd GlowFin_update/govee_control
ufbt update        # or: ufbt update --channel=dev  (match your firmware)
ufbt               # produces dist/govee_control.fap
ufbt launch        # with Flipper connected via USB
```

Simulation build (no bridge hardware): add `cdefines=["GOVEE_TRANSPORT_SIM"]`
to `govee_control/application.fam` before building.

**Bridge:** see [`bridge/README.md`](bridge/README.md). In short:
`cd bridge/esp32c3_govee_bridge && pio run -t upload`, then wire
C3 GPIO4→Flipper 14, GPIO5→Flipper 13, 3V3, GND (power off first).

## Controls & usage

- **Scan for Devices** — live list of Govee devices as they advertise.
- **Up/Down** navigate, **OK** select a device (connects) or change a value.
- In the control view: **Left/Right** adjust the selected item
  (Power Off/On, Brightness 0–100 %, Colour preset, White temperature).
- **Back** returns (disconnects from the control view); Back on the main
  menu exits. A 2 s keepalive runs while connected.

## Troubleshooting

- **"Connect timed out"** — bridge not responding: check wiring (TX/RX
  cross), bridge power, and that the C3 was flashed successfully
  (`pio device monitor` shows `govee-bridge ready`).
- **No devices found** — the bulb must be powered and advertising; the
  bridge filters for `ihoment`/`Govee`/`H6006` names. Govee bulbs only
  advertise when not connected to another central (close the Govee app).
- **API version warning at launch** — the `.fap` was built for a different
  firmware API; rebuild with the matching `ufbt` channel.
- **Debug console** — `ufbt cli` shows the app's packet-level logs; the
  bridge has its own USB debug console at 115200 baud.

## Documentation

- [`bridge/PROTOCOL.md`](bridge/PROTOCOL.md) — UART framing v1 (host ↔ bridge)
- [`BLE_CENTRAL_NOTES.md`](BLE_CENTRAL_NOTES.md) — why a bridge is needed
- [`TODO.md`](TODO.md) — current task list and roadmap
- [`CHANGELOG.md`](CHANGELOG.md) — release history
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — development guidelines

## License

MIT — see [`LICENSE`](LICENSE). Original project © devdotbo; fork
modifications © romanianbuspass.
