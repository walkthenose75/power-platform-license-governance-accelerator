# Data Model — Minimum Dataverse Schema

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Scope

This document is the **logical data model design** for the Power Platform License Governance Accelerator. It is **design documentation only** — it defines the *minimum* project-owned Dataverse tables conceptually and does **not** generate Power Platform assets, a solution, schema files, logical names, publisher prefixes, forms, or flows. Physical schema is assigned at implementation time.

CoE remains the **system of record** for apps, flows, makers, environments, usage, and ownership. This model stores **only** the licensing, recommendation, and remediation data CoE does not provide (Capability Catalog C1–C8 in `docs/product-definition.md`), and references CoE read-only. Table names align with the conceptual tables in `docs/app-navigation.md` §4, and every table is **traceable to the capability it serves** per the Capability → Artifact Traceability in `docs/capability-map.md` (see Capability Traceability below). This model was aligned with the completed capability map so the reuse-versus-build proof stays consistent across documents. **Deployment baseline:** the CoE **Core Components and Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)** are deployed, so inventory/ownership/environment (Core) and usage/last-launched telemetry (Audit) are available to correlate against; the accelerator reads them read-only via canonical IDs and depends on no other CoE solution.

### Design principles

1. **Minimum footprint.** Eight project-owned tables, each justified against CoE. No table is created if a field, status, or CoE reference can serve instead.
2. **No CoE duplication.** CoE inventory/usage/ownership is never copied (charter Principle 1; duplication risks D-1/D-5 in `docs/coe-reuse-analysis.md`).
3. **Loose coupling via canonical identifiers.** Correlation uses stored canonical IDs (Entra user object ID, environment ID, app ID, flow ID, group object ID) — **not hard Dataverse lookups into CoE-managed tables**. This avoids modifying or depending on managed CoE components and survives CoE upgrades. The project-owned adapter/query layer joins to CoE at read time.
4. **Isolation.** All tables live in a **separate project-owned solution and publisher**, never as unmanaged layers on CoE.
5. **Safety and explainability are modeled.** Evidence, dependencies, assignment source, protected identities, dry-run, and immutable audit are first-class. No approval/sign-off fields exist (ADR-002); nothing supports automatic removal.
6. **Privacy by minimization.** Store canonical IDs; snapshot human-readable names only where needed for usability, protected by field-level security.

### Data category summary (as requested)

| # | Table | Data category | Classification |
| --- | --- | --- | --- |
| 1 | SKU Reference | Licensing (reference) | Build New |
| 2 | License Assignment | Licensing | Build New (Graph-supplemented) |
| 3 | Optimization Candidate | Recommendation | Build New |
| 4 | Recommendation Evidence | Recommendation | Build New |
| 5 | Dependency Finding | Recommendation (safety input) | Build New (references CoE) |
| 6 | Reclamation Action | Remediation | Build New |
| 7 | Audit Log Entry | Remediation (accountability) | Build New |
| 8 | Protected Identity / Exception | Remediation (safety control) | Build New |

### Capability traceability

Every table maps to the capability it serves (C0–C8), consistent with the Capability → Artifact Traceability in `docs/capability-map.md`. The two documents are mutually consistent: the capability map lists tables per capability; this lists capabilities per table.

| Table | Serves capability | Data category | Reuse/build posture |
| --- | --- | --- | --- |
| SKU Reference | C1 | Licensing (reference) | New |
| License Assignment | C1 (enables C2 correlation) | Licensing | New (Graph supplement) over reused CoE identity |
| Optimization Candidate | C3, C5 | Recommendation | New over reused CoE usage |
| Recommendation Evidence | C5 | Recommendation | New |
| Dependency Finding | C4 | Recommendation (safety input) | New; references CoE relationships (reuse) |
| Reclamation Action | C6 | Remediation | New (+ supported API) |
| Audit Log Entry | C7 | Remediation (accountability) | New (immutable) |
| Protected Identity / Exception | C4, C6 (safety control) | Remediation (safety) | New |

C0 (CoE discovery) and C8 (visibility) require **no project-owned tables**: C0 is reuse/inspection only, and C8 reads these tables plus the reused CoE Power BI model. This is why the eight tables map to C1–C7 plus the C4/C6 safety control, and none is dedicated to C0 or C8 — matching the capability map, where C0's table column is empty and C8 "reads project tables + CoE model."

---

## Tables

Each table states the three required justifications — **(1) why CoE cannot provide it, (2) why a new table is required, (3) which data category** — followed by its minimum fields. Types are conceptual (Text, Choice, Lookup, DateTime, Boolean, Whole Number, Currency). PII and financial fields are marked for field-level security (FLS).

