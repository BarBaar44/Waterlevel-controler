# Wiring plan — two-tank water filter controller

A build guide written for someone who is comfortable with a screwdriver but not
an electrician. Work through it in order. Every stage ends with a test, so if
something is wrong you find out immediately instead of three stages later.

---

## Before anything else: three rules

**1. Never work on this while it is plugged in.** Unplug the power supply, make
your change, then plug it back in. Every time. This is not about danger to you
(12 V cannot hurt you) — it is about not destroying parts by touching the wrong
two things together while they are live.

**2. Keep water and electronics apart.** The probes and the flow sensor are meant
to get wet. Nothing else is. The ESP32, the MAX485 and the relay live in a box
somewhere dry, above the tanks, never below them. Water runs downhill.

**3. If you are ever unsure, measure it.** A cheap multimeter is the single most
useful thing you can own for this project. You will use it constantly.

---

## The one idea you need to understand: ground

Electricity needs a complete loop. Current leaves the power supply's positive
terminal, travels through your part, and must return to the power supply's
negative terminal. That return path is called **ground**, or **GND**, or the
**negative** terminal. All the same thing.

Here is the rule that catches everyone:

> **Every single component in this project must connect back to the same ground.**

Not "a" ground — *the* ground. The ESP32's GND, the MAX485's GND, the probes'
black wires, the flow sensor's black wire, the relay's ground, the power
supply's negative — all joined together into one common network.

If two parts don't share a ground, they have no shared reference for what
"3.3 volts" even means, and they cannot talk to each other. Missing ground
connections are the number one cause of "I wired it exactly right and nothing
works."

By convention: **red wires carry power, black wires carry ground.** Stick to it
religiously and you will save yourself hours.

---

## Parts and tools you need

### Tools

| Item | Why |
|---|---|
| Multimeter | Non-negotiable. €15 is fine. |
| Small flat screwdriver | For the green screw terminals |
| Wire strippers | Scissors will do at a push |
| Breadboard (830-point) | Build it here first, solder later |
| Jumper wires (male-male, male-female) | A pack of 40 of each |

### Electronic bits

| Item | Quantity | Note |
|---|---|---|
| 24 V DC power supply, 2 A+ | 1 | Confirmed required — both level probes are labelled DC 24V |
| 12 V DC power supply, 2 A | 1 | Currently feeds the valve/relay; may be removed later if the whole build moves to one 24V rail — see note below |
| Barrel jack to screw terminal adapter | 2 | One per supply |
| Buck converter module (24 V or 12 V → 5 V) | 1 | The little blue MP1584 or LM2596 ones |
| 10 kΩ resistor | 5 | Buy a pack, they cost nothing |
| 20 kΩ resistor | 2 | Or use two 10 kΩ in series — same thing |
| 4.7 kΩ resistor | 1 per MAX485 module | **New, required.** Pull-up on `RO` — see stage 3, "the RO pull-up" |
| 560 Ω–1 kΩ resistor | 2 | RS-485 bias resistors — see stage 3 note |
| 2-channel relay module | 1 | Opto-isolated type |

> **Pending decision, not yet built:** collapsing this to a single 24V rail
> feeding both probes and the valve directly, with one buck converter
> stepping 24V straight to 5V for the ESP32/MAX485/relay logic — removing
> the 12V stage entirely. Depends on confirming the relay module's contact
> rating for switching 24V DC specifically (not just its AC rating), which
> is being checked separately. Until confirmed, build with both supplies
> as listed above.

**On resistors:** they have coloured stripes that encode the value. 10 kΩ is
brown-black-orange. 20 kΩ is red-black-orange. But don't trust your eyes —
set your multimeter to the Ω setting and measure each one before you use it.
Takes two seconds and removes all doubt.

---

## Two circuits you will build repeatedly

You will build these three times in total. Understand them once here and the
rest of the guide is just following instructions.

### The voltage divider (used twice)

**Problem:** The MAX485 and the flow sensor both send out signals at 5 volts.
The ESP32 can only accept 3.3 volts. Feeding it 5 V will damage it.

**Solution:** Two resistors that split the voltage. The 5 V signal goes in one
end, and you tap off a lower voltage from the middle.

```
   5 V signal  ────[ 10 kΩ ]────┬────[ 20 kΩ ]──── GND
                                │
                                └──── to ESP32 pin (3.3 V here)
```

Read it as a chain: signal wire, then a 10 kΩ resistor, then a junction, then a
20 kΩ resistor, then ground. The wire to the ESP32 comes off that middle
junction.

