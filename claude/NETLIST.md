# Netlist and placement — Berkey fill controller carrier board

Revision G, 28 Sep 2026. Supersedes revisions A through F.
Flow sensor configuration settled by bench test, 2 Oct 2026 (section 5).

Board: **130 × 80 mm** (was 100 × 80), 2 layer, ground pour on the bottom.

**Status 2 Oct 2026, evening: revision G is ORDERED** at JLCPCB, with
the LCSC parts in the same cart. JLC reopens 5 Oct; expect the boards
in about 14 days at the earliest. Everything in section 7 is closed.
Revision G moves **the whole valve circuit onto the board**:
both relays with their MOSFET drivers, the flyback diodes, the valve
terminal and the main fuse. The bought relay module, its cable, the
relay bay and the separate diode carrier are all gone. Goal: clean
cables in and out, every junction on the board.

The ESP32, RS-485, buck and probe sections of the F layout stay. The
right side of the board is redone and the board grows 30 mm to the
right to hold the relays.

**Valve bench test done, 28 Sep 2026: the valve's common (SR, yellow)
is on the negative rail.** COM goes to `GND`, K1 COM takes `RELAY_24V`,
and D1/D2 anodes go to `GND`.

The revision F EasyEDA netlist export of 26 Sep 2026 matched this file
net by net. Revision G changes are marked **(G)**; the G schematic and
PCB netlist exports matched section 4 on 28 Sep. Net names are the
EasyEDA names, so an export can be diffed against this file directly.

The schematic is drawn with net labels, not long wires: each pin gets a
short stub and a label, and pins with the same label are connected.

---

## 1. Pinouts, confirmed

### U2, THVD1406DR (TI datasheet SLLSF87A, SOIC-8)

| Pin | Name | Net |
|---|---|---|
| 1 | R | `UART_RX` |
| 2 | RE | `+3.3V` |
| 3 | SHDN | `+3.3V` |
| 4 | D | `UART_TX` |
| 5 | GND | `GND` |
| 6 | A | `RS485_A` |
| 7 | B | `RS485_B` |
| 8 | VCC | `+3.3V` |

RE and SHDN tied high select auto-direction mode. Tying RE low instead
leaves the receiver permanently on and echoes every transmitted byte back
into RX2.

### U3, LM2596 buck module

| Pin number (= footprint pad) | Name | Net |
|---|---|---|
| INGND | IN− | `GND` |
| INVCC | IN+ | `+24V` |
| OUTVCC | OUT+ | `+5V` |
| OUTGND | OUT− | `GND` |

**Symbol pin number = footprint pad.** EasyEDA pairs pins and pads by
number only. Names go in the name field.

### K1, K2, Songle SRD-05VDC-SL-C (G)

Pin numbers from the LCSC C35449 symbol and footprint (checked 28 Sep,
symbol and footprint agree). Footprint positions are top view.

| Pin | Function | Pad position | K1 net | K2 net |
|---|---|---|---|---|
| 1 | Coil | bottom left | `K1_DRV` | `K2_DRV` |
| 4 | Coil | top left | `FLOAT_OUT` | `FLOAT_OUT` |
| 5 | COM | middle left, alone | `RELAY_24V` | `VALVE_OPEN_FEED` |
| 3 | NC | top right | `VALVE_CLOSE` (close) | not connected |
| 2 | NO | bottom right | `VALVE_OPEN_FEED` | `VALVE_OPEN` (open) |

The coil has no polarity; pin 4 to `FLOAT_OUT` is a choice. D3/D4 still
go cathode to `FLOAT_OUT`.

**NC/NO confirmed on a real relay, 2 Oct 2026.** Checked on the old
relay module (same SRD-05VDC-SL-C), unpowered, continuity mode: NC
identified from the module's NC terminal, COM to NC beeps, COM to NO
silent. Seen from the underside with the three pin end on the left, NC
sits bottom right, which is top right in top view: matches the
footprint. A paper print overlaid on the module's solder joints lined
up with all five pads.

Coil: 5 V, about 70 Ω, about 72 mA. Contacts: 10 A 30 V DC.

### Q1, Q2, AO3400A (G, SOT-23)

| Pin | Name | Q1 net | Q2 net |
|---|---|---|---|
| 1 | G | `Q1_G` | `Q2_G` |
| 2 | S | `GND` | `GND` |
| 3 | D | `K1_DRV` | `K2_DRV` |

Checked 28 Sep against the LCSC C20917 symbol and footprint: 1 G, 2 S,
3 D. Pads 1 and 2 on one side (pin 1 dot), pad 3 alone on the other.

### ESP32 DevKit, as drawn in the EasyEDA symbols

Two discrete 1×15 female headers, ZX-PM2.54-1-15PY.

| U1 pin | Name | Net | | U4 pin | Name | Net |
|---|---|---|---|---|---|---|
| 1 | EN | | | 1 | D23 | |
| 2 | UP | | | 2 | D22 | |
| 3 | UN | | | 3 | TXD | |
| 4 | D34 | | | 4 | RXD | |
| 5 | D35 | | | 5 | D21 | |
| 6 | D32 | | | 6 | D19 | |
| 7 | D33 | `FLOAT_SENSE` | | 7 | D18 | |
| 8 | D25 | `RELAY_IN1` | | 8 | D5 | |
| 9 | D26 | `RELAY_IN2` | | 9 | TX2 | `UART_TX` |
| 10 | D27 | `FLOW_IN` | | 10 | RX2 | `UART_RX` |
| 11 | D14 | | | 11 | D4 | |
| 12 | D12 | | | 12 | D2 | |
| 13 | D13 | | | 13 | D15 | |
| 14 | GND | `GND` | | 14 | GND | `GND` |
| 15 | VIN | `+5V` | | 15 | 3.3v | `+3.3V` |

