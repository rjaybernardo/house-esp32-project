# Central control architecture and business path

## 1. Central control: two architectures

### A. Hub + ESP32 nodes (recommended for your home now)
```
 Phone app ──┐                        ┌── ESP32 relay (living room)
             ├── Home Assistant hub ──┼── ESP32 LED controller (kitchen cabinets)
 ESP32-S3  ──┘   (Pi 5 / mini PC)     ├── ESP32 sensors (motion, door, temp)
 touch panel                          └── ESP32 IR blaster (aircon)
```
- **The ESP32 touchscreen is the central control *panel*.** Home Assistant is the brain behind it.
- Advantages: the phone app, history, schedules, voice assistants, and other brands' devices all work right away. You learn the whole stack quickly.
- Drawback: one extra box, a Raspberry Pi 5 or N100 mini PC at about USD 60–150.

### B. All-ESP32 (no PC hub; product R&D later)
```
 Phone (web UI / cloud app) ── ESP32-S3 panel = brain ──ESP-NOW/MQTT── ESP32 nodes
```
- The touchscreen panel *is* the hub. It holds the device list, scenes and schedules, and nodes talk to it over ESP-NOW or Wi-Fi.
- Advantages: lower parts cost and a single product to sell. It feels like a commercial system.
- Drawbacks: you write and maintain the firmware for the hub, the UI, OTA updates for everything, and remote phone access (cloud or ESP RainMaker). That's months of work.

**Plan:** build **A** at home to learn and to have something working. Every node you build stays ESPHome, so it carries over. Treat **B**, or a **Matter** product (below), as R&D once you know what customers actually want.

## 2. Ideas to improve the project

1. **Local fallback for every light.** The wall switch still works through a relay behind it, even if Wi-Fi is down. This is the difference between a smart home people love and one they curse.
2. **mmWave presence instead of PIR** for rooms where people sit still, like the living room, office and bathroom.
3. **CCT (tunable white) LED strips** in the cabinets, with *adaptive lighting* (warm at night, cool in daytime) set from Home Assistant.
4. **Energy monitoring** on main circuits and the aircon (BL0942 or a Shelly PM). People notice this feature and will pay for it.
5. **Design your own node PCB**: an ESP32 module, isolated supply, relay or MOSFET outputs, switch input, terminal blocks. You're an EE, so this is your edge. Prototype it at home, have it made at JLCPCB/PCBWay, and it becomes your product.
6. **Separate IoT Wi-Fi network (SSID/VLAN)** with no internet access for devices that don't need it. This improves security and is good practice before you install for customers.
7. **Keep everything in git** (already set up) and take automatic Home Assistant backups. A failed SD card shouldn't cost you a weekend.
8. **Document every circuit you wire**: breaker, box, wire colors, module. This becomes your installation playbook for customers.

## 3. Path to a business

| Stage | What you sell | Hardware | Risk / cost | Notes |
|---|---|---|---|---|
| **1. Your home** (now) | — | DIY + certified modules | Low | Build a portfolio. Photograph and video everything. |
| **2. Installer / integrator** | Design + install + support | **Certified off-the-shelf** (Shelly, Sonoff, Athom, certified PSUs) + Home Assistant or the panel | Low–medium | Fastest way to earn. You sell expertise, and the certifications are already done by the module makers. |
| **3. Smart cabinets** ⭐ | Furniture with built-in lighting, sensors, touch control | Your own low-voltage LED controller board + certified PSU | Medium | Your **unique advantage**: few people can both build cabinets and make them smart. Since it's low-voltage, there are fewer certification hurdles. |
| **4. Own hardware product** | Your own switches, panels, controllers | Your PCB, Matter-enabled | High | Needs radio type approval, electrical safety certification, Matter certification (CSA membership + testing), manufacturing, and support. |

### The customer phone app problem
Home Assistant is great for you but too technical for most customers. Options:
- **Matter** (ESP-Matter SDK on ESP32 / ESP32-C6): your devices appear in **Google Home, Apple Home, Alexa, and SmartThings**, so you never build an app. This is the strongest option for a product (stage 4). Certification costs money, so plan for it.
- **ESP RainMaker**: Espressif's cloud with ready-made phone apps that can be white-labeled with your brand. Medium effort.
- **Home Assistant with locked-down dashboards**: fine for stage 2 installs with tech-friendly clients.

### Before you sell anything
- **Electrical work for customers:** many countries require a licensed electrician for fixed mains wiring. Check the rules where you'll operate, and partner with an electrician if you need to.
- **Selling hardware:** radio products need **type approval** from the national telecom regulator. Using a pre-certified ESP32 module helps but may not be enough on its own. Mains-powered products need **electrical safety certification**. Check your country's requirements before stage 3 or 4.
- **Liability:** have a written warranty and scope for each install. Get business insurance once you have customers.

## 4. Suggested next 90 days
1. Weeks 1–2: Flash `bench-test.yaml`, then build the cabinet light on the bench (docs/wiring.md §1).
2. Weeks 3–4: Home Assistant hub + phone app; the first cabinet LED strip permanently installed.
3. Weeks 5–8: One room fully done: Shelly/Sonoff behind each switch, mmWave presence, scenes. Then the touchscreen panel.
4. Weeks 9–12: Whole house + energy monitoring. Start designing your own cabinet LED controller PCB (the stage 3 prototype).
