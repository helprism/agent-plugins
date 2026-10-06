# Helprism agent plugins

Plugins that connect coding agents to [Helprism](https://helprism.com), the help center that updates itself when you ship.

| Plugin | What it does |
|---|---|
| [`plugins/helprism`](plugins/helprism) | **Helprism sync (beta).** One skill, `helprism-sync`: after a code change, your coding agent decides whether any article in your Helprism help center is now wrong or incomplete, drafts the rewrite, and sends it to Helprism for human review. Works with the Helprism MCP server (`@helprism/mcp` on npm). |

The skill is also served at https://helprism.com/skills/helprism-sync/SKILL.md. More on how it works: https://helprism.com/features/sync

This repository is published from Helprism's main codebase; issues are welcome here. MIT licensed.
