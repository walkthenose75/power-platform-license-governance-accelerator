# Personas

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## How to Read This Document

Personas describe *who* the accelerator serves and *why*, not how any feature is implemented. Because the product is a CoE extension, most personas are already CoE stakeholders; the accelerator adds a licensing lens to roles they already hold. Personas are grouped as **primary** (direct, regular users), **secondary** (periodic or oversight users), and **affected stakeholders** (not users, but impacted by decisions).

---

## Primary Personas

### P1 — Platform / CoE Administrator ("Priya")

- **Role:** Owns the Power Platform CoE and day-to-day platform governance.
- **Context:** Already uses CoE dashboards for apps, flows, makers, owners, and environments.
- **Goals:**
  - See which premium licenses are unused with trustworthy evidence.
  - Reclaim licenses safely without breaking a critical app or flow.
  - Reduce the time spent on manual license audits.
- **Pain points:**
  - Licensing data and CoE inventory live in separate places and are correlated by hand.
  - Fear of removing a license from a service, shared, or break-glass account.
  - No single explainable view of "why is this user a candidate?"
- **Needs from the product:**
  - Correlated, explainable Safe / Review / Blocked recommendations.
  - Dry run and dependency checks before any action.
  - Reuse of the CoE experiences they already know.
- **Success looks like:** Confident, auditable reclamation in minutes, grounded in CoE data.
- **Charter sensitivities:** Will reject a parallel tool that duplicates CoE; expects extension, not replacement.

### P2 — Licensing / Procurement Analyst ("Marco")

- **Role:** Manages Power Platform license entitlements, renewals, and true-ups.
- **Context:** Accountable for how many premium licenses are purchased and whether they are justified.
- **Goals:**
  - Quantify how many premium licenses are genuinely in use.
  - Defer or reduce purchases using evidence of reclaimable capacity.
  - Produce defensible numbers for renewal negotiations.
- **Pain points:**
  - "Assigned" is not the same as "used," and the gap is hard to prove.
  - Group-based assignments obscure who is really consuming a license.
- **Needs from the product:**
  - Optimization estimates with disclosed assumptions and freshness.
  - Clear distinction between direct and group-based assignment.
  - Separation of estimated vs. realized savings.
- **Success looks like:** Evidence-based license planning rather than guesswork.
- **Charter sensitivities:** Needs estimates labeled as directional, never guaranteed.

### P3 — IT Finance / FinOps Lead ("Dana")

- **Role:** Drives cost optimization across cloud and SaaS, including Power Platform.
- **Context:** Sponsors the optimization effort and reports savings upward.
- **Goals:**
  - Turn license waste into quantified, trackable savings opportunities.
  - Track realized value over time.
- **Pain points:**
  - Optimization claims that cannot be substantiated or audited.
  - No visibility into whether identified savings are actually realized.
- **Needs from the product:**
  - Executive-level visibility with assumptions disclosed.
  - A clear line between potential and realized savings.
- **Success looks like:** A credible, repeatable savings narrative.
- **Charter sensitivities:** Will not accept numbers presented as guaranteed savings.

---

## Secondary Personas

### P4 — Environment / Business Unit Administrator ("Sam")

- **Role:** Delegated admin for specific environments or a business unit.
- **Context:** Knows local app/flow criticality that a central admin may not.
- **Goals:**
  - Ensure reclamation does not disrupt locally critical workloads.
  - Flag local exceptions and protected owners.
- **Needs from the product:**
  - Dependency evidence scoped to their environment.
  - A way to see why a local user is Safe, Review, or Blocked.
- **Charter sensitivities:** Expects to consume CoE-sourced data, not a separate inventory.

### P5 — Security & Compliance Officer ("Alex")

- **Role:** Owns identity, security, and audit posture (CISO office / compliance).
- **Context:** Cares about protected identities, least privilege, and auditability.
- **Goals:**
  - Guarantee service, break-glass, and shared accounts are never auto-reclaimed.
  - Ensure every action is least-privilege and auditable.
- **Needs from the product:**
  - Enforced protected-identity rules that cannot be silently bypassed.
  - Complete audit trail; no secrets or personal data exposure.
  - Confidence that no governance control is weakened to enable a feature.
