# Low-Cost Return to Home (RTH) for openXsensor (RP2040)

> **[Deutsche Anleitung hier lesen (README.md)](README.md)**

A simple, **low-cost Return-to-Home (RTH) system** based on the **Raspberry Pi RP2040** and **openXsensor (oXs)**. It brings your model airplane safely back to the starting point and lets it circle reliably overhead—without requiring a complex flight controller.

> **Source Code Repository:**  
> The full source code is maintained in the central project repository:  
> **[Networker-001/oXs_on_RP2040-add-ons](https://github.com)**

---

## The Core Idea: Simple, Safe & Affordable

Unlike full autopilots (such as INAV or ArduPilot), this RTH extension keeps things intentionally simple and transparent:

* **Low-Cost Hardware:** Built on a tiny RP2040 board (e.g., RP2040-Zero) paired with a standard GPS module and an optional MPU6050 gyro sensor.
* **Stabilization Required:** The system relies on basic flight stabilization (either using the built-in **MPU6050 gyro** of oXs or a **stabilized receiver**).
* **Full Control Stays in Your Transmitter:** oXs does **not** take over your servos directly. Instead, it sends a simple steering recommendation via telemetry (`HEADING`) to your radio. All mixing, limits, and manual stick overrides remain safely in your transmitter.
* **Rudder & Aileron Mixing for Clean Turns:** In your transmitter, the RTH signal is gently mixed onto **Rudder** (and lightly onto **Aileron** for coordinated turns) while the gyro keeps the wings level and maintains pitch.
* **Easy & Safe Testing with an RC Car:** Because the guidance logic works identically in 2D, you can mount the hardware on a **standard RC car** and test all settings, directions, and circle radii safely on the ground before your first flight.

---

## Hardware Setup & Test Platform

*(Upload your photos to an `images/` folder in this repository to display them here)*

![Hardware Overview](images/setup_overview.jpg)
*Typical setup: RP2040-Zero, GPS module, and MPU6050 gyro sensor.*

![RC Car Testbed](images/rc_car_test.jpg)
*Safe ground testing: Verifying the RTH circle radius and steering direction on an RC car.*

---

## How It Works in Practice

![Flight Path Diagram](images/rth_flight_path.png)
*The model approaches the Home circle on a straight tangent line and smoothly enters a clockwise orbit.*

1. **Automatic Home Lock:** After powering on, oXs waits for a solid GPS fix and automatically saves your launch position as the **Home point**.
2. **Activating RTH:** When you flip the RTH switch on your transmitter, oXs calculates the exact course back to a virtual circle around your Home point.
3. **Smooth Approach & Orbit:** Your model turns toward the edge of the circle, flies straight toward it, and smoothly transitions into a **continuous clockwise orbit** around you until you switch RTH off or take over with the sticks.

---

## Basic Configuration (USB Terminal)

Connect the RP2040 to your PC via USB (`115200 baud`, CR/LF enabled) and set just two RTH parameters in addition to your normal oXs pin configuration:

| Command | Meaning | Recommended for RC Car | Recommended for Glider / Plane |
| :--- | :--- | :--- | :--- |
| `RTD=<meters>` | **Orbit Radius** around Home | `3` to `5` (meters) | `100` to `150` (meters) |
| `RTS=<km/h>` | **Minimum Speed** for steering updates | `1` to `3` (km/h) | `5` to `12` (km/h) |
| `SAVE` | **Save settings** permanently | — | — |

---

## Transmitter Setup (EdgeTX / OpenTX)

oXs transmits the steering command through the standard **`HEADING` (`Hdg`)** telemetry sensor (`0` to `100`, where **`50.0` is neutral / straight ahead**).

### Step 1: Center the Telemetry Signal (Input)
Create a new Input (e.g., `I:RTH`):
* **Source:** `Hdg` (Telemetry sensor)
* **Weight:** `100%`
* **Offset:** `-50%` *(Turns `50.0` neutral into exact `0%` stick center)*

### Step 2: Mix onto Rudder & Aileron (Mixer)
Add a mixer line controlled by your **RTH Switch** (using **Multiplex: Add** so your sticks always override the RTH):
* **Rudder Channel:** Add `I:RTH` with **`25%` to `40%` weight** as a safe starting value for gliders *(for RC car ground testing: `100%` to `200%`)*.
* **Aileron Channel (optional):** Add `I:RTH` with **`10%` to `20%` weight** to support smooth, coordinated banking into the turn.
* **Stabilization:** Make sure your gyro stabilization mode is active whenever RTH is switched on.

> **Tuning Tip for Maiden Flights:**  
> Assign the rudder mixer weight to a potentiometer or side slider (e.g., `S1` / `LS`) during your first test flights. This lets you smoothly dial in the ideal turn authority in the air without oscillation, and save the final percentage afterward.

> **Important Direction Check Before Takeoff:**  
> Once GPS Home-Lock is active, walk about `15–20 m` away from your starting point. Walk forward at a brisk pace, flip the RTH switch, and check that the rudder deflects to steer the model in a **clockwise** turn back toward Home. If it steers the wrong way, simply invert the `+` / `-` sign of the mixer weight.

---

## Installing the Firmware

1. Download **`oXs.uf2`** from the [Releases](../../releases) page.
2. Hold the **BOOT** button on the RP2040 board while connecting it to your PC via USB.
3. Copy `oXs.uf2` onto the `RPI-RP2` drive that appears.
4. Open a serial terminal, enter your pins, `RTD`, and `RTS`, and type `SAVE`.
