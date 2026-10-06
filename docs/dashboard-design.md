# Dashboard and Analytics Design

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Scope

This document is the **analytics and dashboard design** for the Power Platform License Governance Accelerator. It is **design documentation only** — it creates no reports, no Power BI assets, no datasets/semantic models, no Dataverse charts, and no model-driven dashboards. Measures are described **conceptually**; no DAX, queries, or visuals are produced.

It designs three dashboards:

1. **Executive Leadership** (P6 Executive Sponsor, P3 IT Finance/FinOps)
2. **Power Platform Administrators** (P1 Platform/CoE Admin, P4 Environment/BU Admin)
3. **License Governance Administrators** (P1 authorized admin, P5 Security & Compliance oversight)

It assumes the customer operates the CoE **Core Components and Audit Components (CenterOfExcellenceAuditComponents)**, so dashboards **reuse and extend** CoE usage (Audit), inventory/environment/ownership (Core), and the Power BI governance assets sourced from them, and **build new** only for the licensing layer CoE does not provide. Capability IDs (C1–C8) reference `docs/product-definition.md`; table and app context reference `docs/app-navigation.md`.

### Classification legend (per component)

- **Reuse** — present existing CoE data/visuals as-is (reuse CoE datasets, usage, inventory, sync health, or deep-link to CoE Power BI).
- **Extend** — licensing measures/visuals built on reused CoE data (for example, utilization computed from CoE usage + licensing).
- **Build New** — visuals over net-new project-owned tables (candidates, recommendations, reclamation, audit, protected identities, assignments).

---

## Analytics Architecture and Technology Selection

Three implementation technologies are used **where appropriate**. The selection rule below keeps the design metadata-driven and CoE-reusing.

| Technology | Best for | Strengths | Limits | Typical classification |
| --- | --- | --- | --- | --- |
| **Dataverse charts** | Single-table aggregates over project-owned tables | Native, fast, drill to records/forms, no extra licensing | Single-table aggregation, limited joins, basic visuals | Build New |
| **Model-driven app (MDA) dashboards** | Operational triage composing charts + streamed/interactive lists | In-app, role-aware, drill-through to explainable records, interactive filters | Not for heavy cross-source modeling or time-series | Build New / Extend |
| **Embedded Power BI** | Cross-source analytics, trends/time-series, executive polish | Reuses CoE semantic model, rich visuals, row-level security, embeds in MDA | Requires Power BI governance/licensing; modeling effort | Reuse / Extend |

**CoE Power BI reuse strategy (critical for charter alignment):**

1. **Reuse** the existing CoE Power BI **semantic model/datasets** for usage, adoption, inventory, and environment facts rather than re-modeling them.
2. **Extend** that model with **licensing measures** (entitlement, utilization, optimization, reclamation) sourced from Graph + project-owned tables.
3. **Compose** a project-owned report that references the extended model; **embed** it in the MDA where operational context helps.
4. **Deep-link** to existing CoE governance dashboards for full estate browsing instead of rebuilding them (anti-duplication; risks D-2/D-3 in `docs/coe-reuse-analysis.md`).

---

## Shared Data Sources

All dashboards draw from the same catalog. CoE sources are **read-only**; licensing entitlement is **supplemented** via Microsoft Graph; action/outcome data is **project-owned**.

| Source | Provides | Classification |
| --- | --- | --- |
| CoE Audit usage telemetry | App/flow usage, last-launched, unique users (within retention window) | Reuse |
| CoE inventory & environment | Apps, flows, environments, owners/makers, relationships | Reuse |
| CoE Power BI semantic model | Existing usage/adoption/governance measures | Reuse (extend) |
| CoE sync health / last successful sync | Freshness signals | Reuse |
| Microsoft Graph licensing | Per-user premium assignment, SKU, direct vs. group source | Supplement (feeds Build New) |
| License Assignment (project-owned) | Correlated entitlement records | Build New |
| Optimization Candidate / Recommendation Evidence | Classifications + explainability | Build New |
| Dependency Finding | Safety dependencies (surfacing CoE relationships) | Build New (surfaces Reuse) |
| Reclamation Action / Audit Log Entry | Dry-run, executed, outcomes, immutable audit | Build New |
| Protected Identity / Exception | Never-reclaim coverage | Build New |
| SKU Reference | Premium SKU interpretation and value basis | Build New |

---

## Shared KPI and Metric Definitions

Defined once so the three dashboards stay consistent. All estimates disclose assumptions and freshness; **missing usage is never treated as zero usage**.

