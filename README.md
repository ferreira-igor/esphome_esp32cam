# ESP32-CAM — ESPHome

ESPHome configuration for an **AI-Thinker ESP32-CAM**, providing a network-accessible camera stream together with basic device diagnostics and a remotely controlled camera flash.

The configuration uses the ESP32 Arduino framework, PSRAM, the camera's dedicated I²C bus, and ESPHome's camera web server.

## Features

- 📷 ESP32-CAM video stream
- 🌐 Camera stream available through ESPHome's camera web server
- 💡 Remote camera flash control
- 📶 Wi-Fi signal sensor
- ⏱️ Device uptime sensor
- 🌡️ ESP32 internal temperature sensor
- 🔧 ESPHome captive portal for Wi-Fi recovery
- 🔄 OTA firmware updates
- 🔐 Encrypted ESPHome API
- 💾 PSRAM enabled
- 💡 Built-in status LED
- 🖥️ ESPHome web server for device access

## Hardware

This configuration targets the **AI-Thinker ESP32-CAM**.

The source configuration references the following pinout:

https://randomnerdtutorials.com/esp32-cam-ai-thinker-pinout/

### Camera pinout

The camera uses:

| Function | GPIO |
|---|---:|
| External clock | GPIO0 |
| D0 | GPIO5 |
| D1 | GPIO18 |
| D2 | GPIO19 |
| D3 | GPIO21 |
| D4 | GPIO36 |
| D5 | GPIO39 |
| D6 | GPIO34 |
| D7 | GPIO35 |
| VSYNC | GPIO25 |
| HREF | GPIO23 |
| Pixel clock | GPIO22 |
| Power down | GPIO32 |

The camera's I²C bus is configured as:

| Function | GPIO |
|---|---:|
| SDA | GPIO26 |
| SCL | GPIO27 |

## Status LED

The status LED is connected to:

```yaml
GPIO33
```

and is configured as inverted:

```yaml
status_led:
  pin:
    number: GPIO33
    inverted: true
```

This means the GPIO logic is inverted to match the LED hardware.

## Camera flash

The camera flash is controlled through GPIO4 using the ESP32 LEDC peripheral:

```yaml
output:
  - platform: ledc
    pin: GPIO4
    channel: 2
    id: camera_flash
```

ESPHome exposes it as a monochromatic light:

```text
Flashlight
```

This allows the flash to be controlled from ESPHome-compatible integrations.

## Camera

The camera is configured as:

```yaml
esp32_camera:
  name: "Camera Feed"
```

The camera uses the dedicated I²C bus:

```yaml
i2c_id: camera_i2c
```

and the camera web server provides a live stream on port:

```text
8080
```

The configured mode is:

```yaml
mode: stream
```

Therefore, the camera stream is intended to be accessed through:

```text
http://<ESP32-CAM-IP>:8080/
```

Replace `<ESP32-CAM-IP>` with the IP address assigned to the ESP32-CAM.

> The exact page and stream behavior is provided by ESPHome's `esp32_camera_web_server` component.

## PSRAM

PSRAM is explicitly enabled:

```yaml
psram:
```

This is important for ESP32-CAM applications because camera frame buffers can require significantly more memory than a normal ESP32 application.

## Sensors

The configuration exposes three diagnostic sensors.

### Wi-Fi Signal

```yaml
- platform: wifi_signal
  name: "WiFi Signal"
```

Reports the Wi-Fi signal strength of the ESP32-CAM.

### Uptime

```yaml
- platform: uptime
  name: "Uptime"
```

Reports how long the device has been running since its last reboot.

### Internal Temperature

```yaml
- platform: internal_temperature
  name: "Internal Temperature"
```

Reports the ESP32's internal temperature sensor.

> This is the microcontroller's internal temperature, not the ambient temperature around the camera.

## Wi-Fi

Wi-Fi credentials are intentionally stored outside the YAML configuration:

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

The corresponding values should be defined in ESPHome's `secrets.yaml`.

For example:

```yaml
wifi_ssid: "Your WiFi Network"
wifi_password: "Your WiFi Password"
```

### Wi-Fi fallback access point

If the ESP32-CAM cannot connect to the configured Wi-Fi network, ESPHome can create a fallback access point:

```yaml
ap:
  ssid: ${DEVICE_NAME}
```

The configuration also enables:

```yaml
captive_portal:
```

allowing the device to be configured through the captive portal.

## ESPHome API

The native ESPHome API is enabled with encryption:

```yaml
api:
  encryption:
    key: !secret api_key
  reboot_timeout: 0s
```

The encryption key is stored in `secrets.yaml`.

Example:

```yaml
api_key: "your-base64-api-key"
```

The configuration sets:

```yaml
reboot_timeout: 0s
```

which disables the API client's automatic reboot timeout.

