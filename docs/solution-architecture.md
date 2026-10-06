# Solution Architecture

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Scope

This document is the **solution architecture design** for the Power Platform License Governance Accelerator, targeted at a CoE environment running **Core Components and Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)**. It is **design documentation only** — it generates no Power Platform assets, solution, schema, flows, or code.

It integrates the other design documents at an architectural level: capabilities (`docs/product-definition.md`, `docs/capability-map.md`), reuse analysis (`docs/coe-reuse-analysis.md`), schema (`docs/data-model.md`), UX (`docs/app-navigation.md`), and analytics (`docs/dashboard-design.md`).

---

## Deployment Baseline

The target CoE environment has exactly these CoE solutions deployed and available, and the accelerator depends on **these two CoE solutions only** (plus Microsoft Graph and its own project-owned solution):

| CoE solution | Provides (reused read-only) | Accelerator dependency |
| --- | --- | --- |
| **Center of Excellence – Core Components** (`CenterofExcellenceCoreComponents`) | Environment, app, flow, maker/owner, connector inventory; relationships/ownership; inventory sync; admin apps | Inventory, ownership, environment, maker facts (C2, C4) |
| **Center of Excellence – Audit Components** (`CenterOfExcellenceAuditComponents`, incl. Audit Logs) | App launches, unique users, last-launched; usage-history window; resource-level inactivity signals | Usage/inactivity evidence (C3, C5) |

The **CoE Power BI governance dashboards** source from Core + Audit data and are therefore available to **reuse/extend** (C8). Nurture, ALM Accelerator, Pipelines, and other kit components are **not required** and are **not assumed**.

> Component availability is settled. The only remaining design-time confirmations are exact artifact names/schemas/versions and the **usage-history window length** (Q-COE-2). See `docs/open-questions.md`.

---

## Architecture Overview

The solution is a **four-layer extension** of the customer's CoE. CoE remains the system of record; the accelerator adds the licensing layer and never duplicates or modifies CoE.

```
┌─────────────────────────────────────────────────────────────────────┐
│  EXPERIENCE LAYER (project-owned)                                     │
│  License Governance model-driven app + dashboards                    │
│  (extends CoE Power BI; deep-links to CoE)            [Extend / New]  │
├─────────────────────────────────────────────────────────────────────┤
│  LOGIC LAYER (project-owned)                                          │
│  Correlation · inactivity analytics · dependency safety ·            │
│  recommendation/explainability · reclamation · audit  [Extend / New] │
├─────────────────────────────────────────────────────────────────────┤
│  DATA LAYER                                                          │
│  Project-owned Dataverse (8 tables, data-model.md)         [New]     │
│  Read-only adapter/query layer over CoE (canonical IDs)   [Reuse]    │
├─────────────────────────────────────────────────────────────────────┤
│  SOURCE-OF-RECORD LAYER                                              │
│  CoE Core Components (inventory/ownership)  ─┐                       │
│  CoE Audit Components / Audit Logs (usage)  ─┼─ read-only [Reuse]    │
│  Microsoft Graph (licensing entitlement)    ─┘  supported [Supplement]│
└─────────────────────────────────────────────────────────────────────┘
```

---

## CoE Dependency Map

What the accelerator consumes, from which CoE solution, via what mechanism, and its classification.

| Consumed fact | CoE source | Mechanism | Correlation key | Classification |
| --- | --- | --- | --- | --- |
| Apps, flows, environments inventory | Core Components | Read-only Dataverse query (adapter) | App/flow/environment ID | Reuse |
| Maker/owner and relationships | Core Components | Read-only query | Entra user object ID + resource IDs | Reuse |
| App launches / unique users / last-launched | Audit Components / Audit Logs | Read-only query | Resource IDs + user object ID | Reuse |
| Resource-level inactivity signals | Audit Components | Read-only query | Resource IDs | Reuse (extended per-user) |
| Governance dashboards / Power BI model | CoE Power BI (on Core + Audit) | Reuse semantic model; extend with licensing measures | — | Reuse / Extend |
| Premium license assignment, SKU, source | Microsoft Graph (not CoE) | Supported API (ADR-004) | Entra user object ID | Supplement |

No CoE-managed table, flow, app, or environment variable is modified. No hard Dataverse lookup is created into a CoE-managed table.

---

## Integration Architecture

### Read-only adapter / query layer (reuse)

- A **project-owned adapter** reads Core and Audit tables through supported Dataverse queries, using **canonical identifiers** (Entra user object ID, environment ID, app ID, flow ID, group object ID) — **not hard relationships** into CoE-managed tables.
- This keeps the accelerator **loosely coupled** and **upgrade-safe**: CoE updates to Core/Audit do not break the project solution, and no managed dependency chain is created.
- The adapter isolates version-specific assumptions so that exact CoE schema names (confirmed during design) are contained in one place.

### Licensing acquisition (supplement)

- Premium assignment and SKU data are acquired from **Microsoft Graph** via a **validated, supported interface** (ADR-004), using least-privilege permissions, with throttling/pagination/retry handled, and freshness recorded.
- Direct vs. group-based assignment is distinguished where Graph provides it.

### Reclamation (new, via supported API)

