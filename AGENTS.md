# AGENTS.md: house-esp32-project

ESPHome **2026.10** device configs for a DIY ESP32 smart home. Home Assistant is the hub (phone app + touch panel).
The user is an electronics engineer who is new to IoT. Explain in plain English, and skip basic circuit theory.

## Ground truth
- `esphome config <file>.yaml` (Neovim `<leader>iv`) is the judge. **Every suggested change must pass it.** If you're unsure an option exists, say so instead of guessing.
- `esphome compile <file>.yaml` (`<leader>ib`) proves it builds. Flashing is `esphome run` (`<leader>if`).

## Project rules
- Every device starts with:
  ```yaml
  substitutions:
    device_name: kebab-case-name
    friendly_name: Title Case Name
  packages:
    base: !include common/base.yaml
  ```
  `common/base.yaml` already provides `esp32` (board `esp32dev`, framework `esp-idf`), `logger`, `wifi` + fallback AP, `captive_portal`, encrypted `api`, and `ota`. Don't repeat these.
- **Never add `web_server`** to a real device. It exposes an unauthenticated firmware upload. All control goes through the encrypted Home Assistant API.
- Secrets only via `!secret` (`wifi_ssid`, `wifi_password`, `fallback_ap_password`, `api_encryption_key`, `door_pin`). Never hard-code passwords, keys or PINs.
- Safety-critical behavior (local buttons, door → light, lock relock) runs **on the device** (`on_press`, `script`), not in Home Assistant, so it works without Wi-Fi.
- Comments explain *why*, not what.

## Current ESPHome syntax (2026). Older examples online are often wrong.
| Use this | Not this (old) |
|---|---|
| `ota:` then `- platform: esphome` (a list) | `ota:` with `password:` directly under it |
| OTA encryption reuses the API key (they **must** match), so no separate OTA key | `ota: password: ...` |
| `api:` → `actions:` → `- action: name` + `variables:` | `services:` / `- service:` |
| `esp32_rmt_led_strip` for WS2812B/SK6812 | `neopixelbus`, `fastled_clockless` |
| `ld2410:` + `uart_id:`, `binary_sensor: - platform: ld2410` → `has_target:` | custom UART lambdas |
| `fingerprint_grow:` + `uart:` | custom components |
| `!remove` to delete a key inherited from a package | editing `base.yaml` per device |
| `ignore_strapping_warning: true` inside the pin mapping | ignoring the warning |

Lambdas are C++: read state with `id(my_id).state`, and a trigger's value is `x`. A condition lambda must `return` a bool. Use substitutions inside a lambda as `"${name}"`.

## ESP32 (classic WROOM-32) pin rules
- **Never use GPIO 6–11** (internal flash).
- Strapping pins **0, 2, 5, 12, 15**: GPIO12 pulled high at boot = no boot. Use them only for outputs/LEDs that are safe at boot.
- **GPIO 34–39**: input only, **no internal pull-ups**.
- **ADC2 pins don't work while Wi-Fi is on.** Analog only on ADC1 (**GPIO 32–39**).
- Safe defaults: **4, 13, 16–19, 21–23, 25–27, 32, 33**.
- 3.3 V logic, not 5 V tolerant. Drive loads through a MOSFET or relay. Inductive loads need a flyback diode.

## Pins in use (don't clash when editing)
- `kitchen-cabinet-lights.yaml`: 25 warm PWM, 26 cool PWM, 27 door reed, 32 button, 16/17 LD2410C UART.
- `project-smart-lock.yaml`: 23 coil MOSFET, 2 status LED, 22 door reed, 19 exit button, 16/17/4 R503 fingerprint, keypad rows 32/33/25/26, cols 27/14/13.
- `bench-test.yaml`: 2 onboard LED.

## Docs in this repo
`docs/wiring.md` (circuits), `docs/esp32-iot-reference.md` (pin/power gotchas), `docs/how-it-works-and-phone-apps.md`, `docs/smart-lock.md`, `docs/modules-to-buy.md`.
