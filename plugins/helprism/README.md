# Helprism sync (beta)

Keeps a [Helprism](https://helprism.com) help center true when you ship. After your coding agent finishes a change, or before it opens a pull request, the `helprism-sync` skill writes a one-line impact note, classifies the change, reads the live help center through the Helprism MCP server, and keeps only the articles that now have a sentence that is wrong or a thing a reader needs that is missing. For each one it drafts the full rewritten article and sends it to Helprism with `report_change`. A person accepts, edits or rejects every suggestion in the Helprism Changes inbox. Nothing is published by the skill.

## What it needs

- A Helprism workspace on the Pro or Business plan and an API key from Settings > API.
- The Helprism MCP server, `@helprism/mcp@1.2.0` from npm, connected in your agent with that API key. This plugin does not install or start it. Install commands for Claude Code, Codex, Cursor and Windsurf are in the package README: https://www.npmjs.com/package/@helprism/mcp
- git, in the repository the agent is working in.

## What it sends, and where

Only to your own Helprism workspace at api.helprism.com, through the MCP server you connect: the file paths the change touched, the commit subject as the title, the impact note, and the article prose it drafted. Never a diff, a patch or the contents of any file; the Helprism API rejects a body that carries code. The plugin itself contains no code that runs: it is one SKILL.md, and it fetches nothing.

## Use it

Ask, in the session that wrote the change: **Run helprism-sync on this change.** The agent replies with a short summary and a review link, or with "no change" when the help center is still right, which is the answer for most internal, visual and marketing work.

## Beta

Beta: expect some missed updates and the occasional false alarm. Every suggestion waits for a person in the Helprism Changes inbox, so nothing reaches readers without a human accept.

MIT licensed. Source and issues: https://github.com/helprism/agent-plugins
