# User Journeys

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## How to Read This Document

Each journey describes the experience and decision flow for a persona (see `docs/personas.md`), expressed as goals, steps, and charter guardrails — not as implementation. Journeys deliberately show where the product **reuses or surfaces CoE data** versus where it adds the **missing licensing layer**. No journey includes an approval workflow or automatic license removal, consistent with the charter.

Legend:

- **Reuse/Surface (CoE):** data or experience sourced from the existing CoE investment (the customer runs the CoE Core Components and Audit Components).
- **Extend (Accelerator over CoE):** project-owned logic or visuals built on top of reused CoE data, with no change to managed CoE assets.
- **Supplement (external API):** data CoE does not hold, acquired via a supported interface (for example, Microsoft Graph licensing).
- **New (Accelerator):** a licensing capability with no CoE equivalent.
- **Guardrail:** a charter safety or governance control enforced in the step.

Every **Supplement**, **Extend**, and **New** step marks exactly where **CoE alone is insufficient** and this product adds value. These map to the Capability Catalog (C1–C8) in `docs/product-definition.md`.

---

## Journey 1 — Onboarding: Discover CoE Capabilities and Gaps

**Persona:** P1 Platform/CoE Administrator (with P5 Security review)
**Trigger:** The accelerator is introduced into a tenant that already runs CoE.
**Goal:** Establish what CoE already provides and what licensing capability is genuinely missing.

1. Administrator confirms the deployed CoE version and enabled components. *(Reuse/Surface — CoE)*
2. The product discovers available inventory, ownership, environment, usage, dashboard, and governance capabilities. *(Reuse/Surface — CoE)*
3. The product records licensing-related capability gaps CoE cannot meet. *(New — Accelerator)*
4. Candidate interfaces for licensing data are listed as *unvalidated* until support, permissions, licensing, throttling, and availability are confirmed. *(Guardrail — supported APIs only; ADR-004)*
5. Security reviews the proposed least-privilege access before anything is connected. *(Guardrail — least privilege)*

**Outcome:** A documented gap list and validated-interface plan. No new storage or experience is proposed until this exists.
**Charter checkpoint:** Nothing is built that CoE already provides.

---

## Journey 2 — Build the Licensing Intelligence Picture

**Persona:** P1 Platform/CoE Administrator; P2 Licensing Analyst
**Trigger:** Discovery is complete and interfaces are validated.
**Goal:** Answer "who has a premium license, and is it being used?"

1. The product acquires assigned premium licensing and SKU data via validated interfaces. *(New — Accelerator)*
2. Each assignment is tagged with its context — direct, group-based, service, shared, or exception — where the source supports it. *(New — Accelerator)*
3. Licensing records are correlated with CoE app, flow, maker, owner, environment, and usage data through supported identifiers. *(Reuse/Surface — CoE)*
4. Unmatched or ambiguous records are surfaced explicitly, not discarded. *(Guardrail — data quality/explainability)*
5. Every view discloses data freshness and completeness. *(Guardrail — explainability)*

**Outcome:** A correlated, freshness-labeled view of premium license utilization.
**Charter checkpoint:** CoE inventory is surfaced, not copied; only licensing data is net-new.

---

## Journey 3 — Review Optimization Opportunities (Operational and Executive)

**Persona:** P1 Platform/CoE Admin (operational); P3 FinOps and P6 Executive Sponsor (executive)
**Trigger:** The intelligence picture exists.
**Goal:** Understand where premium capacity may be optimized.

1. The product identifies potentially inactive licensed users from explainable evidence. *(New — Accelerator)*
2. Optimization opportunities are estimated with disclosed assumptions and freshness. *(Guardrail — estimates not guaranteed)*
3. Existing CoE reporting is extended or reused to present the opportunity before any new dashboard is considered. *(Reuse/Surface — CoE)*
4. Executive views separate *potential* from *realized* savings. *(Guardrail — estimated vs. realized)*

**Outcome:** Operational and executive visibility into optimization potential.
**Charter checkpoint:** Analytics and visibility — not workflow, not a parallel platform.

---

## Journey 4 — Investigate a Single Candidate (Dependency Analysis and Explainability)

**Persona:** P1 Platform/CoE Admin; P4 Environment/BU Admin for local context
**Trigger:** A user appears as a potential reclamation candidate.
**Goal:** Decide safely whether the license can be reclaimed.

1. Administrator opens the candidate and sees the evidence behind the flag. *(New — Accelerator)*
2. The product analyzes application, flow, ownership, and related dependencies sourced from CoE. *(Reuse/Surface — CoE)*
3. The assignment source is shown; group-based assignment is clearly distinguished from direct. *(New — Accelerator; Guardrail)*
4. Protected-identity status (service, break-glass, shared, critical owner) is evaluated. *(Guardrail — safety first)*
5. The candidate is classified **Safe**, **Review**, or **Blocked**, with data sources, freshness, dependencies, assignment source, exceptions, uncertainty, and the governing rule all shown. *(Guardrail — explainability)*
6. If required evidence is missing or stale, a Safe classification is prevented. *(Guardrail — missing/stale evidence blocks Safe)*

