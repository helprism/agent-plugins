# Helprism sync (beta)

Keeps a [Helprism](https://helprism.com) help center true when you ship. After your coding agent finishes a change, or before it opens a pull request, the `helprism-sync` skill writes a one-line impact note, classifies the change, reads the live help center through the Helprism MCP server, and keeps only the articles that now have a sentence that is wrong or a thing a reader needs that is missing. For each one it drafts the full rewritten article and sends it to Helprism with `report_change`. A person accepts, edits or rejects every suggestion in the Helprism Changes inbox. Nothing is published by the skill.

## Install

In Claude Code (2.1.275 or later), one step:

```
/plugin install helprism --marketplace helprism/agent-plugins
```

Or add the marketplace first, then install from it:

```
/plugin marketplace add helprism/agent-plugins
/plugin install helprism@helprism
```

## Requirements

Install, then sign in when asked.

- The plugin connects your agent to the hosted Helprism MCP server at `https://api.helprism.com/mcp`. The first time the agent uses it, your browser opens on helprism.com: sign in (or create an account, which starts a 14-day trial), pick the workspace and approve. In Claude Code you can also start the sign-in from `/mcp`. No API key to create or paste, and nothing to install from npm.
- A Helprism workspace on the Pro or Business plan, and an owner or admin to approve the connection. Each approval is listed, and can be revoked, in Helprism under Settings > API > Connected tools.
- git, in the repository the agent is working in.

Prefer a server that runs on your machine? The local package `@helprism/mcp` works the same way: run `npx -y @helprism/mcp@latest login` once to sign in with your browser, then add it to your agent. Commands for each agent are in the package README: https://www.npmjs.com/package/@helprism/mcp

## What it sends, and where

Only to your own Helprism workspace at api.helprism.com, through the Helprism MCP server: the file paths the change touched, the commit subject as the title, the impact note, and the article prose it drafted. Never a diff, a patch or the contents of any file; the Helprism API rejects a body that carries code. The plugin contains no code that runs: it is one SKILL.md and one `.mcp.json` that names the hosted server's URL. It reads no credential from your machine; the sign-in happens in your browser.

## Use it

Ask, in the session that wrote the change: **Run helprism-sync on this change.** The agent replies with a short summary and a review link, or with "no change" when the help center is still right, which is the answer for most internal, visual and marketing work.

## Beta

Beta: expect some missed updates and the occasional false alarm. Every suggestion waits for a person in the Helprism Changes inbox, so nothing reaches readers without a human accept.

MIT licensed. Source and issues: https://github.com/helprism/agent-plugins
