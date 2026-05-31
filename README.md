# Install Dossier

**Dossier** makes your coding agent publish its plan for human review *before*
it writes code, share a `/p/<id>` link for comments, then read the comments back
and revise.

## Install — paste one prompt

Copy this and give it to your coding agent (Claude Code, Cursor, Codex, …):

```
Install Dossier. Read
https://github.com/hugo-clemente/install_dossier/blob/main/INSTALL.md
and follow it for my coding harness.
```

The agent installs the skill, connects the MCP server, and tells you when to
authorize in the browser. The runbook it follows is
[`INSTALL.md`](INSTALL.md).

## Install — by hand

```bash
# 1. the skill
npx skills add hugo-clemente/install_dossier --skill dossier-plan-review -a claude-code

# 2. the MCP server
claude mcp add --transport http dossier https://energetic-toad-410.convex.site/mcp-node
```

The skill + server load only at startup, so **restart your session** to pick them
up (Claude Code: `/exit`, then `claude --continue` to keep context). Then open
`/mcp`, pick `dossier`, and **Authenticate** — your browser opens once to sign in.

## What it does

See [`skills/dossier-plan-review/SKILL.md`](skills/dossier-plan-review/SKILL.md)
for the full ritual: when to publish, how a `/p/<id>` URL is treated as context
(read) vs. an instruction to revise (act), and the skip phrases.
