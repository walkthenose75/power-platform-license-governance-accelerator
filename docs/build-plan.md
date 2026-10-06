# Build Plan

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Assumptions

This is an **implementation-ready build plan**. It is **planning documentation only** — no Power Platform assets, Dataverse schemas, or flows are generated here.

Assumptions (fixed for this plan):

- CoE **Core Components** and **Audit Components** are installed.
- **CoE is the system of record** for apps, flows, makers, environments, usage, and ownership.
- **Microsoft Graph** provides licensing entitlement data (premium assignment, SKU, direct vs. group).
- **No approval workflows**; **safe remediation only** (administrator-initiated, dry-run, dependency-checked, audited).
- **Custom solution components remain separate** from CoE (no managed-layer modification).
- **Deliverable = an unmanaged solution** the customer imports, owns, and may modify. The customer operates it and is the **data controller**; this plan does not include us running their ALM or production operations.

Capability IDs (C0–C8) reference `docs/product-definition.md`; table names reference `docs/data-model.md`; classifications reference `docs/capability-map.md`.

---

## Repository Evaluation — Critical Missing Design Documents

The repository now includes the critical design specifications. Because the deliverable is an **unmanaged solution handed to the customer** (they import, own, and may modify it), the remaining artifacts are reframed for **handoff and self-serve operation** rather than for operating a managed deployment.

| Document | Role for an unmanaged-solution handoff | Status |
| --- | --- | --- |
| **Interface & API Specification** | Validated supported operations/permissions for licensing read, CoE reads, and reclamation (ADR-004) | ✅ Created |
| **Security & Identity Design** | Least-privilege identities, consent, Dataverse roles, FLS, execution separation | ✅ Created |
| **Recommendation Rules Specification** | Deterministic Safe/Review/Blocked logic and inactivity definition (Q-ANALYTICS-1) | ✅ Created |
| **Customer Deployment Inventory** (template) | Replaces "(validate)" with confirmed CoE artifacts; resolves G1/G2 | ✅ Created |
| **Administrator / Setup & Operations Guide** | The instruction manual: prerequisites, import, configuration, safe operation, troubleshooting (absorbs solution packaging & import) | ✅ Created |
| **Validation & Safety Test Checklist** | Self-serve verification that safety guardrails work before trusting reclamation | ✅ Created |
| **Data Handling & Privacy Note** | Transparency on personal data processed; prompts the customer's own privacy/DPIA review (they are the controller) | ✅ Created |
| ~~ALM & Environment Strategy~~ | **Dropped** — the customer owns ALM after import; minimal packaging/import guidance folded into the Operations Guide | N/A |

> Delivery note: for a handoff accelerator, the **Administrator / Setup & Operations Guide** is the highest-value artifact, and the **Safety Test Checklist** is essential because the solution performs destructive license reclamation.

---

## Work Packages

Each work package lists **Objective · Scope · Deliverables · Dependencies · Acceptance Criteria · Risks · Measurable Value · Reuse vs. Build**. No Power Platform assets are produced by this plan; deliverables describe what each package will build.

### WP0 — Enabler: Foundations, Validation, Security (prerequisite — not a value slice)

- **Objective:** establish the minimum platform, validated interfaces, and security needed to deliver value, and resolve blocking gaps.
- **Scope:** project-owned solution + publisher in a dev environment; Graph app registration with least-privilege scopes and admin consent; validated **Interface & API Specification** (licensing read + reclamation operation) per ADR-004; **Security & Identity Design**; run `docs/core-and-audit-components-assessment.md` to confirm CoE artifacts and resolve **G1 (per-user usage granularity)** and **G2 (usage-history window)**; environment variables and connection references; the four Dataverse security roles; **Customer Deployment Inventory** recorded.
- **Deliverables:** solution skeleton; app registration + consent; interface spec; security design; confirmed deployment inventory; security roles.
- **Dependencies:** customer/security approval for app registration and consent; CoE admin access; protected-identity source (Q-SAFETY-1).
- **Acceptance Criteria:** a test Graph call returns premium assignments; maker **Entra object IDs confirmed populated** in CoE; usage-window length recorded; solution deploys to dev; four roles created; "(validate)" labels relevant to WP1–WP3 resolved.
- **Risks:** consent/permission approval delays; **G1 unresolved** (per-user usage may require supplement); **G2** window shorter than threshold; CoE version/schema drift.
- **Measurable Value:** none directly — explicitly an enabler. (Kept separate so WP1 remains the first value slice.)
- **Reuse vs. Build:** Reuse CoE (validate); Supplement Graph (validate); Build only the solution skeleton and security scaffolding.

