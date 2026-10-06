# Requirements

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Scope Preconditions

- Complete discovery of the customer's deployed CoE capabilities before defining new storage, automation, applications, or dashboards.
- Document each capability gap that cannot be satisfied by surfacing or extending CoE.
- Validate support status, permissions, licensing, throttling, and tenant availability for every proposed interface.

## Functional

### Licensing

- Acquire assigned Power Apps licensing information not already available through CoE.
- Acquire required SKU information through validated, supported interfaces.
- Distinguish direct, group-based, service, shared, and exception-account assignment contexts where supported.

### CoE Integration

- Surface existing CoE application and flow inventory without creating a parallel inventory.
- Reuse existing CoE maker, owner, environment, and usage information.
- Correlate licensing records to CoE data through supported, documented identifiers.
- Remain resilient to documented CoE version and configuration differences.

### Analytics

- Identify potentially inactive licensed users from explainable evidence.
- Analyze application, flow, ownership, and other relevant dependencies before recommending reclamation.
- Estimate optimization opportunities without presenting estimates as guaranteed savings.

### Recommendations

- Classify candidates as Safe, Review, or Blocked using documented rules.
- Explain the evidence, uncertainty, dependencies, and exceptions behind every classification.
- Prevent a Safe classification when required evidence is missing or stale.

### Remediation

- Support administrator-initiated reclamation of eligible directly assigned licenses only after dry-run and safety checks.
- Exclude group-assigned licenses from direct removal and explain the controlling assignment source.
- Support bounded bulk remediation without introducing a business approval workflow.
- Record auditable outcomes and expose partial failures explicitly.

---

## Non-Functional

- Solution-aware and ALM-friendly where an extension component is justified.
- Extensible without coupling to undocumented CoE internals.
- Auditable and explainable.
- Least-privilege and compliant with tenant governance controls.
- Dataverse storage is permitted only for documented accelerator-owned data that is not already available from CoE; it is not a requirement to duplicate CoE inventory.
- No unsupported or undocumented APIs.
- No automatic license removal and no new approval engine.