### 1. SKU Reference

- **Why CoE cannot provide it:** CoE inventories resources, not commercial SKUs; it has no premium-SKU reference or value basis.
- **Why a new table is required:** a small, controlled reference set is needed to interpret which assignments are premium and to provide the value basis for estimates.
- **Data category:** Licensing (reference).

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| SKU Name | Text | Display name (primary column) |
| SKU Identifier | Text | Canonical SKU / service-plan identifier — correlation key for acquisition |
| Is Premium | Boolean | Whether the SKU is in licensing-governance scope |
| Premium Capability | Choice | What premium capability it grants (coarse) |
| Unit Cost Basis | Currency | Optional basis for value estimates — **FLS** |
| Active | Boolean | Reference lifecycle |

### 2. License Assignment

- **Why CoE cannot provide it:** CoE holds no per-user premium **license entitlement**, SKU, or assignment-source (direct vs. group) data.
- **Why a new table is required:** this is the foundational licensing fact (C1) that enables correlation, analytics, and reclamation; it is supplemented from Microsoft Graph.
- **Data category:** Licensing.

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| User Object ID | Text | Entra object ID — **primary correlation key** to CoE user/maker and Graph |
| User UPN | Text | Snapshot for readability — **PII, FLS** |
| User Display Name | Text | Snapshot — **PII, FLS** |
| SKU | Lookup → SKU Reference | The premium SKU assigned |
| Assignment Type | Choice | Direct / Group / Service / Shared / Exception / Unknown |
| Assignment Source Ref | Text | Group object ID when group-assigned (canonical ref) |
| Correlation Status | Choice | Matched / Unmatched / Ambiguous — powers the Data Quality view (no separate table) |
| Observed As-of | DateTime | Freshness of this licensing fact |
| Data Source | Choice | Acquisition source (for example, Microsoft Graph) |
| Status | Choice | Active / Removed / Superseded |

### 3. Optimization Candidate

- **Why CoE cannot provide it:** CoE's inactivity is resource-centric; it has no per-user license-reclamation candidate or Safe/Review/Blocked classification.
- **Why a new table is required:** to hold the explainable per-assignment recommendation (C3/C5) and its lifecycle.
- **Data category:** Recommendation.

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| License Assignment | Lookup → License Assignment | The entitlement under evaluation |
| User Object ID | Text | Denormalized correlation key for query |
| Classification | Choice | Safe / Review / Blocked |
| Classification Rule | Text | Governing rule / plain-language reason |
| Inactivity Determination | Choice | For example: no premium activity in window / insufficient evidence |
| Evidence Coverage | Whole Number (%) | Data sufficiency; low coverage cannot yield Safe |
| Overall As-of | DateTime | Combined evidence freshness |
| Estimated Value | Currency | Disclosed as estimate — **FLS** |
| Status | Choice | New / Reviewed / Actioned / Dismissed |
| Reviewed By | Text | Analyst who reviewed (single-actor; not an approval) |
| Reviewed On | DateTime | Review timestamp |

### 4. Recommendation Evidence

- **Why CoE cannot provide it:** CoE has no recommendation or evidence structure.
- **Why a new table is required:** the charter requires every recommendation to be explainable with itemized, auditable evidence; a child table captures each evidence item.
- **Data category:** Recommendation.

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| Optimization Candidate | Lookup → Optimization Candidate | Parent (cascading) |
| Evidence Type | Choice | Usage / Assignment Source / Exception / Freshness / Other |
| Summary | Text | Plain-language evidence item |
| Source | Choice | CoE usage / Graph / Protected list / etc. |
| Collected As-of | DateTime | Freshness of this item |
| Confidence | Choice | High / Medium / Low (uncertainty) |
| Influence | Choice | Supports Safe / Supports Review / Supports Blocked |

### 5. Dependency Finding

- **Why CoE cannot provide it:** CoE holds the relationships but not the **licensing-safety interpretation** of them; it does not mark an app/flow/ownership dependency as blocking a license reclamation.
- **Why a new table is required:** to store the interpreted dependency finding (C4), **referencing** CoE resources by canonical ID rather than duplicating CoE inventory. It is kept separate from Recommendation Evidence because it has a materially different shape (resource references, criticality) and is queried independently for dependency dashboards.
- **Data category:** Recommendation (safety input).

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| Optimization Candidate | Lookup → Optimization Candidate | Parent (cascading) |
| Dependency Type | Choice | App Ownership / Flow Ownership / Connection / Other |
| Resource Type | Choice | App / Flow / Environment / Other |
| Resource ID | Text | Canonical platform ID (environment + app/flow) — **reference to CoE, not a lookup** |
| Resource Name | Text | Snapshot for readability |
| Criticality | Choice | Critical / Standard / Unknown (reuses CoE criticality if available — Q-COE-3) |
| Is Blocking | Boolean | Whether it forces Review/Blocked |
| Source As-of | DateTime | Freshness |

