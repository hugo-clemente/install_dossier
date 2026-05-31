# INSTALL.md — agent runbook

You (the coding agent) are installing **Dossier** for your user. Dossier makes you
publish an implementation plan for human review before writing code, then read the
comments back and revise. Two pieces: a portable **skill** and an **MCP server**.

Do all three steps. Then tell the user what's left for them (the browser sign-in).

---

## Step 1 — Install the skill

The skill is installed with the [`skills`](https://github.com/vercel-labs/skills)
CLI. Run the line for the user's harness (default to Claude Code if unsure):

```bash
# Claude Code
npx skills add hugo-clemente/install_dossier --skill dossier-plan-review -a claude-code

# Cursor
npx skills add hugo-clemente/install_dossier --skill dossier-plan-review -a cursor

# Codex
npx skills add hugo-clemente/install_dossier --skill dossier-plan-review -a codex
```

`-a` takes any harness the `skills` CLI supports (`claude-code`, `cursor`,
`codex`, `opencode`, …). If you don't know the harness, ask the user or use
`claude-code`.

## Step 2 — Connect the MCP server

The skill calls the Dossier MCP server (`publish_plan`, `get_comments`,
`respond_to_comment`, `update_plan`). It's a remote HTTP, OAuth-protected
endpoint:

```
https://energetic-toad-410.convex.site/mcp-node
```

Add it to the user's harness:

```bash
# Claude Code
claude mcp add --transport http dossier https://energetic-toad-410.convex.site/mcp-node
```

For other harnesses, add an **HTTP (remote) MCP server** named `dossier` pointing
at that URL, in the harness's MCP config:

- **Cursor** — add to `~/.cursor/mcp.json` (or project `.cursor/mcp.json`):
  ```json
  { "mcpServers": { "dossier": { "url": "https://energetic-toad-410.convex.site/mcp-node" } } }
  ```
- **Codex / others** — add an MCP server of type `http`/`url` named `dossier`
  with that URL, per the harness's MCP docs.

## Step 3 — Hand off the OAuth sign-in

The server is OAuth-protected. On the **first** tool call the harness opens a
browser to sign in (`/login`) and authorize (`/oauth/consent`). You can't click
that for the user. After Steps 1–2, tell them:

> Dossier is installed. The first time I publish a plan, your browser will open to
> sign in and authorize — approve it once and you're set.

---

## Verify

- Skill present: it shows up in the harness's skill list as `dossier-plan-review`.
- MCP present: the harness lists a `dossier` server; `publish_plan` is callable
  (triggers the one-time OAuth).

That's it. From now on, before writing code, publish the plan and share the
`/p/<id>` link for review.
