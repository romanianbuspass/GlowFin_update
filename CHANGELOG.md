# Changelog

All notable changes to this project are documented in this file, following
[Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.1.0] - 2026-09-08 — Modern Firmware Compatibility + BLE Bridge

### Added
- ESP32-C3 BLE bridge firmware (NimBLE GATT client) in `bridge/`, with a
  documented framed UART protocol v1 (`bridge/PROTOCOL.md`).
- UART transport backend as the default: expansion-aware USART handling,
  stream-buffer RX parser, 7 s connect / 2 s write timeouts, clean shutdown.
- Simulation backend retained behind `cdefines=["GOVEE_TRANSPORT_SIM"]`.
- Full UI flow: main menu, live scan list (GUI-thread-safe custom events),
  device control view — Power, Brightness (0–100 %), 8 colour presets,
  5 white-temperature presets; 2 s connection keepalive.
- App icon (10×10 bulb) and complete `application.fam` metadata.
- `LICENSE` (MIT, upstream attribution), `CONTRIBUTING.md`,
  `RELEASE_NOTES.md`, `TODO.md`, `BLE_CENTRAL_NOTES.md`.
- Verified evidence that no mainstream firmware exports BLE central APIs to
  FAPs (stock 1.4.3, Momentum, Unleashed symbol tables).

### Changed
- `application.fam` modernised: valid category, description, author, URL,
  version 1.1; unused `bt` service requirement dropped.
- Scanner: Govee name filtering, address dedup, mutex-protected results.
- Connection: transport-backed writes serialised by mutex.

### Fixed
- Bus fault on Back key from any sub-view (module views pass the module
  instance, not the app context, to previous callbacks).
- NULL dereference on first scan result (missing
  `view_dispatcher_set_event_callback_context`).
- App exit trap: root previous callback now returns `VIEW_NONE`.
- Scanner stop-path race (callback captured under mutex; join before clear).
- Unguarded allocation failure in the app entry point.
- Tracked build artifacts and personal-path files removed from the repo;
  `.gitignore` aligned with actual build outputs.

## [0.1] - 2025-01-21 — Upstream baseline

- Initial fork from `devdotbo/GlowFin`: PRD, technical design, H6006 packet
  module, mock scanner/connection scaffolding.
