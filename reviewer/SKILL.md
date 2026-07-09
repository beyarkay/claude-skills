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

## How to run it

```bash
~/.claude/skills/reviewer/review.sh --spec <PATH_OR_URL> [scope] [--focus TEXT]
```

Default scope is the working tree vs the merge-base with the base branch
(committed branch work **plus** uncommitted WIP). Override with:

- `--uncommitted` — only staged/unstaged/untracked changes.
- `--commit <SHA>` — one commit.
- `--range <A..B>` / `--range <main...HEAD>` — an explicit range.
- `--base <branch>` — base for the default/auto scope (autodetected otherwise).

Other flags: `--model <M>` (codex model override), `-C <dir>` (repo dir),
`--focus "<text>"` (extra reviewer instructions), `--dry-run` (print the prompt
and command, spend nothing — use this to preview), `--help`.

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

Running this spends the user's `codex`/OpenAI quota. Per the user's
"confirm before paid calls" rule, **get a quick go-ahead before the first real
run** (a `--dry-run` is free and needs no confirmation). Mention roughly what
you're about to review.

## After it runs

- Surface codex's findings to the user; triage them (real blocker vs nit vs
  false positive) — codex digs but can still be wrong, so sanity-check claims
  against the code before acting.
- **The reviewer never edits.** If findings warrant fixes, you make them
  yourself, then optionally re-run the reviewer to confirm.
