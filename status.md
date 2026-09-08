# Project Status

## Flipper Zero Bluetooth Govee Smart LED Controller

### Current Phase
**Phase 1 Development** — Core implementation complete behind a BLE
transport seam; real radio blocked on firmware BLE-central support.

### Overall Progress
🟢 Requirements Definition: 100%  
🟢 Technical Design: 100%  
🟡 Development: 60% (app logic complete; radio blocked)  
🟡 Testing: 30% (on-device crash fixes pending final confirmation)  
🟢 Documentation: 85%

### Completed Items
✅ Product Requirements Document (PRD)  
✅ Technical Implementation Document  
✅ README.md  
✅ ufbt build environment, dual-channel verified (release API 87.1 / dev API 88.2)  
✅ H6006 packet module (power, brightness, colour, white, keepalive, XOR checksum)  
✅ Full UI flow: menu → live scan list → control view (power / brightness /
colour presets / white temperature)  
✅ BLE transport abstraction (`ble_transport.h/.c`) with simulation backend —
all radio I/O isolated behind one interface; `[CENTRAL]` insertion points
marked with the exact ST `aci_*` calls a central-capable firmware needs  
✅ Scanner: Govee name filtering, address dedup, mutex-protected results,
GUI-safe notification via view-dispatcher custom events  
✅ Connection: keepalive thread (2 s), mutex-serialised packet writes  
✅ On-device crash fixes:
- NULL custom-event context (`view_dispatcher_set_event_callback_context`)
- Back-key bus fault (module views pass module instance, not app context,
  to previous callbacks — now a file-scope app reference)
- App exit trap (root previous callback returns `VIEW_NONE`)
- Scanner stop-path race hardening

### In Progress
🔄 On-device regression confirmation of the crash fixes  
🔄 Radio-path decision (see Blockers)

### Next Steps
See `TODO.md` (authoritative task list).

1. **Radio path** — custom firmware exporting GAP/GATT-client wrappers, or
   ESP32 BLE bridge over GPIO UART.
2. **Packet validation** — phone BLE-sniffer capture of a real H6006 session
   vs. the app's logged TX packets.
3. **Persistence** — saved devices via storage/flipper_format.

### Blockers
🔴 **BLE central unavailable to FAPs** — verified against the exported symbol
tables of stock 1.4.3 (API 87.1), Momentum (`dev`), and Unleashed (`dev`):
no GAP observer or GATT client functions are exported to external apps, and
the firmware's own `gap.c` implements peripheral profiles only. Evidence and
options documented in `BLE_CENTRAL_NOTES.md`. The app is architected so that
only `ble_transport.c` changes when a central-capable path exists.

### Key Decisions Made
- Architecture: transport seam with simulation backend; UI and protocol
  logic fully exercisable on stock firmware
- Protocol: direct BLE communication (no cloud dependency)
- UI: native Flipper Zero interface (view dispatcher + module views)
- Scope: H6006 first; packet family shared with H616x strips
- Builds: dual-channel (release/dev) to match user firmware API

### Risk Status
🔴 **High**: BLE-central firmware support (external dependency; mitigations
in `BLE_CENTRAL_NOTES.md`)  
🟡 **Medium**: protocol variance across Govee models (mitigation: modular
drivers); memory constraints (mitigation: bounded fixed arrays)  
🟢 **Low**: UI/UX, storage

### Last Updated
2026-09-03 (supersedes 2025-01-21)
