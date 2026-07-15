---
name: kickoff
description: Start-of-workstream ritual (ADR-0025) — read the lane scoreboard and kickoff brief, reconcile with what merged since, then run the planning workflow before any implementation. Use first in a fresh workspace, e.g. "/kickoff perception".
---

# Kickoff

You are starting a workstream in a fresh workspace. The argument (if given)
names the lane or brief, e.g. `perception` → `docs/kickoffs/perception-next.md`.
The umbrella repo lives at `~/code/pokemon`. Do all of this **before writing
any code**.

## 0 — No argument? Present the menu

Robert shouldn't need to remember slugs. If invoked bare:

1. **First** `git -C ~/code/pokemon pull --ff-only` — the menu must be built
   from current main, or last night's close-outs are invisible. Then list
   `~/code/pokemon/docs/kickoffs/*-next.md` and read the scoreboard
   (`docs/STATUS.md`) for in-flight lanes AND the Blocked-on column.
2. Show a short menu: each open brief's lane, its one-line objective, and its
   target repo (briefs carry a "Target:" line; if one doesn't, infer from its
   pointers and say you inferred).
3. Recommend ONE: highest-priority open brief that (a) targets this repo,
   (b) is not blocked on Robert per the scoreboard (never recommend a lane the
   session can't actually start), with critical-path lanes K/P outranking
   support lanes. Ask Robert to confirm or pick another, then proceed with it
   as the argument.

## 1 — Load the coordination state

1. `git -C ~/code/pokemon fetch origin && git -C ~/code/pokemon pull --ff-only`
   (the umbrella main is the coordination bus; it must be current).
2. Read `~/code/pokemon/docs/STATUS.md` — the lane scoreboard and launch graph.
3. Read the lane's brief in `~/code/pokemon/docs/kickoffs/`. If no brief
   exists, say so and derive the objective from the scoreboard row +
   LAUNCH_PLAN/LAUNCH_STRATEGY, then confirm it with Robert before proceeding.

## 2 — Reconcile with reality

The brief was written at close-out time; other lanes have merged since.

1. Run `~/code/pokemon/tools/workstreams.sh --fetch` for the live git state.
2. In each repo the brief touches:
   `git log --oneline --since="<brief-date>" origin/main`
   — anything merged since the brief was written?
3. Flag explicitly anything that changes the brief's picture (a contract that
   landed, an endpoint that now exists, a verdict that was ratified). If a
   change invalidates the brief's objective, stop and tell Robert rather than
   building on a stale premise.

## 3 — Plan, then build

1. Run the umbrella CLAUDE.md planning workflow: Planner proposes, adversarial
   Plan Reviewer critiques (KISS / smallest-change / no speculative
   generalization), iterate to agreement. Assign the domain specialization the
   lane calls for.
2. State the plan, the exit criterion (block-exit criteria live in
   LAUNCH_STRATEGY), and what "done" will look like — then implement.
3. When the workstream wraps, end with `/close-out`.
