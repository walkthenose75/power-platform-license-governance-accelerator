# Data Handling and Privacy Note

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Controller Note

This note provides **transparency** about the personal data the accelerator processes and prompts the customer to perform their **own privacy review**. It is **not** a completed Data Protection Impact Assessment (DPIA).

> **The customer is the data controller.** The accelerator is delivered as an unmanaged solution that runs in the customer's own tenant on the customer's own data. The customer is responsible for the lawful basis, any DPIA, notices/consultations, and retention decisions required by their jurisdiction and policies. This note helps that assessment; it does not replace it.

It is **documentation only** — no personal data is processed by this document.

## What a DPIA Is (Context)

A **Data Protection Impact Assessment** documents what personal data a system processes, why, the risks to the individuals, and the mitigations. It is commonly expected (for example under GDPR-style regimes) when a system processes personal data in ways that could affect people — such as monitoring activity or making decisions about individuals. This accelerator touches both (usage activity and license-reclamation recommendations), so a customer privacy review is appropriate before enabling reclamation.

## Personal Data Processed

| Data element | Source | Purpose | Stored where | Sensitivity | Retention |
| --- | --- | --- | --- | --- | --- |
| User object ID | Graph / CoE | Correlation anchor | Project tables | Identifier | Rolling window |
| UPN / display name | Graph / CoE | Human-readable context | Project tables (field-secured) | **PII** | Rolling window; minimize |
| Premium license assignment / SKU / source | Graph | Licensing intelligence | Project tables | Identifier + entitlement | Rolling window |
| Usage activity (app launches, last used) per user | CoE Audit | Inactivity evidence | Read from CoE; evidence snapshot | **Behavioral / PII** | Bounded by CoE window |
| Ownership (apps/flows) | CoE Core | Dependency safety | Read from CoE | Identifier | Read-time |
| Recommendations and evidence | Derived | Explainable decisions about a person | Project tables | Decision data | With candidate |
| Reclamation actions / audit | Project | Accountability | Audit Log (immutable) | Action about a person | Compliance retention |

## Automated Decision-Making

- Recommendations are **decision support for a human administrator**, not automated actions.
- **Reclamation is always administrator-initiated**, preceded by a dry run and safety checks — there is **no solely-automated decision** that removes a person's license. This design choice reduces privacy risk (relevant to GDPR Article 22 considerations) and keeps a human in the loop.

## Data Minimization and Protection

- Prefer **object IDs**; store human-readable PII (UPN, names) only where needed for usability, protected by **field-level security**.
- **Immutable audit**; least-privilege identities; read-only access to CoE; no modification of CoE data.
- **Missing usage is never treated as zero usage**, and stale/insufficient evidence cannot drive a Safe decision — reducing the risk of unfair decisions about an individual.
- Retention follows `docs/data-model.md`; operational data is purged on a rolling window; audit is retained per policy.

## Data Residency and Boundaries

- Data remains within the customer's **Microsoft 365 / Power Platform tenant** and the Microsoft Graph it already uses; confirm the correct cloud/base URLs for sovereign clouds.
- No personal data is sent to any third party by the accelerator.

## Customer Responsibilities (Checklist)

- `[ ]` Determine the **lawful basis** and purpose for processing usage and licensing data about individuals.
- `[ ]` Complete a **DPIA / privacy review** per your jurisdiction and policies before enabling reclamation.
- `[ ]` Provide any required **notices** to, or consultations with, affected individuals/works councils.
- `[ ]` Configure **retention** and **data-minimization** settings to your policy.
- `[ ]` Maintain the **protected-identity** list and least-privilege access.
- `[ ]` Confirm **data residency / cloud** requirements.

## Assumptions

- The customer operates the solution in their tenant and is the data controller.
- Field-level security, least-privilege identities, and retention are configured per the security and data-model designs.

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Processing without a lawful basis/DPIA | Compliance exposure | Customer completes privacy review before reclamation (this note prompts it) |
| Excess PII stored | Privacy exposure | Store object IDs; field-secure names; minimize and purge |
| Unfair decision from weak evidence | Harm to individual | Stale/missing evidence cannot be Safe; human-in-the-loop; explainability |
| Over-broad access to personal data | Privacy exposure | Least privilege; FLS; role separation |

## Dependencies

- `docs/data-model.md` (retention, FLS), `docs/security-identity-design.md` (identities, least privilege), `docs/recommendation-rules-specification.md` (evidence/safety).

## Acceptance Criteria

- The personal data processed is transparently documented; the human-in-the-loop and minimization choices are stated; and the customer has a clear checklist to complete their own privacy review before enabling reclamation.

## Constraints Honored

- Documentation only; no personal data processed here. The accelerator minimizes personal data, keeps a human in the loop, performs no automatic removal, and does not modify CoE data; the customer remains the data controller.
