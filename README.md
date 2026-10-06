# Power Platform License Governance Accelerator

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Purpose

This repository contains architecture and governance guidance for extending an existing Microsoft Power Platform Center of Excellence (CoE) investment with license-focused capabilities.

The accelerator is limited to capabilities that are missing from the customer's CoE deployment:

- Licensing intelligence
- License optimization analytics
- Dependency analysis
- Explainable remediation recommendations
- Safe, auditable license reclamation

## Architectural Position

Existing CoE inventory, ownership, environment, application, flow, maker, usage, and governance capabilities must be discovered and reused before any extension is designed. This repository must not establish a parallel governance platform or unnecessary inventory system.

Only supported Microsoft interfaces may be considered. Availability, permissions, licensing terms, and support status must be validated before an interface is adopted.

The accelerator does not introduce business approval workflows. Any remediation capability must use explicit administrator initiation, safety checks, dry-run evidence, and an audit trail.

## Deployment Assumptions

- CoE **Core Components** and **Audit Components** (CenterOfExcellenceAuditComponents, including Audit Logs) are installed.
- **CoE is the system of record** for apps, flows, makers, environments, usage, and ownership.
- **Microsoft Graph** provides licensing entitlement data (premium assignment, SKU, direct vs. group).
- **No approval workflows**; **safe remediation only** (administrator-initiated, dry-run, dependency-checked, audited).
- **Custom solution components remain separate** from CoE (no managed-layer modifications).

## Reuse vs. Build

| Layer | Posture |
| --- | --- |
| CoE Core (inventory, ownership, environment, maker, connector) | **Reuse** (read-only) |
| CoE Audit (usage, last-launched, inactivity) | **Reuse** (read-only) |
| CoE Power BI governance dashboards | **Reuse / Extend** |
| Microsoft Graph licensing | **Supplement** |
| Correlation, analytics, dependency, visibility | **Extend** |
| Acquisition, recommendation, reclamation, audit, protected identities | **Build New** |

The data substrate is reused/extended from CoE; net-new build is confined to the licensing layer CoE does not provide. See [capability-map.md](docs/capability-map.md) for the full proof.

## Documentation

### Foundation and governance
- [Project charter](docs/project-charter.md)
- [Vision](docs/vision.md)
- [Requirements](docs/requirements.md)
- [Guardrails](docs/guardrails.md)
- [Architecture decisions (ADRs)](docs/decisions.md)
- [Charter compliance review](docs/charter-compliance-review.md)

### Product and UX
- [Product definition](docs/product-definition.md)
- [Personas](docs/personas.md)
- [User journeys](docs/user-journeys.md)
- [MVP definition](docs/mvp-definition.md)
- [Open questions](docs/open-questions.md)
- [Backlog](docs/backlog.md)

### Architecture and design
- [Architecture](docs/architecture.md)
- [Solution architecture](docs/solution-architecture.md)
- [App navigation (model-driven UX)](docs/app-navigation.md)
- [Dashboard design](docs/dashboard-design.md)
- [Data model](docs/data-model.md)
- [Capability map (reuse vs. build)](docs/capability-map.md)

### Design specifications (implementation-ready)
- [Interface and API specification](docs/interface-api-specification.md)
- [Security and identity design](docs/security-identity-design.md)
- [Recommendation rules specification](docs/recommendation-rules-specification.md)

### CoE reuse and assessment
- [CoE reuse analysis](docs/coe-reuse-analysis.md)
- [Core + Audit components assessment](docs/core-and-audit-components-assessment.md)
- [Audit components gap analysis](docs/audit-components-gap-analysis.md)
- [CoE Starter Kit reference](docs/reference/coe-toolkit/README.md)

### Delivery
- [Build plan](docs/build-plan.md)

## Repository Structure

```
.
├── docs/                     Architecture, product, UX, analytics, and CoE assessment docs
│   ├── prompts/              Architecture-review prompts (no implementation)
│   └── reference/coe-toolkit CoE Starter Kit reference material
├── solution/                 Project-owned solution scaffold (kept separate from CoE)
│   ├── LicenseGovernanceCore
│   ├── LicenseGovernanceAutomation
│   ├── LicenseGovernanceApp
│   └── LicenseGovernanceCoEAdapter
├── test/                     Test assets (planned)
└── .github/                  Repository-level agent instructions
```

## Status

Design and architecture phase. The documentation set is internally consistent and charter-aligned. An implementation-ready delivery plan is in [build-plan.md](docs/build-plan.md). No Power Platform assets have been generated yet.