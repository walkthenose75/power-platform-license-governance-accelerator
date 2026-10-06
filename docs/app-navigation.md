# Application Navigation and UX Design

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Scope

This document is the **UX and navigation design** for the Power Platform License Governance Accelerator, expressed as a **model-driven app (MDA)**. It is **design documentation only** — it creates no Power Platform assets, no Dataverse schemas, no forms/views metadata, and no flows. Tables are described **conceptually** for navigation purposes; field-level schema is intentionally out of scope.

The app is a **project-owned, licensing-focused experience** that complements the customer's existing CoE Toolkit. It implements the Capability Catalog (C1–C8) from `docs/product-definition.md` and the journeys in `docs/user-journeys.md`. CoE remains authoritative for app/flow/maker/owner/environment inventory, usage telemetry, and governance dashboards; this app **surfaces** that context and adds the licensing layer on top.

### Design assumption and open decision

Per ADR-003, a separate model-driven app is justified only for licensing-specific gaps that CoE cannot meet. This design treats the app as that justified gap-filler. **Where the experience ultimately lives — a standalone app vs. an experience surfaced adjacent to CoE — remains open decision Q-UX-1 in `docs/open-questions.md`.** The navigation below is valid either way; only its entry point changes.

### Classification legend (per navigation item)

- **Reuse / Surface** — present existing CoE data or experience as-is (read-only quick views, deep links); no new UX ownership.
- **Extend** — project-owned UX built on reused CoE data, with no change to managed CoE assets.
- **New** — a licensing experience with no CoE equivalent.

---

## Design Principles (Model-Driven App Best Practices)

1. **Role-tailored sitemap.** Each persona sees only the areas relevant to them, driven by security roles — not a single overloaded navigation.
2. **Interactive dashboards for triage.** Operational work starts from an interactive/streamed dashboard with filters, not raw grids.
3. **Progressive disclosure on forms.** Main forms use tabs/sections so an administrator sees classification and evidence first, with detail on demand.
4. **Surface CoE read-only with quick-view forms and deep links.** Never rebuild CoE browsing; show the minimum CoE context needed and link out to CoE for depth (anti-duplication; risks D-1/D-3 in `docs/coe-reuse-analysis.md`).
5. **Modern command bar with confirmation for high-impact actions.** Reclamation requires explicit confirmation and a prior dry run.
6. **Metadata-driven first; custom pages only when justified.** Prefer forms, views, and dashboards; introduce a custom page only where a guided multi-step experience is genuinely required.
7. **Safety and explainability are UX-level guarantees.** The UI makes evidence, freshness, assignment source, dependencies, and exceptions visible, and it blocks unsafe actions in the command layer, not just in back-end logic.
8. **Accessibility and consistency.** Standard MDA controls, keyboard navigation, and consistent patterns so CoE administrators feel immediate familiarity.

---

## Avoiding CoE Duplication (What This App Deliberately Does Not Build)

| Experience already owned by CoE | This app's approach |
| --- | --- |
| App / flow / maker / environment **inventory browsing** | **Surface** read-only context on licensing records; **deep-link** to the CoE app for full browsing. Do not add inventory-browse areas. |
| Maker onboarding / **nurture** experiences | Out of scope; link to CoE. |
| General **governance dashboards** (compliance, adoption) | **Extend** CoE reporting with licensing visuals; do not re-create CoE's dashboards. |
| Environment administration | Out of scope; CoE/PPAC owns it. |
| Audit of CoE objects | Out of scope; this app only audits **its own reclamation actions**. |

Every area below is checked against this table so the app stays within the licensing gap.

---

## Personas and Role-Specific Experiences (Overview)

Personas are defined in `docs/personas.md`. They map to four security roles, each with a tailored landing experience.

| Persona | Security role | Landing experience | Can execute reclamation? |
| --- | --- | --- | --- |
| P6 Executive Sponsor, P3 IT Finance/FinOps | **License Governance — Executive (read-only)** | Executive Summary dashboard | No |
| P1 Platform/CoE Admin, P4 Environment/BU Admin | **License Governance — Analyst (operational)** | Optimization Overview dashboard | No (triage, dry run, review only) |
| P1 Platform/CoE Admin (authorized) | **License Governance — Administrator** | Optimization Overview dashboard | Yes (guarded) |
| P5 Security & Compliance | **License Governance — Auditor (oversight)** | Audit Log view | No (read-only across reclamation/config) |

P7 Makers/Owners and P8 Service Desk/Ops are **affected stakeholders**, not app users; they are represented in the data as protected identities and dependency owners.

