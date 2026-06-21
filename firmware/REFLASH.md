# Pre-built firmware (HMS-2000D-4T support)

OpenDTU **v26.3.30** + the one-line patch adding serial prefix `0x1421` so a
Hoymiles **HMS-2000D-4T** (serials starting `1421...`) is recognised and works.

## Files
- `opendtu-hms2000d-patched-ota.bin`     — flash via OpenDTU web UI (OTA)
- `opendtu-hms2000d-patched-factory.bin` — full image for USB flashing
- `../0001-...patch`                      — the source code change (if you rebuild)

## Re-flash over the air (easiest)
1. Open the OpenDTU web UI → **Settings → Firmware Upgrade**
2. Upload `opendtu-hms2000d-patched-ota.bin`
3. It reboots into the patched firmware.
   (Your existing config — WiFi, inverter, CMT pins, MQTT — is preserved.)

## Re-flash via USB (if OTA not possible)
Connect the OpenDTU ESP32 by USB, then:
```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 921600 \
  write_flash 0x0 opendtu-hms2000d-patched-factory.bin
```
(or use PlatformIO / ESP flasher with the factory image at offset 0x0)

## After flashing — if the radio shows cmt_connected=false
Do a FULL power cycle (unplug power ~10s, replug) so the CMT2300A chip resets.
A soft reboot is not enough.

## CMT2300A pin profile used for this build
- Pin profile: CMT_NoInt (clk18, cs4, fcs5, sdio23, gpio2/3 = -1)
- Region: North America / 923 MHz, 20 dBm
(Adjust these for your own wiring/region in the OpenDTU web UI after flashing.)

## If you want to rebuild from a newer OpenDTU
Apply `../0001-...patch` (or just add `|| preSerial == 0x1421` to
`HMS_4CH::isValidSerial()`), then `pio run -e generic_esp32`.