TX2 is GPIO17, RX2 is GPIO16. Both are free on WROOM-32, not on WROVER.
None of the used pins are strapping pins.

**Left/right, found 27 Sep 2026:** the DevKit plugs in components up,
so the board sees it in **top view**. With the antenna at the bottom,
**EN's row (U1) is on the right, D23's row (U4) on the left.** The
first F layout had them the other way round, which would have put the
module's 3V3 pin on `+5V`. Fixed by swapping U1 and U4.

---

## 2. Components

| Ref | Value or part | LCSC | Footprint | Notes |
|---|---|---|---|---|
| U1, U4 | 1×15 female header, 2.54 mm | C7499333 | HDR-TH_15P-P2.54-V-F | ESP32 sockets |
| U2 | THVD1406DR | C5215918 | SOIC-8 | RS-485 transceiver |
| U3 | LM2596 buck module, 24 V to 5 V, in a 4-pin socket | — | LM2596 | Includes the module's two mounting holes, for M3 standoffs |
| DC2 | DC-005-20A barrel jack | C130239 | DC-IN-TH_DC005 | 24 V in, flush with the **new** right edge |
| **F2 (G)** | PTC, Littelfuse 1812L110/33MR | C142747 | the part's own 1812 | **Main fuse, right behind DC2.** 1.1 A hold, 1.95 A trip, 33 V, 20 A max, 60 mΩ. Moved onto the board because the PSU is a brick with a fixed cable |
| F1 | PTC, LUTE 1206L010/60NRL | C18198325 | F1206 | Probe supply. 100 mA hold, 60 V |
| **F3 (G)** | PTC, PTTC SMD1812P050TF/60 | C462518 | the part's own 1812 | Relay contacts and valve. 500 mA hold, 1 A trip, 60 V, 150 mΩ |
| **K1, K2 (G)** | Relay, Songle SRD-05VDC-SL-C | C35449 | the part's own | Valve drive. Same relay type as the module it replaces |
| **Q1, Q2 (G)** | N-MOSFET, AO3400A | C20917 | SOT-23 | Low-side coil drivers. Fully on from a 3.3 V gate |
| **D1, D2 (G)** | 1N4007 | owned (earlier AliExpress order) | DO-41, 8.7 mm pitch, pin 1 = cathode | **Valve** flyback. D1 across OPEN and COM, D2 across CLOSE and COM |
| **D3, D4 (G)** | 1N4007 | owned | DO-41, 8.7 mm pitch, pin 1 = cathode | **Relay coil** flyback, across each coil, cathode to `FLOAT_OUT` |
| J2, J3 | WJ500V-5.08-04P | C42377749 | CONN-TH_WJ500V-5.08-4P | Probes 1 and 2 |
| J4 | WJ500V-5.08-03P | C72334 | CONN-TH_3P-P5.00 | Flow sensor |
| J5 | WJ500V-5.08-02P | C8465 | CONN-TH_2P-P5.00 | Float switch. **Moves to the top edge (G)** |
| **J9 (G)** | WJ500V-5.08-03P | C72334 | CONN-TH_3P-P5.00 | **Valve out**: OPEN, CLOSE, COM |
| SB2 | 1×3 male header, 2.54 mm, PZ254V-11-03P | C2937625 | HDR-TH_3P-P2.54-V-M | Flow sensor supply select, with jumper cap. **Cap on 1 to 2 (5 V), set 2 Oct** |
| TP1 | 1×6 male header, 2.54 mm, PZ254V-11-06P | C492405 | HDR-TH_6P-P2.54-V-M | Test header |
| R1, R2 | 680 Ω | C2907340 | R0805 | RS-485 bias |
| R3 | 120 Ω | C2907224 | R0805 | Termination |
| **R4, R5 (G)** | 10 kΩ | C101405 | R0805 | **Now gate pull-downs** (were relay-input pull-ups to 3.3 V). Hold Q1/Q2 off while the ESP32 boots |
| R6 | **10 kΩ** | C101405 | R0805 | Flow divider. **Value set 2 Oct** (5 V sensor) |
| R7 | **20 kΩ** | C2907240 | R0805 | Flow divider. **Fitted, set 2 Oct** |
| R8 | 10 kΩ | C101405 | R0805 | Float sense divider |
| R9 | 20 kΩ | C2907240 | R0805 | Float sense divider |
| R10 | **DNP** | — | R0805 | Flow pull-up footprint. **Not fitted: the sensor has an internal pull-up (bench 2 Oct)** |
| **R11, R12 (G)** | 680 Ω | C2907340 | R0805 | Gate series resistors. Same part as R1/R2 |
| C1, C2, C3 | 100 nF | C495959 | C0805 | C1 within 5 mm of U2 pin 8 |
| C4 | 470 µF | C106651 | CAP-TH_BD8.0-P3.50 | 5 V bulk. Also covers the coil inrush |
| — | 2.54 mm jumper cap ×1 | — | — | For SB2 |

