# Product Definition

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## 1. Product Definition

### 1.1 Product Name

Power Platform License Governance Accelerator.

### 1.2 One-Line Definition

A CoE Toolkit extension that correlates Microsoft Graph licensing data with existing CoE inventory and usage to surface explainable Power Apps Premium optimization opportunities and enable safe, administrator-initiated license reclamation.

### 1.3 Problem Statement

Organizations that have invested in Power Apps Premium capacity frequently cannot answer three operational questions with confidence:

- Who holds a premium license?
- Is that license actually being used?
- Can it be reclaimed without breaking a critical app, flow, or ownership dependency?

The CoE Toolkit already inventories apps, flows, makers, owners, environments, and usage. It does not, on its own, correlate that inventory with assigned commercial licensing in a way that produces safe, explainable reclamation decisions. Administrators are left performing manual, error-prone license audits that risk removing a license from a service account, a break-glass identity, a shared mailbox owner, or a critical application owner.

### 1.4 Product Vision

Extend the customer's existing CoE investment so that license optimization becomes a routine, evidence-based, low-risk operation rather than a periodic manual audit. The accelerator surfaces what CoE already knows, adds only the licensing intelligence CoE lacks, and governs every reclamation action with dependency checks, dry runs, explainability, and audit.

### 1.5 What This Product Is

- A CoE Toolkit **extension**, not a replacement.
- A **licensing intelligence** layer over existing CoE inventory and usage.
- An **analytics and recommendation** surface that classifies candidates as Safe, Review, or Blocked.
- A **safety-governed remediation** capability for eligible, directly assigned licenses.

### 1.6 What This Product Is Not

- Not a new or parallel governance platform.
- Not a replacement for, or re-implementation of, CoE inventory, usage, or governance.
- Not a business approval engine or workflow orchestration product.
- Not an automatic or unattended license-removal system.
- Not a consumer of undocumented endpoints or direct CoE data manipulation.

### 1.7 Strategic Alignment with the Charter

| Charter Principle | Product Commitment |
| --- | --- |
| Do not build a new governance platform | Deliver only the licensing layer missing from CoE. |
| Reuse if it exists in CoE | Consume CoE apps, flows, makers, owners, environments, and usage as the source of truth. |
| Surface if it can be surfaced | Prefer extending CoE dashboards and experiences before building new ones. |
| Only build what is missing | Scope new build to license inventory, analytics, optimization, and reclamation. |
| Safety first | Guard every remediation with dependency checks, dry run, protected-identity rules, and audit. |
| Explainability | Make every recommendation traceable to its evidence, freshness, and rule. |

### 1.8 Value Proposition by Stakeholder

- **Platform / CoE administrators:** Faster, safer, repeatable license optimization grounded in existing CoE data.
- **Licensing and procurement:** Evidence-based reclamation that defers or reduces premium license purchases.
- **IT finance / FinOps:** Quantified, assumption-disclosed savings opportunities.
- **Security and compliance:** Protected identities, auditable actions, and no weakening of existing governance controls.
- **Executive sponsors:** Operational and executive visibility into premium license utilization and realized value.

### 1.9 Success Metrics

| Metric | Intent | Notes |
| --- | --- | --- |
| Premium licenses with correlated usage evidence | Coverage of the intelligence layer | Reported with data-freshness disclosure. |
| Potential reclamation opportunities identified | Optimization value | Estimated, not guaranteed; assumptions disclosed. |
| Manual audit effort reduced | Operational efficiency | Measured qualitatively and by audit cycle time. |
| Accidental removals avoided | Safety outcome | Protected-identity and dependency blocks that prevented unsafe action. |
| Realized reclamations vs. estimated | Value realization | Separates estimate from realized savings. |
| CoE reuse ratio | Charter adherence | Share of capability satisfied by reuse/surfacing vs. new build. |

---

## Capability Catalog — Build Only What Is Missing

**Operating assumption:** the customer operates the CoE **Core Components and Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)**. CoE is the authoritative source for app/flow/maker/owner/environment inventory (Core Components), usage telemetry (Audit Components/Audit Logs), dependency relationships, and governance dashboards. The capabilities below are the **only** net-new or extension work this product introduces — the licensing layer CoE does not provide. Each is justified by (a) **why CoE alone cannot satisfy it**, (b) **the business value**, and (c) whether it **reuses, extends, or supplements** CoE.

