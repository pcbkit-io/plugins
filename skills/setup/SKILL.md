---
name: setup
description: Introduce PCBKit after it is installed and start the person's first circuit board.
---

# Getting started with PCBKit

Tell the person, in two or three sentences, what PCBKit does: it designs printed circuit
boards from a description — composes proven circuits, checks the electrical rules, places and
routes the copper — and exports fabrication files (Gerber, drill, BOM, placement) or a KiCad board.
No account is needed; signing in keeps their boards with their account.

Then ask what they want to build, offering three examples they can pick:

- an LED that blinks every 2 seconds (the classic first board);
- a breakout board for an I2C sensor with a Qwiic connector;
- a small ESP32 board with a button and a status LED.

When they answer, follow the `pcbkit-boards` skill: write a first design at once and show it, so
they see their board within a minute, then lead them one step at a time (`help { topic: "guide" }`).
Say it in their language and in plain words; no experience is needed.