**Removed in G:** J6 (relay module header), J8 and J10 (JST XH to the
relay module), SB3 (relay opto supply select), the external relay
module and the separate diode carrier. See section 8.

---

## 3. Placement

Coordinates in mm from the bottom-left corner of the **130 × 80** board,
Y upward. The left part (X 0 to 85) is the F layout, kept as it is.
X 85 to 130 is new.

**Setting up EasyEDA:** put the canvas origin on the board's bottom-left
corner. Check which way Y grows: if Y grows downward in your editor,
use 80 − Y for every value below. The X/Y fields in the properties panel
set the footprint's **origin**, which is usually but not always the
body centre. After placing a part, check that its body lands in the
range column; if not, shift it by the difference.

### Left side, unchanged from F

| Ref | X range | Y range | Note |
|---|---|---|---|
| U4 | 10 | 8 to 46 | Left row (D23 … 3.3v). Do not move |
| U1 | 35.4 | 8 to 46 | Right row (EN … VIN). Do not move |
| U3 | 41 to 84 | 12 to 33 | `IN+` end toward DC2 |
| U2, C1, R1, R2, R3 | 42 to 62 | 54 to 64 | RS-485 cluster |
| R6 to R10, C2, C3, C4 | 44 to 72 | 38 to 50 | Passive cluster (R4/R5 move to the drivers) |
| SB2 | 74 to 82 | 38 to 50 | |
| TP1 | 64 to 80 | 54 to 64 | |
| J2, J3, J4 | as in F | 66 to 76 | Top edge terminals. **J5 and J9 use the same centre Y as these** |

### Right side, new in G: centre coordinates

| Ref | Centre X | Centre Y | Rotation, orientation | Body range |
|---|---|---|---|---|
| **J5** | 82 | 71 (= J4) | Wire entry facing the top edge, like J4 | 77 to 87 |
| **R11** | 89 | 37 | Horizontal, pin 1 left (from U1.8) | |
| **R4** | 89 | 33 | Horizontal, pin 1 (`GND`) left | |
| **Q1** | 95 | 35 | Gate (pin 1) facing left toward R11 and R4, drain (pin 3) facing right | |
| **D3** | 101 | 38 | Vertical, **cathode band up** (toward K1 pin 4) | Y 33.6 to 42.4 |
| **K1** | 113.5 | 38 | As in the library: coil pins left, NC/NO right | 104 to 123, 30.25 to 45.75 |
| **R12** | 89 | 55 | As R11 | |
| **R5** | 89 | 51 | As R4 | |
| **Q2** | 95 | 53 | As Q1 | |
| **D4** | 101 | 56 | As D3, cathode band up | Y 51.6 to 60.4 |
| **K2** | 113.5 | 56 | As K1 | 104 to 123, 48.25 to 63.75 |
| **J9** | 113.5 | 71 (= J4) | Wire entry facing the top edge. This footprint numbers pin 1 on the **right** (like J2 to J4), so read left to right: COM, CLOSE, OPEN | 106 to 121 |
| **D1** | 99 | 74 | Horizontal, **cathode band right** (toward J9) | X 94.6 to 103.4 |
| **D2** | 99 | 68 | Horizontal, cathode band right | X 94.6 to 103.4 |
| **DC2** | 119.19 (ref point) | 19 (ref point) | Plug opening facing right, front face on X 130. **Barrel axis = pad 1: X 116.09, Y 16.65** (the ref point is pulled up by pad 3). Enclosure JACK_Y = 16.65 | 115.5 to 130 |
| **F2** | 111 | 23 | Pad 1 (`DC_IN`) toward DC2.1 | |
| **F3** | 111 | 15 | Pad 2 (`RELAY_24V`) up, toward K1 COM | |
| **F1** | 105 | 15 | Pad 2 (`PROBE_24V`) left | |
| Mounting holes | 4 and 126 | 4 and 76 | M3, 3.2 mm, non-plated. **X 96 moves to X 126** | |

The coordinates for Q, R and D are a starting grid; nudge them for
routing. What must hold: D3/D4 next to the coil pins, F2 next to DC2.1,
D1/D2 next to J9, and 24 V tracks at least 2 mm from logic.

**Enclosure link:** the enclosure macro puts the glands at X 18, 43, 62,
82 and 113.5 and the jack at Y 19. After placement, send the real
centre X of J2, J3, J4, J5, J9 and the centre Y of DC2, and the macro
follows the board.

**All field wiring now leaves on the top edge** (J2, J3, J4, J5, J9),
plus DC2 on the right. Logic on the left, 24 V switching on the right.

**Routing notes (G):**

- **24 V side.** `RELAY_24V`, `VALVE_OPEN_FEED`, `VALVE_OPEN`, `VALVE_CLOSE`
  at 0.5 mm (0.25 A), kept in the right-hand third. At least 2 mm from
  any logic track. Nothing 24 V passes under the ESP32.
- **`FLOAT_OUT`** from J5 to both coils, 0.5 mm (144 mA with both coils
  on).
- **Coil diodes D3, D4** right at the relay coil pins.
- **`+24V` from DC2 to U3 IN+** is now about 30 mm longer. 1.0 mm.
- **`PROBE_24V`** from F1 to J2/J3: 0.5 mm, top layer, **routed around
  the board edge** (28 Sep): from F1 pad 2 along the bottom edge, up the
  right edge, along the top edge above the terminals, into J3.1. No via,
  no crossings, bottom pour intact. J2.1 to J3.1 was already linked on
  the bottom (from F). F1 stays next to F2 so the whole run is fused at
  100 mA. Keep at least 1 mm between it and the top right mounting hole
  keepout.