**Classification legend** (reuse preference order): **Reuse** (consume CoE as-is / via adapter) → **Extend** (project-owned components over CoE, no managed changes) → **Supplement** (supported external API where CoE holds no data) → **Build new** (no CoE equivalent).

| ID | Capability | Classification | CoE-reuse ref | Backlog |
| --- | --- | --- | --- | --- |
| C0 | CoE discovery & interface validation (prerequisite) | Reuse + external validation | §A, §B.1 | Epic 1 |
| C1 | Licensing entitlement acquisition | Supplement (Graph) | §B.2 | Epic 2 |
| C2 | Licensing↔CoE correlation | Extend | §B.3 | Epic 3 |
| C3 | License-centric optimization analytics | Extend | §B.4 | Epic 4 |
| C4 | Reclamation-safety dependency analysis | Extend | §B.5 | Epic 5 (precondition) |
| C5 | Explainable Safe/Review/Blocked recommendations | Build new | §B.6 | Epic 5 |
| C6 | Safe, administrator-initiated reclamation | Build new (+ Graph supplement) | §B.7 | Epic 6 |
| C7 | Reclamation audit trail | Build new | §B.7–§B.8 | Epic 6 |
| C8 | Licensing visibility (operational + executive) | Extend / surface | §B.8 | Epic 4 |

§ references point to `docs/coe-reuse-analysis.md`.

### C0 — CoE discovery and interface validation (prerequisite)

- **Why not CoE alone:** CoE does not describe its own licensing gap or validate external interfaces; identifying what is missing and confirming supported interfaces is an analysis the project must own.
- **Business value:** prevents duplication and unsafe assumptions, and de-risks every downstream capability before any build.
- **Classification:** **Reuse** (inspect CoE) plus external **Microsoft Graph** interface validation (ADR-004). No CoE-equivalent build.

### C1 — Licensing entitlement acquisition

- **Why not CoE alone:** CoE inventories resources and usage, not commercial **license entitlements**. Per-user premium assignment, SKU, and whether an assignment is **direct vs. group-based** are Microsoft 365 / Graph facts absent from CoE (`§B.2`). Without them you cannot tell who holds a premium license or on what basis.
- **Business value:** the indispensable foundation — you cannot optimize or reclaim what you cannot see; it enables every downstream analytic and the savings case.
- **Classification:** **Supplement** via Microsoft Graph (validated per ADR-004). It acquires entitlement data only, not a copy of CoE-owned inventory.

### C2 — Licensing↔CoE correlation

- **Why not CoE alone:** CoE holds apps/flows/owners/usage but has no join to license entitlement; the licensing source has no join to CoE resources. Neither side alone answers "is this premium license backed by real premium-requiring activity and ownership?"
- **Business value:** converts a raw license list into an owner-, usage-, and dependency-aware picture an administrator can act on, and makes unmatched or stale data visible instead of hidden.
- **Classification:** **Extend** — project-owned correlation logic reading CoE through a stable-identifier adapter; no copy of CoE inventory.

### C3 — License-centric optimization analytics

- **Why not CoE alone:** CoE inactivity is **resource-centric** (an app/flow is unused/orphaned). It does not compute **per-user premium-license** inactivity or quantify reclaimable premium capacity (`§B.4`).
- **Business value:** turns utilization evidence into a quantified, assumption-disclosed optimization opportunity — the number procurement and FinOps need.
- **Classification:** **Extend** — reuse CoE Audit usage telemetry; add the license-centric analytic. Missing usage is never treated as zero usage.

### C4 — Reclamation-safety dependency analysis

- **Why not CoE alone:** CoE shows ownership and relationships but does not **interpret** them for license-removal safety (e.g., "this user is the sole owner of a business-critical flow, so removing their premium license is unsafe") (`§B.5`).
- **Business value:** the core safety capability — it prevents breaking production workloads and prevents accidental removals, a headline success metric.
- **Classification:** **Extend** — reuse CoE relationship and ownership data; add project-owned safety-interpretation rules.

