---
name: reviewer
description: Adversarial second-opinion code review of a diff, powered by a read-only `codex`. Use to hunt for bugs, regressions, edge cases, inconsistencies, and spec-violations in changes — e.g. "review this", "check my changes for bugs", "get a second opinion", "did I actually do what the issue asked", or as a self-check before committing/finishing nontrivial work. codex reads the whole repo to dig deeper but CANNOT edit anything; it only reports findings. Pass the ORIGINAL issue/spec by link, never a paraphrase.
---

# reviewer

Runs a hostile, skeptical code review of a diff using `codex exec` in a
**read-only sandbox**. codex can read the entire repository and run
`git`/`rg`/`cat` to verify its suspicions, but it cannot edit, create, or delete
anything — it reports findings and recommendations only. The engine is
`review.sh` in this directory.

## When to use it

- The user asks for a review, a second opinion, or a bug-hunt on some changes.
- As a **self-check before committing or declaring nontrivial work done** —
  catch your own mistakes with an independent adversary.
- To confirm a change actually satisfies the issue/spec it was meant to.

## Invoking as `/reviewer [prompt]`

When the user runs `/reviewer` (optionally with text), **that invocation is the
go-ahead — kick off the review immediately**, don't deliberate or re-confirm.
Any free text becomes the reviewer's **focus** and is forwarded straight to
codex, e.g. `/reviewer scrutinise the retry logic`. So the flow is:

1. Map the user's text to a `review.sh` call: free text → focus; if they name a
   scope ("just my uncommitted changes", "that last commit") use the matching
   flag; if they point at an issue/spec, pass it via `--spec` (see THE RULE).
2. Run it. The script prints a context banner before codex starts, e.g.:

   ```
   reviewer: diff to review — HEAD (eaa6b52, my-feature-branch) vs main (4a31ccb), incl. uncommitted WIP
   reviewer: +5 / -1 across 2 file(s) · focus: scrutinise the retry logic
   reviewer: starting codex (model <codex default>, read-only sandbox) → report /tmp/reviewer-report.NNN.md
   ```

   Glance at it — if the stat looks wrong (e.g. thousands of lines from
   untracked data dirs pulled in by the default scope), narrow with a scope flag
   before spending.

## How to run it

```bash
~/.claude/skills/reviewer/review.sh [--spec <PATH_OR_URL>] [scope] ["focus text"]
```

Default scope is the working tree vs the merge-base with the base branch
(committed branch work **plus** uncommitted WIP). Override with:

- `--uncommitted` — only staged/unstaged/untracked changes.
- `--commit <SHA>` — one commit.
- `--range <A..B>` / `--range <main...HEAD>` — an explicit range.
- `--base <branch>` — base for the default/auto scope (autodetected otherwise).

Scope options are mutually exclusive. Free-text trailing args (or `--focus
"<text>"`) become the reviewer's focus. Other flags: `--model <M>` (codex model
override), `-C <dir>` (repo dir), `--dry-run` (print the banner + prompt +
command, spend nothing — use this to preview), `--help`.

## THE RULE: link the spec, never paraphrase it

The `--spec` argument is how codex learns what the change was *supposed* to do.
**Always give it the real source, never your own summary** — so the review can't
be biased by how you'd retell the request:

- A local file: `--spec docs/issue-42.md`, `--spec DESIGN.md` → passed as a path
  codex reads itself.
- A GitHub issue/PR: `--spec https://github.com/org/repo/issues/42` → fetched
  verbatim via `gh` (falls back to `curl`).
- Any URL → fetched verbatim via `curl`.
- Repeatable: pass `--spec` several times for multiple sources.

If the request only lives in this chat and there is no durable source, prefer to
**ask the user for the link/file** rather than transcribing it. If there is
genuinely no spec, run without `--spec` (it reviews purely for correctness and
quality) — but do not invent a spec block.

## Cost & etiquette

Running this spends the user's `codex`/OpenAI quota.

- **User-invoked** (`/reviewer …`, or "review this / get a second opinion"): the
  request *is* the go-ahead — run immediately, no extra confirmation. The banner
  already tells them what's being reviewed.
- **Self-initiated** (you decide to review your own work without being asked):
  per the "confirm before paid calls" rule, get a quick go-ahead first. A
  `--dry-run` is free and never needs confirmation.

## After it runs

- Surface codex's findings to the user; triage them (real blocker vs nit vs
  false positive) — codex digs but can still be wrong, so sanity-check claims
  against the code before acting.
- **The reviewer never edits.** If findings warrant fixes, you make them
  yourself, then optionally re-run the reviewer to confirm.