Why it works: the two resistors share the 5 V between them in proportion to
their size. The 20 kΩ is twice as big as the 10 kΩ, so it takes two-thirds of
the voltage — and two-thirds of 5 V is 3.33 V. Exactly what we want.

**On a breadboard:** push the 10 kΩ resistor in so its two legs are in different
rows. Push the 20 kΩ in so one leg shares a row with the 10 kΩ's second leg,
and its other leg goes to the ground rail. Then run a jumper from that shared
row to the ESP32 pin.

### The pull-down and the pull-up (used once each)

**Problem:** When the ESP32 first powers on, its pins are not yet controlled by
your program — they "float" and can be at any voltage. For a couple of hundred
milliseconds, connected things can do random stuff. In our case that means the
relay could click on and the valve could open during every reboot.

**Solution:** A single resistor that gently holds the line at a known state
until the ESP32 takes charge.

- **Pull-down:** one 10 kΩ resistor from the signal wire to GND. Holds it low.
- **Pull-up:** one 10 kΩ resistor from the signal wire to 3.3 V. Holds it high.

The resistor is deliberately weak, so the moment the ESP32 actually drives the
pin, it easily overrides the resistor. It only wins when nothing else is
driving.

We need a **pull-down on GPIO4** (so the MAX485 stays in listening mode at boot)
and a **pull-up on GPIO25** (so the relay stays off at boot, because that relay
module turns on when the pin goes *low*).

---

## Stage 1 — Power

Do this first with nothing else connected. You are just building the two voltage
rails everything else will plug into.

1. Wire the 12 V supply's barrel jack adapter to your breadboard's power rails.
   Positive to the red rail, negative to the blue rail.
2. Plug it in. Measure across the two rails with your multimeter on DC volts.
   You should read very close to 12 V.
3. Unplug. Connect the buck converter: its `IN+` to the 12 V red rail, `IN-` to
   the blue ground rail.
4. Plug in. Measure the buck converter's `OUT+` to `OUT-`. It will be some
   random voltage.
5. Turn the small brass screw on the buck converter's blue potentiometer slowly
   while watching the multimeter, until it reads **5.0 V**. It may take many
   turns; these are multi-turn adjusters.
6. Unplug. Wire `OUT+` to the second red rail on the other side of your
   breadboard, and `OUT-` to the blue ground rail.

> **Critical:** both blue ground rails must be joined with a jumper wire. The
> 12 V ground and the 5 V ground are the *same* ground. This is the rule from
> the top of this document, in practice.

**Test:** Plug in. You should measure 12 V on one red rail, 5.0 V on the other,
both referenced to the shared blue rail. If yes, unplug and continue.

---

## Stage 2 — The ESP32 on its own

Don't connect it to the power rails yet. Just plug it into your computer with a
micro-USB cable.

Flash it with a minimal ESPHome config — nothing but wifi, api, and logger. This
board has a proper USB-to-serial chip on it, so the logger works over UART0
with no special settings, and that leaves UART2 completely free for the
Modbus bus later. Confirm the device appears in ESPHome and you can see log
output.

**Why this stage exists:** you now know the board works and you can see logs.
Everything after this is diagnosable. Skip this and any later problem could be
the board, the wiring, or the config, and you won't know which.

Leave it running on USB power for now. We add the 5 V rail at the very end.

---

## Stage 3 — The MAX485 and one probe

This is the fiddliest stage. Take it slowly.

### First, check the module

With nothing connected, measure the resistance across the green screw terminal's
`A` and `B` positions. If you read around **120 Ω**, the board already has a
terminating resistor fitted — good, you need nothing extra. If you read a very
high number or nothing, note it down; we may add one later.

### Wire the module

| MAX485 pin | Goes to |
|---|---|
| `VCC` | 5 V red rail |
| `GND` | blue ground rail |
| `DI` | ESP32 GPIO17 — direct wire, no resistors |
| `RO` | **via voltage divider** → ESP32 GPIO16 |
| `RE` | joined to `DE` |
| `DE` | joined to `RE`, then to GPIO4, plus 10 kΩ pull-down to GND |

### The RO pull-up — build this before the divider

`RO` needs one more component beyond the divider below, and it goes first:
a **4.7 kΩ resistor from `RO` straight to the 5 V rail**, not to the
divider's junction.

