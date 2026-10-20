---
name: pcbkit-boards
description: Design, change, check, route, share and export printed circuit boards (PCBs) with PCBKit. Use when the person asks for a circuit board, a PCB, a blinking LED, a breakout or carrier board, or to change, move, label, route, check, share or export one for fabrication or KiCad; also firmware, sensor, battery and circuit simulation supported by PCBKit.
---

# Circuit boards with PCBKit

PCBKit's tools write a board in PCBKit (a small board language with units), check it, place and
route it, and export it. The person sees the board beside the conversation and watches it change.

## Simulation requests

For firmware, sensors, a battery-powered instrument, CO₂ indicators, or physical circuit/board simulation, read `help { topic: "simulation" }` first. Follow its tool-only workflow and inspect the actual results. Author C through `files.write` / `lab.run { source }`; require the compiler result `ok:true` and fault-free engine snapshots. `lab.simulate` executes a complete reproducible experiment without a live page. Live `lab.*` actions reach the open board page through MCP; the compiler must be configured there. Never infer executable device support from the parts catalog, claim a failed build ran, or replace unread simulation results with invented values. The illustrative laboratory is a functional POC and is not a fabrication-ready board. For a simulation request, start with that workflow instead of routing a new board.

## The loop

1. **First design at once.** Choose sensible defaults for what the person did not say (power, size)
   and write the whole first design in one `files.write { path: "design.pcbkit", text, build: true }`:
   blocks and parts joined by nets, no coordinates; `build: true` places and routes it in the same
   call. `help { topic: "quickstart" }` has a complete first board and lists every block and the
   registry's parts: describe all you use in one call (`blocks.describe { names: [...] }`,
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
   each call adds another copy of the board to the conversation.
3. **Fix:** the build's answer spells out the first errors and what to do about each; fix them
   with `files.edit { …, build: true }` (change and build in one call; `report.get` only when it
   says there are more). `build.run { all: true }` places and routes everything anew, no
   `route.clear` first. The first time the board checks complete, `view.tour`: a
   short camera tour for the person (it plays once and comes back to the whole board).
4. **Report in a sentence or two**, in plain words, by the board's name, never with ids, with the
   numbers that matter (a blinker's on and off times, the board's size); look at the board first
   when parts moved (`board.snapshot {}`: a picture you see, once per turn), so what you say
   about where things are is what you saw; then offer
   two or three next steps that fit (`help { topic: "guide" }` has the ladder: make it theirs,
   move things, add features, make it sturdy, grow it, order it).
5. **Each further request is one small change** (`files.edit`, `board.move`, … with `build: true`,
   which routes again in the same call; else `route.run`), and the same short report. The person's view goes to the parts you changed by
   itself; for anything else (a new text, a net) `view.focus { ids: [...] }`. What the person asks
   to see (2D or 3D, a layer, the code) the `view.*` tools do on their open board.
   A wish about where a part goes is a rule, not coordinates: "in the middle" `place =
   [area(.centre)]`, "top right" `[area(.topRight)]` (a corner holds a mounting hole: use `area`),
   "on the left edge" `[edge(.left)]`; it stays true when the board changes.
6. **Review before "ready", compute nothing in your head.** Before you call a board ready or
   export it: `analysis.run { lens: "ready" }`, the design review (connectors on edges, decoupling, LED currents,
   programming access, protection, sensors near heat, antennas, holes, silkscreen); fix its
   `error` findings (each has its `fix`), mention the `warning` ones that are the person's choice.
   Every number in a design or an answer (an LED's resistor, a 555's timing, battery life, a
   divider, a current-sense scale, a track's width) comes from the `calc.*` tools (`calc.led`, `calc.divider`, `calc.astable555`, …), never from your head.
   For heat or simulation, record the person's operating conditions in the board: an external
   load belongs on its supply net (`net V3 { voltage = 3.3V; current = 100mA }` for that stated
   load). A maximum current rating is not an operating load. Inspect `analysis.models`, then
   `analysis.run { lens: "heat" }`; if sources or loads are missing, supply the conditions the
   person already gave and run it again. Without them, temperatures remain unknown. Use the
   returned temperatures and assumptions; convergence does not prove physical accuracy. A
   JEDEC junction-to-ambient resistance is not the custom board's resistance, and there is no
   universal temperature offset for a closed box. The result's junction temperature includes
   the board temperature plus the package rise; a measured case temperature is a different
   quantity and cannot be compared directly to a junction limit. Explain missing data without declaring the
   board safe. `analysis.circuit`, `analysis.ac` and `analysis.transient` require explicit device
   models; they do not infer a whole-chip model from a part number. `mosfetChannel` is only a
   long-channel equation without intrinsic charge or body junctions. Do not assign it to a
   named transistor from its ratings or use it alone to claim real switching speed/loss.
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
   and `artifact.url` for its saved fields. Show them with `view.simulation.open` using that URL,
   a copper layer and the desired metric; use its returned page-local job id for subsequent
   `view.simulation` changes. The compute call's job id belongs to another process. A saved
   field is valid only for its exact board geometry; opening it does not rerun the solver.
7. **Export only when asked:** fabrication or KiCad files (`export.fab`, `export.kicad`) when the
   person asks for them or orders, not after a change; the board beside the conversation is enough.
8. **Share when asked** ("send it to a friend", "a link"): `share.link`, then give the person its
   `url`: anyone opens it without an account, read-only, with the board's AI tour first, and it
   follows every later change; a friend's edits go to their own copy. `share.stop` closes it.
9. **Text on the board** goes on the top silkscreen (`Silk.top`) unless the person asks for the
   back, where nothing else is printed; `check.run` warns (`silk-overlap`) when it is not.

## Read what the step needs

- How to lead the person, the ladder, where parts come from: `help { topic: "guide" }`.
- The board format, one section at a time: `help { topic: "format", section: "Silkscreen" }`
  (also `Board`, `Fabrication`, `Placement rules`, `Parts and components`, `Routing`, `Errors`,
  `Recipes`).
- The language: `help { topic: "language", section }`; the standard library: `lang.search`,
  `lang.describe { name }`.
- Parts: `parts.search` (verified pin tables), else the datasheet — ask the person for it or find
  the manufacturer's for the exact part number — into `parts.sheet`, `parts.check`,
  `parts.create`. Never invent pin numbers.
- A chip comes with its datasheet's support circuit: `parts.use { id }` adds `U1 = <id>.Part {}`
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
  `projects.list` lists the conversation's boards; pass one's id as `project` once to go back.
- To work on another board the person names, pass its id from `projects.list` once.

## Be honest about the board

The check is not a review (`analysis.run { lens: "ready" }` is): name connections that did not route and why
(`board.unrouted`, `route.diagnose`), never call a board ready when the check is not `complete`,
and ask the person to review the design before ordering.