### WP1 — Premium License Visibility (FIRST value slice — smallest vertical slice)

- **Objective:** deliver a **read-only, quantified** view of who holds premium licenses and a first-pass utilization indicator, proving the **acquire → correlate → surface** path end to end and sizing the optimization opportunity.
- **Scope:** acquire premium assignments + SKU from Graph (C1; **direct vs. group** distinguished) into **License Assignment** + **SKU Reference**; correlate to CoE maker/user via **Entra object ID** (C2); reuse CoE Audit **last-launched/usage** for a basic "recent usage evidence?" flag; one read-only model-driven view plus a minimal dashboard tile; disclose freshness; surface unmatched records.
- **Deliverables:** minimal solution with License Assignment + SKU Reference; acquisition design; correlation logic; one view + one dashboard tile (counts, direct vs. group, "no recent usage evidence" count, estimated opportunity); freshness indicator.
- **Dependencies:** WP0 (Graph registration, solution, maker Entra ID validated, usage window).
- **Acceptance Criteria:**
  - Shows total premium licenses assigned and the **direct vs. group** split.
  - Flags the count with **no recent usage evidence** within the window.
  - Shows an **estimated opportunity** value with assumptions and freshness disclosed (never presented as guaranteed).
  - **Unmatched** correlation records are visible, not hidden.
  - **No** recommendation, classification, or reclamation is present; **nothing writes to CoE**.
- **Risks:** per-user usage unavailable (fall back to last-launched on owned apps with disclosed lower confidence — G1/G3); unmatched correlation volume; estimate misread (label clearly as estimate).
- **Measurable Value:** the baseline KPI — "**N premium licenses; M with no recent usage evidence (~$X opportunity)**" — immediate, defensible insight for procurement/admins with zero remediation risk.
- **Reuse vs. Build:** **Reuse** CoE maker + Audit usage; **Supplement** Graph; **Build** License Assignment/SKU + correlation + one view/tile.

### WP2 — Explainable Optimization and Dependency Safety (read-only)

- **Objective:** convert visibility into **explainable Safe/Review/Blocked** candidates with dependency and protected-identity safety (C3/C4/C5) — still read-only.
- **Scope:** add **Optimization Candidate**, **Recommendation Evidence**, **Dependency Finding**, **Protected Identity / Exception**; per-user inactivity analytic (evidence coverage, freshness); dependency analysis from CoE relationships; protected-identity handling; classification + explainability; triage view; finalize the **Recommendation Rules Specification**.
- **Deliverables:** the four tables; inactivity analytic; dependency findings; classification/explainability; triage view; rules spec.
- **Dependencies:** WP1; protected-identity source (Q-SAFETY-1); criticality source (Q-COE-3); inactivity definition (Q-ANALYTICS-1).
- **Acceptance Criteria:** each candidate shows classification with evidence, dependencies, assignment source, exceptions, freshness, and the governing rule; **missing/stale evidence blocks Safe**; group-assigned and protected identities are **excluded with reason**; an "estimated reclaimable" count is produced.
- **Risks:** inactivity-definition disputes; criticality not populated; identifier mismatch causing false dependency conclusions.
- **Measurable Value:** a defensible candidate list with evidence; "estimated reclaimable licenses/$".
- **Reuse vs. Build:** **Reuse/Extend** CoE relationships + usage; **Build** candidate/evidence/dependency/protected-identity logic.

### WP3 — Safe Reclamation (dry run + single) and Audit (MVP completion)

- **Objective:** administrator-initiated reclamation of a **single eligible directly-assigned** license with dry run, guardrails, and immutable audit (C6/C7) — completing the MVP (`docs/mvp-definition.md`).
- **Scope:** add **Reclamation Action** and **Audit Log Entry**; dry-run; execute via the supported API; enforce protected/group/stale exclusion; immutable audit; command bar + confirmation; execution-separation security.
- **Deliverables:** the two tables; dry-run; execution; guardrails; immutable audit; command + confirmation.
- **Dependencies:** WP2; validated reclamation API and security sign-off (WP0).
- **Acceptance Criteria:** dry run shows the exact change; a single reclamation executes **only** for eligible direct-assigned licenses; protected/group/stale are blocked; an audit entry (actor/evidence/action/outcome) is written and **immutable**; partial failure is surfaced; **nothing is automatic** and **no approval workflow** exists. Meets the MVP Definition of Done.
- **Risks:** reclamation API behavior/edge cases; accidental removal (mitigated by guardrails + dry run + tests against disposable identities); audit immutability enforcement.
- **Measurable Value:** **realized reclamations (audited)** and **accidental-removals-prevented** — the core payoff, safely.
- **Reuse vs. Build:** **Build** (via supported API) with **Reuse** of protected-identity/ownership evidence.

