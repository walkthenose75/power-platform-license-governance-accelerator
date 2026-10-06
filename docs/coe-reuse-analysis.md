# CoE Reuse Analysis

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Method

This is an **architectural reuse analysis only**. It creates no Power Platform assets, no Dataverse schemas, and no Power Automate flows. Its objective is to **maximize reuse of the customer's existing CoE investment and minimize custom development**, as required by the project charter and by `docs/reference/coe-toolkit/README.md`.

### Three evidence layers (kept deliberately distinct)

Per the reference directive, every statement below is tagged to the layer it comes from. These layers must not be conflated.

| Layer | Meaning | Confidence for design |
| --- | --- | --- |
| **[REF]** Archived reference repository | What the archived Microsoft CoE Starter Kit repo contained (read-only since July 2, 2026). | Indicative only. Names/components vary by version. |
| **[LRN]** Microsoft Learn documentation | What Microsoft documents the Core/Audit components as providing. | Indicative of intended behavior. |
| **[CUST]** Customer-installed deployment | Assumption:

The customer has **Core Components and Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)** deployed and available, and intends to continue using CoE.

Exact solution versions, enabled components, sync health, and deployment configuration must be validated during implementation. | **Authoritative and available.** |

> **Deployment context:** The customer's environment has the **CoE Core Components and Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)** deployed and available. The analysis therefore treats CoE inventory, ownership, environment, and maker data (Core Components) and usage/last-launched telemetry (Audit Components/Audit Logs), together with the CoE Power BI governance dashboards sourced from them, as **available and in use** — not as hypotheses. The accelerator depends on these two CoE solutions only. The remaining step is a routine, non-blocking confirmation of **exact artifact names, schemas, and versions** against the customer's exported solution during design, because the reference guardrails prohibit inventing names.

### Naming disclaimer (required by the reference guardrails)

The reference README explicitly prohibits inventing CoE table, column, flow, app, or dashboard names and requires validation against the customer's actual environment. Accordingly:

- Capability-level descriptions are drawn from the README and Microsoft Learn and are reasonably stable.
- **Specific artifact names are illustrative reference labels only**, marked "(validate)". The components exist in the customer's deployment, but exact names/schemas may differ by version and should be confirmed against the exported solution during design.

### CoE guardrails carried into this analysis

- Do not modify Microsoft-managed CoE tables, apps, flows, environment variables, or connection references.
- Keep all custom licensing components in a **separate project-owned solution**.
- Read CoE data through a **project-owned adapter/query layer**; depend on stable Dataverse relationships and supported APIs, not implementation-specific payloads.
- **Never interpret missing CoE usage data as zero usage.**
- Record **source, collection time, and freshness** for every piece of evidence used in a reclamation recommendation.

---

## Part A — Existing CoE Capability Inventory

This part answers the seven capability questions in scope. Each item states the documented function, the reference component group that provides it, illustrative reference artifacts (to validate), and its relevance to this licensing project.

### A.1 Existing inventory capabilities

- **Documented function [LRN][REF]:** The CoE **Core Components** maintain an automated inventory of the Power Platform estate: environments, Power Apps (canvas and model-driven), cloud flows, desktop flows, makers/owners, connectors, connection references, Power Pages sites, and Copilot Studio agents, along with their relationships. Inventory is populated by scheduled synchronization processes and can run in a cloud-flow inventory mode or a Data Export (BYODL)-style mode depending on configuration.
- **Reference component group:** Core Components.
- **Illustrative reference artifacts (validate):** "Admin | Sync Template v4 (…)" inventory flows; core inventory tables for environments/apps/flows/makers/connectors.
- **Relevance to this project:** This is the **authoritative inventory substrate**. The project must **reuse it as the source of truth** for apps, flows, makers, owners, and environments and must **not** rebuild any of it.

### A.2 Existing Dataverse tables

