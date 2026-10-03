# Bill of materials — Berkey fill controller

Production build. Single 24 V rail. Everything electrical is on the
carrier board: ESP32, THVD1406 transceiver, buck module, **both relays
with their drivers, and the flyback diodes (G)**. The board sits alone
in a printed enclosure; cables only enter and leave.

Board parts sourced from LCSC, everything else via nl.aliexpress.com
unless noted.

Updated 2 Oct 2026: **PCB and LCSC parts ordered.** Revision G
(relays on the board). Updated 3 Oct 2026: **water path reordered, flow
sensor first after the shutoff**; plumbing fittings follow from that.
Spare enclosure gland for a future backup solenoid. **Upper chamber lid
now has four holes, so three stainless M16 tank glands are needed, not
two.** Changes recorded at the bottom.

---

## Ordered 2 Oct 2026, JLCPCB + LCSC in one cart

JLC is closed 1 to 4 Oct, production starts 5 Oct. Expect about 14
days at the earliest.

**In hand, so not ordered:** buck modules, 1N4007, ESP32 DevKit,
DC-005 barrel jacks, 3-pin 5.08 mm terminals (J4, J9), male header
strip (SB2, TP1), jumper caps, M3 × 6 screws (used instead of M3 × 8;
4.4 mm thread engagement in the inserts, enough).

### LCSC

Quantities cover **two populated boards** (decided 28 Sep). LCSC raises
some 0805 passives to its minimum (50 or 100).

| Part | LCSC | Order | For |
|---|---|---|---|
| THVD1406DR | C5215918 | 2 (3 optional) | U2. Third as a hand-soldering spare |
| Relay SRD-05VDC-SL-C | C35449 | 4 | K1, K2 |
| AO3400A | C20917 | 5 | Q1, Q2 |
| PTC 1206L010/60NRL, 100 mA | C18198325 | 2 | F1 |
| PTC 1812L110/33MR, 1.1 A | C142747 | 2 | F2. The spare can also go in F3's place if a future solenoid draws more than about 8 W |
| PTC SMD1812P050TF/60, 500 mA | C462518 | 2 | F3 |
| Female header 1×15, 2.54 mm | C7499333 | 5 | U1, U4, and U3's 4-pin socket cut from a spare strip |
| Terminal WJ500V-5.08-04P | C42377749 | 4 | J2, J3 |
| Terminal WJ500V-5.08-02P | C8465 | 2 | J5 |
| 680 Ω 0805 | C2907340 | 10 | R1, R2, R11, R12 |
| 120 Ω 0805 | C2907224 | 5 | R3 |
| 10 kΩ 0805 | C101405 | 10 | R4, R5, R6, R8 |
| 20 kΩ 0805 | C2907240 | 5 | R7, R9 |
| 100 nF 0805 | C495959 | 10 | C1, C2, C3 |
| 470 µF 16 V, 8 mm | C106651 | 2 | C4 |

**Not needed after the flow sensor bench test (2 Oct):** 0 Ω C17477
(R6 link, only for a 3.3 V sensor) and 4.7 kΩ C17673 (R10, only for an
open-collector output). The sensor runs at 5 V with its own pull-up.

### JLCPCB

Carrier PCB rev G, 130 × 80 mm, 2 layer, 1.6 mm, lead-free HASL,
1 oz, order number at a specified location, flying probe test, qty 5.

### AliExpress (nl.aliexpress.com), ordered 28 Sep

| Part | Order | For |
|---|---|---|
| Cable gland M16 × 1.5, nylon, clamp 4 to 8 mm, metric, with locknut | 5-pack | Probes, 2 used. For strain relief; IP rating irrelevant, the box is away from water (28 Sep) |
| Cable gland M12 × 1.5, nylon, clamp 3 to 6.5 mm, metric, with locknut | 5-pack | Flow, float, valve, 3 used, plus the spare solenoid hole later. Cheapest that fits |
| M3 heat-set inserts, M3 × L5 × OD4.2 | a pack | 8 used: lid bosses and board standoffs (4.0 mm holes) |
| M3 nylon or brass standoffs, height = U3 socket height, + screws | 2 | Buck module support. Measure the socket height first |

---

## Already owned

