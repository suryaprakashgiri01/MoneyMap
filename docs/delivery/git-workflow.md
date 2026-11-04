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
- `main` is **protected**: PR-only, no direct pushes, no force-push, no deletion.
  Every change reaches `main` through a pull request — including documentation and chore commits.
- Force-push allowed only on your own unreleased feature branch (`--force-with-lease`), never `main`.
- PR flow, template and gate: see the project skill `.opencode/skills/raise-pr/SKILL.md`.
- **Delete a merged branch immediately** — remote and local
  (`git push origin --delete <branch>` then `git branch -d <branch>`). The merge commit and the PR
  are the permanent record; the branch itself is disposable.
- **Never reuse a merged branch name.** A post-merge fix starts from `main` on a fresh
  `fix/…` or `docs/…` branch so it gets its own PR and a clean diff. This prevents work committed to
  an already-merged branch from silently never reaching `main`.
- Do not tag stage branches. Tags are reserved for releases (`vX.Y.Z`, §7).
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
git switch main && git pull --ff-only
git switch -c stage/NN-name
# work in small commits
# run the stage gate: ruff, mypy, pytest, (eval if AI-related)
# stage report written
git push -u origin HEAD
gh pr create --base main --body-file <temp.md>      # see .opencode/skills/raise-pr/SKILL.md
# after approval: merge through the PR (--merge for stage/NN-*, --squash for fix/docs/chore)
#   gh pr merge <n> --merge
# then delete the branch, remote and local (§2)
```

`main` is never merged into locally with `git merge`; the merge happens on the PR so protection,
history and the audit trail stay in one place.

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
