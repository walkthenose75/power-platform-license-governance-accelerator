# CoE Starter Kit Reference Sources

## Purpose

This reference file gives GitHub Copilot and project contributors authoritative source material for the Microsoft Power Platform Center of Excellence (CoE) Starter Kit.

This project must maximize reuse of the customer's existing CoE investment. Before proposing new Dataverse tables, Power Automate flows, dashboards, inventory processes, or governance capabilities, review the sources below and determine whether the capability already exists in CoE.

> **Important:** The official CoE Starter Kit repository was archived by Microsoft on July 2, 2026 and is now read-only. The customer intends to continue using its existing CoE deployment, so this repository remains an important reference for understanding the components already installed. New custom components must be isolated from Microsoft-managed CoE components and must not modify them directly.

---

## Official GitHub Repository

### Microsoft Power Platform CoE Starter Kit

- [Official Microsoft CoE Starter Kit repository](https://github.com/microsoft/coe-starter-kit)

The repository contains source assets and documentation for components including:

- Core Components
- Audit Components
- Audit Logs
- Nurture Components
- ALM Accelerator
- Pipeline Accelerator
- Innovation Backlog
- Teams Components
- Power BI assets
- CoE CLI
- Supporting documentation and troubleshooting resources

### Relevant GitHub folders

- [Core Components source](https://github.com/microsoft/coe-starter-kit/tree/main/CenterofExcellenceCoreComponents)
- [Audit Components source](https://github.com/microsoft/coe-starter-kit/tree/main/CenterofExcellenceAuditComponents)
- [Audit Logs source](https://github.com/microsoft/coe-starter-kit/tree/main/CenterofExcellenceAuditLogs)
- [Nurture Components source](https://github.com/microsoft/coe-starter-kit/tree/main/CenterofExcellenceNurtureComponents)
- [CoE Resources](https://github.com/microsoft/coe-starter-kit/tree/main/CenterofExcellenceResources)
- [Primary GitHub documentation folder](https://github.com/microsoft/coe-starter-kit/tree/main/Documentation)
- [Additional GitHub docs folder](https://github.com/microsoft/coe-starter-kit/tree/main/docs)

---

## Microsoft Learn Documentation

### Overview and lifecycle

- [Power Platform CoE Starter Kit overview](https://learn.microsoft.com/en-us/power-platform/guidance/coe/overview)
- [CoE Starter Kit transition to Power Platform admin center](https://learn.microsoft.com/en-us/power-platform/guidance/coe/starter-kit)

### Setup and prerequisites

- [Set up the CoE Starter Kit](https://learn.microsoft.com/en-us/power-platform/guidance/coe/setup)
- [Set up inventory components](https://learn.microsoft.com/en-us/power-platform/guidance/coe/setup-core-components)

### Components and data

- [Use CoE Core Components](https://learn.microsoft.com/en-us/power-platform/guidance/coe/core-components)

The Core Components documentation should be reviewed for existing:

- Environment inventory
- Power Apps inventory
- Cloud flow inventory
- Maker and owner information
- Connector inventory
- Power Pages inventory
- Copilot Studio inventory
- Resource relationships
- Administrative applications
- Inventory synchronization flows

### Related current Power Platform administration sources

Because the CoE Starter Kit is no longer actively maintained, also evaluate supported native capabilities when designing integrations or filling gaps:

- [Power Platform admin center documentation](https://learn.microsoft.com/en-us/power-platform/admin/)
- [Power Platform for Admins V2 connector](https://learn.microsoft.com/en-us/connectors/powerplatformadminv2/)
- [Power Apps for Admins connector](https://learn.microsoft.com/en-us/connectors/powerappsforadmins/)
- [Power Platform API overview](https://learn.microsoft.com/en-us/power-platform/admin/programmability-intro-overview)

These native sources may supplement the customer's existing CoE deployment, but they do not change the project charter: this project remains an extension of the customer's existing CoE investment rather than a replacement governance platform.

---

## Project Reuse Directive

Before creating any new component, answer the following questions:

1. Does an equivalent capability already exist in the customer's installed CoE solutions?
2. Is the relevant CoE component installed, configured, populated, and healthy?
3. Can the existing CoE table be read without modifying the managed solution?
4. Can an existing CoE flow or inventory process supply the required data?
5. Is the required information already available in a CoE app, view, or Power BI report?
6. Can this project extend or reference the capability without creating a hard dependency on internal implementation details?
7. Is a native supported Power Platform API or connector required to fill a gap?
8. Would a proposed custom table or flow duplicate inventory already maintained by CoE?

### Reuse preference order

Use the following preference order when designing the solution:

1. Reuse a healthy existing CoE table or capability as-is.
2. Read existing CoE data through a project-owned adapter or query layer.
3. Extend the experience using project-owned components without modifying CoE-managed assets.
4. Supplement missing data through supported Microsoft Graph, Power Platform API, or admin connector operations.
5. Create a new custom table or flow only when the capability cannot be safely obtained from the preceding options.

---

## Guardrails for CoE Integration

- Do not modify Microsoft-managed CoE tables, apps, flows, cloud flows, environment variables, connection references, or solution components.
- Do not add unmanaged customizations directly to CoE-managed components.
- Do not assume the customer's CoE version matches the archived repository's final version.
- Do not assume every CoE solution or feature is installed.
- Do not assume every inventory or audit flow is enabled or healthy.
- Do not invent CoE table, column, flow, app, or dashboard names.
- Validate component names and schemas against the customer's actual environment or exported solution.
- Treat the customer's deployed CoE environment as the runtime source of truth.
- Keep custom licensing tables, flows, applications, and configuration in a separate project-owned solution.
- Document every direct dependency on a CoE component.
- Prefer stable Dataverse relationships and supported APIs over parsing implementation-specific payloads.
- Never interpret missing CoE usage data as zero usage.
- Record source, collection time, and freshness for all evidence used in a license reclamation recommendation.

---

## Customer Deployment Information to Collect

The public repository describes the reference implementation, but the project must also inspect the customer's actual deployment.

Capture the following when available:

- CoE environment name and environment ID
- Installed CoE solution names and versions
- Core Components solution version
- Inventory architecture: cloud-flow inventory or data export
- Enabled and disabled CoE sync flows
- Last successful inventory synchronization
- Failed or suspended inventory flows
- Installed Audit Components and Audit Logs solutions
- Whether app-launch and unique-user collection is configured
- Available usage-history window
- Power BI reports currently used
- Customer customizations or unmanaged layers
- Known excluded environments
- Service identity used by CoE
- Data retention policies
- Existing critical-app or business-criticality classifications

Store customer-specific findings in a separate document. Do not add tenant names, secrets, IDs, credentials, or sensitive configuration to this public reference file.

Recommended file:

`docs/reference/coe-toolkit/customer-deployment-inventory.md`

---

## Expected Project Analysis

Before designing the final Dataverse schema or Power Automate architecture, create:

`docs/coe-reuse-analysis.md`

The analysis should map each envisioned capability to:

- Existing CoE capability
- Existing CoE table or component
- Existing CoE flow or collection process
- Existing CoE report, app, view, or dashboard
- Customer deployment validation status
- Reuse as-is
- Extend through project-owned component
- Supplement through supported API
- New development required
- Duplication risk
- Upgrade and maintenance risk

The analysis must distinguish between:

- What exists in the archived reference repository
- What Microsoft Learn documents
- What is actually installed and operational in the customer's environment

---

## North Star

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.