### 6. Reclamation Action

- **Why CoE cannot provide it:** CoE performs no per-user license reclamation with dry run and guardrails; its compliance/clean-up is resource-oriented and must not be repurposed.
- **Why a new table is required:** to record each administrator-initiated reclamation attempt, its dry-run, and its outcome (C6).
- **Data category:** Remediation.

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| Optimization Candidate | Lookup → Optimization Candidate | Source recommendation |
| License Assignment | Lookup → License Assignment | The specific assignment being reclaimed |
| Target User Object ID | Text | Canonical correlation key |
| Action Type | Choice | Dry Run / Execute |
| Status | Choice | Pending / Dry-Run Complete / Executed / Failed / Partial / Cancelled |
| Dry-Run Result | Text | What would change and why |
| Outcome | Text | Result / failure detail |
| Interface Operation | Text | Supported API operation used (traceability; ADR-004) |
| Initiated By | Text | Administrator actor |
| Initiated On | DateTime | Start |
| Completed On | DateTime | Completion |

### 7. Audit Log Entry

- **Why CoE cannot provide it:** CoE does not audit license-reclamation decisions or outcomes.
- **Why a new table is required:** an **immutable, dashboardable, long-retained** business audit of reclamation, including an evidence snapshot proving why a decision was made (C7).
- **Data category:** Remediation (accountability).

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| Reclamation Action | Lookup → Reclamation Action | The action audited |
| Actor | Text | Who acted |
| Target User Object ID | Text | Canonical correlation key |
| Action | Choice | Dry Run / Execute / Cancel / Fail |
| Evidence Snapshot | Multiline Text | What was shown at decision time (point-in-time proof) |
| Outcome | Choice | Success / Partial / Failed |
| Failure Detail | Text | Error context when applicable |
| Timestamp | DateTime | When it occurred |

> **Immutability:** no role is granted Update or Delete on this table (see Security). Records are append-only.

### 8. Protected Identity / Exception

- **Why CoE cannot provide it:** CoE has no license-reclamation protection list.
- **Why a new table is required:** a safety-critical **never-reclaim** list enforced before any action (C4/C6 guardrail), covering service, break-glass, shared, critical-owner, and manually excepted identities.
- **Data category:** Remediation (safety control).

| Field | Type | Purpose / Notes |
| --- | --- | --- |
| Identity Type | Choice | User / Group |
| Identity Object ID | Text | Entra object ID — correlation key |
| Identity Name | Text | Snapshot — **PII, FLS** |
| Protection Reason | Choice | Service / Break-Glass / Shared / Critical Owner / Group-Assigned / Manual / Other |
| Source | Choice | Security Group / CoE Criticality / Manual (Q-SAFETY-1) |
| Effective From | DateTime | Start of protection |
| Effective To | DateTime | Optional end |
| Active | Boolean | Current applicability |
| Notes | Text | Justification |

---

## Fields — Cross-Cutting Conventions

- **Canonical ID fields** (User Object ID, Resource ID, Identity Object ID, Assignment Source Ref) are the correlation mechanism; they are indexed for query performance and carry no hard relationship to CoE.
- **Snapshot name fields** (UPN, Display Name, Resource Name, Identity Name) are denormalized for readability only, kept current on refresh, and **field-secured** where they are PII.
- **Freshness fields** (As-of/Collected) appear on every evidence-bearing table so recommendations can disclose data age and block Safe on stale evidence.
- **No approval fields.** There is deliberately no "approver", "approval status", or multi-stage sign-off anywhere (ADR-002).

---

## Relationships

All relationships are **internal** to the project-owned solution. No relationship points into a CoE-managed table.

```
SKU Reference (1) ──< License Assignment (N)
License Assignment (1) ──< Optimization Candidate (N)
Optimization Candidate (1) ──< Recommendation Evidence (N)   [parental/cascade]
Optimization Candidate (1) ──< Dependency Finding (N)        [parental/cascade]
Optimization Candidate (1) ──< Reclamation Action (N)
License Assignment (1) ──< Reclamation Action (N)            [the assignment reclaimed]
Reclamation Action (1) ──< Audit Log Entry (N)              [append-only]
Protected Identity / Exception — standalone (matched by Identity Object ID at query/logic time)
```