- **Documented function [LRN][REF]:** Core Components store the inventory in managed Dataverse tables representing environments, apps, flows, makers, connectors, connection references, Power Pages, and Copilot Studio agents, plus relationship and configuration tables. Audit Components/Audit Logs — **deployed in this environment** — add usage- and telemetry-oriented tables.
- **Reference component group:** Core Components; Audit Components; Audit Logs.
- **Illustrative reference artifacts (validate):** environment table, app table, flow table, maker table, connector table, and audit-log/usage tables. **Exact logical and schema names, and the columns present, must be read from the customer's exported solution — not assumed.**
- **Relevance to this project:** The project should **read these tables through a project-owned adapter** and correlate to licensing data. It must **not** copy them into a parallel inventory. Any project-owned table is justified only for licensing data CoE does not hold (see Part B and the duplication register).

### A.3 Existing Power Automate flows

- **Documented function [LRN][REF]:** CoE ships managed cloud flows that (a) **collect inventory** on a schedule, (b) **collect audit/usage telemetry** (Audit Components are **deployed in this environment**), (c) run **compliance/nurture/notification** processes, and (d) perform **housekeeping/clean-up**. These flows are the data-collection engine behind the tables and dashboards.
- **Reference component group:** Core Components (inventory sync); Audit Components/Audit Logs (usage telemetry); Nurture (maker engagement — not licensing-relevant).
- **Illustrative reference artifacts (validate):** "Admin | Sync Template v4 (…)" family; audit-log collection flows; clean-up/compliance flows.
- **Relevance to this project:** **Reuse these collection processes as the data source.** Do **not** re-implement inventory or usage collection. The project's own automation should be limited to the missing licensing layer (acquisition, correlation, recommendation, administrator-initiated reclamation) and must not modify CoE-managed flows.

### A.4 Existing dashboards

- **Documented function [LRN][REF]:** CoE provides **Power BI governance dashboards** and administrative **model-driven apps / command-center experiences** that visualize environments, apps, flows, makers, connectors, usage, and compliance. These are the existing operational and executive visibility surfaces.
- **Reference component group:** Power BI assets; Core Components admin apps.
- **Illustrative reference artifacts (validate):** the CoE Power BI governance dashboard; the CoE administration/command-center app.
- **Relevance to this project:** **Prefer extending or surfacing existing CoE dashboards** for licensing/optimization visibility before building any new dashboard (charter reuse order; ADR-003). A separate experience is justified only by a documented licensing-specific gap.

### A.5 Existing maker analytics

- **Documented function [LRN][REF]:** CoE inventories **makers/owners** and relates them to the apps, flows, and connectors they own, surfacing per-maker counts and activity in dashboards; Nurture components add maker engagement/onboarding analytics.
- **Reference component group:** Core Components (maker inventory/relationships); Nurture (engagement).
- **Illustrative reference artifacts (validate):** maker table and maker-centric dashboard pages.
- **Relevance to this project:** **Reuse maker/owner data** for dependency analysis (identifying critical app/flow owners). Do **not** rebuild maker analytics. The project adds only the **licensing overlay** on top of maker/owner facts.

### A.6 Existing app usage analytics

- **Documented function [LRN][REF]:** With **Audit Components / Audit Logs** deployed (as in this environment) and **app-launch and unique-user collection configured**, CoE collects **app launches, unique users, and last-launched** information over an available **usage-history window** (bounded by retention). This is resource-level usage telemetry.
- **Reference component group:** Audit Components; Audit Logs.
- **Illustrative reference artifacts (validate):** audit-log usage tables; usage/adoption dashboard pages.
- **Relevance to this project:** This is the **primary usage-evidence source** for "is the premium capability actually being used." Audit Components/Audit Logs are installed and in use, so app-launch/unique-user telemetry is available within the configured usage-history (retention) window. The remaining constraint is the **window length**, which bounds how far back inactivity can be evidenced. **Missing usage data for a given user or app must never be read as zero usage** — absence within the window is not proof of inactivity (reference guardrail; critical for safe reclamation).

### A.7 Existing inactivity detection capabilities

