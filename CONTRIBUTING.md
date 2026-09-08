# Contributing to GlowFin

Contributions are welcome — new Govee models, protocol corrections, UI work,
and bridge improvements especially.

## Environment setup

```bash
pip install ufbt
git clone https://github.com/romanianbuspass/GlowFin_update.git
cd GlowFin_update/govee_control
ufbt update          # release channel, or: ufbt update --channel=dev
ufbt                 # build -> dist/govee_control.fap
ufbt launch          # flash + run on a connected Flipper
```

Bridge firmware (ESP32-C3): `cd bridge/esp32c3_govee_bridge && pio run`.

## Code style

- Flipper Zero C conventions: 4-space indent, `Type* name`, 100-column limit.
- Format before committing: `clang-format -i govee_control/*.c govee_control/*.h`
  (canonical firmware `.clang-format` style; CI check: `clang-format --dry-run --Werror`).
- The build is `-Werror`; zero warnings is the bar.
- Keep the transport seam: all radio I/O belongs in `ble_transport.c`.
  Nothing above `ble_transport.h` may touch UART/BLE directly.

## Protocol changes

The UART protocol is versioned in `bridge/PROTOCOL.md`. Any change must be
made symmetrically in both ends (`govee_control/ble_transport.c` and
`bridge/esp32c3_govee_bridge/src/main.cpp`) in the same commit, and the
document updated.

## Testing new Govee models

1. Capture a real session from the vendor app with a BLE sniffer (e.g.
   nRF Connect / Wireshark + Android HCI snoop log).
2. Compare against the app's TX logs (`ufbt cli` — every packet is logged in
   wire format, or use the sim backend without hardware).
3. Add model detection in `ble_scanner.c` (`ble_scanner_is_govee`) and, if
   the command set differs, a packet module alongside `govee_h6006.c`.

## Pull requests

- One concern per PR; describe on-device test results (firmware fork + API
  version, e.g. "Unleashed 1367e, API 88.4").
- Confirm a clean build on at least the channel matching your firmware.