---

## 1. Sitemap

Navigation hierarchy is **App → Area → Group → Subarea**. Subarea visibility is governed by security role (role-driven sitemap).

```
License Governance (App)
├── Overview (Area)                         [Executive, Operational]
│   └── Insights (Group)
│       ├── Executive Summary               → dashboard
│       └── Optimization Overview           → interactive dashboard
├── Optimization (Area)                     [Operational, Administrator]
│   ├── Work (Group)
│   │   ├── Optimization Candidates         → table views
│   │   └── License Assignments             → table views
│   └── Insight (Group)
│       ├── Dependencies                    → view (surfaced CoE context)
│       └── Data Quality                    → unmatched / correlation view
├── Reclamation (Area)                      [Administrator, Auditor]
│   ├── Actions (Group)
│   │   ├── Reclamation Actions             → table views
│   │   └── Bulk Reclamation                → custom page (fast-follow, justified)
│   └── Accountability (Group)
│       └── Audit Log                       → read-only views
└── Configuration (Area)                    [Administrator, Security/Auditor]
    ├── Safety (Group)
    │   └── Protected Identities & Exceptions → table views
    ├── Reference (Group)
    │   ├── SKU Reference                    → table views
    │   └── Data & Interface Status          → status dashboard
    └── CoE Context (Group)
        ├── Open CoE Admin App               → URL deep link   [Reuse/Surface]
        └── CoE Governance Dashboards        → URL deep link   [Reuse/Surface]
```

### Sitemap detail — business purpose, persona, classification

| Node | Business purpose | Persona | Classification |
| --- | --- | --- | --- |
| **Overview** (Area) | Single entry for at-a-glance licensing value | P6, P3, P1 | Extend |
| Executive Summary | Premium utilization and estimated vs. realized savings for leadership | P6, P3 | Extend (extends CoE reporting with licensing visuals) |
| Optimization Overview | Operational triage of candidates by Safe/Review/Blocked, freshness, environment | P1, P4 | New (licensing triage over reused CoE usage) |
| **Optimization** (Area) | The daily working area for analysts/admins | P1, P4 | New / Extend |
| Optimization Candidates | Review and work explainable reclamation candidates (C3/C5) | P1, P4 | New |
| License Assignments | Inspect premium entitlement and assignment source (C1) | P1, P4 | New (Graph-supplemented) |
| Dependencies | See app/flow/ownership dependencies that affect safety (C4) | P1, P4 | Reuse/Surface + Extend (CoE relationships, read-only) |
| Data Quality | Triage unmatched/ambiguous correlations (C2) | P1 | New (reflects extended correlation) |
| **Reclamation** (Area) | Execute and account for safe reclamation (C6/C7) | P1 (admin), P5 | New |
| Reclamation Actions | Dry-run and administrator-initiated single reclamations | P1 (admin) | New |
| Bulk Reclamation | Guided bounded-bulk review/confirm (post-MVP) | P1 (admin) | New (custom page, justified) |
| Audit Log | Immutable record of reclamation actions and outcomes | P5, P1 | New |
| **Configuration** (Area) | Safety lists, reference data, operational status | P1 (admin), P5 | New / Reuse |
| Protected Identities & Exceptions | Maintain never-reclaim lists (service/break-glass/shared/critical) | P1 (admin), P5 | New |
| SKU Reference | Reference premium SKUs used for interpretation | P1 (admin) | New (reference) |
| Data & Interface Status | Show data freshness, sync health, Graph interface status | P1 (admin), P5 | New |
| Open CoE Admin App | Jump to CoE for full inventory/governance | P1, P4 | Reuse/Surface (deep link) |
| CoE Governance Dashboards | Jump to CoE Power BI governance reporting | P1, P3, P6 | Reuse/Surface (deep link) |

---

## 2. Navigation Hierarchy

- **App shell:** a single model-driven app, "License Governance."
- **Areas** segment the experience by job-to-be-done and persona: Overview (see), Optimization (analyze), Reclamation (act), Configuration (govern).
- **Groups** cluster related subareas within an area for scannability.
- **Subareas** point to dashboards, table views, a justified custom page, or external URLs (CoE deep links).
- **Role-driven visibility:** areas/subareas appear only for roles that need them (see §12). Executives never see Reclamation or Configuration; Auditors see read-only Reclamation/Configuration; Administrators see everything.

---

## 3. Areas

