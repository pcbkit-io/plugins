---
name: pcbkit-boards
description: Design, change, check, route and export printed circuit boards (PCBs) with PCBKit. Use when the person asks for a circuit board, a PCB, a blinking LED, a breakout or carrier board, or to change, move, label, route, check or export one for fabrication or KiCad.
---

# Circuit boards with PCBKit

PCBKit's tools write a board in PCBKit (a small board language with units), check it, place and
route it, and export it. The person sees the board beside the conversation and watches it change.

## The loop

1. **First design at once.** Choose sensible defaults for what the person did not say (power, size)
   and write the whole first design in one `files.write { path: "design.pcbkit", text }`: blocks
   and parts joined by nets, no coordinates. `help { topic: "quickstart" }` has a complete first
   board; `blocks.list` / `blocks.describe` the proven circuits. Leave the outline to the engine
   (it sizes it around the parts and rounds its corners) and give the board
   `holes = MountingHoles() {}` (a plated M3 hole in each corner) unless the person says
   otherwise, it plugs into something (pin headers, a USB stick) or it is tiny (its parts fit in
   about 15 × 15 mm, where four M3 holes would double it; `MountingHoles(screw = "M2")` if it
   still needs them). A blinker that "blinks every 2 seconds" has a 2 s period: on about 1 s, off
   about 1 s.
2. **Open it:** `board.open` once per conversation, right after the first design. The board opens
   for the person and follows every later change by itself, theirs included. Never call it again:
   each call adds another copy of the board to the conversation.
3. **Build:** `check.run`, fix what it reports, `build.run` (places and routes). The first time
   the board checks complete, `board.show { tour: true }`: a short camera tour for the person (it
   plays once and comes back to the whole board).
4. **Report in a sentence or two**, in plain words, by the board's name, never with ids, with the
   numbers that matter (a blinker's on and off times, the board's size); then offer
   two or three next steps that fit (`help { topic: "guide" }` has the ladder: make it theirs,
   move things, add features, make it sturdy, grow it, order it).
5. **Each further request is one small change** (`files.edit`, `board.move`, `board.add`, …), then
   `route.run`, and the same short report. The person's view goes to the parts you changed by
   itself; for anything else (a new text, a net) `board.show { focus: [...] }`.
6. **Export only when asked:** fabrication or KiCad files (`export.fab`, `export.kicad`) when the
   person asks for them or orders, not after a change; the board beside the conversation is enough.
7. **Text on the board** goes on the top silkscreen (`Silk.top`) unless the person asks for the
   back, where nothing else is printed; `check.run` warns (`silk-overlap`) when it is not.

## Read what the step needs

- How to lead the person, the ladder, where parts come from: `help { topic: "guide" }`.
- The board format, one section at a time: `help { topic: "format", section: "Silkscreen" }`
  (also `Board`, `Fabrication`, `Placement rules`, `Parts and components`, `Routing`, `Errors`,
  `Recipes`).
- The language: `help { topic: "pcbkit", section }`; the standard library: `lang.search`,
  `lang.describe { name }`.
- Parts: `parts.search` (verified pin tables), else the datasheet — ask the person for it or find
  the manufacturer's for the exact part number — into `parts.sheet`, `parts.check`,
  `parts.create`. Never invent pin numbers.

## One board per conversation

Every call works on this conversation's board, kept on the server (the first change makes it), so
keep improving it rather than starting over; leave `project` out. Only where answers end with
`Project: p…` (hosts without conversations) pass that id as `project` in every later call about
the same board. To work on another board the person names, pass its id from `projects.list` once.
If the person wants to keep their boards, they can sign in.

## Be honest about the board

The checks are real but not a review: name connections that did not route and why
(`board.unrouted`, `route.diagnose`), never call a board ready when the check is not `complete`,
and ask the person to review the design before ordering.
