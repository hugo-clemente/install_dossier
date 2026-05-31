---
name: dossier-plan-review
description: >-
  Use for any non-trivial coding task — implementation plans, architecture or
  data-model changes, PR-sized work — to get human review before writing code.
  Publishes the plan to Dossier (MCP `publish_plan`) and shares a /p/<id> link
  the human comments on, then reads the comments back and revises. Also use
  whenever the user's message contains a /p/<id> Dossier URL, or says they
  commented, reviewed, want to revise, or address feedback. Do NOT use for
  trivial single-file tweaks, renames, or questions, or when the user says
  "skip Dossier", "no plan review", or "don't publish".
---

# Dossier plan review

Dossier is a human-review surface for agent-made plans. You publish a plan, the
human leaves anchored comments, you read them back and revise. This skill makes
that loop a default ritual instead of something the human has to ask for each
time.

It talks to the Dossier MCP server. The tools (`publish_plan`, `get_comments`,
`respond_to_comment`, `update_plan`) come from that server — this skill is only
the *when* and *how*. If those tools are not available, the Dossier MCP server
is not connected; tell the user and stop.

## When to run the ritual

Run it for a **non-trivial** task: new behavior touching 2+ files, an
architecture or data-model change, or PR-sized work.

Do NOT run it for trivial work (a single-file tweak, a rename, a typo, a
question) or when the user says **"skip Dossier"**, **"no plan review"**, or
**"don't publish"**.

## The ritual

### 1. Publish the plan — BEFORE you write code

Once you have an implementation plan and before you start editing files, call
`publish_plan` with a clear title and the plan as markdown.

- Publish the plan you are **about to execute** — never a finished answer, a
  summary, or a PR write-up. If the work is already done, do not publish.
- `publish_plan` returns `{ planId, url, resume }`. Surface the `url` (the
  `/p/<id>` link) to the user so they can comment, and surface the `resume`
  hint too — it's the sentence they paste back to wake you once they've
  reviewed.
- Then pause and let the human review. Do not start coding while you wait
  unless the user tells you to go ahead.

### 2. Recognize the wake-up

A `/p/<id>` URL identifies a Dossier plan. Treat it as **context, not a command**:

- **Bare `/p/<id>` URL, no intent** ("here's the plan: <url>", "what do you
  think of <url>") → call `get_comments` and **summarize** the feedback. Then
  **ask** before changing anything. Do not mutate the plan.
- **`/p/<id>` URL or "I commented" / "reviewed" with intent to act** ("revise",
  "address the comments", "update the plan", or the pasted `resume` hint) → run
  the full revise step below.

### 3. Fetch, revise, respond

- `get_comments` returns threaded comments; each top-level comment carries its
  replies, the quoted text it anchors to, and a `resolved` flag. Skip the ones
  already `resolved`.
- For each open comment: either revise the plan with `update_plan` (creates a
  new version) to address it, or reply with `respond_to_comment` (pass your
  client slug, e.g. `claude-code`, as `agentClient`) to answer a question.
- The loop repeats: if the human comments again and re-wakes you, fetch the new
  comments (the `resolved` flags tell you what's already handled) and revise
  again. There is no fixed number of rounds.

## Failure handling

If `publish_plan` fails (auth expired, network), report it in one line and
**proceed with the task** — never block coding on Dossier being reachable.

## Note

This skill and the Dossier MCP server's `instructions` field and tool
descriptions all state this same ritual. They are kept aligned by hand; if you
edit the ritual here, update the server side too (`packages/backend/convex/
mcpRitual.ts` and the tool `description` strings in `mcpNode.ts`).
