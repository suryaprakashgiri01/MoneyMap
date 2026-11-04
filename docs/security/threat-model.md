# Threat Model

**Document ID:** SPEC-SEC-001 · **Stage:** 00 · **Status:** Approved v1.0
**Method:** STRIDE over trust boundaries, plus an AI-specific abuse model.
**Related:** ADR-0012 (auth), ADR-0013 (observability), NFR-SEC/PRIV

## 1. Assets

| Asset | Sensitivity | Impact if compromised |
| --- | --- | --- |
| Income, expenses, holdings, goals | High | Financial profile exposure, targeting, reputational harm |
| LIC policy details (sum assured, premiums) | High | Insurance-profile exposure |
| Password hashes / refresh tokens | Critical | Account takeover |
| Computed results (`computed_results`) | High | Full financial snapshot |
| AI prompts / traces | Medium | Indirect exposure of amounts if redaction fails |
| Rate dataset | Low | Integrity matters more than confidentiality |
| Audit and security events | Medium | Tampering hides abuse |

## 2. Actors

- Anonymous internet user; authenticated user (honest); authenticated user (malicious/curious);
  compromised user device; malicious or compromised LLM provider; scrapers/bots; insider (future ops);
  dependency supply chain attacker.

## 3. Trust boundaries

```
TB1 browser ──HTTPS──► API            (untrusted input)
TB2 API ──SQL──► Postgres             (network boundary, credentials)
TB3 API ──HTTPS──► LLM provider       (data leaves our trust domain)
TB4 API ──OTLP──► telemetry backend   (data leaves our trust domain)
TB5 admin/seed CLI ──► DB              (privileged path)
```

## 4. Threats and mitigations

### 4.1 Spoofing

| ID | Threat | Mitigation | Stage |
| --- | --- | --- | --- |
| T-S-01 | Credential stuffing | Argon2id, rate limit per IP+account, generic errors, lockout escalation | 01 |
| T-S-02 | Refresh token theft | Hashed storage, rotation, family reuse detection, httpOnly cookie | 01 |
| T-S-03 | JWT forgery | Strong secret from env, `kid` rotation, short TTL, `ver` claim | 01 |
| T-S-04 | Email enumeration | Uniform 401/409 responses and timing | 01 |

### 4.2 Tampering

| ID | Threat | Mitigation | Stage |
| --- | --- | --- | --- |
| T-T-01 | Client sends another org's `org_id` | Org derived from token only; body `org_id` ignored/rejected | 01 |
| T-T-02 | Client injects numbers into AI path | AI input limited to `result_ref`/`style`/`locale` (ADR-0009) | 04 |
| T-T-03 | Scenario mutates real holdings | Immutable snapshots; test asserts holdings hash unchanged | 03 |
| T-T-04 | Rate data tampering via API | Rates are read-only for users; seed/CLI only | 03 |
| T-T-05 | Mass assignment on PATCH | Strict Pydantic models, unknown fields rejected | 01 |

### 4.3 Repudiation

| ID | Threat | Mitigation | Stage |
| --- | --- | --- | --- |
| T-R-01 | User denies deleting a holding | `audit_events` with request id, append-only | 03 |
| T-R-02 | No record of AI output | Langfuse + `trace_id` on response and feedback | 07 |

### 4.4 Information disclosure

| ID | Threat | Mitigation | Stage |
| --- | --- | --- | --- |
| T-I-01 | Cross-tenant read via ID guessing | Org-scoped repos, UUIDs, 404 on foreign ids, isolation tests | 01 |
| T-I-02 | Amounts in logs/traces | Allow-list attribute keys, redactor, CI regex assertion | 07 |
| T-I-03 | Prompt leaks other users' data | Context builder scoped to one `org_id`; refuses multi-org assembly | 05 |
| T-I-04 | Verbose errors leak internals | Central exception handler with stable codes | 01 |
| T-I-05 | Provider retains prompts | Documented retention posture, zero-retention where available | 04 |
| T-I-06 | Backup/DB snapshot exposure | Managed encryption at rest, least-privilege DB user | 08 |