### C5 — Explainable Safe/Review/Blocked recommendations

- **Why not CoE alone:** CoE has no license-reclamation recommendation engine and no explainability model spanning assignment source, dependencies, exceptions, freshness, and the governing rule (`§B.6`).
- **Business value:** defensible, auditable, trusted decisions — administrators act with confidence and can justify each action.
- **Classification:** **Build new** — consumes reused CoE and supplemented licensing inputs. Missing or stale evidence blocks a Safe classification.

### C6 — Safe, administrator-initiated reclamation

- **Why not CoE alone:** CoE compliance/clean-up is resource-oriented and must not be repurposed; CoE performs no per-user license reclamation with dry run, protected-identity and group-assignment exclusion, and bounded bulk (`§B.7`).
- **Business value:** realizes the savings safely — the payoff of the initiative — without an approval engine or automatic removal.
- **Classification:** **Build new**, using a **supported API** to perform the license change (validated per ADR-004). It never modifies CoE-managed flows.

### C7 — Reclamation audit trail

- **Why not CoE alone:** CoE does not record license-reclamation actions (actor, evidence, action, timestamp, outcome, partial failures).
- **Business value:** compliance, accountability, and rollback/forensic evidence; a prerequisite for security acceptance of any reclamation.
- **Classification:** **Build new** — a project-owned audit record isolated in the project solution.

### C8 — Licensing visibility (operational and executive)

- **Why not CoE alone:** CoE dashboards visualize governance, not license optimization opportunity or realized-vs-estimated savings (`§B.8`).
- **Business value:** operational action for administrators and a credible value narrative for executives and finance.
- **Classification:** **Extend / surface** — extend existing CoE reporting first (ADR-003); build a separate view only for a documented licensing-specific gap.

---

## 2. Functional Requirements

Functional requirements extend and refine `docs/requirements.md` and implement the Capability Catalog (C0–C8) defined earlier in this document. Each is written as a product capability, not an implementation. CoE remains the source of truth for all inventory, ownership, environment, and usage data.

### FR-1 CoE Capability Discovery

- FR-1.1 Discover the deployed CoE version, enabled components, and available inventory, ownership, environment, usage, dashboard, and governance capabilities.
- FR-1.2 Record capability gaps that cannot be met by surfacing or extending CoE.
- FR-1.3 Treat discovery output as a precondition for any new storage, analytics, experience, or remediation capability.

### FR-2 Licensing Intelligence

- FR-2.1 Acquire assigned Power Apps premium licensing information not already available through CoE, using validated supported interfaces.
- FR-2.2 Acquire the SKU reference information required to interpret assignments.
- FR-2.3 Distinguish assignment context — direct, group-based, service, shared, and exception — where the supported source provides it.
- FR-2.4 Disclose the freshness and completeness of acquired licensing data.

### FR-3 Correlation with CoE

- FR-3.1 Correlate licensing records with existing CoE app, flow, maker, owner, environment, and usage data through supported, documented identifiers.
- FR-3.2 Surface unmatched or ambiguous records explicitly rather than silently discarding them.
- FR-3.3 Remain resilient to documented CoE version and configuration differences.

### FR-4 Optimization Analytics

- FR-4.1 Identify potentially inactive licensed users from explainable evidence.
- FR-4.2 Analyze application, flow, ownership, and other relevant dependencies before any reclamation is proposed.
- FR-4.3 Estimate optimization opportunities with disclosed assumptions and data freshness, never presented as guaranteed savings.
- FR-4.4 Prefer extending or reusing existing CoE reporting before introducing a new dashboard.

### FR-5 Explainable Recommendations

- FR-5.1 Classify each candidate as Safe, Review, or Blocked using documented rules.
- FR-5.2 For every classification, expose data sources, evidence freshness, dependencies, assignment source, applied exceptions, uncertainty, and the rule that produced the result.
- FR-5.3 Prevent a Safe classification when required evidence is missing or stale.

### FR-6 Safe Remediation

