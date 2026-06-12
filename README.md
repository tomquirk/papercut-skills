# Paper Cut Skills

Agent skills for finding, fixing, verifying, and publishing small, low-risk maintenance PRs in a target codebase.

This package is designed for the Vercel `skills` CLI and skills.sh ecosystem. It contains four focused skills that turn recurring maintenance workflows into reusable agent instructions.

## Install

List the available skills before installing:

```bash
npx skills add tomquirk/papercut-skills --list
```

Install a single skill:

```bash
npx skills add tomquirk/papercut-skills --skill fix-code-paper-cut
```

Install the full package:

```bash
npx skills add tomquirk/papercut-skills --all
```

For local development, point the CLI at this checkout:

```bash
npx skills add . --list
```

## Skills

| Skill | Purpose |
| --- | --- |
| `fix-code-paper-cut` | Find one isolated code-quality paper cut, apply the smallest safe fix, verify it, and open a draft PR. |
| `fix-ux-paper-cut` | Inspect local web app surfaces, fix one screenshot-visible UX nuisance, capture before/after evidence, and open a draft PR. |
| `fix-sentry-exception-paper-cut` | Use read-only Sentry CLI discovery to select one recent production exception, handle it safely, verify it, and open a draft PR. |
| `fix-doc-drift` | Run a weekly docs drift sweep, update source-backed documentation only, verify the result, and open a draft PR. |

## Requirements

These skills expect the target repository to provide its normal local tooling and documentation.

- GitHub access for branch, commit, push, and draft PR creation.
- `pnpm pr` in the target repository for final verification.
- Existing project docs such as `AGENTS.md`, `doc/index.md`, and workflow-specific docs referenced by each skill.
- For `fix-ux-paper-cut`, a local app URL, normally `http://localhost:3000`, plus a browser-capable agent for screenshots.
- For `fix-sentry-exception-paper-cut`, authenticated `sentry-cli` read access and the correct `SENTRY_ORG` and `SENTRY_PROJECT` values.

## Safety

The skills are scoped to small, reviewable PRs. They instruct agents to preserve unrelated user work, avoid broad refactors, reject high-risk product areas, and keep production Sentry access read-only.

Review skills before installing them, especially any skill that can run commands in a target repository or publish PRs.

## Repository Layout

```text
fix-code-paper-cut/
  SKILL.md
  agents/openai.yaml
fix-doc-drift/
  SKILL.md
  agents/openai.yaml
fix-sentry-exception-paper-cut/
  SKILL.md
  agents/openai.yaml
fix-ux-paper-cut/
  SKILL.md
  agents/openai.yaml
```

Each skill follows the Agent Skills format: a directory named after the skill, a required `SKILL.md` with `name` and `description` frontmatter, and optional agent UI metadata.

## Publishing

There is no separate skills.sh publish command. To publish this package, push it to a public GitHub repo and install it with `npx skills add tomquirk/papercut-skills`. The package can then become discoverable through the skills.sh directory.

## License

MIT
