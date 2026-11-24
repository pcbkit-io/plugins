# PCBKit for agents

Design electronic devices with your agent: circuit boards as KiCad projects, real parts, checks,
routing, a case, and the files a factory needs, shown live in your browser. This repository
holds PCBKit's plugins for the places agents work.

## Install

**Claude Code**

```
claude plugin marketplace add pcbkit-io/plugins
claude plugin install pcbkit@pcbkit
```

Then ask for a device:

> Design a USB-C powered ESP32 weather station and show it to me.

The first time, your agent asks you to sign in to PCBKit: run `/mcp`, choose
`plugin:pcbkit:pcbkit`, and follow the link.

**Any other agent** that speaks MCP can use PCBKit without a plugin: add the server
`https://mcp.pcbkit.io/mcp` (streamable HTTP, sign-in by OAuth).

## What is in a plugin

- PCBKit's server (`https://mcp.pcbkit.io/mcp`): the tools that design, check, route and
  export a device, and a guide your agent reads as it works.
- A short skill that tells your agent when PCBKit is the right tool and how to start.

## License

MIT. PCBKit itself is a service: <https://pcbkit.io/terms>.
