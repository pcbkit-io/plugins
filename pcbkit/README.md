# PCBKit

Design electronic devices with your agent. Describe what you want to build, and your agent
designs it with PCBKit: it chooses real parts, draws the schematic, lays out and routes the
circuit board, checks it with KiCad's own rules, and makes the files a factory needs. You watch
every step live in your browser.

## What you can ask

- "Design a USB-C powered ESP32 board with a temperature sensor."
- "Open my KiCad project from GitHub and check whether it is safe to order."
- "Make the power tracks wider and route the board again."
- "Give me the files for JLCPCB."

## Install

```
/plugin install pcbkit --marketplace pcbkit-io/plugins
```

## How it works

Each device is a project on [PCBKit](https://pcbkit.io), kept in a computer of its own with
KiCad 10 and PCBKit's tools. The first time your agent uses PCBKit you sign in with a free
account, in your browser: in Claude Code run `/mcp` and choose `plugin:pcbkit:pcbkit`; in
Codex run `codex mcp login pcbkit`; ChatGPT and Claude ask by themselves.

Your projects are ordinary KiCad files. Ask your agent to bring them into your folder whenever
you want them.

## Privacy and support

PCBKit keeps your projects and the conversation's tool calls to do the work:
<https://pcbkit.io/privacy>. Questions and problems: <https://pcbkit.io/support>.
