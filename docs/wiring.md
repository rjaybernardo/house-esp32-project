# Wiring

Pin numbers match the configs in `esphome/`. GND is common across the ESP32, the MOSFET sources and the PSU negative in every low-voltage circuit.

## 1. Kitchen cabinet lights: `kitchen-cabinet-lights.yaml`

```
 230 V AC ──► [certified 24 V PSU, e.g. Mean Well LPV/HLG] ──┬── +24 V ──────────────────► LED strip V+ (common anode)
                                                            │
                                                            └── [buck 24→5 V, e.g. MP1584] ──► ESP32 VIN (5V)
                                                                                                    
 ESP32 GPIO25 ──100Ω──┬── gate  Q1 (AO3400 / IRLB8721)   drain ──► strip WW (warm) channel −
                      10k                                source ──► GND
                      │
                     GND
 ESP32 GPIO26 ──100Ω──┬── gate  Q2                       drain ──► strip CW (cool) channel −
                      10k                                source ──► GND
                     GND

 ESP32 GPIO27 ───────── reed switch ───── GND        (magnet on the door; internal pull-up)
 ESP32 GPIO32 ───────── push/touch button ─ GND      (local control, internal pull-up)

 LD2410C:  5V ◄── ESP32 VIN     GND ── GND     TX ──► GPIO16     RX ◄── GPIO17
```
- An LED strip is a resistive load, so no flyback diode is needed. Fuse the 24 V rail (polyfuse or blade fuse) near the PSU.
- **Sizing:** PSU watts ≥ 1.25 × (strip W/m × metres). Check the MOSFET's R_DS(on) at **V_GS = 2.5 V**, and keep I² · R_DS(on) low enough for a SOT-23 package (AO3400: fine up to about 2–3 A per channel).
- For runs over about 5 m at 24 V, feed power at both ends to avoid voltage drop (warm-white tint at the far end).

## 2. Mains light behind an existing wall switch (Shelly 1 Gen3 / Sonoff Mini R4)

```
 Breaker ── L ──┬──────────────► module L
                │
 N ─────────────┼──────────────► module N ───────────────┐
                │                                          │
                └──► wall switch ──► module SW (input)     │
                                                           │
                      module O (relay out) ──► lamp ───────┘ (lamp other side to N)
```
- The wall switch becomes a **signal input**, so it still works without Wi-Fi.
- **You need a neutral in the switch box.** If there's only L and the switched leg, use a no-neutral module (e.g. Shelly 1L) or install the module at the ceiling fixture instead.
- Breaker off, and verify with a tester before touching anything.

## 3. Smart lock: `project-smart-lock.yaml`

```
 230 V ─► [12 V DC UPS module with built-in battery]  ── or ── [12 V PSU] ─► [3S Li-ion BMS/charger] ─┐
                                                                                                    │
                          ┌──────────────────────────── +12 V ────────────────────────────────────┘
                          │
                          ├─► lock coil (+) ─── coil (−) ─► drain Q1 (IRLB8721)  source ─► GND
                          │        └──── flyback diode SS34: cathode to +12 V, anode to drain ────┘
                          │
                          └─► [buck 12→5 V, MP1584 / LM2596] ─► ESP32 VIN

 ESP32 GPIO23 ─100Ω─┬─ gate Q1        10k gate→GND
 ESP32 GPIO22 ── door reed switch ── GND
 ESP32 GPIO19 ── inside exit button ── GND
 ESP32 GPIO2  ── onboard status LED

 R503 fingerprint (OUTSIDE):  V+ (3.3 V) ◄── ESP32 3V3   GND ── GND
                              TX ──► GPIO16   RX ◄── GPIO17   WAKEUP ──► GPIO4   3.3VT (touch power) ◄── 3V3
 3x4 keypad (OUTSIDE):        rows → GPIO32, 33, 25, 26     cols → GPIO27, 14, 13
```
- The ESP32, MOSFET and power stay **inside**. Only the reader and keypad lines cross the door (§1 of docs/smart-lock.md).
- The coil is inductive, so **the flyback diode is mandatory**. A Schottky (SS34) is fine for a solenoid. Add a TVS if the cable run is long.
- Put a 470–1000 µF bulk capacitor on the 12 V rail near the coil so the pull-in surge doesn't brown out the ESP32 buck.

## 4. Battery backup: "a charging module but 5 V"

What you're describing is a **5 V UPS / power-bank module**: one board that charges a Li-ion cell and boosts it to 5 V, with the load running from it while it charges.

### Option A: 5 V UPS module (simplest, for 5 V devices like the cabinet controller or a panel)
```
 USB-C / 5 V in ──► [IP5306-based UPS module, e.g. "MH-CD42" or "5V 2A 18650 UPS board"] ──► 5 V out ──► ESP32 VIN
                                   │
                              1× 18650 cell (protected)
```
- **Check:** "charge and discharge at the same time" (pass-through/load-sharing), output current ≥ 1 A, and **auto-shutdown at light load**. IP5306 chips switch off below about 45–50 mA. An active ESP32 on Wi-Fi draws ~80 mA, so it's usually fine, but it fails if you use deep sleep. Some boards have a "keep on"/no-sleep jumper, so pick one of those.

### Option B: TP4056 charger + boost (common, but has caveats)
```
 5 V in ──► [TP4056 + DW01 protection module ("03962A" type)] ──► B+/B− ── 18650
                                           OUT+/OUT− ──► [MT3608 boost → 5 V] ──► ESP32 VIN
```
- The TP4056 has **no proper load sharing**: with a load attached it may never terminate the charge. That's OK for a hobby backup, but not for a product. For a product, use a charger IC with power-path (e.g. **BQ24074**, **IP5306**, **ETA6003**).
- Get the version **with protection** (two extra chips, DW01 + 8205A) and use cells with their own protection too.

### Option C: run the ESP32 directly from the cell (most efficient)
Li-ion 3.0–4.2 V ──► **low-dropout LDO** (RT9080 / ME6211 / HT7833, ~100 mV dropout) ──► ESP32 **3V3** pin. This skips the boost-to-5 V-then-drop-to-3.3 V losses. Usable down to about 3.4 V. Good for battery sensors.

### For the 12 V lock
The simplest certified route is a **12 V mini DC UPS** (sold for routers and CCTV). DIY route: 3S Li-ion pack with BMS + a CC/CV charger, or a 12 V 7 Ah SLA with a float charger. Size it: (coil pulses + ~1 W of ESP32 and buck losses) × 24 h.

**Monitor the battery:** a resistor divider into **GPIO34** (ADC1). For a 12 V system use **100k (top) / 22k (bottom)**: 14.4 V charging → 2.6 V, which stays under the ~3.1 V ADC range at 12 dB attenuation. Add 100 nF across the 22k. That gives a battery-voltage sensor in Home Assistant, with an alert when it's low.