- **Documented function [LRN][REF]:** CoE surfaces **resource-level inactivity/orphaned-resource signals** — for example last-launched/last-run and orphaned or inactive apps/flows — and can drive compliance/clean-up notifications. This detection is **resource-centric** (an app or flow is unused/orphaned).
- **Reference component group:** Core Components (last-run/orphaned signals); Audit Components (usage-derived inactivity); compliance/clean-up flows.
- **Illustrative reference artifacts (validate):** orphaned/inactive resource views and compliance dashboard pages.
- **Relevance to this project:** CoE detects **inactive resources**, not **inactive premium-licensed users**. The project's needed capability — correlating **per-user premium license assignment** with usage/dependency to recommend **safe license reclamation** — is **not** provided by CoE inactivity detection. This is the core capability gap (see Part B.4–B.7).

---

## Part B — Feature-by-Feature Reuse Analysis

For each envisioned project feature (derived from `docs/product-definition.md`, `docs/requirements.md`, and `docs/backlog.md`), the analysis maps the fields requested for this document — **existing CoE capability, existing CoE table(s), existing CoE flow(s), existing CoE dashboard(s), recommended reuse strategy, build-vs-extension recommendation, and risk of duplication** — plus deployment status and upgrade/maintenance risk required by the reference.

> Reuse preference order (from the reference): **1** reuse as-is → **2** read via project adapter → **3** extend with project-owned components → **4** supplement via supported Graph/Power Platform API/admin connector → **5** build new only if none of the above suffices.

### B.1 CoE capability discovery (FR-1)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | Full inventory + audit components constitute the thing being discovered. [LRN][REF] |
| Existing CoE table(s) | Core/Audit solution tables (validate). [REF] |
| Existing CoE flow(s) | Inventory and audit sync flows (validate). [REF] |
| Existing CoE dashboard(s) | Governance dashboards and admin apps indicate what is populated. [REF] |
| CoE deployment status | All CoE components installed and in use [CUST]; this step only confirms exact names/versions during design. |
| Recommended reuse strategy | **Reuse (read/inspect only).** Project-owned discovery notes; no CoE changes. |
| Build vs Extension | **Neither** — discovery/documentation activity. |
| Risk of duplication | **Low** — discovery, not rebuild. |
| Upgrade / maintenance risk | **Low** — read-only; re-run when CoE version/config changes. |

### B.2 Licensing intelligence: premium assignment + SKU + assignment context (FR-2)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | **None for commercial licensing.** CoE inventories apps/flows/makers, not Microsoft 365/Power Platform **license entitlements**. [LRN] |
| Existing CoE table(s) | None expected for per-user premium assignment/SKU (validate). |
| Existing CoE flow(s) | None in CoE for license entitlement acquisition. |
| Existing CoE dashboard(s) | None in CoE for per-user license entitlement. |
| CoE deployment status | Full CoE in use [CUST], but licensing entitlement is still external to CoE (Graph-sourced). |
| Recommended reuse strategy | **Supplement via supported API** (level 4): Microsoft Graph for assignments/SKUs; distinguish direct vs. group-based assignment where supported. |
| Build vs Extension | **Build (new, project-owned)** acquisition via validated interfaces (ADR-004). |
| Risk of duplication | **Low** — this data is not in CoE. |
| Upgrade / maintenance risk | **Medium** — depends on Graph API support/permissions/throttling; validate before adoption. |

### B.3 Correlation of licensing with CoE inventory/ownership/usage (FR-3)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | Inventory, maker/owner, environment, and usage data to correlate **against**. [LRN][REF] |
| Existing CoE table(s) | Core (apps/flows/makers/env) + Audit (usage) tables (validate). [REF] |
| Existing CoE flow(s) | CoE sync/audit flows keep the CoE side current (validate). [REF] |
| Existing CoE dashboard(s) | CoE dashboards already join inventory + usage + ownership. [REF] |
| CoE deployment status | CoE inventory/usage available [CUST]; confirm identifier fields and match quality during design. |
| Recommended reuse strategy | **Reuse via project-owned adapter** (level 2): read CoE through stable relationships/identifiers; **correlation logic is project-owned**. |
| Build vs Extension | **Extension (thin)** — matching layer only; no inventory rebuild. |
| Risk of duplication | **Medium** — risk of copying CoE data into a project store. Mitigate by referencing, not replicating. |
| Upgrade / maintenance risk | **Medium** — depends on CoE schema/identifier stability across versions. |

