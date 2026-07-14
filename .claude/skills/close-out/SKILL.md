---
name: close-out
description: End-of-workstream ritual (ADR-0025) — verify everything is merged, pushed, and cleaned up; update the docs; write the kickoff brief for the lane's next piece of work. Use when wrapping up a workstream or ending a substantial session.
---

# Close-out

You are closing out a workstream. Work through every gate below **in order**,
run the actual commands (never assume), and finish with a ✅/🚨 checklist so
Robert gets certainty instead of a vague feeling that things shipped.

The umbrella repo lives at `~/code/pokemon` (docs, STATUS scoreboard, ADRs,
kickoff briefs). Per ADR-0025, umbrella doc changes commit **directly to main**
and push immediately.

## Gate 0 — adversarial QA (before anything ships)

If this session's non-trivial work has not already had an adversarial QA pass
(umbrella CLAUDE.md: "QA gates everything" — and it applies to docs and tooling,
not just code), run it now, before the checklist can go green:

1. Spawn independent reviewer agent(s) instructed to **refute, not confirm** —
   verify factual claims against git/GitHub reality, run the code/scripts, check
   links and cross-doc consistency, attack instructions by following them
   literally in the repo's real current state.
2. Fix the findings. If the PR is already merged, findings become follow-up
   fixes pushed in this same session — never left as notes.
3. Record in the Gate-4 report: who reviewed, findings count, what was fixed.

Skip only for trivial mechanical changes (typo-level), and say so explicitly.

## Gate 1 — everything shipped

1. `git status` in this workspace. Every intended deliverable must be committed.
   If something must stay unfinished, commit it as clearly-marked WIP — per
   ADR-0025 **WIP is always pushed at session end**; uncommitted work is
   invisible to every other workspace.
2. Push the branch. Confirm with `git log origin/<branch> -1`.
3. If the work is complete: is the PR merged? (`gh pr view --json state,url`,
   or create one / merge per this repo's convention). Record the merge commit.
   **deckgym-core exception:** card PRs go to *upstream* (bcollazo) and merge
   on the maintainer's clock — there, "shipped" = PR open upstream + CI green;
   never merge card branches into fork `main` directly (CLAUDE.md invariant:
   stay close to upstream); sync fork `main` only after upstream merges.
4. After merge: `git -C ~/code/<this-repo> pull --ff-only` so the local main
   checkout matches GitHub, then delete the branch. PRs here are
   **squash-merged**, so `git branch -d` will refuse ("not fully merged") —
   after confirming state = MERGED via `gh pr view`, use `git branch -D <branch>`
   and `git push origin --delete <branch>`.
   Exception: do NOT delete the branch this Conductor workspace has checked
   out — Conductor needs it until the workspace is closed; say so in the report
   instead.

## Gate 2 — everything documented

1. **Component CHANGELOG** in this repo — entry for what shipped.
2. **Umbrella STATUS scoreboard** (`~/code/pokemon/docs/STATUS.md`): update this
   lane's row — "Last landed" (PR + date), clear/adjust "In flight", set the new
   "Next action". If a narrative entry is warranted (substantial session), add
   it under "Latest changes" and move the displaced entry to
   `docs/status-archive.md`. Refresh the launch-graph node colors if a
   launch-gating item changed state.
3. **ADR** in `~/code/pokemon/docs/decisions/` if a non-trivial decision was
   made (+ index row in `decisions/README.md`).
4. Commit the umbrella changes directly to main and push. Update from any
   **up-to-date** checkout of umbrella main (an umbrella Conductor worktree
   synced to origin/main works too — push with `git push origin HEAD:main`).
   If using `~/code/pokemon` and it has unrelated uncommitted files, use
   `git pull --rebase --autostash` (plain rebase refuses on a dirty tree) and
   leave the unrelated files untouched.

## Gate 3 — the kickoff brief (the handoff)

Write `~/code/pokemon/docs/kickoffs/<lane>-next.md` (copy
`docs/kickoffs/TEMPLATE.md`) for the next piece of work on this lane. Capture
what THIS session uniquely knows and a fresh session cannot recover:

- what just shipped (PR links, ADRs) and the state it left things in
- the next objective and why it's next (link LAUNCH_PLAN/ROADMAP items)
- hard constraints and invariants that bind the work (AGPL boundary, frozen
  contracts, pinned instruments…)
- open questions, known traps, things that were harder than they looked

Do **not** write a step-by-step implementation plan — the fresh session plans
against current main with the Planner + adversarial Plan Reviewer workflow
(umbrella CLAUDE.md); by then other lanes will have merged.

Commit the brief to umbrella main and push.

## Gate 4 — report

End with a checklist, one line per item above, each marked ✅ or 🚨 with the
evidence (commit hash, PR URL, file path). List anything intentionally left
open. If any 🚨 remains that you cannot resolve, say exactly what Robert must
do to close it.
