# ESP32 IoT reference for an electronics engineer

This page maps each part of the system to something you already know, and lists the ESP32 hardware gotchas worth knowing before you wire anything.

## 1. The stack

```
┌──────────────────────────────────────────────────────────────────┐
│ UI          Phone app  ·  Touchscreen wall panel  ·  Voice       │  what you touch
├──────────────────────────────────────────────────────────────────┤
│ Hub         Home Assistant: state of every device, automations,  │  the "controller"
│             dashboards, history                                  │
├──────────────────────────────────────────────────────────────────┤
│ Protocol    ESPHome native API (TCP 6053, encrypted)  or  MQTT   │  the "bus protocol"
├──────────────────────────────────────────────────────────────────┤
│ Network     Wi-Fi 2.4 GHz → IP → mDNS names (device.local)       │  the "physical bus"
├──────────────────────────────────────────────────────────────────┤
│ Device      ESP32 + ESPHome firmware + your circuit              │  sensors / actuators
└──────────────────────────────────────────────────────────────────┘
```

| IoT term | EE analogy |
|---|---|
| **Entity** (e.g. `light.kitchen_cabinet`) | A named net/signal in the system |
| **State** | The value on that net, plus attributes (brightness, color) |
| **ESPHome native API** | A point-to-point link. The device **pushes** changes when they happen (interrupt-driven, not polled) |
| **MQTT** | A shared bus. A broker is the arbiter, and topics like `home/kitchen/light/set` act as addresses. Publish/subscribe |
| **mDNS** (`kitchen-cabinet-lights.local`) | Auto-addressing. No need to hard-code IPs |
| **OTA update** | Dual-bank flash: new firmware goes into the inactive app partition (`ota_0`/`ota_1`), then it swaps on reboot and rolls back if it fails |
| **Automation** | Combinational/sequential logic: trigger → condition → action |
| **Hub** | The MCU in a bigger system: the devices are peripherals and it runs the logic across them |

## 2. Where should the logic live?

This is the main design decision in a smart home:

| Put it on… | When | Example |
|---|---|---|
| **The ESP32** (ESPHome `on_press`, `interval`) | It must work if Wi-Fi or the hub is down, or it needs fast response | Wall button toggles its own relay; motion → cabinet light; over-temp cutoff |
| **Home Assistant** | It involves more than one device, schedules, or presence | "Good night" turns everything off; lights on at sunset; notify the phone if a door opens at 2 AM |
| **The UI** | Never. UIs only send commands and show state | — |

Rule of thumb: **every light must still work from a local switch without the hub.** The phone and touchscreen add control on top, and they shouldn't be the only way to work a light.

## 3. Native API vs MQTT

**Use the native API** (the default in every config here). Home Assistant finds ESPHome devices automatically, the link is encrypted with a key, there's no broker to run, and it has the lowest latency.

**Add MQTT later** only if something other than Home Assistant needs the data, like Node-RED or your own app. ESPHome can do both at once.

## 4. ESP32 hardware gotchas (classic ESP32-WROOM-32)

| Pins | Gotcha |
|---|---|
| **GPIO 6–11** | Connected to the internal SPI flash. **Never use them.** |
| **GPIO 0, 2, 5, 12, 15** | Strapping pins, read at reset. GPIO0 low = download mode. **GPIO12 high at boot sets the flash voltage to 1.8 V and the board won't boot**, so never put a pull-up on it. GPIO2 must be low/floating to enter download mode. These are fine as outputs if you respect the boot state. |
| **GPIO 34–39** | Input only, with **no internal pull-ups/pull-downs**. Fit external ones. |
| **ADC2** (GPIO 0, 2, 4, 12–15, 25–27) | **Unusable while Wi-Fi is on.** Use ADC1 (GPIO 32–39) for analog. |
| **All GPIO** | 3.3 V logic, **not 5 V tolerant**. Keep to about 20 mA per pin for an LED. Use a transistor or MOSFET for anything bigger. |
| **Power** | Wi-Fi TX bursts draw **300–500 mA peaks**. A weak supply or long thin wires causes brownout resets (`Brownout detector was triggered` in the log). Put ≥100 µF bulk capacitance near the module and size the 3.3 V regulator for at least 500 mA. |
| **Relays at boot** | Some pins pulse or float during boot. Pick a pin that's stable at boot for relay/MOSFET drive (GPIO 16–19, 21–23, 25–27, 32, 33), and set `restore_mode` in ESPHome. |

Safe "default" pins for a new design: **GPIO 4, 13, 16–19, 21–23, 25–27, 32, 33** for I/O and **GPIO 32–39** for analog input.

## 5. Driving loads (details in wiring.md)

- **12/24 V LED strips:** low-side N-MOSFET with a logic-level gate, fully on at **V_GS = 3.3 V** (check R_DS(on) at 2.5–3.3 V in the datasheet, not at 10 V). IRLZ44N works, and AO3400 or IRLB8721 are better choices at 3.3 V. Use about 100 Ω in series with the gate and a 10 kΩ gate pull-down so the light stays off while the ESP32 boots. ESPHome's `ledc` PWM defaults to 1 kHz, and you can raise it to about 20 kHz to stop audible whine and camera flicker.
- **Mains (230 V) loads:**
  - Use an isolated AC/DC module such as the HLK-PM01 for the 5 V supply, plus a fuse on the line side.
  - Keep **≥6 mm creepage** between mains and SELV copper (reinforced insulation). Slots in the PCB help.
  - Relay modules need a flyback diode and preferably an optocoupler. Check the relay's **inrush rating**: LED drivers can pull tens of amps of capacitive inrush, which welds small 10 A relay contacts.
  - An SSR with zero-cross switching handles inrush well, but it leaks current through its snubber when off, which can make LED lamps glow faintly.
  - A certified ESP-based smart switch (flashable with ESPHome) inside a wall box is often the most practical option. Your own board inside a wall needs a proper enclosure.

## 6. Wi-Fi planning

- 2.4 GHz only. Check RSSI with the `wifi_signal` sensor (in every config): **better than −67 dBm is good, −67 to −75 is OK, below −80 is unreliable.**
- Metal cabinets and enclosures block the PCB antenna. Mount the module with the antenna end clear, or use a board with a u.FL connector for an external antenna.
- 20–30 ESP32 devices on one home router is usually fine. Give them DHCP reservations so their IPs don't change.
