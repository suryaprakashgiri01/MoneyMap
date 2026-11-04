# Problem Statement

**Document ID:** SPEC-PROD-001 · **Stage:** 00 · **Status:** Approved v1.0

## 1. Context

Most salaried and self-employed Indians interact with financial products through sellers, not
through analysis. A typical individual holds a mix of instruments accumulated over years — a bank
FD, one or two RDs, a PPF account, a Post Office scheme, an LIC endowment policy, and one or two
mutual fund SIPs — without ever having compared them on a common basis.

## 2. Problem

Individuals cannot answer basic questions about their own money:

1. **Where does my salary go?** Fixed obligations, variable expenses and actual savings are not
   separated, so "savings" is whatever is left over and is never measured.
2. **How safe am I?** There is no emergency fund target and no visibility into how many months of
   expenses existing liquid assets cover.
3. **What am I actually earning?** Advertised rates (e.g. "8.2% p.a." on a PPF, "guaranteed
   maturity" on an endowment policy) are not comparable. The true realised return of an LIC policy
   is an XIRR question, not a rate question.
4. **What am I giving up?** Lock-in, liquidity, tax treatment and insurance coverage are invisible
   in a rate-only comparison, so a product that looks like 8% may be materially worse than one at 7%.
5. **What happens if I change my allocation?** There is no low-cost way to simulate
   "redirect ₹5,000/month from my LIC premium to an index SIP" before committing to it.

## 3. Why existing tools do not solve it

- Bank and broker apps show **their own products only**; there is no cross-product view.
- Calculators compute a single future value with no context of the user's actual surplus.
- Generative chat assistants produce **plausible but unverifiable numbers** and cannot be audited.
- Advisory platforms require onboarding, KYC and a sales motion, and are not educational tools.

## 4. What MoneyMap does

MoneyMap gives an individual a transparent, deterministic picture of their financial position and a
comparison of their real options:

- A structured **financial profile** (income, fixed expenses, variable expenses, dependants).
- **Deterministic calculations**: savings, savings ratio, emergency fund target, goal progress,
  FD/RD/PPF/Post Office/SIP projections, and XIRR for existing products.
- **Product comparisons** across return, risk, liquidity, lock-in, tax treatment and horizon,
  using dated, source-cited rate data.
- **What-if scenarios** that re-allocate the monthly surplus and show the deterministic delta.
- **AI explanations** that are constrained to narrate computed results, never to compute them.

## 5. Success in one sentence

A user can state their monthly surplus and, within a few minutes, see which of their existing
products is underperforming and what a different allocation would deterministically produce —
with every number traceable to an input, a source, or a documented assumption.

## 6. Non-negotiable constraints

- The LLM never produces a financial number.
- Every number carries provenance (`provided`, `sourced`, `assumed`, `calculated`, `generated`).
- Missing data is surfaced as missing, never silently defaulted.
- The product is educational. It is not a regulated investment adviser and must never imply that
  it is (see `non-goals-and-disclaimers.md`).
