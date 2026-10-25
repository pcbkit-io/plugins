---
name: pcbkit-boards
description: Design, change, check, route, share and export printed circuit boards (PCBs) with PCBKit. Use when the person asks for a circuit board, a PCB, a blinking LED, a breakout or carrier board, or to change, move, label, route, check, share or export one for fabrication or KiCad; also firmware, sensor, battery and circuit simulation supported by PCBKit.
---

# Circuit boards with PCBKit

PCBKit's tools write a board in PCBKit (a small board language with units), check it, place and
route it, and export it. The person sees the board beside the conversation (in a terminal or an
IDE: on its live page, a link `board.open` gives) and watches it change.

## Devices that run

A device with a program (a game, a timer, a sensor display, a controller) is one board that runs
in the simulator and can be ordered and flashed as it is. Read `help { topic: "simulation" }`
first. Choose the processor, display, controls and power from `lab.catalog`'s orderable `parts`
and say why in a line each (what the program needs, what the person asked, how it is flashed:
an Arduino Nano's USB, an RP2040/RP2350's BOOTSEL, an STM32's ST-Link or DFU, an ESP32's
USB-UART); STM32 programs use ST's HAL, ESP32 programs ESP-IDF's low-level drivers, and an ESP32's
Wi-Fi and Bluetooth do not run here (say so); a battery device needs its cell on a
connector, a charger from USB, a switch and the processor's supply. Design and route it until
`check` is complete, write the program into the folder the processor names (`files.write`), build
it with `lab.run` (require `ok:true`, read the diagnostics), look at its screen (`lab.frame`),
press its buttons (`lab.control`, `lab.advance`) and report what the snapshots show — never what
the code should do. Without the board open, `lab.run` builds on PCBKit's servers and saves the
image; then `lab.simulate { steps }` runs it with the inputs you set and returns what the pins,
serial port and displays did: report those. A processor `lab.catalog` does not list cannot run
here: say so, still write its program, and offer a listed one or a bench test. Finish as for any board: `parts.select` until every part has an orderable
part, `export.run { format: "firmware" }` and the fabrication files or `fab.order`, and tell the
person in plain steps how to order it, charge it and flash the ordered board. `lab.simulate`
runs a reproducible experiment without a page; live `lab.*` commands act on the board the person
has open. Never claim a failed build ran or invent readings.

## The loop

0. **Build, or ask once.** When the person says what to build, build it now: choose defaults for
   what they left out and say them in the answer. Ask first only when the device itself is
   undecided (a "motor controller" with no motor named, a "gadget" with no idea): one short
   message with the two or three questions that change the design, each with the default you
   would take. Once they answer or describe the device, build it; a choice still open is a
   default you state, never another question before the board. A part that misses one figure
   (a controller rated 30 W for a 20 W board, a variant to confirm) is the closest one, built in
   and named in the answer; a chip the registry lacks is read from its datasheet in the same turn.
1. **First design at once.** Choose sensible defaults for what the person did not say (power, size)
   and write the whole first design in one `files.write { path: ".pcbkit/design.pcbkit", text, build: true }`:
   blocks and parts joined by nets, no coordinates; `build: true` places and routes it in the same
   call. `help { topic: "quickstart" }` has a complete first board and lists every block and the
   registry's parts: describe all you use in one call (`lang.describe { ids: [...] }`,
   `parts.use { ids: [...] }`, whose `design` is the file's text: no need to read it). Leave the outline to the engine
   (it sizes it around the parts and rounds its corners) and give the board
   `holes = MountingHoles() {}` (a plated M3 hole in each corner) unless the person says
   otherwise, it plugs into something (pin headers, a USB stick), it is carried or worn (a
   business card, a badge: a lanyard hole at most) or it is tiny (its parts fit in
   about 15 × 15 mm, where four M3 holes would double it; `MountingHoles(screw = "M2")` if it
   still needs them). A blinker that "blinks every 2 seconds" has a 2 s period: on about 1 s, off
   about 1 s.
2. **Open it:** `board.open` once per conversation, right after the first design. The board opens
   for the person and follows every later change by itself, theirs included. Never call it again:
   each call adds another copy of the board to the conversation. Where the host draws no board (a
   terminal or an IDE: Codex, Claude Code), its answer is the board's live page: give the person
   that link in your answer, once; it shows everything you do from then on.
