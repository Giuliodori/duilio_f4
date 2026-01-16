# 4. Power System

## 4.1 Power domains overview
DUILIO F4 is a **control board**. It has **no power stages** on-board and is intended to drive **external motor drivers** only.

- **Logic power (5 V domain):** board electronics, communications, logic I/O.
- **Servo power (5 V rail):** RC/servo output headers.
- **Motor power:** always external. **Motor power never flows through DUILIO F4.**

DUILIO F4 can be supplied from a wide input **VIN (6 V to 43 V)** which is regulated to 5 V for logic and auxiliary rails.
VIN is for the **control electronics and low-power outputs only**. It is **not** a motor supply.
Input transient protection is provided on VIN, but requires an external fuse for correct operation.

WARNING: Do not route motor power through DUILIO F4.

> NOTE: Power distribution and current limits (system-level)
> - Individual pin current ratings are maximum limits, not guarantees.
> - The sum of all output currents depends on the active power source.
> - When powered from VIN: total available 5 V current approx. 3 A continuous, 5 A peak (shared).
>- When powered ONLY from USB or Raspberry Pi GPIO: total available 5 V current is typically < 0.8 A and depends on the actual current capability of the USB source or Raspberry Pi supply.

> - Exceeding the total budget may cause voltage drop, reset, or thermal shutdown.

## 4.2 Power scenarios
DUILIO F4 supports three practical power scenarios. The total 5 V current budget is defined by the system-level limits above.

- **VIN-powered system:** VIN feeds the onboard regulation, providing 5 V rails for logic and auxiliary outputs. Use this for full system operation with external drivers.
- **USB-only logic power:** USB powers logic for bench setup and firmware work. This is limited to low current loads and should not feed external peripherals.
- **Raspberry Pi stack power:** 5 V is supplied through the Raspberry Pi header when the link is closed. Use only for compact Pi-based systems with low 5 V load.

WARNING: Use one 5 V source at a time and avoid multiple active 5 V inputs to prevent back-powering.

## 4.3 Logic power inputs
Logic power can be sourced from the following 5 V inputs. **Use ONE source at a time.**

- **USB 5 V**
  - Use for: bench setup, firmware work, quick checks.
  - Limitations: USB current is limited; not suitable for servos or external 5 V loads.
  - WARNING: Do not combine USB power with any other 5 V source (risk of back-feeding a host).

- **External regulated 5 V**
  - Use for: installations with a dedicated 5 V regulator / BEC.
  - Requirements: regulated 5 V within tolerance, sized for board + intended 5 V peripherals.
  - Safety: keep wiring short, use proper gauge, and avoid noisy/long 5 V runs.

- **Raspberry Pi 5 V (HAT / header)**
  - Use for: compact Pi-based systems where the Pi provides the only 5 V source.
  - Limitations: do not use Pi 5 V to power servos or external loads.
  - WARNING: Incorrect jumper settings or simultaneous USB power can back-power the Pi or the USB host.

WARNING: One 5 V source, one direction of power flow.

## 4.4 Raspberry Pi power path
DUILIO F4 can power a Raspberry Pi from its 5 V rail (up to **5.1 V / 5 A**).
This path is **disabled by default**: the Raspberry Pi 5 V link is an **open solder jumper**.

When the Pi 5 V link is closed, the DUILIO 5 V rail is connected to the Raspberry Pi 5 V pins.
This connection is **bidirectional** by nature, therefore the system must be designed with a single intended power direction.

WARNING: BACK-POWER RISK
- If the Pi link is closed and USB power is applied to either board, back-powering can occur.
- Incorrect configuration can create unsafe current paths and can damage:
  - DUILIO F4
  - Raspberry Pi
  - USB host / PC

## 4.5 Servo power output
DUILIO F4 provides a dedicated 5 V servo power rail for the RC/servo output headers.

