# Product Backlog

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## CoE-First Backlog Alignment

This backlog is governed by `docs/project-charter.md` and `docs/coe-reuse-analysis.md`. The customer runs the **CoE Core Components and Audit Components (CenterOfExcellenceAuditComponents)**, so CoE inventory, ownership, usage, dependency, and dashboard capabilities are available and must be reused or extended before anything is built.

Every Epic carries a **Reuse Strategy** section that classifies its work against three options:

- **Reuse Existing CoE Capability** — consume a CoE capability as-is or read it through a project-owned adapter (reuse preference levels 1–2).
- **Extend Existing CoE Capability** — add project-owned components that extend or surface CoE without modifying managed CoE assets (level 3).
- **Build New Capability** — build only where the capability is genuinely missing, supplementing via supported APIs where required (levels 4–5).

Reuse preference order: **reuse as-is → read via adapter → extend → supplement via supported API → build new.**

### Refactoring Notes

- **Epic 2 refactored** from "License Inventory" to "Licensing Intelligence Acquisition." The former title implied a parallel inventory, which would duplicate CoE (duplication risks D-1/D-5 in `docs/coe-reuse-analysis.md`). It is reframed as acquisition of licensing **entitlement** data — per-user premium assignment and SKU — that is genuinely absent from CoE and sourced via Microsoft Graph.
- **Epics 4 and 6 retitled** ("Optimization Analytics and Visibility", "Safe Reclamation and Audit") to make the reuse/extend boundary explicit; scope is unchanged.
- **No Epic was a pure CoE duplicate requiring removal.** Each Epic was evaluated against existing CoE functionality, and a per-Epic duplication guard is recorded. The only build-new work is the licensing layer CoE does not provide.

---

## Epic 1 — CoE Capability Discovery and Validation

**CoE-first classification:** Reuse (inspect/validate) — no new CoE-equivalent build.

### Story

Assess existing CoE capabilities and identify gaps relevant to licensing intelligence, analytics, dependency analysis, and remediation.

Success Criteria:

- Existing inventory, ownership, environment, maker, usage, flow, application, dashboard, and governance capabilities are documented.
- Reuse candidates and version/configuration dependencies are documented.
- Any proposed new capability has a documented gap justification.

### Reuse Strategy

- **Reuse Existing CoE Capability:** Inspect and baseline the CoE Core Components and Audit Components (inventory, usage, maker, ownership, environment, dashboard, and governance capabilities) as the authoritative source (`docs/coe-reuse-analysis.md` §A, §B.1).
- **Extend Existing CoE Capability:** None — this Epic is discovery only.
- **Build New Capability:** None in CoE terms. The only net-new outputs are project-owned documentation (the gap list) and the external **Microsoft Graph licensing** interface validation (ADR-004), which is an evaluation rather than a CoE build.

**Status note:** Largely satisfied by `docs/coe-reuse-analysis.md`; remaining work is confirming exact CoE artifact names/schemas during design and completing the Graph interface validation. **Duplication guard:** pure reuse — no duplication risk.

---

## Epic 2 — Licensing Intelligence Acquisition

**CoE-first classification:** Build New (supplement via supported API) — not inventory duplication.

> Refactored from "License Inventory" to remove the parallel-inventory connotation. This Epic acquires licensing **entitlement** data absent from CoE; it does **not** re-inventory apps, flows, makers, owners, or environments.

### Story

Evaluate supported sources for licensing information and define the minimum licensing intelligence missing from CoE.

Success Criteria:

- Interface support status, permissions, throttling, licensing, and tenant availability are validated.
- Direct and group-based assignment contexts are distinguishable where the supported source provides them.
- No CoE inventory is duplicated.

### Reuse Strategy

- **Reuse Existing CoE Capability:** Use CoE user/maker records as correlation anchors; do not re-collect any inventory CoE already maintains.
- **Extend Existing CoE Capability:** None directly — licensing entitlement is external to CoE.
- **Build New Capability:** Acquire per-user premium license assignment and SKU data via validated **Microsoft Graph** interfaces, distinguishing direct vs. group-based assignment (`docs/coe-reuse-analysis.md` §B.2). This is the project's core gap.

**Duplication guard:** Must not create a parallel inventory of apps/flows/makers/environments (risks D-1/D-5); acquires only licensing entitlement data missing from CoE.