- **Cascade behavior:** Recommendation Evidence and Dependency Finding are **parented** to Optimization Candidate (cascade delete), since they have no meaning without it. Reclamation Action and Audit Log Entry use **referential, restrict-delete** behavior so remediation history is never lost when a candidate is cleaned up.
- **Protected Identity/Exception is intentionally unrelated** by hard relationship: membership is dynamic and matched by Identity Object ID during classification/reclamation, keeping the model minimal and resilient.
- **CoE correlation** (user ↔ maker, resource ID ↔ app/flow, environment) is performed by the adapter/query layer using canonical IDs — not modeled as Dataverse relationships.

---

## Security

### Ownership model

All eight tables are **organization-owned** (tenant-wide governance data with no meaningful per-record owner). This keeps the security model simple and correct: privileges are evaluated at the organization level, and governance data is not accidentally scoped to a business unit or user. Accountability for actions is captured explicitly via **Initiated By / Actor** fields and created-by metadata, not via record ownership.

### Security roles and privilege matrix

Four roles align to the role-specific experiences in `docs/app-navigation.md`. C = Create, R = Read, W = Write/Update, D = Delete.

| Table | Executive | Analyst | Administrator | Auditor / Security |
| --- | --- | --- | --- | --- |
| SKU Reference | R (via BI) | R | C R W D | R |
| License Assignment | R (aggregate, via BI) | R | C R W D | R |
| Optimization Candidate | R (aggregate) | R, W* | C R W D | R |
| Recommendation Evidence | — | R | C R W D | R |
| Dependency Finding | — | R | C R W D | R |
| Reclamation Action | — | R | **C R W** | R |
| Audit Log Entry | — | R | **C R** (no W/D) | R |
| Protected Identity / Exception | — | R | C R W D | R |

- **W\*** (Analyst on Optimization Candidate): limited to review-status fields (Status, Reviewed By/On) via **field-level security** and business logic — analysts triage but do not reclassify arbitrarily.
- **Execution separation (least privilege, NFR-4):** only **Administrator** can Create a Reclamation Action; Analysts cannot execute. The actual license change is performed through a supported API with its own least-privilege permissions; the Dataverse record is the control and record of that action.
- **Immutable audit:** **no role** has Write or Delete on Audit Log Entry. Only the Administrator/system context Creates entries; they can never be altered or removed — tamper resistance by privilege design.
- **Executive** is primarily served by embedded Power BI with row-level/aggregate access, not raw table browsing; direct table privileges are minimal.

### Field-level security

Field-security profiles restrict sensitive columns regardless of table privileges:

- **PII** — User UPN, User Display Name, Identity Name: visible to Administrator and Auditor; hidden from Executive; visible to Analyst only where operationally necessary.
- **Financial** — Unit Cost Basis, Estimated Value: visible to Administrator, Auditor, and (aggregated) Executive via BI; restricted for Analyst if policy requires.

### Reinforcement at the app/command layer

The `docs/app-navigation.md` command model disables reclamation for protected, group-assigned, stale, or Blocked records. Table privileges and field security enforce the same boundaries at the data layer, so safety is not UI-only.

---

## Auditing

Two complementary layers:

1. **Business audit (table).** The **Audit Log Entry** table is the immutable, queryable, dashboardable record of reclamation decisions and outcomes, including a point-in-time **evidence snapshot** so a decision remains provable even if underlying evidence later changes. It drives the "Audit activity over time" visuals in `docs/dashboard-design.md` and satisfies the P5 Security/Compliance persona.
2. **Platform audit (native Dataverse auditing).** Enable table/column auditing as defense-in-depth on the most sensitive tables and fields:
   - **Reclamation Action** — all status transitions and outcomes.
   - **Protected Identity / Exception** — all changes to the never-reclaim list.
   - **License Assignment** — Status and Assignment Type changes.
   - **Optimization Candidate** — Classification and Status changes.

Native auditing provides full change history/forensics; the Audit Log Entry provides business-level, long-retained accountability. Audit content avoids secrets and minimizes PII (field-secured snapshots where needed).

---

## Retention

Retention balances usefulness, auditability, privacy minimization, and the usage-history window (Q-COE-2). Operational data is regenerable; audit data is not.