- The license change is performed through a **supported API** (Graph/admin), administrator-initiated, after a dry run and safety checks. CoE compliance/clean-up flows are **never** repurposed.

### Isolation

- All project logic, tables, app, and dashboards live in a **separate project-owned solution and publisher**, never as unmanaged layers on CoE.

---

## End-to-End Data Flow

1. **Acquire (C1):** Graph → **License Assignment** (+ **SKU Reference**); assignment source captured. *Supplement.*
2. **Correlate (C2):** License Assignment ↔ CoE Core (maker/owner/app/flow/environment) + Audit (usage) via canonical IDs → correlation status; unmatched/ambiguous surfaced. *Reuse → Extend.*
3. **Analyze (C3):** per-user inactivity from Audit usage within the usage-history window → **Optimization Candidate** (+ evidence coverage, estimated value). *Reuse → Extend.* (Missing usage ≠ zero usage.)
4. **Assess safety (C4):** CoE relationships → **Dependency Finding**; apply **Protected Identity / Exception**. *Reuse → Extend.*
5. **Recommend (C5):** classify Safe/Review/Blocked with **Recommendation Evidence**; missing/stale evidence blocks Safe. *New.*
6. **Reclaim (C6) + Audit (C7):** administrator dry run → **Reclamation Action** → execute via supported API → immutable **Audit Log Entry**. *New (+ supplement).*
7. **Visualize (C8):** dashboards read project tables and **reuse/extend** the CoE Power BI model; deep-link to CoE. *Extend / Reuse.*

---

## Solution and ALM Architecture

- **Project-owned solution + publisher**, deployed as **managed** to downstream environments; CoE Core/Audit are **external prerequisites**, not solution dependencies (coupling is via canonical IDs, avoiding a managed dependency chain).
- **Environment variables** and **connection references** hold environment-specific configuration (Graph connection, scopes, thresholds), never hard-coded IDs or secrets.
- **Loose coupling** so CoE upgrades do not force redeployment; the adapter absorbs schema-name variance.
- **Documented prerequisites:** Core Components and Audit Components deployed and healthy; validated Graph interface; least-privilege permissions; protected-identity source configured.

---

## Security Architecture (Summary)

Full detail is in `docs/data-model.md`. Key points:

- **Least privilege** for Graph acquisition and for the reclamation operation; read-only analytics separated from reclamation execution.
- **Four Dataverse security roles** (Executive, Analyst, Administrator, Auditor); only Administrator can create/execute a Reclamation Action.
- **Field-level security** on PII and financial fields; **immutable** Audit Log Entry (no Update/Delete for any role).
- No CoE governance control (DLP, tenant, security) is weakened; no CoE-managed component is modified.

---

## Reuse / Extend / Build-New Summary

| Layer | Posture |
| --- | --- |
| CoE Core Components (inventory/ownership/environment/maker) | **Reuse** (read-only) |
| CoE Audit Components / Audit Logs (usage/last-launched/inactivity) | **Reuse** (read-only) |
| CoE Power BI governance dashboards | **Reuse / Extend** |
| Microsoft Graph licensing | **Supplement** |
| Correlation, analytics, dependency, visibility logic/visuals | **Extend** |
| Acquisition tables, recommendation, reclamation, audit, protected identities | **New** |

The source-of-record and analytics substrate is **reused** from CoE Core + Audit; net-new build is confined to the licensing layer CoE does not provide.

---

## Assumptions and Dependencies

- **Core Components and Audit Components are deployed, healthy, and reasonably fresh** (freshness recorded per recommendation).
- **Microsoft Graph licensing interface is validated** (support, permissions, licensing, throttling, availability — ADR-004).
- **Protected-identity source** (service/break-glass/shared/critical) is available or configured (Q-SAFETY-1).
- **Usage-history window** length is confirmed and bounds inactivity confidence (Q-COE-2).
- Open decisions affecting architecture are tracked in `docs/open-questions.md`.

---

## Risks and Mitigations

| Risk | Consequence | Mitigation |
| --- | --- | --- |
| CoE version/schema drift (Core/Audit) | Adapter queries break | Isolate schema assumptions in the adapter; confirm names during design; canonical IDs over implementation payloads |
| Archived kit (no further updates) | Long-term divergence | Do not hard-depend on kit internals; design for eventual admin-center-native sources |
| Usage-history window too short | Weak inactivity confidence | Disclose window/freshness; block Safe on insufficient evidence; never treat missing as zero |
| Unvalidated/ unsupported Graph operation | Acquisition or reclamation fails | Validate per ADR-004 before build; handle throttling/partial failure |
| Over-broad permissions | Security exposure | Least privilege; separate read from reclamation; audit privileged actions |
| Accidental CoE coupling/modification | Upgrade breakage / managed-layer risk | No hard lookups into CoE tables; separate solution; no unmanaged layers on CoE |

---

## Constraints Honored

- Design documentation only — no Power Platform assets, solution, schema, or flows were generated.
- Architecture targets a **Core Components + CenterOfExcellenceAuditComponents** deployment and depends on those two CoE solutions only (plus Graph and the project solution).
- CoE remains the system of record; nothing duplicates CoE inventory/usage, and no CoE-managed component is modified.
- No approval workflow and no automatic/unattended license removal appear anywhere in the architecture.