### 4.5 Denial of service

| ID | Threat | Mitigation | Stage |
| --- | --- | --- | --- |
| T-D-01 | AI endpoint abuse (cost DoS) | Per-user/minute + daily cap, global circuit breaker | 06 |
| T-D-02 | Expensive comparison with huge inputs | Input bounds (≤ 10 instruments, ≤ 600 months) | 03 |
| T-D-03 | Auth brute force | Rate limits + progressive lockout | 01 |
| T-D-04 | Oversized payloads | Body size limit at proxy + Pydantic constraints | 01 |

### 4.6 Elevation of privilege

| ID | Threat | Mitigation | Stage |
| --- | --- | --- | --- |
| T-E-01 | User acts as another org member | No roles in MVP; `org_id` from token | 01 |
| T-E-02 | Model invokes mutating tool | Tool set is empty/read-only allow-list (FR-GRD-05) | 04 |
| T-E-03 | SQL injection | ORM/parameterised queries only, no raw SQL in app code | 01 |

## 5. AI-specific threats

| ID | Threat | Mitigation | Test |
| --- | --- | --- | --- |
| AI-01 | Prompt injection via `holding.notes` / memory content | Treat as data, fenced blocks, never in system prompt, injection-pattern validator on memory writes | Eval suite ≥ 20 cases |
| AI-02 | Numeric hallucination | Grounding check with fail-closed retry (ADR-0004) | AC-AI-01 |
| AI-03 | Advice drift ("you should buy X") | Forbidden-language scan + template constraints | AC-GRD-03 |
| AI-04 | Invented product rate | Prompt forbids rates; output scan; grounding catches numbers | T-05 |
| AI-05 | Record modification via model | No mutating tools; AI layer has read-only DB access scope | AC-GRD-04 |
| AI-06 | Data exfiltration via provider | Minimal context, redacted traces, documented provider policy | Privacy review |
| AI-07 | Indirect injection via retrieved generic memory | Generic memory is curated + reviewed; no user-authored retrieval | Seed review |
| AI-08 | Denial-of-wallet | Budget caps, `max_tokens`, per-user quotas | AC-GRD-05 |

## 6. Out-of-scope threats (accepted risk)

- Nation-state adversaries and physical device compromise.
- Malicious browser extensions reading the page (mitigated only by CSP).
- Compromise of the managed Postgres provider itself.
- User sharing their own credentials.

## 7. Residual risks

| Risk | Likelihood | Impact | Acceptance rationale |
| --- | --- | --- | --- |
| Rate data goes stale between manual reviews | Medium | Medium | Staleness flags + `partial` status; user instructed to verify |
| Grounding false-rejects valid paraphrase | Medium | Low | Degrades to `unavailable`, never to wrong numbers |
| LLM provider policy change | Medium | Medium | Gateway abstraction; config switch |
| Single-instance availability | Medium | Low | Acceptable for MVP; documented |

## 8. Security verification plan

| Control | How verified | Stage |
| --- | --- | --- |
| Tenant isolation | Parametrised cross-org test matrix per resource | 01–03 |
| AuthN/AuthZ | AC-AUTH-*, token reuse, expiry tests | 01 |
| Input validation | Fuzz-ish property tests on money/date fields | 02–03 |
| PII redaction | Log/trace capture assertions | 07 |
| Injection resistance | Versioned eval dataset | 06 |
| Dependencies | `pip-audit`, `npm audit`, secret scanning in CI | 01+ |

## 9. Review cadence

- Threat model reviewed at every stage gate that adds a trust boundary (04, 05, 07, 08, 09).
- Any new external data flow requires an update to §3, §4 and `privacy-and-data-handling.md`.