| Part | Qty | Note |
|---|---|---|
| ESP32 DevKit, ESP-WROOM-32, 30-pin | 1 | **Row spacing measured: 25.4 mm. EN (pin 1) is at the antenna end** |
| QDY30A-B level probe, 0–1 m, RS-485 | 2 | 24 V. Addresses set: 1 = upper, 2 = lower |
| Valve motor | 1 | ACARPS, DC 12–24 V, 6 W, **15 s travel**, 8 Nm, IP54. Wires, **confirmed 28 Sep: BL blue opens, BR brown closes, SR yellow common, as labelled.** Stops by itself at both ends: **0.03 A max moving, 0.00 A stopped, 10 to 11 s travel** (28 Sep, bench, no water pressure). **This is the primary** |
| Valve motor, spare | 1 | 6 Nm, **12 V**, travel time unspecified. 12 V on a 24 V rail is disqualifying. Drawer spare only |
| Valve body, stainless | 1 | **Interchangeable with the brass body** |
| Valve body, brass | 1 | Superseded by the stainless one |
| Manual ball valve, upstream shutoff | 1 | **Received. Male/male.** Joins the flow sensor through a flat-face union (see Plumbing) |
| 24 V DC 8 A PSU | 1 | **Brick with a fixed cable and barrel plug.** Plugs straight into DC2. **Plug fit and centre-positive polarity confirmed 28 Sep** (already powered this build). Its own short-circuit protection covers its cable; F2 on the board covers everything after DC2 |
| 2-channel relay module | 1 | Songle SRD-05VDC-SL-C. Stays on the **breadboard** until the PCB is in use, then becomes a spare. The PCB carries its own relays (G) |
| MAX485 module (HW-97) | 1 | Keep. The working breadboard transceiver and the fallback |
| RS-485 module, MAX13487E, red | 5 | 5 V part. Not used in this design. Spares |
| 1N4007 | **4 needed** | **Confirmed in stock 28 Sep, plenty.** D1, D2 valve flyback and D3, D4 relay coil flyback, all on the board (G) |
| LM2596 buck module | 1 | **Confirmed in stock 28 Sep, plenty.** Same module type as used on an earlier board with this footprint |
| DC-005 barrel jacks, 3-pin 5.08 mm terminals, male header strip, jumper caps | — | **Confirmed in hand 2 Oct.** DC2, J4, J9, SB2, TP1 |
| Flow sensor YF-B1-S, stainless | 1 | **Arrived 1 Oct, bench tested 2 Oct.** 5 V, internal pull-up, about 990 pulses/L at 2 L/min (start value). ±3 % band 2 to 6 L/min. Rated for line pressure; mounted first after the shutoff, so permanently pressurised |
| Float switch, stainless, M10 × 1.5, NC | 2 | **Arrived 30 Sep.** One 45 mm stem, one 75 mm. Both pass the meter check. **Only one is fitted**; choice waits for the lid |
| Brass YF-B1 flow sensor | 1 | Bench part only |
| Heatshrink and hook-up wire | — | Enough for the float switch cable extension |

> **The lead question is closed by a screwdriver.** The valve body is
> interchangeable between the two motors on hand, and a stainless body is
> already owned. Fit the 8 Nm 24 V motor to the stainless body. Check stem
> engagement and mounting screws before assuming it drops on, and measure
> the stainless body's thread against the existing fittings — if the brass
> one is BSP and the stainless is NPT, the problem has moved to the
> plumbing.

## On order (AliExpress)

| Part | Qty | Note |
|---|---|---|
| Adjustable flow restrictor, male/female | 1 | **Adjustable, confirmed.** Sits between the flow sensor and the valve. Check which end is the inlet on arrival: it decides the far end of the sensor's outlet union |
| Cable gland, M16 × 1.5, 304 stainless, EPDM | **2 ordered, 3 needed** | One for the lower chamber port in the upper chamber's base, two in the lid: upper probe, and the lower probe's cable on its way out. **Order one more** (3 Oct) |
| Stainless lid, 22 cm raised pot cover | 2 | Comes in pairs. Once drilled, it is drilled |
| EPDM washer, M16 × 30 × 3 mm | pack of 50 | Two per gland, **six total** |
| EPDM washer, M10 × 16 × 1.5 mm | pack of 50 | Two for the float switch. **Not the 3 mm** — the float has its own O-ring and thread engagement is short |
| Step drill | 1 | **Four holes** in thin stainless. A twist bit grabs and tears |