- **RS-485 pair** side by side, same length, ground either side, away
  from U3 and from the relay area.

---

## 4. Netlist

Revision F verified nets, with revision G changes marked **(G)**.

| Net | Nodes |
|---|---|
| **`DC_IN` (G)** | DC2.1, F2.1 |
| `+24V` | **F2.2 (G)**, F1.1, F3.1, U3 INVCC (IN+) |
| `PROBE_24V` | F1.2, J2.1, J3.1 |
| **`RELAY_24V` (G)** | F3.2, K1 COM |
| **`VALVE_OPEN_FEED` (G)** | K1 NO, K2 COM |
| **`VALVE_OPEN` (G)** | K2 NO, J9.1, D1 cathode |
| **`VALVE_CLOSE` (G)** | K1 NC, J9.2, D2 cathode |
| `+5V` | U3 OUTVCC (OUT+), U1.15 VIN, C3.2, C4.1 (+), J5.1, SB2.1, TP1.1 |
| `+3.3V` | U4.15 3.3v, U2.2, U2.3, U2.8, C1.1, C2.2, R1.1, SB2.3, TP1.3 **(G: R4.1, R5.1, SB3.3 removed)** |
| `GND` | DC2.2, U3 INGND, U3 OUTGND, U1.14, U4.14, U2.5, C1.2, C2.1, C3.1, C4.2, R2.2, R7.2, R9.2, J2.2, J3.2, J4.2, TP1.2, TP1.6, **R4.1, R5.1, Q1 S, Q2 S, J9.3, D1 anode, D2 anode (G)** |
| `RS485_A` | U2.6, R1.2, R3.1, J2.3, J3.3, TP1.4 |
| `RS485_B` | U2.7, R2.1, R3.2, J2.4, J3.4, TP1.5 |
| `UART_TX` | U4.9 TX2, U2.4 |
| `UART_RX` | U4.10 RX2, U2.1 |
| `FLOW_VCC` | SB2.2, J4.1, R10.1 |
| `FLOW_RAW` | J4.3, R6.1, R10.2 |
| `FLOW_IN` | U1.10 D27, R6.2, R7.1 |
| `FLOAT_OUT` | J5.2, R8.1, **K1 coil +, K2 coil +, D3 cathode, D4 cathode (G)** |
| `FLOAT_SENSE` | U1.7 D33, R8.2, R9.1 |
| `RELAY_IN1` | U1.8 D25, **R11.1 (G)** |
| `RELAY_IN2` | U1.9 D26, **R12.1 (G)** |
| **`Q1_G` (G)** | R11.2, Q1 G, R4.2 |
| **`Q2_G` (G)** | R12.2, Q2 G, R5.2 |
| **`K1_DRV` (G)** | K1 coil −, Q1 D, D3 anode |
| **`K2_DRV` (G)** | K2 coil −, Q2 D, D4 anode |

### Schematic worksheet for revision G, pin by pin

**Change on existing parts:**

| Part | Pin | Rev F label | Rev G label |
|---|---|---|---|
| R4 | 1 | `+3.3V` | `GND` |
| R4 | 2 | `RELAY_IN1` | `Q1_G` |
| R5 | 1 | `+3.3V` | `GND` |
| R5 | 2 | `RELAY_IN2` | `Q2_G` |
| DC2 | 1 | `+24V` | `DC_IN` |
| J6, SB3 | all | | delete the parts and their labels |

`RELAY_IN1` and `RELAY_IN2` stay on U1.8 and U1.9; each now has R11.1
or R12.1 as its only other pin.

**New parts:**

| Part | Pin | Net |
|---|---|---|
| F2 | 1 | `DC_IN` |
| F2 | 2 | `+24V` |
| F3 | 1 | `+24V` |
| F3 | 2 | `RELAY_24V` |
| R11 | 1 | `RELAY_IN1` |
| R11 | 2 | `Q1_G` |
| R12 | 1 | `RELAY_IN2` |
| R12 | 2 | `Q2_G` |
| Q1 | 1 G | `Q1_G` |
| Q1 | 2 S | `GND` |
| Q1 | 3 D | `K1_DRV` |
| Q2 | 1 G | `Q2_G` |
| Q2 | 2 S | `GND` |
| Q2 | 3 D | `K2_DRV` |
| K1 | 4 coil | `FLOAT_OUT` |
| K1 | 1 coil | `K1_DRV` |
| K1 | 5 COM | `RELAY_24V` |
| K1 | 3 NC | `VALVE_CLOSE` |
| K1 | 2 NO | `VALVE_OPEN_FEED` |
| K2 | 4 coil | `FLOAT_OUT` |
| K2 | 1 coil | `K2_DRV` |
| K2 | 5 COM | `VALVE_OPEN_FEED` |
| K2 | 3 NC | no-connect flag |
| K2 | 2 NO | `VALVE_OPEN` |
| D3 | cathode (band) | `FLOAT_OUT` |
| D3 | anode | `K1_DRV` |
| D4 | cathode (band) | `FLOAT_OUT` |
| D4 | anode | `K2_DRV` |
| D1 | cathode (band) | `VALVE_OPEN` |
| D1 | anode | `GND` |
| D2 | cathode (band) | `VALVE_CLOSE` |
| D2 | anode | `GND` |
| J9 | 1 | `VALVE_OPEN` |
| J9 | 2 | `VALVE_CLOSE` |
| J9 | 3 | `GND` |

