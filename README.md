# house-esp32-project

A DIY smart home built on ESP32: control the lights, cabinets, locks and sensors from a **phone app** and a **wall touchscreen**. It's built in my own house first, as groundwork for a smart-home and smart-cabinet business.

**Stack:** [ESPHome](https://esphome.io) firmware on each ESP32, with [Home Assistant](https://www.home-assistant.io) as the hub (phone app, automations, remote access).

```
Phone app ─────┐                                         ┌─ Cabinet LED controllers (ESP32)
               ├─ Wi-Fi ─► Home Assistant (hub) ─ Wi-Fi ─┼─ Wall lights (Shelly/Sonoff, ESP32-based)
Touch panel ───┘           Pi 5 / mini PC                ├─ Smart lock (ESP32)
(ESP32-S3)                    │                          ├─ Sensors: mmWave, doors, temp
                              └─ remote access ─►        └─ BLE devices via ESP32 Bluetooth proxy
                                 phone anywhere
```

## Devices

| Device | Config | Status |
|---|---|---|
| Bench test (new-board check) | [`esphome/bench-test.yaml`](esphome/bench-test.yaml) | ✅ Validated + compiled |
| Kitchen cabinet lights: CCT strip, door sensor, mmWave, local button | [`esphome/kitchen-cabinet-lights.yaml`](esphome/kitchen-cabinet-lights.yaml) | ✅ Validated, needs hardware |
| Smart lock: fingerprint, keypad, exit button, phone | [`esphome/project-smart-lock.yaml`](esphome/project-smart-lock.yaml) | ✅ Validated, needs hardware |
| Home Assistant hub + phone app | — | ⏳ Next |
| Touch wall panel (ESP32-S3 + LVGL) | — | ⏳ After the hub |
| Wall lights (Shelly / Sonoff behind each switch) | — | Planned |
| IR blaster for aircon / TV | — | Planned |

## Docs
- [Modules to buy](docs/modules-to-buy.md): **first order with PHP prices**, what to buy, what to check on each, ready-made Bluetooth products, starter cart
- [Wiring](docs/wiring.md): cabinet lights, mains switch, smart lock, battery/UPS options
- [How it works & phone apps](docs/how-it-works-and-phone-apps.md): Wi-Fi, Bluetooth, internet access, building your own app
- [Smart lock](docs/smart-lock.md): architecture, actuator choice, security
- [Light control hardware](docs/light-control-hardware.md): modules per type of light
- [ESP32 IoT reference](docs/esp32-iot-reference.md): where logic lives, pin and power gotchas
- [Architecture & business path](docs/architecture-and-business.md): central control, improvements, home → business plan

## Repo layout
```
esphome/
  common/base.yaml        shared: board, Wi-Fi, encrypted HA API + OTA (no open web server)
  *.yaml                  one config per device
  secrets.yaml.example    copy to secrets.yaml (git-ignored)
docs/                     design notes, wiring, buying guide
```

## Flash a device
```bash
uv tool install esphome
cd esphome && cp secrets.yaml.example secrets.yaml   # Wi-Fi + keys: openssl rand -base64 32
sudo usermod -aG uucp $USER                          # serial port access (log out/in once)
esphome run bench-test.yaml                          # first flash over USB, later over Wi-Fi
```