---

## Epic 3 — Data Correlation

**CoE-first classification:** Extend (thin correlation logic over reused CoE data).

### Story

Correlate licensing information with existing CoE data through supported identifiers.

Success Criteria:

- Ownership, usage, and dependency evidence can be surfaced from CoE.
- Identifier gaps, stale data, and unmatched records are visible rather than silently ignored.

### Reuse Strategy

- **Reuse Existing CoE Capability:** Read CoE inventory, ownership, and usage through a project-owned adapter using stable, supported identifiers (`docs/coe-reuse-analysis.md` §B.3).
- **Extend Existing CoE Capability:** Add project-owned correlation/matching logic that joins licensing records to CoE records and surfaces unmatched or ambiguous records.
- **Build New Capability:** Only the correlation layer itself — no inventory.

**Duplication guard:** Reference and relate CoE data; do not copy CoE tables into a project store (risk D-5).

---

## Epic 4 — Optimization Analytics and Visibility

**CoE-first classification:** Extend (license-centric analytic + dashboard extension over reused CoE data).

### Story

Define explainable license optimization opportunities from validated licensing and CoE evidence.

Success Criteria:

- Optimization estimates disclose assumptions and data freshness.
- Existing CoE reporting is extended or reused before a new dashboard is considered.

### Reuse Strategy

- **Reuse Existing CoE Capability:** Reuse CoE Audit usage telemetry (installed and in use) and CoE resource-level inactivity signals; reuse existing CoE Power BI/admin dashboards as the visibility surface (`docs/coe-reuse-analysis.md` §A.6, §A.7, §B.4, §B.8).
- **Extend Existing CoE Capability:** Add a **license-centric** (per-user premium) inactivity analytic over reused usage, and extend CoE reporting with licensing/optimization visuals.
- **Build New Capability:** Only the license-centric analytic logic and any licensing-specific visual that cannot be surfaced by extending CoE reporting.

**Duplication guard:** Do not rebuild dashboards or re-collect usage (risks D-2/D-3). Estimates disclose assumptions and freshness; missing usage is never treated as zero usage.

---

## Epic 5 — Explainable Recommendations

**CoE-first classification:** Build New (consumes reused CoE + supplemented licensing inputs).

### Story

Classify remediation candidates using dependency, exception, assignment-source, and data-quality evidence.

Success Criteria:

- Safe, Review, and Blocked classifications are explainable.
- Missing or stale evidence cannot result in a Safe classification.

### Reuse Strategy

- **Reuse Existing CoE Capability:** Reuse CoE ownership, dependency, and usage evidence (via Epic 3 correlation) as recommendation inputs.
- **Extend Existing CoE Capability:** Interpret reused CoE dependency relationships with project-owned licensing-safety rules.
- **Build New Capability:** The Safe/Review/Blocked classification and explainability model (`docs/coe-reuse-analysis.md` §B.6); missing or stale evidence blocks a Safe classification.

**Duplication guard:** No CoE equivalent exists — low duplication risk; keep all logic in the project-owned solution.

---

## Epic 6 — Safe Reclamation and Audit

**CoE-first classification:** Build New (administrator-initiated reclamation via supported APIs + project-owned audit).

### Story

Define safe, administrator-initiated reclamation for eligible directly assigned licenses.

Success Criteria:

- Dry-run evidence and dependency checks are available before execution.
- Service, break-glass, shared, group-assigned, and critical-owner identities are protected.
- Outcomes and partial failures are auditable.
- No business approval workflow or automatic license removal is introduced.

### Reuse Strategy

- **Reuse Existing CoE Capability:** Consume CoE ownership/dependency evidence for pre-action checks and reuse protected-identity and exception inputs; reuse CoE-sourced evidence for explainability.
- **Extend Existing CoE Capability:** None against CoE-managed assets — CoE compliance/clean-up flows are never modified or repurposed (risk D-7).
- **Build New Capability:** Administrator-initiated reclamation with dry run, protected-identity and group-assignment exclusion, bounded bulk action, and a project-owned audit trail via supported APIs (`docs/coe-reuse-analysis.md` §B.7). No approval workflow; no automatic removal.

**Duplication guard:** Do not modify or repurpose CoE compliance flows; isolate all reclamation logic in the project-owned solution.