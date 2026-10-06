# Repository Instructions for Coding Agents

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Governing Authority

`docs/project-charter.md` is the governing document for this repository. Read it before proposing or implementing any architecture, requirement, schema, automation, application, dashboard, integration, or remediation change.

When another document, backlog item, prompt, or implementation detail conflicts with the project charter:

1. Follow the project charter.
2. Stop before implementing the conflicting requirement.
3. Identify the conflict explicitly.
4. Recommend the smallest charter-compliant correction.
5. Request a documented architecture decision if the conflict cannot be resolved without changing project scope.

Do not reinterpret an ambiguous requirement in a way that creates a new governance platform.

## North Star

This repository extends the customer's existing Microsoft Power Platform Center of Excellence (CoE) investment. It does not replace CoE or create a parallel governance system.

The accelerator is limited to capabilities that are demonstrably missing from the customer's CoE deployment:

- Licensing intelligence
- License optimization analytics
- Cross-source dependency analysis
- Explainable license recommendations
- Safe and auditable license reclamation

Follow this order for every capability:

1. Reuse an existing CoE capability.
2. Surface existing CoE data or functionality.
3. Extend an existing CoE experience or process.
4. Build only the smallest missing capability.

## Mandatory CoE-First Decision Gate

Before proposing or implementing new functionality:

1. Identify the relevant capability already available in the customer's CoE deployment.
2. Determine whether it can be reused, surfaced, configured, or extended.
3. Document the specific gap that CoE cannot satisfy.
4. Confirm that the proposed change addresses only that gap.
5. Record dependencies on CoE version, configuration, schema, solutions, flows, dashboards, and optional components.

Do not proceed when CoE discovery is incomplete and the change could duplicate existing functionality.

## Prohibited Duplication

Do not create a parallel source of truth for data already owned by CoE, including:

- Applications
- Cloud and desktop flows
- Makers
- Owners
- Environments
- Connectors
- Usage and activity
- Governance status
- Existing CoE operational or inventory data

Prefer references, supported relationships, queries, views, and extensions over copying CoE data.

When a retained copy is technically unavoidable, require a documented justification covering:

- The CoE capability gap
- Authoritative source
- Data ownership
- Synchronization and reconciliation
- Freshness
- Retention and deletion
- Failure handling
- CoE version compatibility

Never modify undocumented CoE internals or assume that optional CoE components are installed.

## Supported Interfaces Only

Use only documented, supported Microsoft interfaces. Candidate sources can include Microsoft Graph, supported Power Platform APIs, supported Power Platform admin connectors, and existing CoE data, but naming a source does not approve a particular operation.

Before adopting an interface, verify:

- The exact operation is documented and supported.
- The operation is available in the target tenant and cloud.
- Required permissions follow least privilege.
- Licensing and capacity requirements are understood.
- Throttling, pagination, batching, retries, and service limits are handled.
- Authentication is appropriate for interactive and unattended use.
- Error and partial-failure behavior is explicit.
- Preview or beta status is disclosed and approved.

Do not:

- Use undocumented endpoints.
- Scrape administrative portals.
- Manipulate CoE or platform databases directly.
- Assume beta or preview APIs are production-supported.
- Hide integration failures behind empty or success-shaped results.
- Hard-code tenant, environment, solution, connection, user, group, or SKU identifiers.

If a supported interface cannot be confirmed, stop and record the assumption as an architectural risk rather than fabricating an integration.

## Architecture Preferences

### Reuse Before Development

Prefer configuration, composition, and extension over replacement. New components must have a clear boundary, a documented owner, and a requirement that cannot be met by CoE.

Keep coupling to CoE explicit and isolated. Account for differences in deployed CoE versions and enabled components.

### Analytics Before Workflow

Favor visibility, dashboards, analytics, dependency insights, and explainable recommendations over workflow orchestration.

Do not introduce a business approval engine. Approval workflows are outside the repository's scope unless the project charter is formally revised.

Use authorization, explicit administrator initiation, dry-run evidence, safety checks, and auditing to govern remediation. Do not disguise an approval workflow as a status machine, task queue, or notification process.

### Model-Driven Application Patterns

First determine whether an existing CoE experience can surface or host the requirement.

When a documented gap justifies a separate application experience, prefer supported model-driven app patterns:

- Solution-aware components
- Dataverse security roles and least privilege
- Views and forms appropriate to administrator workflows
- Accessible, responsive experiences
- Environment variables and connection references
- Auditable commands with explicit confirmation
- Standard platform capabilities before custom code

Do not build a separate model-driven app merely to reproduce CoE inventory, administration, dashboards, or governance experiences.

### Data Architecture

CoE remains authoritative for its existing domains. Add storage only for accelerator-owned information that CoE does not provide.

For every new data element, document:

- Business purpose
- Authoritative source
- Ownership
- Correlation identifier
- Freshness expectation
- Retention
- Sensitivity
- Audit requirement
- Reconciliation behavior

Do not make Dataverse an unconditional replication layer for CoE data.

## Explainable Recommendations

Every recommendation must be understandable and independently reviewable by an administrator.

For each recommendation, expose:

- Recommendation category, such as Safe, Review, or Blocked
- Source evidence
- Evidence timestamp and freshness
- Relevant application, flow, ownership, and usage dependencies
- License assignment source
- Applied exclusions and exceptions
- Rule or rationale that produced the result
- Known uncertainty and missing evidence
- Estimated impact
- Permitted next action

