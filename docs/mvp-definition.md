# MVP Definition

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Operating Assumption

The customer operates the CoE **Core Components and Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)**. CoE is authoritative for app/flow/maker/owner/environment inventory (Core Components), usage telemetry (Audit Components/Audit Logs), dependency relationships, and governance dashboards. The MVP therefore builds **only** the missing licensing layer and reuses or extends everything CoE already provides. Capability IDs (C0–C8) reference the Capability Catalog in `docs/product-definition.md`.

## MVP Thesis

> Prove that an administrator can go from "who holds a premium license" to a **safe, explainable, audited reclamation of a single eligible license** — end to end — using reused CoE evidence plus supplemented licensing data, without building any parallel inventory, approval workflow, or automatic removal.

The MVP is the **smallest end-to-end slice that demonstrates safe value**. Breadth (bulk at scale, executive dashboards, broader SKUs) is deliberately deferred so that **safety and explainability** can be proven first.

---

## MVP Scope — Capabilities Included

Each in-scope capability restates the three required justifications: **why CoE alone is insufficient**, **business value**, and **reuse / extend / supplement**.

### C0 — CoE discovery and interface validation (prerequisite)

- **Why not CoE alone:** CoE does not validate the external interfaces this product needs or declare its own licensing gap.
- **Business value:** de-risks the MVP; prevents duplication and unsupported-API assumptions.
- **Classification:** Reuse (inspect CoE) + external **Microsoft Graph** validation (ADR-004).
- **MVP boundary:** Largely satisfied by `docs/coe-reuse-analysis.md`; MVP requires confirming exact CoE artifact names/schemas and completing the Graph licensing interface validation.

### C1 — Licensing entitlement acquisition

- **Why not CoE alone:** CoE holds no per-user premium license assignment or SKU data.
- **Business value:** the foundation — you cannot optimize what you cannot see.
- **Classification:** Supplement (Microsoft Graph).
- **MVP boundary:** Premium Power Apps per-user entitlement in scope; **direct vs. group-based** assignment must be distinguished. Broader SKU coverage is out of MVP.

### C2 — Licensing↔CoE correlation

- **Why not CoE alone:** neither CoE nor the licensing source joins entitlement to resources, owners, and usage.
- **Business value:** produces an actionable, owner- and usage-aware view; surfaces unmatched/stale records.
- **Classification:** Extend (project-owned adapter over reused CoE data).
- **MVP boundary:** Correlation for the in-scope premium population; unmatched records are shown, not hidden.

### C3 — License-centric optimization analytics

- **Why not CoE alone:** CoE inactivity is resource-centric, not per-user-license.
- **Business value:** quantifies reclaimable premium capacity with disclosed assumptions.
- **Classification:** Extend (reuse CoE Audit usage; add license-centric analytic).
- **MVP boundary:** Per-user inactivity using a **documented threshold and the CoE usage-history window**; estimates disclose assumptions and freshness; missing usage never treated as zero.

### C4 — Reclamation-safety dependency analysis

- **Why not CoE alone:** CoE shows relationships but does not interpret them for license-removal safety.
- **Business value:** prevents breakage and accidental removals — the core safety outcome.
- **Classification:** Extend (reuse CoE relationships/ownership; add safety rules).
- **MVP boundary:** Application, flow, and ownership dependencies evaluated before any recommendation.

### C5 — Explainable Safe/Review/Blocked recommendations

- **Why not CoE alone:** CoE has no license-reclamation recommendation or explainability model.
- **Business value:** defensible, trusted, auditable decisions.
- **Classification:** Build new.
- **MVP boundary:** Full Safe/Review/Blocked classification with evidence, freshness, dependencies, assignment source, exceptions, uncertainty, and governing rule. Missing/stale evidence blocks Safe.

### C6 — Safe, administrator-initiated reclamation (scoped for MVP)

- **Why not CoE alone:** CoE performs no guardrailed per-user license reclamation.
- **Business value:** realizes the savings safely — the MVP payoff.
- **Classification:** Build new (+ supported-API supplement for the license change).
- **MVP boundary:** Dry run is **always** available. MVP includes **administrator-initiated reclamation of a single, eligible, directly assigned license** with protected-identity and group-assignment exclusion enforced. **Bounded bulk reclamation is a fast-follow** (post-MVP) to keep the initial blast radius minimal while safety is proven.

