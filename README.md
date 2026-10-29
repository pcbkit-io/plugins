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

**Claude Code** (PCBKit runs on your computer, on the project in your working directory):

```sh
claude plugin marketplace add pcbkit-io/plugins && claude plugin install pcbkit@pcbkit
```

**Codex CLI** (the same, on your computer):

```sh
codex plugin marketplace add pcbkit-io/plugins && codex plugin add pcbkit@pcbkit
```

Then start the agent in your KiCad project's folder, or in an empty folder for a new board
(`mkdir blinker && cd blinker && claude`, or `codex`). Claude Code asks before each PCBKit tool
the first time ("Yes, and don't ask again"; or allow them all at once: `"permissions": { "allow":
["mcp__plugin_pcbkit_pcbkit"] }` in `.claude/settings.json`). Codex runs PCBKit's tools without
asking (the plugin says so: they act on the project's files on your computer), `codex exec` too.
If you also added PCBKit to claude.ai, Claude Code lists that connector beside the plugin: the
plugin's tools are the ones that work on your files.

**Cursor, VS Code, JetBrains, Gemini CLI and other agents**: install the CLI
(`curl -fsSL https://pcbkit.io/install.sh | sh`) and add the MCP server `pcbkit mcp`; every host's
line: [pcbkit.io/docs/agent-hosts](https://github.com/pcbkit-io/pcbkit/blob/main/docs/agent-hosts.md).

**Claude** (claude.ai, the desktop and phone apps) and **ChatGPT**: add PCBKit from their
directory. There PCBKit runs on its servers (`https://mcp.pcbkit.io/mcp`) and the board opens in
the conversation as an interactive app.

Then ask: "Make a board with an LED that blinks every 2 seconds."

## Where it runs

**Claude Code and Codex: on your computer.** The plugin starts `pcbkit mcp`, PCBKit's command-line
tool, on the project in the directory you work in: a KiCad project with a `.pcbkit/` folder (a new
board starts there). Checking, placing, routing, analysis and exports run on your machine with all
its cores, the files are written into your project (git keeps the history) and nothing is uploaded.
The first `board.open` opens the board in your browser on app.pcbkit.io, attached to the files on
your computer: it follows every change, yours in the page included (moving parts, routing: written
into your files, routed on your machine), and shows what the agent is doing. The page reaches your
computer through PCBKit's relay with everything sealed under a key that stays in the link (the
relay cannot read it), so no port is opened on your machine and it works in every browser, also
when the agent runs over SSH or in a container. The tools work offline, but for the board's page
and what needs PCBKit's service, which they reach at pcbkit.io when asked: the parts catalogue's stock and prices, parts made from datasheets, share
links, fab orders.

The first start installs the `pcbkit` release this plugin pins from
[github.com/pcbkit-io/cli](https://github.com/pcbkit-io/cli) (macOS, Linux; checked against the
release's SHA-256 sums) into `~/.local/share/pcbkit/cli/<version>/`: about 32 MB, a few seconds,
once. `pcbkit` on your PATH at that
version is used as it is; `PCBKIT_BIN` names another binary. Updating the plugin
(`claude plugin update pcbkit@pcbkit`, `codex plugin add pcbkit@pcbkit`) brings the next release.
Windows: install `pcbkit` from the releases and use it with an agent that runs in WSL or Git Bash.

**Claude and ChatGPT: on PCBKit's servers.** The board's files and the requests you make through
the tools are sent to `mcp.pcbkit.io`, where the boards are kept (one per conversation, more with
`project.new`; signed-in boards with your account) and exports are kept as download links for 7
days. No account is needed to start. See the [privacy policy](https://pcbkit.io/privacy) and the
[terms](https://pcbkit.io/terms).

## What it contains

One plugin for both agents; each reads its own manifest and both share the skills.

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json`: the plugin for
  Claude Code.
- `.codex-plugin/plugin.json`, `mcp.json`, `.agents/plugins/marketplace.json`: the plugin for Codex.
- `bin/pcbkit-mcp`: starts `pcbkit mcp`, installing the pinned release first if needed.
- `skills/pcbkit-boards`: how to design, change, check, route and export a board with its tools.
- `skills/setup`: introduces PCBKit and starts the first board.
- `assets/`: the logo and icons.

## Limits

Two-sided and multilayer boards of everyday size and density; very large or high-speed boards may
not route completely, and PCBKit says which connections are missing. Review every design before you
order it.

Support: [pcbkit.io/support](https://pcbkit.io/support), support@pcbkit.io.
