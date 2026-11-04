# Non-Goals, Disclaimers & Regulatory Stance

**Document ID:** SPEC-PROD-006 · **Stage:** 00 · **Status:** Approved v1.0

## 1. Regulatory stance

MoneyMap is an **educational financial planning and scenario-analysis tool**. It is **not**:

- an investment adviser or portfolio manager,
- a distributor or agent of any financial product,
- an insurance intermediary or broker,
- a depository participant, payment system provider, or account aggregator,
- a tax adviser or filing service.

The product must never be positioned, marketed, or described in a way that implies any of the above.

**Consequence:** MoneyMap must not issue directives ("buy", "sell", "surrender", "invest in") or
rank products as universally best. It may compare, explain trade-offs, and show deterministic
outcomes of user-chosen scenarios.

## 2. Mandatory disclaimers

| ID | Where | Text (canonical) |
| --- | --- | --- |
| DISC-01 | Every AI response payload (`disclaimer` field) and UI footer | "This is an educational analysis of the numbers you provided. It is not investment, tax or insurance advice." |
| DISC-02 | Comparison and scenario views | "Projections are deterministic calculations based on the assumptions shown. Actual returns vary and are not guaranteed." |
| DISC-03 | Rate/source panel | "Rates are as published on the date shown and may have changed. Verify with the institution before acting." |
| DISC-04 | LIC analysis view | "This analysis uses the policy values you entered. It does not recommend surrendering or continuing any policy." |
| DISC-05 | Signup | "MoneyMap does not execute transactions and cannot access your bank accounts." |

Disclaimers are enforced server-side (`FR-DISC-01`), not only in the UI, so API consumers receive
them too.

## 3. Explicit non-goals

1. No execution of investments, payments, redemptions, or premium payments.
2. No guaranteed, "assured", or risk-free return claims.
3. No personalised regulated advice; no suitability determination.
4. No automated insurance surrender/lapse/switch decisions.
5. No autonomous modification of user records by AI.
6. No tax computation, ITR preparation, or slab optimisation.
7. No bank/AA aggregation, scraping of user accounts, or credential collection for third parties.
8. No training or fine-tuning of foundation models on user data.
9. No sale, sharing, or brokering of user financial data; no ad-tech or affiliate links.
10. No prediction of market prices, NAVs, or interest-rate movements.
11. No social/community features, public profiles, or leaderboards.
12. No enterprise/multi-team collaboration UI in MVP.

## 4. Language rules (enforced in prompt + output guardrails)

**Forbidden patterns:** "you should", "I recommend", "best product", "guaranteed", "risk-free",
"sure-shot", "assured returns", "invest in X", "surrender your policy", "switch now".

**Required patterns:** "based on the inputs you provided", "under the assumption that",
"this comparison shows", "the calculation assumes", "verify with the institution".

## 5. Failure honesty

If MoneyMap cannot answer within scope, it must say so and stop. It must not:

- approximate an answer with invented numbers,
- answer a tax question with a guessed slab,
- answer a market-timing question at all,
- continue after a failed deterministic calculation.

## 6. Change control

Any change to this document requires review by the product owner and, where the regulatory posture
is affected, an ADR recording the reasoning and the resulting obligations. Marketing copy, UI text,
prompts, and API descriptions are all in scope of §4 and must be reviewed against it.