| KPI | Conceptual definition | Focus area | Classification |
| --- | --- | --- | --- |
| Premium Utilization Rate | % of premium-licensed users with evidence of premium-requiring activity in the usage window | Usage / Optimization | Extend |
| Evidence Coverage | % of premium users with sufficient, fresh usage evidence to classify | Risk / Data quality | Build New |
| Potentially Inactive Licensed Users | Count/% flagged inactive, qualified by evidence coverage | Optimization / Usage | Build New (over Extend) |
| Estimated Reclaimable Licenses | Count of Safe candidates | Optimization / Reclamation | Build New |
| Estimated Optimization Value | Monetary estimate of Safe candidates (assumptions disclosed) | Optimization | Build New |
| Realized Reclamations / Savings | Executed and audited reclamations and their value | Value realization | Build New |
| Estimated vs. Realized | Comparison of opportunity to realized value | Value realization | Build New |
| Accidental Removals Prevented | Count of protected/group/stale/dependency blocks that stopped unsafe action | Risk | Build New |
| Protected-Identity Coverage | % of sensitive accounts covered by exceptions | Risk | Build New |
| Direct vs. Group Assignment | Split of premium assignments by source (only direct is reclaimable) | Risk / Optimization | Build New |
| Dependency Risk | Candidates with critical app/flow/ownership dependencies | Dependency | Build New (over Extend) |
| Reclamation Pipeline Status | Counts by operational status (dry-run, executed, failed) | Reclamation | Build New |
| Data Freshness / As-of | Age of last successful CoE sync and licensing acquisition | Risk / Trust | Build New (references Reuse) |
| Correlation / Unmatched Rate | % of licensing records unmatched to CoE | Risk / Data quality | Build New |
| Premium by Environment / SKU / BU | Distribution of premium licenses | Usage / Optimization | Extend |

---

## Cross-Cutting Analytics Guardrails

Applied to every dashboard and component:

- **Freshness on every surface.** Each dashboard shows an as-of/freshness indicator; stale data is flagged, not hidden.
- **Estimated vs. realized separated.** Optimization value (estimate) and realized savings (audited) are never merged into one "savings" number.
- **Missing ≠ zero usage.** Inactivity visuals pair with an Evidence Coverage indicator; low coverage lowers confidence rather than inflating "inactive."
- **No approval analytics.** The reclamation "pipeline" is an **operational status** funnel (dry-run → executed → audited), not an approval/sign-off chain (ADR-002).
- **Drill to explanation.** Operational visuals drill through to the explainable candidate record (evidence, dependencies, assignment source, exceptions) per `docs/app-navigation.md`.
- **Safety is a metric, not a blocker to override.** Protected/group exclusions are shown as risk-avoidance outcomes.
- **No CoE duplication.** Where CoE already visualizes a fact, reuse or deep-link; add only the licensing lens.

---

## Dashboard 1 — Executive Leadership

### Audience

P6 Executive Sponsor and P3 IT Finance/FinOps. Read-only, non-operational. Aligns to the "Executive Summary" in `docs/app-navigation.md`.

### Goals

- Show controlled premium spend and **realized** value.
- Show remaining **estimated** optimization opportunity without overstating it.
- Provide confidence that risk is controlled (no accidental removals) and that CoE is being reused rather than replaced.

### KPIs

Estimated Optimization Value; Realized Savings; Estimated vs. Realized; Premium Utilization Rate; Potentially Inactive (high level, with coverage); Accidental Removals Prevented; CoE Reuse Ratio (program metric).

### Charts

- **Line/area:** realized savings over time.
- **Clustered column:** estimated vs. realized by period.
- **Bar:** premium distribution by business unit/environment.
- **Waterfall (optional):** from estimated opportunity to realized value.

### Visualizations

- **KPI cards:** estimated value, realized savings, utilization rate.
- **Gauge/donut:** premium utilization.
- **As-of/freshness banner** with a confidence note on evidence coverage.

### Filters

Time period; business unit/environment; SKU. (Deliberately few and high-level.)

### Business questions answered

- How much premium are we carrying, and how much is justified by use?
- What have we **actually** saved, and what is still **estimated** as reclaimable?
- Are we avoiding risky removals?
- Are we extending CoE rather than building a new platform?

### Data sources

Embedded Power BI reusing CoE usage/adoption and environment datasets, **extended** with licensing measures (Graph), and combined with project-owned Reclamation/Audit for realized value.

### CoE reuse opportunities

- **Reuse** CoE Power BI semantic model and usage/environment facts.
- **Extend** it with licensing/optimization measures instead of a new model.
- **Deep-link** to CoE governance dashboards for estate depth.

### Implementation and component classification