- **Charter sensitivities:** Hard stop on automatic removal and on unsupported interfaces.

### P6 — Executive Sponsor ("Jordan")

- **Role:** CIO / Head of Platform or equivalent budget owner.
- **Context:** Approves the investment and wants outcomes, not operational detail.
- **Goals:**
  - See premium utilization and optimization value at a glance.
  - Confirm the initiative reuses existing CoE investment.
- **Needs from the product:**
  - Executive visibility that reuses or extends existing CoE reporting.
  - Evidence that spend is controlled and value is realized.
- **Charter sensitivities:** Expects no new platform and minimal net-new build.

---

## Affected Stakeholders (Not Direct Users)

### P7 — Maker / App or Flow Owner ("Riley")

- **Role:** Builds and owns apps/flows that may depend on a premium license.
- **Why they matter:** Reclaiming a license from a critical owner can break production workloads. Their ownership and dependency data (sourced from CoE) is central to classification.
- **Product impact:** Dependency analysis must protect them; they are the reason Blocked and Review classifications exist.

### P8 — Service Desk / Operations ("Taylor")

- **Role:** Handles exceptions, break-glass access, and incident response.
- **Why they matter:** Owns or operates service/shared/break-glass accounts that must be protected from reclamation.
- **Product impact:** Their accounts must be reliably identified as protected and excluded.

---

## What CoE Already Provides vs. The Gap This Product Fills

The customer operates the CoE Core Components and Audit Components (CenterOfExcellenceAuditComponents), so every persona already receives inventory, ownership, usage, and governance dashboards from CoE. This product does **not** re-serve those needs; it adds the **licensing layer** on top. Capability IDs below reference the Capability Catalog (C1–C8) in `docs/product-definition.md`.

| Persona | Already served by CoE | Gap this product fills (why CoE alone is insufficient) |
| --- | --- | --- |
| P1 Platform/CoE Admin | App/flow/maker inventory, usage, governance dashboards | Correlated, explainable, safe license reclamation CoE cannot produce (C2–C7) |
| P2 Licensing/Procurement | Nothing — CoE holds no license entitlement data | Premium entitlement visibility and optimization estimates (C1, C3) |
| P3 IT Finance/FinOps | High-level adoption reporting | Quantified, estimate-vs-realized savings visibility (C3, C8) |
| P4 Environment/BU Admin | Local app/flow inventory and ownership | Scoped dependency and safety evidence for licensing decisions (C2, C4) |
| P5 Security & Compliance | Governance posture and CoE audit | Protected-identity enforcement and reclamation audit (C4, C6, C7) |
| P6 Executive Sponsor | Estate dashboards | License optimization and realized-value narrative (C8) |

---

## Persona-to-Capability Map

| Persona | Primary Capability Used | Charter Guardrail Most Relevant |
| --- | --- | --- |
| P1 Platform/CoE Admin | Recommendations (C5), safe reclamation (C6), dependency analysis (C4) | Safety first; explainability; reuse CoE |
| P2 Licensing/Procurement | Optimization analytics (C3), assignment-context insight (C1) | Estimates not guaranteed; direct vs. group |
| P3 IT Finance/FinOps | Executive visibility and savings tracking (C8) | Estimated vs. realized separation |
| P4 Environment/BU Admin | Scoped dependency evidence (C2, C4) | Reuse CoE; no parallel inventory |
| P5 Security & Compliance | Protected identities (C4), audit (C7) | No auto-removal; least privilege; supported APIs |
| P6 Executive Sponsor | Executive dashboards (C8) | No new platform; reuse CoE investment |
| P7 Maker/Owner (affected) | (Protected by) dependency analysis (C4) | Safety first |
| P8 Service Desk/Ops (affected) | (Protected by) protected-identity rules (C4, C6) | Never auto-remove protected accounts |

---

## Anti-Personas (Explicitly Not Served)

- **Approval/sign-off coordinator:** The product is not a business approval engine; no persona is served by building approval workflows.
- **Unattended automation operator:** No persona is served by automatic or scheduled license removal.
- **Net-new governance platform owner:** No persona is served by replacing or duplicating CoE.