> **Flow sensor fittings:** the YF-B1-S seals with a flat washer on its
> face, so the parts either side need flat-face unions or nuts with
> gaskets, not a taper thread into PTFE tape. Follow the arrow.

---

## Carrier board, soldered parts

All LCSC. Reference designators match NETLIST.md revision G.

### Semiconductor

| Ref | Part | LCSC | Qty | Note |
|---|---|---|---|---|
| U2 | THVD1406DR, SOIC-8 | C5215918 | 2 | 3.0–5.5 V, auto-direction, 500 kbps. Buy a spare |
| **K1, K2** | Relay, Songle SRD-05VDC-SL-C | C35449 | 2 | **New in G.** 5 V coil, 10 A 30 V DC contacts. Same relay as on the module. Buy a spare |
| **Q1, Q2** | N-MOSFET, AO3400A, SOT-23 | C20917 | 4 | **New in G.** Coil drivers, fully on from a 3.3 V gate. Cheap, buy spares |
| D1 to D4 | 1N4007, DO-41 | owned | 4 | See Already owned |

Running U2 from 3.3 V is the entire point: it removes the level-shifting
divider and the `RO` pull-up. Do not power it from 5 V out of habit.

**Hand solder this rather than paying for assembly.** One SOIC-8 on an
otherwise through-hole board does not justify the JLCPCB assembly setup
fee, the stencil, or the extra week. Fine tip, flux, drag solder, about a
minute. The spare is the insurance.

**RE and SHDN are real pins and both go to `+3.3V`.** See NETLIST.md
section 1.

### Protection

| Ref | Part | LCSC | Qty | Note |
|---|---|---|---|---|
| **F2** | PTC resettable fuse, Littelfuse 1812L110/33MR | C142747 | 2 | **Main fuse, on the board since G**, right behind DC2. 1.1 A hold, 1.95 A trip, 33 V, 20 A max. Whole board draws about 0.5 A at most. Buy a spare |
| F1 | PTC resettable fuse, LUTE 1206L010/60NRL | C18198325 | 2 | Datasheet confirmed: 100 mA hold, 250 mA trip, 60 V, 40 A interrupt, 1.6 Ω. Between `+24V` and `PROBE_24V`. Buy a spare |
| **F3** | PTC resettable fuse, PTTC SMD1812P050TF/60 | C462518 | 2 | **New in revision G.** Between `+24V` and `RELAY_24V`, protects the relay contacts and valve cable. 500 mA hold, 1 A trip, 60 V, 150 mΩ, 1812. Buy a spare |

### Connectors

| Ref | Part | LCSC | Qty | For |
|---|---|---|---|---|
| U1, U4 | 1×15 female header, 2.54 mm | C7499333 | 2 | ESP32 socket, one per row, 25.4 mm apart |
| U3 | 4-pin female header, 2.54 mm | C7499333 offcut | 1 | LM2596 socket |
| DC2 | DC-005-20A barrel jack | C130239 (owned) | 1 | 24 V in. **5.5 × 2.1 mm, not 2.5.** Flush with the board edge |
| J2, J3 | WJ500V-5.08-04P | C42377749 | 2 | Probes. Pin 1 is the fused probe 24 V |
| J4, **J9** | WJ500V-5.08-03P | C72334 (owned) | **2** | J4 flow sensor. **J9, new in G: valve out, silkscreen COM CLOSE OPEN left to right** |
| J5 | WJ500V-5.08-02P | C8465 | 1 | Float switch. **Moves to the top edge (G)** |
| SB2 | 1×3 male header, 2.54 mm, PZ254V-11-03P | C2937625 (owned strip) | 1 | Flow sensor supply select. **Cap on 1 to 2, 5 V** |
| TP1 | 1×6 male header, 2.54 mm, PZ254V-11-06P | C492405 (owned strip) | 1 | Test header: +5V, GND, +3.3V, A, B, GND |