3. **Fix:** the build's answer spells out the first errors and what to do about each; fix them
   with `files.edit { …, build: true }` (change and build in one call; `check.run` only when it
   says there are more). A build completes the board: the copper that still matches the design
   stays and only what is unrouted or broken by the change is routed (copper the tools made
   themselves may be redone to fix it; a person's, drawn or changed in KiCad or the app, never). When a
   connection does not fit, ask the person before `build.run { make_room: true }` (routes the
   copper close around it again) or `build.run { all: true }` (places and routes everything
   anew, no `route.clear` first): both change routing they may have finished. The first time the board checks complete, `view.tour`: a
   short camera tour for the person (it plays once and comes back to the whole board).
4. **Report in a sentence or two**, in plain words, by the board's name, never with ids, with the
   numbers that matter (a blinker's on and off times, the board's size); look at the board first
   when parts moved (`board.snapshot {}`: a picture you see, once per turn), so what you say
   about where things are is what you saw; then offer
   two or three next steps that fit (`help { topic: "guide" }` has the ladder: make it theirs,
   move things, add features, make it sturdy, grow it, order it).
5. **Each further request is one small change** (`files.edit`, `board.move`, … with `build: true`,
   which routes again in the same call; else `build.run`), and the same short report. The person's view goes to the parts you changed by
   itself; for anything else (a new text, a net) `view.camera { ids: [...] }`. What the person asks
   to see (2D or 3D, a layer, the code) the `view.*` tools do on their open board.
   A wish about where a part goes is a rule, not coordinates: "in the middle" `place =
   [area(.center)]`, "top right" `[area(.topRight)]` (a corner holds a mounting hole: use `area`),
   "on the left edge" `[edge(.left)]`; it stays true when the board changes. Parts a person
   sees or touches as a set (a light's LEDs, keys, a display's digits, test points) get an
   arrangement: `row([...])`, `column([...])` or `grid([...], columns = 3, pitch = 8mm)`. Blocks
   and part modules already hold their capacitors and pull-ups at the pins they serve (as
   preferences): keep their rules as they come; one you add goes at its pin the same way,
   `place = [soft(near(U1.pin.AVDD, max = 2mm), weight = 2)]`. A rule without `soft` is a
   requirement: each one the engine cannot keep is an error on the board, so require only what
   must hold (a connector's edge, a sensor away from heat) and leave the rest of the layout to
   the engine; no `inside(box(…))` per part on a board of a given size. Give `at`
   only to what the person fixes mechanically (a sensor's position, a connector, holes). The
   router stitches a net's pours on both layers with vias by itself, and references with no
   room on the silkscreen are left out by themselves: neither needs a workaround.
6. **Review before "ready", compute nothing in your head.** Before you call a board ready or
   export it: `analysis.run { lens: "ready" }`, the design review (connectors on edges, decoupling, LED currents,
   programming access, protection, sensors near heat, antennas, holes, silkscreen); fix its
   `error` findings (each has its `fix`), mention the `warning` ones that are the person's choice.
   Every number in a design or an answer (an LED's resistor, a 555's timing, battery life, a
   divider, a current-sense scale, a track's width) comes from `calc.run` (`kind`: `led`, `divider`, `astable555`, …, with its `params`: `help { command: "calc.run" }`), never from your head.
   An antenna feed (50 Ω) or a USB / Ethernet pair (90 / 100 Ω) states its impedance and the
   router sizes it from the board's stack: `net ANT { impedance = 50ohm }`, `pair ETH_TX = (TXP,
   TXN) { impedance = 100ohm }` (`calc.run { kind: "impedance" }` shows the numbers); on two layers say that the line is wide.
   For heat or simulation, record the person's operating conditions in the board: an external
   load belongs on its supply net (`net V3 { voltage = 3.3V; current = 100mA }` for that stated
   load). A maximum current rating is not an operating load. Inspect `analysis.inputs { parts: true }`, then
   `analysis.run { lens: "heat" }`; if sources or loads are missing, supply the conditions the
   person already gave and run it again. Without them, temperatures remain unknown. Use the
   returned temperatures and assumptions; convergence does not prove physical accuracy. A
   JEDEC junction-to-ambient resistance is not the custom board's resistance, and there is no
   universal temperature offset for a closed box. The result's junction temperature includes
   the board temperature plus the package rise; a measured case temperature is a different
   quantity and cannot be compared directly to a junction limit. Explain missing data without declaring the
   board safe. `analysis.circuit` (dc, ac or transient) requires explicit device
   models; they do not infer a whole-chip model from a part number. `mosfetChannel` is a
   long-channel equation (with an optional body effect) without intrinsic gate charge. Do not assign
   it to a named transistor from its ratings or use it alone to claim real switching speed/loss.
   For a passive circuit, `analysis.netlist` assembles the board's typed nominal R/C/L values
   and explicit named-net sources; pass its returned `circuit` to the chosen solver. If it is
   null, resolve the missing models or conditions. Do not silently exclude unsupported parts.
   A requested component selection is a partial circuit: explain the excluded boundary loads
   and do not present its answer as a simulation of the whole board.
   Read AC answers from `samples`; use `observe` with the wanted voltage node or current device
   (at most eight). Large raw arrays live in the artifact URL, not in the editable board's
   files. Do not replace unread solver numbers with hand calculations. Do not volunteer an
   RC corner or thermal margin before a calculation supports it.
   Defaults, including 25 °C ambient, are assumptions, not conditions supplied by the person.
   Do not derive an enclosure's allowable ambient by subtracting the board or chip temperature
   from a limit: cooling inside the enclosure has not been modelled. Recommend checking the
   actual assembly under its stated load. A lower controlled supply or a suitable switching
   regulator can reduce LDO loss; a series input resistor is not a substitute for a regulated
   input and needs its own load-range/dropout analysis before it can be recommended.
   A physical `analysis.dcir` or `analysis.electrothermal` calculation returns compact diagnostics
   and `artifact.url` for its saved fields. Show them with `view.analysis { url, layer, metric }`
   using that URL; use the page-local job id it returns for subsequent
   `view.analysis { id, layer, metric }` changes. The compute call's job id belongs to another process. A saved
   field is valid only for its exact board geometry; opening it does not rerun the solver.
   For anything the lenses do not answer, run an analysis script (`analysis.script`, its API in
   `help { topic: "analysis" }`): it reads the board, calls the solvers and reports a lens shown on
   the board. A template answers the usual questions — `impedance` (are the traces their target
   impedance), `eye` (does a link work at a bit rate; `equalize: "auto"` trains FFE/CTLE/DFE),
   `reflections` (ringing, a series resistor; an IBIS model when the person has one), `crosstalk`,
   `returnpath` (gaps under signals, unstitched layer changes), `emissions`, `pdn` (rail impedance
   against its target), `planes` (plane resonances), `load` (a rail's dip when a load's current
   jumps), `converter` (a buck's ripple, peak against the inductor's saturation, efficiency, loop
   and compensation), `heatup`, `response` (a filter's Bode plot), `tolerance` (Monte Carlo yield).
   Each declares its inputs (the answer's `controls`, with their values): run it again with
   `inputs` for the person's conditions — `{ template: "load", inputs: { rail: "+3V3", load: {
   shape: "pulse", base: 0.05, peak: 0.4, width: 1e-3, period: 4e-3 } } }` for a radio's bursts.
   Write your own script for anything else (a sweep, a custom plot), keep a board's own checks as
   `.pcbkit/analysis/<name>.js`, and quote what the script reports: its numbers with their
   conditions, where on the board (coordinates and the nearest part), the component values it
   read or designed (compensation, taps), and its assumptions.
7. **Export only when asked:** fabrication or KiCad files (`export.run { format: "fab" }` or
   `{ format: "kicad" }`) when the
   person asks for them or orders, not after a change; the board beside the conversation is enough.
   **To order** ("order it", "where can I make it", "how much", "10 assembled"): when the person
   asks to order, hand the board over: PCBKit never places an order or takes payment, the fab's page
   only shows the board, so a missing part number or a warning is something to say, not a reason
   to refuse. Do not hand over a board whose check is not `complete` (say why and what fixes it).
   `fab.shops` lists
   the fabs with an estimate where one publishes prices, what each cannot make of this design
   (`limits`), which of the board's fabrication options (colours, finish, thickness, …) its page
   gets chosen already (`filled`) and which the person chooses there by hand (`manual`). Set the
   options the person wants with `fab.options` first. When the person picks a fab,
   `fab.order { shop }` puts the board on that fab's order page and gives its `url`: hand it to the
   person, who checks the options, pays and orders there (PCBKit sells nothing). A fab with
   `how: page` gives its quote page and the archive to upload. **Assembled (PCBA):**
   `fab.order { shop, assembly: true }` where the fab's `assembly` is set: `upload` (PCBWay,
   AISLER) sends the BOM and positions in the fab's own format with the board and the fab buys
   the parts by MPN, so first fill the MPNs (`parts.select`; `fab.shops`' `assembly.without_mpn`
   names the parts without one); `page` (JLCPCB, NextPCB) gives the BOM and placement list to
   upload there. Parts still without an MPN go to the fab without one: name them in the answer
   (the fab's page asks for them, or `parts.select` fills them first if the person wants). The person's own part numbers go in as fields (`fields { "MPN": "…", "LCSC":
   "C25744" }`). Name the bottom-side and through-hole parts: they cost more; say what to check in
   the fab's placement preview (pin 1 of each IC, parts on the bottom). Files only ("give me the
   Gerbers"): `export.run { format: "fab" }` (`force: true` for a board not complete, said plainly).
8. **Share when asked** ("send it to a friend", "a link"): `share.link`, then give the person its
   `url`: anyone opens it without an account, read-only, with the board's AI tour first, and it
   follows every later change; a friend's edits go to their own copy. `share.stop` closes it.
9. **Text on the board** goes on the top silkscreen (`Silk.top`) unless the person asks for the
   back, where nothing else is printed; `check.run` warns (`silk-overlap`) when it is not. A
   label of a part (a connector's `IN 12V`, `+`/`-`, a switch's `ON`) is `Text { text = "IN 12V";
   near = J1 }` without `at`: it stays beside the part wherever the part goes.

## Read what the step needs

- How to lead the person, the ladder, where parts come from: `help { topic: "guide" }`.
- The board format, one section at a time: `help { topic: "format", section: "Silkscreen" }`
  (also `Board`, `Fabrication`, `Placement rules`, `Parts and components`, `Routing`, `Errors`,
  `Recipes`).
- The language: `help { topic: "language", section }`; the standard library: `lang.search`,
  `lang.describe { ids }`.
- Parts: `parts.search` (verified pin tables), else the datasheet: `parts.datasheet { id }` reads
  the maker's PDF (pin table, package drawing, application circuit) — then `parts.sheet`,
  `parts.create { dry_run: true }`, `parts.create`. A part missing from the registry is one of
  those calls, not a reason to stop building or to ask the person for the PDF (ask only when the
  tool cannot read it). Never invent pin numbers.
- A chip comes with its datasheet's support circuit: `parts.use { ids: [id] }` adds `U1 = <id>.Part {}`
  and `U1Support = <id>.Support(U1) {}` (decoupling at every supply pin, bulk capacitors,
  pull-ups, the crystal with its load capacitors, reset/boot buttons, straps, an RP2040's flash).
  Connect the chip's pins in that line, pass the inputs it names (a regulator's `vout`, USB
  `dp`/`dn`), keep both lines, and do not add those parts again by hand.

## One board per conversation, and boards that work together

Every call works on this conversation's board, kept on the server (the first change makes it), so
keep improving it rather than starting over; leave `project` out. Only where answers end with
`Project: p…` (hosts without conversations) pass that id as `project` in every later call about
the same board. If the person wants to keep their boards, they can sign in.

- **Several boards that work together** (a transmitter and its receiver for a radio-controlled
  car, a remote and what it drives, a main board and its daughter board) are **one design with
  one piece of the outline per board**: `outline = [rect(…), translated(rect(…), (…))]`, each
  piece sized by you, 5 mm or more apart; say which parts go on which piece with a rule bound in
  the design (`receiver = inside(board.outline[1], parts = [J5, rxPower])`). Each piece is
  connected by itself, so both may use `GND`; the person sees both beside the conversation and
  orders them together (the fabrication files also hold each piece alone).
  `help { topic: "format", section: "Several boards" }` has a whole example.
- **A board that has nothing to do with the current one:** `project.new { name }` starts it in
  the same conversation; the calls after it go to the new board and the person's view follows.
  `project.list` lists the conversation's boards; `project.open { id }` goes back to one.
- To work on another board the person names, `project.open { id }` with its id from `project.list`.

## Be honest about the board

The check is not a review (`analysis.run { lens: "ready" }` is): name connections that did not route and why
(`route.diagnose`), never call a board ready when the check is not `complete`,
and ask the person to review the design before ordering.
