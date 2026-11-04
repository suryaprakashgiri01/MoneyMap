# Product Scope

**Document ID:** SPEC-PROD-002 · **Stage:** 00 · **Status:** Approved v1.0

## 1. Positioning

MoneyMap is an **educational financial planning and scenario-analysis tool** for individuals.
It explains and compares; it does not advise, execute, or guarantee.

## 2. In scope (MVP = Stages 01–09)

| # | Capability | Notes |
| --- | --- | --- |
| S1 | Account creation, login, session management | Email + password, personal workspace |
| S2 | Financial profile | Monthly gross/net income, fixed expenses, variable expenses, dependants, currency = INR |
| S3 | Savings metrics | Monthly surplus, savings ratio, expense ratio |
| S4 | Emergency fund | Target = N months of essential expenses (N user-set, default 6) |
| S5 | Goals | Name, target amount, target date, monthly contribution, progress |
| S6 | Product catalogue | FD, RD, PPF, Post Office (TD/RD/MIS/SCSS), Mutual fund SIP, LIC policy |
| S7 | Existing holdings | User-recorded products with dates and amounts |
| S8 | Deterministic engine | Future value, maturity, XIRR, comparison metrics |
| S9 | Comparison view | Normalised table across return/risk/liquidity/lock-in/tax/horizon |
| S10 | LIC analysis | User-supplied policy inputs → XIRR, cover-per-rupee, opportunity cost |
| S11 | What-if scenarios | Re-allocate surplus, compare deterministic outcomes |
| S12 | AI explanations | Grounded narration of computed results, with citations to the result payload |
| S13 | Memory | Explicit, user-approved preferences and goals only |
| S14 | Audit + observability | Correlation IDs, LLM traces, no raw amounts in traces |
| S15 | Public deployment | Single-region MVP, HTTPS, rate limited |

## 3. Out of scope (MVP)

1. Investment or payment execution of any kind.
2. Trading, brokerage, order placement.
3. Bank account aggregation / AA / scraping of user accounts.
4. Personalised regulated investment advice or portfolio recommendations ("buy X").
5. Guaranteed or projected-to-be-guaranteed returns presented as certainty.
6. Autonomous insurance surrender/lapse decisions.
7. Automatic financial transactions of any kind.
8. Training or fine-tuning a foundation model on user financial data.
9. Multi-agent architectures; LangGraph; autonomous tool loops.
10. Tax slab computation (old vs new regime engine), ITR assistance, or tax filing.
11. Real-time market data, NAV feeds, or live rate scraping.
12. Multi-currency, non-INR instruments, crypto, derivatives.
13. Shared team workspaces with invites/roles beyond the personal org (schema supports it; UI does not).
14. Mobile native apps.
15. Document/OCR ingestion of statements or policy PDFs.

## 4. Scope-change protocol

Any addition to §2 or removal from §3 requires:

1. A written amendment to this document (bump version + changelog entry).
2. An ADR if the change affects architecture, data model, or security posture.
3. An update to `docs/delivery/stage-roadmap.md` and the relevant stage task file.

Silent scope expansion inside a feature branch is a review-blocking defect.

## 5. Ambiguities resolved in Stage 0

| Ambiguity | Resolution | Where |
| --- | --- | --- |
| Is this a chat product or a calculator product? | Calculator-with-narration. AI is invoked on a computed result, not on free-form chat. | ADR-0009 |
| Where do interest rates come from? | Curated, dated, source-cited seed dataset; user may override. No scraping. | ADR-0008 |
| Do we support organisations at MVP? | Schema supports `org_id`; MVP creates exactly one personal org per user. | ADR-0010 |
| Do we need vector memory? | No. Structured memory only until a written retrieval requirement exists. | ADR-0007 |
| Tax in comparisons? | Tax *treatment notes* only; no slab arithmetic. | ADR-0011 |
