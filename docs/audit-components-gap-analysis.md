# Audit Components — Gap Analysis

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and How This Differs From the Assessment

This is a **gap analysis**, not an inventory. It is deliberately distinct from `docs/core-and-audit-components-assessment.md`:

- The **assessment** validates *what CoE Core + Audit actually provide* (tables, columns, views, apps, flows) and confirms reuse.
- This **gap analysis** judges *whether the deployed Audit Components are sufficient for the licensing project's usage and inactivity needs*, and where they are not, defines the **resolution** (Reuse As-Is / Extend / Supplement / Build New), owner, and whether it is blocking.

Scope: the **usage and inactivity evidence** the project depends on (capabilities C3 optimization analytics, C4 dependency safety, C5 recommendations, and C8 visibility). It assumes the environment has **Core Components and Audit Components (CenterOfExcellenceAuditComponents, including Audit Logs)** deployed. It generates **no** Power Platform assets.

### Resolution legend

- **Reuse As-Is** — the deployed Audit capability closes the need directly.
- **Extend** — reuse Audit data but add project-owned logic on top.
- **Supplement** — Audit does not provide it; acquire via a supported external interface (ADR-004) — never a parallel collection.
- **Build New** — reserved for the licensing layer, not usage collection.

### Gap status legend

- **Closed** — met by reuse · **Partial** — met with extension · **Open** — requires supplement/decision · **Conditional** — depends on a validation outcome.

---

## Why This Gap Analysis Matters

CoE Audit was designed for **resource-centric governance** (is an *app/flow* used?), while licensing optimization is **user-centric** (is a *licensed person* doing premium-requiring work?). The gaps below all stem from that difference. Closing them with reuse/extension wherever possible — and supplementing only where genuinely necessary — is what keeps the project charter-compliant and low-build.

---

## Gap Register

Each row: the project **Need**, what **Audit provides**, the **Gap**, **Status**, **Resolution**, **Owner**, and **Blocking?**

### G1 — Per-user app usage (who used which app)

- **Need (C3/C5):** determine whether a *specific licensed user* shows premium-requiring activity.
- **Audit provides:** app-launch telemetry from the Office 365 audit log; aggregates (unique users, last launched) onto the app record.
- **Gap:** whether **per-user** launch detail is **retained** in Dataverse (vs. aggregated away) is deployment-dependent.
- **Status:** **Conditional** (pivotal).
- **Resolution:** **Reuse As-Is** if per-user usage is retained; **Supplement** (supported usage/analytics interface, ADR-004) if only aggregates remain.
- **Owner:** Architect + CoE admin. **Blocking:** **Yes** — determines the entire C3 approach.

### G2 — Usage-history window vs. inactivity threshold

- **Need (C3):** evidence must span at least the chosen inactivity threshold (for example 60/90 days).
- **Audit provides:** usage history bounded by the Audit Logs retention/clean-up window.
- **Gap:** if the window is shorter than the threshold, inactivity cannot be evidenced confidently.
- **Status:** **Conditional** (Q-COE-2).
- **Resolution:** **Reuse As-Is** within the window, disclosing the bound; align the threshold to the window; **Supplement** only if a longer look-back is mandated.
- **Owner:** Product + CoE admin. **Blocking:** **Yes** for the inactivity definition.

### G3 — Per-user flow usage

- **Need (C3/C4):** activity signal for users whose premium entitlement is justified by **flows**, not apps.
- **Audit provides:** app-launch telemetry is primary; **flow run/usage coverage is typically weaker**.
- **Gap:** flow-owning users may lack sufficient usage evidence from Audit alone.
- **Status:** **Open/Conditional.**
- **Resolution:** **Reuse As-Is** if flow usage is captured; otherwise **Supplement** (supported flow analytics) or widen evidence (ownership + last-run) with disclosed lower confidence.
- **Owner:** Architect. **Blocking:** No (but affects flow-driven recommendations).

### G4 — Last-used / last-launched recency

- **Need (C3):** most-recent activity per owned resource.
- **Audit provides:** Last Launched (app) / Last Run (flow) signals.
- **Gap:** none material (coverage/freshness to confirm).
- **Status:** **Closed.**
- **Resolution:** **Reuse As-Is.**
- **Owner:** Analyst. **Blocking:** No.

### G5 — Unique users / adoption

- **Need (C3/C8):** corroborate whether an owned app is used at all; trend context.
- **Audit provides:** unique-user and adoption metrics.
- **Status:** **Closed.**
- **Resolution:** **Reuse As-Is** (consume) / **Extend** for license-centric trend visuals.
- **Owner:** Analyst. **Blocking:** No.

### G6 — Per-user, license-centric inactivity

