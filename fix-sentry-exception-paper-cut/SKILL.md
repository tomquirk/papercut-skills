---
name: fix-sentry-exception-paper-cut
description: Analyse recent production Sentry errors with sentry-cli, then fix and publish exactly one isolated unhandled error paper cut. Use when Codex should inspect Sentry issues or events for a target web app, choose a small current exception, handle it with a clear user-facing failure path or structured internal logging, run verification, and open a new draft PR without hiding unexpected bugs.
---

# Fix Sentry Exception Paper Cut

Use this skill to turn one recent production Sentry exception into a small, verified draft PR. Treat Sentry telemetry as the discovery source, but keep Sentry commands read-only.

## Project

- Sentry CLI: prefer `pnpm exec sentry-cli`; fall back to `node_modules/.pnpm/node_modules/.bin/sentry-cli` only if needed.
- Sentry environment: `production`
- Web Sentry config: `apps/web/src/lib/sentry/`
- Release naming: `SENTRY_RELEASE` is derived from `COMMIT_SHA`; see `doc/dev/deployments.md`.
- Web logger: `@/utils/server/logger`
- Background jobs logger: `@/lib/logger`
- User-facing action failures: `ActionError`
- JSON route failures: `ApiRouteError` with `jsonError(...)`

## Workflow

1. Prepare the repo and docs context.
   - Read `AGENTS.md`, `doc/index.md`, `doc/dev/logging-and-analytics.md`, `doc/dev/user-facing-errors.md`, `doc/dev/web-patterns.md`, and `doc/dev/deployments.md`.
   - Run `git status -sb` and preserve unrelated user work.
   - Create temporary investigation files only under `tmp/sentry-errors/`; do not commit them.

2. Confirm read access and the Sentry project.
   - Run:
     ```bash
     SENTRY_CLI="${SENTRY_CLI:-pnpm exec sentry-cli}"
     $SENTRY_CLI --version
     $SENTRY_CLI info --no-defaults
     $SENTRY_CLI organizations list
     ```
   - Set `SENTRY_ORG` and `SENTRY_PROJECT` from the user's environment, Sentry CLI defaults, or `projects list` output:
     ```bash
     : "${SENTRY_ORG:?Set SENTRY_ORG to the Sentry organisation slug}"
     $SENTRY_CLI projects list --org "$SENTRY_ORG"
     : "${SENTRY_PROJECT:?Set SENTRY_PROJECT to the Sentry project slug}"
     ```
   - If authentication, token scope, organisation lookup, or project lookup fails, stop and tell the user exactly what is missing. Do not run `sentry-cli login`, install components, change Sentry config, or create tokens unless the user asks.
   - Do not run mutating Sentry commands such as `issues resolve`, `issues mute`, `issues unresolve`, `send-event`, release writes, source-map uploads, or code-mapping updates.

3. Pull recent production issue and event signals.
   - Start with unresolved production errors from the last 48 hours:
     ```bash
     mkdir -p tmp/sentry-errors
     $SENTRY_CLI issues list \
       --org "$SENTRY_ORG" \
       --project "$SENTRY_PROJECT" \
       --query "is:unresolved environment:production level:error lastSeen:-48h" \
       --max-rows 100 \
       --pages 1 > tmp/sentry-errors/recent-issues.txt
     ```
   - If Sentry rejects the relative `lastSeen` filter, rerun with `is:unresolved environment:production level:error` and use the displayed last-seen timestamps manually.
   - Pull recent events with tags for route, release, transaction, environment, and runtime clues:
     ```bash
     $SENTRY_CLI events list \
       --org "$SENTRY_ORG" \
       --project "$SENTRY_PROJECT" \
       --show-tags \
       --max-rows 100 \
       --pages 1 > tmp/sentry-errors/recent-events.txt
     ```
   - If this is empty, widen the issue query to 7 days or relax to `is:unresolved environment:production`, while still preferring error-level issues.
   - Do not use `--show-user` unless user context is essential for grouping; never paste private emails, names, IP addresses, raw URLs with sensitive query strings, cookies, tokens, or request bodies into the final answer or PR body.

