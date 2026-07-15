---
name: cards
description: Lane R card-implementation loop — sync + regenerate the scorecard, claim the next batch family from the shared queue (category-disjoint parallelism guardrail), branch from upstream/main, implement TDD with an adversarial QA gate, PR to upstream, close out. Use in a fresh deckgym-core Conductor workspace.
---

# /cards — one batch family, end to end

You are in a deckgym-core Conductor workspace. This skill runs ONE batch family
from claim to upstream PR. Coordination lives in the umbrella repo
(`~/code/pokemon`): the **scorecard** (`docs/card-scorecard.md`, generated) and
the **queue** (`docs/card-queue.md`, claims). Umbrella `main` is the
coordination bus (ADR-0025) — claims commit directly to it.

**Why the guardrails exist:** every attack batch touches
`effect_mechanic_map.rs` and every ability batch touches
`effect_ability_mechanic_map.rs` — merge hotspots. Parallel sessions must hold
**different categories**. And branches MUST come from `upstream/main`: fork
`main` carries our private tooling (`.claude/skills/{cards,close-out,kickoff}`)
which must never appear in an upstream PR, and a stale base causes duplicate
work (a Small Balloon PR was closed as a dup on 2026-07-07 for exactly this).

## 1 — Refresh ground truth

```bash
. "$HOME/.cargo/env"
git -C ~/code/pokemon pull --ff-only
~/code/pokemon/tools/card-scorecard.sh     # syncs fork↔upstream, regenerates scorecard
```

Commit the regenerated scorecard to umbrella main:
`git -C ~/code/pokemon add docs/card-scorecard.md && git -C ~/code/pokemon commit -m "docs: card scorecard refresh" && git -C ~/code/pokemon push`
(if push rejects: `git -C ~/code/pokemon pull --rebase --autostash` and retry).

## 2 — Pick + claim a family (the parallelism guardrail)

1. Read `~/code/pokemon/docs/card-queue.md`.
2. In-flight = rows with status `claimed:*` or `PR#*` (an open PR's category is
   still hot until merged — rebases touch the same map).
3. Pick the **highest tier/rank `todo` family whose category differs from every
   in-flight row's category**. Meta-relevant (Tier 1) always outranks tail.
4. If all of attack/ability/trainer+tool are in flight: **stop** and tell
   Robert which sessions hold them — do not start a fourth parallel batch.
5. Claim: edit the row's status to `claimed:<family-slug>`, then commit **only
   that file** (`git -C ~/code/pokemon add docs/card-queue.md && git -C
   ~/code/pokemon commit -m "cards: claim <slug>" -- docs/card-queue.md`) and
   push. All sessions share the `~/code/pokemon` working copy — finish the
   claim (committed AND pushed) in one uninterrupted step.
6. **Post-claim verification (the race is real — two sessions can pick
   *different* families of the *same* category before either pushes):**
   `git -C ~/code/pokemon pull --rebase --autostash`, re-read the queue. If
   another `claimed:*`/`PR#*` row now holds YOUR category, exactly one session
   yields — deterministic rule: **the claim on the lower-priority family (later
   tier/rank) yields**. If that's you: revert your row to `todo`, commit+push,
   and go back to step 3. Only proceed once your claim is the sole in-flight
   row for its category.

## 3 — Branch from upstream (never from fork main)

```bash
git fetch upstream
git checkout -b <family-slug> upstream/main
```

Use `-b` (fails if the branch exists), **never `-B`** (it would silently
destroy prior work). If the branch already exists: inspect it with
`git log <family-slug> --not upstream/main` — resume it if it's earlier WIP for
this family (check it out, `git rebase upstream/main`); delete it deliberately
only if it's confirmed abandoned. If git says the branch is checked out in
another worktree, that's a claim collision — go back to step 2.

Note: switching to an upstream-based branch removes fork-only files (including
this skill and `/close-out`) from the working tree — expected; these
instructions are self-contained and upstream's own `implement-cards` skill
remains in the tree. The Conductor workspace branch is left behind by design;
mention it in the close-out report.

## 4 — Plan (Batch Planner role)

Pull each card's effect text from `database.json`. Grep
`src/actions/effect_mechanic_map.rs`, `effect_ability_mechanic_map.rs`,
`shared_mutations.rs`, `src/actions/attacks/`, `src/actions/abilities/` and
classify the family: **map-only** (mechanic exists → add mappings) /
**parameter-extension** / **new-mechanic**. Prefer reusing an existing
mechanic over writing a new one — that's the codebase's DRY mechanism. Write
the batch spec: cards (including reprint ids the same effect clears), shared
code plan, one test per card behavior.

## 5 — Implement (TDD, upstream's conventions)

Follow the repo's `implement-cards` skill: tests FIRST, at the public `Game`
API level (model: `tests/tools/raikou_rocky_helmet_order_test.rs`), then the
implementation. Match upstream style; no drive-by refactors. Green bar:
`cargo test --features test-utils` + `cargo clippy`.

## 6 — Adversarial QA (Gate 0 — before the PR)

Spawn an independent QA reviewer agent instructed to REFUTE:
- Re-derive each card's expected behavior **from `database.json` text
  independently**, then check the tests assert that derivation — never judge
  tests by reading the implementation first.
- Every card in the batch has a focused test; edge cases considered (KO
  mid-effect, status interactions, zero targets, bench-full).
- **Fork-cleanliness gate:** `git diff upstream/main --stat` contains ONLY the
  card work — no `.claude/`, no tooling, no unrelated files.
- Full suite + clippy green.
Fix all findings before proceeding.

## 7 — PR to upstream

```bash
git push -u origin <family-slug>
gh pr create --repo bcollazo/deckgym-core \
  --title "Implement <effect name> (<N> cards)" \
  --body "<cards covered, mechanic approach, tests added>"
```

Title matches upstream's merged-PR convention (e.g. "Implement flip-until-tails
bonus-damage attacks (16 cards)"). Small, single-family PRs are the velocity
strategy — upstream review latency is the bottleneck, so be easy to approve.

## 8 — Close out

1. Queue row status → `PR#<n>`; commit + push umbrella main.
2. Regenerate the scorecard (`card-scorecard.sh --no-sync` is fine) + commit.
3. If meta coverage changed, update the Lane R row in
   `~/code/pokemon/docs/STATUS.md`.
4. Report the ✅/🚨 checklist with evidence (PR URL, test count, QA findings
   fixed). Then apply the ADR-0025 close-out gates — note the `/close-out`
   skill file left this worktree at step 3 (it lives on fork main, not
   upstream); its gates are: everything pushed → docs updated (this section
   already covers them) → kickoff brief if the lane's direction changed →
   evidence checklist.