### WP4 — Bounded Bulk Reclamation (fast-follow)

- **Objective:** bounded bulk reclamation with the same guardrails via a guided experience.
- **Scope:** bounded-bulk design; Bulk Reclamation custom page; per-item dry-run/exclusion/audit; enforced cap.
- **Deliverables:** bounded-bulk capability; custom page; per-item audit.
- **Dependencies:** WP3; bulk-bounds decision (Q-SAFETY-4).
- **Acceptance Criteria:** bounded set only; per-item exclusions explained; per-item audit; cap enforced; **no approval engine**; no automatic removal.
- **Risks:** blast radius (mitigated by bounds + dry run + per-item audit).
- **Measurable Value:** safe efficiency at scale.
- **Reuse vs. Build:** **Build** (extends WP3 safely).

### WP5 — Operational and Executive Dashboards (extend CoE Power BI)

- **Objective:** extend CoE Power BI with licensing/optimization visuals and an executive value view (estimated vs. realized) (C8).
- **Scope:** extend the reused CoE semantic model with licensing measures; operational triage dashboard; executive summary; freshness and estimated-vs-realized separation; deep-link to CoE.
- **Deliverables:** extended model; operational dashboard; executive summary.
- **Dependencies:** WP1 (visibility) and WP3 (realized); CoE Power BI reuse validated (assessment area 10).
- **Acceptance Criteria:** reuses the CoE model (no new parallel model); estimated vs. realized shown distinctly; freshness disclosed.
- **Risks:** CoE model extensibility; over-building a parallel model (mitigated by reuse-first).
- **Measurable Value:** executive value narrative; operational triage efficiency.
- **Reuse vs. Build:** **Reuse/Extend** CoE Power BI; **Build** licensing measures + triage view. (May start after WP2; finalize after WP3.)

### WP6 — Packaging, Documentation, and Handoff

- **Objective:** produce the **unmanaged solution file** and the materials a customer needs to import, trust, operate, and modify it.
- **Scope:** export the **unmanaged solution** (publisher/prefix; connection references and environment variables parameterized for the importer); finalize the **Administrator / Setup & Operations Guide**, **Validation & Safety Test Checklist**, and **Data Handling & Privacy Note**; final safety review; confirm the solution has **no managed dependency on CoE** (loose coupling) so import never fails on CoE schema differences.
- **Deliverables:** unmanaged solution file; the three handoff documents; final safety sign-off.
- **Dependencies:** WP1–WP3 at minimum (WP4/WP5 if included).
- **Acceptance Criteria:** the unmanaged solution imports cleanly into a clean environment with CoE present; connection references/environment variables are set on import; the safety checklist passes; the guides let an administrator configure and operate without further help.
- **Risks:** import failures from an unintended CoE dependency (mitigated by canonical-ID loose coupling); customers skipping safety validation (mitigated by a prominent checklist and guardrails).
- **Measurable Value:** a self-serve, safe, well-documented accelerator the customer can adopt and extend.
- **Reuse vs. Build:** **Build** the packaging and documentation; the customer owns ALM and operations post-import.

---

## Suggested Build Order

```
WP0 Enabler ──▶ WP1 Visibility ──▶ WP2 Explainable Candidates ──▶ WP3 Safe Reclamation (MVP ✔)
  (gate)        (first value)                        │                         │
                                                     └──▶ WP5 Dashboards ◀──────┘ (start after WP2, finalize after WP3)
                                                                                   │
                                                                         WP4 Bounded Bulk (fast-follow)
                                                     WP6 Packaging & Handoff (final; produces the unmanaged solution + guides)
```

**Gates:**

1. **WP0 blocking validations** (Graph consent; G1 per-user usage; G2 window) must pass before WP1.
2. **Security sign-off** required before WP3 execution.
3. **MVP Definition of Done** (`docs/mvp-definition.md`) is the exit of WP3.
4. **Handoff** requires WP6 completion (unmanaged solution exported + guides + safety checklist passed).

Rationale: WP1 is the smallest end-to-end slice that delivers measurable value with zero remediation risk; each subsequent package adds one safe increment (explainability → single reclamation → bulk), with dashboards and hardening running alongside.