**Outcome:** A fully explained, defensible classification for one user.
**Charter checkpoint:** Every recommendation is explainable and dependency-aware.

---

## Journey 5 — Safe License Reclamation (Dry Run, Guardrails, Audit)

**Persona:** P1 Platform/CoE Administrator (authorized)
**Trigger:** One or more candidates are classified Safe and the administrator chooses to act.
**Goal:** Reclaim eligible licenses without risk and with full auditability.

1. Administrator explicitly initiates reclamation; nothing runs automatically. *(Guardrail — administrator-initiated)*
2. The product runs a **dry run** showing exactly what would change and why. *(Guardrail — dry run first)*
3. Only eligible, **directly assigned** licenses are actionable; group-assigned licenses are excluded with the controlling source explained. *(Guardrail — group-assigned excluded)*
4. Protected identities are blocked from action. *(Guardrail — protected identities)*
5. Administrator confirms; the product performs the bounded action. *(New — Accelerator)*
6. Outcomes are recorded as an audit entry (actor, target, evidence, action, timestamp, outcome); partial failures are surfaced explicitly. *(Guardrail — auditable, partial failures visible)*

**Outcome:** Safe, auditable reclamation of eligible licenses.
**Charter checkpoint:** No approval engine, no automatic removal — authorization, dry run, dependency checks, and audit instead.

---

## Journey 6 — Bounded Bulk Reclamation

**Persona:** P1 Platform/CoE Administrator (authorized)
**Trigger:** Multiple Safe candidates need action in one operation.
**Goal:** Act on a bounded set efficiently while preserving every safety control.

1. Administrator selects a bounded set of Safe candidates. *(New — Accelerator)*
2. A dry run reports the full set, including any item that would be skipped and why. *(Guardrail — dry run)*
3. Protected and group-assigned items are excluded and explained; they cannot be forced through. *(Guardrail — safety first)*
4. Administrator confirms the bounded batch; no sign-off workflow is introduced. *(Guardrail — no approval engine)*
5. Each item produces its own audit outcome; partial failures do not fail silently. *(Guardrail — auditable, partial failures visible)*

**Outcome:** Efficient, still-safe bulk reclamation.
**Charter checkpoint:** Bulk action is a capability, not a workflow engine.

---

## Journey 7 — Handle Exceptions and Protected Accounts

**Persona:** P5 Security & Compliance Officer; P8 Service Desk/Operations
**Trigger:** Service, break-glass, shared, or group-assigned accounts appear in analysis.
**Goal:** Ensure protected identities are never reclaimed and exceptions are governed.

1. Protected-identity definitions are sourced or configured from an authoritative list. *(Guardrail — safety first)*
2. Matching accounts are classified **Blocked** with the protecting rule shown. *(Guardrail — explainability)*
3. Security confirms protected-identity rules cannot be silently bypassed. *(Guardrail — governance not weakened)*
4. Any exception change is itself auditable. *(Guardrail — auditability)*

**Outcome:** Protected identities are consistently and transparently excluded.
**Charter checkpoint:** Safety-first exclusions are enforced and auditable.

---

## Journey 8 — Report Value to Leadership

**Persona:** P3 IT Finance/FinOps Lead; P6 Executive Sponsor
**Trigger:** A reporting cycle or renewal discussion.
**Goal:** Communicate controlled spend and realized value.

1. The product presents optimization potential and realized reclamations separately. *(Guardrail — estimated vs. realized)*
2. Reporting reuses or extends existing CoE/executive reporting surfaces. *(Reuse/Surface — CoE)*
3. Assumptions and data freshness are disclosed alongside every figure. *(Guardrail — explainability)*
4. The narrative confirms the initiative extended existing CoE investment rather than building a new platform. *(Charter checkpoint)*

**Outcome:** A credible, auditable value story for leadership and procurement.
**Charter checkpoint:** Visibility and reuse, with honest estimate/realized separation.

---

## Cross-Journey Guardrail Summary

| Guardrail | Journeys Enforcing It |
| --- | --- |
| Reuse/surface CoE before building | 1, 2, 3, 4, 8 |
| Supported, validated interfaces only | 1, 2 |
| Explainable recommendations | 3, 4, 5, 7, 8 |
| Dependency analysis before remediation | 4, 5, 6 |
| Direct vs. group assignment distinction | 2, 4, 5, 6 |
| Protected identities never reclaimed | 4, 5, 6, 7 |
| Dry run before action | 5, 6 |
| Administrator-initiated; no auto-removal | 5, 6 |
| No approval workflow | 5, 6 |
| Auditable outcomes; partial failures visible | 5, 6, 7 |
| Estimated vs. realized savings | 3, 8 |
