# Seeed XIAO ESP32‑C6 ESPHome Device Builder Package

This repository contains a reusable **ESPHome Device Builder package** for the Seeed XIAO ESP32‑C6 (esp32c6) boards. The project provides a shared base configuration package that can be included in device YAMLs to keep device files concise and consistent. The configuration is tailored for Home Assistant Bluetooth proxy and scanning functionality. I use it with the "Bermuda BLE Trilateration" HACS add-on for room-level presence detection.

Quick overview

- Purpose: maintain one canonical, reusable base configuration for Seeed XIAO ESP32‑C6 boards and simple device examples that include it.

- Layout:

  - `examples/` — device example YAMLs and helpers.

  - `examples/common/Seeed xiao ESP32-c6 base.yaml` — shared base configuration (board, wifi, API/OTA, sensors, antenna outputs).

  - `examples/common/Seeed xiao ESP32-c6 base IRK.yaml` — IRK capture variant base configuration.

![Seeed XIAO ESP32-C6 PCB](docs/seeed%20c6%20pcb.jpg)

**Note:** The Seeed XIAO ESP32-C6 board is officially supported in ESPHome as of version 2025.12.0 (December 2025). The board definition includes GPIO aliases and RF switch pin definitions (GPIO3 = `RF_SWITCH_EN`, GPIO14 = `RF_ANT_SELECT`).

## How to use the base package

The generic device YAML includes the ESP32-C6 base configuration via `packages` and provides substitutions:

```yaml
substitutions:
  device_name: esphomec6-garage
  friendly_name: Garage C6
  api_key: "ZmFrZWFwaWtleWZha2VleGFtcGxlZmFrZWtleQ=="
  ota_password: "ChangeMe!2025"

packages:
  device: !include "common/Seeed xiao ESP32-c6 base.yaml"
```

The base configuration uses the `${api_key}` and `${ota_password}` from your device specific YAML, uses `!secret` for Wi‑Fi values (managed by ESPHome Builder), and handles the `esphome:` section automatically.

What the base config provides:

- Board & SDK: selects `esp32c6` variant and `seeed_xiao_esp32c6` board with `esp-idf` framework.

- Boot actions: toggles GPIO3 and GPIO14 used for antenna selection on boot (FM8625H RF switch).

- Logger & status LED: configures serial log level (USB_SERIAL_JTAG) and board LED (GPIO15) behavior.

- API & OTA: supports encrypted API (uses `${api_key}`) and OTA (uses `${ota_password}`).

- Wi‑Fi: uses `!secret` for `wifi_ssid`, `wifi_password`, and `wifi_captive`; provides fallback captive AP settings with disconnect tracking.

- BLE: enables BLE scanning and Bluetooth proxying with configurable scan profiles:
  - **Aggressive** (default): 160ms interval, 160ms window (100% duty cycle) for maximum presence detection accuracy
  - **Balanced**: 320ms interval, 160ms window (50% duty cycle) for reduced power consumption
  - Profile selection persists across reboots

- Sensors: uptime, internal temperature, Wi‑Fi RSSI, Wi‑Fi info (BSSID, IP, SSID, MAC), Wi‑Fi disconnects (since boot), and SNTP time.

- Antenna control: a template switch manages two outputs (`ant_gpio3`, `ant_gpio14`) for FM8625H RF switch control.

## IRK Capture Variant

This repository also includes an alternate configuration for capturing iPhone, Apple Watch, and Android BLE Identity Resolving Keys (IRKs):

- **Base config**: `examples/common/Seeed xiao ESP32-c6 base IRK.yaml` — uses ESP-IDF framework with NimBLE for IRK capture
- **Device example**: `examples/ESPHome device config C6 IRK.yaml` — minimal device configuration for IRK capture