| Component | Technology | Focus area | Classification |
| --- | --- | --- | --- |
| Realized savings trend | Embedded Power BI | Optimization/value | Build New (over reused model) |
| Estimated vs. realized | Embedded Power BI | Value realization | Build New |
| Premium utilization gauge | Embedded Power BI | Usage | Extend |
| Premium by BU/environment | Embedded Power BI | Usage/optimization | Extend |
| Accidental removals prevented card | Embedded Power BI | Risk | Build New |
| CoE reuse ratio card | Embedded Power BI | Program | Build New |
| Freshness/confidence banner | Embedded Power BI | Risk/trust | Build New (references Reuse) |
| Link to CoE governance dashboards | Power BI/URL | — | Reuse |

---

## Dashboard 2 — Power Platform Administrators

### Audience

P1 Platform/CoE Admin and P4 Environment/BU Admin. Operational, estate-wide. These admins already use CoE dashboards daily; this dashboard adds the **licensing lens** over CoE, and must not re-create CoE's inventory or adoption dashboards. Aligns to and refines "Optimization Overview."

### Goals

- Understand where premium licenses are concentrated and underused across the estate.
- Surface **risk** (group-assigned, stale evidence, dependency hotspots) and **usage** patterns.
- Identify environment/SKU optimization targets and know where to act — with CoE reuse throughout.

### KPIs

Premium Utilization Rate; Potentially Inactive (with coverage); Estimated Reclaimable; Direct vs. Group split; Dependency Risk; Premium by environment/SKU; Data Freshness; Unmatched Rate.

### Charts

- **Bar:** premium licenses by environment.
- **Stacked column / heatmap:** utilization by environment and SKU.
- **Line:** usage trend (reusing CoE usage).
- **Pie:** direct vs. group assignment.
- **Bar:** dependency hotspots (apps/flows with the most premium-owner dependencies).
- **Bar:** top environments by estimated reclaimable.

### Visualizations

- **KPI tiles:** utilization, inactive (with coverage), estimated reclaimable, unmatched rate.
- **Data freshness / sync-health tiles** (reusing CoE sync health + acquisition status).
- **Interactive filtering** across the page.

### Filters

Environment; business unit; SKU; assignment type (direct/group); evidence freshness; classification.

### Business questions answered

- Where is premium concentrated, and where is it underused?
- Which environments/SKUs are the best optimization targets?
- How much is **group-assigned** (not directly reclaimable)?
- Where are the dependency risks, and is my data fresh and well-matched?

### Data sources

CoE usage/inventory/environment (**reuse**), Microsoft Graph licensing (**supplement**), and project-owned correlation + candidates + dependency findings (**build new**).

### CoE reuse opportunities

- **Reuse** CoE usage/adoption and environment inventory as the base.
- **Extend** with the licensing overlay and optimization measures.
- **Do not rebuild** CoE estate dashboards; **deep-link** for full inventory browsing.

### Implementation and component classification

| Component | Technology | Focus area | Classification |
| --- | --- | --- | --- |
| Premium by environment | Embedded Power BI | Usage/optimization | Extend |
| Utilization by environment/SKU | Embedded Power BI | Usage | Extend |
| Usage trend | Embedded Power BI | Usage | Reuse (CoE usage) |
| Direct vs. group split | Dataverse chart / Power BI | Risk | Build New |
| Dependency hotspots | Embedded Power BI | Dependency | Build New (over Extend) |
| Estimated reclaimable by environment | Dataverse chart / Power BI | Optimization/reclamation | Build New |
| Inactive with coverage | Power BI | Optimization/risk | Build New (over Extend) |
| Data freshness / sync-health tiles | MDA dashboard / Power BI | Risk/trust | Reuse + Build New |
| Unmatched correlation rate | Dataverse chart | Data quality/risk | Build New |
| Link to CoE inventory/dashboards | URL | — | Reuse |

---

## Dashboard 3 — License Governance Administrators

### Audience

P1 authorized License Governance Administrator and P5 Security & Compliance (oversight, read-only). Action- and accountability-oriented. Aligns to "Optimization Overview" (triage) + "Data & Interface Status" + Audit in `docs/app-navigation.md`.

### Goals

- Triage candidates (Safe/Review/Blocked) and act on **reclamation opportunities** safely.
- Monitor the reclamation **operational pipeline** and **audit** outcomes.
- Ensure **safety** (protected-identity coverage, group/stale exclusions) and **data quality** (freshness, unmatched).

### KPIs

Candidates by classification; Estimated Reclaimable + value; Reclamation Pipeline Status; Realized Reclamations; Accidental Removals Prevented; Protected-Identity Coverage; Correlation/Unmatched Rate; Evidence staleness count.

### Charts