| Area | Job-to-be-done | Primary personas | Notes |
| --- | --- | --- | --- |
| **Overview** | "Show me the value and where to act." | P6, P3 (exec); P1, P4 (ops) | Dashboards only; no record editing. Executive landing. |
| **Optimization** | "Analyze candidates and understand why." | P1, P4 | Core analyst workspace; explainability-first. |
| **Reclamation** | "Act safely and prove it." | P1 (admin), P5 | Guarded actions + immutable audit. |
| **Configuration** | "Keep it safe and current." | P1 (admin), P5 | Protected lists, reference data, status. |

---

## 4. Tables Exposed to Users (Conceptual — No Schema)

Project-owned tables hold only the licensing data CoE does not provide. CoE tables are **referenced read-only** and are **not** exposed for browsing or editing in this app. No fields, keys, or relationships are defined here (out of scope).

| Conceptual table | Purpose in the UX | Primary persona | Classification | CoE relationship |
| --- | --- | --- | --- | --- |
| **License Assignment** | One record per user's premium entitlement; shows SKU and assignment source (direct/group) | P1, P4 | New (Supplement via Graph) | Correlated to CoE user/maker (read-only) |
| **Optimization Candidate** | A user flagged for potential reclamation, with Safe/Review/Blocked classification and estimated value | P1, P4 | New | Correlated to CoE usage/ownership |
| **Recommendation Evidence** | Child of candidate; each evidence item/rule (freshness, assignment source, exception, uncertainty) for explainability | P1, P5 | New | Derived from reused CoE + licensing |
| **Dependency Finding** | Child of candidate; an app/flow/ownership dependency affecting safety | P1, P4 | New (surfaces CoE) | References CoE app/flow/owner (read-only) |
| **Reclamation Action** | An administrator-initiated reclamation with dry-run result, status, outcome | P1 (admin) | New | Target user correlated to CoE |
| **Audit Log Entry** | Immutable record (actor, target, evidence, action, timestamp, outcome) | P5, P1 | New | — |
| **Protected Identity / Exception** | Never-reclaim list (service/break-glass/shared/critical, with reason/source) | P1 (admin), P5 | New | May reference a security group/CoE criticality |
| **SKU Reference** | Reference data for premium SKUs | P1 (admin) | New (reference) | — |

> Data model and schema (columns, types, relationships, validation) are defined outside this document and are intentionally not generated here, consistent with the charter and task constraints.

---

## 5. User Actions (Tasks by Persona)

What users can **do** in the app; the ribbon placement of each is in §11.

| Action | Description | Persona | Guardrail |
| --- | --- | --- | --- |
| View utilization & value | Read dashboards of premium utilization and savings | P6, P3, P1 | Freshness and estimated-vs-realized disclosed |
| Triage candidates | Filter/sort Safe/Review/Blocked, by environment/SKU/freshness | P1, P4 | — |
| Inspect explanation | Open a candidate and read evidence, dependencies, assignment source, exceptions, rule | P1, P4, P5 | Missing/stale evidence prevents Safe |
| Jump to CoE context | Open related CoE app/flow/user record or CoE dashboards | P1, P4 | Read-only surface; no duplication |
| Run dry run | Preview exactly what a reclamation would change and why | P1 (analyst/admin) | Always available before execution |
| Mark as reviewed | Record analyst review on a Review candidate | P1, P4 | Not an approval; single-actor status |
| Initiate reclamation | Execute a single eligible direct-assigned reclamation | P1 (admin) | Blocked for protected/group/stale; confirmation required |
| Manage protected identities/exceptions | Add/maintain never-reclaim entries | P1 (admin), P5 | Auditable |
| Review audit trail | Read immutable reclamation history | P5, P1 | Read-only |
| Check data/interface status | See freshness, sync health, Graph status | P1 (admin), P5 | — |
| Bulk reclamation (post-MVP) | Guided bounded-bulk review/confirm | P1 (admin) | Bounded; dry run; confirmation |

---

## 6. Forms

Form designs are conceptual (tabs/sections and surfaced CoE context), not metadata.

### Optimization Candidate — main form

- **Header:** classification (Safe/Review/Blocked), estimated value, overall evidence freshness, assignment source.
- **Tab: Summary** — why this user is a candidate (plain-language rule), status, environment.
- **Tab: Evidence** — subgrid of **Recommendation Evidence**; uncertainty and freshness per item.
- **Tab: Dependencies** — subgrid of **Dependency Finding**; **quick-view forms** surfacing the related CoE app/flow/owner read-only.
- **Tab: Licensing** — **quick-view** of the related **License Assignment** (SKU, direct/group source).
- **Tab: Reclamation** — related **Reclamation Actions** and their outcomes.
- **Business-rule UX:** when evidence is missing/stale or the assignment is group-based/protected, the classification shows **Blocked/Review** and the Initiate Reclamation command is disabled with a reason.

