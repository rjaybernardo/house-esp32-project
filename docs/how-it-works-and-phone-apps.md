# How ESP32 IoT works: Wi-Fi, Bluetooth, internet, and phone apps

## 1. What happens when you tap "light on" on your phone

```
 Phone app ──(1)──► Home Assistant ──(2)──► ESP32 ──(3)──► MOSFET/relay ──► light
     ▲                    │                    │
     └────────(5)─────────┴────────(4)─────────┘
```
1. The phone app sends `turn_on light.kitchen_cabinet` over a WebSocket to Home Assistant. On home Wi-Fi it goes directly; away from home it goes through the remote-access tunnel (§4).
2. Home Assistant sends the command over the **ESPHome native API**: a TCP connection on port 6053, protobuf messages, Noise encryption. The connection stays open, so there's no polling and no connect delay.
3. Inside the ESP32, the Wi-Fi driver and lwIP TCP/IP stack run as FreeRTOS tasks on core 0, and ESPHome's main loop runs your components. The `api` component decodes the message and calls the light. The light ramps the `ledc` PWM duty cycle, and the GPIO drives the gate.
4. The ESP32 **pushes** the new state back ("on, 80%").
5. Home Assistant forwards it to **every** connected screen: the phone, the touch panel, other phones.

On a local network, the whole round trip takes roughly **20–80 ms**.

The same chain runs in reverse for sensors. The reed switch changes, the ESP32 pushes `door: open`, and Home Assistant updates the screens and runs any automation, such as a phone notification.

## 2. Wi-Fi

| Topic | What to know |
|---|---|
| **Mode** | Each device is a Wi-Fi *station* (client) on your router. If it can't connect, it falls back to an *access point* ("X Setup" hotspot + captive portal) so you can fix the settings. |
| **Band** | 2.4 GHz only (the ESP32-C6 adds Wi-Fi 6 on 2.4 GHz). Make sure your router's 2.4 GHz is on. A separate **IoT SSID** is best. |
| **Addressing** | DHCP gives the IP and **mDNS** gives the name (`front-door-lock.local`). Home Assistant finds ESPHome devices automatically via mDNS. Set DHCP reservations on the router. |
| **Giving devices your Wi-Fi password** | DIY: compiled in from `secrets.yaml`. For customers: provisioning over **Bluetooth** (§3) or the captive portal, so nothing is hard-coded. |
| **Coverage** | Check the `WiFi Signal` sensor after installing. Below −75 dBm, add a mesh node/AP or move the device. |

## 3. Bluetooth (BLE)

The classic ESP32, ESP32-S3, and ESP32-C3/C6 have BLE. **The ESP32-S2 has none.** Wi-Fi and BLE share one radio and time-slice it, so both work at the same time but use more RAM.

| Use | How (ESPHome) | Why it matters for you |
|---|---|---|
| **Wi-Fi setup from a phone** | `esp32_improv` | The customer opens a web page or app and sends Wi-Fi credentials over BLE. This is the professional install flow, and the device never needs reflashing. |
| **Bluetooth proxy** | `bluetooth_proxy` | Every mains-powered ESP32 becomes a BLE antenna for Home Assistant. Cheap BLE thermometers, BLE locks, plant sensors and similar can then be read anywhere in the house. |
| **Presence** | `ble_presence`, or Bermuda in HA | Detect which room a phone or BLE tag is in, and run lights and AC by occupancy. **Never use it alone to unlock a door**, because BLE presence can be spoofed or relayed. |
| **Direct phone↔ESP32 control** | `esp32_ble_server` (advanced) or custom firmware | Control with no Wi-Fi or internet at all, e.g. a backup way to unlock the smart lock. Needs your own app (§5). |

## 4. Connecting everything through the internet

**Rule: never port-forward an ESP32 or Home Assistant to the internet.** Every remote path goes through the hub over an encrypted tunnel.

```
 Phone (anywhere) ──internet──► [remote access] ──► Home Assistant ──LAN──► all ESP32s
```

| Option | Cost | Effort | Notes |
|---|---|---|---|
| **Home Assistant Cloud (Nabu Casa)** | Monthly subscription | 2 clicks | Easiest. Also connects Google Home / Alexa voice. Funds HA development. |
| **Tailscale** (HA add-on + phone app) | Free for personal use | Low | Private VPN with no open ports. The phone reaches HA as if it were at home. |
| **Cloudflare Tunnel** | Free | Medium | A public HTTPS URL with no open ports. Add Cloudflare Access login on top. |
| **WireGuard on your router** | Free | Medium | Full home VPN. Good if your router supports it. |

The HA phone app takes an **internal URL** (at home) and an **external URL** (remote) and switches between them automatically.

**Between two houses or sites** (e.g. customers later): each site has its own hub, and you get remote support access via Tailscale. Don't run one shared cloud for many customers' devices until you're ready to run it as a product (§5, option C).

## 5. Phone app options, from zero code to fully your own

| | Option | Code you write | Best for |
|---|---|---|---|
| **A** | **Home Assistant Companion app** (Android/iOS) with dashboards you design | None (YAML/drag-and-drop dashboards) | **Your home now.** Also stage-2 installs for tech-friendly clients. |
| **B** | **Matter devices** → **Google Home / Apple Home / Alexa** apps | ESP32 firmware with ESP-Matter SDK (ESP-IDF, C++) | **A product you sell.** Customers use apps they already have, and you never maintain a phone app. Needs Matter certification to sell. |
| **C** | **Your own branded app + your own cloud** | Phone app + backend + firmware | Full control of brand and features. The largest effort, and it's ongoing. |
| **D** | **ESP RainMaker** (Espressif's cloud + open-source phone apps you can rebrand) | Firmware (RainMaker SDK) + app branding | A middle path between B and C. |

### If you build your own app (option C)
```
 Flutter app (Android + iOS, one codebase)
      │  HTTPS / MQTT over TLS
      ▼
 Your backend: MQTT broker (EMQX / Mosquitto / AWS IoT Core) + user accounts + database
      │  MQTT over TLS, one certificate per device
      ▼
 ESP32 firmware (ESPHome `mqtt:` component, or custom ESP-IDF code)
```
- **Flutter** (Dart) builds Android and iOS from one codebase. `mqtt_client` handles MQTT, and `flutter_blue_plus` handles BLE provisioning and direct control.
- Alternative: an app that talks to **Home Assistant's WebSocket API** instead of your own cloud. It's quicker, but each customer still needs an HA hub.
- What makes option C hard isn't the UI. It's the **accounts, per-device security, OTA rollouts, uptime, and support**. Start with A, and move to B or C once customers ask for it.

**Recommended path:** use A at home now, prototype B on an ESP32-C6 alongside it, and only build C if the business needs a brand experience that Matter can't give.
