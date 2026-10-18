# Helprism agent plugins

Plugins that connect coding agents to [Helprism](https://helprism.com), the help center that updates itself when you ship.

| Plugin | What it does |
|---|---|
| [`plugins/helprism`](plugins/helprism) | **Helprism sync (beta).** One skill, `helprism-sync`: after a code change, your coding agent decides whether any article in your Helprism help center is now wrong or incomplete, drafts the rewrite, and sends it to Helprism for human review. Connects the hosted Helprism MCP server (`https://api.helprism.com/mcp`): install, then sign in with your browser when asked. |

Install in Claude Code (2.1.275 or later):

```
/plugin install helprism --marketplace helprism/agent-plugins
```

or, in two steps, `/plugin marketplace add helprism/agent-plugins`, then `/plugin install helprism@helprism`. Then sign in with your browser when asked.

The skill is also served at https://helprism.com/skills/helprism-sync/SKILL.md. More on how it works: https://helprism.com/features/sync

This repository is published from Helprism's main codebase; issues are welcome here. MIT licensed.
