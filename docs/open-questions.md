# Open Questions

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose

This document tracks **unresolved product and architecture decisions** that must be answered before or during implementation. It assumes the customer operates the CoE Core Components and Audit Components (CenterOfExcellenceAuditComponents), so questions focus on the **missing licensing layer** (Capability Catalog C0–C8 in `docs/product-definition.md`), not on CoE itself.

Each question records: context, why it matters, candidate options, suggested owner, and the decision gate (when it must be resolved). Questions are **not** decisions — settled decisions live in `docs/decisions.md`.

## Already Settled (Do Not Re-Open Without a Charter-Level Change)

- CoE is reused, not replaced; no parallel inventory (charter; ADR-001).
- No business approval workflows (ADR-002).
- Reuse/extend CoE experiences before building a separate app (ADR-003).
- Only validated, supported interfaces (ADR-004).
- No automatic or unattended license removal; safety-first guardrails (guardrails).

---

## Q-DATA — Interfaces and Licensing Source

### Q-DATA-1 — Graph licensing permission model

- **Context:** C1 acquires premium assignment and SKU data from Microsoft Graph.
- **Why it matters:** determines least-privilege design, approval path, and whether unattended acquisition is possible.
- **Options:** application permissions vs. delegated; which directory/licensing scopes; admin-consent process.
- **Owner:** Security & Platform admin. **Gate:** before C1 build (MVP prerequisite).

### Q-DATA-2 — Direct vs. group-based assignment fidelity

- **Context:** C1/C6 must distinguish direct from group-assigned licenses; only direct is reclaimable.
- **Why it matters:** group-assigned licenses must be excluded from direct removal and explained.
- **Options:** rely on Graph assignment-source data; define behavior when the source is ambiguous.
- **Owner:** Architect. **Gate:** before C5/C6 design.

### Q-DATA-3 — Acceptable data-freshness SLA

- **Context:** recommendations depend on licensing and CoE usage freshness.
- **Why it matters:** defines when a recommendation is actionable vs. must be blocked as stale.
- **Options:** a maximum evidence age for Safe; per-signal freshness thresholds.
- **Owner:** Product + Security. **Gate:** before C5 rules finalized.

## Q-COE — CoE Integration Specifics

### Q-COE-1 — Exact CoE artifact names and schema

- **Context:** the reuse analysis uses illustrative "(validate)" names.
- **Why it matters:** the adapter (C2) must bind to the customer's actual tables/columns/relationships.
- **Options:** confirm from the exported CoE solution during design.
- **Owner:** Architect + CoE admin. **Gate:** before C2 build.

### Q-COE-2 — CoE usage-history window length

- **Context:** C3 inactivity depends on how far back CoE Audit usage reaches.
- **Why it matters:** the inactivity threshold cannot exceed the available window without caveats.
- **Options:** confirm the configured retention/window; align the threshold to it.
- **Owner:** CoE admin. **Gate:** before C3 analytics finalized.

### Q-COE-3 — Business-criticality classification availability

- **Context:** C4 safety is stronger when critical apps/flows/owners are known.
- **Why it matters:** determines whether criticality is reused from CoE or designated by the project.
- **Options:** reuse an existing CoE criticality classification if populated; otherwise define a project-owned, administrator-maintained designation.
- **Owner:** Product + CoE admin. **Gate:** before C4 rules finalized.

## Q-SAFETY — Protected Identities and Reclamation Safety

### Q-SAFETY-1 — Authoritative source of protected identities

- **Context:** service, break-glass, shared, and critical accounts must never be reclaimed.
- **Why it matters:** the exclusion list is central to safety; it must be authoritative and maintainable.
- **Options:** existing security group(s); a CoE attribute; a project-owned, administrator-maintained list; or a combination.
- **Owner:** Security. **Gate:** before C6 (hard MVP prerequisite).

### Q-SAFETY-2 — Handling of group-assigned candidates

- **Context:** group-assigned licenses cannot be removed per-user.
- **Why it matters:** defines whether the product only detects-and-blocks or also advises the correct group-administration path.
- **Options:** MVP detect-and-block with explanation; fast-follow advisory. (No direct group changes — charter.)
- **Owner:** Product + Security. **Gate:** C6 MVP (detect/block) now; advisory later.

