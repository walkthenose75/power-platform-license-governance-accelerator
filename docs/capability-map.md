# Capability Map — Reuse vs. Build

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose

This document exists to **prove exactly what is reused from CoE versus what is built new**. It maps every solution capability to three columns — **CoE Capability**, **Project Extension**, and **New Capability** — and records, for each, the **User Need**, **Capability**, **Data Source**, **Owner**, and **Reuse Strategy**.

It assumes the customer operates the CoE **Core Components and Audit Components (CenterOfExcellenceAuditComponents)**. Capabilities are the canonical set C0–C8 from the Capability Catalog in `docs/product-definition.md`; traceability points to `docs/coe-reuse-analysis.md` (§), `docs/app-navigation.md` (tables/UX), `docs/dashboard-design.md` (analytics), and `docs/data-model.md` (schema).

### How to read this document

- **CoE Capability** — the part already provided by CoE and **reused/surfaced** as-is.
- **Project Extension** — project-owned logic/visuals built **on top of** reused CoE data, with no change to managed CoE assets.
- **New Capability** — functionality with **no CoE equivalent** (supplemented via supported API or built new).
- **Reuse Strategy** uses the preference order: **reuse as-is (1) → read via adapter (2) → extend (3) → supplement via supported API (4) → build new (5)**.
- **Owner** — the accountable system/team: **CoE** (Microsoft-managed, reused), **Project** (License Governance team, project-owned), or **External** (for example Microsoft Graph / Entra as a source).

---

## Master Capability Map

| ID | Capability | User Need (persona) | CoE Capability (reuse) | Project Extension | New Capability | Reuse Strategy |
| --- | --- | --- | --- | --- | --- | --- |
| **C0** | CoE discovery & interface validation | Know what CoE already provides before building (P1) | Core/Audit components are the subject of discovery | Project-owned gap record | External interface validation (Graph) | Reuse (inspect) + validate (1) |
| **C1** | Licensing entitlement acquisition | Know who holds a premium license and on what basis (P2, P1) | None (CoE has no entitlement data); CoE user identity aids correlation | — | Premium assignment + SKU + direct/group source | Supplement via Graph (4) |
| **C2** | Licensing↔CoE correlation | See licensing joined to apps/flows/owners/usage (P1, P4) | Inventory, ownership, usage to correlate against | Correlation/adapter logic over reused CoE data | Correlation status + unmatched surfacing | Reuse via adapter → extend (2→3) |
| **C3** | License-centric optimization analytics | Know which licensed users are inactive and the opportunity (P2, P3, P1) | Resource-level usage telemetry (Audit) | License-centric analytic over reused usage | Per-user inactivity + estimated value | Reuse usage → extend (2→3) |
| **C4** | Reclamation-safety dependency analysis | Know dependencies before removing a license (P1, P5) | Ownership/relationship data | Safety interpretation over reused relationships | Dependency findings + blocking rules | Reuse relationships → extend (2→3) |
| **C5** | Explainable Safe/Review/Blocked recommendations | Get defensible, explainable decisions (P1, P5) | None (consumes reused CoE evidence) | Interpretation of reused dependency evidence | Classification + itemized evidence model | Build new over reused inputs (5) |
| **C6** | Safe, administrator-initiated reclamation | Reclaim eligible licenses safely (P1 admin) | None (CoE compliance is resource-only; not repurposed) | — | Dry run, guardrails, bounded bulk, action record | Build new + supplement (4→5) |
| **C7** | Reclamation audit trail | Prove what was reclaimed and why (P5, P1) | None (CoE audits its own objects, not reclamation) | — | Immutable, dashboardable audit | Build new (5) |
| **C8** | Licensing visibility (operational + executive) | See optimization value and where to act (P6, P3, P1) | CoE Power BI governance + usage/adoption | Licensing measures on reused CoE model | Operational triage + optimization visuals | Extend / surface (3) |

---

## Capability Detail