Missing, stale, unmatched, or contradictory evidence must not produce a Safe recommendation. Do not silently default missing information to inactivity or no dependency.

Optimization estimates must distinguish potential savings from realized savings and disclose assumptions.

## Dependency Analysis Before Remediation

Never recommend or perform license reclamation based only on sign-in or usage inactivity.

Before a candidate can be considered safe, evaluate available supported evidence for:

- Application ownership
- Flow ownership
- Critical or business-essential assets
- Shared ownership and continuity
- Maker and administrator responsibilities
- Environment roles and operational duties
- Service, automation, shared, emergency, and break-glass identities
- Direct versus group-based license assignment
- Other documented customer exceptions
- Data completeness and freshness

Unresolved dependencies must result in Review or Blocked, not Safe.

Surface unmatched identities and failed dependency checks explicitly. Never suppress them to complete a batch.

## Safe License Reclamation Guardrails

License reclamation must be:

- Explicitly initiated by an authorized administrator
- Limited to eligible directly assigned licenses
- Preceded by a dry run
- Protected by dependency and exception checks
- Explainable before confirmation
- Idempotent where the supported interface permits
- Audited with actor, target, evidence, action, timestamp, and outcome
- Able to report partial failures without claiming complete success

Never automatically remove licenses from:

- Service or automation accounts
- Break-glass or emergency accounts
- Shared accounts
- Group-assigned users
- Critical application owners
- Critical flow owners
- Identities with unresolved dependencies
- Identities with stale or incomplete evidence
- Customer-defined protected populations

Do not attempt to remediate group-assigned licensing by directly removing a user license. Identify and report the controlling assignment source.

Automatic or unattended license removal is prohibited unless the project charter and guardrails are formally revised.

## Security and Operational Requirements

- Apply least privilege to data acquisition and remediation.
- Separate read-only analytics responsibilities from remediation permissions where practical.
- Never commit credentials, tokens, secrets, tenant-specific identifiers, or personal data.
- Use environment variables and connection references for environment-specific configuration.
- Preserve DLP, tenant, environment, and customer security policies.
- Do not weaken a governance control to make a component work.
- Make failures visible, actionable, and auditable.
- Design for pagination, throttling, retries, idempotency, and partial success where applicable.
- Minimize stored personal and licensing data.
- Treat bulk remediation and privileged operations as high-impact.

## ALM and Solution Guidance

- Keep justified components solution-aware and deployable through supported ALM practices.
- Reuse existing customer solution and pipeline conventions where they are compatible with the charter.
- Avoid unmanaged production changes.
- Use environment variables, connection references, and documented configuration.
- Keep CoE adapters isolated from licensing-domain logic to reduce version coupling.
- Do not assume solution names, publisher prefixes, environment IDs, connection IDs, or CoE schema details.
- Document prerequisites, rollback, validation, and operational ownership for high-impact changes.

## Coding Agent Workflow

For every implementation request:

1. Read `docs/project-charter.md` and relevant repository documentation.
2. Restate the charter-compliant objective internally.
3. Search for existing CoE and repository capabilities before adding new ones.
4. Identify the exact missing capability and its boundary.
5. Validate proposed interfaces and platform assumptions.
6. Assess duplication, dependency, security, ALM, and remediation risks.
7. Implement the smallest complete change that satisfies the documented gap.
8. Add or update tests and directly related documentation.
9. Verify that existing CoE behavior and repository behavior are preserved.
10. Report assumptions, unresolved risks, validation performed, and CoE capabilities reused.

Stop and request clarification when:

- The target CoE version or enabled capabilities materially affect the design.
- A requirement appears to duplicate CoE.
- A supported interface cannot be verified.
- Remediation behavior is ambiguous.
- An approval workflow is requested.
- Required dependency evidence is unavailable.
- The change would create a new source of truth or governance platform.

## Review Checklist

Before considering work complete, confirm:

- [ ] The change complies with `docs/project-charter.md`.
- [ ] Existing CoE capabilities were evaluated first.
- [ ] No CoE functionality or inventory was unnecessarily duplicated.
- [ ] The implementation addresses a documented gap.
- [ ] Every interface is documented and supported for the intended operation.
- [ ] Analytics or reporting reuse was preferred over new workflow.
- [ ] A model-driven app component, if any, is justified by a gap and does not reproduce CoE.
- [ ] Recommendations are explainable and disclose freshness and uncertainty.
- [ ] Dependency analysis precedes any remediation.
- [ ] Group-assigned and protected identities cannot be directly remediated.
- [ ] Dry-run, authorization, auditing, and partial-failure behavior are explicit.
- [ ] No approval engine or unattended license removal was introduced.
- [ ] Security, ALM, testing, documentation, and rollback expectations are satisfied.

## Required Response Style

When proposing or completing work, state:

- Which CoE capabilities are reused or surfaced
- Which documented gap is addressed
- Why new development is necessary
- Which supported interfaces are used
- What assumptions remain
- What dependency and safety checks apply
- What validation was performed

Do not describe duplicated CoE functionality as an accelerator feature. Do not present unsupported assumptions as facts. Prefer concise, actionable architecture and implementation guidance grounded in the project charter.