| Table | Retention stance | Mechanism |
| --- | --- | --- |
| SKU Reference | Maintained reference; no auto-purge; no PII | Manual lifecycle |
| License Assignment | Current + bounded rolling history (for example ~13 months) then purge/aggregate; minimize PII | Bulk delete / long-term retention |
| Optimization Candidate | Active + a post-resolution window (for example 12–24 months); regenerable | Bulk delete |
| Recommendation Evidence | Follows parent candidate (cascade) | Cascade / bulk delete |
| Dependency Finding | Follows parent candidate (cascade) | Cascade / bulk delete |
| Reclamation Action | Compliance-driven, extended retention (confirm policy) | Long-term retention |
| Audit Log Entry | **Immutable, long/compliance retention; not auto-purged without policy** | Policy-governed; long-term retention |
| Protected Identity / Exception | Active entries retained; deactivated kept for audit | Soft-deactivate, not delete |

- **PII minimization:** prefer canonical IDs; purge or hash snapshot names on the operational-data retention boundary where feasible.
- **Open dependencies:** final retention periods for Reclamation Action and Audit Log Entry require sign-off from Security/Compliance (related to Q-VALUE-1 and data-protection policy); the usage window (Q-COE-2) bounds how far back licensing history is meaningful. See `docs/open-questions.md`.

---

# Tables We Explicitly Will Not Create

CoE is the **system of record** for the following. This project **references them read-only via canonical identifiers** and **never duplicates** them (charter Principle 1; ADR-001; duplication risks D-1/D-5). Creating any of these would duplicate CoE inventory and is out of scope.

| CoE-owned fact (system of record) | What this project consumes | Correlation key used | Why we will not create it |
| --- | --- | --- | --- |
| **Apps** inventory | App identity/ownership context for dependencies | App ID (+ environment ID) | CoE already inventories apps |
| **Flows** inventory | Flow identity/ownership context for dependencies | Flow ID (+ environment ID) | CoE already inventories flows |
| **Makers / Users** inventory | Owner/maker correlation to licensed users | Entra user object ID | CoE already inventories makers; identity is in Entra |
| **Environments** inventory | Environment attribution for analytics | Environment ID | CoE already inventories environments |
| **Usage / activity** telemetry | Usage evidence for inactivity and analytics | Resource IDs + user object ID | CoE Audit already collects usage (missing ≠ zero) |
| **Ownership / relationships** | Dependency analysis inputs | Resource IDs + user object ID | CoE already maintains relationships |
| **Connectors / connection references** | Context where relevant to dependencies | Resource IDs | CoE already inventories connectors |
| **Power Pages / Copilot Studio** inventory | Out of current scope | — | CoE already inventories these |

> Artifact names and schemas on the CoE side are **not assumed**; the adapter binds to the customer's actual CoE schema at implementation (names vary by version). This project holds **no** inventory, usage, or ownership tables of its own.

---

## Minimality Rationale (What Was Deliberately Not Modeled)

- **No "Licensed User" table.** User identity lives in Entra and CoE; we store the **User Object ID** on licensing records instead.
- **No "Unmatched/Correlation" table.** Correlation quality is a **status field** on License Assignment, which powers the Data Quality view without a new table.
- **No separate cost/price table.** A **Unit Cost Basis** field on SKU Reference is sufficient for value estimates.
- **No approval/workflow tables.** Excluded by the charter (ADR-002); reclamation safety is modeled via dry run, protected identities, dependency findings, and immutable audit — not sign-off.
- **No inventory tables.** Reused from CoE (see above).
- **Evidence vs. Dependency kept as two tables** only because they have materially different shapes and independent query/dashboard needs; otherwise they would be merged.

Result: **eight** project-owned tables — three licensing, three recommendation, two remediation (plus the safety-control and reference tables within those categories) — the minimum required to deliver C1–C8 without duplicating CoE.

---

## Cross-References

- Capability map and reuse-vs-build proof: `docs/capability-map.md` (Capability → Artifact Traceability).
- Capability definitions and justifications: `docs/product-definition.md` (Capability Catalog), `docs/coe-reuse-analysis.md` (§A–§B).
- UX tables and experiences realized on this schema: `docs/app-navigation.md`.
- Analytics consuming this schema: `docs/dashboard-design.md`.
- Backlog epics and Reuse Strategy: `docs/backlog.md`.
- Open decisions affecting the model (retention periods, criticality source, protected-identity source, usage window): `docs/open-questions.md`.

---

## Constraints Honored

- Design documentation only — no Power Platform assets, solution, schema files, forms, or flows were generated.
- CoE remains the system of record; no CoE table is duplicated and no CoE-managed component is modified.
- Loose coupling via canonical IDs; all tables isolated in a project-owned solution.
- The model contains no approval-workflow structures and nothing enabling automatic or unattended license removal; estimates and realized outcomes are distinct, and evidence freshness is first-class.
