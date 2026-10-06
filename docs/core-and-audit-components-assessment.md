# CoE Core + Audit Components — Comprehensive Assessment and Validation

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Method

This is a **comprehensive assessment and validation checklist** for the customer's **actual CoE deployment** — **Core Components** and **Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)**. Its objective is to **eliminate unnecessary custom development** and to **replace every remaining architectural assumption with a confirmed CoE artifact**.

This is **assessment documentation only** — no Power Platform assets, Dataverse schemas, or flows are generated, and nothing in CoE is modified.

### Validate, do not invent (guardrail)

Per `docs/reference/coe-toolkit/README.md`, CoE table, column, flow, app, dashboard, and report names must **not be invented** and must be validated against the customer's actual environment or exported solution.

- Every CoE name below is an **expected reference label**, marked **(validate)** — a hypothesis to confirm, not a fact.
- Core/Audit publisher prefix is commonly `admin_` **(validate)**; display names are more stable than schema names.
- Record confirmed artifacts in `docs/reference/coe-toolkit/customer-deployment-inventory.md` (no secrets/IDs).

### How to validate (tools)

- Maker portal → Tables, Columns, Relationships, Views, Forms; CoE admin app; Power BI workspace.
- Dataverse Web API metadata (`EntityDefinitions`, `RelationshipDefinitions`) → authoritative logical names.
- Exported solution inspection (no modification); Advanced Find for data population and row counts.
- **Last successful inventory and audit sync + flow health** → confirm freshness before trusting any signal.

### Checkbox legend

- `[ ]` not yet validated · `[x]` validated and confirmed · `[!]` validated, deviates from expected (record details)

### Classification legend (per capability)

- **Reuse As-Is** — consume the CoE capability directly (read-only / deep-link); no project build.
- **Extend** — project-owned logic/visuals on top of reused CoE data; no change to managed CoE.
- **Supplement** — acquire via a supported external interface where CoE does not provide the data.
- **Build New** — no CoE equivalent; reserved for the licensing layer (acquisition, recommendation, reclamation, audit), not these CoE areas.

---

# Part A — Core Components (Areas 1–10)

### 1. Environment Inventory

- **CoE Component:** Core Components.
- **Purpose:** Inventory of all environments and attributes.
- **Dataverse Tables (validate):** Environment — `admin_environment`.
- **Important Columns (validate):** Environment ID/unique name; Display Name; Type; Region; Dataverse-enabled; Managed Environment status; Owner; Created On.
- **Relationships (validate):** 1:N Apps, Flows, Connectors, Connection References.
- **Existing Views (validate):** "Active Environments"; "Environments by Type".
- **Existing Dashboards (validate):** Environment overview pages (admin app / Power BI).
- **Existing Apps (validate):** CoE Admin Command Center → Environments area.
- **Existing Power Automate Flows (validate):** "Admin | Sync Template v4 (Environments)".
- **Relevance to this project:** Environment attribution and candidate scoping; environment ID is a correlation key.
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` table/logical name · `[ ]` columns populated · `[ ]` relationships · `[ ]` row count + freshness.

### 2. Power App Inventory

- **CoE Component:** Core Components.
- **Purpose:** Inventory of canvas and model-driven apps.
- **Dataverse Tables (validate):** Power Apps App — `admin_app`.
- **Important Columns (validate):** App ID; Display Name; Environment (lookup); Owner/Maker (lookup); App Type; Created/Modified; **Last Launched (Audit-populated, validate)**; shared-with; connectors used; business-criticality (if configured).
- **Relationships (validate):** N:1 Environment; N:1 Maker; N:N Connectors.
- **Existing Views (validate):** "All Apps"; "Apps by Environment"; "Orphaned Apps".
- **Existing Dashboards (validate):** App inventory/adoption pages.
- **Existing Apps (validate):** CoE Admin Command Center → Apps.
- **Existing Power Automate Flows (validate):** "Admin | Sync Template v4 (Apps)".
- **Relevance to this project:** Apps are dependency objects for reclamation safety (C4); app ID is a correlation key.
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` table/name · `[ ]` owner+environment lookups populated · `[ ]` app type · `[ ]` last-launched present (Audit) · `[ ]` freshness.

### 3. Cloud Flow Inventory

