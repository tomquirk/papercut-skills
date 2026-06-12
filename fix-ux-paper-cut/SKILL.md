---
name: fix-ux-paper-cut
description: Find, fix, verify, and publish one small low-risk UX paper cut in the Nuggets web app. Use when Codex should inspect localhost:3000 or another local app URL, including signup, sign-in, and authenticated app pages, choose an isolated visual, copy, layout, accessibility, or layout-shift nuisance, apply a contained frontend fix, capture before/after screenshots, run checks, and open a draft PR whose description includes a before/after screenshot table.
---

# Fix UX Paper Cut

Use this skill to ship exactly one tiny, screenshot-visible UX improvement from discovery through draft PR. Prefer judgement, restraint, and a clean paper trail over cleverness.

## Paper-Cut Ideas

- `fix-loading-state-paper-cut`: Improve one loading skeleton, spinner, or disabled state so the page does not jump, flicker, or feel broken.

## Workflow

1. Prepare the repo.
   - Read `AGENTS.md`, `doc/index.md`, and any relevant docs for the touched area before changing behaviour.
   - Run `git status -sb` and preserve unrelated user work.
   - Work against `http://localhost:3000` unless the user provides a different URL.
   - Use `browser:control-in-app-browser` for visual inspection and screenshots when available. For authentication, follow the repo browser-login guidance in `.agents/skills/agent-browser/SKILL.md`: try the saved local insider state first, then sign in as the documented local test user if needed.

2. Explore auth and app surfaces.
   - Start signed out. Visit `/signin`, `/signup`, and one pricing-to-signup path; step through forms far enough to inspect validation, loading, and success or verification states.
   - If creating a signup account is useful, use disposable local-only credentials and stop before external checkout, real payment, or non-local email-provider flows unless a documented local test path covers them.
   - Then load the saved state or sign in and sample a few authenticated app pages from the main navigation, such as browse, ask, account/settings, billing, organisation, admin, or available content pages.
   - Keep this as discovery, not broad QA. Stop exploring once one clear screenshot-visible paper cut appears.
   - Do not use production accounts, real customer data, or real payment details.

3. Find one candidate.
   - Inspect a small number of obvious surfaces, using desktop and mobile viewports when practical.
   - Watch for obvious layout shift during load, refresh, interaction, and viewport changes, especially content jumping after data, images, fonts, or auth state resolve.
   - Choose a candidate only when it is visible in a screenshot, easy to reproduce locally, and likely fixable in one small component, route, style, or copy change.
   - Prefer issues like poor contrast ratio, inconsistent colours, text truncation, cramped spacing, unclear microcopy, poor focus/hover state, awkward empty state layout, inconsistent alignment, missing accessible label, a small responsive layout defect, a visible layout shift, or one loading skeleton, spinner, or disabled state that makes the page feel jumpy, flickery, or broken.
   - Ignore button label casing by itself; do not choose title-case, sentence-case, or capitalisation-only changes as the paper cut.
   - Reject anything involving payments, auth semantics, permissions, data model changes, migrations, analytics meaning, broad design direction, legal/marketing copy, new dependencies, or more than a few closely related files.
   - Once a candidate passes the filter, stop hunting and commit to that one issue.

4. Capture the baseline.
   - Record the route, viewport, relevant user state, and minimal reproduction steps.
   - Note when the shift happens, such as initial load, refresh, tab switch, data arrival, image load, font swap, resize, or interaction.
   - Save a before screenshot outside the committed source tree, such as `tmp/ux-paper-cut/<slug>-before.png`.
   - If the screenshot contains sensitive data, choose a safer route/state or redact before using it in the PR.

5. Apply the smallest fix.
   - Follow existing component, styling, and naming patterns.
   - Keep the patch narrow; avoid opportunistic refactors.
   - Update docs only when behaviour or canonical product explanation changes.
   - Do not add dependencies unless the user explicitly approves and the repo health check is run afterwards.

6. Verify the result.
   - Reload the local app and revisit the exact same route, viewport, and state.
   - Save an after screenshot next to the before screenshot.
   - Compare the screenshots visually; if the improvement is not obvious, refine once or pick a smaller clearer candidate.
   - Re-check for layout shift using the same trigger from the baseline, and make sure the fix does not introduce a new jump elsewhere in the viewport.
   - Run `pnpm pr` after code changes. Add targeted checks like `pnpm typecheck` or `pnpm test` when the touched area warrants them.

7. Publish the PR.
   - Use the GitHub `yeet` workflow for branch, commit, push, and draft PR creation.
   - Stage only files that belong to this paper-cut fix.
   - Prefer a branch name like `codex/fix-<short-ux-issue>` and a terse commit message.
   - Create the draft PR with a temporary screenshot note if needed, then upload the screenshots using the GitHub screenshot upload procedure below and replace the PR body with the final description.
   - Do not leave local filesystem paths in the PR description. If image upload is blocked, keep the PR draft and tell the user what is needed.

## GitHub Screenshot Upload

GitHub does not provide a direct image upload API for PR bodies. Use the PR comment composer as a temporary upload target, extract the generated `https://github.com/user-attachments/assets/...` URLs, clear the unsent comment, then update the PR body with `gh pr edit`.

1. Resolve the PR and image paths.
   - Get the PR number and URL with `gh pr view --json number,url -q '"\(.number) \(.url)"'`.
   - Convert the before and after screenshot paths to absolute paths.
   - If a path contains awkward shell characters or screenshot-tool Unicode, copy it to `/tmp/<simple-name>.png` before upload.

2. Open the PR page with a browser tool that supports file upload and JavaScript evaluation.
   - Prefer the active browser automation available in the session; use `tool_search` to find browser upload tools when needed.
   - If GitHub asks for login or SSO, open the login flow for the user and wait for confirmation before continuing.

3. Upload both screenshots in the PR comment composer.
   - Scroll to the comment area and find a file input, trying selectors like `input[type="file"][id*="comment"]`, `input[type="file"][id="fc-new_comment_field"]`, then `input[type="file"]`.
   - Upload the before and after images to the same unsent comment, waiting a few seconds for GitHub to process them.
   - Read the comment textarea value with `document.getElementById('new_comment_field') || document.querySelector('textarea[id*="comment"]')`.
   - Extract the two `user-attachments/assets` URLs before clearing the textarea. Do not submit the comment.
   - If the textarea has not been populated yet, wait a few more seconds and retry once.

4. Update and verify the PR body.
   - Use the uploaded URLs in the screenshot table, preserving the route, viewport, and verification notes.
   - Update the draft PR with `gh pr edit <number> --body-file <body-file>` or the equivalent GitHub tool from the `yeet` workflow.
   - Reload the PR page and confirm the before/after images render in the description.

## PR Description

Use a concise PR body with this shape:

```markdown
## Summary
- Fixes <one-sentence UX paper cut>.
- Keeps the change contained to <file/component/surface>.

## Screenshots
Route: `<route>`  
Viewport: `<width>x<height>`  
Layout shift checked: `<trigger and result>`

| Before | After |
| --- | --- |
| <img src="<uploaded-before-image-url>" alt="Before: <short description>" width="480"> | <img src="<uploaded-after-image-url>" alt="After: <short description>" width="480"> |

## Verification
- `pnpm pr`
```

If a required check cannot run, keep the PR draft and state the blocker and next step in `Verification`.
