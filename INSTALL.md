# INSTALL.md — agent runbook

You (the coding agent) are installing **Dossier** for your user. Dossier makes you
publish an implementation plan for human review before writing code, then read the
comments back and revise. Two pieces: a portable **skill** and an **MCP server**.

Do the steps in order. **Important:** the skill and the MCP server are both loaded
only at session **startup** — they will NOT be live in the session that installs
them. So Steps 1–2 install, **Step 3 restarts the session** (keeping context), and
Step 4 runs in the resumed session to authenticate + verify. Don't try to call
`publish_plan` before the restart — the tool won't exist yet.

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

## Step 3 — Restart the session so the skill + server load

Both the skill and the MCP server only register at startup, so the session that
ran Steps 1–2 can't use them yet. Restart **keeping context** — don't start a
fresh chat, you'd lose this conversation.

**Claude Code:** exit (`/exit`, or Ctrl-C twice), then resume:

```bash
claude --continue     # or: claude -c   — resumes the most recent session here
```

**Other harnesses:** restart the session however that harness reloads MCP servers
+ skills (most require a full restart; some have a reload/reconnect command — use
it if present).

You (the agent) **cannot cross this boundary yourself.** After Steps 1–2, stop
and tell the user:

> Skill + MCP server are installed. They load only on startup, so restart me to
> pick them up: type `/exit`, then run `claude --continue`. When you're back,
> say "finish Dossier setup" and I'll authorize and run a quick test.

When the session resumes (you'll have this context back), continue at Step 4.

## Step 4 — (resumed session) authenticate, then smoke-test

**Authenticate** the OAuth-protected server. In Claude Code this is the `/mcp`
menu — a command only the **user** can type. Ask them:

> Open the MCP menu: type `/mcp`, pick the `dossier` server, choose
> **Authenticate**. Your browser opens to sign in (`/login`) and authorize
> (`/oauth/consent`). Tell me once it shows connected.

(If `/mcp` already shows `dossier` connected, skip straight to the test.)

**Smoke test:** call `publish_plan` with an example plan to prove the chain works
end to end and give the user something to look at:

```
publish_plan({
  title: "Dossier install smoke test",
  format: "markdown",
  content: "# Dossier is installed\n\nExample plan published to confirm the install works end to end.\n\n## What just happened\n1. The dossier-plan-review skill is installed.\n2. The Dossier MCP server is connected and authorized.\n3. publish_plan returned this shareable /p/<id> link.\n\nLeave a comment on any line — your agent can read it back with get_comments and revise."
})
```

It returns a `url` (the `/p/<id>` link) and a `resume` hint. **Open the URL in the
user's browser:**

```bash
open "<url>"        # macOS
xdg-open "<url>"    # Linux
start "<url>"       # Windows
```

Then confirm:

> Done — Dossier is installed and working. I opened your example plan: `<url>`.
> From now on, before I write code I'll publish my plan there and share the link
> so you can leave comments.

---

## Verify

- Skill present: shows up in the harness's skill list as `dossier-plan-review`
  (after the Step 3 restart).
- MCP present: `/mcp` lists `dossier` as connected.
- End to end: Step 4's `publish_plan` returned a `/p/<id>` URL and it opened in
  the browser.

From now on, before writing code, publish the plan and share the `/p/<id>` link
for review.
