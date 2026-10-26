---
name: pcbkit-boards
description: Design, change, check, route, share, order and export printed circuit boards (PCBs) with PCBKit, and write and simulate their firmware. Use when the person asks for a circuit board, a PCB, a device with a program (a game, a timer, a sensor display), a breakout or carrier board, or to change, move, label, route, check, analyse, share, order or export one for fabrication or KiCad.
---

# Circuit boards with PCBKit

PCBKit's tools write a board in PCBKit (a small board language with units), place and route it,
check it and export it. The person watches the board beside the conversation (in a terminal or an
IDE: on the live page `board.open` links to): every part, track and analysis appears there as you
work. Work so that what they see is a story: a first board within a minute, then one visible step
at a time, each told in a sentence.

## The loop

1. **Build at once, or ask once.** When the person says what to build, build it now: choose
   defaults for what they left out and name them in the answer. Ask first when the device itself
   is undecided or what it connects to is not named ("a motor controller" with no motor, "reads a
   few sensors" with no sensors, no supply for a board on a robot or a car, a device far from
   power and Wi-Fi with no radio, region, range or battery named): one short message with the
   three or four questions that change the design (supply and its noise, the inputs and buses,
   connectors, size and mounting; for a remote one the radio band of the country and the range,
   battery and solar, how often it reports), each with the default you would take, and the
   trade-off that decides most of it in a sentence (a remote one: sleep current against battery
   life). Once they answer,
   build; a choice still open is a default you state. A part that misses one figure is the
   closest one, named in the answer; a chip the registry lacks is read from its datasheet in the
   same turn.
2. **Write the whole first design in one call:** `help { topic: "quickstart" }` (a complete board,
   every block and registry part), then `lang.describe { ids: [...] }` for the blocks and
   `parts.use { ids: [...] }` for the chips you use (one call each; `parts.use` answers the
   design's text: no need to read it), then
   `files.write { path: ".pcbkit/design.pcbkit", text, build: true }`: blocks and parts joined by
   nets, no coordinates. `build: true` places, routes and checks in the same call.
3. **Open it:** `board.open` once per conversation, right after the first design; it follows every
   later change by itself, the person's included. Where the host draws no board (Codex, Claude
   Code), its answer is the live page's link: give it in your answer, once. Never call it again.
4. **Fix** what the build's answer spells out with `files.edit { …, build: true }` (one call
   changes and builds; `check.run` only when it says there are more). A build keeps the copper that
   still matches and routes only what is new or broken. Ask before `build.run { all: true }` or
   `{ make_room: true }`: both redo routing the person may have finished. `route.diagnose { net }`
   says why a connection does not fit and the one thing to do.
5. **Celebrate** the first complete check with `view.tour` (a short camera tour, once per board).
6. **Report in a sentence or two**, in plain words, by the board's name, never with ids: the numbers
   that matter (size, a blinker's on and off times) and what you saw (`board.snapshot {}` once per
   turn after parts moved). Offer two or three next steps (`help { topic: "guide" }` has the
   ladder: make it theirs, move things, add features, make it sturdy, grow it, order it).
7. **Each further request is one small change** with `build: true` (`files.edit`, `board.move`,
   `board.set`, …) and the same short report. The view goes to what you changed by itself; to show
   something else, `view.camera { ids }`; what the person asks to see, the `view.*` tools do.

## A good board by default

- **Compact.** Leave the outline to the engine (`auto()`: the smallest board around the parts,
  corners rounded) and the layout to rules; give `at` only to what the person fixes mechanically.
  A wish about where a part goes is a rule: "in the middle" `place = [area(.center)]`, "top right"
  `[area(.topRight)]`, "on the left edge" `[edge(.left)]`; parts seen or touched as a set get
  `row([...])`, `column([...])` or `grid([...], columns = 3, pitch = 8mm)`. A rule without `soft` is
  a requirement and an error when it cannot hold: require only what must hold (a connector on an
  edge, a sensor away from heat). Blocks and part modules already keep their capacitors at the
  pins they serve; one you add goes the same way,
  `place = [soft(near(U1.pin.AVDD, max = 2mm), weight = 2)]`.