Here is why. While the module is transmitting, `RE` is held high and the
chip's `RO` output goes high-impedance — it stops driving the line at all.
With nothing but the divider connected, the 20 kΩ leg pulls that floating
line down to ground. A line held low is read by the ESP32 as a UART break,
which shows up in the logs as a run of stray `0x00` bytes exactly when the
module switches from sending to listening. This is a real fault that cost
an entire evening on this project before being tracked down — every
symptom of it looked like corruption on the bus, and nothing about the bus
itself was ever wrong.

The pull-up holds `RO` at a defined high level whenever the chip isn't
driving it, so there's nothing left to read as a break. Build it first,
as its own step:

```
5V ──[4.7k]──── RO
```

One leg into the same breadboard row as the `RO` wire, the other leg into
the 5 V rail. Nothing else shares that row except the top of the divider's
10 kΩ, described next.

### The divider — and the one thing that goes wrong every time

Build the divider on `RO` exactly as described earlier: 10 kΩ from `RO` to a
junction, 20 kΩ from that junction to ground, jumper from the junction to
GPIO16.

**The gap is not optional and it is not obvious.** A breadboard's centre
gap splits every column into two separate halves with no connection
between them whatsoever, except through a component leg that physically
spans it. The `RO` wire and the pull-up's near leg live on one side of
that gap. GPIO16 has to live on the *other* side, reached only by walking
through the 10 kΩ and then the 20 kΩ.

If GPIO16's wire ends up on the same side as `RO` — even one row off from
where it should be — it reads the raw `RO` voltage directly, undivided,
which on a 5 V line is enough to be right at the edge of what the ESP32's
input can tolerate. This exact mistake happened more than once during
this build and is the single most likely thing to go wrong when wiring a
second probe later. When you build it: place the 10 kΩ so one leg is
clearly on the near side of the gap and the other leg is clearly on the
far side, and only then run GPIO16's wire from a row on that far side.

A correctly built network rests at **5 × 20/(10+20) = 3.33 V** on the
GPIO16 side when idle — not 2.9 V, a figure that got used verbally during
this project's debugging and is simply wrong arithmetic. 3.33 V is what
to expect from a meter, and it's safely within range for the pin.

For `RE` and `DE`: push a short jumper between the two pins so they are
electrically one point. Run a wire from there to GPIO4. Add one 10 kΩ resistor
from that same point down to the ground rail.

### Wire ONE probe

Only one, until you know its address. **Do not assume a fresh probe is on
address 1.** Both probes on this project were found already sitting on
address 2, which is not what any seller listing or earlier revision of
this document said. Two devices on the same address jam each other and
produce a bus that looks completely dead.

**Addresses on this build, settled and verified: 1 = upper chamber
probe, 2 = lower chamber probe.** Both confirmed reading real depth in
water. Label the physical units.

If you ever need to renumber one, do it with that probe alone on the
bus, and note that the two commands go to *different* addresses: the
address write goes to the probe's current address, and the save
(`0x06` to register `0x000F`, value 0) goes to its **new** address.
Sending the save to the old address loses the change at the next power
cycle. Power cycle and confirm the new address answers while the old
one times out before believing it worked.

**Wire colours confirmed from the actual label on this probe** — don't assume
the seller's generic listing colours, check your own unit's laser-etched
label:

| Probe wire | Goes to |
|---|---|
| Red (V+) | 24 V red rail |
| Green (V−) | Common ground — same ground as everything else |
| Blue (A) | MAX485 screw terminal `A` |
| Yellow (B) | MAX485 screw terminal `B` |

If the cable has a bare shield wire, connect it to the ground rail at this end
only. Leave the other end unconnected.

> **Confirmed: this probe needs 24 V, not 12 V.** The label states it plainly
> — 12 V was not enough to get a reliable signal on the bus in testing. Wire
> a genuine 24 V supply from the start rather than trying 12 V first.

### Add RS-485 bias resistors

Beyond the 100 kΩ terminator across `A`/`B` from earlier, the bus also needs
two bias resistors so the line has a defined rest state when nothing is
transmitting — without them, the receiver can pick up stray noise as false
data right at the moment the module switches from sending to listening.

| Resistor | From | To |
|---|---|---|
| 560 Ω–1 kΩ | `A` terminal | 5 V rail |
| 560 Ω–1 kΩ | `B` terminal | Ground rail |

Use resistors genuinely in the 560 Ω–1 kΩ range. Far higher values (tens of
kΩ) don't pull enough current to actually bias the line and won't do the job
even though they're "connected."

### Test

