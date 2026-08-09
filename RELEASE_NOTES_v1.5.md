# v1.5 Release Notes

## No more copying files

The biggest change in this release: you can now set up a Seeed XIAO ESP32-C6 device entirely from the **ESPHome Device Builder** UI in Home Assistant. Create the device, pick the Seeed Studio XIAO ESP32C6 board, and paste a small `packages:` block into the generated YAML — that's it. No more needing to dig through the Home Assistant ESPHome add-on directory and uploading files manually.

Changes:

- The base configuration is pulled straight from GitHub at build time — no files to copy.
- Brand new step-by-step instructions (with screenshots) in the updated [README](README.md).

## The configuration files have been reorganized

If you're upgrading from an earlier version, the file layout has changed significantly. Since everything is now pulled from GitHub at build time, there's no need to copy any files manually. The directory structure has also been simplified.

The four configs now available:

- **`Seeed XIAO ESP32-c6 base.yaml`** — the standard configuration: Wi-Fi, sensors, status LED, external antenna control. No Bluetooth proxy.
- **`Seeed XIAO ESP32-c6 proxy.yaml`** — everything in `Seeed XIAO ESP32-c6 base.yaml`, plus Bluetooth proxy/BLE tracking with selectable scan profiles (Low/Medium/High).
- **`Seeed XIAO ESP32-c6 IRK.yaml`** — a lightweight companion for my separate [IRK Capture package](https://github.com/DerekSeaman/irk-capture). External antenna control is now available here too.
- **`Seeed XIAO ESP32-c6 remote.yaml`** — the small snippet you paste into ESPHome Device Builder to pull in whichever of the above you want.

## Other improvements

- Added 802.11k/v support for better Wi-Fi roaming between access points.
- Kept the Wi-Fi Channel diagnostic sensor available across all three configurations, including IRK.
- External antenna control (onboard vs. U.FL, via the FM8625H RF switch) now works the same way across base, proxy, and IRK configurations.
- Refreshed screenshots throughout the README to reflect the current setup.

## Upgrading from an earlier version

1. Follow the new setup steps in the [README](README.md) to recreate your device via ESPHome Device Builder, using the `Seeed XIAO ESP32-c6 remote.yaml` snippet in place of your old device YAML's `packages:` block.
2. Pick `Seeed XIAO ESP32-c6 base.yaml` or `Seeed XIAO ESP32-c6 proxy.yaml` depending on whether you want Bluetooth proxy functionality.