- **Finished like a product.** A new board is matte black with plated edges unless the person
  asks otherwise (the board's `fab { mask = .green; edgePlating = false }`, or `fab.options`):
  the design says nothing about it. Give it `holes = MountingHoles() {}` (a plated M3 hole in each corner)
  unless it plugs into something (pin headers, a USB stick), is worn or hung (a badge, a
  keychain: a lanyard hole) or is tiny (parts within about 15 × 15 mm; `MountingHoles(screw =
  "M2")` if it still needs them); a device that goes into a case (a handheld, a remote) keeps
  them for the case's screws.
- **Power that enters or leaves on a connector** is protected where it comes in (`InputFuse`,
  `ReversePolarity`), has bulk capacitance beside the connector (the review sizes it), states its
  current on the net (`net VOUT { current = 3A }`: the router widens its copper), and shows it is
  on with an LED. An output that must not appear before it is right (a USB-PD trigger before the
  charger agrees on the voltage, a charger's output) goes through a switch the controller's
  power-good drives. A motor driver's supply: reverse polarity by a P-MOSFET (`ReversePolarity`),
  never a series diode (a braking motor pushes energy back into the supply), and its TVS and bulk
  capacitor on the driver's side of it.
- **A temperature or humidity sensor** goes to a far corner from the radio, the processor and the
  regulator (`far(U1, min = 15mm)`) on a slotted island without pour
  (`U3 = thermalIsland(SHT40_AD1B.Part) { … }`); judge its warming with the heat lens at the
  device's real duty cycle (a logger that sleeps: each hot part's average `dissipation`), and
  quote `sensor-warm`'s number.
- **Labels** go on the top silkscreen beside their part: `inLabel = Text { text = "IN 12V"; near = J1 }`.
- **Numbers come from `calc.run`**, never from your head: an LED's resistor, a 555's timing (a
  blinker that "blinks every 2 seconds" is on about 1 s, off about 1 s), a divider, battery life,
  a track's width (`help { command: "calc.run" }` lists each `kind`). An antenna feed or a USB /
  Ethernet pair states its impedance and the router sizes it: `net ANT { impedance = 50ohm }`,
  `pair USB = (DP, DN) { impedance = 90ohm }`; a CAN bus's `CANH`/`CANL` route together by their
  names alone (declare no pair for it: a declared pair is held to USB's skew and coupling).
- **Connectors are what the cable mates with:** a JST-style cable `wireConnector(4)` (PH 2.0 mm;
  `pitch = 2.5mm` XH, `1.25mm` GH, `1mm` SH), pin headers `header(n)`, the registry's parts for
  USB, terminals and the rest; name the series in the answer.
- **Review before "ready":** `analysis.run { lens: "ready" }` (connectors on edges, decoupling,
  LED currents, programming access, protection, holes, silkscreen): fix its `error` findings
  (each has its `fix`), name the `warning` ones that are the person's choice.

## Devices that run

A device with a program is one board that runs in the simulator and is ordered and flashed as it
is. Read `help { topic: "simulation" }` first. Choose the processor, display, controls and power
from `lab.catalog`'s orderable `parts` and say why in a line each (what the program needs, how it
is flashed: an Arduino Nano's USB, an RP2040's BOOTSEL, an STM32's ST-Link, an ESP32's USB-UART;
an ESP32's radio does not run here: say so). A battery device needs its cell on a connector (a protected cell, or protection on the
board: say which), a charger from USB or the panel (its thermistor input wired or set), a
switch, a divider from the cell to an ADC pin so the program knows its charge, and a supply
that holds over the cell's whole range (`Ldo` says when it does not). Its life comes from the board:
`calc.run { kind: "battery", params: { capacity, sleep: true, period, profile } }` adds what every
part draws asleep (sleep and quiescent currents from the sheets, dividers) between the awake
`profile`; name what dominates and what it does not know. Route it until `check` is complete, write
the program into the folder the processor names (`files.write`), build it with `lab.run` (require
`ok: true`), look at its screen (`lab.frame`), press its buttons (`lab.control`, `lab.advance`) and
report what the snapshots show, never what the code should do. Without the board open, `lab.run`
builds on PCBKit's servers and `lab.simulate { steps }` runs it: report what its pins, serial port
and displays did. Never claim a failed build ran or invent readings.

## Analysis

Answer with the tools' numbers, not by eye: `analysis.run { lens }` (`power`, `current`, `heat`,
`ready`), and an analysis script for the rest (`analysis.script { template }`, its API in
`help { topic: "analysis" }`) — "is this bus/pair routed right" `impedance` and `returnpath`; "will
the inputs be noisy" `response` (their filter) and `pdn` or `load` (the supply's ripple and dips);
"is the regulator OK" `converter` or `heatup`; "will it overheat, are the tracks wide enough"
the nets' currents in the design, then `current` (each track's rise for what it carries) and
`heat` or `heatup` with each hot part's dissipation (`calc.run` from its datasheet: a driver's
R_DS(on) losses), and `analysis.dcir` only for a drop the lenses do not give; "will the link work
at this speed" `eye`; "how many boards will pass" `tolerance`. Record the person's conditions in the board first and quote results
with their conditions and assumptions: `help { topic: "simulation", section: "Conditions and what
a result means" }` says what each result does and does not prove. Show a saved field with
`view.analysis`.