- **Column:** candidates by classification (Safe/Review/Blocked).
- **Funnel:** reclamation operational status (dry-run → executed) — not approval.
- **Bar:** estimated reclaimable value by SKU/environment.
- **Pie:** blocked reasons (protected / group / stale / dependency).
- **Histogram:** evidence freshness distribution.
- **Line/column:** audit activity over time by outcome.

### Visualizations

- **Interactive/streamed triage list** of Safe candidates with filters and drill-through to the explainable record.
- **KPI tiles:** estimated reclaimable, realized, protected-identity coverage, unmatched rate.
- **Data & interface status** panel (freshness, sync health, Graph status).

### Filters

Classification; environment; SKU; assignment type; evidence freshness; blocked reason; actor; time period.

### Business questions answered

- Which users can I **safely reclaim now**, and why are others Review/Blocked?
- What is in the reclamation pipeline, and what did we **actually** reclaim (audited)?
- Are protected identities fully covered, and are group-assigned users correctly excluded?
- How stale or unmatched is my evidence?

### Data sources

Project-owned Optimization Candidate, Recommendation Evidence, Dependency Finding, Reclamation Action, Audit Log Entry, Protected Identity/Exception, License Assignment, SKU Reference (**build new**); correlated CoE ownership/usage evidence surfaced on drill-through (**reuse/extend**); CoE sync health for freshness (**reuse**).

### CoE reuse opportunities

- **Reuse** CoE ownership/usage evidence in drill-through explanations and CoE sync health for freshness.
- Otherwise predominantly **build new** — this is the licensing-specific action/accountability layer CoE does not provide.

### Implementation and component classification

| Component | Technology | Focus area | Classification |
| --- | --- | --- | --- |
| Candidates by classification | Dataverse chart | Optimization | Build New |
| Reclamation status funnel | Dataverse chart / MDA | Reclamation | Build New |
| Estimated reclaimable value by SKU/env | Dataverse chart / Power BI | Optimization/reclamation | Build New |
| Blocked reasons breakdown | Dataverse chart | Risk/safety | Build New |
| Evidence freshness distribution | Dataverse chart / Power BI | Risk/data quality | Build New |
| Audit activity over time | Embedded Power BI | Accountability | Build New |
| Protected-identity coverage tile | Dataverse chart / MDA | Risk | Build New |
| Unmatched correlation records | MDA list + Dataverse chart | Data quality | Build New |
| Interactive Safe-candidate triage list | MDA interactive dashboard | Reclamation | Build New |
| Drill-through CoE context (owner/usage) | MDA quick-view / Power BI | Dependency/usage | Reuse / Extend |
| Data & interface status panel | MDA dashboard | Risk/trust | Reuse + Build New |

---

## Reuse / Extend / Build-New Roll-Up

| Layer | Classification | Examples |
| --- | --- | --- |
| CoE usage, inventory, environment, sync health, governance dashboards | **Reuse** | Usage trends, environment facts, deep links, freshness |
| Licensing measures over reused CoE data | **Extend** | Premium utilization, distribution, dependency hotspots |
| Project-owned licensing analytics | **Build New** | Candidates, recommendations, reclamation pipeline, audit, protected identities, assignments |

The **data substrate is mostly reused/extended from CoE**; the **net-new analytics** are confined to the licensing optimization, recommendation, reclamation, and audit surfaces CoE lacks.

---

## Focus-Area Coverage Matrix

| Focus area | Executive | Platform Admins | License Governance Admins |
| --- | --- | --- | --- |
| License optimization | High (value) | High (targets) | High (candidates/value) |
| Risk analysis | Medium (posture) | High (group/stale/dependency) | High (blocked reasons/coverage) |
| Usage analytics | Medium (utilization) | High (patterns/trends) | Medium (evidence freshness) |
| Dependency analysis | Low | Medium (hotspots) | High (per-candidate, drill) |
| Reclamation opportunities | Medium (realized) | Medium (where to act) | High (pipeline/triage/audit) |

---

## Open Analytics Decisions (Cross-Reference)

- **Q-ANALYTICS-1** — precise definition of "inactive premium-licensed user" (drives inactivity visuals).
- **Q-COE-2** — CoE usage-history window length (bounds trend depth and inactivity confidence).
- **Q-VALUE-1** — definition of "realized savings" (drives executive value visuals).
- **Q-UX-2** — operational vs. executive visibility split (MVP-minimal operational view; executive dashboard fast-follow).

See `docs/open-questions.md` for context, owners, and gates.

---

## Constraints Honored

- No reports, Power BI assets, datasets, Dataverse charts, or model-driven dashboards were generated — design documentation only.
- Dashboards reuse and extend CoE analytics and avoid duplicating CoE's existing dashboards.
- No approval-workflow analytics and no automatic-removal actions are designed anywhere; estimates and realized value are kept distinct, and missing usage is never treated as zero.