**Removed in G:** J6 (relay module header), SB3, and the J8/J10 JST XH
headers proposed on 27 Sep. Nothing connects to an external module.

**Terminal pitch closed:** all WJ500V footprints measure 5.08 mm in the
editor, despite the P5.00 in some footprint names.

### Passives

All 0805.

| Ref | Value | LCSC | Qty | Purpose | Fit? |
|---|---|---|---|---|---|
| R1, R2 | 680 Ω | C2907340 | 2 | RS-485 bias | always |
| R3 | 120 Ω | C2907224 | 1 | Bus termination | always |
| R4, R5 | 10 kΩ | C101405 | 2 | **Gate pull-downs (G)**, relays off during boot | always |
| R6 | **10 kΩ** | C101405 | 1 | Flow divider upper leg | **fit (5 V sensor, 2 Oct)** |
| R7 | **20 kΩ** | C2907240 | 1 | Flow divider lower leg | **fit** |
| R8 | 10 kΩ | C101405 | 1 | Float sense divider upper leg | for GPIO33 sensing |
| R9 | 20 kΩ | C2907240 | 1 | Float sense divider lower leg | same |
| R10 | — | — | 0 | Flow sensor pull-up | **Not fitted.** Sensor has an internal pull-up (2 Oct). DNP footprint stays empty |
| **R11, R12** | 680 Ω | C2907340 | 2 | **New in G.** Gate series resistors. Same part as R1/R2 | always |
| C1, C2, C3 | 100 nF | C495959 | 3 | Decoupling. C1 within 5 mm of U2 pin 8 | always |
| C4 | 470 µF | C106651 | 1 | 5 V bulk. **16 V is plenty** | always |

### Also needed at assembly

| Item | Qty | Note |
|---|---|---|
| 2.54 mm jumper cap | 1 | SB2. Owned |
| M3 standoff, height = U3 socket height, + screws | 2 | Support the buck module through its two mounting holes. On order |

---

## Interconnect

| Part | Qty | Note |
|---|---|---|
| Ferrule kit 0.25–1.5 mm² + crimper | 1 | Every stranded end into a screw terminal |
| Shielded twisted pair, 2 × 2 × 0.34 | 5 m | A/B pair plus probe ground |
| Valve cable, 3-core | as needed | BL blue to J9 OPEN, BR brown to J9 CLOSE, SR yellow to J9 COM, from J9 on the board out through the top wall to the valve. Measure its OD for the gland |

**Removed in G:** the 5-core Dupont relay cable, the JST XH pigtails and
the bridge wire. Nothing inside the box is wired except the board.

**Probe cables land whole in J2 and J3**, all four wires: red 24 V
in pin 1, green GND in pin 2, blue A in pin 3, yellow B in pin 4.

---

## Power

| Ref | Part | Qty | Note |
|---|---|---|---|
| U3 | LM2596 buck module, 24 V → 5 V | 1 | Owned. 40 V input rating. **Not MP1584**, 28 V max is too close to 24 V |
| — | SS34 Schottky, optional | 1 | In series with `+24V` if reverse polarity protection is wanted. Costs 0.4 V, irrelevant at 24 V |

3.3 V comes from the ESP32's own regulator. Nothing else on the board
draws from it beyond U2 and a few resistors, well inside what the AMS1117
supplies.

A barrel jack hides polarity in a way a screw terminal did not, and
centre-negative supplies exist. The LM2596 will not survive 24 V
backwards. **This brick is confirmed centre-positive** (28 Sep).

**No heatsink on U3 (28 Sep).** The 5 V load is about 0.3 to 0.4 A
(ESP32 plus two relay coils at about 72 mA each, plus the flow sensor).
At 24 V in, the module loses roughly 0.5 to 0.7 W, spread over the IC,
diode and inductor. An LM2596 module handles well over 1 A without a
heatsink.

**Flashing is over OTA.** First flash and any recovery happen with the
module out of its socket, so USB 5 V and `VIN` never meet.

---

## Valve circuit, on the board (G)

K1 and K2 switch the valve; Q1 and Q2 drive their coils from GPIO25 and
GPIO26. At rest K1's NC feeds the close wire. Opening needs both relays
on. The float's NC contact is the coil supply for both relays, so a
tripped float drops them and the valve drives closed with no firmware
involved. Full description in NETLIST.md section 4, picture in
`wiring-overview.html`.

