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

## Tools

Seven areas — `help` (learn), `design` (write the source, build, check), `board` (the built board:
read it, change one thing, show it to the person), `parts`, `analysis`, `lab` (firmware),
`project` (export, share, order) — and `board.open`. `design.edit { path, edits }` here means the
tool `design` called with `{ op: "edit", why: "…", path, edits }`. **Every call says `why`**: one
sentence the person reads on their board while it runs, in their terms ("Putting the crystal
beside the MCU to keep its traces short"); what the call names (`id`, `ids`, `net`) lights up
there. A tool lists its ops and their required fields; `help { command: "board.move" }` gives one
op's whole input. `help { topic: "quickstart" }` has the practice every board follows (finish,
power, sensors, numbers, connectors, devices, ordering): read it before the first design.

## The loop

1. **Build at once, or ask once.** Before the first design, sort what the wish leaves open. A gap
   any sensible default fills is yours: build now and name the defaults in the answer. A figure the
   person left out of a part they named (a named motor's current, a sensor's accuracy, a cable's
   length) is such a gap. A wish that says only what a gadget does (a light that comes on in the
   dark, an electronic die, a plant that says when it is thirsty) is built at once: USB power, a
   common sensor, your MCU, each named. A gap that decides the board is the person's: what it
   drives or reads ("a motor controller" with no motor, "a few sensors" with none named), a supply
   it must live on (a robot's or a car's battery and its noise), how it talks and how far (no Wi-Fi
   out there: radio band, range, battery), how many inputs or outputs a tool for their work has
   (keys, channels, relays). A wrong guess there is a board to throw away, so do not build yet:
   answer with one short message, the three or four questions that change the design, each with the
   default you would take, and the trade-off that decides most of it in a sentence (a remote one:
   sleep current against battery life). Once they answer, build; a choice still open is a default
   you state. A part that misses one figure is the closest one, named in the answer; a chip the
   registry lacks is read from its datasheet in the same turn.
2. **Write the whole first design in one call:** `help { topic: "quickstart" }` (a complete board,
   every block and registry part), then `help.describe { ids: [...] }` for the blocks and
   `parts.use { ids: [...] }` for the chips you use (one call each; `parts.use` answers the
   design's text: no need to read it), then
   `design.write { path: ".pcbkit/design.pcbkit", text, build: true }`: blocks and parts joined by
   nets, no coordinates. `build: true` places, routes and checks in the same call.
3. **Open it:** `board.open` once per conversation, right after the first design; it follows every
   later change by itself, the person's included. Where the host draws no board (Codex, Claude
   Code), its answer is the live page's link: give it in your answer, once. Never call it again.
4. **Fix** what the build's answer spells out with `design.edit { …, build: true }` (one call
   changes and builds; `design.check` only when it says there are more). A build keeps the copper that
   still matches and routes only what is new or broken. The engine's own placement and copper
   are yours to redo (`design.run { all: true }`) without asking; ask only before redoing what the
   person placed or routed themselves. Never end a turn on errors the build names: do what it says
   (`soft(…)`, a named zone such as `area(.bottom)` instead of a box of coordinates, fewer `far`s)
   and build again. `design.diagnose { id: "GND" }` says why a connection does not fit and the one
   thing to do.
5. **Celebrate** the first complete check with `board.tour` (a short camera tour, once per board),
   and make it orderable: `parts.select {}` picks a real, stocked part for every component (the
   board is not ready for a factory without them); name the parts it could not find and fix them
   (`parts.alternatives { id }`, `parts.select { pin }`) before calling the board ready.
6. **Report in a sentence or two**, in plain words, by the board's name, never with ids: the numbers
   that matter (size, a blinker's on and off times) and what you saw (`board.snapshot {}` once per
   turn after parts moved). A complete check is not the review: name what
   `analysis.run { lens: "ready" }` still has to fix or to consider. Offer two or three next steps (`help { topic: "guide" }` has the
   ladder: make it theirs, move things, add features, make it sturdy, grow it, order it).
7. **Each further request is one small change** with `build: true` (`design.edit`, `board.move`,
   `board.set`, …) and the same short report. The view goes to what you changed by itself; to show
   something else, `board.camera { ids }`; what the person asks to see, `board.look { view, layers, theme, panel }` and the other `view` ops do.

## Be honest

Speak to the person about their board: never about this skill, the tools, their rules or how
you were told to work.

The check is not a review: name connections that did not route and why, never call a board ready
when its check is not `complete` or the review has errors left, and ask the person to review the
design before ordering.