## Ordering, files and sharing

- **Order** ("order it", "how much", "10 assembled"): only a board whose check is `complete` (else
  say why and what fixes it). `parts.select` until every part has an orderable part, `fab.shops`
  (estimates where fabs publish prices, how each takes assembly, what each cannot make), show the
  person that comparison in a short table, `fab.options` for their wishes (a cheap order may drop
  plated edges or a black mask: say what it changes), then `fab.order { shop, quantity }` (`assembly: true`
  for assembled boards; a fab that assembles in one click hands over the BOM and positions too)
  and give the person its `url`: they check and pay there; PCBKit sells nothing. Name parts without an MPN, bottom-side and
  through-hole parts, and what to check in the fab's preview (pin 1 of each IC). A device that runs:
  `export.run { format: "firmware" }` too, and plain steps to charge and flash the ordered board.
- **Files only when asked:** `export.run { format: "fab" }` (Gerber, drill, BOM, placement) or
  `{ format: "kicad" }`; give the download links.
- **Share** ("a link for a friend"): `share.link`, give its `url` (read-only, with the board's tour,
  following later changes); `share.stop` closes it.

## Parts

`help { topic: "quickstart" }` lists the blocks and registry parts; `parts.search` finds any other
(verified pin tables, and the open libraries' components with their source and licence). A chip
the registry lacks is read from its datasheet: `parts.datasheet { id }`, then `parts.sheet`,
`parts.create { dry_run: true }`, `parts.create`; never invent pin numbers, and ask the person for
the PDF only when the tool cannot read one. A chip comes with its datasheet's support circuit:
`parts.use` adds `U1 = <id>.Part {}` and `U1Support = <id>.Support(U1) {}`; keep both lines,
connect the chip's pins in the first and pass the inputs it names, never add those parts by hand.

## One board per conversation

Every call works on this conversation's board, kept on the server: keep improving it, leave
`project` out (only where answers end with `Project: p…` pass that id as `project`). Boards that
work together (a transmitter and its receiver, a main board and its daughter board) are one design
with one outline piece per board (`help { topic: "format", section: "Several boards" }`). A board
that has nothing to do with this one: `project.new { name }`; `project.list` and
`project.open { id }` go back. The person can sign in to keep their boards.

## Be honest

The check is not a review: name connections that did not route and why, never call a board ready
when its check is not `complete` or the review has errors left, and ask the person to review the
design before ordering.
