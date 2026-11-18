## Physischer Hardware-Aufbau & Pinbelegung

Das System ist in drei logische Funktionsebenen unterteilt: Der **RC-Empfänger** bildet die obere Ebene, der **RP2040** arbeitet in der mittleren Ebene und das **GPS-Modul** schließt das System nach unten ab. Die Belegung ist in der vorkompilierten Firmware (`RTH_CRSF.uf2`) fest auf **GPIO 0** (Telemetrie) sowie **GPIO 28 / 29** (GPS) eingestellt, lässt sich aber jederzeit über CoolTerm anpassen.

### Physikalische Signal- und Verdrahtungs-Matrix (Top-Down)

```text
+-----------------------------------+

|           RC-EMPFÄNGER            |
|   (z.B. ELRS / CRSF, FrSky ...)   |
|                                   |
|   [+5V]  [GND]  [RX / TLM]        |  (Bei CRSF an Empfänger-RX)
+-----+------+--------+-------------+

      |      |        |
      |      |        |  (Direkt bei CRSF; 1 kOhm bei S.Port/HoTT/MPX)
      |      |        |
      v      v        v
+-----+------+--------+-------------+

|   [+5V]  [GND]  [GPIO 0]          |  TLM-Ausgang (Standard)
|                                   |
|                                   |
|           RP2040-ZERO             |
|         (oXs RTH-System)          |
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
|            GPS-MODUL              |
|        (z.B. u-blox M8 / M10)     |
+-----------------------------------+
```

### Zuleitungen am Waveshare RP2040-Zero

```text
========================================================================
ZULEITUNGEN AM WAVESHARE RP2040-ZERO
========================================================================

                                +-------------------------------------+

                                |             [ USB-C ]               | <-- USB-Anschluss oben
                                +-------------------------------------+
Empfänger [RX / TLM] <----------| [GP0] (TLM-Ausgang)            [5V] | <--- [+5V] (vom Empfänger)

                                | [GP1]                         [GND] | <--- [GND] (Empfänger & GPS)
                                | [GP2]                         [3V3] | ---> [+3.3V] (zum GPS)
                                |                              [GP29] | <--- GPS [TX]
                                |                +-------+     [GP28] | ---> GPS [RX]
                                | [GP3]          |  BOOT |     [GP27] |
                                | [GP4]          +-------+     [GP26] |
                                | [GP5]                        [GP15] |
                                |                                     |
                                | [GP6]          +-------+     [GP14] |
                                | [GP7]          | RESET |      [RX0] | (GP13)
                                |                +-------+      [TX0] | (GP12)
                                |          (RGB) <-- Status-LED       |
                                +-------------------------------------+

                                   |   |   |   |   |   |   |   |   |
                                  GP8 GP9 GP10 GP11 GND 3V3 GP22 GP21 GP20
```