4. Summarise and group before editing.
   - Group by issue ID, title/message, culprit, transaction or route, release, environment, event count, user count, newest timestamp, and stack top frame if available.
   - For a promising candidate, capture a focused issue row:
     ```bash
     ISSUE_ID="<sentry-issue-id>"
     $SENTRY_CLI issues list \
       --org "$SENTRY_ORG" \
       --project "$SENTRY_PROJECT" \
       --id "$ISSUE_ID" \
       --max-rows 10 > "tmp/sentry-errors/issue-$ISSUE_ID.txt"
     ```
   - If sentry-cli output lacks the stack frame needed to locate code, use the Sentry issue page only for read-only stack inspection. Keep sentry-cli as the discovery source and do not copy sensitive payloads into local notes, the final answer, or the PR description.

5. Pick exactly one isolated candidate.
   - Choose an error with a clear title, culprit, transaction, stack frame, release, or source path, recent production occurrence, and a fix likely contained to one route, action, component, job, or data helper.
   - Prefer unhandled expected failures: invalid user input, missing records, permission checks, expired state, external service recoverable failures, parse errors, absent optional data, or client paths where the app should show a graceful state.
   - Reject broad incidents, release/source-map configuration work, migrations, payments or auth semantics that need product judgement, suspected infrastructure outages, data corruption, browser-extension noise, third-party script errors without actionable local context, and anything that requires production writes to diagnose.
   - Stop hunting once one candidate passes the filter.

6. Trace the code path.
   - Use Sentry issue title, culprit, stack frames, transaction, route, release, tags, and nearby source files to locate code with `rg`.
   - Read the smallest relevant surrounding files and existing tests before changing code.
   - Reproduce locally or in a unit test when practical. If the production event cannot be reproduced safely, write a focused test for the expected failure branch.
   - If the Sentry release points to a deployed commit, use it only to confirm recency and likely affected code; do not assume the issue is fixed until deployed telemetry confirms it.

7. Handle the error appropriately.
   - If the failure is expected and user-actionable, convert it into a user-facing message:
     - Server actions: throw `ActionError` with a specific sentence case message.
     - JSON routes: throw `ApiRouteError` and return through `jsonError(...)`.
     - UI loaders or pages: render an existing empty, not-found, or permission state where that is the established pattern.
   - If the failure is not user-actionable but should not report as an exception, log it with structured, non-PII context and return or skip only when the caller can continue safely.
   - Keep genuinely unexpected failures as errors. Do not add broad catch blocks that mask bugs, return fake success, swallow failed writes, or call `Sentry.captureException` for already-handled expected outcomes.
   - Follow `doc/dev/user-facing-errors.md`: avoid vague messages, status-code-only messages, and raw exception text.
   - Follow `doc/dev/logging-and-analytics.md`: use one structured event, include useful context, and never log secrets or PII.

8. Verify.
   - Run focused tests for the touched code where available.
   - Run `pnpm pr` after code changes.
   - If verification is blocked, state the command, failure reason, and next step.
   - Do not claim the Sentry issue is resolved until the fix has deployed and fresh Sentry telemetry confirms it.

9. Publish the PR.
   - Use the GitHub `yeet` workflow for branch, commit, push, and draft PR creation.
   - Stage only files that belong to this Sentry exception paper-cut fix. Do not stage temporary Sentry output, unrelated worktree changes, or other untracked skill folders.
   - Prefer a branch name like `codex/fix-<short-exception-cause>` and a terse commit message.
   - Open a new draft PR after verification passes. If verification is blocked, still open a draft PR only when the user has enough context to review the partial fix; otherwise stop and report the blocker.
   - Do not paste sensitive production payloads, private emails, names, IP addresses, tokens, cookies, raw stack dumps, raw request URLs, or local filesystem paths into the PR description.

10. Handoff.
   - Report the sentry-cli query window, selected issue summary, root cause, files changed, user-facing or logging behaviour added, verification commands, and draft PR URL.
   - Include residual risk, especially when production reproduction was not possible or the fix needs deployment before Sentry can confirm resolution.

## PR Description

Use a concise PR body with this shape:

```markdown
## Summary
- Handles one recent production Sentry exception from `<service/action/route/component>`.
- Converts `<root cause>` into `<user-facing message or structured log/skip behaviour>`.

## Production signal
- Source: Sentry via `sentry-cli`
- Org/project: `<org>/<project>`
- Query/window: `<query terms or timestamp range>`
- Issue: `<sanitised issue ID/title, count, newest timestamp or last seen>`

## Verification
- `<focused test command if run>`
- `pnpm pr`

## Residual risk
- `<deployment or telemetry confirmation still needed, or "None known">`
```
