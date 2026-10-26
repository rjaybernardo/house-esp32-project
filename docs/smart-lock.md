# ESP32 smart lock

**Config:** [`esphome/project-smart-lock.yaml`](../esphome/project-smart-lock.yaml) (validated against ESPHome 2026.10)
**Unlock methods:** fingerprint, keypad PIN, inside exit button, phone app, touch panel. Every unlock is logged in Home Assistant.

## 1. Architecture: split the secure side from the outside

```
   OUTSIDE (attackable)          │ door │              INSIDE (secure)
                                 │      │
  R503 fingerprint ──UART + IRQ──┼──────┼──► ESP32 ──GPIO23──► MOSFET ──► lock coil
  3x4 keypad ───────7 wires──────┼──────┼──►   │                   ▲
                                 │      │      │                   │
                                 │      │   reed switch       12 V PSU + battery backup
                                 │      │   exit button       (buck → 5 V for ESP32)
```
Only **signal** wires go outside. If someone pries off the keypad, the most they can do is short logic lines. There's no coil wire to bridge with a 12 V battery. **This is the most common flaw in DIY locks.**

## 2. Choose the actuator

| Actuator | Power-loss behavior | Good for | Watch out |
|---|---|---|---|
| **12 V solenoid bolt / cabinet lock** | Fail-secure (stays locked) | Cabinets, inner doors, prototyping | Not continuous-duty, so pulse it (config: 5 s). Can draw 0.5–2 A. |
| **Electric strike** (in the frame) | Selectable fail-safe or fail-secure | **Main doors**. You keep your existing lockset and handle | Inside handle still opens mechanically, which is good for fire exits. Check the current rating. |
| **Motorized deadbolt retrofit** (gear motor/servo on the thumb-turn) | Stays in its last position | Retrofit without cutting the frame | Needs bolt-position feedback (microswitch/hall) and stall/current sensing. More firmware work. |
| **Maglock** | Fail-safe (opens when power is lost) | Gates, commercial doors | Needs constant power. Unlocks on power loss, so you need a UPS. |

**Fire safety:** the door must always open from inside **without power, phone, or code** (mechanical handle or thumb-turn), and there must be a **mechanical key override** outside for when the electronics fail.

## 3. Pinout (ESP32 DevKit)

| Function | GPIO | Notes |
|---|---|---|
| Lock coil (MOSFET gate) | 23 | 100 Ω series, 10 kΩ pull-down, flyback diode across the coil (1N5819/SS34) |
| Status LED | 2 | Onboard LED |
| Door reed switch | 22 | Internal pull-up, switch to GND |
| Exit button (inside) | 19 | Internal pull-up, switch to GND |
| Fingerprint RX / TX / wake | 16 / 17 / 4 | R503 at 3.3 V, 57600 baud |
| Keypad rows | 32, 33, 25, 26 | |
| Keypad columns | 27, 14, 13 | |

## 4. Security built into the config
- **No web server** on the lock. The only control path is the **encrypted** Home Assistant API, and OTA uses the same key.
- **Keypad lockout:** after 5 wrong codes the keypad ignores input for 60 s (`max_wrong_codes`, `lockout_time`).
- **The PIN is in `secrets.yaml`**, so it's never committed to git.
- **Auto-relock:** the coil is pulsed and the lock always returns to `LOCKED`.
- **The door reed switch** reports whether the door is actually closed, not just whether "lock" was commanded.

### Still to do for production or a product
- [ ] Enable **flash encryption + secure boot** on the ESP32, so nobody can read the PIN or key from the chip by desoldering it.
- [ ] Require confirmation for phone unlocks in HA. The HA companion app can ask for biometrics on lock actions.
- [ ] HA automations: notify on every unlock and on "door left open > 2 min", and auto-lock at night.
- [ ] Battery backup sized for the lock + ESP32 for at least 24 h. Monitor the battery voltage on an ADC1 pin (GPIO34–39).
- [ ] Tamper switch on the outside unit → alert.

## 5. Enroll fingerprints
After the lock appears in Home Assistant: **Developer tools → Actions → `esphome.front_door_lock_enroll_fingerprint`**, `finger_id: 1`, `num_scans: 2`. Place the finger when the reader's ring LED asks. Delete with `delete_fingerprint`.
