# Git Workflow

**Document ID:** SPEC-DLV-003 · **Stage:** 00 · **Status:** Approved v1.0
**Related:** ADR-0001, `AGENTS.md` §1

## 1. Identity (mandatory)

```
git config user.name  -> suryaprakashgiri01
git config user.email -> suryaprakashgiri0@gmail.com
```

Verify before the first commit in any clone. If wrong, fix repo config first, then global.
`gh` operations use the same account. Commits with any other author are rejected at review.

## 2. Branches

| Pattern | Use |
| --- | --- |
| `main` | protected, always green |
| `stage/NN-name` | one per stage (`stage/02-financial-engine`) |
| `fix/short-slug` | defects, merged back to the owning stage branch or `main` |
| `docs/short-slug` | documentation-only changes |
| `chore/short-slug` | tooling, CI, dependency bumps |

Rules:
- No direct commits to `main` after the initial bootstrap.
- Force-push allowed only on your own unreleased feature branch (`--force-with-lease`), never `main`.
- Rebase before merge to keep history linear on `main` (squash for noisy WIP, merge commit for stages).

## 3. Commit messages (Conventional Commits)

```
<type>(<scope>): <imperative summary, ≤ 72 chars>

<body: what changed and why, wrapped at 100>

Refs: FR-CALC-02, AC-CALC-01
```

`type` ∈ `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`, `chore`, `style`, `revert`.
`scope` ∈ `api`, `domain`, `engine`, `ai`, `memory`, `db`, `auth`, `web`, `infra`, `docs`, `evals`.

Examples:
```
feat(engine): add FD quarterly-compounding maturity calculation
test(engine): add golden fixtures for RD maturity cases
docs(architecture): add ADR-0004 deterministic-first numerics
fix(auth): revoke token family on refresh reuse
```

## 4. Commit atomicity

- One logical change per commit; the repo should be green at every commit on a stage branch where
  practical.
- Do not mix formatting-only changes with behaviour changes.
- Do not commit generated files, `.env`, coverage output, or `PROGRESS.local.md`.

## 5. Per-stage sequence

```
git switch main && git pull
git switch -c stage/NN-name
# work in small commits
# run: ruff, mypy, pytest, (eval if AI-related)
# stage report written
git switch main && git merge --no-ff stage/NN-name
```

## 6. Progress ledger

- `PROGRESS.local.md` tracks rollups on this machine only; it is listed in `.gitignore`.
- Update it **after** the commit it describes exists, never before.
- It records: branch, commit hash + subject, tests run, blockers, next action.

## 7. Releases / tags

- After Stage 08 (first public deploy) and Stage 09 (launch): annotated tag `v0.x.0`.
- Tags are immutable. A bad release gets a new tag, never a moved one.

## 8. Review checklist (self-review is mandatory)

1. Diff contains only in-scope files.
2. Spec IDs referenced.
3. Tests added and executed; output pasted in the task report.
4. No secrets, no real financial data, no debug prints.
5. Migrations reversible-in-practice (or explicitly documented as forward-only).
6. Docs updated where behaviour changed.

## 9. History hygiene

- Never rewrite `main`.
- Never rebase a branch someone else has reviewed without noting it.
- Wrong author on a feature branch:
  `git rebase <merge-base> --exec "git commit --amend --reset-author --no-edit"` then force-push that
  branch only.
