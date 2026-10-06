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

## Documentation

- [Project charter](docs/project-charter.md)
- [Vision](docs/vision.md)
- [Architecture](docs/architecture.md)
- [Requirements](docs/requirements.md)
- [Backlog](docs/backlog.md)
- [Guardrails](docs/guardrails.md)
- [Architecture decisions](docs/decisions.md)
- [Charter compliance review](docs/charter-compliance-review.md)