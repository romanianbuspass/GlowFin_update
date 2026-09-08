# BLE Central Limitation on Flipper Zero

Verified 2026-09-02 against the actual SDK artifacts:

- Stock release firmware 1.4.3 (`API 87.1`, `targets/f7/api_symbols.csv`)
- Momentum firmware (`Next-Flip/Momentum-Firmware@dev`, `targets/f7/api_symbols.csv`)
- Unleashed firmware (`DarkFlippers/unleashed-firmware@dev`)

## Finding

No channel exports BLE **GAP observer** (scanning) or **GATT client**
(connect / service discovery / characteristic write) functions to external
FAPs. The only exported BLE functions are peripheral-side:

- `furi_hal_bt_start_advertising` / `stop_advertising`, extra-beacon API
- `ble_gatt_service_add`, `ble_gatt_characteristic_init/update/delete`
  (GATT *server* — the Flipper acting as a peripheral)
- `bt_profile_start` (HID / serial peripheral profiles)

The firmware's own `targets/f7/ble_glue/gap.h` contains no observer or
central entry points at all — Flipper's `gap.c` implements peripheral
profiles only, even though the STM32WB coprocessor stack
(`stm32wb5x_BLE_Stack_full_fw`) supports central mode.

## Consequence

A Govee controller (which is a BLE **central**: scans, connects, performs
GATT client writes) cannot be realised as a FAP on any current mainstream
firmware. It requires one of:

1. **Custom firmware build** adding central-mode wrappers around the ST WPAN
   calls (`aci_gap_start_general_discovery_proc`, `aci_gap_create_connection`,
   `aci_gatt_disc_all_primary_services`, `aci_gatt_write_char_value`, ...)
   and exporting them in `targets/f7/api_symbols.csv`.
2. **External radio bridge** (e.g. ESP32 dev board on GPIO UART running a
   BLE-proxy), with the Flipper app as the UI front-end.

## What this app does in the meantime

All radio I/O is isolated behind `ble_transport.h`. The default build uses
the simulation backend in `ble_transport.c`:

- scan emits one demo `ihoment_H6006` device,
- connect always succeeds,
- every command packet is logged in its exact 20-byte wire format via
  `FURI_LOG_I` (visible over `ufbt cli` / serial logs).

Each function in `ble_transport.c` carries a `[CENTRAL]` marker naming the
ST `aci_*` call a firmware patch would map to, so a future central-capable
firmware requires changes in exactly one file.