### C7 — Reclamation audit trail

- **Why not CoE alone:** CoE does not record license-reclamation actions.
- **Business value:** compliance, accountability, rollback evidence; required for security acceptance.
- **Classification:** Build new.
- **MVP boundary:** Every dry run and executed reclamation produces an audit record (actor, target, evidence, action, timestamp, outcome); partial failures are explicit.

### C8 — Licensing visibility (minimal for MVP)

- **Why not CoE alone:** CoE dashboards do not show license optimization or realized-vs-estimated savings.
- **Business value:** operational action for admins; foundation for the value narrative.
- **Classification:** Extend / surface (extend CoE reporting first).
- **MVP boundary:** A **minimal operational view** of correlated utilization and optimization candidates, extending existing CoE reporting. Rich executive/trend dashboards are a fast-follow.

---

## Explicitly Out of MVP (Fast-Follow, Post-MVP)

- Bounded **bulk** reclamation at scale (MVP proves single-candidate safety first).
- Executive and trend/time-series dashboards (C8 breadth).
- Broader SKU coverage beyond the initial premium scope (C1 breadth).
- Group-assignment **advisory** beyond detect-and-block (recommending the correct group-administration path).
- Finance/FinOps export or integration for realized-savings tracking.
- Richer dependency insight (for example deeper connection-reference context).

## Explicitly Never (Charter Exclusions, Not a Timeline Question)

- Business **approval workflows** / sign-off orchestration (ADR-002).
- **Automatic or unattended** license removal.
- A **parallel inventory** or governance platform duplicating CoE (charter Principle 1).
- Use of **unsupported or undocumented** interfaces (ADR-004).
- Modifying Microsoft-managed CoE assets.

---

## MVP Acceptance Criteria (Definition of Done)

Given a tenant running the CoE Core Components and Audit Components and a validated Microsoft Graph licensing interface, the MVP is done when an authorized administrator can:

1. See **correlated premium utilization** for the in-scope population, with data freshness disclosed and unmatched records visible.
2. Open a candidate and view an **explainable Safe/Review/Blocked** classification showing evidence, freshness, dependencies, assignment source, exceptions, uncertainty, and the governing rule.
3. Confirm that **missing or stale evidence blocks a Safe** classification.
4. Confirm that **group-assigned and protected identities** (service, break-glass, shared, critical owner) are excluded with the controlling reason shown.
5. Run a **dry run** that shows exactly what would change and why.
6. Execute an **administrator-initiated reclamation of a single eligible directly assigned license**, producing an **audit record** and surfacing any partial failure.
7. Confirm that **nothing is removed automatically** and that **no approval workflow** exists anywhere in the flow.
8. Confirm that **no parallel inventory** was created and that CoE-managed assets were not modified.

## MVP Prerequisites and Dependencies

- Core Components and Audit Components deployed and operational; inventory and Audit usage reasonably fresh (freshness recorded).
- Microsoft Graph licensing interface validated for support, least-privilege permissions, licensing, throttling, and availability (ADR-004).
- Authoritative source for **protected identities** (service/break-glass/shared/critical owner) confirmed or configured.
- Agreed **inactivity definition** (threshold + signals) and confirmation that the CoE usage-history window covers it. Open items tracked in `docs/open-questions.md`.

## MVP Success Metrics

- Premium licenses in scope with correlated usage evidence (coverage, with freshness).
- Explainable candidates produced (Safe/Review/Blocked), with zero Safe classifications on missing/stale evidence.
- Accidental-removal blocks demonstrated (protected-identity and group-assignment exclusions that prevented unsafe action).
- At least one safe, audited single-license reclamation completed end to end in a controlled setting.
- CoE reuse ratio: share of MVP capability satisfied by reuse/extend vs. build-new.

## MVP Risks (See `docs/product-definition.md` §5 for the full register)

- Usage/licensing freshness gaps producing unsafe recommendations — mitigated by freshness disclosure and Safe-blocking.
- Group-assignment mishandling — mitigated by detect-and-exclude.
- Scope creep toward bulk/approval/automation — mitigated by the explicit exclusions above.
- Unvalidated Graph interface — mitigated by making C0 a hard prerequisite.