Each block lists all requested fields: **User Need, Capability, CoE Capability, Project Extension, New Capability, Data Source, Owner, Reuse Strategy.**

### C0 — CoE discovery and interface validation

- **User Need:** P1 Platform/CoE Admin — confirm what CoE already provides and that external interfaces are supported before any build.
- **CoE Capability (reuse):** CoE Core/Audit components (inventory, usage, dashboards) are the subject being inspected.
- **Project Extension:** a project-owned capability-gap record.
- **New Capability:** external Microsoft Graph licensing interface validation (ADR-004).
- **Data Source:** CoE solution/component metadata; Graph interface validation.
- **Owner:** Project (analysis) + CoE (reused subject).
- **Reuse Strategy:** Reuse (inspect), level 1, plus external validation. Reference `docs/coe-reuse-analysis.md` §A, §B.1.

### C1 — Licensing entitlement acquisition

- **User Need:** P2 Licensing/Procurement and P1 — know who holds a premium license, which SKU, and whether it is direct or group-assigned.
- **CoE Capability (reuse):** none for entitlement; CoE user/maker identity is reused as a correlation anchor.
- **Project Extension:** none directly (entitlement is external to CoE).
- **New Capability:** acquisition of premium assignment, SKU, and assignment source.
- **Data Source:** Microsoft Graph (licensing); project **License Assignment** and **SKU Reference** tables.
- **Owner:** Project (tables/logic); External (Microsoft Graph as source).
- **Reuse Strategy:** Supplement via Graph, level 4. Reference §B.2.

### C2 — Licensing↔CoE correlation

- **User Need:** P1, P4 — see licensing joined to the apps, flows, owners, and usage CoE already knows.
- **CoE Capability (reuse):** CoE inventory, ownership, and usage (the data correlated against).
- **Project Extension:** project-owned correlation/adapter logic over reused CoE data via canonical IDs.
- **New Capability:** correlation status and explicit unmatched/ambiguous surfacing.
- **Data Source:** CoE inventory/usage/ownership (read-only) + project **License Assignment**; canonical IDs.
- **Owner:** Project (correlation) reusing CoE (source of record).
- **Reuse Strategy:** Reuse via adapter → extend, levels 2→3. Reference §B.3.

### C3 — License-centric optimization analytics

- **User Need:** P2, P3, P1 — identify potentially inactive licensed users and quantify the optimization opportunity.
- **CoE Capability (reuse):** CoE Audit **resource-level** usage telemetry.
- **Project Extension:** a license-centric (per-user premium) analytic layered on reused usage.
- **New Capability:** inactivity determination with evidence coverage and disclosed estimated value.
- **Data Source:** CoE Audit usage (reuse) + **License Assignment** + **Optimization Candidate**.
- **Owner:** Project (analytic) reusing CoE (usage).
- **Reuse Strategy:** Reuse usage → extend, levels 2→3. Reference §B.4. (Missing usage ≠ zero usage.)

### C4 — Reclamation-safety dependency analysis

- **User Need:** P1, P5 — understand app/flow/ownership dependencies before removing a license to avoid breakage.
- **CoE Capability (reuse):** CoE ownership and relationship data.
- **Project Extension:** project-owned safety interpretation of reused relationships.
- **New Capability:** dependency findings with criticality and blocking determination.
- **Data Source:** CoE ownership/relationships (read-only) + project **Dependency Finding**; CoE criticality if available (Q-COE-3).
- **Owner:** Project (interpretation) reusing CoE (relationships).
- **Reuse Strategy:** Reuse relationships → extend, levels 2→3. Reference §B.5.

### C5 — Explainable Safe/Review/Blocked recommendations