### License Assignment — main form

- **Summary:** user (quick-view to CoE user), SKU (quick-view to SKU Reference), assignment source (direct/group), acquisition freshness.
- **Related:** the Optimization Candidate (if any).

### Reclamation Action — main form

- **Summary:** target user, source candidate, dry-run result, status, outcome, timestamps, actor.
- **Tab: Dry run** — what would change and why (pre-execution).
- **Tab: Audit** — subgrid of **Audit Log Entry**.
- **Guardrail UX:** Execute is unavailable until a successful dry run exists and safety checks pass.

### Protected Identity / Exception — main form

- **Summary:** identity, protection reason/type, source (group/criticality/manual), effective dates.
- **Related:** affected candidates (shown as Blocked).

### Quick-create forms

- **Protected Identity / Exception** quick-create for fast safety additions by administrators.

### Quick-view forms (surfacing CoE — Reuse)

- CoE **User/Maker**, CoE **App**, CoE **Flow** read-only quick views embedded on candidate/dependency forms, with a command to open the full CoE record.

---

## 7. Views

Representative public/system views (interactive dashboards noted separately).

| Table | Key views |
| --- | --- |
| Optimization Candidate | Safe candidates; Review; Blocked; My environment; Stale evidence; By SKU; High estimated value |
| License Assignment | All premium; Direct-assigned; Group-assigned; Unmatched to CoE |
| Reclamation Action | Pending dry run; Dry-run complete; Executed; Failed/partial |
| Audit Log Entry | Recent actions; By actor; By outcome (read-only) |
| Protected Identity / Exception | Active exceptions; By type/source |
| SKU Reference | Premium SKUs |
| Dependency Finding | Critical dependencies; By app; By flow |

**Interactive dashboards**

- **Optimization Overview** (operational): streamed candidate list with filters (classification, freshness, environment, SKU) for triage (P1, P4).
- **Executive Summary**: premium utilization, estimated vs. realized savings, trend — **extending CoE reporting / Power BI** rather than re-creating CoE dashboards (P6, P3).
- **Data & Interface Status**: freshness, sync health, Graph interface status (P1 admin, P5).

---

## 8. Subgrids

| Host form | Subgrid | Purpose |
| --- | --- | --- |
| Optimization Candidate | Recommendation Evidence | Explainability detail |
| Optimization Candidate | Dependency Finding | Safety dependencies (with CoE quick-views) |
| Optimization Candidate | Reclamation Actions | History/outcomes for the candidate |
| Reclamation Action | Audit Log Entry | Immutable action trail |
| Protected Identity / Exception | Affected candidates | Shows what the exception blocks |
| License Assignment | Related candidate | Link entitlement to optimization |

---

## 9. Custom Pages (Only When Justified)

- **Bulk Reclamation (post-MVP, fast-follow).** Justified because bounded-bulk is a **guided multi-step, safety-critical** task (select a bounded set → review combined dry-run and exclusions → confirm) that standard grids and the command bar do not express well. It enforces the bound, shows excluded (protected/group/stale) items, and requires explicit confirmation. **New.**
- **Everything else uses standard forms, views, and dashboards.** No custom pages are introduced for Overview, Optimization, Audit, or Configuration — this keeps the app metadata-driven, maintainable, and accessible (best practice).

---

## 10. Business Process Flows

**Decision: no business process flow in the MVP.**

- A multi-stage BPF risks resembling a **business approval workflow**, which the charter prohibits (ADR-002). Reclamation safety is instead enforced through **form logic, status, command gating, dry run, and audit** — not a staged sign-off.
- **Optional, post-MVP:** a **single-actor safety-guidance** process (Triage → Dependency Review → Dry Run → Execute → Audited) could be offered *only* as a checklist for one administrator, explicitly **not** a multi-party approval and **not** introducing hand-offs. Adopt only if it demonstrably helps administrators and remains within ADR-002. This remains a design option, not a committed element.

---

## 11. Command Bar Actions

Modern commands with role- and rule-based visibility; high-impact actions require confirmation.

