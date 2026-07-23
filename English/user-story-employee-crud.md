# US-001 · Employee CRUD Management
### Human Resources Management System

---

| Field | Detail | Field | Detail |
|---|---|---|---|
| **ID** | US-001 | **Module** | Employee Management |
| **Epic** | Human Resources Management | **Sprint** | - |
| **Priority** | High (Must Have) | **Story Points** | - |
| **Status** | `Ready for Development` | **Version** | 1.0 |
| **Author** | Product Manager | **Date** | 07/22/2026 |

---

## Table of Contents

1. [User Story](#user-story)
2. [Context & Description](#context--description)
3. [Personas / Roles Involved](#personas--roles-involved)
4. [Data to Manage](#data-to-manage-employee-entity)
5. [Acceptance Criteria](#acceptance-criteria)
6. [Business Rules](#business-rules)
7. [User Flow](#user-flow-happy-path)
8. [Definition of Done](#definition-of-done)
9. [Assumptions & Dependencies](#assumptions--dependencies)
10. [Risks](#risks)
11. [ISO Compliance](#iso-compliance--control-traceability)
12. [Quality Metrics](#quality-metrics-iso-9001-clause-911)
13. [Out of Scope](#out-of-scope)

---

## User Story

> **As** an HR Administrator,
> **I want** to create, update, and deactivate employee records in the system,
> **so that** staff information is always up to date and available for internal business processes.

---

## Context & Description

The company currently manages employee records through a manual Excel-based process, which produces data inconsistencies, duplicate entries, and difficulties during audits.

This Employee CRUD module centralizes the management of the full employee lifecycle — from onboarding to offboarding — integrating with payroll, access control, and management reporting systems.

The company operates under **ISO/IEC 27001:2022** *(amend. 2024)* (Information Security) and **ISO 9001:2015** (Quality Management) certifications, so this module must comply with the applicable controls and clauses of both standards.

**Complementary diagram:** [diagrams.drawio](./diagrams.drawio) · Page 1 — Employee lifecycle

---

## Personas / Roles Involved

| Role | Profile | Primary need |
|---|---|---|
| **HR Admin** | Primary system user | Create, edit, and deactivate employees quickly and without errors |
| **Area Manager** | Reads team roster | View the status and details of their direct reports |
| **Employee** | Reads their own data | View and request corrections to their personal and employment data |
| **Internal Auditor** | Reviews change history | Query a full trail of modifications with timestamps and responsible users |

**Complementary diagram:** [diagrams.drawio](./diagrams.drawio) · Page 2 — Role permissions matrix

---

## Data to Manage (Employee Entity)

<details>
<summary><strong>Personal Data</strong></summary>

- First name and last name (independent fields)
- National ID / Tax ID (`dni_cuil_cifrado`) *(stored encrypted — see BR-09)*
- Contact phone
- Personal email
- Date of birth
- Gender (M / F / Non-binary / Prefer not to say)
- Nationality
- Address (related entity 1:1 — street, number, city, province, postal code)

</details>

<details>
<summary><strong>Employment Data</strong></summary>

- Employee ID / Internal number
- Start date
- Area / Department
- Position / Job title
- Contract type (permanent, fixed-term, intern, freelance)
- Work modality (on-site, remote, hybrid)
- Assigned office (related entity N:1 — name, address)
- Direct supervisor (linked to another employee record in the system)
- Status (Active / Inactive / On leave)

</details>

<details>
<summary><strong>System & Access Data</strong></summary>

- Corporate email (`email_corporativo`, auto-generated on record creation — see BR-04)
- System role (related entity N:1 — name, permission description)
- Created / last modified date and user (audit trail)

</details>

**Complementary diagram:** [diagrams.drawio](./diagrams.drawio) · Page 5 — Entity-Relationship

---

## Acceptance Criteria

<details>
<summary><strong>AC-01 — Successful employee creation</strong></summary>

- **Given** I am authenticated as an HR Admin.
- **When** I fill in all required fields in the new employee form and click "Save".
- **Then** the system creates the record, automatically generates an Employee ID, assigns a corporate email, and displays the message "Employee successfully registered".

</details>

<details>
<summary><strong>AC-02 — Creation with missing required fields</strong></summary>

- **Given** I attempt to save an employee record without completing a required field.
- **When** I click "Save".
- **Then** the system highlights each empty required field in red, displays a descriptive message for each missing field, and does not save the record.

</details>

<details>
<summary><strong>AC-03 — Creation with duplicate National ID / Tax ID</strong></summary>

- **Given** I enter a National ID or Tax ID that already exists in the system.
- **When** I click "Save".
- **Then** the system blocks the action and displays: "An employee with this National ID / Tax ID already exists".

</details>

<details>
<summary><strong>AC-04 — Employee record update</strong></summary>

- **Given** I select an active employee and edit one or more fields.
- **When** I save the changes.
- **Then** the system updates the record and logs in the audit trail: the modified field, the previous value, the new value, the date/time, and the user who made the change.

</details>

<details>
<summary><strong>AC-05 — Soft deletion (deactivation)</strong></summary>

- **Given** I select the "Deactivate" option on an active employee.
- **When** I confirm the action and provide a reason (resignation / termination / contract expiry / other) and an end date.
- **Then** the employee's status changes to "Inactive", the reason and end date are recorded, and the record remains accessible in read-only mode for audit purposes.

</details>

<details>
<summary><strong>AC-06 — Search and filtering</strong></summary>

- **Given** I access the employee list.
- **When** I apply filters by name, employee ID, area, status, or start date.
- **Then** the system displays only the records matching the selected criteria in under 2 seconds.

</details>

<details>
<summary><strong>AC-07 — List export</strong></summary>

- **Given** I am viewing the employee list (with or without filters applied).
- **When** I click "Export".
- **Then** the system downloads an `.xlsx` file containing the visible data, excluding sensitive fields such as Tax ID for roles without the required permission.

</details>

<details>
<summary><strong>AC-08 — Automatic access revocation on deactivation <em>(ISO 27001 control 5.18)</em></strong></summary>

- **Given** an HR Admin confirms the deactivation of an employee.
- **When** the status changes to "Inactive".
- **Then** the system automatically revokes access to the corporate email account, notifies the IT team with the offboarded employee's details, and records the revocation in the audit log with exact date and time.

</details>

<details>
<summary><strong>AC-09 — Sensitive data masking by role <em>(ISO 27001 control 8.24)</em></strong></summary>

- **Given** a user with the Area Manager or Employee role accesses a profile.
- **When** they view the employee's data.
- **Then** the `dni_cuil_cifrado` field is displayed masked (e.g., `••••••••123`) and cannot be copied or exported by that role.

</details>

---

## Business Rules

| ID | Rule |
|---|---|
| BR-01 | Only users with the **HR Admin** or **Superadmin** role can create, edit, or deactivate employees |
| BR-02 | An employee record **cannot be physically deleted**; it can only be set to Inactive (soft delete) |
| BR-03 | The Employee ID is **auto-generated, sequential, and non-editable** |
| BR-04 | The corporate email is auto-generated using the pattern `firstname.lastname@company.com`, using the `nombre` and `apellido` fields of the record, upon confirming the registration |
| BR-05 | A deactivated employee cannot be reactivated without approval from a **Superadmin** |
| BR-06 | Every modification is logged in the audit trail with the user, date/time, modified field, and old/new values |
| BR-07 | Area Managers can only **view** employees in their own department — they cannot edit records |
| BR-08 | Audit logs are retained for a **minimum of 5 years** and cannot be modified or deleted by any role *(ISO 27001 control 8.15)* |
| BR-09 | The `dni_cuil_cifrado` field is stored **encrypted at rest** (AES-256 or equivalent) and displayed in plain text only to roles with explicit permission *(ISO 27001 control 8.24)* |

---

## User Flow (Happy Path)

```
HR Admin
    │
    ├─► Navigates to HR > Employees
    │         │
    │         ├─► [List view] Filter / Export
    │         │
    │         └─► [+ New Employee]
    │                   │
    │                   ├─► Tab: Personal Data
    │                   ├─► Tab: Employment Data
    │                   └─► Tab: System Access
    │                                   │
    │                                   └─► [Save]
    │                                           │
    │                             System generates Employee ID + email
    │                                           │
    │                                    Confirmation
    │
    ├─► [Edit employee] → Modify fields → Save → Audit log entry
    │
    └─► [Deactivate] → Modal: reason + end date → Confirm → Status: Inactive
                                                                        │
                                                            Access revocation (AC-08)
                                                            IT team notification
                                                            Audit log entry
```

| Step | Action | Description |
|---|---|---|
| 1 | Access the module | HR Admin navigates to **HR > Employees** from the main menu |
| 2 | Employee list | Paginated table: Employee ID · Name · Area · Status · Start Date. Supports filtering and export |
| 3 | Start creation | Clicks **"+ New Employee"** → form opens organized in tabs |
| 4 | Fill the form | Tabs: Personal Data / Employment Data / System Access |
| 5 | Save | Real-time validation → Employee ID and email generated → confirmation shown |
| 6 | Edit employee | Click employee → Edit → modify fields → Save → audit log entry created |
| 7 | Deactivate employee | Click "Deactivate" → modal with reason and end date → Confirm → automatic access revocation + IT notification |

**Complementary diagram:** [diagrams.drawio](./diagrams.drawio) · Page 3 — Onboarding and Offboarding

---

## Definition of Done

- [x] All acceptance criteria (AC-01 through AC-09) validated and approved by QA
- [x] Code reviewed via pull request and approved by at least one peer
- [x] Unit and integration test coverage >= 80%
- [x] Feature tested in staging environment with no regressions
- [x] Technical and user documentation updated
- [x] Product Manager validated the demo in staging
- [x] Compliance with Personal Data Protection Law (Argentina Law 25.326) verified with Legal
- [x] Encryption of sensitive fields verified by the Security team
- [x] Access revocation process tested end-to-end with IT
- [x] ISO traceability review documented and approved by the Quality Manager

---

## Assumptions & Dependencies

**Assumptions**
- The authentication system (SSO / login) is already implemented
- The areas and job titles table is pre-loaded in the system
- The payroll module can consume data from this module via internal API
- The IT team has an API or process available to revoke access at the corporate email provider

**Dependencies**

| Dependency | Description | Status |
|---|---|---|
| US — Area & Department Management | Must exist so the creation form can assign an area and department | Pending creation and estimation |
| US — Centralized audit log | Required for AC-04 and BR-08; must be defined as its own story or as an infrastructure component | Pending creation and estimation |
| US-001-SEC | Encryption and masking of sensitive data | Satellite technical story — see ISO Compliance section |
| — | Corporate email API | Required for auto-generation, notification, and revocation of assigned email |
| — | IT access revocation API / process | Required for AC-08 |

**Complementary diagram:** [diagrams.drawio](./diagrams.drawio) · Page 4 — Story dependencies

---

## Risks

| Risk | Impact | Mitigation |
|---|---|---|
| HR team resistance to abandoning Excel | High | Early training and a parallel transition period |
| Legacy data migration with inconsistent fields | Medium | Data cleanup sprint before go-live |
| Compliance with Law 25.326 (Personal Data Protection) | High | Review with Legal before development begins |
| Email provider without an available revocation API | Medium | Evaluate manual fallback with automated IT notification as contingency |
| Performance impact from encrypting searchable fields | Medium | Design indexes on non-sensitive fields; encrypt at rest only |

---

## ISO Compliance — Control Traceability

This section maps applicable regulatory requirements to the artifact where each is resolved. It is intended for the **Quality Manager and ISO Auditor**.

### ISO/IEC 27001:2022 *(amend. 2024)* — Information Security

| Control | Description | How it is covered |
|---|---|---|
| **5.12** | Classification of information | AC-07, AC-09: export and display restricted by role |
| **5.15** | Access control | BR-01, BR-07: differentiated permissions by role |
| **5.18** | Access rights | AC-08, BR-08: automatic revocation on deactivation + log |
| **5.31** | Legal and contractual requirements | DoD: Legal verification; Law 25.326 |
| **8.10** | Deletion of information | BR-02: soft delete, no physical deletion |
| **8.15** | Logging | BR-06, BR-08: immutable log with minimum 5-year retention |
| **8.24** | Use of cryptography | BR-09, AC-09: AES-256 encryption at rest + UI masking |
| **8.13** | Information backup | Delegated to infrastructure backup policy — out of scope for this US |
| **8.5** | Secure authentication | Delegated to the authentication module (SSO) — assumed as implemented |

### ISO 9001:2015 — Quality Management

| Clause | Description | How it is covered |
|---|---|---|
| **6.1** | Actions to address risks | Risks section with defined mitigations |
| **6.3** | Planning of changes | US versioning (v1.0 → v1.1); changelog documented |
| **8.1** | Operational planning and control | Assumptions, dependencies, and user flow documented |
| **8.5.1** | Control of production and provision | DoD with test coverage >= 80% and staging validation |
| **8.5.2** | Traceability | US ID, epic, sprint, and cross-references between stories |
| **8.6** | Release of products | DoD includes PM and Quality Manager approval |
| **9.1.1** | Monitoring and measurement | See Quality Metrics section below |
| **9.1.2** | Customer satisfaction | Pending: post go-live satisfaction survey for the HR team |

### Satellite Technical Story

| ID | Description | Controls covered |
|---|---|---|
| **US-001-SEC** | Encryption at rest for sensitive data (`dni_cuil_cifrado`) and role-based UI masking | ISO 27001: 8.24, 5.12 |

---

## Quality Metrics *(ISO 9001 clause 9.1.1)*

These metrics must be collected during the first 30 days post go-live and reported to the Quality Manager.

| Metric | Description | Acceptable threshold |
|---|---|---|
| **Creation error rate** | % of employee creation attempts resulting in a system error (user errors excluded) | <= 1% |
| **Average creation time** | Time from when the Admin starts the form to receiving confirmation | <= 3 minutes |
| **List response time** | Load time for the employee list with filters applied | <= 2 seconds (already in AC-06) |
| **Successful revocation rate** | % of deactivations where access revocation completed without manual intervention | >= 98% |
| **User satisfaction** | Post go-live survey with the HR team (scale 1-5) | >= 4.0 |

---

## Out of Scope

The following features are **explicitly excluded** from this User Story:

- Leave and vacation management
- Payroll and salary processing → Payroll module
- Digital employee file / document attachments
- Interactive org chart
- Backup and infrastructure policy → IT / DevOps responsibility
- Authentication module and SSO → assumed as already implemented

---

*PM Challenge · July 2026 · Francisco Roldán — [roldanfrancisco.personal@gmail.com](mailto:roldanfrancisco.personal@gmail.com) · [LinkedIn](https://www.linkedin.com/in/roldanfrancisco) · [GitHub](https://github.com/roldn)*
