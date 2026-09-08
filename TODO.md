# TODO

## Flipper Zero Govee Controller — Current Task List

Last updated: 2026-09-03 (reflects commits through `176ffa5`)

### Blocked — awaiting a radio-path decision

Real BLE central operation (GAP scanning, GATT connect/write) is impossible
from a FAP on any mainstream firmware (stock 1.4.3, Momentum, Unleashed —
see `BLE_CENTRAL_NOTES.md` for the verified evidence). One of:

- [ ] **Option A — custom firmware**: wrap ST WPAN central calls
      (`aci_gap_start_general_discovery_proc`, `aci_gap_create_connection`,
      `aci_gatt_disc_all_primary_services`, `aci_gatt_write_char_value`) and
      export them in `targets/f7/api_symbols.csv`. Then implement only
      `ble_transport.c` — `[CENTRAL]` markers are in place.
- [ ] **Option B — ESP32 BLE bridge** on the Flipper GPIO UART; Flipper app
      becomes the UI front-end, ESP32 runs the GATT client.

### Validation

- [ ] Capture a real H6006 session with a phone BLE sniffer and diff against
      the app's `FURI_LOG` TX lines (sim transport already logs exact
      20-byte wire-format packets).
- [ ] On-device regression pass: Back key from every view, scan start/stop
      cycles, connect → control → back → rescan. (Crash fixes through
      `176ffa5` need user confirmation.)

### Features (unblocked, can proceed any time)

- [ ] Saved-devices persistence (storage module; PRD phase 1/2). Save/load
      bonded device list via `storage` + `flipper_format`.
- [ ] Multi-device support (PRD phase 2): connection layer is currently
      single-peer; needs a connection pool and per-device state.
- [ ] H616x strip models: same 0x33 packet family — extend device detection
      and the colour/music-mode commands.
- [ ] Effects/scene engine (PRD phase 3).
- [x] ~~App icon (`fap_icon` in `application.fam`, 10x10 PNG)~~ — done v1.1.

### Housekeeping

- [ ] Push commits to origin (no credentials on the build machine; patches
      delivered out-of-band in the meantime).
- [x] ~~Update `README.md` feature claims to match sim-backend reality~~ — done v1.1.