- **User Need:** P1, P5 — defensible, explainable classifications an administrator can trust and justify.
- **CoE Capability (reuse):** none for recommendations; reused CoE evidence is an input.
- **Project Extension:** interpretation of reused dependency/usage evidence into a licensing decision.
- **New Capability:** the Safe/Review/Blocked classification and itemized explainability model.
- **Data Source:** project **Optimization Candidate** + **Recommendation Evidence**, consuming reused CoE and supplemented licensing inputs.
- **Owner:** Project.
- **Reuse Strategy:** Build new over reused inputs, level 5. Missing/stale evidence blocks Safe. Reference §B.6.

### C6 — Safe, administrator-initiated reclamation

- **User Need:** P1 (authorized admin) — reclaim eligible, directly assigned licenses safely with a dry run and guardrails.
- **CoE Capability (reuse):** none; CoE compliance/clean-up is resource-oriented and must not be repurposed.
- **Project Extension:** none against CoE-managed assets.
- **New Capability:** dry run, protected-identity and group-assignment exclusion, bounded bulk, and the action record — the license change executed via a supported API.
- **Data Source:** project **Reclamation Action**; supported Graph/admin API for the change (ADR-004).
- **Owner:** Project; External (supported API performs the change).
- **Reuse Strategy:** Build new + supplement, levels 4→5. No approval workflow; no automatic removal. Reference §B.7.

### C7 — Reclamation audit trail

- **User Need:** P5, P1 — an immutable record proving what was reclaimed, by whom, on what evidence, and with what outcome.
- **CoE Capability (reuse):** none; CoE audits its own objects, not license reclamation.
- **Project Extension:** none.
- **New Capability:** immutable, dashboardable audit with a point-in-time evidence snapshot.
- **Data Source:** project **Audit Log Entry** (append-only).
- **Owner:** Project.
- **Reuse Strategy:** Build new, level 5. Reference §B.7–§B.8.

### C8 — Licensing visibility (operational + executive)

- **User Need:** P6, P3, P1 — executive value narrative and operational "where to act," without rebuilding CoE dashboards.
- **CoE Capability (reuse):** CoE Power BI governance dashboards and usage/adoption datasets.
- **Project Extension:** licensing/optimization measures on the reused CoE semantic model; deep links to CoE.
- **New Capability:** operational triage dashboard and optimization visuals over project tables.
- **Data Source:** CoE Power BI model + usage (reuse) + project tables (new) + Graph.
- **Owner:** Project (visuals) reusing CoE (model/datasets).
- **Reuse Strategy:** Extend / surface, level 3. Do not rebuild CoE dashboards (risks D-2/D-3). Reference §B.8.

---

## Data Source and Ownership Catalog

Proof of ownership for every data source the solution touches.

| Data source | Provides | Owner | Classification |
| --- | --- | --- | --- |
| CoE inventory (apps/flows/makers/environments) | Resource and ownership facts | CoE | Reuse |
| CoE Audit usage telemetry | App/flow usage, last-launched, unique users | CoE | Reuse |
| CoE ownership/relationships | Dependency inputs | CoE | Reuse |
| CoE Power BI semantic model / dashboards | Governance analytics | CoE | Reuse (extend) |
| CoE sync health | Freshness signals | CoE | Reuse |
| Microsoft Graph licensing | Premium assignment, SKU, direct/group source | External (Microsoft) | Supplement |
| Entra identity | User object IDs (correlation anchor) | External (Microsoft) | Reuse (reference) |
| License Assignment, SKU Reference | Licensing facts | Project | New |
| Optimization Candidate, Recommendation Evidence, Dependency Finding | Recommendation + explainability | Project | New |
| Reclamation Action, Audit Log Entry | Remediation + accountability | Project | New |
| Protected Identity / Exception | Safety control | Project | New |

---

## Reuse-vs-Build Scorecard (The Proof)