The IRK variant provides the same base features (antenna control, WiFi, sensors) but adds IRK capture functionality through the [irk-capture](https://github.com/DerekSeaman/irk-capture) external component.

**Device YAML example for IRK capture:**

```yaml
substitutions:
  device_name: esphomec6-garage
  friendly_name: Garage C6
  api_key: "ZmFrZWFwaWtleWZha2VleGFtcGxlZmFrZWtleQ=="
  ota_password: "ChangeMe!2025"

packages:
  device: !include "common/Seeed xiao ESP32-c6 base IRK.yaml"
```

**Key differences from the standard base:**

- Uses **ESP-IDF framework** with NimBLE stack (native BLE support for IRK capture)
- Bluetooth proxy and BLE tracker are disabled to avoid conflicts with IRK capture component
- Includes IRK-specific text sensors, switches, and buttons
- Advertises as a BLE Heart Rate Sensor to trigger pairing with iPhone, Apple Watch, and Android devices
- Automatically captures and publishes IRK keys during the pairing process

**How it works:**

1. Flash the device using ESPHome CLI or Device Builder
2. The device advertises as a BLE Heart Rate Sensor (configurable name)
3. Pair your iPhone, Apple Watch, or Android device with the advertised BLE device
4. The IRK is automatically captured and published to Home Assistant sensors
5. Use the captured IRKs with Home Assistant's [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/) integration for reliable presence detection

## Using with ESPHome Device Builder

This is an **ESPHome Device Builder package** designed to work seamlessly with the ESPHome Builder tool in Home Assistant:

1. Install the ESPHome and ESPHome Device Builder add-ons from the Home Assistant Add-on Store
2. In your ESPHome configuration directory, create a `common` folder:

   ```text
   config/
   └── esphome/
       └── common/
           └── Seeed xiao ESP32-c6 base.yaml  ← Place the base config here
   ```

3. Copy the `Seeed xiao ESP32-c6 base.yaml` file to the `config/esphome/common/` directory
   - For IRK capture functionality, use `Seeed xiao ESP32-c6 base IRK.yaml` instead (see [IRK Capture Variant](#irk-capture-variant) section)
4. Create your device YAML using the minimal structure shown above:
   - Update the `device_name` and `friendly_name` substitutions for your specific device
   - Generate new `api_key` and `ota_password` values (ESPHome Builder can generate these)
   - The file should include the base via `packages: device: !include "common/Seeed xiao ESP32-c6 base.yaml"`
5. ESPHome Builder automatically handles:
   - Wi-Fi secrets storage (no manual `secrets.yaml` needed)
   - Firmware compilation
   - Initial upload to your ESP32-C6 device
6. The device will automatically be discovered by Home Assistant

**Note:** The base configuration uses `!secret` references for Wi-Fi credentials, which ESPHome Builder manages automatically. You only need to provide the `api_key` and `ota_password` substitutions in your device YAML. To get fresh API and OTA keys, I suggest creating a new device in ESPHome Device Builder (using any hardware model), then replace all of the YAML with my device file but re-use the fresh API/OTA keys.

## Status LED Patterns

The onboard LED (GPIO15) provides visual feedback about the device state:

| Pattern | Meaning |
|---------|---------|
| Solid ON | Everything OK - WiFi connected, API connected with active client |
| Slow blink (~1Hz) | Warning - WiFi connected but API client not connected/subscribed |
| Fast blink (~2-3Hz) | Error - No WiFi connection |
| Very fast blink (~10Hz) | Critical error during boot or OTA in progress |

## ESPHome Device Page

Here's what the device looks like in Home Assistant's ESPHome integration:

![ESPHome Device Page](docs/screenshot-1.jpg)

The device page shows:

- **Device info**: Board type, firmware version, and MAC address
- **Controls**: BLE Scan Profile selector (Aggressive/Balanced) and External Antenna toggle switch
- **Configuration**: Firmware management and OTA updates
- **Diagnostic**: BSSID, internal temperature, IP address, MAC address, SSID, uptime, Wi-Fi disconnects (since boot), and Wi-Fi RSSI

## IRK Capture Device Page

Here's what the IRK capture variant looks like in Home Assistant:

![IRK Capture Device Page](docs/screenshot-irk.jpg)

The IRK capture device page shows:

- **Device info**: Board type, firmware version, and MAC address
- **Controls**: BLE Advertising toggle, BLE Device Name input, External Antenna toggle, and Generate New MAC button
- **Sensors**: IRK (captured Identity Resolving Key) and Device MAC (paired device address)
- **Configuration**: Firmware management and OTA updates
- **Diagnostic**: BSSID, internal temperature, IP address, MAC address, SSID, uptime, Wi-Fi disconnects (since boot), and Wi-Fi RSSI

## IRK Capture Troubleshooting

If you're having trouble capturing IRKs:

- **Pairing fails**: "Forget" the pairing on your iPhone, Apple Watch, or Android device in Bluetooth settings and try pairing again
- **Still not working**: Change the BLE device name in Home Assistant and attempt pairing with the new name
- **After successful capture**: "Forget" the paired ESP32 device from your phone's Bluetooth settings to allow fresh pairings for future captures
