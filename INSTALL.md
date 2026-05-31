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

## Step 3 — Smoke test: publish an example plan, then open it

This single step proves the whole chain works (skill + MCP + auth) and shows the
user a real result. The server is OAuth-protected, so this **first** tool call is
what triggers the one-time browser sign-in (`/login` → `/oauth/consent`). Tell
the user up front:

> I'm publishing a tiny example plan to confirm Dossier works. Your browser will
> open once to sign in and authorize — approve it, and I'll open the published
> plan for you.

Then call `publish_plan` with an example plan:

```
publish_plan({
  title: "Dossier install smoke test",
  format: "markdown",
  content: "# Dossier is installed\n\nThis is an example plan published to confirm the install works end to end.\n\n## What just happened\n1. The dossier-plan-review skill is installed.\n2. The Dossier MCP server is connected and authorized.\n3. publish_plan returned this shareable /p/<id> link.\n\nLeave a comment on any line — your agent can read it back with get_comments and revise."
})
```

`publish_plan` returns a `url` (the `/p/<id>` link) and a `resume` hint. **Open
the URL in the user's browser** so they can see the plan immediately:

```bash
open "<url>"        # macOS
xdg-open "<url>"    # Linux
start "<url>"       # Windows
```

Then confirm to the user:

> Done — Dossier is installed and working. I just opened your example plan:
> `<url>`. From now on, before I write code I'll publish my plan there and share
> the link so you can leave comments.

---

## Verify

- Skill present: shows up in the harness's skill list as `dossier-plan-review`.
- MCP present: the harness lists a `dossier` server.
- End to end: Step 3's `publish_plan` returned a `/p/<id>` URL and it opened in
  the browser.

From now on, before writing code, publish the plan and share the `/p/<id>` link
for review.
