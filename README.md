# MoneyMap

> Understand your money. Compare your options. Plan your next move.

MoneyMap is an educational financial planning and scenario-analysis tool for individuals. It computes
your salary-to-expense picture deterministically, compares savings and investment products on more than
advertised return, and explains the results in plain language — without inventing numbers.

**It is not an investment adviser.** It does not execute transactions, guarantee returns, or give
personalised regulated advice. See [`docs/product/non-goals-and-disclaimers.md`](docs/product/non-goals-and-disclaimers.md).

## Status

**Stage 00 — specification complete.** No application code exists yet; implementation begins at Stage 01.

| Stage | Branch | Status |
| --- | --- | --- |
| 00 Product spec | `stage/00-product-spec` | ✅ complete |
| 01 Backend foundation | `stage/01-backend-foundation` | ⬜ not started |
| 02 Financial engine | `stage/02-financial-engine` | ⬜ |
| 03 Profile & products | `stage/03-user-financial-profile` | ⬜ |
| 04 LLM gateway | `stage/04-llm-gateway` | ⬜ |
| 05 Memory | `stage/05-ai-memory` | ⬜ |
| 06 Guardrails & evals | `stage/06-guardrails-evals` | ⬜ |
| 07 Observability | `stage/07-observability` | ⬜ |
| 08 Frontend & deploy | `stage/08-frontend-deployment` | ⬜ |
| 09 Hardening | `stage/09-production-hardening` | ⬜ |

## What it will do

- **Profile** — monthly income (salaried or variable), fixed and variable expenses, dependants.
- **Metrics** — surplus, savings ratio, expense ratio, emergency-fund target and coverage.
- **Goals** — target amount, date, required monthly contribution, feasibility.
- **Products** — FD, RD, PPF, Post Office schemes, mutual fund SIP, LIC policy, with dated and
  source-cited rates.
- **Comparisons** — return, risk, liquidity, lock-in, tax treatment, horizon on a normalised basis.
- **Scenarios** — re-allocate the surplus, see deterministic deltas, without touching real records.
- **Explanations** — an LLM narrates a server-computed result; every number it states must exist in
  that result.

## Design commitments

| Commitment | Where |
| --- | --- |
| LLM never computes money | [ADR-0004](docs/architecture/decisions/ADR-0004-deterministic-first.md) |
| `Decimal` everywhere, exact golden tests | [ADR-0003](docs/architecture/decisions/ADR-0003-decimal-money.md) |
| Provider-independent AI gateway | [ADR-0005](docs/architecture/decisions/ADR-0005-model-gateway.md) |
| Org-scoped tenancy from day one | [ADR-0010](docs/architecture/decisions/ADR-0010-org-tenancy.md) |
| Rates are dated, sourced, and refusable | [ADR-0008](docs/architecture/decisions/ADR-0008-rate-dataset.md) |
| Structured memory, no vectors | [ADR-0007](docs/architecture/decisions/ADR-0007-structured-memory.md) |
| No PII or amounts in telemetry | [ADR-0013](docs/architecture/decisions/ADR-0013-observability.md) |

## Repository map

```
docs/
  product/        problem, scope, personas, stories, metrics, disclaimers
  requirements/   functional + non-functional requirements, acceptance criteria
  architecture/   system design, data model, AI workflow, memory, ADRs
  security/       threat model, privacy and data handling
  quality/        test strategy, evaluation strategy, definition of done
  delivery/       deployment, stage roadmap, git workflow, per-stage tasks
backend/          FastAPI application (Stages 01+)
frontend/         Next.js application (Stage 08)
infra/            compose, seed data, deployment config
tests/            cross-cutting tests (integration, security, e2e)
```

## Local development (available from Stage 01)

```bash
cp .env.example .env          # fill in secrets locally
docker compose up -d          # api + postgres
uv run alembic upgrade head
uv run python -m app.infra.seed
uv run pytest
```

Frontend:

```bash
cd frontend && npm ci && npm run dev
```

## Working on this repo

1. Read [`AGENTS.md`](AGENTS.md) — binding rules for agents and humans.
2. Read [`docs/delivery/git-workflow.md`](docs/delivery/git-workflow.md).
3. Every change references a spec ID; every stage ends with a report and a gate check.

## Known limitations (by design)

- INR only; no multi-currency.
- No tax computation (treatment notes only).
- No live rate feeds — rates are manually verified and dated.
- No bank aggregation or document upload.
- No chat interface; explanations attach to computed results.
- Single-region, best-effort availability for the MVP.

## Licence

TBD before public release.