Diodes are given by anode and cathode because the pin number of the
cathode differs per library symbol. Check which pin carries the K or the
bar in the symbol you pick. Fuses have no polarity; the numbers above
only fix which side faces the supply.

No-connect flags: K2 pin 3 and DC2 pin 3.

**Removed nets (G):** `RELAY_VCC` (SB3.2, J6.2). J6, J8, J10 and SB3
nodes are gone from every net.

Unconnected: DC2.3, K2 NC, and all ESP32 pins not listed above.

### Valve wires, measured 28 Sep 2026

Bench test confirms the label:

| Wire | Does |
|---|---|
| SR, yellow | common |
| **BL, blue** | **opens** |
| **BR, brown** | **closes** |

Stops by itself at both ends (internal limit switches), so it never
stalls against an end stop. **Current measured 28 Sep:** at most
0.03 A while moving (10 A range, 0.01 A resolution), **0.00 A once
stopped**, both directions. So keeping 24 V on the close wire at rest
is safe, and F3 (500 mA hold) has a wide margin. The reading may rise
somewhat under water pressure; still far below F3. **Travel on the
bench: 10 to 11 s open and close** (label says 15 s). Re-measure current
and travel once installed, under pressure. The board and firmware use function names
(`VALVE_OPEN`, `VALVE_CLOSE`, COM) rather than the label codes.

### Field wiring per terminal

| Terminal | Pin 1 | Pin 2 | Pin 3 | Pin 4 |
|---|---|---|---|---|
| J2, J3 (probes) | red, 24 V | green, GND | blue, A | yellow, B |
| J4 (flow, YF-B1-S) | red, supply | black, GND | yellow, signal | |
| J5 (float) | wire 1, +5 V | wire 2, `FLOAT_OUT` | | |
| **J9 (valve, G)** | OPEN: **BL, blue** | CLOSE: **BR, brown** | COM: **SR, yellow** | |

| TP1 pin | Net |
|---|---|
| 1 | `+5V` |
| 2 | `GND` |
| 3 | `+3.3V` |
| 4 | `RS485_A` |
| 5 | `RS485_B` |
| 6 | `GND` |

### How the valve circuit works (G)

- **At rest, both coils off:** K1 sits on NC, so `RELAY_24V` drives
  `VALVE_CLOSE` and the valve runs **closed**. K2 is open, so nothing can
  reach the open wire.
- **Opening needs both relays on:** K1 on connects `RELAY_24V` to
  `VALVE_OPEN_FEED`, K2 on passes it to `VALVE_OPEN`.
- **Either relay dropping out stops the open drive.** K1 dropping out
  drives the valve closed.
- **Float interlock, in copper:** the float's NC contact sits between
  `+5V` (J5.1) and `FLOAT_OUT` (J5.2), and `FLOAT_OUT` is the coil
  supply for both relays. Water lifts the float, the contact opens,
  both coils lose power whatever the MOSFETs do, K1 falls to NC, the
  valve is driven closed. No firmware involved, and it works with the
  ESP32 unplugged.
- **Boot:** R4 and R5 hold both gates low until the ESP32 drives the
  pins. Both relays stay off, so the valve stays closed through every
  reboot.

**Valve common confirmed on the negative rail (28 Sep bench test).**
J9.3 is `GND`, K1 COM is `RELAY_24V`, D1 and D2 have their anodes on
`GND` and their cathode bands on `VALVE_OPEN` and `VALVE_CLOSE`.

### Why the relays moved onto the board (G, 28 Sep)

- Clean cables in and out, every junction inside the box on the board.
  Nothing runs to a module: J6, J8, J10, the Dupont cable, the XH
  pigtails and the loose bridge wire are gone.
- **The 3.3 V drive problem disappears.** The module's opto inputs fed
  from a 3.3 V GPIO could leave a channel half energised; that is why
  SB3, the JD-VCC jumper choice and the release bench test existed. A
  MOSFET driven from 3.3 V switches cleanly. The AO3400A is specified
  at 2.5 V gate drive.
- The module's opto isolation was nominal anyway: its grounds were
  measured common on 20 Sep.
- The enclosure loses its relay bay.

**F3 stays.** A pinched valve cable would otherwise short the 8 A
supply through board traces. The valve draws 0.25 A while running;
500 mA hold keeps F3 cold in normal use.

### Why the probes are fed through the board

Revision F routes the probes' 24 V through J2.1 and J3.1, so each probe
cable lands whole in one terminal. F1 protects that trace against a
pinched probe cable.

### Circuit notes

- **RS-485 bias:** 680/120/680 at 3.3 V gives 268 mV idle differential,
  above the 200 mV threshold. The THVD1406 has built-in fail-safe too.
- **Flow divider:** R6/R7 bring a 5 V signal down to 3.33 V at D27.
  The sensor drives its output high itself (internal pull-up), and the
  20 kΩ leg pulls `FLOW_IN` low between pulses.
- **Float divider:** R8/R9 bring `FLOAT_OUT` down to 3.33 V at D33.
  High when dry. When tripped, `FLOAT_OUT` is pulled to ground through
  R9 (or through a coil and its MOSFET), so D33 reads low. **The
  firmware must not enable GPIO33's internal pull-up** (section 7,
  firmware): about 45 kΩ against the 20 kΩ leg leaves the pin near
  1.0 V when tripped, which is not a valid low.