- FR-6.1 Support administrator-initiated reclamation of eligible, directly assigned licenses only after dependency checks and a dry run.
- FR-6.2 Exclude group-assigned licenses from direct removal and explain the controlling assignment source.
- FR-6.3 Protect service, break-glass, shared, group-assigned, and critical-owner identities from removal.
- FR-6.4 Support bounded bulk remediation without introducing a business approval workflow.
- FR-6.5 Record auditable outcomes and expose partial failures explicitly.

### FR-7 Visibility and Audit

- FR-7.1 Provide operational and executive visibility into premium utilization and optimization opportunities.
- FR-7.2 Maintain an auditable record of recommendations acted upon, including actor, evidence, action, timestamp, and outcome.
- FR-7.3 Make failures visible and actionable rather than suppressed.

### Out-of-Scope Functionality

- Approval workflows or business sign-off orchestration.
- Automatic or scheduled unattended license removal.
- Re-implementation of CoE inventory, usage, or governance.
- Writing back to or mutating CoE internals.
- Chargeback/showback billing engines (future consideration only).

---

## 3. Nonfunctional Requirements

Nonfunctional requirements extend `docs/requirements.md`. They constrain *how* the product behaves, independent of specific features.

### NFR-1 Architecture and Reuse

- Extension-first: CoE remains authoritative; the accelerator surfaces and extends before it builds.
- No duplication of CoE inventory without a documented gap, data-ownership rationale, retention need, and reconciliation strategy.
- Clear isolation between any CoE adapter and licensing-domain logic to limit version coupling.

### NFR-2 Supported Interfaces Only

- Use only documented, supported Microsoft interfaces (Microsoft Graph, supported Power Platform APIs, supported admin connectors, existing CoE data).
- Validate support status, permissions, licensing, throttling, data availability, and tenant compatibility before adoption.
- No undocumented endpoints, portal scraping, or direct database manipulation.

### NFR-3 Safety and Governance

- No automatic license removal; all remediation is explicitly administrator-initiated.
- Dry run and dependency checks precede any reclamation.
- Protected-identity rules are enforced and cannot be silently bypassed.
- Missing or stale evidence blocks Safe classification.

### NFR-4 Security and Privacy

- Least privilege for both data acquisition and remediation; separate read-only analytics from remediation permissions where practical.
- No secrets, tokens, tenant-specific identifiers, or personal data committed to source.
- Minimize stored personal and licensing data; disclose sensitivity and retention.
- Preserve existing DLP, tenant, environment, and security policies; never weaken a control to make a feature work.

### NFR-5 Reliability and Data Quality

- Handle pagination, throttling, retries, and partial success for every integration.
- Disclose data freshness and completeness wherever recommendations are presented.
- Surface unmatched records and failed checks instead of hiding them.

### NFR-6 Auditability and Explainability

- Every recommendation is traceable to evidence, freshness, dependencies, assignment source, exceptions, and the governing rule.
- Every remediation produces an auditable record with actor, target, evidence, action, timestamp, and outcome.

### NFR-7 ALM and Operability

- Solution-aware and ALM-friendly for any justified extension component.
- Environment variables and connection references for environment-specific configuration.
- Documented prerequisites, rollback, and validation for high-impact operations.

### NFR-8 Extensibility and Maintainability

- Extensible without coupling to undocumented CoE internals.
- Resilient to documented CoE version and configuration variance.
- Configuration-driven protected-identity and exception handling.

### NFR-9 Accessibility and Usability

- Administrator experiences meet accessibility expectations and present evidence clearly.
- Recommendations are understandable without requiring the administrator to reconstruct the underlying query.

---

## 4. Future Roadmap

The roadmap is directional and subject to CoE discovery and interface validation. Nothing in the roadmap authorizes approval workflows or automatic license removal, which remain charter-level exclusions.

### Horizon 1 — Foundation and MVP (see `docs/mvp-definition.md`)

- CoE discovery and gap documentation.
- Licensing intelligence for Power Apps premium via validated interfaces.
- Correlation with CoE inventory and usage.
- Explainable Safe / Review / Blocked recommendations.
- Read-only optimization visibility, reusing or extending CoE reporting.
- Dry-run, guardrailed, administrator-initiated reclamation for directly assigned licenses with audit.

### Horizon 2 — Breadth and Insight