**Flyback diodes.** D1 bridges OPEN and COM, D2 bridges CLOSE and COM,
across the valve windings. **COM is on the negative rail (bench test,
28 Sep)**, so the anodes go to COM (GND), cathode bands to OPEN and
CLOSE. That direction is fixed in copper. D3 and D4 sit across the
relay coils, cathode to `FLOAT_OUT`. All four are 1N4007.

---

## Enclosure

Printed, from `claude/BerkeyEnclosure.FCMacro` (FreeCAD, parametric).
**154 × 112 × 40 mm** base plus a 3 mm lid.

| Part | Qty | Note |
|---|---|---|
| Printed base and lid | 1 | PETG or ASA rather than PLA if it sits anywhere warm |
| M3 heat-set insert | 8 | 4 for the lid bosses, 4 for the board standoffs. 4.0 mm hole in the print |
| M3 × 6 screw | 8 | Lid and board. Owned |
| Cable gland, **M16 × 1.5**, nylon PA66, IP68, with locknut | 2 (5 in hand) | J2 and J3 probes, top wall. **Clamping range 4 to 8 mm** for the 7 mm probe lead |
| Cable gland, **M12 × 1.5**, nylon PA66, IP68, with locknut | 3 now, 4 later (5 in hand) | J4 flow, J5 float and J9 valve, all top wall. The fourth goes in the spare right-wall hole when a solenoid is fitted. **Clamping range 3 to 6.5 mm.** Measure the valve lead first: over 6 mm, use an M16 there and change its hole in the macro |
| **Blanking plug, M12 × 1.5**, nylon, with locknut | 1 | **New 3 Oct.** Closes the spare right-wall hole until it is used |
| Wall bracket or shelf | 1 | Above the tanks, never below. Four mounting ears on the base |

| Wall | Glands |
|---|---|
| Top | J2 probe (M16), J3 probe (M16), J4 flow (M12), J5 float (M12), J9 valve (M12) |
| Right | the DC2 plug tube, and **one spare M12 hole level with J9** (blanking plug) for a future backup solenoid |
| Left, bottom | none |

The spare hole is on the right wall because the top wall has no room:
between J5 and J9 two M12 locknuts would clash, and right of J9 the
hole would cut into the corner screw boss.

**Metric thread only.** PG9 and PG11 glands look almost identical and
do not fit the 12.3 and 16.3 mm holes. Locknut on the inside.

> **The barrel jack costs an IP65 face, and that is accepted.** A printed
> tube bridges the gap and guides the plug in, so power disconnects from
> outside. The box is splash resistant, not sealed. That is fine above
> the tanks in a dry room.

**Measure before printing:** LM2596 height in its socket (sets the box
height, 30 mm above the board allowed), the DC2 hole centre height above
the board (6.5 mm assumed), and the gland clamping ranges.

---

## Plumbing

Order along the run (**3 Oct 2026**):

supply → manual shutoff (male/male) → **union 1** → flow sensor (arrow
with the flow) → **union 2** → adjustable restrictor → valve → lid inlet

The flow sensor goes first so it stays full of water at mains pressure
between fills and reads from the first second of a fill. The restrictor
stays after it: pressure collapses past a restrictor. See
`physical-layout.html` for the reasoning.

| Part | Qty | Note |
|---|---|---|
| Uponor 16 mm MLCP to ½" BSP compression fitting | 1 | Supply transition |
| **Union 1:** flat-face union, G½ swivel nut with flat washer × G½ **female** | 1 | Shutoff's male outlet into the female end, swivel nut onto the sensor inlet. Stainless |
| **Union 2:** flat-face union, G½ swivel nut with flat washer × G½ to suit the restrictor | 1 | Swivel nut onto the sensor outlet. Far end **male** if the restrictor's female end is its inlet, **female** if its male end is. Check the restrictor's arrow before buying. Stainless |
| G½ fittings, nipples, adapters | as needed | Restrictor to valve, valve to inlet pipe. Match actual thread on the real parts. **Leave a straight G½ section or a union directly after the valve**, so a backup solenoid drops in later |
| PTFE tape | 1 | Taper threads only, never on the flow sensor's flat faces |

