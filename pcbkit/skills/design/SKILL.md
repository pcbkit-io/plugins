---
name: design
description: Design, change and check electronic devices with PCBKit: circuit boards as KiCad projects (schematic, placement, routing, design-rule checks), real parts with stock and prices, a printed case, and the files a factory needs. Use when the person asks to make or change a PCB, a schematic, a KiCad project or an electronic device, to choose or look up electronic parts, to route or check a board, or to get Gerbers, a bill of materials or an order ready.
---

# Designing a device with PCBKit

PCBKit's tools come from this plugin's `pcbkit` server. Each device is a project on PCBKit, in a
computer of its own with KiCad and PCBKit's commands, and the person watches it change live in
their browser. The server tells you how to do the work itself (its instructions, and its `guide`
tool for each step): follow those. This page is only what is particular to working from here.

## Before the first call

If PCBKit's tools are not among yours, or a call says the person must sign in, ask them to run
`/mcp`, choose `plugin:pcbkit:pcbkit` and sign in (a free account, in the browser), then go on
from where you were. Do not try to do the work without the tools.

## Starting

- Something new: `new_project`. A device they already have: `projects`, then `status` for where
  its work stands.
- A KiCad project in a public GitHub repository: `new_project` with the repository's address
  brings it in whole.
- A KiCad project in this folder that is not on GitHub: make a project, then put its files in
  with `write`, each as it is (the `.kicad_pro`, `.kicad_sch`, `.kicad_pcb`, its own libraries).
- Call `open` as soon as there is something to see and give the person its link: they follow
  every step there.

## While working

- The project's files live on PCBKit, not in this folder: read and change them with PCBKit's
  tools (`read`, `edit`, `write`, `shell`), not with your own file tools.
- Say in a sentence what you will make and what you decided, then do the work in the same turn:
  the schematic, its checks, the board placed, routed and checked. Ask only what would change
  the design's cost, size or safety.
- A device is done when its checks are clean, every part is a real part with its number, and
  the factory's files are made. Say what was verified and what is assumed.

## Bringing the result here

When the person wants the files in this folder (to commit them, to open them in KiCad), ask
`download` for the project (no `path`: the whole project as a zip, or a `path` for one file or
folder), fetch the link with `curl -fL -o` and unpack it. Never print a file's contents as text
to copy it.

## When you stop

Say where the work stands with `status` (what is done, what is left, what you wait for from the
person): the next conversation, here or in another app, goes on from it.