Add the `uart`, `modbus` and `modbus_controller` sections to your ESPHome config
and watch the logs. Success looks like a plausible number that changes when you
lower the probe into a bucket of water.

**If it doesn't work,** the usual suspects in order: `A` and `B` swapped (harmless,
just swap them back), a missing ground connection somewhere, or the wrong
register address.

---

## Stage 4 — The flow sensor

Simple after stage 3.

| Sensor wire | Goes to |
|---|---|
| Red | 5 V red rail |
| Black | blue ground rail |
| Yellow | **via voltage divider** → ESP32 GPIO27 |

Second divider, same as the first: 10 kΩ from yellow to a junction, 20 kΩ from
junction to ground, jumper from junction to GPIO27.

**Test:** add a `pulse_meter` sensor in ESPHome. Blow through the sensor, or
gently spin the internal rotor with a cocktail stick. The pulse count in the
logs should climb.

---

## Stage 5 — The relay and valve, dry

Do this with the valve sitting on the bench, not connected to any pipe. You want
to hear it turn and see which way it goes before it is anywhere near water.

### Identify the valve's three wires first

Do this before wiring anything permanently. Connect the valve's presumed common
wire to 12 V negative, then briefly touch each of the other two wires to 12 V
positive in turn. One will drive it open, the other closed. Watch the slot on
the shaft, or listen — it takes about five seconds and stops on its own.

Write down which colour does what. The listing says red/yellow/blue with yellow
as common, but sellers swap these constantly.

### Wire the relay

| Connection | Goes to |
|---|---|
| Relay `VCC` | 5 V red rail |
| Relay `GND` | blue ground rail |
| Relay `IN1` | ESP32 GPIO25, plus 10 kΩ pull-up to the ESP32's 3.3 V pin |
| Relay channel 1 `COM` | 12 V red rail |
| Relay channel 1 `NO` | valve's **open** wire |
| Relay channel 1 `NC` | valve's **close** wire |
| Valve's **common** wire | blue ground rail |

`COM`, `NO` and `NC` are the three screw terminals on the relay's output side.
`NO` means "normally open" and `NC` means "normally closed" — those describe
what the relay does when it is switched *off*.

**Why this arrangement matters:** when the relay is off, `COM` connects to `NC`,
which sends 12 V down the close wire. So the resting, unpowered, ESP-has-crashed,
WiFi-is-down state is *valve closed*. That safety behaviour comes from the wiring
itself, not from software. It is the single most important decision in this build.

### Test

In ESPHome, make a switch on GPIO25 with `inverted: true` and
`restore_mode: ALWAYS_OFF`. Toggle it and watch the valve turn. Then reboot the
ESP while watching — the valve should stay closed throughout, with no twitch.

---

## Stage 6 — Bring it together

1. Add the second probe: blue to `A`, yellow to `B`, red to 24 V, green to
   ground — in parallel with the first. Check this probe's own label before
   assuming it matches the first one's colours. Both probes must be on
   different Modbus addresses first — on this build, 1 (upper) and 2
   (lower), already done and verified.
2. Switch the ESP32 from USB power to the 5 V rail. **Unplug the USB cable
   first.** Then connect the board's `5V` (sometimes labelled `VIN`) pin to the
   5 V red rail and one of its `GND` pins to the blue rail.

> **Never have USB and the 5 V rail connected at the same time.** On this board
> that pin sits directly on the USB power line with nothing in between, so you
> would be shorting two power supplies together.

---

## Quick reference: the complete pin map

| ESP32 pin | Connects to | Extra components |
|---|---|---|
| GPIO17 | MAX485 `DI` | none |
| GPIO16 | MAX485 `RO` | 4.7 kΩ pull-up to 5V, **then** 10 kΩ / 20 kΩ divider — pull-up and GPIO16 are on opposite sides of the breadboard's centre gap |
| GPIO4 | MAX485 `RE` + `DE` | 10 kΩ pull-down to GND |
| GPIO27 | Flow sensor yellow | 10 kΩ / 20 kΩ divider |
| GPIO25 | Relay `IN1` | 10 kΩ pull-up to 3.3 V |
| `5V` | 5 V rail | not while USB is plugged in |
| `GND` | ground rail | |

---

## Things that will go wrong, and what they mean