- **CoE Component:** Core Components.
- **Purpose:** Inventory of cloud flows (and desktop flows where present).
- **Dataverse Tables (validate):** Flow — `admin_flow`; Flow Action Details — `admin_flowactiondetail` (optional).
- **Important Columns (validate):** Flow ID; Display Name; Environment (lookup); Owner/Maker (lookup); State; Trigger type; Created/Modified; **last run (Audit-dependent, validate)**; connectors used.
- **Relationships (validate):** N:1 Environment; N:1 Maker; N:N Connectors.
- **Existing Views (validate):** "All Flows"; "Flows by Owner"; "Suspended Flows".
- **Existing Dashboards (validate):** Flow inventory pages.
- **Existing Apps (validate):** CoE Admin Command Center → Flows.
- **Existing Power Automate Flows (validate):** "Admin | Sync Template v4 (Flows)".
- **Relevance to this project:** Flows are dependency objects for reclamation safety (C4); flow ID is a correlation key.
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` table/name · `[ ]` owner+environment lookups · `[ ]` state/trigger · `[ ]` freshness.

### 4. Maker Inventory

- **CoE Component:** Core Components.
- **Purpose:** Inventory of makers who own/create resources.
- **Dataverse Tables (validate):** Maker — `admin_makeruser`; relates to `systemuser`/Entra.
- **Important Columns (validate):** Maker ID; Display Name; Email/UPN (**PII**); **Entra/Azure AD object ID (key correlation column — validate)**; counts; first/last created.
- **Relationships (validate):** 1:N Apps; 1:N Flows.
- **Existing Views (validate):** "All Makers"; "Makers by App Count".
- **Existing Dashboards (validate):** Maker analytics pages.
- **Existing Apps (validate):** CoE Admin Command Center → Makers.
- **Existing Power Automate Flows (validate):** "Admin | Sync Template v4 (Makers)".
- **Relevance to this project:** **Entra object ID is the primary correlation anchor** between CoE and licensing (C2/C4).
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` table/name · `[ ]` **Entra object ID populated (blocking)** · `[ ]` UPN (PII) · `[ ]` app/flow relationships.

### 5. Ownership Relationships

- **CoE Component:** Core Components.
- **Purpose:** Links resources to owners/makers and environments.
- **Dataverse Tables (validate):** Expressed as lookups on App/Flow to Maker/Environment; shared-with/permissions data (validate).
- **Important Columns (validate):** Owner/Maker lookup; co-owner/shared-with; environment ownership.
- **Relationships (validate):** App→Maker; Flow→Maker; App/Flow→Environment; co-ownership N:N (validate).
- **Existing Views (validate):** "Apps by Owner"; "Orphaned resources".
- **Existing Dashboards (validate):** Ownership/orphaned pages.
- **Existing Apps (validate):** CoE Admin Command Center (resource records).
- **Existing Power Automate Flows (validate):** Inventory sync flows (owner captured during sync).
- **Relevance to this project:** Core input to reclamation-safety dependency analysis (C4) — critical owners.
- **Reuse Strategy:** **Reuse As-Is** (interpreted in Dependency Finding, not duplicated).
- **Validation:** `[ ]` owner lookups populated · `[ ]` co-ownership available · `[ ]` orphaned view.

### 6. Connector Inventory

- **CoE Component:** Core Components.
- **Purpose:** Inventory of connectors (standard/premium/custom).
- **Dataverse Tables (validate):** Connector — `admin_connector`.
- **Important Columns (validate):** Connector ID; Name; Tier (Standard/Premium/Custom); Publisher; **Is Premium (validate)**.
- **Relationships (validate):** N:N Apps; N:N Flows; 1:N Connection References.
- **Existing Views (validate):** "Premium Connectors"; "Custom Connectors".
- **Existing Dashboards (validate):** Connector usage pages.
- **Existing Apps (validate):** CoE Admin Command Center → Connectors.
- **Existing Power Automate Flows (validate):** "Admin | Sync Template v4 (Connectors)".
- **Relevance to this project:** Premium-connector usage is corroborating evidence of premium-requiring activity (C3/C5/C4).
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` table/name · `[ ]` premium/tier column · `[ ]` app/flow relationships.

### 7. Connection Reference Inventory

- **CoE Component:** Core Components.
- **Purpose:** Inventory of connection references/identities.
- **Dataverse Tables (validate):** Connection Reference — `admin_connectionreference`; Connection Identity — `admin_connectionidentity`.
- **Important Columns (validate):** Connection ID; Connector (lookup); Owner; Environment (lookup); Created; status.
- **Relationships (validate):** N:1 Connector; N:1 Environment; N:1 Owner.
- **Existing Views (validate):** "Connection References by Environment".
- **Existing Dashboards (validate):** Connection inventory pages (validate).
- **Existing Apps (validate):** CoE Admin Command Center (connections).
- **Existing Power Automate Flows (validate):** Connection sync flow (validate).
- **Relevance to this project:** Dependency context for safety (C4); secondary priority.
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` tables/names · `[ ]` owner/connector lookups · `[ ]` linkage to apps/flows.