### Q-SAFETY-3 — Reclamation mechanism (which supported operation)

- **Context:** C6 executes the license change via a supported interface.
- **Why it matters:** must safely target a single direct assignment, be idempotent where possible, and report partial failure.
- **Options:** validated Microsoft Graph / supported admin operation (per ADR-004).
- **Owner:** Architect. **Gate:** before C6 build.

### Q-SAFETY-4 — Bounded-bulk limits (fast-follow)

- **Context:** bulk reclamation is post-MVP but needs guardrails when introduced.
- **Why it matters:** defines maximum batch size/rate to contain blast radius.
- **Options:** fixed cap; rate limiting; per-run confirmation.
- **Owner:** Product + Security. **Gate:** before bulk fast-follow.

## Q-ANALYTICS — Inactivity Definition

### Q-ANALYTICS-1 — What "inactive premium-licensed user" means

- **Context:** C3/C5 hinge on a precise, explainable inactivity definition.
- **Why it matters:** drives every recommendation and its defensibility.
- **Options:** no premium-app launch in N days; inclusion of flow runs/maker activity; multi-signal definition with disclosed thresholds.
- **Owner:** Product + Architect. **Gate:** before C3/C5 finalized.

### Q-ANALYTICS-2 — Treatment of absent usage evidence

- **Context:** charter guardrail — missing usage must never be read as zero usage.
- **Why it matters:** determines whether absent evidence yields Review/Blocked rather than Safe.
- **Options:** default to Review/Blocked; require minimum evidence coverage for Safe.
- **Owner:** Architect + Security. **Gate:** before C5 rules finalized.

## Q-UX — Experience Surface

### Q-UX-1 — Where to surface the experience

- **Context:** ADR-003 prefers reusing/extending CoE experiences.
- **Why it matters:** determines whether licensing visibility and reclamation live in an extended CoE experience, a project-owned app, or reporting only.
- **Options:** extend the CoE model-driven experience; a separate project-owned app for a documented gap; Power BI-first for visibility with a minimal action surface.
- **Owner:** Product + Architect. **Gate:** before C6/C8 design.

### Q-UX-2 — Operational vs. executive visibility split

- **Context:** C8 serves both operators and executives.
- **Why it matters:** defines the MVP-minimal view vs. fast-follow executive dashboards.
- **Options:** MVP operational view extending CoE reporting; executive dashboard later.
- **Owner:** Product. **Gate:** C8 scoping.

## Q-SCOPE — Scope and SKU Coverage

### Q-SCOPE-1 — Premium SKUs in scope for MVP

- **Context:** C1 targets premium licensing.
- **Why it matters:** bounds MVP effort and the savings case.
- **Options:** Power Apps Premium per-user first; add per-app and Power Automate premium later.
- **Owner:** Product + Procurement. **Gate:** before C1 build.

## Q-VALUE — Savings and Value Measurement

### Q-VALUE-1 — Definition of "realized savings"

- **Context:** success metrics separate estimated from realized savings.
- **Why it matters:** avoids presenting estimates as guaranteed and defines the finance handshake.
- **Options:** realized = confirmed reclamations over a period; reconciliation cadence; ownership of the figure.
- **Owner:** Finance/FinOps + Product. **Gate:** before C8 executive reporting.

---

## Assumptions to Validate (Cross-Reference)

These product assumptions (see `docs/product-definition.md` §6) remain open until confirmed and are tracked here by reference: interface support/permissions (A-3, Q-DATA-1), assignment-context availability (A-4, Q-DATA-2), protected-identity source (A-7, Q-SAFETY-1), CoE experience extensibility (A-6, Q-UX-1), and savings expectations (A-9, Q-VALUE-1).

## Resolution Process

- Record each resolved question as a short decision in `docs/decisions.md` (new ADR) and update the dependent capability or MVP boundary.
- Do not implement a capability whose **gate** questions are unresolved.
- Re-open a settled decision only through a documented charter-level change.
