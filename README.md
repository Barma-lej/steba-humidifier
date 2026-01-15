# Steba Humidifier ESPHome Integration

[![Buy Me A Coffee](https://img.shields.io/badge/donate-Coffee-ff813f.svg)](https://www.buymeacoffee.com/barma)

Control and monitor your Steba LB7 using an ESP32-C3 Supermini microcontroller.
This project emulates button presses via GPIO and sniffs the TM1628 display controller to read real-time device status (temperature, humidity, power state, heating, ionization, water level).
It is designed for seamless integration with Home Assistant via ESPHome API.

---

## Features

- **Hardware Button Emulation**
  Trigger Power, Ionization, Heating, and Intensity controls via short GPIO pulses (0.15s duration).
- **Display Sniffing (TM1628)**
  Real-time reading of:
  - Temperature
  - Humidity
  - Binary states: Power, Heating, Ionization, "No Water".
- **Home Assistant Integration**
  - Auto-discovery via ESPHome API
  - Encrypted API communication
  - OTA firmware updates
  - Built-in web dashboard
- **Stable Monitoring**
  - Sensor filters (delta / throttle)
  - Debouncing for binary sensors
- **Configurable GPIO Mapping**
  - All pins defined via `substitutions` in `steba-humidifier.yaml`.

---

## Hardware

### Required Components

- **Steba Humidifier LB7**
- **ESP32-C3 Supermini**
  (board: `esp32-c3-devkitm-1` or compatible)
- **SN74AHCT125N Quadruple Bus Buffer Gates**
- 4 x **4N38 DIP-6 Optocoupler**
- 4 x **100 kOhm resistors**
- 4 x **150 kOhm resistors**
- Jumper wires, soldering tools.

### Wiring Overview

- **Button emulation (outputs)**
  Connect GPIOs to the humidifier's front-panel button pads (through transistors or optocouplers if needed):

  - `GPIO04` → Power button
  - `GPIO03` → Ionization button
  - `GPIO01` → Water Heating button
  - `GPIO00` → Intensity (mist level) button

- **TM1628 display sniffing (inputs)**

  - `GPIO5` → TM1628 **STB**
  - `GPIO6` → TM1628 **CLK**
  - `GPIO7` → TM1628 **DIO**



### Wiring Diagram

![Schematic diagram](/docs/steba_lb7_humidifier_schematic_20251215.svg)

---

## Step-by-Step: First Flash of ESP32-C3 Supermini via Home Assistant

This section describes flashing using the **Home Assistant ESPHome Dashboard** (easiest for HA users).

### 1. Prerequisites

- **Home Assistant** instance with **ESPHome add-on** installed
  - Settings → Add-ons → ESPHome (or install from official add-ons repository)
- **ESP32-C3 Supermini** board (new, never flashed)
- **USB cable** (data cable, not power-only)
- USB drivers installed (optional):
  - **macOS**: <https://github.com/WCHSoftGroup/ch34xser_macos>
  - **Windows**: <https://www.wch.cn/downloads/CH341SER_EXE.html>
  - **Linux**: usually automatic

### 2. Prepare Configuration Files

Clone or download the repository to your computer:

```bash
git clone https://github.com/Barma-lej/steba-humidifier.git
cd steba-humidifier
```

Create `secrets.yaml` from the example:

```bash
cp secrets.yaml.example secrets.yaml
```

Edit `secrets.yaml` with your credentials:

```yaml
wifi_ssid: "Your_WiFi_Network"
wifi_password: "Your_WiFi_Password"
fallback_ap_password: "some_strong_password"

web_server_user: "admin"
web_server_password: "Your_WEB_SERVER_Password"

ota_password: "OTA_Password_from_new_created_steba-humidifier.yaml"

# Home Assistant API keys
steba_humidifier: "API_Password_from_new_created_steba-humidifier.yaml"
```

### 3. Access ESPHome Dashboard in Home Assistant

1. In Home Assistant, go to:
   **Settings → Add-ons → ESPHome**
2. Click the **"Open Web UI"** button (or navigate to `http://<HA_IP>:6052`)
3. You should see the ESPHome dashboard with a list of managed devices (empty at first).

### 4. Create New Device

1. Click the **"+ New Device"** button (bottom right, or "Create Device" if empty).
2. A wizard will appear. Choose one of:
   - **"Continue"** (to use the wizard)
   - Or skip directly to editing YAML
3. Fill in device name:
   - **Device Name**: `steba-humidifier`
   - **Area**: (optional) `Bedroom`, `Living Room`, etc.
4. Click **"Next"** → **"Skip"** (or "Continue") through additional questions.

### 5. Upload Configuration File

1. After the wizard completes, the device appears in the ESPHome dashboard.
2. Click the **"Edit"** button (pencil icon) next to `steba-humidifier`.
3. The YAML editor opens. Replace all content with:

   - Copy the full contents of `steba-humidifier.yaml` from the repository
   - !! **Make sure** `secrets.yaml` values are saved beforehand (`ota_password:` and `steba_humidifier:` must be from new created `steba-humidifier.yaml` pasted)!!
   - Paste into the ESPHome editor

4. Click **"Save"** at the bottom.

Alternatively, if you're familiar with HA file structure, place `steba-humidifier.yaml` directly in:

```text
<HA_CONFIG>/esphome/steba-humidifier.yaml
```

Then refresh the ESPHome dashboard.

### 6. Connect ESP32-C3 via USB

1. Plug the ESP32-C3 Supermini into your Home Assistant host (or any USB port connected to HA) via USB cable.
2. If needed, put the board into **download mode**:
   - Hold **BOOT** button
   - Press and release **RESET** button
   - Release **BOOT** button
3. The board should appear as a serial device.

### 7. Compile and Flash

1. In the ESPHome dashboard, click the **"Install"** button (or **three dots → Install**) next to `steba-humidifier`.
2. A menu appears with options:
   - **"Wirelessly"** (for already-flashed devices)
   - **"USB"** (for first flash or bricked boards)
3. Click **"USB"**.
4. ESPHome will:
   - Compile the firmware
   - Detect connected USB devices (show a list)
   - Ask you to select the COM port or serial device
   - Flash the firmware
5. Wait for completion (2–5 minutes).

**If port not auto-detected:**
- Click **"Manual download"** to get the binary file
- Use esphome CLI on a different machine:

  ```bash
  esphome run steba-humidifier.yaml
  ```

### 8. Monitor First Boot

1. After flashing, click **"View Logs"** (or the monitor icon).
2. You should see boot messages:

   ```text
   [xx:xx:xx][I][app:094]: ESPHome version 2025.12.0
   [xx:xx:xx][I][wifi:504]: WiFi connected! IP: 192.168.1.100
   [xx:xx:xx][I][api:136]: API Server started on port 6053
   ```

3. If Wi-Fi fails to connect, the device starts a fallback AP:
   - SSID: `steba-humidifier`
   - Password: (from `fallback_ap_password` in `secrets.yaml`)
   - Connect to it via your phone/laptop and reconfigure Wi-Fi

### 9. Device Should Now Appear Online

Back in the ESPHome dashboard:

- The `steba-humidifier` device should show **"Status: Online"** (green indicator)
- If offline, check Wi-Fi connection or restart the device

---

## Alternative: CLI Method (Command Line)

If you prefer command-line flashing or if Home Assistant ESPHome dashboard doesn't work:

### Option A: Direct CLI Flashing

1. Install ESPHome CLI on your PC:

   ```bash
   pip install esphome
   ```

2. From the repository root:

   ```bash
   esphome run steba-humidifier.yaml
   ```

3. ESPHome will compile, detect USB ports, and flash automatically.

### Option B: Manual Download & Flash

1. In ESPHome dashboard, click **"Install"** → **"Manual download"**.
2. This downloads `steba-humidifier.bin`.
3. Use `esptool.py`:

   ```bash
   pip install esptool
   esptool.py -p /dev/ttyUSB0 write_flash 0x0 steba-humidifier.bin
   ```

   Or use the official **[ESP Flash Download Tool](https://www.espressif.com.cn/en/support/download/other-tools)**.

---

## Adding to Home Assistant

### 1. Web Dashboard (Optional)

Open the ESPHome web server (enabled in `steba-humidifier.yaml`):

```text
http://<ESP_IP>/
```

Login with:

- Username: `web_server_user` from `secrets.yaml`
- Password: `web_server_password` from `secrets.yaml`

You can inspect sensors and trigger buttons from the browser.

### 2. ESPHome Integration in HA

1. In Home Assistant go to:
   **Settings → Devices & Services → ESPHome**.
2. Click **"Add Integration"** (or **"+ Add device"** if ESPHome already enabled).
3. Enter the device address:

   ```text
   <ESP_IP>:6053
   ```

4. When prompted, paste the **encryption key** from the `api.encryption.key` field in `steba-humidifier.yaml`.
5. Confirm; Home Assistant will create a new device with entities such as:

   - `sensor.steba_humidifier_temperature`
   - `sensor.steba_humidifier_humidity`
   - `binary_sensor.steba_humidifier_power`
   - `binary_sensor.steba_humidifier_heat`
   - `binary_sensor.steba_humidifier_ion`
   - `binary_sensor.steba_humidifier_no_water`
   - `button.power`, `button.heat`, `button.ion`, `button.intensity`

### 3. Example Lovelace Card

```yaml
type: entities
title: Steba Humidifier
entities:
  - entity: sensor.steba_humidifier_temperature
  - entity: sensor.steba_humidifier_humidity
  - entity: binary_sensor.steba_humidifier_power
  - entity: binary_sensor.steba_humidifier_heat
  - entity: binary_sensor.steba_humidifier_ion
  - entity: binary_sensor.steba_humidifier_no_water
  - entity: button.power
  - entity: button.heat
  - entity: button.ion
  - entity: button.intensity
```

---

## Subsequent Updates (OTA)

Once the first USB flash is done, you can update firmware over Wi-Fi:

1. **Via ESPHome Dashboard**: Click **"Install"** → **"Wirelessly"**.
2. **Via CLI**: `esphome run steba-humidifier.yaml` will detect the device and upload OTA.

ESPHome will:

1. Compile the firmware.
2. Upload it via OTA to the device (no USB required).
3. Reboot the device with the new firmware.

If OTA fails, you can always re-flash over USB with the steps from the "First Flash" section.

---

## Configuration Details

### GPIO Pinout (from `steba-humidifier.yaml`)

In `steba-humidifier.yaml`, pins are defined via `substitutions`:

```yaml
substitutions:
  name: "steba-humidifier"
  friendly_name: "Steba Humidifier"
  room: "Wohnzimmer"
  project_version: "2025.12.0"

  # Buttons
  intensity_gpio: GPIO00
  heatwater_gpio: GPIO01
  ion_gpio: GPIO03
  power_gpio: GPIO04

  # TM1628 sniffer
  stb_gpio: 5
  clk_gpio: 6
  dio_gpio: 7
```

To use different pins, change these values and re-flash.

### Button Outputs

Buttons are implemented as ESPHome `output`-based buttons:

```yaml
button:
  - platform: output
    name: "Power"
    output: power_output
    duration: 0.15s
  - platform: output
    name: "Ion"
    output: ion_output
    duration: 0.15s
  - platform: output
    name: "Heat"
    output: heatwater_output
    duration: 0.15s
  - platform: output
    name: "Intensity"
    output: intensity_output
    duration: 0.15s
```

If your humidifier reacts slowly, increase `duration` to `0.2s`–`0.3s`.

### TM1628 Sniffer Component

Custom component is defined in `components/tm1628_sniffer` and used in YAML as:

```yaml
external_components:
  - source:
      type: local
      path: components
    components: [tm1628_sniffer]

binary_sensor:
  - platform: tm1628_sniffer
    stb_pin: 5
    clk_pin: 6
    dio_pin: 7

    temperature:
      name: "Temperature"
      filters:
        - delta: 1.0
        - throttle: 2s

    humidity:
      name: "Humidity"
      filters:
        - delta: 1.0
        - throttle: 2s

    power_sensor:
      name: "Power"
      filters:
        - delayed_on: 200ms
        - delayed_off: 200ms

    heat_sensor:
      name: "Heating"
      filters:
        - delayed_on: 200ms
        - delayed_off: 200ms

    ion_sensor:
      name: "Ionization"
      filters:
        - delayed_on: 200ms
        - delayed_off: 200ms

    no_water_sensor:
      name: "No Water"
      filters:
        - delayed_on: 200ms
        - delayed_off: 200ms
```

(Exact YAML may differ; refer to the current `steba-humidifier.yaml` in the repo.)

---

## Troubleshooting

### Flashing Problems

| Problem                              | Possible Fix                                                                 |
|--------------------------------------|------------------------------------------------------------------------------|
| Board not detected as serial port    | Re-plug USB, change cable/port, install CH340 driver                        |
| "Timed out waiting for packet header"| Enter download mode (BOOT+RESET), use `--chip esp32c3`                      |
| "Failed to open serial port"         | Close other apps using the port (Arduino IDE, serial monitor, etc.)        |

### Runtime Problems

| Problem                      | Possible Fix                                                                 |
|-----------------------------|------------------------------------------------------------------------------|
| No TM1628 data / 0 values   | Check wiring of STB/CLK/DIO, pull-ups, shared ground, check logs            |
| Buttons do nothing          | Verify button-pad connections, increase pulse `duration`, test GPIO outputs |
| Wi-Fi not connecting        | Recheck `wifi_ssid` / `wifi_password`, use fallback AP to reconfigure       |
| Unstable readings           | Increase `delta` and `throttle` filters, reduce log level from `DEBUG`      |

View live logs:

```bash
esphome logs steba-humidifier.yaml
```

---

## Development Notes

- The TM1628 sniffer is implemented in C++ (`tm1628_sniffer.cpp`) and exposes sensors/entities to ESPHome.
- ESPHome framework: `esp32` / `esp-idf`.
- Project metadata in YAML:

  ```yaml
  project:
    name: "barma-lej.steba-humidifier"
    version: "${project_version}"
  ```

---

## Contributing

1. Fork the repository.
2. Create a feature branch:

   ```bash
   git checkout -b feature/your-change
   ```

3. Test on your own Steba model (please mention the exact model in PR).
4. Open a Pull Request with:
   - Description of the change
   - Tested hardware (model, wiring)
   - Logs or screenshots from Home Assistant if relevant.

---

## License & Safety

### License

This project is licensed under the **MIT License**.
See the [`LICENSE`](LICENSE) file for details.

### Safety Disclaimer

> **⚠ WARNING**
> Modifying mains-powered appliances is inherently dangerous and will void the manufacturer's warranty.

- Always disconnect the humidifier from mains power before working on it.
- Ensure proper insulation and strain relief for all wires.
- Avoid exposing electronics to moisture or water.
- Double-check polarity and voltage compatibility before powering on.

Use this project **at your own risk**.

---

**💡 Tip:** If you like this project just buy me a cup of ☕️ or 🥤:

[![Buy Me A Coffee](https://www.buymeacoffee.com/assets/img/custom_images/white_img.png)](https://www.buymeacoffee.com/barma)
