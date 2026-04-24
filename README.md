# esphome_smabluetooth
ESPHome external component to read SMA solar inverters via classic Bluetooth (SMANET2 / SPP protocol).

**Beta quality** — the component is significantly more stable than earlier releases but has not yet been validated over extended multi-day/multi-week runs. Use in production at your own discretion.

## Status
- Tested and working on:
  - **SB3000TL-20**
  - **SB5000TL-20**
- Runs on ESP-IDF (not Arduino); BLE disabled to preserve RAM
- Non-blocking BT protocol task on core 0; ESPHome main loop never stalled
- Automatic night mode: disconnects from inverter at sunset, reconnects at sunrise — no polling during darkness
- Single-phase friendly: if you configure Phase B/C sensors on a 1-phase inverter, they will publish `0` (so Home Assistant shows 0 instead of `unknown/unavailable`).
- Improved robustness for older/quirky SMA firmwares: some missing/unsupported records are treated as non-fatal so the read loop stays alive.

## Hardware compatibility
Only the original ESP32 supports classic Bluetooth (SPP). Other variants do **not** work:

| Supported | Not supported |
|-----------|---------------|
| ESP32 (all flash/antenna variants: WROOM, WROVER, U, D, S) | ESP32-S2, S3 |
| | ESP32-C2, C3, C5, C6, C61 |

## Known limitations
- Single inverter per ESP32 (singleton BT connection)
- Not yet validated over weeks of continuous operation

## Usage
See `esphome/sample/smabluesolar.yaml` for a full configuration example.

### Key options
- `sma_inverter_delay_values`: delay between SMA queries inside the BT task (helps stability on slow inverters)
- `sma_inverter_btgetbyte_timeout`: low-level byte read timeout (ms) — useful if logon/read frequently hits timeouts
- `sma_inverter_reconnect_backoff`: list of reconnect delays after consecutive BT failures (recommended)

### Reconnect backoff (recommended)
Classic Bluetooth (SPP) on SMA inverters can be flaky (out of range, busy, temporary page timeouts). To avoid aggressive reconnect spam, you can configure a backoff list.

```yaml
sensor:
  - platform: smabluetooth_solar
    # ...
    sma_inverter_reconnect_backoff: [5s, 10s, 20s, 40s, 60s]
```

Behavior:
- After each consecutive BT connect/discovery failure, the next delay in the list is used.
- Once the last entry is reached, the component keeps using that delay.
- After a successful connection, the backoff resets to the first entry.

For local development, replace the `external_components` GitHub URL with a `path:` pointing to the repository root.
