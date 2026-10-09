# Water filter controller — handover summary

Continuation notes for picking this project up in a new chat.

**State, 3 Oct 2026: carrier PCB revision G is ORDERED**
(JLCPCB board plus LCSC parts in one cart, two boards' worth of parts).
JLC reopens 5 Oct; boards expected in about 14 days at the earliest.
Both probes read real centimetres on a shared RS-485 bus, verified in
water. The fill logic is written but not yet flashed. The stainless
flow sensor (arrived 1 Oct), both float switches (arrived 30 Sep) and
the manual shutoff are in hand. AliExpress plumbing parts, lid, glands
and washers are due in the coming days, so the water side gets plumbed
while the boards are on the way.

**3 Oct: the water path changed.** The flow sensor now sits first after
the shutoff, ahead of the restrictor and the valve. See "Decided on
3 Oct".

**3 Oct, evening: the lid has four holes, not three.** The lower probe's
cable comes up through the base gland, crosses the upper chamber and
must leave through the lid too. A third stainless M16 gland is needed;
two are on order.

**6 Oct:** cable glands, jumpers, nylon nuts and brass inserts received.
Still waiting on the PCB, the flow restrictor and the Berkey lid.

**9 Oct: backup solenoid ordered** (½", DC24V, NBR, stainless, NC).
Still a version 2 part, fitted once the main build works. Position
reviewed and kept directly after the ball valve. **Measure its coil
current on arrival**: over 0.45 A, F3 becomes the 1.1 A part. See
"Decided on 9 Oct" and BOM.md, "Backup solenoid".

Probe addressing, settled: **address 1 = upper chamber, address 2 =
lower chamber.** Label the physical units. See "The addressing trap".

## Next session: start here (updated 9 Oct 2026)

When the solenoid arrives: scratch test inside a port, measure coil
current at 24 V (sets F3), confirm it clicks open and shuts cleanly on
the bench, check the flow arrow. Not plumbed in until v2.

### Done 2 and 3 Oct

**2 Oct:** flow sensor bench test (5 V, internal pull-up, so SB2 cap
1 to 2, R6 10 kΩ, R7 20 kΩ, R10 empty; start K factor 990 pulses/L).
Relay NC/NO checked on a real relay. Final EasyEDA round, Gerbers,
paper print, order number location. **PCB and parts ordered.**
EasyEDA's BOM import into LCSC was trimmed of owned parts (DC2, 3-pin
terminals, SB2, TP1) and raised to two boards. `wiring-overview.html`
redrawn for revision G.

**3 Oct:** water path reordered (flow sensor first). Plumbing fittings
specified to match. `waterfilter.yaml`: K factor 990, and a false leak
abort after every normal stop fixed. `physical-layout.html` brought to
revision G. `README.md` and `Hardware` refreshed. Backup solenoid
redesigned (still deferred) and a spare enclosure hole added for it.
Evening: drawing labels fixed (B1 to B3 are Black Berkey elements, one
float switch), lid changed to four holes, `BOM.md`, `README.md` and this
file updated to match.

### To do while the boards are on the way

| # | Task | Notes |
|---|---|---|
| 1 | **Plumb the run** as the AliExpress parts arrive: supply → shutoff → union 1 → **flow sensor** (arrow with the flow) → union 2 → restrictor → valve → lid inlet | Both unions are flat-face, swivel nut on the sensor side, no PTFE on those faces. Union 2's far end depends on which end of the restrictor is its inlet. Check the restrictor is BSP. **Leave a straight G½ section or a union directly after the valve** for the future solenoid. Details in BOM.md, Plumbing |
| 2 | **Swap the stainless body onto the 8 Nm motor** | Stem engagement, screws, thread |
| 3 | **K factor calibration** on the breadboard at 2 L/min: set the restrictor by watching L/min in HA, fill 1 to 2 L into a jug on a scale, three runs, average. **Record the unrestricted rate and the handle position too** | Start value 990, already in the yaml. 2 L/min is the bottom edge of the sensor's ±3 % band; if runs scatter, use 2.5 L/min |
| 4 | **Re-time valve travel and current** under water pressure | Then revisit `safety_margin_cm`, `no_flow_grace_ms`, `valve_travel_hold` |
| 5 | Measure for the enclosure: LM2596 height in its socket, DC2 barrel axis height above the board (macro assumes 6.5 mm). Then print | Macro settings block. Includes the spare right-wall hole |
| 6 | **Order a third stainless M16 gland.** Drill the lid when it arrives: **four holes**, upper probe gland, lower probe gland, water inlet, float. Then pick the 45 or 75 mm float; only one is fitted | Step drill, EPDM washers both sides. Probe glands at least 40 mm apart centre to centre |

### When the boards arrive

| # | Task | Notes |
|---|---|---|
| 7 | DC2 footprint check: plug centre beeps to the rear lug | |
| 8 | Solder. U2 first (drag solder, flux), then SMD passives, then through-hole. SB2 cap on 1 to 2. R10 empty | |
| 9 | **First power-up in stages:** no ESP32, no buck: 24 V in, nothing warm, 24 V after F2 and on K1 COM. Buck alone: `+5V` at TP1 pin 1. Then ESP32: `+3.3V` at TP1 pin 3 | |
| 10 | **Relay test with the valve disconnected:** both click from GPIO25/26, release, stay off through a reboot, drop when the float lifts | |
| 11 | The `waterfilter.yaml` PCB switch-over: GPIO25/26 drop `inverted`, GPIO33 drop `pullup`, drop `flow_control_pin`, header comments, K factor | NETLIST section 7 |

---

## Decided on 9 Oct

| # | Decision |
|---|---|
| 1 | **Backup solenoid bought now**: ½" (DN15, matches the G½ run), DC24V (the existing rail), NBR. Viton rejected, it pays off only with hot or chlorinated water |
| 2 | **Stays directly after the ball valve.** At rest the ball valve holds mains pressure and the solenoid sees none; upstream would put the less proven diaphragm seal under pressure 24/7 for no gain |
| 3 | **Wiring unchanged from 3 Oct**: parallel with OPEN at J9, no PCB change. Confirmed against `waterfilter.yaml`: both relays stay on for the whole fill, so the solenoid stays open; on a stop K1 drops at once and the solenoid shuts in under a second while the ball valve closes dry behind it |
| 4 | **Accepted:** in series, the leak check only catches both valves passing. Yearly check of the ball valve alone with the solenoid's ferrule lifted |
| 5 | **F3 sizing waits for a measurement.** Coil wattage not stated; over 0.45 A, the spare 1.1 A F2 part goes in F3's place |

## Decided on 3 Oct

| # | Decision |
|---|---|
| 1 | **Flow sensor first after the shutoff.** After the valve, the short run drained into the Berkey between fills and the open outlet gave almost no back pressure, so the turbine started every fill dry. Upstream it never empties |
| 2 | **Accepted:** the sensor is under mains pressure around the clock, outside the motorised valve's protection. It is built for line pressure; the manual shutoff covers maintenance; it strengthens the case for the deferred leak sensor |
| 3 | **Restrictor stays between sensor and valve.** Downstream of the sensor because pressure collapses past a restrictor; upstream of the valve because it limits flow through the ball valve while it travels. The old claim that it keeps the closed valve off full line pressure was wrong: a restrictor only drops pressure while water flows |
| 4 | **Flat-face unions either side of the sensor.** Union 1 swivel × female onto the shutoff; union 2 swivel × whatever the restrictor's inlet needs |
| 5 | **Leak check skips the closing travel.** `stop_fill` cleared `fill_active` at once while the valve still took 10 to 15 s to close and water still flowed, so the 10 s leak check would latch a false abort after every normal fill. It now also requires `valve_power` off, which happens only after `valve_travel_hold` |
| 6 | **Backup solenoid, redesigned, still deferred.** Directly after the motorised valve, wired in parallel with the valve's open winding (J9 OPEN and COM, twin ferrules). It opens and closes with every fill, shuts in under a second (cutting most of the ~500 mL overshoot), and a float trip drops both relays so it closes without the ESP32. No PCB change: D1 is its flyback diode. Shares K1/K2 with the main valve, so both relays welding defeats both; accepted. Spec in BOM.md, "Deferred: backup solenoid" |
| 7 | **Spare M12 hole on the enclosure's right wall**, level with J9, with a blanking plug. The top wall has no room left |
| 8 | **Lid with four holes.** The lower probe's cable crosses the upper chamber and leaves through its own M16 gland in the lid, snugged lightly, carrying no weight; the base gland holds the probe. The lid is now captive on both probe cables, so both need coiled slack above it, and the lower probe's cable stays slightly slack inside the upper chamber so it never pulls on the base gland |
| 9 | **One float switch.** The other of the pair is a spare |

## Decided on 27 and 28 Sep (revision G)

| # | Decision |
|---|---|
| 1 | **Board revision G, a big one.** Board grows to **130 × 80 mm** |
| 2 | **Both relays on the board:** K1, K2 Songle SRD-05VDC-SL-C (C35449), driven by Q1, Q2 AO3400A MOSFETs (C20917) through 680 Ω gate resistors, 10 kΩ gate pull-downs, 1N4007 across each coil. The bought relay module becomes a breadboard spare |
| 3 | **Valve circuit on the board:** K1 COM from `RELAY_24V`, K1 NC to the close wire, K1 NO to K2 COM, K2 NO to the open wire. D1, D2 (1N4007) across the valve windings. **J9** 3-pin valve out on the top edge |
| 4 | **F3** protects the relay contacts and valve cable: PTTC SMD1812P050TF/60, C462518. **F2, the main fuse, on the board** right behind DC2: Littelfuse 1812L110/33MR, C142747, 1.1 A hold. The PSU is a brick with a fixed cable |
| 5 | **Float interlock in copper:** float NC between `+5V` and `FLOAT_OUT`, which is the coil supply for both relays |
| 6 | **Removed:** J6, SB3, the relay cable, the J8/J10 JST XH idea, the diode carrier, the relay bay, the 24 V distribution block |
| 7 | **Clean cables in and out, every junction on the board.** All field cables leave on the top edge (J2, J3, J4, J5, J9); DC2 on the right edge is the only power entry |
| 8 | **Enclosure** 154 × 112 × 40 mm, all five glands on the top wall: 2 × M16 (probes, clamp 4 to 8 mm), 3 × M12 (flow, float, valve, clamp 3 to 6.5 mm), nylon, metric thread, not PG. (3 Oct: plus one spare M12 on the right wall) |
| 9 | **1N4007 for all four diodes**, from an earlier AliExpress order |
| 10 | **No heatsink on the buck module**: 0.3 to 0.4 A load |
| 11 | **`waterfilter.yaml` stays breadboard-correct** until the PCB is in use. The PCB changes are listed in NETLIST section 7 |

### Valve test procedure (done 28 Sep, kept for the installed re-measure)

**Needs:** the 24 V PSU, the barrel-to-screw adapter, a
multimeter, a stopwatch, masking tape and a pen. The stainless body
fitted if it is already swapped, so the timing is real.

The motor draws well under 0.25 A, the risk is only a short between
bare leads. Wire everything with the PSU unplugged, plug it in last,
keep only one live lead end free at a time, and insulate the unused
motor wire. The multimeter's 10 A input usually has its own internal
fuse, so wiring it in series with plus also acts as a fuse.

**Never** connect BL and BR to the same supply pole at the same time.
The motor would be driven both ways at once.

1. PSU unplugged. Meter in series with the plus lead.
2. SR yellow to PSU minus.
3. Plug in the PSU. Touch **BL** blue to plus and hold it. The motor
   runs to **open** and stops by itself. Time it, note the current.
4. Release BL. Touch **BR** brown to plus and hold it. Runs to
   **closed**, stops by itself. Time it.
5. Record open time, close time, running current, and whether it
   stopped by itself at both ends.

The times feed `safety_margin_cm`, `no_flow_grace_ms` and
`valve_travel_hold`. The running current confirms F3's 500 mA hold.

---

## The project in one paragraph

Automating a **Big Berkey gravity water filter** (two stacked stainless
chambers, ~7.6 L upper / ~8.5 L lower, 21.6 cm diameter) with an ESP32
running ESPHome. Two hydrostatic level probes report depth over
RS-485/Modbus; a flow sensor counts litres; a motorised ball valve opens
the supply. The strategy is **batch fill**: wait until the lower chamber
is nearly empty, fill the upper chamber in one go, let it drip through,
repeat. Filling is permitted only when a Home Assistant boolean says
someone is home.

---

## Changed in the enclosure session, 27 Sep (revision G)

**Partly superseded on 28 Sep:** the relay module, relay bay, J8 and the
diode carrier below were replaced by relays on the board. Kept as a
record of how the design got there.

| Change | Detail |
|---|---|
| **Printed enclosure** | `claude/BerkeyEnclosure.FCMacro`, parametric FreeCAD macro, base and lid with a locating lip, M3 heat-set inserts, four mounting ears. Replaces the bought IP65 junction box |
| **Probe glands M16, not M12** | The probe lead measures 7 mm, above most M12 clamping ranges |
| **DC2 plug tube** | A printed tube guides the plug in, so power disconnects from outside. The box is splash resistant, not sealed, accepted |

---

## Changed in the schematic and layout session, 25–26 Sep

| Change | Detail |
|---|---|
| **Schematic redrawn with net labels** | The first version used long crossing wires and had three shorts: D25 and D26 joined, D33 directly on the 5 V float line, R8 on the flow supply. Labels make that class of error impossible. Verified against the netlist export |
| **Probe 24 V through the board** | J2.1 and J3.1 carry `PROBE_24V`, so each probe cable lands whole in one terminal |
| **F1 added** | PTC resettable fuse, 1206, 0.1 A hold, between `+24V` and `PROBE_24V`. A pinched probe cable would otherwise short the 8 A supply through a board trace |
| **Test pads became a header** | TP1, 1×6: +5V, GND, +3.3V, A, B, GND |
| **Pin to pad mapping fixed** | GPIO names had been typed into the pin number field of the ESP32 header symbols, leaving every socket pad unconnected. Schematic DRC caught it. U3's pin numbers now match its footprint's pad names (INGND, INVCC, OUTVCC, OUTGND) |
| **ESP32 was mirrored left to right, caught 27 Sep** | The DevKit plugs in components up, so the board sees its top view. With the antenna at the bottom, EN's row belongs on the right. U1 and U4 swapped, rerouted |
| **R10 as a DNP footprint** | Below J4, marked on the silkscreen. Stays empty (2 Oct) |
| **U3 mounting holes kept** | For M3 standoffs under the buck module. Causes two accepted DRC errors inside its footprint |
| **Layout** | ESP32 pin 1 at the bottom with an antenna keepout; DC2 flush with the right edge; non-plated M3 holes at the corners; power routed at 1.0 mm, 3.3 V at 0.5 mm, signals at 0.3 mm; UART on the bottom layer. Full detail in NETLIST.md section 6 |

---

## Earlier PCB session

| Change | Detail |
|---|---|
| **THVD1406 pin numbers confirmed** | TI datasheet SLLSF87A. **The part does have enable pins**: RE on pin 2, SHDN on pin 3, both tied to `+3.3V`. Tying RE low instead would echo every transmitted byte back into GPIO16 |
| **J7 dropped** | It broke out GPIO32 and GPIO35 for a second RS-485 bus, which was a workaround from the period when the probes would not talk. That was fixed on the existing bus |
| **Leak sensor considered and deferred** | Version 2 |
| **Design stays in EasyEDA** | KiCad rejected. The part search is the LCSC catalog, ordering is one button |
| **24 V enters on a barrel jack** | DC2, a DC-005-20A, 5.5 × 2.1 mm. Costs an IP65 face, accepted |
| **ESP32 socket is two discrete 1×15 headers** | U1 and U4, 25.4 mm apart |
| **Flyback diodes clarified** | D1 and D2 go **across** the windings, not in series |

---

## Hardware, confirmed

| Part | Model | Notes |
|---|---|---|
| Controller | ESP32 DevKit, ESP-WROOM-32, 30-pin | Logger on UART0, UART2 free for Modbus. Header rows 25.4 mm apart, EN at the antenna end |
| Level probes ×2 | QDY30A-B, 0–1 m, RS-485 | **DC 24 V** from the label. Register 2 = 17 (cm), register 3 = 1, on both. **Address 1 = upper, 2 = lower** |
| Probe wire colours | **Red V+, Green V−, Blue A, Yellow B** | From the label on the physical unit. Land in J2/J3 pins 1 to 4 in that order |
| Flow sensor | **YF-B1-S, stainless, G½, arrived 1 Oct** | **5 V, internal pull-up, 2 mA.** Red +, black GND, yellow signal. About 990 pulses/L at 2 L/min (start value, calibrate), ±3 % from 2 to 6 L/min. Flat-face seal. **First after the shutoff, permanently pressurised.** The brass YF-B1 is a bench part |
| Valve motor | ACARPS, DC 12–24 V, 6 W | **15 s travel** (label), 8 Nm, IP54. **BL blue opens, BR brown closes, SR yellow common.** Stops by itself at both ends; **0.03 A max moving, 0.00 A stopped, 10 to 11 s travel on the bench** (28 Sep). Mount it dry |
| Valve motor, spare | 6 Nm, 12 V | Drawer spare |
| Valve body | Stainless, already owned | Interchangeable with the brass one |
| Manual shutoff | Ball valve, male/male | **Received** |
| RS-485, breadboard | MAX485 module (HW-97) | Working. `RO` needs a 4.7 kΩ pull-up **and** a divider |
| RS-485, PCB | THVD1406DR, SOIC-8, LCSC C5215918 | Run it at 3.3 V. RE and SHDN both to `+3.3V` |
| RS-485, spares | MAX13487E red boards ×5 | **+5 V part.** Not used |
| Relays | Songle SRD-05VDC-SL-C | On the board as K1, K2 (rev G), MOSFET driven. The 2-channel module stays on the breadboard |
| Float switch | Stainless, M10 × 1.5, NC | **Arrived 30 Sep**, 45 mm and 75 mm stems, both pass. **One fitted**, the other a spare. Choice waits for the lid |
| Power supply | 24 V DC 8 A brick | Fixed cable, plugs straight into DC2, the only power entry. **Plug fit and centre-positive polarity confirmed 28 Sep.** F2 on the board is the main fuse |

---

## The float interlock

The most important thing in the design, and the least obvious.

**The float switch's NC contact is the coil supply for both relays.**
Not in the valve's wiring. On the rev G board: `+5V` goes out on J5.1,
through the float, back on J5.2 as `FLOAT_OUT`, and `FLOAT_OUT` feeds
both relay coils.

When water lifts the float, the contact opens, both coils lose power
whatever the MOSFETs and the ESP32 are doing, and K1 falls back to
`COM`–`NC`, which is the contact feeding the valve's **close** wire.
The valve is driven shut. No GPIO, no firmware, no ESP32 involvement.
A future backup solenoid on J9 OPEN loses power at the same moment.

Two wrong ways to wire it, both of which look reasonable:

- **In the valve's open wire.** This only stops *driving* the valve. The
  actuator has no spring return, so it freezes where it is, and a valve
  frozen open keeps filling.
- **In the `COM` feed.** Kills both contacts and freezes the valve the
  same way.

GPIO33 senses the trip through R8/R9, so the firmware can refuse to
restart. That is sensing only. The interlock works with the ESP32
unplugged, and that is the test that matters.

---

## Overshoot

Fill volume is bounded by the **lower** chamber's headroom, so overshoot
comes straight out of the safety margin. 1 cm of depth is 366 mL in
either chamber. Valve travel is 15 s by the label, 10 to 11 s on the
bench.

| Fill rate | Overshoot | In cm |
|---|---|---|
| Unrestricted, **~7.5 L/min estimated** | ~1.9 L | ~5.1 |
| 5 L/min | ~1.25 L | ~3.4 |
| 2.5 L/min, fallback if 2 calibrates poorly | ~625 mL | ~1.7 |
| **2 L/min, the target** | **~500 mL** | **~1.4** |

> **The unrestricted figure is an estimate, not a measurement.** The true
> number arrives for free during the K-factor calibration. Nothing in the
> design turns on it: the restrictor is mandatory and 2 L/min is the
> target.

`safety_margin_cm` is set to **3.5**. **Calibrate the K-factor at the
rate you will actually run**, not at full bore. A backup solenoid, if
fitted, would cut the overshoot to a few millilitres; leave the margin
as it is anyway.

### Three timing bugs the 15 s figure created

All are fixed in the current `waterfilter.yaml`:

- **`no_flow_grace_ms` was 12 s.** The valve would not have finished
  opening before the firmware aborted for no flow. Now **25 s**.
- **`stop_fill` dropped `valve_power` after 8 s.** Channel 2 would have
  broken the open circuit mid-travel. Now **20 s**, which must always
  exceed the measured travel time.
- **The leak check fired during the closing travel** (fixed 3 Oct).
  `fill_active` goes false the moment a stop is decided, but water keeps
  flowing for the 10 to 15 s the valve takes to close, so every normal
  fill would have latched "flow detected with valve closed". The check
  now also waits for `valve_power` to drop.

---

## The fill logic

Written, not yet flashed. The whole thing reduces to one invariant:

```
allowed_cm = (lower_full - lower_now) - upper_now - safety_margin
```

**Three independent stops.** Upper level reaches target, flow meter counts
the volume, elapsed time exceeds the limit. Whichever fires first closes
the valve. The float interlock is a fourth, and the only one outside the
firmware.

**Aborts latch.** Permission withdrawn, API lost, probe stale, no flow
with the valve open, flow with the valve closed, float tripped,
implausible level. An abort refuses to restart until cleared by hand.

**Flow with the valve closed means a passing valve.** With the sensor
upstream of the valve and never drained, any flow once the valve has
finished closing is real.

**Permission comes from Home Assistant.** An `input_boolean`, read over
the API. Unknown, unavailable, HA restarting, API disconnected and
just-booted all read as "do not fill". A falling edge stops an
in-progress fill. No `restore_mode` anywhere downstream of it.

**A reboot mid-fill is an abort, never a resume.**

**`enable_filling` is off by default** and must stay off until the
probes are mounted and zeroed.

### GPIO allocation

| Function | Pin | Board |
|---|---|---|
| Modbus TX / RX | GPIO17 / GPIO16 | U4.9 TX2 / U4.10 RX2 |
| MAX485 `RE`+`DE` | GPIO4 | Breadboard only, gone on the PCB |
| Relay K1 (close/feed) | GPIO25 | U1.8, via R11 to Q1, R4 pull-down |
| Relay K2 (open) | GPIO26 | U1.9, via R12 to Q2, R5 pull-down |
| Flow pulses | GPIO27 | U1.10, via R6/R7 |
| Float sense | GPIO33 | U1.7, via R8/R9 |

GPIO32 and GPIO35 are unused on this board revision.

---

## The Modbus debugging saga — what was actually wrong

Recorded so none of it gets re-tried. Four separate faults, each masking
the next.

### Fault 1: an ESPHome regression in Modbus inter-frame timing

ESPHome 2026.6.x cut the inter-frame timeout from ~50 ms to ~5 ms.
Combined with the ESP-IDF UART delivering bytes in batches, short replies
arrive in pieces and the partial frame is discarded. Issue #17180. The
workaround, `rx_full_threshold: 1` and `rx_timeout: 2`, is in the config.
**Confirmed working on 2026.8.2.**

### Fault 2: a self-inflicted UART break

While `RE` is high during transmit, the MAX485's `RO` output goes
high-impedance. With only the divider on that line, the 20 kΩ leg pulls
the floating line to ground, which the ESP32 reports as a run of `0x00`
bytes. **Fix: a 4.7 kΩ pull-up from `RO` to 5 V**, before the divider.
At 3.3 V there is no divider, so neither is needed on the PCB.

### Fault 3: breadboard build errors

`DI` and `RO` sharing a row. The pull-up wired in series instead of to
the rail. GPIO16 tapped on the wrong side of the centre gap. A missing
ground. The correctly built divider rests at **3.33 V**, not 2.9 V.

### Fault 4: both probes were already on address 2

Symptom: byte-perfect outgoing frames and **zero bytes ever received**.
Every project document said the probes ship on address 1. They did not.
**When outgoing frames are provably correct and nothing comes back, scan
addresses before measuring anything.**

### The addressing trap

The two halves of an address change go to **different addresses**:

1. `0x06` to register `0x0000` with the new address, sent to the probe's
   **current** address.
2. `0x06` to register `0x000F` with value 0, sent to the probe's **new**
   address. This commits it.

Send the save to the old address and the change lives in RAM only. Do it
with **one probe on the bus**, power cycle to verify, then label the probe
physically and immediately.

### Function code 0x03 is fine

`register_type: holding` is confirmed correct.

---

## Register map

| Register | Meaning | Confirmed |
|---|---|---|
| `0x0002` | Unit code — 16 = m, **17 = cm**, 18 = mm | **17**, both probes |
| `0x0003` | Decimal places, 0–4 | **1**, both probes |
| `0x0004` | Measurement, signed 16-bit | live, divide by 10 |
| `0x0000` | Slave address, writable FC06 | |
| `0x000F` | Save to user area, write 0 FC06 | |

UART: **9600 baud, 8N1.** `force_new_range: true` is mandatory on every
sensor.

---

## Design decisions already settled

- **Fail-closed valve wiring.** Relay `NC` feeds the close wire.
- **A second relay** in series with the open wire, off at rest.
- **Relays, drivers and flyback diodes on the carrier board** (revision G). One board, clean cables in and out.
- **Flow restriction is a separate adjustable part**, not partial valve
  opening.
- **Flow sensor first after the shutoff** (3 Oct), so it never drains.
- **Fill volume is bounded by the lower chamber's headroom.**
- **Stainless over brass** for anything wetted.
- **3.3 V transceiver** instead of a 5 V part with level shifting.
- **Design and order in EasyEDA Pro.**
- **Flashing is over OTA.**
- **Probes fed through the board, fused** (revision F).
- **Valve circuit fused by F3, main fuse F2 on the board** (revision G). DC2 is the only power
  entry.
- **Printed enclosure**, all field glands on the top wall, one spare on
  the right wall (revision G, spare 3 Oct).
- **One float switch**, in the lid (3 Oct).
- **Rejected as over-engineering**: modulating valve with position
  feedback.
- **Dropped, not deferred**: a second independent RS-485 bus.
- **Deferred to a version 2**: a floor leak sensor, and fitting the
  backup solenoid (**bought 9 Oct**) directly after the main valve,
  wired in parallel with its open winding at J9 (no PCB change; spare
  gland hole ready).

---

## Physical, settled

See `physical-layout.html` for the full picture. In summary:

- **Water path**: supply → shutoff → union → **flow sensor** → union →
  restrictor → valve → (backup solenoid, bought, fitted in v2) → lid inlet. The sensor stays
  full and pressurised; the restrictor stays after it.
- **Lower probe**: cable gland only, M16 × 1.5, 304 stainless, EPDM. **This
  port is wet** — gland body and flange O-ring on the **upper** side,
  locknut below. Its cable then crosses the upper chamber, slightly slack,
  and leaves through its own lid gland.
- **Upper chamber lid**: replacement 22 cm stainless pot cover, **four
  holes** — M16 upper probe gland, M16 lower probe cable gland, a bare
  water inlet hole, 10.5 mm for the float. Probe glands at least 40 mm
  apart. Step drill. EPDM washers either side of every fitting.
- **Water inlet**: deliberately unsealed, end above the maximum water
  line. The gap vents displaced air.
- **Upper probe**: hangs from the lid gland. Mark the cable at the gland
  and leave coiled slack, **both required**.
- **Lid is captive on both probe cables.** Coiled slack above the lid on
  both, so it lifts off without dragging either probe.
- **Float switch**: one, on the lid, below the maximum fill line and
  above the normal fill target. Cable extension joint **above** the lid.
- **Probes hang vertically, tip down**, 10 to 15 mm above the floor.
- **Electronics box** above the tank, never below, drip loop in every
  cable. Printed, from the FreeCAD macro. Only the carrier board inside;
  every junction on the board. Spare M12 hole on the right wall, plugged.
- Chamber diameter 21.6 cm gives **366 mL per cm** of depth.

---

## Open items

### Once parts arrive

| Item | Why it matters |
|---|---|
| **Order a third stainless M16 gland** | Four-hole lid needs two, plus the base port |
| Swap the stainless body onto the 8 Nm motor | Stem, screws, thread standard |
| **Re-time the valve close** under pressure | Sets `safety_margin_cm`, `no_flow_grace_ms` and the `stop_fill` hold |
| Choose 45 mm or 75 mm float stem | Gap between lid underside and maximum water line |
| Probe zero calibration | After final mounting |
| Set the restrictor to 2 L/min | Record the handle position and the unrestricted rate |
| Confirm the restrictor is BSP | Not NPT |
| **Restrictor inlet end** | Decides the far end of union 2 |
| Flow K factor | Against a kitchen scale, **at the real fill rate**. Start 990 |

### Before printing the enclosure

| Item | Why it matters |
|---|---|
| LM2596 height in its socket | Sets the box height |
| DC2 hole centre above the board | Sets the plug tube height |
| Gland clamping ranges | Probe 7 mm, flow, float and valve cables |
| M12 blanking plug | For the spare right-wall hole |

### Tests that must pass before a first real fill

| Test | Expect |
|---|---|
| Reboot the ESP32 with the valve wired | No movement at all |
| Pull the HA boolean mid-fill | Valve closes |
| **Pull ESP32 power mid-fill** | Valve closes. No software runs during this one |
| Lift the float mid-fill | Valve drives **closed**, not merely stops |
| **A normal fill runs to its stop** | No "flow detected with valve closed" afterwards. Confirms the leak check waits out the travel |
| **Lift the lid off** | Neither probe reading moves. Confirms the slack above the lid is enough |

---

## Companion documents

- `claude/NETLIST.md` — **revision G, ordered.** Pinouts, netlist,
  placement, routing, order settings, firmware changes for the PCB,
  revision history
- `BOM.md` — **current, 3 Oct.** Board parts with LCSC numbers,
  what is owned, what is on order from AliExpress, plumbing fittings in
  run order, the deferred solenoid spec, three stainless M16 tank glands
- `wiring-overview.html` — **current for revision G (redrawn 2 Oct).**
  Cable routing, valve circuit and float interlock, terminal pin order
- `physical-layout.html` — **current, 3 Oct evening.** Water path, probe
  mounting, four-hole lid, restrictor, and the rev G electronics box
- `claude/BerkeyEnclosure.FCMacro` — parametric FreeCAD enclosure for the
  130 × 80 board, base and lid, **spare right-wall gland added 3 Oct**.
  Dimensions in one settings block
- `claude/pcb-rev-F.html` — screenshots of the rev F layout. **Predates G**
- `waterfilter.yaml` — ESPHome config for the **breadboard**, fill logic
  written, `enable_filling` off, K factor 990. Changes at the PCB
  switch-over in NETLIST section 7
- `README.md` — public-facing overview, refreshed 3 Oct evening
- `wiring-plan.md` — **stale, historical**: breadboard guide, two
  supplies, 5 s valve. Do not build from it
- `Functions`, `Hardware` — the original one-paragraph brief
- Protocol PDFs in the project files — the authoritative register map