> **The restrictor is not a nice-to-have.** At the current unrestricted
> estimate of **~7.5 L/min**, about **1.9 litres** arrives after the stop
> decision, roughly 5 cm of depth. At 2 L/min it is about 500 mL.
>
> **That 7.5 L/min is an estimate, not a measurement.** Record the real
> unrestricted rate in the K-factor session.
>
> **Set it to 2 L/min** by watching litres per minute in Home Assistant,
> in the same session that calibrates the flow sensor K-factor against a
> kitchen scale. Record the handle position. Do not run a fill
> unrestricted even once.
>
> Calibrate the K-factor at the rate you will actually run, not at full
> bore. Start value 990 pulses/L. 2 L/min is the bottom edge of the
> sensor's ±3 % band; if the runs scatter, 2.5 L/min is inside it for
> about 125 mL more overshoot.

---

## Deferred: backup solenoid (version 2, not ordered)

Directly after the motorised valve, before the lid inlet. Wired **in
parallel with the valve's open winding**: its two wires land in J9 OPEN
and J9 COM with twin ferrules, entering through the spare right-wall
gland. No PCB change.

| Requirement | Why |
|---|---|
| 24 V **DC** coil, normally closed | An AC coil on DC overheats. Many listings offer 24 V AC only |
| **Direct acting, 0 bar minimum** | A pilot-operated valve chatters or half-opens at 2 L/min |
| **8 W or less** | F3 holds 500 mA. Above that, fit the spare F2 part (1.1 A hold, same 1812 package) in F3's place |
| G½, 304 stainless body, NBR or EPDM seal | Same rules as everything else wetted. Scratch test on arrival |

Candidate seen 3 Oct: AliExpress "2W series" stainless NC, direct acting,
0 to 10 bar, 24 V DC option, female BSP, NBR or Viton. **Coil wattage
not stated**; confirm with the seller. Variant: ½" (DN15), DC24V, NBR.

---

## Open once parts arrive

| Item | Status |
|---|---|
| **Order a third stainless M16 gland** | The lid now carries two probe glands (3 Oct) |
| Swap the stainless body onto the 8 Nm motor | Check stem engagement, mounting screws, and the body's thread standard |
| **Re-time the valve close** | Sets `safety_margin_cm`, `no_flow_grace_ms` and the valve-power hold in the config |
| Float stem length | Measure the gap between lid underside and maximum water line, then pick 45 mm or 75 mm |
| Restrictor thread | Confirm **BSP, not NPT** |
| **Restrictor inlet end** | Decides the far end of union 2 |
| **Measure the unrestricted flow rate** | Replaces the 7.5 L/min estimate. Free during the K-factor session |
| Probe zero calibration | After final mounting. Two weighed 2 L additions |
| Spare element hole blanking plug | Confirm it is in. An open hole passes unfiltered water straight through |
| **Relay test, valve disconnected** | Both relays click on from their GPIOs, release cleanly, stay off through a reboot; lifting the float drops both |
| DC2 barrel check | Plug in, meter on continuity: plug centre must beep to DC2's rear lug |

---

## Changed 3 Oct 2026

| Item | Change |
|---|---|
| **Water path** | Flow sensor moved from after the valve to first after the shutoff. It stays full and pressurised; the firmware's flow-with-valve-closed check now means a passing valve |
| **Plumbing** | Two flat-face unions specified by end type. The shutoff's gender no longer drives the restrictor's |
| **Manual shutoff** | Received |
| **Enclosure** | Spare M12 hole on the right wall, level with J9, plus a blanking plug. For the deferred backup solenoid |
| **Backup solenoid design** | From "powered straight off the float contact" to "in parallel with the valve's open winding at J9". See Deferred |
| **Upper chamber lid** | Three holes to four. The lower probe's cable crosses the upper chamber and leaves through its own lid gland. Stainless M16 tank glands 2 → 3, EPDM M16 washers 4 → 6 |
| **Float switch** | Confirmed: one fitted, the other is a spare |

## Changed in revision G

