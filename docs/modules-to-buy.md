# Modules to buy, and what to check on each

**Priority:** ⭐ = buy first (hub + first room + cabinet), ○ = later.

Interactive version with checkboxes and running totals: [Smart Home Parts List](https://claude.ai/artifact/XhiMW8oXrsazpx7CHxxYWZ) (private to the owner).

## 0. First order: start here

Skip the generic ESP32 "37-in-1" starter kits. They're mostly beginner parts (DHT11, joystick, buzzer, 7-segment) this project doesn't use. This order covers what [`bench-test.yaml`](../esphome/bench-test.yaml) and [`kitchen-cabinet-lights.yaml`](../esphome/kitchen-cabinet-lights.yaml) need on the bench.

| Part | Qty | Search on Shopee/Lazada | Check | Est. price (PHP) |
|---|---|---|---|---|
| ESP32 DevKit V1 | 2–3 | `ESP32 DevKit V1 CP2102 Type-C` | CP2102 or CH340 chip visible in the photos, USB-C | ₱500–1,100 (₱220–380 each) |
| ESP32 expansion board | 1 | `ESP32 expansion board 38pin` | Pin count matches your DevKit (30 or 38). The DevKit is too wide for one breadboard. | ₱80–150 |
| Breadboard + jumper wires | 1 set | `breadboard 830`, `jumper wire M-M M-F` | Both M-M and M-F jumpers | ₱150–270 |
| Resistor + capacitor kits | 1 each | `resistor kit`, `electrolytic capacitor kit` | Includes 100 Ω, 10 kΩ, ≥100 µF | ₱200–400 |
| MOSFET, breadboard-friendly | 2–4 | `IRLZ44N` or `D4184 MOSFET module` | Through-hole, logic-level. Keep the SMD AO3400 for your own PCB. | ₱80–250 |
| 24 V CCT LED strip + 24 V adapter | 1 roll + 1 | `24V COB LED strip CCT`, `24V 2A adapter` | CRI ≥ 90. Use the wall adapter on the bench only; the permanent install gets a certified PSU. | ₱650–1,300 |
| Buck converter | 1 | `MP1584 buck` | Set it to 5.0 V **before** connecting the ESP32 | ₱40–100 |
| LD2410C + reed switches | 1 + 2 | `LD2410C`, `reed switch door sensor` | The **C** version of the LD2410. Reed gap ≥ 15 mm. | ₱330–650 |
| **Total** | | | | **≈ ₱2,000–4,200** |

Prices are rough estimates from typical Shopee/Lazada Philippines listings (October 2026), not live quotes. They vary by seller, and shipping is extra.

**Saving tips:** buy from one or two stores so you pay shipping once, and use free-shipping vouchers. PH electronics shops that also sell on Shopee/Lazada (Makerlab Electronics, Circuitrocks, e-Gizmo) tend to have consistent stock. Buy the full 5 m strip roll: 1–2 m for the bench, the rest for the real cabinet.

**Hold off until the cabinet light works:** the hub (learn Home Assistant in Docker on your laptop first), the touch panel, Sonoff Mini R4 relays (easier to find in PH than Shelly) and the lock parts.

## 1. Controllers (ESP32 boards)
| | Module | Check before buying |
|---|---|---|
| ⭐ | **ESP32 DevKit V1 (ESP32-WROOM-32E)**, 3–5 pcs | 4 MB flash, USB-UART chip **CP2102 or CH340** (both work on Linux), USB-C preferred. Get the **-32UE** version (u.FL antenna) for devices inside metal enclosures. |
| ○ | **ESP32-C6 DevKit** | Wi-Fi 6 + BLE + **Thread/Zigbee**, for Matter product prototyping (business stage 4) |
| ✗ | ESP32-S2, ESP8266 | Avoid: the S2 has **no Bluetooth**, and the ESP8266 is legacy |

## 2. Hub and phone
| | Module | Check |
|---|---|---|
| ⭐ | **Raspberry Pi 5 (4–8 GB)** + **NVMe/SSD** + official 27 W PSU, *or* an **Intel N100 mini PC** | Boot from SSD, not an SD card (SD cards wear out). Wired Ethernet to the router. |
| ⭐ | Home Assistant Companion app | Free, Android/iOS |
| ○ | **Zigbee USB dongle** (SONOFF ZBDongle-E or Home Assistant Connect ZBT-1) | Only if you buy Zigbee products (§7) |

## 3. Touch panel
| | Module | Check |
|---|---|---|
| ⭐ | **Guition ESP32-S3-4848S040** (4", 480×480) | **ESP32-S3 with 8 MB PSRAM** (N16R8). Display/touch chips (ST7701S + GT911) supported by ESPHome LVGL. Pick the **230 V-powered variant** for a standard wall box, or the 5 V one for the bench. |
| ○ | Sunton ESP32-8048S043 (4.3", 800×480) | Larger. Same checks: PSRAM, ESPHome display support |
| ○ | Cheap Android tablet + wall mount | The no-DIY fallback. Runs the HA dashboard in kiosk mode. |

## 4. Lighting
| | Module | Check |
|---|---|---|
| ⭐ | **CCT LED strip, 24 V** (warm + cool white) | 24 V over 12 V (less current and voltage drop), **CRI ≥ 90**, LEDs/m (COB strips = no dots), W/m for PSU sizing |
| ⭐ | **Certified LED PSU 24 V** (Mean Well LPV/HLG or equivalent) | Safety marks, wattage ≥ 1.25× load, IP rating if inside cabinets near the sink |
| ⭐ | **Logic-level MOSFETs**: AO3400 (SMD) or IRLB8721 (TO-220), or a ready "PWM MOSFET module" | Fully on at **V_GS = 2.5–3.3 V**. Check R_DS(on) at 2.5 V, not at 10 V. Plus 100 Ω and 10 kΩ resistors. |
| ⭐ | **Buck 24/12 → 5 V** (MP1584 or LM2596 module) | Input range covers your rail, ≥ 1 A output. Set to 5.0 V **before** connecting the ESP32. |
| ⭐ | **Shelly 1 Gen3 / 1PM Gen3** or **Sonoff Mini R4**, one per wall light | **Neutral in the switch box?** If not, use Shelly 1L. PM = power metering. ESP32-based, so it can be flashed with ESPHome. |
| ○ | **Athom** relays, plugs and switches (pre-flashed ESPHome) | Plug-and-play with HA, no flashing |
| ○ | WS2812B / WS2815 addressable strip + 74AHCT125 level shifter | For accent/RGB effects |
| ○ | RobotDyn AC dimmer (triac + zero-cross) | Only for dimmable mains lamps. Test each lamp model for flicker. |

## 5. Sensors
| | Module | Check |
|---|---|---|
| ⭐ | **HLK-LD2410C mmWave** | The "C" version has BLE configuration plus easy pins. Detects people standing still. |
| ⭐ | **Reed switches** (NC/NO, surface-mount, with magnet) | Gap distance ≥ 15 mm for cabinet doors |
| ○ | SHT40 or BME280 (I2C) | Temperature/humidity. Better than DHT22. |
| ○ | BH1750 (I2C) | Ambient light: only turn on lights when it's dark |
| ○ | IR LED (940 nm) + VS1838B receiver + 2N2222 driver | Aircon/TV control |

## 6. Smart lock
| | Module | Check |
|---|---|---|
| ⭐ | **R503 capacitive fingerprint reader** | 3.3 V, UART 57600, ring LED, IP-rated front, supported by ESPHome `fingerprint_grow` |
| ⭐ | **3x4 matrix keypad** (metal, weatherproof), or a keypad with **Wiegand** output | Wiegand keypads (often with built-in RFID) only need 2 data wires, and ESPHome has a `wiegand` component |
| ⭐ | **Electric strike 12 V** (main door) or **solenoid lock 12 V** (cabinet/inner door) | **Fail-safe vs fail-secure** (strike: selectable), coil current, duty cycle |
| ⭐ | IRLB8721 MOSFET + **SS34 flyback diode** + 1000 µF / 25 V capacitor | |
| ⭐ | **12 V mini DC UPS** (router/CCTV type) | Output ≥ 2 A, battery capacity for 24 h |
| ○ | PN532 NFC module | Tap a phone or card to unlock |

## 7. Ready-made products (buy instead of build)

If you just want something that works, these connect to the system with no DIY wiring.

### Bluetooth products an ESP32 can control
Every mains-powered ESP32 running ESPHome can be a **Bluetooth proxy** (`bluetooth_proxy:`). Home Assistant then sees BLE devices anywhere in the house. Some can also be controlled by the ESP32 directly with no hub (`ble_client`).

| Product | What it does | How it connects | Note |
|---|---|---|---|
| **SwitchBot Bot** | A small robot finger that presses an existing wall switch or button | BLE → ESP32 proxy → HA | No rewiring, so it's renter-friendly. Slow, and a battery to replace. |
| **SwitchBot Lock / Lock Pro** | Motorized thumb-turn lock retrofit | BLE → ESP32 proxy → HA | Commercial-grade for the main door, without DIY |
| **Nuki Smart Lock** | Motorized lock retrofit | BLE direct to ESP32 (community `nuki_lock` ESPHome component) or Matter | Can run fully local |
| **Xiaomi LYWSD03MMC thermometers** (flashed with pvvx/BTHome firmware) | Temp/humidity in every room | BLE → ESP32 (`pvvx_mithermometer`) or proxy | Very cheap, 1-year battery |
| **Shelly BLU Button / Door-Window / Motion** | Battery buttons and sensors | BLE (BTHome) → ESP32 proxy → HA | Wireless scene buttons anywhere |
| **Cheap BLE LED strip controllers** (ELK-BLEDOM type) | RGB strips | BLE → HA "LED BLE" integration, or `ble_client` | Hit-or-miss compatibility. Wi-Fi controllers are more reliable. |

**BLE limits:** about 5–15 m through walls. It's fine for battery sensors, buttons and locks. **For mains lighting, Wi-Fi (Shelly/Sonoff/your ESP32) or Zigbee is more reliable than BLE.**

### Zigbee products (needs the USB dongle in §2)
Aqara, IKEA, Sonoff Zigbee switches and sensors: large catalog, cheap, years on a coin cell, and a mesh network. A good fit for battery sensors around the house.

### Honest recommendation for the main door
For your **front door**, a commercial lock (SwitchBot/Nuki/Yale) integrated with HA is more secure than a first DIY build: it has tamper resistance, tested mechanics and certified batteries. Build the DIY ESP32 lock for **cabinets and inner doors** first, and move it to the front door once it has proven itself.

## Starter cart (first room + cabinet + lock prototype)
- [ ] 5× ESP32-WROOM-32E DevKit
- [ ] Raspberry Pi 5 4 GB + NVMe HAT + SSD + PSU *(or N100 mini PC)*
- [ ] 1× Guition ESP32-S3-4848S040 panel
- [ ] 5 m 24 V CCT COB strip + 24 V 60 W certified PSU
- [ ] 10× AO3400 or IRLB8721, resistor kit, SS34 diodes, capacitor kit
- [ ] 2× MP1584 buck modules
- [ ] 2× Shelly 1 Gen3 (or Sonoff Mini R4)
- [ ] 2× LD2410C, 5× reed switches
- [ ] R503 fingerprint reader, 3x4 metal keypad, 12 V solenoid lock, 12 V mini UPS
- [ ] 1× IP5306 5 V UPS board + 2× protected 18650 cells
- [ ] Breadboard, jumper wires, Wago connectors, project enclosures
