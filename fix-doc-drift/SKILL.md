---
name: fix-doc-drift
description: Find, fix, verify, and publish one weekly documentation-drift cleanup PR for the Nuggets repo. Use when Codex should compare docs under doc/ with current code, package scripts, routes, configuration, env schemas, tests, and ADRs; update stale or missing canonical explanations in a coherent docs-only batch; and skip the run when drift is not clearly evidenced.
---

# Fix Doc Drift

Use this skill to run a weekly documentation-drift sweep for Nuggets. Prefer source-backed corrections, canonical explanations, and a useful PR trail over broad rewriting.

## Weekly Contract

- Run at most once per week unless the user explicitly asks for an extra pass.
- Produce one coherent docs-only PR, not several unrelated cleanups.
- Fix multiple stale references when they share a theme, such as changed commands, moved routes, renamed env vars, outdated architecture notes, or stale setup steps.
- Skip the PR when the pass finds only subjective wording preferences, formatting churn, or drift that cannot be proven from the repo.

## Workflow

1. Prepare the repo.
   - Read `AGENTS.md`, `doc/index.md`, and any docs that appear related to likely drift.
   - Run `git status -sb` and preserve unrelated user work.
   - Stay in the current branch unless the user asks to switch.
   - Use Australian English.

2. Pick a drift theme.
   - Start from likely drift sources: `package.json` scripts, app routes, env schemas, deployment config, feature docs, ADRs, README-style setup notes, tests that encode expected behaviour, and recently changed files when git history is useful.
   - Compare docs against code and configuration with `rg`, targeted file reads, and narrow source inspection.
   - Choose one theme once evidence is clear. Examples: local setup commands, deployment flow, analytics events, route names, auth/data boundaries, env vars, billing behaviour, or feature-specific docs.
   - Avoid a repo-wide rewrite. A good weekly batch normally touches one to four closely related docs.

3. Reject unsafe or noisy work.
   - Do not change product code, tests, configuration, generated files, dependencies, migrations, permissions, payments, auth semantics, analytics meaning, or legal/marketing claims.
   - Do not invent product behaviour from intuition; use code, tests, config, existing docs, ADRs, or explicit user context as evidence.
   - Do not make style-only edits, large prose rewrites, glossary churn, or speculative future-state documentation.
   - If the code and docs disagree but the intended truth is unclear, open a draft PR only when the doc update can state the uncertainty and the required reviewer check plainly. Otherwise stop and report the ambiguity.

4. Update canonical docs.
   - Keep canonical explanations in `doc/`; use `AGENTS.md` only for editing rules.
   - Update `doc/index.md` when adding, removing, renaming, or materially relocating docs.
   - Prefer direct, precise wording with links to the relevant canonical page instead of repeating the same explanation in multiple places.
   - Keep the diff small and reviewable, even though the run is a weekly batch.

5. Verify.
   - Re-read the changed docs and the source files used as evidence.
   - Check links and paths touched by the edit with `rg` or targeted shell commands.
   - Run `pnpm pr` after changes. Add narrower checks only when the docs reference generated artefacts, examples, or commands that can be cheaply validated.
   - If verification is blocked, keep the PR draft-ready and record the blocker plus the exact next check.

6. Publish the PR.
   - Use the GitHub `yeet` workflow for branch, commit, push, and draft PR creation.
   - Stage only files that belong to the documentation-drift batch.
   - Prefer a branch name like `codex/fix-doc-drift-<theme>` and a terse commit message.
   - Create a draft PR unless the user explicitly asks for ready review.

## PR Description

Use a concise PR body with this shape:

```markdown
## Summary
- Fixes documentation drift around <theme>.
- Keeps the change contained to <docs/files/surface>.

## Evidence
- <Source file, config, test, command, or ADR that proves the updated doc is current.>
- <Second source if useful.>

## Verification
- `pnpm pr`
```

If a required check cannot run, state exactly what blocked it and what should be run next.
