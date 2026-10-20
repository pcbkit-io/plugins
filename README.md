# PCBKit

![PCBKit](assets/logo.png)

[PCBKit](https://pcbkit.io) designs printed circuit boards with you. Describe what the board
should do — "a USB-C powered sensor node with an ESP32 and a temperature sensor" — and it is
written in PCBKit, a small board language with real units, built from proven circuit blocks
(regulators, USB, microcontroller support, protection). PCBKit checks the design (electrical rules,
clearances, the fab's limits), places the parts, routes the copper and fills the pours, then
exports Gerber and drill files with the bill of materials and placement list for the fab, or a
KiCad board to continue in KiCad. Ask for changes in plain words ("move the connector to the left
edge", "make the power tracks wider") and it changes the board and routes it again.

## Install

**Claude** (claude.ai, Cowork, the desktop app): add PCBKit from the directory.

**Claude Code:**

```sh
claude plugin marketplace add seralexeev/pcbkit-plugins
claude plugin install pcbkit@pcbkit
```

**ChatGPT:** add PCBKit from the apps and plugins directory. A workspace admin can also import this
repository as a marketplace (Admin → Plugins → Import marketplace).

**Codex:**

```sh
codex plugin marketplace add seralexeev/pcbkit-plugins
codex plugin add pcbkit@pcbkit
```

Then ask: "Make a board with an LED that blinks every 2 seconds."

In Claude and ChatGPT the board opens in the conversation as an interactive app (PCBKit's own
renderer, 2D and 3D): look at it, select and move parts, route it, play its tour, share it by link,
or go fullscreen for the whole editor; the menu lists your boards and asks the assistant to fix
what the check finds.

## What it contains

One plugin for both agents; each reads its own manifest and both share the skills.

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json`: the plugin for
  Claude and Claude Code.
- `plugin.json`, `mcp.json`, `.agents/plugins/marketplace.json`: the plugin for ChatGPT and Codex.
- Both connect to the PCBKit MCP server, `https://mcp.pcbkit.io/mcp` (remote, streamable HTTP).
- `skills/pcbkit-boards`: how to design, change, check, route and export a board with its tools.
- `skills/setup`: introduces PCBKit and starts the first board.
- `assets/`: the logo and icons.

The plugin runs nothing on your computer. The tools run on PCBKit's servers: the board's files
and the requests you make through the tools are sent to `mcp.pcbkit.io`, where the boards are kept
(one per conversation, more with `project.new`; signed-in boards with your account) and exports are kept as download links
for 7 days. No account is needed to start. See the [privacy policy](https://pcbkit.io/privacy) and
the [terms](https://pcbkit.io/terms).

## Limits

Two-sided and multilayer boards of everyday size and density; very large or high-speed boards may
not route completely, and PCBKit says which connections are missing. Review every design before you
order it.

Support: [pcbkit.io/support](https://pcbkit.io/support), support@pcbkit.io.
