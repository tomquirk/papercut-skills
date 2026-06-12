---
name: fix-code-paper-cut
description: Find, fix, verify, and publish one small low-risk code paper cut, excluding typo-only cleanup. Use when Codex should scan a codebase for an obvious isolated code smell, tiny duplicated code, misleading variable or function name, redundant branch, stale comment, local type cleanup, or similarly contained maintainability issue, then make the smallest safe patch and open a draft PR.
---

# Fix Code Paper Cut

Use this skill to ship exactly one tiny code-quality improvement from discovery through draft PR. Prefer boring, obvious, locally provable fixes over clever refactors.

## Workflow

1. Prepare the repo.
   - Read `AGENTS.md`, `doc/index.md`, and any docs relevant to the area before behaviour changes.
   - Run `git status -sb` and preserve unrelated user work.
   - Inspect the existing patterns in the touched package or module before editing.
   - If the worktree is already on a feature branch, avoid switching branches unless the user asks.

2. Find one candidate.
   - Search narrowly for obvious signals: duplicate local helper logic, repeated literals in the same module, misleading names, redundant conditions, stale comments, trivially dead local code, unnecessary wrappers, or small type cleanups.
   - Prefer candidates contained to one file or a few closely related files with existing tests or easy static verification.
   - Use `rg`, `git grep`, type errors, lint output, or quick source inspection. Do not run broad mechanical rewrites.
   - Do not select typo-only fixes in comments, docs, strings, identifiers, tests, or file names. Typos are out of scope unless they must change as part of the selected code-quality fix.
   - Once one candidate is safe and worthwhile, stop hunting and commit to that single issue.
   - If no clear candidate appears after a bounded pass, stop and report that no safe paper cut was found rather than forcing a noisy PR.

3. Reject risky work.
   - Reject anything involving product behaviour changes, auth, permissions, payments, data model changes, migrations, analytics meaning, dependency changes, generated files, build configuration, broad abstractions, public API renames, cross-package architecture, or more than a small local diff.
   - Reject changes that require guessing business meaning or domain terminology.
   - Reject formatting-only churn unless it is limited to files already touched for the fix.

4. Apply the smallest fix.
   - Follow existing naming, typing, testing, and module patterns.
   - Keep the patch narrow and direct; do not opportunistically refactor neighbouring code.
   - Update docs only when behaviour or canonical explanation changes.
   - Add or update targeted tests only when the change touches behaviour, shared helpers, or a riskier branch.

5. Verify.
   - Run the narrowest useful check first when it helps confidence.
   - Run `pnpm pr` after code changes when the repo uses that command.
   - Run a quick smoke test of the affected area when local setup supports it. Prefer the smallest realistic manual, browser, API, or CLI check that exercises the touched surface.
   - If local setup is not sufficient for a smoke test, do not fake it. Record the blocker and the exact area, route, flow, or state that reviewers should smoke test in the PR description.
   - If verification fails, fix failures caused by the patch and rerun once.
   - If verification is blocked by the environment, keep the PR draft-ready and record the blocker and next step.

6. Publish the PR.
   - Use the GitHub `yeet` workflow for branch, commit, push, and draft PR creation.
   - Stage only files that belong to this one cleanup.
   - Prefer a branch name like `codex/fix-<short-code-issue>` and a terse commit message.
   - Create a draft PR unless the user explicitly asks for ready review.
   - When the PR needs screenshots or visual evidence, upload local images through the GitHub PR comment file input to get hosted `https://github.com/user-attachments/assets/...` URLs. Do not commit screenshot artefacts to the product repository just to make the PR body render.

## PR Screenshots

When adding screenshots to a GitHub PR:

1. Capture screenshots outside the committed source tree, using absolute paths such as `/tmp/before.png` and `/tmp/after.png`.
2. Open the PR page in an authenticated browser session and locate the comment form file input. Prefer browser MCP tools when available; otherwise use `agent-browser --headed --profile ~/.agent-browser-github`.
3. Upload each image to the comment form, wait for GitHub to process the files, then read the comment textarea value. GitHub inserts Markdown like `![image](https://github.com/user-attachments/assets/<id>)`.
4. Copy the generated `user-attachments/assets` URLs, clear the draft comment textarea, and do not submit the staging comment.
5. Update the PR body with the hosted image URLs, using a before/after table or `<img width="...">` tags as appropriate.
6. Reload the PR page or inspect the body to confirm the images render.

If the upload fails, first check that the browser is authenticated, the file paths are absolute, and enough time has passed for GitHub to replace the local upload marker with hosted URLs.

## PR Description

Use a concise PR body with this shape:

```markdown
## Summary
- Fixes <one obvious code paper cut>.
- Keeps the change contained to <file/module/surface>.

## Why
- <One sentence explaining why the previous code was noisy, duplicated, misleading, or unnecessarily risky.>

## Verification
- `pnpm pr`
- Smoke test: <what was exercised locally, or `Not run: <local setup blocker>. Review smoke test should cover <area/route/flow/state>.`>
```

If a required check or smoke test cannot run, say exactly what blocked it and what should be run next or manually exercised during review.