- **Coil current:** 2 × 72 mA from `+5V` through the float contact,
  within the float's 1.2 A rating and the buck's capacity. C4 covers
  the switch-on inrush.
- **Flow sensor current:** 2 mA at 5 V, measured 2 Oct. Negligible.
- **Probe return current:** 40 to 60 mA through J2.2/J3.2 into the
  pour. Not a defect.
- **Valve return current:** up to 0.25 A running through J9.3 into the
  pour on the right side, away from the RS-485 and ESP32 ground.

---

## 5. Configuration after assembly

### SB2, flow sensor supply

| Cap on | Supply to J4.1 |
|---|---|
| **pins 1 to 2** | **5 V. Chosen, 2 Oct** |
| pins 2 to 3 | 3.3 V |

### R6, R7 and R10, set by the flow sensor bench test

| Sensor | R6 | R7 | R10 | SB2 cap |
|---|---|---|---|---|
| 5 V, open-collector | 10 kΩ | 20 kΩ | **4.7 kΩ** | 1 to 2 |
| **5 V, internal pull-up (this one)** | **10 kΩ** | **20 kΩ** | **not fitted** | **1 to 2** |
| 3.3 V, open-collector | **0 Ω** | DNP | **10 kΩ** | 2 to 3 |
| 3.3 V, internal pull-up | **0 Ω** | DNP | not needed | 2 to 3 |

If R10 is ever needed it sits between `FLOW_VCC` and `FLOW_RAW`, 0805.

SB3 no longer exists. There is no relay module jumper to set.

### Flow sensor, YF-B1-S, bench test 2 Oct 2026