| Symptom | Most likely cause |
|---|---|
| Nothing works at all | Missing ground connection between two sections |
| Probe gives no response | `A` and `B` swapped, or wrong Modbus address |
| A run of stray `0x00` bytes at the start of every reply | Missing pull-up on `RO` — see "The RO pull-up" above |
| GPIO16 reads close to 5 V instead of ~3.33 V | GPIO16's wire is on the same side of the breadboard's centre gap as `RO`, reading the undivided voltage — see "The divider" above |
| Probe worked, then two probes broke it | Both still on the same address |
| Perfect outgoing frames, nothing ever received | Nothing is listening on the address being polled. Scan addresses before reaching for the multimeter |
| Flow count stays at zero | Divider built wrong — measure the junction, should be ~3.3 V when idle |
| Relay clicks on when ESP reboots | Pull-up resistor on GPIO25 missing or on the wrong rail |
| Readings drift downwards over days | Water got into the probe's breather vent — keep that cable end dry |
| ESP32 keeps rebooting | 5 V supply can't deliver enough current, or USB and 5 V rail both connected |

---

## What still needs confirming

- **Probe supply voltage — resolved.** Confirmed 24 V DC from the label on
  the physical unit. Wiring throughout this document now assumes 24 V.
- **Probe 1 Modbus communication — resolved.** Confirmed working end to
  end: register 2 reads 17 (cm), register 3 reads 1 decimal place, and
  the measurement register tracks real depth. The probe accepts standard
  Modbus function 0x03 without issue — an earlier exception reply during
  debugging turned out to be caused by the missing `RO` pull-up
  corrupting the request, not a genuine rejection of the function code.
- **Second probe's Modbus address — resolved.** Both probes were found
  already on address 2, not address 1 as previously stated here. One was
  renumbered to 1 and saved, verified across a power cycle and in water.
  Addresses are now 1 (upper) and 2 (lower), both reading correctly on
  the shared bus.
- **Relay contact rating for 24 V DC** — pending. If the whole build moves
  to a single 24V rail (see the power note in the parts list), the relay
  module's contacts need to be rated for switching 24V DC specifically, not
  just AC. Being checked separately; until confirmed, the valve stays on
  its own 12V rail as documented in stage 5.
- **Valve wire colours.** Identify them on the bench as described in stage 5.
- **Valve material.** The original CR02 valve is brass with no lead-free or
  DZR claim on its listing — the same gap exists on the deferred backup
  solenoid. A **stainless steel** electric ball valve (CR02 wiring, DN15,
  12V DC) sidesteps this entirely and is a common, easy-to-find part, not a
  special order. Pending spec confirmation before it replaces the CR02 as
  the primary valve — everything in this document (wiring, relay logic,
  fail-closed behaviour) applies identically either way, since only the
  body material changes, not the wiring.

---

## Deferred for later — not part of this build

**A float switch as an independent backstop.** Two Modbus probes give you two
readings, but both travel the same wire through the same code. A float switch
is a magnet moving past a reed contact when water physically lifts it — no
software between the water and the trip, so it fails independently of a bug,
a bus glitch, or a bad register read that could affect both probes at once.
With the batch-fill strategy limiting how long any single fill runs, and a
simple cross-check between the two probes' readings catching a good deal of
the same territory in software, this is being left out of the first build
rather than treated as mandatory. Adding it later means: one wire to a GPIO
with the internal pull-up enabled, one wire to ground, mounted in the upper
chamber below the maximum fill line.

**A second, independent shutoff valve as a hardware interlock.** Even with a
float switch reinstated, both it and the probes would still close the same
valve through the same relay. If that valve's motor is ever physically jammed
open, nothing electrical can free it. Closing that gap means a second small
solenoid valve in series with the primary valve, wired so a float switch's
contact powers it directly with no relay or ESP32 involved. Given the lead
concern below, this should be a **stainless steel** solenoid, not brass —
these are a common, easy-to-find category, not a special order.

**A second, independent MAX485 bus.** Right now both probes share one
transceiver and one pair of A/B wires — a broken solder joint, a dead MAX485
chip, or a shorted bus wire takes out both readings at once. Giving each probe
its own adapter on its own UART removes that shared point of failure, using
hardware already on hand. If added:

| Function | Pin |
|---|---|
| Second MAX485 `DI` | GPIO32 |
| Second MAX485 `RO` (via divider) | GPIO35 |
| Second MAX485 `RE`+`DE` (pull-down) | GPIO33 |

Wired and built exactly like the first MAX485 module in stage 3, just on its
own UART with its own probe. This is a straightforward addition later — it
doesn't touch anything else in the build — so there's no rush to do it now.

Neither is a blocker for getting the core system running. All three are worth
revisiting once it has been in service for a while.
