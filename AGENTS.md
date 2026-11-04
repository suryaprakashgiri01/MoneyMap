# MoneyMap — Agent & Contributor Rules

These rules apply to any AI agent or human contributor working in this repository.
They are binding. Read `docs/delivery/git-workflow.md` and `docs/quality/definition-of-done.md`
before submitting work.

## 1. Git identity (mandatory)

- Commits in this repository MUST be authored by:
  - `user.name` = `suryaprakashgiri01`
  - `user.email` = `suryaprakashgiri0@gmail.com`
- Never use, set, or fall back to any other identity.
- Verify with `git config user.name` / `git config user.email` before the first commit.
  If they differ, fix repo config first, then global config.
- All GitHub operations (`gh`, push, PR) come from the `suryaprakashgiri01` account only.
- If commits were authored under a wrong identity on a feature branch:
  `git rebase <merge-base> --exec "git commit --amend --reset-author --no-edit"` then force-push
  to that feature branch only.

## 2. Specification first

- No feature code without an approved specification in `docs/`.
- Every implementation task references its specification ID(s) (`FR-xxx`, `NFR-xxx`, `ADR-xxxx`).
- Do not silently change product scope. Scope changes require an explicit spec update + ADR.

## 3. Deterministic money

- The LLM never computes financial values. All numbers come from the Python financial engine.
- Money uses `Decimal`. Floats are forbidden in domain and financial code.
- Every computed response carries provenance: `provided` | `sourced` | `assumed` | `calculated` | `generated`.

## 4. Security & privacy

- Never commit secrets, real bank statements, or personal financial data.
- All fixtures and seeds use synthetic data.
- Tenant isolation (`org_id`) is enforced in the data-access layer and covered by tests.

## 5. Tooling discipline

- Prefer `uv` for Python environments and dependency management.
- Use `rtk` for noisy shell commands when available.
- Use `graphify` for repository/dependency exploration instead of bulk file reads.
- Do not invent commands for tools that are not installed. `laaya` is NOT installed.

## 6. Honest reporting

- Never claim a test passed unless it was executed in this session.
- Report: changed files, tests run + results, known limitations, follow-ups.
- If something is blocked, say so plainly instead of fabricating a result.

## 7. Progress tracking

- Local progress is tracked in `PROGRESS.local.md` (git-ignored).
- Update it **only after** a commit exists for the completed work.