## OTA updates

OTA updates use the ESPHome platform:

```yaml
ota:
  - platform: esphome
    password: !secret ota_key
```

The OTA password is stored in `secrets.yaml`.

Example:

```yaml
ota_key: "your-ota-password"
```

## Web server

ESPHome's web server is enabled:

```yaml
web_server:
  ota: False
  log: False
```

OTA through the web interface is explicitly disabled, while web-server logging is also disabled.

The camera stream is provided separately through:

```yaml
esp32_camera_web_server:
  - port: 8080
    mode: stream
```

## Device name

The configuration uses a substitution:

```yaml
substitutions:
  DEVICE_NAME: esp32-cam
```

This value is used for the ESPHome device name:

```yaml
esphome:
  name: ${DEVICE_NAME}
```

and for the fallback Wi-Fi access point:

```yaml
ap:
  ssid: ${DEVICE_NAME}
```

The friendly name presented by ESPHome is:

```text
ESP32 CAM
```

## I²C configuration

The camera I²C bus is defined as:

```yaml
i2c:
  - id: camera_i2c
    sda: GPIO26
    scl: GPIO27
    scan: true
```

The bus is given the ID:

```text
camera_i2c
```

and is assigned to the camera component.

I²C scanning is enabled, which can be useful during development and troubleshooting.

## Installation

### 1. Install ESPHome

Install ESPHome using your preferred method.

For a Home Assistant installation, ESPHome can be installed and managed through the ESPHome add-on.

### 2. Create the configuration

Save the configuration as:

```text
esp32-cam.yaml
```

### 3. Configure secrets

Create or update:

```text
secrets.yaml
```

with at least:

```yaml
wifi_ssid: "Your WiFi Network"
wifi_password: "Your WiFi Password"
ota_key: "Your OTA Password"
api_key: "Your ESPHome API Encryption Key"
```

### 4. Compile and install

Compile the configuration and flash it to the ESP32-CAM.

For the first installation, a USB-to-serial adapter or another compatible programming method is typically required.

After the initial installation, OTA updates can be used.

## First boot

After flashing:

1. The ESP32-CAM initializes the ESPHome firmware.
2. It attempts to connect to the configured Wi-Fi.
3. If Wi-Fi is unavailable, the fallback AP can be started.
4. The ESPHome API becomes available after network connection.
5. The camera web server starts on port `8080`.
6. The diagnostic sensors become available.
7. The flashlight can be controlled through ESPHome.
8. OTA updates are available through the ESPHome API.

## Accessing the camera

Once the device has an IP address, access the camera web server at:

```text
http://<ESP32-CAM-IP>:8080/
```

The ESP32-CAM IP address can be obtained from the network/router or through the ESPHome/Home Assistant integration.

## Security considerations

The configuration already keeps the following credentials outside the YAML file:

- Wi-Fi password
- OTA password
- ESPHome API encryption key

Do not commit `secrets.yaml` to a public Git repository.

A typical `.gitignore` should include:

```gitignore
secrets.yaml
```

The camera web server itself is configured without authentication in this YAML. Therefore, access to the camera's web server should be restricted to a trusted network.

In particular, **do not expose port 8080 directly to the Internet** without an appropriate authentication and security layer.

## Important limitations

- The configuration is specifically intended for the AI-Thinker ESP32-CAM pinout.
- The camera requires PSRAM, which is enabled in the configuration.
- The camera stream is exposed on TCP port `8080`.
- The web server camera stream is not configured with authentication in this YAML.
- The internal temperature sensor measures the ESP32 itself and should not be interpreted as ambient temperature.
- The flash uses GPIO4 and may be subject to the electrical limitations of the ESP32-CAM board.
- The I²C bus is dedicated to the camera configuration.
- The exact camera sensor module installed on an ESP32-CAM can vary between boards; the YAML itself does not explicitly specify a camera model.

## Troubleshooting

### Camera does not initialize

Check:

- Correct ESP32-CAM board selection.
- Camera ribbon cable orientation.
- Camera module seating.
- PSRAM availability.
- GPIO assignments.
- Power supply stability.

### Camera stream is unavailable

Check that the ESP32-CAM is connected to Wi-Fi and then access:

```text
http://<ESP32-CAM-IP>:8080/
```

Also verify that port `8080` is not blocked by the network.

### Wi-Fi does not connect

Verify the values in:

```text
secrets.yaml
```

and check the fallback AP created using:

```text
esp32-cam
```

### Flash does not work

Verify GPIO4 and check the ESP32-CAM board's flash LED circuit.

The flash output uses LEDC channel 2.

### API does not connect

Verify that:

- the API encryption key matches;
- the ESP32-CAM is reachable on the network;
- the device has obtained an IP address;
- `reboot_timeout: 0s` is intentional.