### 8. Existing CoE Administration Apps

- **CoE Component:** Core Components.
- **Purpose:** Model-driven apps surfacing CoE inventory and governance.
- **Dataverse Tables (validate):** Surfaces the Core tables above.
- **Important Columns (validate):** n/a (application components).
- **Relationships (validate):** App → Core tables.
- **Existing Views (validate):** Embedded per-table views.
- **Existing Dashboards (validate):** Embedded app dashboards.
- **Existing Apps (validate):** "CoE Admin Command Center"; "Power Platform Admin View" (validate which are installed).
- **Existing Power Automate Flows (validate):** n/a (consumes inventory flows).
- **Relevance to this project:** Host/extension surface (ADR-003) and **deep-link targets** for the License Governance app's CoE Context (`docs/app-navigation.md`).
- **Reuse Strategy:** **Reuse As-Is** (deep-link) / **Extend** (host licensing views, pending Q-UX-1).
- **Validation:** `[ ]` apps installed/used · `[ ]` deep-linkable URLs · `[ ]` extension feasibility.

### 9. Existing Dashboards

- **CoE Component:** Core Components (+ Audit for usage pages).
- **Purpose:** In-app operational dashboards.
- **Dataverse Tables (validate):** Read Core (and Audit) tables.
- **Important Columns (validate):** n/a.
- **Relationships (validate):** Dashboards → tables.
- **Existing Views (validate):** Underlying views.
- **Existing Dashboards (validate):** Environment/app/flow/maker/connector dashboards (validate).
- **Existing Apps (validate):** Within CoE admin app.
- **Existing Power Automate Flows (validate):** n/a.
- **Relevance to this project:** Extend for licensing/optimization visibility (C8) rather than rebuild.
- **Reuse Strategy:** **Extend.**
- **Validation:** `[ ]` dashboards present · `[ ]` Core-only vs. Audit-dependent pages · `[ ]` extensibility.

### 10. Existing Power BI Reports

- **CoE Component:** Core + Audit (Power BI governance dashboard).
- **Purpose:** Executive/operational analytics over the estate.
- **Dataverse Tables (validate):** Reads Core inventory + Audit usage via Dataverse/TDS or Data Export.
- **Important Columns (validate):** Measures for environments/apps/flows/makers/connectors/usage (validate).
- **Relationships (validate):** Semantic model over Core + Audit.
- **Existing Views (validate):** Report pages.
- **Existing Dashboards (validate):** "Power Platform Governance" / CoE Power BI (validate name).
- **Existing Apps (validate):** Power BI workspace/app (validate).
- **Existing Power Automate Flows (validate):** n/a (refresh schedule).
- **Relevance to this project:** **Reuse the semantic model; extend with licensing measures** (C8) — do not build a new model.
- **Reuse Strategy:** **Reuse As-Is** (model) / **Extend** (licensing measures).
- **Validation:** `[ ]` report deployed/used · `[ ]` data source (Dataverse vs Data Export) · `[ ]` model extensibility.

---

# Part B — Audit Component Assessment (Areas 11–19)

> Audit Components / Audit Logs ingest Office 365 audit events (app launches) and aggregate usage onto resource records. **The key licensing question is per-user usage granularity and retention** — assessed explicitly below.

### 11. App Launch Telemetry

- **CoE Component:** Audit Components / Audit Logs.
- **Purpose:** Capture app-launch events from the Office 365 audit log.
- **Dataverse Tables (validate):** Audit Log — `admin_auditlog` (raw launch events) (validate).
- **Important Columns (validate):** App (lookup/ID); **User (UPN / object ID) — validate per-user granularity**; launch timestamp; environment.
- **Relationships (validate):** Audit Log N:1 App; relates to user.
- **Existing Views (validate):** "Recent App Launches" (validate).
- **Existing Dashboards (validate):** Usage/adoption pages (Power BI).
- **Existing Apps (validate):** Surfaced in CoE app/Power BI.
- **Existing Power Automate Flows (validate):** "Admin | Audit Logs | ..." collection flows (validate).
- **Relevance to this project:** **Primary usage evidence** for "is the premium capability used" (C3). Per-user granularity is critical.
- **Reuse Strategy:** **Reuse As-Is** (if per-user granularity present); else **Supplement**.
- **Validation:** `[ ]` audit log table/name · `[ ]` **per-user launch granularity present (blocking)** · `[ ]` collection flow healthy · `[ ]` event volume/row count.

