# Berkey Fill Controller

Automatically refilling a [Big Berkey](https://www.berkeyfilters.com/) gravity
water filter, using an ESP32 microcontroller running
[ESPHome](https://esphome.io/).

A Big Berkey is a countertop water filter made of two stacked stainless steel
chambers. You pour water into the top chamber, it drips slowly through carbon
filter elements by gravity, and clean water collects in the bottom chamber where
a tap dispenses it. It works well, but somebody has to keep refilling the top by
hand.

This project does the refilling: two sensors watch the water level in both
chambers, a motorised valve opens the tap water supply when the filter is
running low, and a flow meter counts how much water goes in.

> ### Status, October 2026: designed, not yet filling
>
> The level sensing works and is verified in water. The fill logic is written
> but not yet flashed, and stays disabled until the probes are mounted and
> calibrated. A custom carrier board (revision G) with the relays, fuses and
> transceiver on it is ordered. The water side is being plumbed while the board
> is on the way. If you want a finished appliance, this is not it yet. If you
> are building something similar, the design notes and the list of things that
> went wrong are useful today.

---

## How it is meant to work

The strategy is **batch filling**, not continuous topping up.

Water only moves through a Berkey by gravity, slowly, at roughly 220 to 440 mL
per minute depending on how many filter elements are installed. The supply is
restricted to about 2 L per minute, still several times faster. So rather than
trying to trickle water in to match the drip rate, the controller waits until
the bottom chamber is nearly empty, then fills the top chamber in one go, then
waits again.

The important constraint: everything you add at the top eventually ends up at
the bottom. So the amount you are allowed to add is limited by how much room is
left in the **bottom** chamber, not by how much room is left in the top one.
Overfill and you flood the worktop.

Every fill has three independent stops: the upper level reaching its target,
the flow meter counting the allowed volume, and a timeout. Whichever comes
first closes the valve. Filling is only allowed while Home Assistant says
someone is home.

## How it is meant to fail

If the ESP32 crashes, loses WiFi, reboots, or the firmware has a bug, the valve
closes. That behaviour comes from how the relay is wired, not from any code: the
relay's "normally closed" contact drives the valve's close wire, so an
unpowered relay is already commanding the valve shut. No software has to run
correctly for this to happen.

This is the single most important decision in the build, and it is easy to
destroy by accident. Swap the two relay output wires and everything still works
perfectly during testing, while the valve now sits open every time the system
fails.

A **float switch** in the lid of the top chamber is the last line of defence.
Its contact is the power supply for both relay coils. If water ever lifts it,
both relays drop and the valve drives shut, with the ESP32 playing no part.

---

## Water path

supply → manual shutoff → **flow sensor** → flow restrictor → motorised valve
→ outlet above the water line in the top chamber

The flow sensor sits **before** the valve on purpose. Turbine flow sensors need
water pressure behind them to read well. After the valve, the pipe would drain
into the filter between fills and the sensor would start every fill dry. Before
it, the sensor is always full, and any flow while the valve is closed means the
valve is leaking through.

---

## Hardware

| Part | What it is | Status |
|---|---|---|
| ESP32 DevKit (ESP-WROOM-32) | The small WiFi computer running everything | Working |
| 2 × QDY30A-B level probes | Hydrostatic depth sensors, 0 to 1 m range, RS-485 | **Working, verified in water** |
| RS-485 transceiver | MAX485 module on the breadboard; THVD1406 at 3.3 V on the carrier board | Breadboard working |
| YF-B1-S flow sensor, stainless | Counts litres by spinning a small turbine, about 990 pulses per litre at 2 L/min | Bench tested, calibration pending |
| Motorised ball valve, stainless body, 24 V | Opens and closes the water supply. 10 to 15 s travel | Bench tested |
| Adjustable flow restrictor | Holds the fill rate at about 2 L/min | On order |
| Float switch, stainless | Hardware overfill interlock in the top chamber's lid. One fitted | Bench tested |
| Replacement lid for the top chamber | Stainless, drilled with four holes: both probe cables, the water inlet and the float switch | On order |
| Carrier board, revision G | ESP32 socket, transceiver, 5 V buck, both relays with drivers, flyback diodes, three resettable fuses | Ordered |
| 24 V DC power supply | The only supply. Everything runs from one rail | Working |

A **hydrostatic probe** measures water depth by sensing the pressure of the
water column sitting above it. It hangs vertically near the bottom of the
chamber, and the height of its sensing diaphragm sets the zero point, which is
why mounting it rigidly matters more than the sensor's own accuracy.

Both probe cables leave through glands in the top chamber's lid. The upper
probe hangs from its lid gland. The lower probe hangs from a sealed gland in a
spare filter element hole in the top chamber's floor, and its cable crosses the
top chamber and leaves through its own lid gland. The lid is therefore tied to
both cables, so both need slack above it.

**RS-485** is a robust two wire signalling standard used in industrial
equipment, and **Modbus** is the message format spoken over it. The probes are
Modbus devices with numbered addresses, so several can share the same pair of
wires.

Anything that touches the water is stainless steel, EPDM or silicone. Brass
was rejected because the parts sit upstream of drinking water.

---

## What is in this repository

| File | What it covers |
|---|---|
| `waterfilter.yaml` | The ESPHome configuration, including the fill logic. Currently correct for the breadboard. |
| `HANDOVER.md` | Project state, decisions, the fill logic explained, and the debugging history. Read this before changing anything. |
| `BOM.md` | Every part, what is owned and ordered, and the plumbing in run order. |
| `claude/NETLIST.md` | The carrier board, net by net, plus the firmware changes for moving from breadboard to board. |
| `wiring-overview.html` | Wiring reference for the carrier board: cables, valve circuit, float interlock, terminal pin order. Open it in a browser. |
| `physical-layout.html` | Where the parts physically sit, the water path, the four-hole lid, and how to mount the probes. |
| `claude/BerkeyEnclosure.FCMacro` | Parametric FreeCAD macro for the printed enclosure. |
| `wiring-plan.md` | Historical breadboard guide. Out of date; kept for the record. |

Start with `HANDOVER.md`.

---

## Getting started

1. Install [ESPHome](https://esphome.io/guides/installing_esphome.html).
2. Create a `secrets.yaml` alongside `waterfilter.yaml` containing
   `wifi_ssid`, `wifi_password`, `wifi_bck`, `waterlevel_key` and
   `waterlevel_ota`.
3. Build the circuit from `wiring-overview.html` and `claude/NETLIST.md`.
4. Flash: `esphome run waterfilter.yaml`.
5. Two level sensors should appear in Home Assistant, reporting centimetres.
   Lower a probe into a bucket and watch the number move.
6. Leave **Enable filling** off until the probes are mounted and zeroed.

### Probe addressing

Both probes share one pair of wires, so each needs its own Modbus address.
**On this build, address 1 is the upper chamber and address 2 is the lower
chamber.** Label the physical probes, because two identical units both reading
near zero in air cannot be told apart afterwards.

Do not assume a new probe arrives on address 1. Both of ours were already on
address 2, which presents as a completely dead bus: perfectly formed requests
going out, and absolutely nothing coming back. If you see that, scan addresses
before you reach for a multimeter.

---

## Things that cost real time here

Condensed from `HANDOVER.md`, in case any of it saves you an evening.

* **A missing 4.7 kΩ pull-up resistor on the MAX485's `RO` pin.** While the
  module is transmitting, that output stops driving the line, and the voltage
  divider then pulls it to ground. The ESP32 reads a permanently low receive
  line as a UART break and reports a run of junk `0x00` bytes at the start of
  every reply. This looked exactly like bus corruption for multiple sessions.
  The pull-up is required, and it is missing from most tutorials. A 3.3 V
  transceiver avoids the whole problem.
* **A breadboard's centre gap.** It splits every column into two unconnected
  halves. If the ESP32's receive wire lands on the wrong side of the resistor
  that bridges the gap, it reads the full undivided 5 V instead of the safe
  3.3 V.
* **An ESPHome regression in Modbus frame timing** (issue
  [#17180](https://github.com/esphome/esphome/issues/17180)). The workaround,
  `rx_full_threshold: 1` and `rx_timeout: 2`, is already in the config.
  Confirmed working on ESPHome 2026.8.2.
* **`force_new_range: true` is mandatory on every sensor.** These probes reject
  requests that read several registers at once, and ESPHome merges adjacent
  registers by default.
* **Saving a new probe address goes to the probe's new address**, while the
  write that changes it goes to the old one. Get this backwards and the change
  lives in memory only and vanishes at the next power cut.
* **A slow valve breaks naive timing.** This valve takes 10 to 15 seconds to
  travel. Every timer that assumes otherwise (no-flow grace, how long to keep
  the open circuit powered, when to start checking for leaks) fires in the
  middle of the travel.

---

## Open items

* Plumb the water side as parts arrive.
* Calibrate the flow sensor against a kitchen scale, at the real fill rate.
* Re-time the valve under water pressure.
* Drill the lid (four holes), mount the probes and calibrate their zero points.
* Assemble and bring up the carrier board, then move the firmware from the
  breadboard settings to the board settings.
* Print the enclosure.

Deliberately deferred to a version 2: a floor leak sensor, and a backup
solenoid valve directly after the motorised valve, switched together with its
open winding.

---

## Safety

Water and electronics are a bad combination, and this project puts one
directly above the other. The electronics live in a dry enclosure mounted
**above** the tank, never below it. Every cable entering that enclosure gets a
drip loop, so water tracking down the outside of a cable falls off before it
reaches anything electrical.

Nothing here runs at a voltage that can hurt you. Everything here runs at a
voltage that can flood a kitchen.

## Disclaimer

Documentation of a personal project, shared because the notes may be useful. No
warranty of any kind. If you build something from this, you are responsible for
what it does, including what it does to your floor.
