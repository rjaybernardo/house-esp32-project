# Hardware for controlling lights with ESP32

Pick the module based on **what kind of light** it drives and **whether the wall switch must keep working**. It should, every time: see esp32-iot-reference.md §2.

## 1. Low-voltage lights (12/24 V LED strips, cabinet, cove, under-shelf)

The safest category and the best place to start. Only the power supply touches mains, so buy a certified one.

| Module | Use | ESPHome component | Notes |
|---|---|---|---|
| **N-MOSFET module / your own board** (AO3400, IRLB8721, IRLZ44N) | Single-color strip, dimmable | `ledc` + `monochromatic` | Logic-level gate at 3.3 V. Add a gate pull-down. 1 channel per color. |
| **Multi-channel MOSFET board** (3–5 ch) | RGB / RGBW / CCT (warm↔cool white) strips | `rgb`, `rgbw`, `cwww` lights | CCT strips are the best fit for homes: one strip covers warm-white evenings and cool-white daytime. |
| **WS2812B / SK6812 addressable strip** | Per-LED color and effects, accent lighting | `esp32_rmt_led_strip` | 5 V (or WS2815 at 12 V for long runs). Needs a level shifter (74AHCT125) for reliable data at 3.3 V. Inject power every ~2–5 m. |
| **QuinLED Dig-Uno / Dig-Quad** | Ready-made ESP32 LED controller with fusing and level shifting | ESPHome or WLED | Good reference design to study before laying out your own board. |
| **H801 / generic "Wi-Fi LED controllers"** | Cheap off-the-shelf RGBW controller | Flashable (check the chip; some are ESP8266 or locked) | Cheap, but quality varies. |

## 2. Mains lights (230 V ceiling lights, wall switches, outlets)

| Module | Type | Keeps wall switch? | ESPHome | Notes |
|---|---|---|---|---|
| **Shelly 1 / 1PM Gen3, Shelly Plus 1** | Certified relay that fits behind the switch | ✅ `SW` input | ✅ (ESP32-based) | Goes behind the existing switch in the wall box. The PM version measures power. The most practical option for retrofits. |
| **Sonoff Mini R4** | Certified relay that fits behind the switch | ✅ `S1/S2` input | ✅ (ESP32, check the revision) | Similar to the Shelly and cheaper. |
| **Athom relays / wall switches / plugs** | Certified, **pre-flashed with ESPHome** | ✅ | ✅ out of the box | No flashing needed. Good for installs. |
| **Touch wall switches** (ESP-based, neutral-required) | Replaces the wall switch | It *is* the switch | Many are flashable | Check that a **neutral wire** is present in the switch box. Older homes often only have line + switched line. |
| **Opto-isolated relay module** (5 V coil, with flyback diode) | DIY relay | Wire the switch to a GPIO | `gpio` switch | For learning on the bench. Put it in an enclosure before it goes anywhere near a wall. |
| **SSR** (e.g. Omron G3MB-202P, zero-cross) | Silent switching, high inrush tolerance | Wire to a GPIO | `gpio` switch | Off-state leakage can make LED lamps glow faintly. |
| **AC dimmer module** (e.g. RobotDyn, triac + zero-cross) | Dims mains loads | — | `ac_dimmer` | Only for **dimmable** lamps. Phase-cut LED dimming can flicker, so test each lamp model. |
| **Smart bulbs** (Athom or other ESPHome-flashable bulbs) | Bulb replacement | ❌ Cutting power at the wall makes them unreachable | ✅ | Fine for lamps. Not great as the main lighting. |

### Wiring a relay behind the switch
```
Line ──┬──────────── Shelly/Sonoff L
       │
Neutral ─────────── Shelly/Sonoff N
                        │
Existing wall switch ── SW input (now just a signal, toggles the relay)
                        │
               Relay O ─┴── Lamp ── Neutral
```
The wall switch becomes an input. The lamp works from the switch, the phone, the touchscreen, and automations, and the switch still works if Wi-Fi or the hub is down (on-device automation).

## 3. Sensors worth adding

| Sensor | What for | ESPHome |
|---|---|---|
| **HLK-LD2410 / LD2450 mmWave** | Detects a **still** person (PIR only sees movement). Keeps the lights on while you sit still. | `ld2410`, `ld2450` |
| **PIR (HC-SR501, AM312)** | Cheap motion trigger | `gpio` binary_sensor |
| **Reed switch** | Cabinet or door open → light on | `gpio` binary_sensor |
| **BH1750 / LDR** | Ambient light: only switch on when it's dark | `bh1750`, `adc` |
| **SHT40 / BME280** | Temperature/humidity (more accurate than DHT22) | `sht4x`, `bme280_i2c` |
| **BL0942 / HLW8012** | Energy metering per circuit | `bl0942`, `hlw8012` |
| **IR LED + VS1838B** | Control aircon and TV | `remote_transmitter`, `remote_receiver` |

## 4. Wireless options between devices

| Link | Range / topology | Use for |
|---|---|---|
| **Wi-Fi** (all ESP32s) | Router-based | Mains-powered devices: relays, LED controllers, panels. Default choice. |
| **ESP-NOW** | ESP32↔ESP32 direct, no router, low latency | Battery buttons, fallback links, an all-ESP32 system with no hub (see the business roadmap) |
| **Thread / Zigbee** (ESP32-C6, ESP32-H2) | Low-power mesh | Battery sensors that last for years, and Matter-over-Thread products |
| **BLE** | Short range | Phone-proximity presence, cheap BLE sensors via ESPHome `bluetooth_proxy` |