---

## Reuse vs. Build-New Analysis

| WP | Reused from CoE / Graph | Built new (project-owned) |
| --- | --- | --- |
| WP0 | CoE validation; Graph app registration | Solution skeleton; security roles |
| WP1 | CoE maker (Core), usage (Audit); Graph licensing (Supplement) | License Assignment, SKU Reference; correlation; 1 view/tile |
| WP2 | CoE relationships/ownership (Core), usage (Audit) | Candidate, Evidence, Dependency Finding, Protected Identity; rules |
| WP3 | Protected-identity/ownership evidence (reused) | Reclamation Action, Audit Log; dry-run; execution via supported API |
| WP4 | — | Bounded-bulk capability; custom page |
| WP5 | CoE Power BI model + usage (Reuse/Extend) | Licensing measures; triage dashboard |
| WP6 | — | Unmanaged solution export; handoff guides (admin/ops, safety checklist, privacy note) |

**Summary:** inventory, ownership, usage, and dashboards are reused/extended from CoE; licensing entitlement is supplemented from Graph; the only net-new build is the licensing layer (acquisition, correlation, recommendation, reclamation, audit, protected identities). No inventory or usage collection is rebuilt (`docs/data-model.md` "Tables We Explicitly Will Not Create").

---

## Testing Strategy

**Principles (safety-first):**

- **Never execute reclamation against real users in test.** Use **disposable test identities** and dry-run parity.
- **Missing usage is never treated as zero usage**; **missing/stale evidence blocks Safe**; freshness is always honored.
- The **immutable audit** and **execution separation** are verified, not assumed.

**Test levels:**

- **Unit:** correlation/matching, per-user inactivity analytic, Safe/Review/Blocked rules, estimate calculation, protected/group-exclusion logic.
- **Integration:** Graph acquisition (test tenant or mock), Dataverse adapter reads against a **test CoE environment**, reclamation operation in **dry-run/sandbox** against disposable identities, throttling/pagination/retry.
- **Data quality:** unmatched/ambiguous records surfaced; stale/missing evidence blocks Safe; freshness disclosed; direct-vs-group correctness.
- **Security:** role privilege matrix; field-level security on PII/financial; **Analyst cannot execute** reclamation; audit is append-only (no update/delete); least-privilege Graph scopes.
- **Safety/acceptance:** protected identities never reclaimed; group-assigned excluded; dry-run correctness; no automatic removal; **no approval artifacts anywhere**; partial failure surfaced.
- **UAT:** per persona (Executive / Analyst / Administrator / Auditor) against `docs/user-journeys.md`.
- **Regression + performance:** re-run safety and data-quality suites each increment; validate paging/throttling at volume.

**Environments & data:** non-production CoE + licensing test data; seeded sample records; disposable identities for any reclamation path. No production reclamation outside a controlled, signed-off exercise.

**Acceptance alignment:** per-WP acceptance criteria above, culminating in the **MVP Definition of Done** at WP3. A detailed **Test Plan** (identified as missing above) will expand these into executable cases.

---

## Delivery Risks (Consolidated)

See `docs/product-definition.md` §5 for the full product risk register. Delivery-specific risks:

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Graph app registration / admin consent delays | Blocks WP1 | Start WP0 consent early; treat as a gate with an owner |
| Per-user usage granularity gap (G1) | Weakens inactivity (C3) | Validate early; supplement via supported interface or adjust inactivity definition |
| Usage-history window too short (G2) | Lower inactivity confidence | Align threshold to window; disclose bound |
| Security sign-off latency before WP3 | Delays reclamation | Sequence Security & Identity Design in WP0; review gate before WP3 |
| CoE version/schema drift | Breaks adapter | Isolate schema in the adapter; confirm via assessment; canonical IDs |
| Scope creep to approval/bulk/automation | Charter breach | Enforce charter exclusions; WP4 bounded; no approval engine ever |
| Estimate misinterpreted as guaranteed savings | Credibility | Separate estimated vs. realized everywhere; disclose assumptions |

---

## Constraints Honored

- Planning documentation only — no Power Platform assets, Dataverse schemas, or flows were generated.
- All assumptions respected: CoE Core + Audit as system of record, Graph for licensing, no approval workflows, safe remediation only, custom components separate from CoE.
- WP1 is the smallest vertical slice delivering measurable business value; reclamation is introduced only later, always administrator-initiated, dependency-checked, dry-run, and audited.
