# Kostengünstiges Return to Home (RTH) für openXsensor (RP2040)

> **[English Version available here (README_ENG.md)](README_ENG.md)**

Ein einfaches, **kostengünstiges Return-to-Home (RTH) System** auf Basis des **Raspberry Pi RP2040** und **openXsensor (oXs)**. Es bringt dein Flugmodell auf Knopfdruck sicher zum Startpunkt zurück und lässt es dort zuverlässig kreisen – ganz ohne komplizierten Flight Controller.

> **Zentraler Quellcode:**  
> Den vollständigen Programmcode für dieses Projekt findest du im Haupt-Repository:  
> **[Networker-001/oXs_on_RP2040-add-ons](https://github.com)**

---

## Das Grundprinzip: Einfach, sicher und günstig

Im Gegensatz zu ausgewachsenen Autopiloten (wie INAV oder ArduPilot) bleibt dieses RTH-System bewusst überschaubar und transparent:

* **Kostengünstige Hardware:** Ein kleiner RP2040-Mikrocontroller (z. B. RP2040-Zero für wenige Euro), ein einfaches GPS-Modul und ein Gyrosensor (MPU6050) genügen.
* **Stabilisierung erforderlich:** Das System benötigt eine grundlegende Fluglagestabilisierung – entweder über den **integrierten MPU6050-Gyro** von oXs oder über einen **stabilisierten Empfänger** (z. B. FrSky S6R/S8R, Jeti Assist, Lemon RX etc.).
* **Die Hauptsteuerung bleibt im Sender:** oXs greift **nicht** direkt in die Servos ein. Stattdessen sendet oXs lediglich eine Lenkempfehlung per Telemetrie (`HEADING`) an den Sender. Alle Mischer, Servowege und der jederzeitige manuelle Eingriff über die Steuerknüppel bleiben sicher im Sender.
* **Mischung von Seitenruder und Querruder:** Im Sender wird das RTH-Signal sanft auf das **Seitenruder** (und leicht auf das **Querruder** zur Unterstützung der Kurvenlage) gemischt, während der Stabi das Modell sauber in der Waage hält.
* **Gefahrlos mit einem RC-Car erprobbar:** Da die Kursberechnung am Boden exakt genauso funktioniert wie in der Luft, kannst du die gesamte Technik vor dem Erstflug auf einem **einfachen RC-Car** montieren und Wirkrichtung sowie Kreisradius in Ruhe auf der Wiese oder dem Parkplatz testen.

---

## Hardware-Aufbau & Testträger


![Hardware Übersicht](images/setup_overview.jpg)
*Typischer Aufbau: RP2040-Zero, GPS-Modul und MPU6050-Gyrosensor.*

![RC-Car Testträger](images/rc_car_test.jpg)
*Gefahrloser Bodentest: Überprüfung von Kreisradius und Lenkrichtung auf einem RC-Car.*

---

## So funktioniert der Rückflug in der Praxis

![Flugbahn Diagramm](images/rth_flight_path.png)
*Das Modell steuert den Zielkreis tangential an und geht ohne Überschwingen in einen Rechtskreis über.*

1. **Automatischer Home-Punkt (Anzeige im Feld `Sats`):** Nach dem Einschalten prüft oXs den GPS-Empfang und zeigt den Status direkt durch einen Offset im Telemetriefeld **`Sats`** an:
   * **Unter `50`:** Noch kein stabiler 3D-Fix.
   * **Ab `50` (`50` + Satellitenanzahl, z. B. `58`):** Stabiler GPS-Fix erkannt – der 9-Sekunden-Countdown zum Speichern läuft.
   * **Ab `100` (`100` + Satellitenanzahl, ab `106`):** Nach 9 Sekunden stabilem Empfang ist der Startpunkt als **Home-Position** gespeichert und RTH ist startklar.
2. **RTH aktivieren:** Legst du im Flug den RTH-Schalter am Sender um, berechnet oXs den optimalen Kurs zurück zu einem virtuellen Kreis um deinen Standort.
3. **Anflug und Kreisen:** Das Modell dreht auf die Kreisbahn ein, fliegt geradlinig darauf zu und geht weich in einen **kontinuierlichen Rechtskreis** über dem Startpunkt über – so lange, bis du RTH wieder ausschaltest oder selbst steuerst.

---

## Grundeinstellung am PC (USB-Terminal)

Verbinde den RP2040 per USB-Kabel mit dem PC (`115200 Baud`, CR/LF aktiv) und stelle zusätzlich zu deiner gewohnten oXs-Pinbelegung nur zwei Werte ein:

| Befehl | Bedeutung | Empfehlung RC-Car (Test) | Empfehlung Segler / Motormodell |
| :--- | :--- | :--- | :--- |
| `RTD=<Meter>` | **Ziel-Kreisradius** um den Startpunkt | `3` bis `5` (Meter) | `100` bis `150` (Meter) |
| `RTS=<km/h>` | **Mindestgeschwindigkeit** für Lenkbefehle | `1` bis `3` (km/h) | `5` bis `12` (km/h) |
| `SAVE` | **Einstellungen dauerhaft speichern** | — | — |

---

## Einrichtung im Sender (EdgeTX / OpenTX)

oXs überträgt das Lenksignal über den Standard-Telemetriewert **`HEADING` (`Hdg`)** im Bereich von `0` bis `100` (wobei **`50.0` genau Geradeaus / Neutral** entspricht).

### Schritt 1: Telemetriesignal zentrieren (Inputs)
Lege einen neuen Input an (z. B. `I:RTH`):
* **Quelle:** `Hdg` (Telemetriesensor)
* **Gewichtung:** `100%`
* **Offset:** `-50%` *(Verschiebt den Neutralwert `50.0` auf exakt `0%` Knüppelmitte)*

### Schritt 2: Auf Seiten- und Querruder mischen (Mischer)
Füge in den Mischern für Seite und Quer jeweils eine Zeile hinzu, die über deinen **RTH-Schalter** aktiviert wird (Mischertyp: **Addieren**, damit deine Knüppel jederzeit Vorrang haben):
* **Seitenruder:** `I:RTH` mit **`25%` bis `40%` Gewichtung** als Startwert beim Segler *(für RC-Car-Tests am Boden: `100%` bis `200%`)*.
* **Querruder (optional):** `I:RTH` mit **`10%` bis `20%` Gewichtung** für eine saubere Kurvenneigung ohne Schieben.
* **Stabilisierung:** Stelle sicher, dass beim Einschalten von RTH gleichzeitig dein Stabi/Gyro aktiv ist.

> **Tipp zum Einfliegen:**  
> Lege die Mischer-Gewichtung für das Seitenruder beim ersten Testflug auf einen Drehgeber oder seitlichen Schieber (z. B. `S1` / `LS`). So kannst du im Flug stufenlos testen, ab welchem Ausschlag das Modell ruhig und ohne Aufschaukeln kreist, und den ermittelten Wert danach fest im Mischer eintragen.

> **Wichtiger Wirkrichtungstest vor dem Start:**  
> Warte, bis im Telemetriefeld **`Sats`** ein **Wert ab `106`** angezeigt wird (Home-Position nach 9 Sekunden gespeichert), und gehe ca. `15–20 m` vom Startpunkt weg. Gehe zügig vorwärts, lege den RTH-Schalter um und prüfe, ob das Seitenruder so ausschlägt, dass das Modell eine **Rechtskurve** zurück zum Startpunkt einleitet. Lenkt es in die falsche Richtung, drehe einfach das Vorzeichen (`+` / `-`) bei der Mischer-Gewichtung um.

---

## Firmware aufspielen

1. Lade die fertige Firmware-Datei **`oXs.uf2`** unter [Releases](../../releases) herunter.
2. Halte die **BOOT-Taste** auf dem RP2040-Board gedrückt, während du es per USB mit dem PC verbindest.
3. Kopiere die Datei `oXs.uf2` einfach auf das neu erscheinende Laufwerk `RPI-RP2`.
4. Öffne ein serielles Terminal, gib deine Pins sowie `RTD` und `RTS` ein und speichere mit `SAVE`.
