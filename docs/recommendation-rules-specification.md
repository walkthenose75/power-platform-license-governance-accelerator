# Recommendation Rules Specification

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Scope

This document specifies the **classification and inactivity rules** behind the accelerator's recommendations, implementation-ready and deterministic. It resolves the proposal for **Q-ANALYTICS-1** (inactivity definition).

- **WP1** uses a **simplified precursor** — a read-only "recent usage evidence?" flag — for opportunity sizing (no classification, no action).
- **WP2** implements the full **Safe / Review / Blocked** classification with explainability (C5).

It is **architecture documentation only** — no Power Platform assets, Dataverse schemas, or flows are generated. Outputs map to **Optimization Candidate** and **Recommendation Evidence** in `docs/data-model.md`.

---

## Explicit Assumptions

- Usage evidence comes from CoE Audit (per-user where available — gap G1); ownership/dependency from CoE Core; licensing/source from Graph.
- The **inactivity window** is **≤ the CoE usage-history window** (gap G2); it is configurable via environment variables.
- Protected-identity and exception data (Q-SAFETY-1) and criticality (Q-COE-3) are available or configured.
- Estimates are **directional**, never guaranteed.

---

## Definitions

- **In-scope premium-licensed user:** holds an in-scope premium SKU (SKU Reference.Is Premium).
- **Usage window (N):** rolling look-back (default proposal **90 days**, configurable, must be ≤ CoE usage window).
- **Recent usage evidence (WP1 precursor):** any usage signal (app launch / last-launched / flow run / premium-connector use) on the user's owned resources within N.
- **Inactivity (WP2):** **no** premium-requiring activity evidence within N **and** evidence coverage ≥ the minimum threshold.
- **Evidence coverage:** the degree to which the user's owned premium-relevant resources have usable, fresh usage telemetry within N (for example, a minimum fraction and a maximum staleness). Low coverage means the data cannot support a Safe decision.

## Evidence Model

Each contributing item (→ Recommendation Evidence) carries: **type** (usage / assignment source / exception / dependency / freshness), **source**, **as-of/freshness**, **confidence**, and **influence** (supports Safe / Review / Blocked). The classification is explainable as the ordered set of items that fired.

## Classification Rules (Deterministic, Ordered)

Rules are evaluated **in order**; the first matching outcome wins. This guarantees determinism and explainability.

1. **Active → not a candidate.** If recent premium-requiring usage exists within N, the user is **Active** and excluded from reclamation (shown as Active, not a candidate).
2. **Blocked — protected identity.** If the user is a protected identity (service / break-glass / shared / critical owner) → **Blocked**.
3. **Blocked — group-assigned.** If the in-scope license is **group-assigned** (`assignedByGroup` populated) → **Blocked** (not directly reclaimable; show the controlling group).
4. **Blocked — insufficient/stale evidence.** If evidence coverage < threshold or evidence is stale beyond the freshness limit → **Blocked** (cannot assert Safe). *Missing usage is never treated as zero usage.*
5. **Blocked/Review — critical dependency.** If the user is the owner of a business-critical app/flow → **Blocked**; if a non-critical but material dependency exists → **Review**.
6. **Review — uncertainty/exceptions.** If correlation is ambiguous/unmatched, an exception requires human judgment, or confidence is moderate → **Review**.
7. **Safe.** Otherwise — **directly assigned**, not protected, sufficient fresh evidence of inactivity, no critical/material dependency, correlation matched → **Safe**.

**Precedence summary:** Active > Protected > Group-assigned > Insufficient/stale evidence > Critical dependency > Ambiguity/exception > Safe.

## Safety Invariants (Non-Negotiable)

- Missing or stale evidence **cannot** yield Safe (Rule 4).
- Group-assigned **cannot** be Safe or actionable (Rule 3).
- Protected identities are **never** Safe/actionable (Rule 2).
- No classification triggers any automatic action; reclamation is always a separate, administrator-initiated, dry-run-first step (WP3).

## Thresholds (Configurable via Environment Variables)

| Parameter | Proposed default | Notes |
| --- | --- | --- |
| Usage window N | 90 days | Must be ≤ CoE usage window (G2) |
| Evidence coverage minimum | e.g., owned premium resources have usage telemetry within N | Below → Blocked (Rule 4) |
| Freshness limit | e.g., evidence as-of ≤ 7 days old | Beyond → treat as stale |
| Criticality source | CoE criticality if populated, else project designation | Q-COE-3 |

All thresholds are disclosed in the explanation and are tenant-configurable; defaults are proposals pending Q-ANALYTICS-1 ratification.

## Explainability Output

For each candidate: the **outcome**, the **rule that fired**, the **evidence list** (with freshness/confidence), **assignment source**, **applied exceptions**, **dependencies**, **uncertainty**, and **estimated value** — sufficient for an administrator to independently justify the decision.

## Estimate Model

- **Estimated opportunity** = count of Safe (and, separately, Review) candidates × SKU unit cost basis (SKU Reference), with assumptions and freshness disclosed.
- **Estimated vs. realized** are kept distinct; realized value comes only from audited reclamations (WP3).

## WP1 vs. WP2 Applicability

- **WP1:** compute only the **recent usage evidence?** boolean and the opportunity size ("no recent usage evidence" count × unit cost). No Safe/Review/Blocked, no reclamation.
- **WP2:** implement Rules 1–7, the evidence model, thresholds, and explainability in full.

## Dependencies

- G1 (per-user usage granularity) and G2 (usage window) from `docs/audit-components-gap-analysis.md`.
- Q-ANALYTICS-1 (ratify this definition), Q-COE-2 (window), Q-COE-3 (criticality), Q-SAFETY-1 (protected source).
- `docs/interface-api-specification.md` (evidence sources), `docs/data-model.md` (output tables).

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Inactivity definition disputed | Rework / low trust | Ratify via Q-ANALYTICS-1; parameterize; disclose |
| Per-user usage missing (G1) | Weak evidence coverage | Coverage threshold blocks Safe; supplement or widen signals with disclosed lower confidence |
| Window shorter than threshold (G2) | Unsafe inactivity calls | Clamp N ≤ window; disclose the bound |
| Criticality not populated | Weak dependency safety | Project-owned criticality designation; default material dependencies to Review |
| Estimate misread as guaranteed | Credibility | Separate estimated vs. realized; disclose assumptions |

## Acceptance Criteria (supports WP1/WP2)

- **WP1:** the precursor flag and opportunity estimate are produced with freshness and assumptions disclosed; no classification or action is present.
- **WP2:** Rules 1–7 produce deterministic, explainable Safe/Review/Blocked; missing/stale evidence never yields Safe; group-assigned and protected are never Safe; every outcome exposes its evidence, rule, source, exceptions, dependencies, and uncertainty.

## Constraints Honored

- Documentation only; no assets generated. Deterministic, explainable, safety-first rules; no automatic action and no approval workflow; estimates are directional and separated from realized value.