### B.4 Optimization analytics: inactive licensed users (FR-4)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | **Resource-level** usage/inactivity (app/flow last used), via Audit Logs (installed and in use). [LRN][REF] |
| Existing CoE table(s) | Audit usage tables (deployed; confirm exact names during design). [REF] |
| Existing CoE flow(s) | Audit-log usage collection flows (deployed and enabled; confirm names). [REF] |
| Existing CoE dashboard(s) | Usage/adoption dashboard pages. [REF] |
| CoE deployment status | Audit Logs installed and in use [CUST]; usage telemetry available within the retention window. |
| Recommended reuse strategy | **Reuse usage evidence + extend**: consume CoE usage; add **per-user, license-aware** inactivity analysis the project owns. |
| Build vs Extension | **Extension** — license-centric analytic over reused resource usage. |
| Risk of duplication | **Medium-High** — risk of rebuilding usage collection. Mitigate by consuming CoE usage, not re-collecting it. |
| Upgrade / maintenance risk | **Medium** — usage evidence quality depends on Audit config; **missing ≠ zero usage.** |

### B.5 Dependency analysis before remediation (FR-5 / FR-6 precondition)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | **Ownership and relationship** data (maker→app/flow, app/flow→environment/connector). [LRN][REF] |
| Existing CoE table(s) | Core inventory + relationship tables (validate). [REF] |
| Existing CoE flow(s) | Inventory sync flows (validate). [REF] |
| Existing CoE dashboard(s) | Dashboards show ownership/relationships; criticality classification may exist if the customer configured it. [REF][CUST] |
| CoE deployment status | CoE relationships available [CUST]; confirm whether business-criticality classification is populated. |
| Recommended reuse strategy | **Reuse CoE relationships + extend**: project-owned rules that interpret dependencies for licensing safety. |
| Build vs Extension | **Extension (logic only)** — no new inventory. |
| Risk of duplication | **Medium** — do not rebuild relationship inventory; interpret CoE's. |
| Upgrade / maintenance risk | **Medium** — depends on relationship-data completeness and criticality configuration. |

### B.6 Explainable recommendations — Safe / Review / Blocked (FR-5)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | **None** — CoE does not produce license-reclamation recommendations with assignment-source, exception, and freshness explainability. [LRN] |
| Existing CoE table(s) | None for this purpose. |
| Existing CoE flow(s) | None for this purpose. |
| Existing CoE dashboard(s) | None for this purpose. |
| CoE deployment status | Full CoE in use [CUST]; no CoE equivalent for license-reclamation recommendations. |
| Recommended reuse strategy | **Build new** (level 5), consuming reused CoE + supplemented licensing data as inputs. |
| Build vs Extension | **Build (new, project-owned)** recommendation + explainability model. Missing/stale evidence must block **Safe**. |
| Risk of duplication | **Low** — genuinely missing capability. |
| Upgrade / maintenance risk | **Low-Medium** — depends on input stability. |

### B.7 Safe remediation: dry run, guardrails, bounded bulk, audit (FR-6 / FR-7)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | CoE has **resource** compliance/clean-up and notifications, **not per-user premium license reclamation** with dry-run + protected-identity guardrails. [LRN][REF] |
| Existing CoE table(s) | None for license reclamation. |
| Existing CoE flow(s) | CoE compliance flows are resource-oriented and must not be modified/repurposed. |
| Existing CoE dashboard(s) | Dashboards show compliance posture, not license reclamation actions. |
| CoE deployment status | Full CoE in use [CUST]; no CoE equivalent for per-user license reclamation. |
| Recommended reuse strategy | **Build new** using **supported APIs** for the license change; **reuse/configure** protected-identity and exception inputs. |
| Build vs Extension | **Build (new, project-owned)** — administrator-initiated, dry-run, guardrailed, audited. **No approval workflow; no automatic removal.** |
| Risk of duplication | **Low** — missing capability. |
| Upgrade / maintenance risk | **Medium** — depends on supported license-change API stability/permissions. |