| Item | Change |
|---|---|
| **K1, K2, Q1, Q2, R11, R12, D3, D4** | Both relays and their MOSFET drivers on the board. The relay module becomes a breadboard spare |
| **D1, D2, J9** | Valve flyback diodes and the valve terminal on the board |
| **F3** | New PTC protecting the relay contacts and valve cable |
| **F2** | Moved from an inline cable fuse onto the board (C142747). The PSU is a brick; the inline holder and the barrel-to-screw adapter are dropped |
| **R4, R5** | Now gate pull-downs |
| **J6, SB3, relay cable** | Removed |
| **Board** | 100 × 80 → 130 × 80 mm |
| **24 V distribution block, diode carrier, relay bay** | Dropped. DC2 is the only power entry, the board the only thing in the box |
| **Enclosure** | Printed from the FreeCAD macro, 154 × 112 × 40 mm. All glands on the top wall. Probe glands M16 |
| **R6, R7, R10, SB2** (2 Oct) | Set by the flow sensor bench test: 10 kΩ, 20 kΩ, not fitted, 5 V |

## Changed in revision F

| Item | Change |
|---|---|
| **Probe 24 V** | Now through the board to J2.1 and J3.1, fused by F1. One cable per probe, no splitting in the box |
| **F1** | New on-board PTC |
| **Test points** | Five bare pads replaced by one 1×6 header, TP1 |
| **R10** | Kept as an empty DNP footprint |
| **U1/U4 swapped** | Corrects a left/right mirror of the ESP32 on the PCB |
| **U3 holes** | The module's two mounting holes are drilled, for standoffs |

## Dropped earlier

| Item | Why |
|---|---|
| **J7, the spare GPIO breakout** | Existed for a second RS-485 bus, a workaround from when the probes would not talk. That was fixed. GPIO32 and GPIO35 are unused. The name J7 is not reused |
| **Leak sensor on the spare GPIOs** | Deferred to a version 2 |
| J1 screw terminal for 24 V in | Replaced by DC2, a barrel jack |
| Solder bridge SB1 | A 0 Ω resistor at R6 did the same job; not needed now |
| Solder jumpers for SB2 and SB3 | Became 1×3 headers with caps; SB3 later removed |
| 470 µF at 35 V | It sits on the 5 V rail. 16 V is smaller |
| JLCPCB assembly for U2 | One SOIC-8 does not justify the setup fee |
| IP65 ABS junction box, 5 × M12 glands | Replaced by the printed enclosure in G |

## Corrected along the way

| Claim | Correction |
|---|---|
| "No enable pins on an auto-direction part" | The THVD1406 SOIC-8 has RE on pin 2 and SHDN on pin 3. Both tie to `+3.3V` |
| Diodes "one per motor wire, band to the positive side" | They go across the windings, not in series. Polarity depends on which rail SR sits on |
| U1 as a single 30-pin ESP32 footprint | Two discrete 1×15 sockets, U1 and U4 |
| Unrestricted flow ~12 L/min, ~3 L overshoot | Revised to ~7.5 L/min and ~1.9 L. **Both are estimates** |
| "1N5408, not 1N4007" | 1N4007 (1 A) is ample for the valve's 0.25 A |
| Adhesive heatsink on the buck module | Not needed at 0.3 to 0.4 A |
| Flow sensor might need a pull-up (R10) | Bench 2 Oct: internal pull-up, R10 stays empty |
| GPIO names typed into the ESP32 symbols' pin number field | EasyEDA pairs pins and pads by number. Names go in the name field |
| M12 glands for the probe cables | The probe lead measures 7 mm. M16 |
| Flow sensor after the valve | Drains between fills and starts each fill dry. Moved first after the shutoff (3 Oct) |
| "Restrictor upstream keeps the closed valve off full line pressure" | A restrictor only drops pressure while water flows. Its real benefit there is limiting flow through the valve while it travels |
| "Backup solenoid powered straight off the float contact" | On rev G the float switches the 5 V coil supply, not 24 V, and an always-on coil would sit hot 24/7. In parallel with the valve's open winding instead (3 Oct) |
| Lid with three holes | The lower probe's cable has to leave the upper chamber too. Four holes, two of them probe glands (3 Oct) |