- **Need (C3/C5):** classify a *licensed user* (not a resource) as inactive.
- **Audit provides:** resource-level inactivity/orphaned detection.
- **Gap:** no per-user, license-aware inactivity determination.
- **Status:** **Partial.**
- **Resolution:** **Extend** — project-owned analytic combining Audit usage + Core ownership + licensing, with evidence-coverage and freshness.
- **Owner:** Architect. **Blocking:** No (core project logic, expected build).

### G7 — Owner-to-usage correlation

- **Need (C2/C4):** join who owns a resource (Core) with whether it is used (Audit) and who is licensed (Graph).
- **Audit provides:** usage keyed to resources/users; Core provides ownership.
- **Gap:** the cross-source join is not a CoE capability.
- **Status:** **Partial.**
- **Resolution:** **Extend** — project correlation logic over reused Core + Audit via canonical IDs (no duplication).
- **Owner:** Architect. **Blocking:** No (expected build; depends on G1 keys).

### G8 — "Premium-requiring" activity signal

- **Need (C3/C5):** ideally know the activity actually *required* a premium license, not just that an app launched.
- **Audit provides:** app launches, not premium-feature-level detail.
- **Gap:** Audit cannot confirm premium-feature use directly.
- **Status:** **Open (heuristic).**
- **Resolution:** **Extend** — approximate using **premium-connector usage** (Core connector inventory) and app/flow premium classification as corroborating evidence; disclose as a heuristic with uncertainty; do **not** overstate. **Supplement** only if a stronger supported signal exists.
- **Owner:** Architect + Product. **Blocking:** No (affects confidence, not feasibility).

### G9 — Audit data freshness and sync health

- **Need (all):** recommendations require current evidence.
- **Audit provides:** scheduled collection; last-successful-sync and flow health.
- **Gap:** stale/failed syncs degrade evidence silently if unmonitored.
- **Status:** **Closed (operational).**
- **Resolution:** **Reuse As-Is** — surface freshness/sync health; block Safe on stale evidence (never treat missing as zero).
- **Owner:** CoE admin + project. **Blocking:** No (but enforce the freshness guardrail).

---

## Gap Summary

| Gap | Need | Status | Resolution | Blocking |
| --- | --- | --- | --- | --- |
| G1 Per-user app usage | C3/C5 | Conditional | Reuse / Supplement | **Yes** |
| G2 Usage-history window | C3 | Conditional | Reuse (bounded) / Supplement | **Yes** |
| G3 Per-user flow usage | C3/C4 | Open/Conditional | Reuse / Supplement | No |
| G4 Last-used recency | C3 | Closed | Reuse As-Is | No |
| G5 Unique users / adoption | C3/C8 | Closed | Reuse / Extend | No |
| G6 Per-user inactivity | C3/C5 | Partial | Extend | No |
| G7 Owner-to-usage correlation | C2/C4 | Partial | Extend | No |
| G8 Premium-activity signal | C3/C5 | Open (heuristic) | Extend / Supplement | No |
| G9 Freshness / sync health | All | Closed | Reuse As-Is | No |

**Reading:** most needs are met by **reuse or extension** of deployed Audit + Core data. Only **G1** and **G2** are **blocking**, and both are *validation/decision* gaps, not build gaps. No gap forces a parallel usage-collection build — consistent with the charter and `docs/data-model.md`.

---

## Blocking Gaps — Resolution Path (Before C3/C5 Build)

1. **G1 per-user usage granularity** — validate whether CoE Audit retains per-user launch detail (see assessment areas 11/18). If yes → Reuse. If no → record a scope decision to **Supplement** via a supported usage/analytics interface (ADR-004) or adjust the inactivity definition (Q-ANALYTICS-1).
2. **G2 usage-history window** — confirm the retention window length (Q-COE-2) and set the inactivity threshold within it, disclosing the bound. Supplement only if a longer mandated look-back cannot be met.

Until G1 and G2 are resolved, **no Safe reclamation recommendation should be treated as production-ready**.

---

## What This Does Not Change

- **No parallel usage collection** is built — Audit remains the usage source (reuse/extend); supplement is external and supported only.
- **No new inventory/ownership** — per `docs/data-model.md` "Tables We Explicitly Will Not Create".
- **Build New remains limited** to the licensing layer (acquisition, recommendation, reclamation, audit).

---

## Related Documents

- Validation of the actual artifacts: `docs/core-and-audit-components-assessment.md`.
- Reuse strategy and classifications: `docs/coe-reuse-analysis.md`, `docs/capability-map.md`.
- Schema and "will not create": `docs/data-model.md`.
- Open decisions (Q-COE-2 window, Q-ANALYTICS-1 inactivity definition): `docs/open-questions.md`.

---

## Constraints Honored

- Gap-analysis documentation only — no Power Platform assets, Dataverse schemas, or flows were generated; nothing in CoE is modified.
- Resolutions prefer reuse and extension; supplement is external and supported only; no gap justifies duplicating CoE usage collection.
