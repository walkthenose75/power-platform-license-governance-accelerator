# Build Checkpoint

_Checkpoint created 2026-10-07. Resume by telling the assistant: **"Resume the License Governance build."**_

## Summary

- **WP0 (validation + foundation): complete.**
- **WP1 data model: complete** — standalone solution, 8 tables, 7 relationships.
- **Model-driven app: created and published.**
- **Only remaining step:** wire the 8 tables into the app's navigation (blocked at pause time by a transient Dataverse connectivity blip from the build shell — not a design issue).

Everything is built in a **standalone, unmanaged, project-owned solution**, fully isolated from CoE (read-only reuse of CoE only).

## Environment

- **Target:** Contoso - Dev (development/build environment). Exact instance URL, environment ID, and tenant ID are stored in the **session workspace** (`resume-state.json`), not in this public repo.
- **Access:** a disposable **build service principal** (`LicenseGovAccelerator-Build`, application user with System Administrator in Contoso - Dev). Its credentials live **only** in the session workspace (`contoso-dev-sp.json`) — never committed. It is revocable at any time from Entra / Power Platform admin.

## Built and persistent in Contoso - Dev

- **Publisher** `lga` + **unmanaged solution** `LicenseGovernanceAccelerator` (v1.0.0.0).
- **8 tables** (prefix `lga_`) with full columns, organization-owned, per `docs/data-model.md`:
  - `lga_skureference` (SKU Reference)
  - `lga_licenseassignment` (License Assignment)
  - `lga_optimizationcandidate` (Optimization Candidate)
  - `lga_recommendationevidence` (Recommendation Evidence)
  - `lga_dependencyfinding` (Dependency Finding)
  - `lga_reclamationaction` (Reclamation Action)
  - `lga_auditlogentry` (Audit Log Entry)
  - `lga_protectedidentity` (Protected Identity)
- **7 relationships** with delete behaviors: RemoveLink (SKU→License, License→Candidate), Cascade (Candidate→Evidence, Candidate→Dependency), Restrict (Candidate→Reclamation, License→Reclamation, Reclamation→Audit). Protected Identity standalone.
- **Icon web resource** `lga_licensegovicon.svg`.
- **Model-driven app** "License Governance" (unique name `lga_licensegovernance`) — created via `pac model create` and published. (App ID in `resume-state.json`.)

## Pending — next action on resume

1. **Wire the app navigation:** add the 8 `lga_` tables as app components and set the sitemap (Optimization / Reclamation / Configuration groups), then publish.
   - Was blocked by intermittent "connection forcibly closed by remote host" errors from the shell to the Dataverse endpoint. The token endpoint and `pac` both worked; the data endpoint degraded mid-session.
   - **Alternative (works from a browser now):** maker portal → Solutions → License Governance Accelerator → open the **License Governance** app → + Add page → Dataverse tables → add the 8 `lga_` tables → Save → Publish.

## After the app navigation (remaining WP1 / build)

- Security roles (4: Executive / Analyst / Administrator / Auditor) + field-level security (per `docs/data-model.md`).
- Curated public views (e.g., Safe candidates, Direct vs. group) and tidy main forms.
- Graph **licensing acquisition + correlation** → `lga_licenseassignment` (delegated connector per ADR-005; needs a Graph connection/consent step).
- Optional: seed sample data (CoE inventory/usage is currently empty in Contoso - Dev).
- WP6 packaging: export the **unmanaged solution** into `solution/` for version control, finalize the handoff guides.

## Key validation findings (see `docs/reference/coe-toolkit/customer-deployment-inventory.md`)

- CoE **Core Components 4.50.9** + **Governance/Audit Components 3.27.7** installed (managed).
- Real CoE tables confirmed: `admin_app`, `admin_flow`, `admin_maker` / `admin_powerplatformuser`, `admin_connector`, `admin_connectionreference`, `admin_auditlog`, `admin_environment`.
- **Gap G1 resolved:** `admin_auditlog` retains **per-user** app-launch events (`admin_userid`, `admin_userupn`, `admin_appid`, `admin_appispremium`, `admin_creationtime`).
- **Correlation key:** UPN (no dedicated AAD object-id column on the maker table).
- **CoE data currently empty** (sync not yet populated) — end-to-end testing will need synced or seeded data.

## How resume works

On "Resume the License Governance build", the assistant will: read this checkpoint + `resume-state.json` + `contoso-dev-sp.json` from the session workspace, re-acquire a Dataverse token, verify connectivity, and continue from the pending app-navigation wiring.
