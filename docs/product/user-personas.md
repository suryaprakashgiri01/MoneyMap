# User Personas

**Document ID:** SPEC-PROD-003 · **Stage:** 00 · **Status:** Approved v1.0

Personas are decision tools, not marketing. Each has a primary job and a "what must not happen".

---

## P1 — Ravi, 28, salaried engineer (primary)

- **Income:** ₹85,000/month net, salaried, single, no dependants.
- **Context:** 3 years into his career. Holds a bank FD (₹1.5L), one RD (₹5,000/month), a
  ₹3,000/month index SIP, and a PPF opened for the tax deduction.
- **Financial literacy:** can read a rate; cannot compute XIRR; does not know his savings ratio.
- **Job to be done:** "Tell me where my money actually goes and whether my RD is a bad idea."
- **Success:** sees savings ratio and a comparison showing the RD vs SIP vs PPF on the same basis.
- **What must not happen:** being told "buy Nifty index funds" as a directive, or seeing a number
  that does not reconcile with his own inputs.
- **Product surface used:** profile → metrics → holdings → comparison → explanation.

## P2 — Meera, 34, mid-career parent (primary)

- **Income:** ₹1,40,000/month net; household expenses ₹95,000; one child, school fees a fixed cost.
- **Context:** bought an LIC endowment policy 6 years ago on a relative's advice ("₹1L premium,
  ₹22L maturity in 20 years"). She has never computed the real return.
- **Financial literacy:** medium. Knows PPF is safe; does not know XIRR, lock-in cost, or cover-per-rupee.
- **Job to be done:** "Is this policy actually giving me anything, and what am I giving up?"
- **Success:** deterministic XIRR of her policy vs a like-for-like alternative over the same horizon,
  with liquidity and cover differences made explicit — **not** a surrender instruction.
- **What must not happen:** an AI claim that she "should surrender" a policy; a maturity figure that
  ignores premiums paid to date.
- **Product surface used:** holdings (LIC) → LIC analysis → comparison → scenario → explanation.

## P3 — Arjun, 41, freelancer (primary)

- **Income:** ₹1.2L–₹2.6L/month, irregular; no EPF; no employer insurance.
- **Context:** 4 months of runway in a savings account, no emergency fund policy, three FDs of
  varying maturities, one Post Office MIS for monthly cash flow.
- **Job to be done:** "How much buffer do I need for a bad quarter, and how do I ladder what's left?"
- **Success:** emergency fund target computed from *his* essential expenses, gap quantified,
  scenario showing a 6-month ramp to close the gap.
- **What must not happen:** a default assumption that hides his irregularity (e.g. treating one
  month's income as fixed salary).
- **Product surface used:** profile (variable income mode) → emergency fund → scenario.

## P4 — Priya, 25, first-job starter (secondary)

- **Context:** ₹35,000/month, savings in a savings account only, no products.
- **Job to be done:** "What should I even be doing with the surplus?"
- **Success:** understands the 3-bucket framing (buffer → goals → long-term) with deterministic
  numbers, and comparisons only among instruments she can actually access.
- **Constraint:** every screen must be usable without financial jargon.

---

## Anti-personas (explicitly not served)

| Anti-persona | Why excluded |
| --- | --- |
| Insurance/AMC sales agent looking for leads | Out of scope; no lead-gen, no contact export |
| Trader wanting signals or timing calls | Out of scope; no market prediction |
| CA/tax filer wanting slab computation | Out of scope (see `product-scope.md` §3.10) |
| Enterprise finance team wanting shared budgets | Org schema exists, but no team UI in MVP |

## Persona-derived design constraints

1. **Precision over reassurance.** Never round a number in a way that changes a conclusion.
2. **Explain every unfamiliar term inline** (XIRR, lock-in, cover-per-rupee, savings ratio).
3. **Show units and periodicity** on every amount (₹/month vs ₹/year vs lump sum).
4. **Never hide a missing input behind a default.** Ask, or mark the field `assumed`.
5. **No personality.** No financial advice voice, no urgency, no "you should buy".