| Item | Result |
|---|---|
| Model | YF-B1-S, stainless body, G½ both ends |
| Material | Scratch test inside the thread: grey, passes. Rotor and inner housing are plastic ("stainless steel exterior") |
| Thread | About 9 mm per side, about 20.1 mm measured outside diameter. Seals with a flat washer on the face, so the mating fittings need flat-face unions or nuts with gaskets |
| Direction | Inlet 15.4 mm, outlet 13.5 mm. Follow the arrow on the body |
| Supply | Listing: 3.5 to 24 V (another table says 5 to 24 V). Run at 5 V |
| Current | 2 mA at 5 V |
| Wires | Red +, black GND, yellow signal |
| Output | **Internal pull-up.** Yellow sits at 5 V idle and averages down while spinning. Spec: high level above 4.5 V at 5 V, 50 % ±10 % duty |
| Pulse formula | f (Hz) = 18 × Q (L/min) − 3, ±3 % from 2 to 6 L/min. Table starts at 2 L/min |
| Pulses per litre | 1080 − 180/Q. **About 990 at 2 L/min** (listing's 1077 is the high flow figure). Use 990 as the start K factor |
| Still open | Calibrate K with water at the real restricted flow: fill 1 or 2 litres into a jug, divide pulses by litres, three runs, average |

---

## 6. PCB layout

**Revision F layout complete for the 100 × 80 board. Revision G reuses
its left part and redoes the right.**

### Board setup

| Item | Setting |
|---|---|
| Outline | **130 × 80 mm (G)**, origin bottom left |
| Mounting holes | Pads, 3.2 mm hole, 3.2 mm pad, **plated: no**, no net, at (4,4) **(126,4)** (4,76) **(126,76)**. Keep copper 3 mm from the centre |
| ESP32 | U4 at X 10, U1 at X 35.4, pin 1 at the bottom (Y ≈ 8) |
| Antenna keepout | Between the header rows at the bottom, both layers |
| DC2 | Bottom right, **flush with the new right edge**, plug opening facing out |
| U3 | Bottom middle, INVCC toward DC2, rotated 180°, layer Top |
| C1 | About 3 mm from U2 pin 8 |

### Design rules (JLCPCB two-layer preset, raised)

| Rule | Value |
|---|---|
| Clearance track, SMD pad, TH pad, via | 0.2 mm |
| Clearance to copper pour | 0.3 mm |
| Track width | min 0.2, default 0.3, max 2.54 |
| Via | 0.61 outer, 0.305 hole |

### Track widths

| Net | Width |
|---|---|
| `DC_IN`, `+24V`, `+5V` | 1.0 mm |
| `+3.3V`, `PROBE_24V`, **`RELAY_24V`, `VALVE_OPEN_FEED`, `VALVE_OPEN`, `VALVE_CLOSE`, `FLOAT_OUT`, `K1_DRV`, `K2_DRV` (G)** | 0.5 mm |
| Everything else | 0.3 mm |

A via next to a 1.0 mm track needs 1.0 mm from the track centre to the
via centre; next to 0.5 mm, 0.75 mm. Use 1.5 mm for margin. **Every via
needs its net set.**

### Routing decisions kept from F

- `+5V` feeds from C4, along a lane above both header rows (about Y 50),
  down into U1 pin 15.
- `+3.3V` crosses the 5V lane with short bottom-layer hops.
- UART on the bottom layer, a via at U2 pins 1 and 4, straight to U4
  pins 10 and 9.
- ESP32 signals thread between U4's pins, one 0.3 mm track per gap.
- Tracks leaving a row of pins in order must arrive in the same order.
- RS-485: U2 pins 6/7 up into J3, parallel lanes to J2, B takes one via.
- `PROBE_24V` J3.1 to J2.1 on the bottom layer below the pin row.
- **`RELAY_IN1`/`RELAY_IN2` (G)** now run right, from D25/D26 to R11/R12
  in the driver strip, instead of to J6.

### Silkscreen

Kept from F: J2/J3 `B A GND 24V`, J4 `SIG GND VCC`, TP1 net names,
`3V3`/`5V` at SB2, `24V` at DC2, `+` at C4, `R10 DNP`, `USB` and `ANT`
markers.

**New for G:** J5 `5V FLT` (moved), J9 `COM CLOSE OPEN` (left to right, pin 1 is on the right), diode bands on D1 to
D4, `K1 CLOSE` and `K2 OPEN`, a `24V` warning line along the relay
area, board name "Berkey fill rev G 2026-09".

### DRC

F and G: clean except two accepted errors inside the LM2596 footprint
(its own mounting holes, kept for M3 standoffs).

### EasyEDA Pro lessons

- **Symbol pin number = footprint pad.** Names go in the name field.
- Schematic DRC catches single-use net labels and pin/pad mismatches.
- "Connection" errors in PCB DRC are only unrouted ratsnest lines.
- Ratsnest lines show the shortest link to any pad on a net, not the
  intended one.
- Mounting holes: Place > Pad with plating off.
- **The pour fill can go missing** (28 Sep: the Gerber preview showed a bare bottom). Rebuild all pours (Shift+B) right before DRC and again right before the Gerber export, then check the bottom side in the Gerber preview.
- **The EasyEDA BOM export leaves out DNP parts and includes parts you
  already own.** After importing it into the LCSC cart, remove the owned
  parts (DC2, the 3-pin terminals, SB2, TP1) and raise quantities for the
  number of boards you will populate.

---

## 7. Pre-order checklist, revision G

### Must do, in order

| # | Item | Status |
|---|---|---|
| G1 | **Bench: valve wires and SR rail.** BL blue opens, BR brown closes, SR yellow common, as labelled. Stops by itself at both ends. **SR on minus: common is the negative rail** | **Done 28 Sep** |
| G2 | Find the 1N4007s (four needed: D1 to D4) | **Done 28 Sep**, plenty |
| G3 | Parts checked on LCSC: F2 C142747, F3 C462518, K1/K2 C35449, Q1/Q2 C20917. Read the relay and MOSFET pin drawings, then meter check relay NC/NO | **Done.** Pins in section 1. **NC/NO confirmed on a real relay 2 Oct** |
| G4 | **Schematic**: remove J6 and SB3; add K1, K2, Q1, Q2, R11, R12, D1 to D4, F3, J9; R4/R5 to gate pull-downs; move J5. Net labels per section 4. Schematic DRC | **Done 28 Sep.** Netlist export diffed against section 4: all 26 nets match |
| G5 | **Board outline to 130 × 80**, move the two right mounting holes to X 126 | **Done 28 Sep** |
| G6 | **Place** per section 3. Rip up the old right side (J6, J5, SB3, DC2) | **Done 28 Sep.** DC2 left at centre Y about 16.5; the enclosure follows the board instead (JACK_Y) |
| G7 | **Route** the new nets at the section 6 widths; 24 V side kept right | **Done 28 Sep.** `PROBE_24V` around the board edge. New SMD GND pads (Q1 S, Q2 S, R4.1, R5.1) each have a via to the bottom pour |
| G8 | Silkscreen per section 6 | **Done 28 Sep.** J9 fixed to COM CLOSE OPEN |
| G9 | Rebuild the pour **over the full 130 × 80**, add the GND vias, DRC with the connection check on: no ratsnest lines left | **Done 28 Sep** |
| G10 | **Netlist export diffed against section 4** | **Done 28 Sep.** PCB netlist identical to the schematic netlist, which matched section 4 |
| G11 | 3D view: J2 to J5 and J9 wire entries face the edge; relay and diode orientation | **Done 28 Sep** |
| G12 | Gerber export and preview | **Done 2 Oct.** Final round: pours rebuilt, DRC (only the two U3 errors), re-exported, bottom pour and 130 × 80 outline checked in the preview. Old zip discarded |
| G13 | 1:1 paper print, checked against the parts on hand | **Done 2 Oct.** ESP32, buck module and the relay pattern (paper over the old module's solder joints) all line up. Terminals and DC2 not on hand at the time; library footprints for known LCSC parts, terminal pitch already measured |
| G14 | JLC order number location (`JLCJLCJLCJLC`, or paid removal) | **Done 2 Oct** |
| G15 | **Order JLC + LCSC in one cart** | **Done 2 Oct, evening.** JLC production starts 5 Oct |

### Order settings (JLCPCB), as ordered

| Setting | Value |
|---|---|
| Layers | 2 |
| Size | **130 × 80 mm**. Outside the 100 × 100 promo price, a few euros more |
| Thickness | 1.6 mm |
| Surface finish | Lead-free HASL |
| Copper | 1 oz |
| Order number | Specify location |
| Electrical test | Flying probe, fully tested (default) |
| Gold fingers, castellated holes, edge plating, blind slots, UL marking, humidity card | No |
| Quantity | 5 (minimum) |

### Firmware changes when moving from the breadboard to the PCB

`waterfilter.yaml` stays breadboard-correct until the board is in use.
At the switch-over:

1. **GPIO25 and GPIO26: remove `inverted: true`.** The MOSFETs switch a
   relay on when the pin goes high. Keep `restore_mode: ALWAYS_OFF`.
   Getting this wrong energises both relays at boot, which opens the
   valve.
2. **GPIO33: remove `pullup: true`.** The R8/R9 divider sets the level.
   Keep `inverted: true` (low = tripped).
3. **Remove `flow_control_pin: GPIO4`.** The THVD1406 is auto-direction.
4. Update the header comments: the float breaks the coil supply on the
   board, and there is no JD-VCC jumper any more.
5. **GPIO27 (flow):** no internal pull-up; the sensor drives the line.
   K factor 990 pulses per litre until calibrated (section 5).

### Bench, while the boards are on the way

- Flow K factor calibration on the breadboard, water at 2 L/min; this also gives the real flow rate
- Re-time the valve with the stainless body fitted
- Plumb the run as the AliExpress parts arrive (BOM.md, Plumbing)

### Bench, after the boards arrive

- DC2 centre pin continuity to the rear lug (checks the footprint; the brick itself is confirmed centre-positive)
- **First power-up without the ESP32 and buck module seated**: 24 V in,
  nothing warm, 24 V after F2 and on K1 COM; then the buck alone, `+5V` at TP1 pin 1;
  then seat the ESP32, `+3.3V` at TP1 pin 3
- **Relay test before the valve is connected:** both relays click on
  from their GPIOs and release cleanly; both stay off through a reboot;
  lifting the float cuts both

### Closed

| Item | Result |
|---|---|
| ESP32 row spacing | 25.4 mm |
| ESP32 pin 1 end | EN (U1.1) and D23 (U4.1) at the antenna end |
| Terminal pitch | All WJ500V footprints 5.08 mm |
| C4 polarity | + pad on `+5V` |
| DC2 pad 1 | Rear pad, centre pin, on `+24V` |
| F1 datasheet | LUTE 1206L010/60NRL: 100 mA hold, 250 mA trip, 60 V |
| F3 part | C462518, SMD1812P050TF/60 |
| Relay release at 3.3 V drive | **No longer needed (G)**, MOSFET drivers |
| Relay NC/NO position | **2 Oct:** confirmed on a real SRD-05VDC-SL-C, matches the footprint |
| Flow sensor supply and output type | **2 Oct:** 5 V, internal pull-up. SB2 1 to 2, R6 10 kΩ, R7 20 kΩ, R10 not fitted |
| Float switch rating | 1.2 A, 20 W; flipped to NC (opens when lifted); wire to stem isolation OL. Bench 30 Sep |
| Order | **2 Oct:** JLCPCB board and LCSC parts ordered together |

---

## 8. Revision history

### G, from F (27 and 28 Sep 2026)
| Change | Why |
|---|---|
| **K1, K2 on the board**, SRD-05VDC-SL-C, with Q1/Q2 AO3400A drivers, R11/R12 gate resistors, D3/D4 coil diodes | Whole valve circuit on the board. Clean 3.3 V drive; removes the module, its cable and the relay bay |
| **R4, R5 now gate pull-downs** | The drivers are active high. Pull-downs keep both relays off during boot |
| **D1, D2 valve flyback on the board**, J9 valve out | Every junction on the board; the valve cable lands on J9 |
| **F3**, PTC between `+24V` and `RELAY_24V` | Protects the valve circuit against a pinched cable |
| **F2 on the board**, main PTC right behind DC2, new net `DC_IN` | The PSU is a brick with a fixed cable, so an inline fuse holder has nowhere to go. The brick's own short-circuit protection covers its cable |
| **J5 moved to the top edge**, DC2 to the new right edge | All field cables leave through the top wall; the right edge holds only DC2 |
| **Board 100 × 80 → 130 × 80**, right mounting holes to X 126 | Room for the relays and drivers |
| **Removed:** J6, SB3, net `RELAY_VCC`, and the J8/J10 JST XH idea from 27 Sep | No module to connect to. SB3 only existed for the module's opto inputs |
| New nets `RELAY_24V`, `VALVE_OPEN_FEED`, `VALVE_OPEN`, `VALVE_CLOSE`, `Q1_G`, `Q2_G`, `K1_DRV`, `K2_DRV` | |

Earlier on 27 Sep, G briefly had J8 (JST XH, 24 V out to an external
relay module) and a stripboard diode carrier. Both were superseded on
28 Sep by putting the relays on the board.

**2 Oct 2026, no copper change:** flow sensor bench test fixes the
assembly values (SB2, R6, R7, R10). Relay NC/NO confirmed, final Gerber
export and paper print done, order number location set. **Ordered the
same evening.**

### F, from E
Schematic redrawn with net labels; net names adopted from EasyEDA;
probe 24 V through J2.1/J3.1 with F1; TP1 header; R10 as DNP; U1 and U4
swapped on the PCB to fix a mirrored ESP32; U3 pin numbers recorded.

### E, from D
R10 added as DNP, J7 removed, TP1 to TP5 pads added.

### D, from C
SB1 removed (0 Ω at R6 instead). SB2 and SB3 became 1×3 headers.

### C, from B
Barrel jack DC2, J6 as 2.54 mm header, U1 split into U1 and U4.

### B, from A
U2 pin numbers, RE and SHDN to 3.3 V, J6 to 5 pins, SB2 and SB3 added.
