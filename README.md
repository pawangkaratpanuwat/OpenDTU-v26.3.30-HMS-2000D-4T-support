# My patched OpenDTU — HMS-2000D-4T (serial prefix 0x1421) support

This is a personal fork of [tbnobody/OpenDTU](https://github.com/tbnobody/OpenDTU)
(based on tag **v26.3.30**) with one fix: the **Hoymiles HMS-2000D-4T**
(serial numbers starting `1421...`, e.g. `1421A0208266`, P/N CV010682) was
reported as "Unknown" and never polled because OpenDTU's `isValidSerial()`
check didn't include the `0x1421` prefix. This fork adds it.

See the patch: [`0001-Add-support-for-HMS-2000D-4T-inverters-with-serial-p.patch`](0001-Add-support-for-HMS-2000D-4T-inverters-with-serial-p.patch)
(one line changed in `lib/Hoymiles/src/inverters/HMS_4CH.cpp`).

## Install

### Option A — Flash a pre-built binary (no build tools needed)
Pre-built binaries are in [`firmware/`](firmware/):
- `opendtu-hms2000d-patched-ota.bin` — upload via the OpenDTU web UI: **Settings → Firmware Upgrade**. Your existing config (WiFi, inverter, CMT pins, MQTT) is preserved.
- `opendtu-hms2000d-patched-factory.bin` — full image for a first-time flash over USB:
  ```bash
  pip install esptool
  esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 921600 \
    write_flash 0x0 firmware/opendtu-hms2000d-patched-factory.bin
  ```

After flashing, if the radio shows `cmt_connected=false`, fully power-cycle the
ESP32/CMT2300A module (unplug ~10s, replug) — a soft reboot doesn't reset the
CMT2300A chip. See [`firmware/REFLASH.md`](firmware/REFLASH.md) for full notes.

### Option B — Build from source
Requires [PlatformIO](https://platformio.org/):
```bash
git clone https://github.com/pawangkaratpanuwat/DTU.git
cd DTU
pio run -e generic_esp32                      # build
pio run -e generic_esp32 -t upload --upload-port /dev/ttyUSB0   # flash over USB
```
Or flash over the air once running: `curl -F "data=@.pio/build/generic_esp32/firmware.bin" http://<device-ip>/api/firmware/update` (with MD5 + admin auth — see OpenDTU docs below).

### If you have a different inverter and OpenDTU reports "Unknown"
Your serial prefix probably isn't in `isValidSerial()` either. Check your
inverter's serial number against the patch above for the pattern, add your
prefix the same way, then rebuild (Option B).

---

# OpenDTU

[![OpenDTU Build](https://github.com/tbnobody/OpenDTU/actions/workflows/build.yml/badge.svg)](https://github.com/tbnobody/OpenDTU/actions/workflows/build.yml)
[![cpplint](https://github.com/tbnobody/OpenDTU/actions/workflows/cpplint.yml/badge.svg)](https://github.com/tbnobody/OpenDTU/actions/workflows/cpplint.yml)
[![Yarn Linting](https://github.com/tbnobody/OpenDTU/actions/workflows/yarnlint.yml/badge.svg)](https://github.com/tbnobody/OpenDTU/actions/workflows/yarnlint.yml)
[![Yarn Prettier](https://github.com/tbnobody/OpenDTU/actions/workflows/yarnprettier.yml/badge.svg)](https://github.com/tbnobody/OpenDTU/actions/workflows/yarnprettier.yml)

## !! IMPORTANT UPGRADE NOTES !!

If you are upgrading from a version before 15.03.2023 you have to upgrade the partition table of the ESP32. Please follow the [this](docs/UpgradePartition.md) documentation!

## Background

This project was started from [this](https://www.mikrocontroller.net/topic/525778) discussion (Mikrocontroller.net).
It was the goal to replace the original Hoymiles DTU (Telemetry Gateway) with their cloud access. With a lot of reverse engineering the Hoymiles protocol was decrypted and analyzed.

## Documentation

The documentation can be found [here](https://tbnobody.github.io/OpenDTU-docs/).
Please feel free to support and create a PR in [this](https://github.com/tbnobody/OpenDTU-docs) repository to make the documentation even better.

## Breaking changes

Generated using: `git log --date=short --pretty=format:"* %h%x09%ad%x09%s" | grep BREAKING`

```code
* 8cab3335      2025-08-07      BREAKING CHANGE: WebAPI endpoint `/api/limit/config` requires different parameters
* 8372deaf      2025-04-18      BREAKING CHANGE: Logging newline changed from "\r\n" to "\n"
* 1b637f08      2024-01-30      BREAKING CHANGE: Web API Endpoint /api/livedata/status and /api/prometheus/metrics
* e1564780      2024-01-30      BREAKING CHANGE: Web API Endpoint /api/livedata/status and /api/prometheus/metrics
* f0b5542c      2024-01-30      BREAKING CHANGE: Web API Endpoint /api/livedata/status and /api/prometheus/metrics
* c27ecc36      2024-01-29      BREAKING CHANGE: Web API Endpoint /api/livedata/status
* 71d1b3b       2023-11-07      BREAKING CHANGE: Home Assistant Auto Discovery to new naming scheme
* 04f62e0       2023-04-20      BREAKING CHANGE: Web API Endpoint /api/eventlog/status no nested serial object
* 59f43a8       2023-04-17      BREAKING CHANGE: Web API Endpoint /api/devinfo/status requires GET parameter inv=
* 318136d       2023-03-15      BREAKING CHANGE: Updated partition table: Make sure you have a configuration backup and completly reflash the device!
* 3b7aef6       2023-02-13      BREAKING CHANGE: Web API!
* d4c838a       2023-02-06      BREAKING CHANGE: Prometheus API!
* daf847e       2022-11-14      BREAKING CHANGE: Removed deprecated config parsing method
* 69b675b       2022-11-01      BREAKING CHANGE: Structure WebAPI /api/livedata/status changed
* 27ed4e3       2022-10-31      BREAKING: Change power factor from percent value to value between 0 and 1
```

## Currently supported Inverters

A list of all currently supported inverters can be found [here](https://www.opendtu.solar/hardware/inverter_overview/)
