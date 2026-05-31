# INSTALL.md — agent runbook

You (the coding agent) are installing **Dossier** for your user. Dossier makes you
publish an implementation plan for human review before writing code, then read the
comments back and revise. Two pieces: a portable **skill** and an **MCP server**.

Do the steps in order. **Important:** the skill and the MCP server both load only at
session **startup** — they will NOT be live in the session that installs them. So
Steps 1–3 install, **Step 4 restarts the session** (keeping context), and Step 5 runs
in the resumed session to verify. Don't try to call `publish_plan` before the
restart — the tool won't exist yet.

## Step 0 — Ask where to install

Ask the user one question before installing:

> Install Dossier **globally** (available in all your projects) or **just this
> project**? (Default: global.)

Use their answer to pick the flags below. Global is the right default — Dossier is a
workflow you want everywhere, not per-repo.

## Step 1 — Install the skill

The skill is installed with the [`skills`](https://github.com/vercel-labs/skills)
CLI. `-g` installs globally (user-level); omit it for project-level. `-a` is the
harness (default Claude Code if unsure; supports `claude-code`, `cursor`, `codex`,
`opencode`, …):

```bash
# global (all projects)
npx skills add hugo-clemente/install_dossier --skill dossier-plan-review -a claude-code -g

# or project-only (drop -g)
npx skills add hugo-clemente/install_dossier --skill dossier-plan-review -a claude-code
```

## Step 2 — Connect the MCP server (via mcp-remote, so auth is automatic)

The skill calls the Dossier MCP server (`publish_plan`, `get_comments`,
`respond_to_comment`, `update_plan`). The server is remote + OAuth-protected:

```
https://energetic-toad-410.convex.site/mcp-node
```

Register it **through `mcp-remote`** — a small stdio bridge that runs the OAuth
browser flow itself on first connect and caches the token (`~/.mcp-auth`). This is
what removes the manual `/mcp` Authenticate step: when the session restarts and the
server connects, `mcp-remote` opens the browser automatically.

```bash
# global (all projects):  -s user
claude mcp add dossier -s user -- npx -y mcp-remote https://energetic-toad-410.convex.site/mcp-node

# or project-only:  -s local  (the default; drop -s)
claude mcp add dossier -- npx -y mcp-remote https://energetic-toad-410.convex.site/mcp-node
```

**Other harnesses:** register a **stdio** MCP server named `dossier` running
`npx -y mcp-remote https://energetic-toad-410.convex.site/mcp-node`. Example for
Cursor (`~/.cursor/mcp.json` for global, or project `.cursor/mcp.json`):

```json
{ "mcpServers": { "dossier": { "command": "npx", "args": ["-y", "mcp-remote", "https://energetic-toad-410.convex.site/mcp-node"] } } }
```

> If a harness can't run a stdio command, fall back to a remote HTTP server at the
> same URL — but then auth is a manual in-app step (e.g. Claude Code's `/mcp` menu)
> rather than automatic.

## Step 3 — (stop here) hand off the restart

The skill + server load only at startup, so the session that ran Steps 1–2 can't use
them yet. You (the agent) **cannot cross this boundary yourself.** Stop and tell the
user:

> Dossier's skill + MCP server are installed. They load only when I restart. Type
> `/exit`, then run `claude --continue` (keeps this conversation). When you're back,
> say "finish Dossier setup" — your browser will pop open once to sign in, then I'll
> publish your first plan.

## Step 4 — Restart the session (user action)

**Claude Code:** `/exit` (or Ctrl-C twice), then:

```bash
claude --continue     # or: claude -c  — resumes this session with context intact
```

**Other harnesses:** restart however that harness reloads MCP servers + skills.

## Step 5 — (resumed session) auth happens automatically, then smoke-test

On restart the `dossier` server connects and **`mcp-remote` opens the browser by
itself** to sign in (`/login`) and authorize (`/oauth/consent`) — no `/mcp` click
needed. Tell the user up front:

> Your browser will open once to sign in to Dossier. Approve it, and I'll publish a
> quick example plan.

(If a harness fell back to remote HTTP in Step 2, auth is manual instead: have the
user open `/mcp` → `dossier` → Authenticate.)

Once connected, **smoke-test** by publishing an example plan — proves the whole chain
and gives the user something to look at:

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
  (after the Step 4 restart).
- MCP present: `/mcp` lists `dossier` as connected.
- End to end: Step 5's `publish_plan` returned a `/p/<id>` URL and it opened in
  the browser.

From now on, before writing code, publish the plan and share the `/p/<id>` link
for review.