### 12. Unique User Metrics

- **CoE Component:** Audit Components.
- **Purpose:** Count of unique users per app.
- **Dataverse Tables (validate):** Aggregated onto Power Apps App (unique-users column) and/or an aggregation table (validate).
- **Important Columns (validate):** Unique user count; period; last computed (validate).
- **Relationships (validate):** App 1:N audit events; aggregate on App.
- **Existing Views (validate):** "Apps by Unique Users" (validate).
- **Existing Dashboards (validate):** Adoption pages.
- **Existing Apps (validate):** CoE app/Power BI.
- **Existing Power Automate Flows (validate):** Audit aggregation flows (validate).
- **Relevance to this project:** Corroborates whether an app a user owns is used at all; context for inactivity reasoning.
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` unique-users column/table · `[ ]` computation period · `[ ]` freshness.

### 13. App Adoption Metrics

- **CoE Component:** Audit Components.
- **Purpose:** Adoption trends (launches/users over time).
- **Dataverse Tables (validate):** Audit log aggregates; Power BI usage model (validate).
- **Important Columns (validate):** Launches over time; active users; trend period (validate).
- **Relationships (validate):** Derived from audit events.
- **Existing Views (validate):** Adoption views (validate).
- **Existing Dashboards (validate):** Power BI adoption pages.
- **Existing Apps (validate):** Power BI.
- **Existing Power Automate Flows (validate):** Audit aggregation flows.
- **Relevance to this project:** Trend context for optimization analytics (C3) and executive visibility (C8).
- **Reuse Strategy:** **Reuse As-Is** (consume) / **Extend** (license-centric trend).
- **Validation:** `[ ]` adoption data present · `[ ]` trend window · `[ ]` freshness.

### 14. Flow Usage Metrics

- **CoE Component:** Audit Components (+ Core flow data).
- **Purpose:** Flow run/usage signals.
- **Dataverse Tables (validate):** Flow run data on Flow / usage table (validate) — **flow usage in CoE is typically less complete than app launches**.
- **Important Columns (validate):** Last run; run counts (validate; may be limited).
- **Relationships (validate):** Flow 1:N runs (if captured).
- **Existing Views (validate):** "Flows by Last Run" (validate).
- **Existing Dashboards (validate):** Flow usage pages (validate).
- **Existing Apps (validate):** CoE app/Power BI.
- **Existing Power Automate Flows (validate):** Audit/analytics flows (validate).
- **Relevance to this project:** Usage evidence for flow-owning licensed users; **note potential coverage gap vs. apps**.
- **Reuse Strategy:** **Reuse As-Is** (if present) / **Supplement** (if incomplete; via supported analytics).
- **Validation:** `[ ]` flow usage captured? · `[ ]` completeness vs. apps · `[ ]` freshness.

### 15. Last Used / Last Launched Signals

- **CoE Component:** Audit Components.
- **Purpose:** Most-recent activity per app/flow.
- **Dataverse Tables (validate):** Last Launched on App; Last Run on Flow (validate).
- **Important Columns (validate):** Last Launched (datetime); Last Run (datetime).
- **Relationships (validate):** On App/Flow records.
- **Existing Views (validate):** "Inactive Apps (by last launched)"; "Stale Flows" (validate).
- **Existing Dashboards (validate):** Inactivity pages.
- **Existing Apps (validate):** CoE app/Power BI.
- **Existing Power Automate Flows (validate):** Audit update flows.
- **Relevance to this project:** Resource-level recency feeds **per-user** inactivity reasoning (C3) when combined with ownership.
- **Reuse Strategy:** **Reuse As-Is.**
- **Validation:** `[ ]` last-launched/last-run populated · `[ ]` coverage · `[ ]` freshness vs. window.

### 16. Inactivity Detection Capabilities

- **CoE Component:** Audit Components (+ Archive/Clean-up).
- **Purpose:** Flag inactive/orphaned resources by thresholds.
- **Dataverse Tables (validate):** Inactive/orphaned flags on App/Flow; clean-up tables (validate).
- **Important Columns (validate):** Inactive flag; threshold/last-activity (validate).
- **Relationships (validate):** On App/Flow.
- **Existing Views (validate):** "Inactive Apps"; "Orphaned Flows" (validate).
- **Existing Dashboards (validate):** Compliance/clean-up pages.
- **Existing Apps (validate):** CoE app (compliance).
- **Existing Power Automate Flows (validate):** "Admin | Archive and Clean up ..." / compliance flows (validate).
- **Relevance to this project:** CoE detects **resource** inactivity; the project needs **per-user license** inactivity — a gap to extend, not reuse directly.
- **Reuse Strategy:** **Extend** (resource-level → license-centric per-user analytic, C3).
- **Validation:** `[ ]` resource inactivity available · `[ ]` thresholds/config · `[ ]` suitability as input.

### 17. Historical Usage Retention

- **CoE Component:** Audit Components / Audit Logs.
- **Purpose:** How far back usage history is retained (the window).
- **Dataverse Tables (validate):** Audit log retention; archival/clean-up policy (validate).
- **Important Columns (validate):** Event date; retention/cleanup settings (validate).
- **Relationships (validate):** Audit log lifecycle.
- **Existing Views (validate):** n/a.
- **Existing Dashboards (validate):** n/a.
- **Existing Apps (validate):** Clean-up configuration.
- **Existing Power Automate Flows (validate):** Archive/clean-up flows (validate).
- **Relevance to this project:** **The window bounds how far back inactivity can be evidenced** (Q-COE-2). Longer window = higher confidence; short window must be disclosed.
- **Reuse Strategy:** **Reuse As-Is** (consume within the window; disclose the bound).
- **Validation:** `[ ]` **usage-history window length confirmed (blocking for C3)** · `[ ]` clean-up cadence · `[ ]` oldest retained event.

### 18. User-to-App Usage Relationships

- **CoE Component:** Audit Components.
- **Purpose:** Which user launched which app (per-user usage).
- **Dataverse Tables (validate):** Audit log linking user ↔ app (validate per-user retention).
- **Important Columns (validate):** **User (UPN / Entra object ID)**; App; timestamp (validate).
- **Relationships (validate):** Audit event N:1 App; N:1 User.
- **Existing Views (validate):** "App Usage by User" (validate; may not exist if aggregated).
- **Existing Dashboards (validate):** Usage-by-user pages (validate).
- **Existing Apps (validate):** Power BI (if modeled).
- **Existing Power Automate Flows (validate):** Audit collection flows.
- **Relevance to this project:** **This is the pivotal signal** — per-user app usage lets the project determine whether a *specific licensed user* is active (C3/C5). If CoE aggregates away per-user detail, this must be **supplemented**.
- **Reuse Strategy:** **Reuse As-Is** (if per-user usage retained) / **Supplement** (if only aggregates retained).
- **Validation:** `[ ]` **per-user usage retained (blocking)** · `[ ]` user key (UPN/object ID) · `[ ]` join to licensing feasible.

### 19. Owner-to-Usage Correlation Opportunities

- **CoE Component:** Core (ownership) + Audit (usage).
- **Purpose:** Combine who owns a resource with whether it is used.
- **Dataverse Tables (validate):** Join of App/Flow owner (Core) with usage (Audit).
- **Important Columns (validate):** Owner (Entra object ID); last launched/unique users (validate).
- **Relationships (validate):** Owner ↔ owned resource ↔ usage.
- **Existing Views (validate):** Owner+usage combined views (validate; may require join).
- **Existing Dashboards (validate):** Ownership+usage pages (validate).
- **Existing Apps (validate):** CoE app / Power BI (partial).
- **Existing Power Automate Flows (validate):** Inventory + audit flows (separately).
- **Relevance to this project:** The **correlation backbone** — owner (Core) + usage (Audit) + licensing (Graph) → per-user optimization (C2/C3/C4). CoE provides the two CoE-side halves; the join is project logic.
- **Reuse Strategy:** **Extend** (project correlation over reused Core ownership + Audit usage; C2).
- **Validation:** `[ ]` owner+usage joinable by stable keys · `[ ]` coverage · `[ ]` freshness alignment.

---

## Classification Summary

| # | Area | Component | Classification |
| --- | --- | --- | --- |
| 1 | Environment Inventory | Core | Reuse As-Is |
| 2 | Power App Inventory | Core | Reuse As-Is |
| 3 | Cloud Flow Inventory | Core | Reuse As-Is |
| 4 | Maker Inventory | Core | Reuse As-Is |
| 5 | Ownership Relationships | Core | Reuse As-Is |
| 6 | Connector Inventory | Core | Reuse As-Is |
| 7 | Connection Reference Inventory | Core | Reuse As-Is |
| 8 | CoE Administration Apps | Core | Reuse As-Is / Extend |
| 9 | Existing Dashboards | Core (+Audit) | Extend |
| 10 | Existing Power BI Reports | Core + Audit | Reuse As-Is / Extend |
| 11 | App Launch Telemetry | Audit | Reuse As-Is / Supplement |
| 12 | Unique User Metrics | Audit | Reuse As-Is |
| 13 | App Adoption Metrics | Audit | Reuse As-Is / Extend |
| 14 | Flow Usage Metrics | Audit | Reuse As-Is / Supplement |
| 15 | Last Used / Last Launched | Audit | Reuse As-Is |
| 16 | Inactivity Detection | Audit | Extend |
| 17 | Historical Usage Retention | Audit | Reuse As-Is |
| 18 | User-to-App Usage | Audit | Reuse As-Is / Supplement |
| 19 | Owner-to-Usage Correlation | Core + Audit | Extend |

No area is **Build New** — Build New is reserved for the licensing layer (acquisition, recommendation, reclamation, audit), which is not a CoE capability.

---

## Capabilities We Will Reuse Directly

These are confirmed CoE capabilities the project consumes with **no custom rebuild** (subject to validation):

- **Environment, app, flow, maker, connector, connection-reference inventory** (Core) — read-only via canonical IDs.
- **Ownership relationships** (Core) — interpreted for safety, not duplicated.
- **App launch telemetry, unique users, adoption, last-launched/last-run** (Audit) — primary usage evidence.
- **Historical usage within the retention window** (Audit) — consumed with freshness disclosed.
- **CoE administration apps** (Core) — deep-link targets for CoE context.
- **CoE Power BI semantic model** (Core + Audit) — reused and extended with licensing measures.
- **Existing views** across Core/Audit tables — reused as drill-through/list sources.

## Capabilities We Explicitly Will Not Build

Because CoE already provides them, the project will **not** build:

- Any **environment / app / flow / maker / connector / connection** inventory table or collection flow.
- Any **ownership/relationship** store — interpret CoE relationships in Dependency Finding only.
- Any **usage/app-launch/unique-user/last-launched collection** mechanism — consume CoE Audit.
- Any **resource-level inactivity** detection — reuse CoE; extend only to per-user, license-centric inactivity.
- A **new Power BI semantic model** or governance dashboards — extend the CoE model.
- **Browse/navigation UX** for inventory — deep-link to CoE apps/views.

This confirms and operationalizes the "Tables We Explicitly Will Not Create" list in `docs/data-model.md` and the reuse postures in `docs/capability-map.md`.

---

## Critical Validation Gaps (Blocking Before Build)

These assumptions **must be replaced with confirmed CoE artifacts** before design is finalized:

- `[ ]` **Maker Entra/Azure AD object ID** present and populated (correlation anchor — C2/C4).
- `[ ]` **Per-user app-launch usage retained** in CoE Audit (areas 11/18) — determines whether per-user inactivity is reused from CoE or **supplemented**.
- `[ ]` **Usage-history window length** confirmed (area 17; Q-COE-2) — bounds inactivity confidence.
- `[ ]` **Flow usage completeness** vs. app launches (area 14) — determines supplement need for flow-owning users.
- `[ ]` **App/flow owner + environment lookups** populated (areas 2/3/5) — C4 dependency safety.
- `[ ]` **Inventory and audit sync health/freshness** — underpins every recommendation.

If per-user usage (area 18) is **not** retained, raise it as a scope decision: supplement via a supported analytics/usage interface (ADR-004) or adjust the inactivity definition (Q-ANALYTICS-1).

---

## Exit Criteria

- Every area's tables, columns, relationships, views, dashboards, apps, and flows are confirmed (or deviations recorded) in `docs/reference/coe-toolkit/customer-deployment-inventory.md`.
- All project-relevant "(validate)" labels are replaced with confirmed artifacts.
- All blocking validations pass (or a supplement decision is recorded).
- The two reuse sections are finalized and the classification summary is confirmed.

---

## Constraints Honored

- Assessment/validation documentation only — no Power Platform assets, Dataverse schemas, or flows were generated, and nothing in CoE is modified.
- CoE names are treated as **expected and to-be-validated**, never invented or assumed authoritative.
- Findings maximize reuse and eliminate custom development: CoE Core + Audit remain the system of record; the project builds only the missing licensing layer.