- The servo rail is separated from logic power.
- Current protection is provided by a **PTC resettable fuse** (RC/servo output current limited to approximately **2 A**).
- Protection diodes help prevent unsafe reverse current paths when an external 5 V source is present.

Use this rail only for **small RC servos and receivers** within the current limit and total 5 V power budget.

WARNING: Do NOT use the servo rail to power high-current loads or motors.
WARNING: A PTC fuse is not a power supply. It only limits fault current and may trip under load peaks.

## 4.6 Backup battery
An optional **3 V backup battery** can be connected to preserve position memory and avoid re-homing.
It is **not** a primary power source and does **not** power the main 5 V rail.

- With the battery installed: position memory is retained when main power is removed.
- Without the battery: the board operates normally but loses that memory on power-off.

WARNING: Observe polarity and use only the specified **3 V** battery.
Reverse polarity or higher voltage can damage the board.

## 4.7 Grounding strategy
DUILIO F4 requires a **common ground (GND)** with external devices such as motor drivers, sensors, and a Raspberry Pi.

Without a shared ground, control signals have no valid reference and the system will malfunction.

Typical grounding mistakes:
- Connecting only signal wires without GND.
- Using isolated supplies without bonding grounds where required.
- Relying on chassis/earth ground instead of signal ground.

WARNING: Always connect GND between DUILIO F4 and each external driver.

## 4.8 Power-related jumpers
Power-related jumpers are **solder links** that configure 5 V routing.
The main user-facing power jumper is the **Raspberry Pi 5 V link**, which is **open by default** to avoid back-powering.

Close the link only when a single power direction is intentionally chosen for the system.

WARNING: Do not change solder jumpers while the board is powered.
This can create short circuits or unsafe power paths.

Advanced jumper configurations are covered in Chapter 10.

DUILIO F4 integrates on-board protection elements intended to improve robustness against wiring errors and transient events.
These protections are **not a replacement for proper external power design**.

## 4.9 Power input protection and current limiting

### Input transient protection (TVS diode)

The main VIN input is protected by a **TVS diode connected in parallel** to the supply rails.
This device clamps high-voltage transients and protects the board against short overvoltage events and supply spikes.

To operate correctly, the TVS diode **requires an external fuse** installed upstream of the board power input.
Without an upstream fuse, the TVS may be forced to dissipate excessive energy during a fault condition.

**Recommended external protection:**
- One fuse in series with VIN
- Typical rating: **2 A to 5 A**, depending on supply capability and application
- Fast or automotive-type fuse recommended

The external fuse ensures that, in the event of sustained overvoltage or reverse-energy conditions, the fault is safely cleared.

### 5 V rail protection (PTC fuses)

The regulated 5 V rails are protected by **two independent resettable PTC fuses**.
These devices limit current during overload or short-circuit conditions and automatically recover once the fault is removed.

PTC protection is applied to:
- The 5 V rail feeding RC servos and external peripherals
- The internal 5 V logic distribution

Each rail is protected by a **PTC fuse with a nominal hold current of approximately 2 A**.
PTC fuses are intended to **limit fault current**, not to regulate power.

Under sustained overload or high peak current, the voltage may drop and the PTC may trip temporarily until the fault is removed and the device cools down.


### Raspberry Pi power jumper and PTC bypass

When the Raspberry Pi 5 V solder jumper is **left open**, the 5 V rail supplying the Pi is protected by the PTC fuse.

When the solder jumper is **closed**, the PTC fuse is **bypassed** in order to provide the maximum available current to the Raspberry Pi.
This configuration is intended for applications requiring high peak current (e.g. Raspberry Pi 4 / 5 under load).

**Important considerations when the jumper is closed:**
- Current limiting relies entirely on the external power source and upstream protection
- Adequate supply capability and wiring are mandatory
- An external fuse on VIN is strongly recommended

Improper configuration or insufficient upstream protection can lead to excessive current flow and thermal stress.

