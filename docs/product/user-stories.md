# User Stories

**Document ID:** SPEC-PROD-004 · **Stage:** 00 · **Status:** Approved v1.0

Format: `US-<epic><n>` — *As a … I want … so that …* · Acceptance is owned by the referenced FR
in `docs/requirements/functional-requirements.md`.

Epics: **A** Account, **B** Profile & metrics, **C** Goals & emergency fund, **D** Products &
holdings, **E** Comparison, **F** Scenarios, **G** AI explanation, **H** Memory & privacy,
**I** Transparency & feedback.

## A — Account

- **US-A1** As a new user I want to create an account with email and password so that my financial
  data is private to me. → FR-AUTH-01
- **US-A2** As a returning user I want to log in and stay logged in safely so that I do not
  re-enter credentials constantly. → FR-AUTH-02
- **US-A3** As a user I want to log out of all devices so that a lost device cannot access my data.
  → FR-AUTH-04
- **US-A4** As a user I want to delete my account and all my financial data so that I retain control.
  → FR-PRIV-03

## B — Profile & metrics

- **US-B1** As a user I want to record my monthly net income so that my surplus is computed from
  reality, not an assumption. → FR-PROF-01
- **US-B2** As a freelancer I want to describe irregular income so that my metrics are not based on
  a single lucky month. → FR-PROF-02
- **US-B3** As a user I want my fixed and variable expenses separated so that I can see what I can
  actually cut. → FR-PROF-03
- **US-B4** As a user I want to see my monthly surplus, savings ratio and expense ratio so that I
  understand my position at a glance. → FR-METR-01..03
- **US-B5** As a user I want to see how each metric was computed so that I trust it. → FR-TRAN-01

## C — Goals & emergency fund

- **US-C1** As a user I want an emergency fund target based on my essential expenses so that I know
  my real buffer requirement. → FR-EMRG-01..02
- **US-C2** As a user I want to see how many months my liquid assets cover so that I know my gap.
  → FR-EMRG-03
- **US-C3** As a user I want to define a financial goal with target amount and date so that saving
  becomes a directed activity. → FR-GOAL-01
- **US-C4** As a user I want to see required vs current monthly contribution for a goal so that I
  know if my plan is feasible. → FR-GOAL-03

## D — Products & holdings

- **US-D1** As a user I want to record an existing FD/RD/PPF/Post Office/SIP/LIC product with its
  real dates and amounts so that analysis uses facts, not guesses. → FR-HOLD-01
- **US-D2** As a user I want the product catalogue to show which rates are sourced and as of when
  so that I know how current the comparison is. → FR-RATE-02
- **US-D3** As a user I want to edit or delete a holding so that my records stay accurate.
  → FR-HOLD-03
- **US-D4** As a user with an LIC policy I want its true return computed from my premiums and
  maturity so that I can see through the advertised rate. → FR-LIC-01..03

## E — Comparison

- **US-E1** As a user I want to compare candidate products on return, risk, liquidity, lock-in, tax
  treatment and horizon so that I can judge them on more than rate. → FR-CMP-01..02
- **US-E2** As a user I want comparisons normalised to the same horizon and contribution so that
  the difference is meaningful. → FR-CMP-03
- **US-E3** As a user I want to know when a comparison is incomplete because a rate is stale or
  missing so that I do not act on a partial picture. → FR-TRAN-03

## F — Scenarios

- **US-F1** As a user I want to simulate moving ₹X/month from product A to product B so that I see
  the deterministic impact before committing. → FR-SCN-01..02
- **US-F2** As a user I want scenarios to never mutate my real holdings so that experiments are safe.
  → FR-SCN-03
- **US-F3** As a user I want to compare two scenarios side by side so that I can choose. → FR-SCN-04

## G — AI explanation

- **US-G1** As a user I want a plain-language explanation of my computed result so that I understand
  what the numbers mean. → FR-AI-01
- **US-G2** As a user I want the explanation to quote only numbers the engine produced so that I can
  verify every claim. → FR-AI-03
- **US-G3** As a user I want to know when the assistant cannot answer from my data so that I am not
  given invented figures. → FR-AI-05
- **US-G4** As a user I want the explanation to state that this is educational, not advice.
  → FR-DISC-01

## H — Memory & privacy

- **US-H1** As a user I want to choose what is remembered about me so that I control my profile.
  → FR-MEM-02
- **US-H2** As a user I want to view and delete anything remembered so that I can correct the record.
  → FR-MEM-03
- **US-H3** As a user I want my data invisible to any other user so that my finances stay private.
  → FR-TEN-01

## I — Transparency & feedback

- **US-I1** As a user I want to see the assumptions behind a calculation so that I can challenge them.
  → FR-TRAN-02
- **US-I2** As a user I want to flag a wrong or confusing answer so that the product improves.
  → FR-FDBK-01

## Traceability rule

Every FR must map to ≥1 user story; every user story must map to ≥1 FR and ≥1 test case in
`docs/quality/test-strategy.md`. Unmapped items are defects, enforced in Stage 00 review and
re-checked at each stage gate.