- Broader SKU coverage beyond the initial premium scope, subject to interface validation.
- Richer dependency insight (for example, deeper flow and connection-reference context) sourced from CoE.
- Trend and time-series optimization views built on validated data freshness.
- Group-assignment advisory insight (identifying the controlling group and recommending the correct administrative path) without performing group changes directly.

### Horizon 3 — Scale and Integration

- Multi-environment and multi-region reporting rollups where supported.
- Enhanced reconciliation and data-quality tooling for correlation gaps.
- Optional export/integration with finance/FinOps reporting, respecting data-sensitivity and supported interfaces.

### Deferred / Explicitly Gated

- Any approval workflow — requires a charter-level scope change.
- Any automatic or scheduled unattended remediation — requires a charter-level scope change.
- Chargeback/showback billing — future consideration only, outside current scope.
- Predictive or ML-based inactivity scoring — only after explainability guarantees can be preserved.

---

## 5. Risks

| ID | Risk | Impact | Likelihood | Mitigation |
| --- | --- | --- | --- | --- |
| R-1 | CoE version/configuration variance invalidates assumed data | High | Medium | Make CoE discovery a precondition; isolate version-specific assumptions; document dependencies. |
| R-2 | Unsupported or unavailable interface assumed | High | Medium | Validate every interface (support, permissions, licensing, throttling, availability) before adoption; record evidence. |
| R-3 | Stale or incomplete usage/licensing evidence yields unsafe recommendation | High | Medium | Disclose freshness; block Safe on missing/stale evidence; surface uncertainty. |
| R-4 | Group-assigned license treated as directly removable | High | Medium | Detect assignment source; exclude group-assigned from direct removal; advise correct path. |
| R-5 | Critical owner or break-glass/service/shared identity reclaimed | Critical | Low | Enforce protected-identity rules and dependency checks; dry run before any action. |
| R-6 | Duplication of CoE data creates a competing source of truth | High | Medium | Surface/reference CoE; justify and reconcile any retained copy; avoid parallel inventory. |
| R-7 | Scope creep toward a parallel governance platform or approval engine | High | Medium | Enforce charter exclusions; require documented gap and architecture decision for new capability. |
| R-8 | Over-broad permissions increase security exposure | High | Low | Least privilege; separate read from remediation; audit privileged operations. |
| R-9 | Identifier mismatch produces false dependency conclusions | Medium | Medium | Use supported stable identifiers; surface unmatched records; never silently default to "no dependency." |
| R-10 | Estimated savings misinterpreted as guaranteed | Medium | Medium | Separate estimated from realized; disclose assumptions and freshness in every view. |
| R-11 | Parallel user experience fragments administrator workflow | Medium | Medium | Extend/surface CoE experiences first; justify any separate app by a documented gap. |

---

## 6. Assumptions

| ID | Assumption | Validation Required |
| --- | --- | --- |
| A-1 | The customer has CoE Toolkit deployed and intends to keep using it. | Confirm version and enabled components during discovery. |
| A-2 | CoE inventory, ownership, environment, and usage data are sufficiently complete to support correlation. | Measure coverage and freshness during discovery. |
| A-3 | Supported interfaces exist to acquire the required premium licensing and SKU data for the tenant. | Validate support, permissions, licensing, throttling, and availability (ADR-004). |
| A-4 | Assignment context (direct vs. group vs. service/shared/exception) is obtainable where needed. | Confirm against the supported source per tenant. |
| A-5 | Administrators performing reclamation hold the appropriate least-privilege roles. | Confirm role model with security/platform owners. |
| A-6 | Existing CoE dashboards/experiences can be extended to surface licensing intelligence. | Assess extension feasibility before building new experiences. |
| A-7 | Protected-identity definitions (service, break-glass, shared, critical owners) can be sourced or configured. | Confirm the authoritative source for exception lists. |
| A-8 | Tenant governance (DLP, security, retention) permits the proposed read and remediation operations. | Confirm with security/compliance before execution. |
| A-9 | Estimated savings are acceptable as directional guidance, not financial commitments. | Confirm expectations with finance/procurement stakeholders. |

Open items that are not yet assumptions are tracked in `docs/open-questions.md`.