### B.8 Visibility and audit surfaces (FR-7)

| Dimension | Finding |
| --- | --- |
| Existing CoE capability | **Dashboards and admin experiences** already exist for governance visibility. [LRN][REF] |
| Existing CoE table(s) | Power BI datasets / admin app tables (validate). [REF] |
| Existing CoE flow(s) | N/A (presentation layer). |
| Existing CoE dashboard(s) | CoE governance dashboards. [REF] |
| CoE deployment status | CoE dashboards installed and in use [CUST]; confirm the preferred extension targets during design. |
| Recommended reuse strategy | **Extend/surface CoE dashboards first** (level 3); reclamation audit trail is **project-owned**. |
| Build vs Extension | **Extension** for visibility; **Build** only the reclamation audit record. |
| Risk of duplication | **High if ignored** — rebuilding dashboards is the most likely duplication. Mitigate by extending CoE reporting. |
| Upgrade / maintenance risk | **Low-Medium** — keep extensions isolated from CoE-managed report assets. |

---

## Part C — Reuse vs. Build Summary (Maximize Reuse / Minimize Custom)

| Envisioned feature | Primary decision | Why |
| --- | --- | --- |
| CoE capability discovery (B.1) | **Reuse (inspect)** | Discovery of existing assets. |
| Inventory of apps/flows/makers/owners/env | **Reuse as-is (level 1–2)** | Fully provided by CoE Core Components. |
| App/flow **usage** evidence | **Reuse (level 1–2)** | Provided by CoE Audit (installed and in use); never treat missing as zero. |
| Maker/owner **dependency** data | **Reuse + extend logic (level 2–3)** | Relationships exist in CoE; interpretation is project-owned. |
| **Licensing entitlement** (premium + SKU + assignment context) | **Supplement via supported API (level 4)** | Not a CoE capability; Graph-sourced. |
| **Correlation** licensing↔CoE | **Extend: thin logic over reused data (level 2→3)** | Matching layer only; no inventory rebuild. |
| **Inactive licensed-user** analytics | **Extend (level 3)** | License-centric view over reused resource usage. |
| **Safe/Review/Blocked recommendations** | **Build new (level 5)** | Genuinely missing; consumes reused + supplemented inputs. |
| **Safe reclamation** (dry run/guardrails/audit) | **Build new via supported APIs (level 4–5)** | Missing; must honor safety guardrails. |
| **Visibility/dashboards** | **Extend/surface CoE first (level 3)** | Avoid rebuilding governance dashboards. |
| **Reclamation audit trail** | **Build new (level 5)** | Action-specific, project-owned. |

**Net position:** The large majority of required data (inventory, ownership, usage, dependency, visibility) is **reused or extended** from CoE. Genuinely new build is confined to four thin, well-bounded areas: **licensing acquisition (via Graph), correlation logic, the recommendation/explainability model, and safe administrator-initiated reclamation with audit.** This maximizes reuse and minimizes custom development, consistent with the charter.

---

## Part D — Duplication Risk Register