| Capability | Dominant posture | Reused from CoE | Built new |
| --- | --- | --- | --- |
| C0 Discovery | **Reuse** | Inventory/usage/dashboards (inspected) | Gap record, interface validation |
| C1 Acquisition | **Build (supplement)** | User identity (anchor) | Entitlement/SKU/source |
| C2 Correlation | **Reuse → Extend** | Inventory/usage/ownership | Correlation logic + unmatched surfacing |
| C3 Analytics | **Reuse → Extend** | Usage telemetry | License-centric inactivity + value |
| C4 Dependency | **Reuse → Extend** | Relationships/ownership | Safety findings/rules |
| C5 Recommendations | **Build** | Evidence inputs | Classification + explainability |
| C6 Reclamation | **Build (+supplement)** | — | Dry run, guardrails, action |
| C7 Audit | **Build** | — | Immutable audit |
| C8 Visibility | **Extend / Surface** | Power BI model, usage, dashboards | Licensing measures + triage view |

**Summary of proof:**

- **Data substrate is overwhelmingly reused/extended from CoE** — all inventory, usage, ownership, and base analytics come from CoE (C0, C2, C3, C4, C8).
- **Net-new build is confined to four areas** CoE does not provide: **licensing acquisition (C1), recommendation/explainability (C5), safe reclamation (C6), and audit (C7)** — plus thin extension logic for correlation, analytics, dependency, and visibility.
- **No capability re-creates a CoE capability.** Where CoE already provides a fact or experience, this solution reuses, extends, or deep-links; it never duplicates (charter Principle 1; ADR-001).
- **CoE reuse ratio:** of the nine capabilities, **five are reuse/extend-dominant** and **four are build-dominant**, and even the build-dominant capabilities consume reused CoE evidence rather than rebuilding it.

---

## Capability → Artifact Traceability

Where each capability is realized, and whether each artifact is Reuse (R), Extend (E), or New (N).

| Capability | Backlog epic | Project tables (`docs/data-model.md`) | UX (`docs/app-navigation.md`) | Analytics (`docs/dashboard-design.md`) |
| --- | --- | --- | --- | --- |
| C0 | Epic 1 | — | CoE Context deep links (R) | Data & Interface Status (N, references R) |
| C1 | Epic 2 | License Assignment, SKU Reference (N) | License Assignments (N) | Direct vs. group split (N) |
| C2 | Epic 3 | License Assignment correlation status (N) over CoE (R) | Dependencies, Data Quality (R+E/N) | Unmatched rate (N) |
| C3 | Epic 4 | Optimization Candidate (N) over CoE usage (R) | Optimization Overview (E/N) | Utilization, inactive (E/N) |
| C4 | Epic 5 (precond.) | Dependency Finding (N) over CoE relationships (R) | Candidate → Dependencies tab (R+N) | Dependency hotspots (N over E) |
| C5 | Epic 5 | Optimization Candidate, Recommendation Evidence (N) | Candidate Evidence tab (N) | Candidates by classification (N) |
| C6 | Epic 6 | Reclamation Action (N) | Reclamation area, commands (N) | Reclamation pipeline (N) |
| C7 | Epic 6 | Audit Log Entry (N) | Audit Log (N) | Audit activity over time (N) |
| C8 | Epic 4 | reads project tables (N) + CoE model (R) | Overview area (E) | Executive Summary (E over R) |

---

## Cross-References

- Capability definitions and justifications: `docs/product-definition.md` (Capability Catalog), `docs/coe-reuse-analysis.md` (§A–§B).
- Backlog alignment and Reuse Strategy per epic: `docs/backlog.md`.
- UX realization and anti-duplication: `docs/app-navigation.md`.
- Analytics realization and CoE Power BI reuse: `docs/dashboard-design.md`.
- Minimum schema and ownership: `docs/data-model.md`.
- Open decisions affecting reuse/build: `docs/open-questions.md`.

---

## Constraints Honored

- Design documentation only — no Power Platform assets, schemas, or flows were generated.
- Every capability is explicitly classified across CoE Capability, Project Extension, and New Capability, with owner and data source, proving reuse versus build.
- No capability duplicates CoE; CoE remains the system of record for inventory, usage, ownership, and governance dashboards.
