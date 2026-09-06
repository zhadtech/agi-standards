# Agent standards

A copy-paste kit that makes a coding agent resumable across sessions without
re-reading the codebase every time.

The problem it solves: a session hits a limit and dies; the next agent starts cold,
re-explores the repo to rebuild context, and burns a large fraction of its budget
rediscovering things the previous session already knew. A progress file alone doesn't
fix this, because a progress file records *what happened* rather than *what to do next*
— so the new agent still explores.

The fix is four files with strictly separated jobs, and a protocol that keeps them true.

## Install

```bash
./install.sh /path/to/new/project
```

Then fill in section 8 of `AGENTS.md` (the project's commands). That's the five minutes
with the highest return in the whole kit.

## The files

| File | Job | Loaded |
|---|---|---|
| `AGENTS.md` | rules and behavior — never facts about current work | every session, automatically |
| `CLAUDE.md` | one-line pointer at `AGENTS.md` so you maintain one file | every session |
| `INDEX.md` | path → what it is → **when to open it** | on demand, one row at a time |
| `PLAN.md` | current task as verifiable steps with checkboxes | at boot |
| `PROGRESS.md` | `NEXT ACTION`, dead ends, decisions, append-only log | at boot, top block only |

The split matters. Rules are stable and belong in the cached prefix. Repo structure is
semi-stable and should be looked up, not memorized. Task state is volatile and gets
rewritten constantly. Mixing them means every task update re-bills your rules.

## The loop

**Boot** → `AGENTS.md` → `PROGRESS.md` NEXT ACTION → `PLAN.md` first unchecked step →
`INDEX.md` rows for the files that step touches. Then start working. No exploration.

**Per step** → do it → run its verification → tick the `PLAN.md` box and append the
`PROGRESS.md` entry *in the same turn* → next step.

**At ~60% context** → finish the current step, rewrite `NEXT ACTION`, stop.

## Why it survives a hard cutoff

- `NEXT ACTION` sits at the top of `PROGRESS.md` and is literal enough to act on cold.
- Every entry carries `file:line` anchors and the exact verify command, so nothing is re-derived.
- **Dead ends are recorded with reasons** — the next agent skips the failures instead of
  repeating them. This is where most of the savings actually come from.
- Bookkeeping happens per step, so an abrupt death loses at most one step of state.
- `INDEX.md`'s "open it when" column turns search into lookup.

## Files

- [AGENTS.md](AGENTS.md) — the standards file. 40 numbered rules, ~120 lines.
- [PLAYBOOK.md](PLAYBOOK.md) — 145 collected tips from practitioners, with sources. Reference, not context. Promote rules from here into `AGENTS.md` when you get bitten.
- [templates/](templates/) — blank `INDEX.md`, `PLAN.md`, `PROGRESS.md`.
- [install.sh](install.sh) — copies the kit into a project, never overwrites.

## Maintaining it

Add a rule when an agent gets the same thing wrong twice. Delete one that never fires.
Anything that must hold every single time belongs in a hook, a lint rule, or CI — not
in a prompt. A 400-line rulebook is followed worse than a 120-line one.
