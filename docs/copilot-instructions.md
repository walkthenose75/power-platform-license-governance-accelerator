# Repository Agent Guidance Prompt

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

Using `docs/project-charter.md`, recommend comprehensive repository-level guidance for coding agents.

The instructions should act as permanent repository-level guidance for GitHub Copilot and Claude.

The guidance should:

- Make `docs/project-charter.md` the governing document
- Require CoE-first architecture decisions
- Prohibit duplication of CoE Toolkit functionality
- Prohibit unsupported or undocumented APIs and require interface validation
- Prefer reuse before new development
- Favor dashboards and analytics over workflow approvals
- Prefer extending or surfacing existing CoE experiences; consider model-driven app patterns only for documented gaps
- Require explainable recommendations
- Require dependency analysis before remediation
- Require administrator-initiated, auditable, safe license reclamation guardrails
- Prohibit business approval workflows and unattended license removal
- Prohibit unnecessary copies of CoE inventory

The output should be optimized specifically for coding agents. This prompt defines architectural guidance only and must not generate Power Platform components, code, flows, Dataverse tables, dashboards, or implementation artifacts.