---
name: raise-pr
description: Use whenever opening or updating a pull request in MoneyMap — prefilled template, branch/base rules, verification gate and one-command gh flow for this repo. Avoids re-deriving values each time and saves tokens.
---

# Raise a PR — MoneyMap (prefilled, one flow)

Use this every time a PR is opened or updated in this repo. Do not re-derive anything below; it is
prefilled for MoneyMap. Keep bodies short — reviewers read the first 10 lines.

## Hard rules (settled; do not re-ask)

- Identity: commits and all `gh` ops as **suryaprakashgiri01** / `suryaprakashgiri0@gmail.com`.
- Base branch: **`main`**. `main` is **restricted**: PR-only, no direct pushes, no force-push,
  no deletion. If a push to `main` is rejected, that is correct behaviour — open a PR instead.
- Head branch: one purpose, `stage/NN-name`, `fix/…`, `docs/…`, `chore/…`.
- **Never pass `--body` inline in PowerShell** — backticks are PowerShell escapes and mangle
  markdown. Always `--body-file <temp.md>` (write file → use → delete).
- One PR = one subject. Unrelated cleanup is a separate tiny PR.
- Delete every merged branch immediately (remote + local) and never reuse its name — see
  **Merging** below.
- Never merge a PR whose checks are failing or whose gate was not actually executed.
- Update `PROGRESS.local.md` (git-ignored) **only after** the merge commit exists.

## Gate (run before pushing)

```powershell
# Stage 00 and docs-only changes:
#   1. relative markdown links resolve
#   2. no empty stubs
# Code stages (01+) replace this with:
#   make check    # ruff, ruff format, mypy --strict, pytest
```

Docs link check (no dependencies):

```powershell
$missing=@()
Get-ChildItem -Recurse -Filter *.md | Where-Object { $_.Length -gt 0 } | ForEach-Object {
  $f=$_; $txt=[string](Get-Content $f.FullName -Raw)
  [regex]::Matches($txt,'\]\(([^)#][^)]*\.md)\)') | ForEach-Object {
    if (-not (Test-Path (Join-Path $f.DirectoryName $_.Groups[1].Value))) {
      $missing += "$($f.Name) -> $($_.Groups[1].Value)"
    }
  }
}
if ($missing.Count -eq 0) { "OK: links resolve" } else { $missing }
```

## Template (fill placeholders, keep sections)

```markdown
## What
<n–4 bullets max; bold file paths / endpoints / commands>

## Why / verification
<spec IDs (FR-*/NFR-*/AC-*/ADR-*), and exactly what was executed: commands + result>

## Checklist
- [ ] Gate executed locally (docs link check, or `make check` from Stage 01)
- [ ] Tests added/updated where behaviour changed; output pasted above
- [ ] No secrets, no real financial data, no `.env*`, no `PROGRESS.local.md`
- [ ] Spec IDs referenced; docs updated if behaviour/assumptions changed
- [ ] Diff contains only in-scope files
- [ ] Review order given below (riskiest first)

## Review order
1. <file — one line why/risk>
2. <…>
```

## Flow (Windows PowerShell)

```powershell
# 0. sanity
git config user.name; git config user.email     # must be suryaprakashgiri01 / gmail
git status --short                              # must be clean before branching

# 1. branch, gate, commit
git switch main; git pull --ff-only
git switch -c <type>/<subject>
# ... change files ...
# run the gate above
git add <explicit paths>
git commit -m "<type>(<scope>): <imperative subject>"

# 2. push and open
git push -u origin HEAD
$body = "$env:TEMP\moneymap-pr-body.md"
# write the template into $body (write tool, not echo with backticks)
gh pr create --base main --head <branch> --title "<type>: <subject>" --body-file $body
Remove-Item $body

# 3. report the URL + initial check status only (never poll repeatedly)
gh pr checks <number>
```

## Merging

- Merge through the PR only (`gh pr merge <n> --squash` or `--merge`), never `git push origin main`.
- Default merge style: **squash** for `fix/`, `docs/`, `chore/`; **merge commit** (`--merge`) for
  `stage/NN-*` so the stage boundary stays visible in history.
- **Delete the branch immediately after merge**, remote then local:

```powershell
gh pr merge <n> --merge          # or --squash
git switch main; git pull --ff-only
git push origin --delete <branch>
git branch -d <branch>           # -D only if the branch was rebased/squashed upstream
```

- **Never reuse a merged branch name.** Post-merge work starts from `main` on a fresh branch and gets
  its own PR. A commit pushed to an already-merged branch never lands on `main`.
- Do not keep merged branches "just in case" and do not tag stage branches; the merge commit, the PR
  and the stage report are the record. Tags are for releases (`vX.Y.Z`).
- After merge: confirm `main` is green, then update `PROGRESS.local.md`.

## Repo-specific notes

- Repo: `suryaprakashgiri01/MoneyMap` (remote `origin`).
- Branch protection on `main`: required PR, no direct pushes, no force pushes, no deletions.
- Stage branches are long-lived for the duration of a stage; each PR from a stage branch should still
  be one reviewable subject. If a stage branch accumulates unrelated work, split the PRs.
- Commit author must be verified again after any tool that rewrites commits
  (`git rebase --exec "git commit --amend --reset-author --no-edit"` if needed).
- No secrets are ever needed to open a PR; if a command asks for a token, stop.