| ID | Potential duplication | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| D-1 | Rebuilding app/flow/maker/environment **inventory** | Medium | High | Read CoE via adapter; never create a parallel inventory (charter Principle 1). |
| D-2 | Re-collecting **usage** telemetry instead of consuming CoE Audit | Medium | High | Audit Components are installed and in use — consume CoE usage directly; do not stand up parallel collection. |
| D-3 | Rebuilding **dashboards** rather than extending CoE reporting | High | Medium | Extend/surface CoE Power BI/admin apps first (ADR-003). |
| D-4 | Re-deriving **maker analytics** | Medium | Medium | Reuse CoE maker/owner data; add only the licensing overlay. |
| D-5 | Copying CoE tables into a **project store** for convenience | Medium | High | Reference/relate; justify any retained copy with ownership, retention, reconciliation. |
| D-6 | Re-implementing **resource inactivity** that CoE already surfaces | Low-Medium | Medium | Reuse CoE resource signals; build only the **license-centric** analytic. |
| D-7 | Repurposing/modifying CoE **compliance/clean-up flows** | Low | High | Never modify managed CoE flows; keep reclamation in a separate solution. |

---

## Part E — Upgrade and Maintenance Risk (Archived Kit)

- **Kit is archived/read-only (since July 2, 2026).** Core Components and Audit Components are deployed and in use today, but the kit will not receive further updates and Microsoft is shifting governance toward Power Platform admin center-native capabilities. Design the adapter so licensing logic does not hard-depend on kit internals. [REF]
- **Version drift:** The installed version may differ from the archived final version. Do not assume table/column/flow/dashboard names; confirm them against the exported solution during design. [CUST]
- **Health/freshness:** Although the components are installed, inventory or audit sync flows can still fail, be suspended, or be stale at runtime — affecting evidence quality. Record collection time and freshness for every recommendation. [CUST]
- **Isolation requirement:** Keep all custom components in a separate project-owned solution; document every direct dependency on a CoE component to contain upgrade impact.

---

## Part F — Design-Time Confirmations

The customer runs the **CoE Core Components and Audit Components (CenterOfExcellenceAuditComponents)**, so the availability of the components this accelerator depends on is settled and is **not a blocker**. The remaining items are routine design-time confirmations, not a gating inventory.

**CoE (light confirmation during design):**

- Confirm the **exact table, column, flow, app, and dashboard names/schemas** to replace the "(validate)" labels used here (the reference guardrails prohibit inventing names).
- Confirm the **Core Components inventory mode** (cloud-flow inventory vs. Data Export) and the **usage-history window** length that bounds how far back inactivity can be evidenced.
- Confirm which **Power BI reports / admin apps** are the preferred **extension targets**.
- Confirm whether **business-criticality / critical-app** classification is populated (input to dependency analysis).
- Monitor **sync health/freshness** at runtime and record collection time and freshness for every recommendation.

**External interfaces (still required — unrelated to CoE deployment):**

- Validate **Microsoft Graph licensing** and any **Power Platform admin API/connector** operations for support, permissions, licensing, throttling, and availability per ADR-004. This is the one genuine prerequisite, because licensing entitlement is sourced outside CoE.

---

## Part G — Conclusion and Recommendations

1. **Reuse is the default.** CoE already provides inventory, ownership, usage, dependency data, and visibility surfaces. The project should consume these through a project-owned adapter and extend existing dashboards rather than rebuild anything.
2. **Build only the missing licensing layer.** Net-new development is limited to: licensing acquisition (Graph), correlation logic, the explainable recommendation model, and safe administrator-initiated reclamation with audit.
3. **Licensing entitlement is the true gap.** CoE does not track per-user premium license assignment or SKU; this is the only area requiring external supported-API supplementation.
4. **Usage/inactivity evidence is available but must be handled safely.** Audit components are installed and in use; evidence is bounded by the retention window, and missing usage data for a user or app must never be interpreted as zero usage.
5. **Protect against duplication and upgrade risk.** Keep custom components isolated, reference rather than copy CoE data, never modify managed CoE assets, and document every dependency.
6. **Confirm exact artifact names during design, and validate the external interface.** Component availability is settled (full CoE in use); the remaining steps are confirming exact CoE names/schemas and validating the external Microsoft Graph licensing interface (ADR-004), which is the one true prerequisite.

This analysis contains no Power Platform assets, schemas, or flows, consistent with the task constraints and the project charter.