| Location | Command | Behavior & guardrail | Role |
| --- | --- | --- | --- |
| Optimization Candidate (form/view) | **Run Dry Run** | Produces a preview of the reclamation and its evidence | Analyst, Administrator |
| Optimization Candidate | **View Explanation** | Opens the evidence/dependency explanation | Analyst, Administrator, Auditor |
| Optimization Candidate | **Open CoE Record** | Deep-links to the related CoE app/flow/user (surface) | Analyst, Administrator |
| Optimization Candidate | **Mark as Reviewed** | Single-actor status; not an approval | Analyst, Administrator |
| Optimization Candidate | **Add Exception** | Creates a Protected Identity/Exception | Administrator |
| Optimization Candidate | **Initiate Reclamation** | **Disabled** when Blocked, stale, group-assigned, or protected; requires a prior dry run; shows a **confirmation dialog** | Administrator only |
| Reclamation Action | **Execute** | Available only after successful dry run and safety pass; **confirmation dialog**; writes audit | Administrator only |
| Reclamation Action | **Cancel** | Cancels a pending action; audited | Administrator |
| Reclamation Action / Audit | **Export Audit** | Exports the audit record for compliance | Administrator, Auditor |
| Candidate view | **Refresh Licensing Data** | Invokes the supported acquisition/refresh (conceptual); surfaces freshness | Administrator |
| Candidate view | **Select for Bulk** (post-MVP) | Adds to a bounded-bulk set (opens Bulk Reclamation page) | Administrator |

**Command guardrail summary:** execution commands are gated by **security role AND business rule** (protected/group/stale/Blocked disable execution), and every execution requires confirmation and produces an audit entry. There is no command anywhere that performs automatic or unattended removal.

---

## 12. Security-Role-Specific Experiences

Four roles tailor the sitemap, commands, and record access. Field-level security and command visibility reinforce the sitemap-level gating.

| Role | Visible areas | Record access | Commands | Landing |
| --- | --- | --- | --- | --- |
| **Executive (read-only)** | Overview only (Executive Summary) | Read dashboards | None | Executive Summary |
| **Analyst (operational)** | Overview, Optimization | Read candidates/assignments/dependencies; write "reviewed" status | Dry Run, View Explanation, Open CoE Record, Mark as Reviewed | Optimization Overview |
| **Administrator** | All areas | Read all; execute reclamation; manage exceptions/reference | All, incl. Initiate/Execute Reclamation, Add Exception, Refresh | Optimization Overview |
| **Auditor (oversight)** | Overview, Reclamation (Audit), Configuration (read) | Read-only across reclamation, audit, protected identities | View Explanation, Export Audit | Audit Log |

**Role-driven UX notes**

- Executives get a deliberately minimal, read-only app so the value story is clear and risk-free.
- Analysts can analyze and prepare (including dry run) but **cannot execute** — separating analysis from action (least privilege; NFR-4).
- Administrators hold the only execute privileges, always behind dry run + confirmation + audit.
- Auditors (P5) get oversight without the ability to act, satisfying the security/compliance persona.

---

## Reuse / Extend / New Roll-Up

| UX element | Classification |
| --- | --- |
| CoE inventory/usage context (quick views, deep links) | **Reuse / Surface** |
| Executive Summary and licensing visuals on CoE reporting | **Extend** |
| Dependency surfacing from CoE relationships | **Reuse/Surface + Extend** |
| Optimization Candidates, Recommendation Evidence, Data Quality | **New** |
| License Assignments (Graph-supplemented) | **New** |
| Reclamation Actions, Audit Log, Bulk Reclamation page | **New** |
| Protected Identities/Exceptions, SKU Reference, Status | **New** |

The majority of **data** is reused/extended from CoE; the **net-new UX** is confined to the licensing intelligence, recommendation, reclamation, audit, and safety surfaces CoE does not provide.

---

## Open UX Decisions (Cross-Reference)

- **Q-UX-1** — standalone app vs. experience surfaced adjacent to CoE (entry point and governance).
- **Q-UX-2** — operational vs. executive visibility split (MVP-minimal operational view; executive dashboard fast-follow).
- **Q-COE-3** — whether critical-app/owner criticality is reused from CoE or designated in Protected Identities/Exceptions.
- **Q-SAFETY-1** — authoritative source for protected identities (drives the Configuration → Safety experience).

See `docs/open-questions.md` for full context, owners, and decision gates.

---

## Constraints Honored

- No Power Platform assets, Dataverse schemas, forms/views metadata, or flows were generated — this is design documentation only.
- The app extends and surfaces CoE and avoids duplicating CoE's core experiences.
- No business approval workflow and no automatic/unattended removal are designed anywhere in the navigation or command model.
