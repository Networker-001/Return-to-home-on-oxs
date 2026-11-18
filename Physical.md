## Physical Hardware Setup & Pinout

The system is divided into three logical functional layers: The **RC Receiver** forms the top layer, the **RP2040** processes the data in the middle layer, and the **GPS Module** forms the bottom layer. In the pre-compiled firmware (`RTH_CRSF.uf2`), the pinout is pre-configured to **GPIO 0** (Telemetry) and **GPIO 28 / 29** (GPS), and can be changed anytime via CoolTerm.

### Physical Signal and Wiring Matrix (Top-Down)

```text
+-----------------------------------+

|           RC RECEIVER             |
|   (e.g. ELRS / CRSF, FrSky ...)   |
|                                   |
|   [+5V]  [GND]  [RX / TLM]        |  (Connect to Receiver RX for CRSF)
+-----+------+--------+-------------+

      |      |        |
      |      |        |  (Direct for CRSF; 1 kOhm resistor for S.Port/HoTT/MPX)
      |      |        |
      v      v        v
+-----+------+--------+-------------+

|   [+5V]  [GND]  [GPIO 0]          |  TLM Output (Default)
|                                   |
|                                   |
|           RP2040-ZERO             |
|         (oXs RTH System)          |
|                                   |
|                                   |
|  [3.3V]  [GND]  [GPIO 28] [GPIO 29]
+-----+------+--------+--------+----+

      |      |        |        ^
      |      |        |        |
      v      v        v        |
+-----+------+--------+--------+----+

|  [3.3V]  [GND]  [GPS RX] [GPS TX] |
|                                   |
|            GPS MODULE             |
|        (e.g. u-blox M8 / M10)     |
+-----------------------------------+
```

### Wiring on the Waveshare RP2040-Zero

```text
========================================================================
WIRING ON THE WAVESHARE RP2040-ZERO
========================================================================

                                +-------------------------------------+

                                |             [ USB-C ]               | <-- USB Port on Top
                                +-------------------------------------+
Receiver [RX / TLM] <-----------| [GP0] (TLM Output)             [5V] | <--- [+5V] (from Receiver)

                                | [GP1]                         [GND] | <--- [GND] (Receiver & GPS)
                                | [GP2]                         [3V3] | ---> [+3.3V] (to GPS)
                                |                              [GP29] | <--- GPS [TX]
                                |                +-------+     [GP28] | ---> GPS [RX]
                                | [GP3]          |  BOOT |     [GP27] |
                                | [GP4]          +-------+     [GP26] |
                                | [GP5]                        [GP15] |
                                |                                     |
                                | [GP6]          +-------+     [GP14] |
                                | [GP7]          | RESET |      [RX0] | (GP13)
                                |                +-------+      [TX0] | (GP12)
                                |          (RGB) <-- Status LED       |
                                +-------------------------------------+

                                   |   |   |   |   |   |   |   |   |
                                  GP8 GP9 GP10 GP11 GND 3V3 GP22 GP21 GP20
```
