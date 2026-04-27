# AI Model Commit Trailer Enforcement

## Purpose / Big Picture
Every commit created through the repository harness should record whether an AI model was used. The required commit-message trailer is `AI-Model: <model-version>` when AI was used, or `AI-Model: none` when no AI model was used. This keeps provenance visible in normal Git history without adding a separate metadata store.

## Progress
- [x] 2026-04-27T16:12Z - Confirmed the repository currently has only a pre-commit hook and no commit-message enforcement.
- [x] 2026-04-27T16:33Z - Added commit-message hook validation and focused tests.
- [x] 2026-04-27T16:33Z - Updated harness documentation, npm scripts, and generated repository map.
- [x] 2026-04-27T16:33Z - Ran focused and broader verification.

## Surprises & Discoveries
- 2026-04-27: `README.md` is empty and no `package.json` exists, even though the guidance documents reference `npm run ...` commands.
- 2026-04-27: Adding `package.json` was necessary so the documented `npm run hooks:install` first-run path actually configures the new hook.

## Decision Log
- 2026-04-27: Use a Git trailer named `AI-Model` because trailers are parseable by Git tooling and fit commit-message footers.
- 2026-04-27: Enforce lowercase `none` exactly for non-AI commits so automation can distinguish it from arbitrary text.
- 2026-04-27: Require non-`none` values to include at least one digit, which catches generic labels such as `codex` while allowing model IDs like `gpt-5.5`, `gpt-5.3-codex`, and `claude-sonnet-4.5`.
- 2026-04-27: Keep hook validation dependency-free and test it with Node's built-in `node:assert/strict` module.

## Context and Orientation
The current hook installer is `scripts/install-git-hooks.mjs`; it sets `core.hooksPath` to `.githooks`. The existing hook wrapper is `.githooks/pre-commit`, which delegates to `scripts/git-hooks/pre-commit.mjs`.

This change will add `.githooks/commit-msg` and `scripts/git-hooks/commit-msg.mjs`. The `commit-msg` hook receives the commit-message file path from Git and exits non-zero when the required trailer is absent or invalid. Focused tests will live next to the hook logic in `scripts/git-hooks/commit-msg.test.mjs`.

## Milestones

### Milestone 1 - Commit-Message Enforcement
Files:
- `.githooks/commit-msg`
- `scripts/git-hooks/commit-msg.mjs`
- `scripts/git-hooks/commit-msg.test.mjs`
- `package.json`
- `package-lock.json`

Tasks:
1. Add a shell wrapper that delegates Git's `commit-msg` hook to Node.
2. Validate that the final commit-message trailer block contains exactly one `AI-Model` trailer.
3. Accept `AI-Model: none` or a model version value containing at least one digit.
4. Add focused tests for accepted, rejected, duplicate, and misplaced trailers.

Verification:
- `node scripts/git-hooks/commit-msg.test.mjs`

### Milestone 2 - Harness Documentation
Files:
- `AGENTS.md`
- `README.md`
- `.codex/local-environment.yaml`
- `.agents/DOCUMENTATION.md`
- `docs/QUALITY_SCORE.md`
- `docs/generated/repo-map.json`
- `docs/exec-plans/completed/2026-04-27-ai-model-commit-trailer.md`

Tasks:
1. Document the required commit-message trailer in the repository harness guidance.
2. Refresh generated repository map data so the new hook files are represented.
3. Update running implementation notes and this ExecPlan.

Verification:
- `python3 scripts/check_execplan.py`
- `python3 scripts/knowledge/check_docs.py`

## Acceptance / Verification
- Commits made through the configured hook path fail unless the commit message ends with exactly one valid `AI-Model` trailer.
- Valid examples include `AI-Model: none`, `AI-Model: gpt-5.5`, and `AI-Model: gpt-5.3-codex`.
- Invalid examples include a missing trailer, an empty trailer, `AI-Model: codex`, `AI-Model: None`, or duplicate `AI-Model` trailers.
- Focused command: `node scripts/git-hooks/commit-msg.test.mjs`
- Hook smoke command: create a temporary commit-message file and run `node scripts/git-hooks/commit-msg.mjs <file>`.
- Docs command: `python3 scripts/check_execplan.py`
- Docs command: `python3 scripts/knowledge/check_docs.py`
- Rollback: remove `.githooks/commit-msg`, `scripts/git-hooks/commit-msg.mjs`, and `scripts/git-hooks/commit-msg.test.mjs`, then refresh `docs/generated/repo-map.json`.

## Outcomes & Retrospective
The repository now has an executable `commit-msg` hook that requires exactly one `AI-Model` Git trailer. The documented npm harness path now exists, installs hooks, and exposes a focused `test:git-hooks` command. Verification passed with:
- `npm run hooks:install`
- `npm run test:git-hooks`
- valid and invalid direct `node scripts/git-hooks/commit-msg.mjs <file>` smoke checks
- `npm run lint`
- `npm run typecheck`
- `npm run test:unit`
- `npm run verify`
- `npm run verify:docs`
- `npm run verify:execplan`
