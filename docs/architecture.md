# Architecture

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Architectural Scope

The architecture is an extension of the customer's existing CoE deployment, not a standalone governance platform. CoE remains authoritative for existing inventory, ownership, environment, application, flow, maker, usage, and governance capabilities.

The accelerator may add only capabilities that a documented discovery confirms are missing:

- Licensing intelligence
- License optimization analytics
- Cross-source dependency analysis
- Explainable remediation recommendations
- Safe and auditable license reclamation

## Reuse-First Boundaries

- Surface or reference existing CoE data rather than copying it into a parallel inventory.
- Correlate licensing information with CoE records using stable, supported identifiers.
- Extend existing CoE dashboards and operational experiences when practical; create a separate experience only when the requirement cannot be met through extension.
- Treat CoE schema and solution changes as external dependencies and isolate version-specific assumptions.
- Document each retained data element that is not already available from CoE and justify its ownership and lifecycle.

## Integration Constraints

Candidate interfaces include Microsoft Graph, supported Power Platform APIs, supported Power Platform admin connectors, and existing CoE data. No interface is approved solely by being listed here. Support status, permissions, licensing, throttling, data availability, and tenant compatibility must be validated before adoption.

Undocumented endpoints, direct manipulation of CoE internals, and assumptions about optional CoE components are prohibited.

## Remediation Boundary

The architecture must not introduce a business approval engine. Remediation is an explicitly initiated administrative operation with:

- Dependency and exception checks
- Clear explanation of recommendation evidence
- Dry-run results
- Protection for service, break-glass, shared, group-assigned, and critical-owner identities
- Auditable outcomes and actionable failures

Automatic or unattended license removal is outside the approved scope unless the project charter and guardrails are formally revised.

## Architectural Risks

- CoE version or configuration differences can invalidate assumed schemas and relationships.
- Licensing data can be incomplete or delayed, producing unsafe recommendations.
- Group-based assignment can make direct license removal ineffective or misleading.
- Identifier mismatches can create false dependency conclusions.
- Broad permissions can increase security and operational risk.
- Duplicated data can drift from CoE and create competing sources of truth.