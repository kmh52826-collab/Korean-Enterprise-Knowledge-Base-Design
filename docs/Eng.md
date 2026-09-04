# Data Table Definition

> Validation Management Platform data model documentation  
> **Security note**: Example values have been anonymized for repository publication.

## Contents

- [1. Document Overview](#1-document-overview)
- [2. Notation](#2-notation)
- [3. Table List](#3-table-list)
- [4. Domain Index](#4-domain-index)
- [5. Detailed Table and Column Definitions](#5-detailed-table-and-column-definitions)

## 1. Document Overview

- Tables: **51**
- Columns: **783**
- Domains: **25**
- Naming convention: physical table and column names use `snake_case`.
- Date and time: timestamps are stored in UTC and displayed using the applicable user or site time zone.
- GxP principle: approved records are not directly updated; revisions, status history, and Audit Trail records preserve traceability.

## 2. Notation

| Notation | Meaning |
|---|---|
| `Y` | Applied or required |
| `N` | Not applied or optional |
| `-` | Not applicable or not specified |
| PK | Primary Key |
| FK | Foreign Key |
| Not Null | NULL values are not allowed |
| Audit | Subject to Audit Trail recording |

## 3. Table List

| No. | Domain | Logical Table Name | Physical Table Name | PK | Main References | GxP Criticality | Audit |
|---:|---|---|---|---|---|:---:|:---:|
| 1 | Organization | Organization / Customer | [`organization`](#table-organization) | `organization_id` | - | High | Y |
| 2 | Security | User | [`app_user`](#table-app_user) | `user_id` | `organization_id` | High | Y |
| 3 | Security | Role | [`role`](#table-role) | `role_id` | - | High | Y |
| 4 | Security | User Role | [`user_role`](#table-user_role) | `user_role_id` | `user_id, role_id` | High | Y |
| 5 | Compliance | Electronic Signature | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 6 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 7 | File | File Asset | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id` | High | Y |
| 8 | System | System / Equipment Asset | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 9 | Library | Library Item Master | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 10 | Validation | Validation Project | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by` | High | Y |
| 11 | QIA | Quality Impact Assessment | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id` | High | Y |
| 12 | QIA | QIA Module Assessment Item | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id` | High | Y |
| 13 | VA | Vendor Audit Assessment | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id` | High | Y |
| 14 | URS | User Requirements Specification | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by` | High | Y |
| 15 | FDS | Functional Design Specification | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by` | High | Y |
| 16 | FDS | FDS Detail Item | [`fds_item`](#table-fds_item) | `fds_item_id` | `fds_id, created_by, updated_by` | High | Y |
| 17 | FDS | FDS Interface Definition | [`fds_interface`](#table-fds_interface) | `fds_interface_id` | `fds_id, created_by, updated_by` | High | Y |
| 18 | DQ | Design Qualification Assessment | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 19 | DQ | DQ Assessment Item | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, reviewed_by, created_by, updated_by` | High | Y |
| 20 | FRA | Functional Risk Assessment | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 21 | FRA | FRA Risk Item | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by` | High | Y |
| 22 | IQ | Installation Qualification Assessment | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 23 | IQ | IQ Test Item | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, executed_by, created_by, updated_by` | High | Y |
| 24 | OQ | Operational Qualification Assessment | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 25 | OQ | OQ Test Item | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, executed_by, created_by, updated_by` | High | Y |
| 26 | PQ | Performance Qualification Assessment | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 27 | PQ | PQ Test Item | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, executed_by, created_by, updated_by` | High | Y |
| 28 | RTM | Requirements Traceability Matrix | [`rtm_assessment`](#table-rtm_assessment) | `rtm_id` | `project_id, created_by, updated_by` | Critical | Y |
| 29 | RTM | RTM Traceability Item | [`rtm_item`](#table-rtm_item) | `rtm_item_id` | `rtm_id, requirement_id, created_by, updated_by` | Critical | Y |
| 30 | VSR | Validation Summary Report | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by` | Critical | Y |
| 31 | VSR | VSR Activity Summary Item | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by` | Critical | Y |
| 32 | Workflow | Workflow Instance | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 33 | Workflow | Workflow Step | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, assignee_id, created_by, updated_by` | Critical | Y |
| 34 | Workflow | Approval Action History | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, actor_id, signature_id` | Critical | Y |
| 35 | Traceability | Traceability Link | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 36 | File | Evidence File Link | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 37 | Validation | Project Member | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 38 | Validation | Validation Activity Master | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 39 | Validation | Project Activity | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 40 | Validation | Activity Dependency | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 41 | DDS | Detailed Design Specification | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by` | High | Y |
| 42 | DDS | DDS Detail Item | [`dds_item`](#table-dds_item) | `dds_item_id` | `dds_id, created_by, updated_by` | High | Y |
| 43 | Deviation | Deviation | [`deviation`](#table-deviation) | `deviation_id` | `project_id, resolved_by, approved_by, created_by, updated_by` | Critical | Y |
| 44 | Report | Report Generation Job | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, report_schedule_id, created_by, updated_by` | High | Y |
| 45 | AI | AI Generation Job | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 46 | AI | AI Generation Result | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, created_by, updated_by` | High | Y |
| 47 | AI | AI Result Item | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 48 | Notification | Notification Delivery | [`notification_delivery`](#table-notification_delivery) | `notification_delivery_id` | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` | High | Y |
| 49 | System | Backup Execution History | [`backup_execution`](#table-backup_execution) | `backup_execution_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 50 | Report | Report Schedule | [`report_schedule`](#table-report_schedule) | `report_schedule_id` | `project_id, created_by, updated_by` | High | Y |
| 51 | File | File Cleanup Execution History | [`file_cleanup_execution`](#table-file_cleanup_execution) | `file_cleanup_execution_id` | `requested_by, created_by, updated_by` | High | Y |

## 4. Domain Index

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role)
- **Compliance**: [`electronic_signature`](#table-electronic_signature), [`audit_trail`](#table-audit_trail)
- **File**: [`file_asset`](#table-file_asset), [`evidence_link`](#table-evidence_link), [`file_cleanup_execution`](#table-file_cleanup_execution)
- **System**: [`system_asset`](#table-system_asset), [`backup_execution`](#table-backup_execution)
- **Library**: [`library_item`](#table-library_item)
- **Validation**: [`validation_project`](#table-validation_project), [`project_member`](#table-project_member), [`validation_activity`](#table-validation_activity), [`project_activity`](#table-project_activity), [`activity_dependency`](#table-activity_dependency)
- **QIA**: [`qia_assessment`](#table-qia_assessment), [`qia_module_item`](#table-qia_module_item)
- **VA**: [`vendor_audit`](#table-vendor_audit)
- **URS**: [`requirement`](#table-requirement)
- **FDS**: [`fds_spec`](#table-fds_spec), [`fds_item`](#table-fds_item), [`fds_interface`](#table-fds_interface)
- **DQ**: [`dq_assessment`](#table-dq_assessment), [`dq_item`](#table-dq_item)
- **FRA**: [`fra_assessment`](#table-fra_assessment), [`fra_item`](#table-fra_item)
- **IQ**: [`iq_assessment`](#table-iq_assessment), [`iq_item`](#table-iq_item)
- **OQ**: [`oq_assessment`](#table-oq_assessment), [`oq_item`](#table-oq_item)
- **PQ**: [`pq_assessment`](#table-pq_assessment), [`pq_item`](#table-pq_item)
- **RTM**: [`rtm_assessment`](#table-rtm_assessment), [`rtm_item`](#table-rtm_item)
- **VSR**: [`vsr_assessment`](#table-vsr_assessment), [`vsr_item`](#table-vsr_item)
- **Workflow**: [`workflow_instance`](#table-workflow_instance), [`workflow_step`](#table-workflow_step), [`approval_action`](#table-approval_action)
- **Traceability**: [`traceability_link`](#table-traceability_link)
- **DDS**: [`dds_spec`](#table-dds_spec), [`dds_item`](#table-dds_item)
- **Deviation**: [`deviation`](#table-deviation)
- **Report**: [`report_generation`](#table-report_generation), [`report_schedule`](#table-report_schedule)
- **AI**: [`ai_generation_job`](#table-ai_generation_job), [`ai_generation_result`](#table-ai_generation_result), [`ai_result_item`](#table-ai_result_item)
- **Notification**: [`notification_delivery`](#table-notification_delivery)

---

# 5. Detailed Table and Column Definitions

## Organization

<a id="table-organization"></a>
### 1. Organization / Customer (`organization`)

| Item | Definition |
|---|---|
| Description | Stores customer and operating organization master data. |
| Primary Key | `organization_id` |
| Main References | - |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 14 | Organization ID | `organization_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Organization / Customer record. | `00000000-0000-0000-0000-000000000001` |
| 15 | Organization Code | `organization_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Organization Code for the record. | `ORG-DA-01` |
| 16 | Organization Name | `organization_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Organization Name for the record. | `Sample Corporation` |
| 17 | Organization Type | `organization_type` | `varchar(50)` | N | N | - | Y | `HEAD_OFFICE` | N | N | N | Y | Organization Type for the record. | `Sample Organization Type` |
| 18 | Status | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE'` | N | N | N | Y | Status for the record. | `ACTIVE` |
| 19 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description for the record. | `Sample Description` |
| 20 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-01 00:00:00` |
| 21 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 16:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

## Security

<a id="table-app_user"></a>
### 2. User (`app_user`)

| Item | Definition |
|---|---|
| Description | Stores user accounts and basic profile information. |
| Primary Key | `user_id` |
| Main References | `organization_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1 | User ID | `user_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the User record. | `00000000-0000-0000-0000-000000000001` |
| 2 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | References organization.organization_id. | `00000000-0000-0000-0000-000000000001` |
| 3 | Username | `username` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Username for the record. | `user.sample` |
| 4 | Password Hash | `password_hash` | `varchar(255)` | N | N | - | Y | - | N | N | N | N | Password Hash for the record. | `[REDACTED_PASSWORD_HASH]` |
| 5 | Full Name | `full_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Full Name for the record. | `Sample User` |
| 6 | Email | `email` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | Email for the record. | `user.sample@example.com` |
| 7 | Department Name | `department_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Department Name for the record. | `Information Strategy Team` |
| 8 | Position Title | `position_title` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Position Title for the record. | `Sample Position Title` |
| 9 | Status | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE'` | N | N | N | Y | Status for the record. | `ACTIVE` |
| 10 | Last Login At | `last_login_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Last Login At for the record. | `2026-08-26 16:00:00` |
| 11 | Password Changed At | `password_changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Password Changed At for the record. | `2026-08-01 09:00:00` |
| 12 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-01 00:00:00` |
| 13 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 16:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-role"></a>
### 3. Role (`role`)

| Item | Definition |
|---|---|
| Description | Master data for authorization roles such as author, reviewer, and approver. |
| Primary Key | `role_id` |
| Main References | - |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 22 | Role ID | `role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Role record. | `00000000-0000-0000-0000-000000000001` |
| 23 | Role Code | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Role Code for the record. | `SYSTEM_ADMIN` |
| 24 | Role Name | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Role Name for the record. | `Sample Role Name` |
| 25 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description for the record. | `Sample Description` |
| 26 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-01 00:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-user_role"></a>
### 4. User Role (`user_role`)

| Item | Definition |
|---|---|
| Description | Maps users to authorization roles. |
| Primary Key | `user_role_id` |
| Main References | `user_id, role_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 27 | User Role ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the User Role record. | `00000000-0000-0000-0000-000000000001` |
| 28 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 29 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | References role.role_id. | `00000000-0000-0000-0000-000000000001` |
| 30 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-01 00:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

## Compliance

<a id="table-electronic_signature"></a>
### 5. Electronic Signature (`electronic_signature`)

| Item | Definition |
|---|---|
| Description | Preserves electronic-signature evidence for document review and approval. |
| Primary Key | `signature_id` |
| Main References | `signer_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 31 | Signature ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Electronic Signature record. | `00000000-0000-0000-0000-000000000001` |
| 32 | Signer ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 33 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Target Table Name for the record. | `requirement` |
| 34 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Target Record ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 35 | Signature Action | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Signature Action for the record. | `APPROVE` |
| 36 | Signature Meaning | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Signature Meaning for the record. | `Sample Signature Meaning` |
| 37 | Signed At | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Signed At for the record. | `2026-08-26 16:30:00` |
| 38 | Target Version | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Target Version for the record. | `v1.0` |
| 39 | Content Hash | `content_hash` | `varchar(128)` | N | N | - | Y | - | N | N | N | Y | Content Hash for the record. | `[SAMPLE_SHA256_HASH]` |
| 40 | Authentication Method | `authentication_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Authentication Method for the record. | `PASSWORD` |
| 41 | Authentication Result | `authentication_result` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Authentication Result for the record. | `SUCCESS` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-audit_trail"></a>
### 6. Audit Trail (`audit_trail`)

| Item | Definition |
|---|---|
| Description | Records before and after values, actors, and context for auditable data changes. |
| Primary Key | `audit_id` |
| Main References | `actor_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 42 | Audit ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Audit Trail record. | `00000000-0000-0000-0000-000000000001` |
| 43 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 44 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Action Type for the record. | `REPORT_GENERATE` |
| 45 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Target Table Name for the record. | `requirement` |
| 46 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Target Record ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 47 | Old Values | `old_values` | `jsonb` | N | N | - | N | - | N | N | N | Y | Old Values for the record. | `Sample Old Values` |
| 48 | New Values | `new_values` | `jsonb` | N | N | - | N | - | N | N | N | Y | New Values for the record. | `Sample New Values` |
| 49 | Reason For Change | `reason_for_change` | `text` | N | N | - | N | - | N | N | N | Y | Reason For Change for the record. | `Sample Reason For Change` |
| 50 | Client IP | `client_ip` | `varchar(45)` | N | N | - | N | - | N | N | N | Y | Client IP for the record. | `192.0.2.10` |
| 51 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 16:30:00` |
| 52 | Actor Type | `actor_type` | `varchar(20)` | N | N | - | Y | `USER` | N | Y | N | Y | Actor Type for the record. | `USER` |
| 53 | Request ID | `request_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Request ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 54 | Session ID | `session_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Session ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 55 | Target Version | `target_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Target Version for the record. | `v1.0` |
| 56 | Target Revision Number | `target_revision_number` | `integer` | N | N | - | N | - | N | N | N | Y | Target Revision Number for the record. | `1` |
| 57 | Request URI | `request_uri` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Request URI for the record. | `/api/fds/approve` |
| 58 | User Agent | `user_agent` | `text` | N | N | - | N | - | N | N | Y | Y | User Agent for the record. | `Mozilla/5.0` |

[↑ Back to top](#dvt-data-table-definition)

---

## File

<a id="table-file_asset"></a>
### 7. File Asset (`file_asset`)

| Item | Definition |
|---|---|
| Description | Stores metadata for attachments, evidence, generated reports, and exported files. |
| Primary Key | `file_id` |
| Main References | `uploader_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 59 | File ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the File Asset record. | `00000000-0000-0000-0000-000000000001` |
| 60 | Uploader ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 61 | Original File Name | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Original File Name for the record. | `sample_document.pdf` |
| 62 | Stored File Path | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | Stored File Path for the record. | `sample/path` |
| 63 | File Category | `file_category` | `varchar(30)` | N | N | - | Y | `ATTACHMENT` | N | Y | N | Y | File Category for the record. | `REPORT` |
| 64 | File Size Bytes | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | File Size Bytes for the record. | `1048576` |
| 65 | MIME Type | `mime_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | MIME Type for the record. | `application/pdf` |
| 66 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 16:30:00` |
| 67 | Is Temporary | `is_temporary` | `boolean` | N | N | - | Y | `False` | N | Y | N | Y | Is Temporary for the record. | `True` |
| 68 | Expires At | `expires_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Expires At for the record. | `2026-09-09 18:00:00+00` |
| 69 | Cleanup Status | `cleanup_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | Cleanup Status for the record. | `ACTIVE` |
| 70 | Cleanup Execution ID | `cleanup_execution_id` | `uuid` | N | Y | `file_cleanup_execution.file_cleanup_execution_id` | N | - | N | Y | N | Y | References file_cleanup_execution.file_cleanup_execution_id. | `00000000-0000-0000-0000-000000000001` |
| 71 | Cleaned At | `cleaned_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Cleaned At for the record. | `2026-09-09 19:00:00+00` |
| 72 | Cleanup Error Message | `cleanup_error_message` | `text` | N | N | - | N | - | N | N | N | Y | Cleanup Error Message for the record. | `Sample Cleanup Error Message` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-evidence_link"></a>
### 36. Evidence File Link (`evidence_link`)

| Item | Definition |
|---|---|
| Description | Provides a many-to-many link between documents or test items and evidence files. |
| Primary Key | `evidence_link_id` |
| Main References | `project_id, file_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 523 | Evidence Link ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Evidence File Link record. | `00000000-0000-0000-0000-000000000001` |
| 524 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 525 | File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | References file_asset.file_id. | `00000000-0000-0000-0000-000000000001` |
| 526 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Target Entity Type for the record. | `IQ_ITEM` |
| 527 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Target Entity ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 528 | Evidence Type | `evidence_type` | `varchar(50)` | N | N | - | Y | `TEST_RESULT` | N | Y | N | Y | Evidence Type for the record. | `TEST_RESULT` |
| 529 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description for the record. | `Sample Description` |
| 530 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-01 10:00:00` |
| 531 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 532 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-01 10:00:00` |
| 533 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 534 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-file_cleanup_execution"></a>
### 51. File Cleanup Execution History (`file_cleanup_execution`)

| Item | Definition |
|---|---|
| Description | Stores temporary and expired file cleanup execution history and outcomes. |
| Primary Key | `file_cleanup_execution_id` |
| Main References | `requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 762 | File Cleanup Execution ID | `file_cleanup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the File Cleanup Execution History record. | `00000000-0000-0000-0000-000000000001` |
| 763 | Cleanup Type | `cleanup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | Cleanup Type for the record. | `EXPIRED_FILE` |
| 764 | Execution Type | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | Execution Type for the record. | `SCHEDULED` |
| 765 | Target Base At | `target_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Target Base At for the record. | `2026-09-02 00:00:00+00` |
| 766 | Execution Status | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Execution Status for the record. | `COMPLETED` |
| 767 | Scanned File Count | `scanned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Scanned File Count for the record. | `100` |
| 768 | Target File Count | `target_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Target File Count for the record. | `20` |
| 769 | Cleaned File Count | `cleaned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Cleaned File Count for the record. | `19` |
| 770 | Failed File Count | `failed_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Failed File Count for the record. | `1` |
| 771 | Requested By | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 772 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Requested At for the record. | `2026-09-02 01:00:00+00` |
| 773 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Started At for the record. | `2026-09-02 01:00:05+00` |
| 774 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completed At for the record. | `2026-09-02 01:05:00+00` |
| 775 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Retry Count for the record. | `0` |
| 776 | Max Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Max Retry Count for the record. | `3` |
| 777 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next Retry At for the record. | `2026-09-02 01:15:00+00` |
| 778 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Error Code for the record. | `FILE_DELETE_FAILED` |
| 779 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Error Message for the record. | `Sample Error Message` |
| 780 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 01:00:00+00` |
| 781 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 782 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 01:05:00+00` |
| 783 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |

[↑ Back to top](#dvt-data-table-definition)

---

## System

<a id="table-system_asset"></a>
### 8. System / Equipment Asset (`system_asset`)

| Item | Definition |
|---|---|
| Description | Stores identification and classification data for systems and equipment subject to validation. |
| Primary Key | `system_id` |
| Main References | `organization_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 73 | System ID | `system_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the System / Equipment Asset record. | `00000000-0000-0000-0000-000000000001` |
| 74 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | References organization.organization_id. | `00000000-0000-0000-0000-000000000001` |
| 75 | Management Number | `management_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Management Number for the record. | `EQ-MES-2024-001` |
| 76 | System Name | `system_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | System Name for the record. | `MES` |
| 77 | System Type | `system_type` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | System Type for the record. | `Sample System Type` |
| 78 | Department Name | `department_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | N | Department Name for the record. | `Information Strategy Team` |
| 79 | Location | `location` | `varchar(200)` | N | N | - | N | - | N | N | N | N | Location for the record. | `sample/path` |
| 80 | Vendor | `vendor` | `varchar(100)` | N | N | - | N | - | N | N | N | N | Vendor for the record. | `Sample Vendor` |
| 81 | Model Name | `model_name` | `varchar(100)` | N | N | - | N | - | N | N | N | N | Model Name for the record. | `FillMaster 500` |
| 82 | Description | `description` | `text` | N | N | - | N | - | N | N | N | N | Description for the record. | `Sample Description` |
| 83 | Identification Status | `identification_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Identification Status for the record. | `COMPLETED` |
| 84 | Is Cs Included | `is_cs_included` | `boolean` | N | N | - | Y | `True` | N | N | N | Y | Is Cs Included for the record. | `True` |
| 85 | Version | `version` | `varchar(50)` | N | N | - | N | - | N | N | N | N | Version for the record. | `v3.2.1` |
| 86 | GAMP Category | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | N | GAMP Category for the record. | `Category 4` |
| 87 | GxP Type | `gxp_type` | `varchar(50)` | N | N | - | N | - | N | N | N | N | GxP Type for the record. | `GMP` |
| 88 | Status | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | N | N | Y | Status for the record. | `Sample Status` |
| 89 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-25 00:00:00` |
| 90 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-25 00:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-backup_execution"></a>
### 49. Backup Execution History (`backup_execution`)

| Item | Definition |
|---|---|
| Description | Stores scheduled or manual backup execution status and retry history. |
| Primary Key | `backup_execution_id` |
| Main References | `requested_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 723 | Backup Execution ID | `backup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Backup Execution History record. | `00000000-0000-0000-0000-000000000001` |
| 724 | Backup Type | `backup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | Backup Type for the record. | `FULL` |
| 725 | Execution Type | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | Execution Type for the record. | `SCHEDULED` |
| 726 | Backup Target | `backup_target` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Backup Target for the record. | `ALL` |
| 727 | Backup Base At | `backup_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Backup Base At for the record. | `2026-09-02 18:00:00+00` |
| 728 | Execution Status | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Execution Status for the record. | `COMPLETED` |
| 729 | Backup Location | `backup_location` | `text` | N | N | - | N | - | N | N | N | Y | Backup Location for the record. | `sample/path` |
| 730 | Backup Size Bytes | `backup_size_bytes` | `bigint` | N | N | - | N | - | N | N | N | Y | Backup Size Bytes for the record. | `1073741824` |
| 731 | Requested By | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 732 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Requested At for the record. | `2026-09-02 18:00:00+00` |
| 733 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Started At for the record. | `2026-09-02 18:00:05+00` |
| 734 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completed At for the record. | `2026-09-02 18:20:00+00` |
| 735 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Retry Count for the record. | `0` |
| 736 | Max Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Max Retry Count for the record. | `3` |
| 737 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next Retry At for the record. | `2026-09-02 18:30:00+00` |
| 738 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Error Code for the record. | `BACKUP_STORAGE_UNAVAILABLE` |
| 739 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Error Message for the record. | `Sample Error Message` |
| 740 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 18:00:00+00` |
| 741 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 742 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 18:20:00+00` |
| 743 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |

[↑ Back to top](#dvt-data-table-definition)

---

## Library

<a id="table-library_item"></a>
### 9. Library Item Master (`library_item`)

| Item | Definition |
|---|---|
| Description | Master data for reusable URS, IQ, and OQ library items. |
| Primary Key | `library_id` |
| Main References | - |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 91 | Library ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Library Item Master record. | `00000000-0000-0000-0000-000000000001` |
| 92 | Module Type | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Module Type for the record. | `URS` |
| 93 | Code | `code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Code for the record. | `URS-AT-L01` |
| 94 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Category for the record. | `Sample Category` |
| 95 | Title | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 96 | Requirement Text | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Requirement Text for the record. | `Sample Requirement Text` |
| 97 | Expected Result | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | Expected Result for the record. | - |
| 98 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Acceptance Criteria for the record. | `Sample Acceptance Criteria` |
| 99 | Regulation | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Regulation for the record. | `21 CFR 11.10(e)` |
| 100 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | N | N | Y | Is Active for the record. | `True` |
| 101 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-25 00:00:00` |
| 102 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-25 00:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

## Validation

<a id="table-validation_project"></a>
### 10. Validation Project (`validation_project`)

| Item | Definition |
|---|---|
| Description | Defines the execution scope and unit of validation for each system. |
| Primary Key | `project_id` |
| Main References | `system_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 103 | Project ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Validation Project record. | `00000000-0000-0000-0000-000000000001` |
| 104 | Project Code | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | Project Code for the record. | `VP-SYS-008-20260422` |
| 105 | Project Name | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | Project Name for the record. | `Sample Project Name` |
| 106 | System ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | References system_asset.system_id. | `00000000-0000-0000-0000-000000000001` |
| 107 | Progress Rate | `progress_rate` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Progress Rate for the record. | `0` |
| 108 | Status | `status` | `varchar(20)` | N | N | - | Y | `IN_PROGRESS` | N | N | N | Y | Status for the record. | `Sample Status` |
| 109 | Start Date | `start_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | Start Date for the record. | `2026-04-15 00:00:00` |
| 110 | Validation Type | `validation_type` | `varchar(50)` | N | N | - | Y | `NEW_VALIDATION` | N | N | N | Y | Validation Type for the record. | `Sample Validation Type` |
| 111 | Validation Level | `validation_level` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Validation Level for the record. | `Level 4` |
| 112 | Context Status | `context_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Context Status for the record. | `CONFIRMED` |
| 113 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Remarks for the record. | - |
| 114 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-25 00:00:00` |
| 115 | GAMP Category | `gamp_category` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | GAMP Category for the record. | `Category 3` |
| 116 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 117 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 118 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 119 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-project_member"></a>
### 37. Project Member (`project_member`)

| Item | Definition |
|---|---|
| Description | Stores project participants and their execution roles. |
| Primary Key | `project_member_id` |
| Main References | `project_id, user_id, role_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 535 | Project Member ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Project Member record. | `00000000-0000-0000-0000-000000000001` |
| 536 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 537 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 538 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | References role.role_id. | `00000000-0000-0000-0000-000000000001` |
| 539 | Member Status | `member_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | Member Status for the record. | `ACTIVE` |
| 540 | Joined At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Joined At for the record. | `2026-09-01 10:00:00` |
| 541 | Left At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Left At for the record. | - |
| 542 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-01 10:00:00` |
| 543 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 544 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-01 10:00:00` |
| 545 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 546 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-validation_activity"></a>
### 38. Validation Activity Master (`validation_activity`)

| Item | Definition |
|---|---|
| Description | Master data for validation activities such as VP, QIA, URS, IQ, OQ, PQ, RTM, and VSR. |
| Primary Key | `activity_id` |
| Main References | `created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 547 | Activity ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Validation Activity Master record. | `00000000-0000-0000-0000-000000000001` |
| 548 | Activity Code | `activity_code` | `varchar(20)` | N | N | - | Y | - | Y | Y | N | Y | Activity Code for the record. | `URS` |
| 549 | Activity Name | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Activity Name for the record. | `Sample Activity Name` |
| 550 | Display Order | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Display Order for the record. | `5` |
| 551 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Active for the record. | `True` |
| 552 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `09/01/2026 10:00:00` |
| 553 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 554 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `09/01/2026 10:00:00` |
| 555 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 556 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-project_activity"></a>
### 39. Project Activity (`project_activity`)

| Item | Definition |
|---|---|
| Description | Stores selected project activities, activation status, and progress. |
| Primary Key | `project_activity_id` |
| Main References | `project_id, activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 557 | Project Activity ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Project Activity record. | `00000000-0000-0000-0000-000000000001` |
| 558 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 559 | Activity ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | References validation_activity.activity_id. | `00000000-0000-0000-0000-000000000001` |
| 560 | Is Selected | `is_selected` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | Is Selected for the record. | `True` |
| 561 | Is Required | `is_required` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | Is Required for the record. | `True` |
| 562 | Activity Status | `activity_status` | `varchar(20)` | N | N | - | Y | `LOCKED` | N | Y | N | Y | Activity Status for the record. | `READY` |
| 563 | Activated At | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activated At for the record. | `2026-09-01 10:00:00` |
| 564 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Started At for the record. | `2026-09-01 11:00:00` |
| 565 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completed At for the record. | `2026-09-02 15:00:00` |
| 566 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Approved At for the record. | `2026-09-02 17:00:00` |
| 567 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-01 10:00:00` |
| 568 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 569 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-01 10:00:00` |
| 570 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 571 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-activity_dependency"></a>
### 40. Activity Dependency (`activity_dependency`)

| Item | Definition |
|---|---|
| Description | Stores predecessor conditions and decision rules used to activate successor activities. |
| Primary Key | `activity_dependency_id` |
| Main References | `successor_activity_id, predecessor_activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 572 | Activity Dependency ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Activity Dependency record. | `00000000-0000-0000-0000-000000000001` |
| 573 | Successor Activity ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | References validation_activity.activity_id. | `00000000-0000-0000-0000-000000000001` |
| 574 | Predecessor Activity ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | References validation_activity.activity_id. | `00000000-0000-0000-0000-000000000001` |
| 575 | Dependency Type | `dependency_type` | `varchar(20)` | N | N | - | Y | `REQUIRED` | N | Y | N | Y | Dependency Type for the record. | `REQUIRED` |
| 576 | Required Status | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Required Status for the record. | `APPROVED` |
| 577 | Condition Type | `condition_type` | `varchar(50)` | N | N | - | Y | `STATUS` | N | Y | N | Y | Condition Type for the record. | `STATUS` |
| 578 | Condition Value | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Condition Value for the record. | `APPROVED` |
| 579 | Condition Description | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | Condition Description for the record. | `Sample Condition Description` |
| 580 | Evaluation Order | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Evaluation Order for the record. | `1` |
| 581 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Active for the record. | `True` |
| 582 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-01 10:00:00` |
| 583 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 584 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-01 10:00:00` |
| 585 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 586 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## QIA

<a id="table-qia_assessment"></a>
### 11. Quality Impact Assessment (`qia_assessment`)

| Item | Definition |
|---|---|
| Description | Stores project-level QIA conclusions and 21 CFR Part 11 assessment results. |
| Primary Key | `qia_id` |
| Main References | `project_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 120 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Quality Impact Assessment record. | `00000000-0000-0000-0000-000000000001` |
| 121 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 122 | Part 11 Q1 | `p11_q1` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Part 11 Q1 for the record. | `Yes` |
| 123 | Part 11 Q2 | `p11_q2` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Part 11 Q2 for the record. | `Yes` |
| 124 | Part 11 Q3 | `p11_q3` | `varchar(10)` | N | N | - | Y | `No'` | N | N | N | Y | Part 11 Q3 for the record. | `No` |
| 125 | Part 11 Q4 | `p11_q4` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Part 11 Q4 for the record. | `Yes` |
| 126 | Part 11 Q5 | `p11_q5` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Part 11 Q5 for the record. | `Yes` |
| 127 | Part 11 Q6 | `p11_q6` | `varchar(20)` | N | N | - | Y | `Closed'` | N | N | N | Y | Part 11 Q6 for the record. | `Closed` |
| 128 | Part11 Result | `part11_result` | `text` | N | N | - | Y | - | N | N | N | Y | Part11 Result for the record. | `Sample Part11 Result` |
| 129 | GxP Scope Status | `gxp_scope_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | GxP Scope Status for the record. | `CONFIRMED` |
| 130 | Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Version for the record. | `v1.0` |
| 131 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 132 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 133 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 134 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `DRAFT` |
| 135 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-qia_module_item"></a>
### 12. QIA Module Assessment Item (`qia_module_item`)

| Item | Definition |
|---|---|
| Description | Stores module and process-level GxP assessment answers and results. |
| Primary Key | `qia_module_item_id` |
| Main References | `qia_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 136 | QIA Module Item ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the QIA Module Assessment Item record. | `00000000-0000-0000-0000-000000000001` |
| 137 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | References qia_assessment.qia_id. | `00000000-0000-0000-0000-000000000001` |
| 138 | Module Code | `module_code` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Module Code for the record. | `QM` |
| 139 | Module Name | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Module Name for the record. | `Sample Module Name` |
| 140 | Module Description | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | Module Description for the record. | `Sample Module Description` |
| 141 | Process Code | `process_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Process Code for the record. | `PRC-001` |
| 142 | Process Name | `process_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Process Name for the record. | `Sample Process Name` |
| 143 | Q1 Val | `q1_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q1 Val for the record. | `O` |
| 144 | Q2 Val | `q2_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q2 Val for the record. | `O` |
| 145 | Q3 Val | `q3_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q3 Val for the record. | `X` |
| 146 | Q4 Val | `q4_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q4 Val for the record. | `O` |
| 147 | Q5 Val | `q5_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q5 Val for the record. | `X` |
| 148 | Q6 Val | `q6_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q6 Val for the record. | `X` |
| 149 | Q7 Val | `q7_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q7 Val for the record. | `X` |
| 150 | Q8 Val | `q8_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q8 Val for the record. | `X` |
| 151 | Q9 Val | `q9_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q9 Val for the record. | `X` |
| 152 | Q10 Val | `q10_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | Q10 Val for the record. | `X` |
| 153 | Result Type | `result_type` | `varchar(20)` | N | N | - | Y | `Non-GxP'` | N | N | N | Y | Result Type for the record. | `GxP` |
| 154 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 10:00:00` |
| 155 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 10:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

## VA

<a id="table-vendor_audit"></a>
### 13. Vendor Audit Assessment (`vendor_audit`)

| Item | Definition |
|---|---|
| Description | Stores vendor audit planning, execution, results, and defect counts. |
| Primary Key | `audit_id` |
| Main References | `project_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 156 | Audit ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Vendor Audit Assessment record. | `00000000-0000-0000-0000-000000000001` |
| 157 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 158 | Document Number | `document_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Document Number for the record. | `VA-2026-001` |
| 159 | Vendor Name | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Vendor Name for the record. | `Sample Vendor` |
| 160 | System Name | `system_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | System Name for the record. | `MES v3.2.1` |
| 161 | Audit Type | `audit_type` | `varchar(50)` | N | N | - | Y | `ON_SITE` | N | N | N | Y | Audit Type for the record. | `Sample Audit Type` |
| 162 | Audit Date | `audit_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | Audit Date for the record. | `2026-08-26 00:00:00` |
| 163 | Auditor Name | `auditor_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Auditor Name for the record. | `Sample User` |
| 164 | Audit Result | `audit_result` | `varchar(20)` | N | N | - | Y | `COMPLIANT` | N | N | N | Y | Audit Result for the record. | `Sample Audit Result` |
| 165 | Critical Defects | `critical_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Critical Defects for the record. | `0` |
| 166 | Major Defects | `major_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Major Defects for the record. | `0` |
| 167 | Minor Defects | `minor_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Minor Defects for the record. | `1` |
| 168 | Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Version for the record. | `v1.0` |
| 169 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 170 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 171 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 172 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 173 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 174 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

## URS

<a id="table-requirement"></a>
### 14. User Requirements Specification (`requirement`)

| Item | Definition |
|---|---|
| Description | Stores project URS items with document revision information. |
| Primary Key | `requirement_id` |
| Main References | `project_id, created_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 175 | Requirement ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the User Requirements Specification record. | `00000000-0000-0000-0000-000000000001` |
| 176 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 177 | Item Number | `item_number` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Item Number for the record. | `URS-001` |
| 178 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Category for the record. | `Sample Category` |
| 179 | Title | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 180 | Requirement Text | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Requirement Text for the record. | `Sample Requirement Text` |
| 181 | Regulation | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Regulation for the record. | `21 CFR 11.50` |
| 182 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 183 | Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0'` | N | N | N | Y | Version for the record. | `v1.0` |
| 184 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision Number for the record. | `1` |
| 185 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 186 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 187 | Is RTM Linked | `is_rtm_linked` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | Is RTM Linked for the record. | `False` |
| 188 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 189 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 190 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

## FDS

<a id="table-fds_spec"></a>
### 15. Functional Design Specification (`fds_spec`)

| Item | Definition |
|---|---|
| Description | Stores FDS document headers, versions, and statuses. |
| Primary Key | `fds_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 191 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Functional Design Specification record. | `00000000-0000-0000-0000-000000000001` |
| 192 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 193 | FDS No | `fds_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FDS No for the record. | `VP-SYS-008-20260422` |
| 194 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 195 | Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Version for the record. | `v1.0` |
| 196 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 197 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 198 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 199 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 200 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 201 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 202 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 203 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 204 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-fds_item"></a>
### 16. FDS Detail Item (`fds_item`)

| Item | Definition |
|---|---|
| Description | Stores functional, screen, and detailed design items within an FDS. |
| Primary Key | `fds_item_id` |
| Main References | `fds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 205 | FDS Item ID | `fds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the FDS Detail Item record. | `00000000-0000-0000-0000-000000000001` |
| 206 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | N | N | Y | References fds_spec.fds_id. | `00000000-0000-0000-0000-000000000001` |
| 207 | Item Type | `item_type` | `varchar(20)` | N | N | - | Y | `FUNCTION` | N | N | N | Y | Item Type for the record. | `FUNCTION` |
| 208 | FDS No | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | FDS No for the record. | `FDS-001` |
| 209 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Category for the record. | `Sample Category` |
| 210 | Feature Name | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Feature Name for the record. | `Sample Feature Name` |
| 211 | Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Description for the record. | `Sample Description` |
| 212 | Related Screen | `related_screen` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Related Screen for the record. | `Sample Related Screen` |
| 213 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision Number for the record. | `1` |
| 214 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 215 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 216 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 217 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 218 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 219 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-fds_interface"></a>
### 17. FDS Interface Definition (`fds_interface`)

| Item | Definition |
|---|---|
| Description | Stores system interface, transfer data, frequency, and method definitions. |
| Primary Key | `fds_interface_id` |
| Main References | `fds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 220 | FDS Interface ID | `fds_interface_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the FDS Interface Definition record. | `00000000-0000-0000-0000-000000000001` |
| 221 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | Y | N | Y | References fds_spec.fds_id. | `00000000-0000-0000-0000-000000000001` |
| 222 | Interface ID | `interface_id` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Interface ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 223 | Source System | `source_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Source System for the record. | `MES` |
| 224 | Target System | `target_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Target System for the record. | `SAP S/4HANA` |
| 225 | Interface Data | `interface_data` | `text` | N | N | - | Y | - | N | N | N | Y | Interface Data for the record. | `Sample Interface Data` |
| 226 | Transfer Cycle | `transfer_cycle` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Transfer Cycle for the record. | `Sample Transfer Cycle` |
| 227 | Transfer Method | `transfer_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Transfer Method for the record. | `REST API` |
| 228 | FDS Mapping | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | FDS Mapping for the record. | `FDS-008` |
| 229 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 10:00:00` |
| 230 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 231 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 10:00:00` |
| 232 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 233 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## DQ

<a id="table-dq_assessment"></a>
### 18. Design Qualification Assessment (`dq_assessment`)

| Item | Definition |
|---|---|
| Description | Stores project-level Design Qualification document headers. |
| Primary Key | `dq_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 234 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Design Qualification Assessment record. | `00000000-0000-0000-0000-000000000001` |
| 235 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 236 | DQ No | `dq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | DQ No for the record. | `DQ-VP-SYS-008-20260422` |
| 237 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 238 | Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Version for the record. | `v1.0` |
| 239 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 240 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 241 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 242 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 243 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 244 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 245 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 246 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 247 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-dq_item"></a>
### 19. DQ Assessment Item (`dq_item`)

| Item | Definition |
|---|---|
| Description | Stores qualification results for URS and FDS/DDS design alignment. |
| Primary Key | `dq_item_id` |
| Main References | `dq_id, requirement_id, reviewed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 248 | DQ Item ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the DQ Assessment Item record. | `00000000-0000-0000-0000-000000000001` |
| 249 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | N | N | Y | References dq_assessment.dq_id. | `00000000-0000-0000-0000-000000000001` |
| 250 | Requirement ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | References requirement.requirement_id. | `00000000-0000-0000-0000-000000000001` |
| 251 | URS No | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | URS No for the record. | `URS-001` |
| 252 | URS Description | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | URS Description for the record. | `Sample URS Description` |
| 253 | FDS Mapping | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | FDS Mapping for the record. | `FDS-001` |
| 254 | FDS Feature Name | `fds_feature_name` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | FDS Feature Name for the record. | `Sample FDS Feature Name` |
| 255 | DDS Mapping | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | DDS Mapping for the record. | `DDS-001` |
| 256 | DDS Description | `dds_description` | `text` | N | N | - | N | - | N | N | N | Y | DDS Description for the record. | `User Auth Table Schema` |
| 257 | Result Status | `result_status` | `varchar(20)` | N | N | - | Y | `PENDING_REVIEW` | N | N | N | Y | Result Status for the record. | `PASS` |
| 258 | Reviewed By | `reviewed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 259 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Remarks for the record. | `Sample Remarks` |
| 260 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 261 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 262 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 263 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 264 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## FRA

<a id="table-fra_assessment"></a>
### 20. Functional Risk Assessment (`fra_assessment`)

| Item | Definition |
|---|---|
| Description | Stores project-level Functional Risk Assessment document headers. |
| Primary Key | `fra_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 265 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Functional Risk Assessment record. | `00000000-0000-0000-0000-000000000001` |
| 266 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 267 | FRA No | `fra_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FRA No for the record. | `FRA-VP-SYS-008-20260422` |
| 268 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 269 | Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Version for the record. | `v1.0` |
| 270 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 271 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 272 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 273 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 274 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 275 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 276 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 277 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 278 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-fra_item"></a>
### 21. FRA Risk Item (`fra_item`)

| Item | Definition |
|---|---|
| Description | Stores functional risk scenarios, ratings, and mitigation strategies. |
| Primary Key | `fra_item_id` |
| Main References | `fra_id, requirement_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 279 | FRA Item ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the FRA Risk Item record. | `00000000-0000-0000-0000-000000000001` |
| 280 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | N | N | Y | References fra_assessment.fra_id. | `00000000-0000-0000-0000-000000000001` |
| 281 | Requirement ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | N | N | Y | References requirement.requirement_id. | `00000000-0000-0000-0000-000000000001` |
| 282 | URS No | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | URS No for the record. | `URS-001` |
| 283 | Feature Name | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Feature Name for the record. | `Sample Feature Name` |
| 284 | Risk Scenario | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | Risk Scenario for the record. | `Sample Risk Scenario` |
| 285 | PI Score | `pi_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | PI Score for the record. | `H` |
| 286 | LL Score | `ll_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | LL Score for the record. | `M` |
| 287 | DL Score | `dl_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | DL Score for the record. | `L` |
| 288 | Risk Value | `risk_value` | `integer` | N | N | - | N | `1` | N | N | N | Y | Risk Value for the record. | `1` |
| 289 | Risk Level | `risk_level` | `varchar(20)` | N | N | - | Y | `LOW` | N | Y | N | Y | Risk Level for the record. | `HIGH` |
| 290 | Mitigation Strategy | `mitigation_strategy` | `varchar(20)` | N | N | - | Y | `Test` | N | N | N | Y | Mitigation Strategy for the record. | `Test` |
| 291 | Test Reference | `test_reference` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Test Reference for the record. | `OQ-AT-01` |
| 292 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 293 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 294 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 295 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 296 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 297 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## IQ

<a id="table-iq_assessment"></a>
### 22. Installation Qualification Assessment (`iq_assessment`)

| Item | Definition |
|---|---|
| Description | Stores project-level Installation Qualification document headers. |
| Primary Key | `iq_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 298 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Installation Qualification Assessment record. | `00000000-0000-0000-0000-000000000001` |
| 299 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 300 | IQ No | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ No for the record. | `IQ-VP-SYS-010-20260529` |
| 301 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Installation Qualification` |
| 302 | Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Version for the record. | `v1.0` |
| 303 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 304 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 305 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 306 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Protocol Status for the record. | `APPROVED` |
| 307 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Record Status for the record. | `DRAFT` |
| 308 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 309 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 310 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 311 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 312 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-iq_item"></a>
### 23. IQ Test Item (`iq_item`)

| Item | Definition |
|---|---|
| Description | Stores IQ procedures, outcomes, evidence links, and execution information. |
| Primary Key | `iq_item_id` |
| Main References | `iq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 313 | IQ Item ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the IQ Test Item record. | `00000000-0000-0000-0000-000000000001` |
| 314 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | N | N | Y | References iq_assessment.iq_id. | `00000000-0000-0000-0000-000000000001` |
| 315 | Step No | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Step No for the record. | `1` |
| 316 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 317 | Test Case | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Test Case for the record. | `Sample Test Case` |
| 318 | URS No | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | URS No for the record. | `URS-001` |
| 319 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Test Description for the record. | `Sample Test Description` |
| 320 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Expected Result for the record. | `Sample Expected Result` |
| 321 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `APPROVED` | N | N | N | Y | Protocol Status for the record. | `Sample Protocol Status` |
| 322 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual Result for the record. | `Sample Actual Result` |
| 323 | Qualification Result | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Qualification Result for the record. | `Pass` |
| 324 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 325 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Executed At for the record. | `2026-06-22 00:00:00` |
| 326 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Record Status for the record. | `Sample Record Status` |
| 327 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 328 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 329 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 330 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 331 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## OQ

<a id="table-oq_assessment"></a>
### 24. Operational Qualification Assessment (`oq_assessment`)

| Item | Definition |
|---|---|
| Description | Stores project-level Operational Qualification document headers. |
| Primary Key | `oq_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 332 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Operational Qualification Assessment record. | `00000000-0000-0000-0000-000000000001` |
| 333 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 334 | OQ No | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ No for the record. | `OQ-VP-SYS-010-20260529` |
| 335 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Operational Qualification` |
| 336 | Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Version for the record. | `v1.0` |
| 337 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 338 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 339 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 340 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Protocol Status for the record. | `APPROVED` |
| 341 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Record Status for the record. | `DRAFT` |
| 342 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 343 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 344 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 345 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 346 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-oq_item"></a>
### 25. OQ Test Item (`oq_item`)

| Item | Definition |
|---|---|
| Description | Stores OQ procedures, outcomes, evidence links, and execution information. |
| Primary Key | `oq_item_id` |
| Main References | `oq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 347 | OQ Item ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the OQ Test Item record. | `00000000-0000-0000-0000-000000000001` |
| 348 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | N | N | Y | References oq_assessment.oq_id. | `00000000-0000-0000-0000-000000000001` |
| 349 | Step No | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Step No for the record. | `1` |
| 350 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 351 | Test Case | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Test Case for the record. | `Audit Trail` |
| 352 | URS No | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | URS No for the record. | `URS-001` |
| 353 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Test Description for the record. | `Sample Test Description` |
| 354 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Expected Result for the record. | `Sample Expected Result` |
| 355 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `APPROVED` | N | N | N | Y | Protocol Status for the record. | `Sample Protocol Status` |
| 356 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual Result for the record. | `Sample Actual Result` |
| 357 | Qualification Result | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Qualification Result for the record. | `Pass` |
| 358 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 359 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Executed At for the record. | `2026-06-22 00:00:00` |
| 360 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Record Status for the record. | `Sample Record Status` |
| 361 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 362 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 363 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 364 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 365 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## PQ

<a id="table-pq_assessment"></a>
### 26. Performance Qualification Assessment (`pq_assessment`)

| Item | Definition |
|---|---|
| Description | Stores Performance Qualification document headers and execution plans. |
| Primary Key | `pq_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 366 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Performance Qualification Assessment record. | `00000000-0000-0000-0000-000000000001` |
| 367 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 368 | PQ No | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ No for the record. | `PQ-VP-SYS-008-20260422` |
| 369 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Performance Qualification` |
| 370 | Start Scheduled Date | `start_scheduled_date` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Start Scheduled Date for the record. | `Sample Start Scheduled Date` |
| 371 | Target Completion Date | `target_completion_date` | `date` | N | N | - | N | - | N | N | N | Y | Target Completion Date for the record. | `2024-04-15 00:00:00` |
| 372 | Execution Method | `execution_method` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Execution Method for the record. | `Sample Execution Method` |
| 373 | Primary Executor | `primary_executor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Primary Executor for the record. | `Sample Primary Executor` |
| 374 | Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Version for the record. | `v1.0` |
| 375 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 376 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 377 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 378 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 379 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Protocol Status for the record. | `APPROVED` |
| 380 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Record Status for the record. | `DRAFT` |
| 381 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 382 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 383 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 384 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 385 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-pq_item"></a>
### 27. PQ Test Item (`pq_item`)

| Item | Definition |
|---|---|
| Description | Stores PQ procedures, outcomes, evidence links, and execution information. |
| Primary Key | `pq_item_id` |
| Main References | `pq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 386 | PQ Item ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the PQ Test Item record. | `00000000-0000-0000-0000-000000000001` |
| 387 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | N | N | Y | References pq_assessment.pq_id. | `00000000-0000-0000-0000-000000000001` |
| 388 | Step No | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Step No for the record. | `1` |
| 389 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Category for the record. | `Sample Category` |
| 390 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Test Description for the record. | `Sample Test Description` |
| 391 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Expected Result for the record. | `Sample Expected Result` |
| 392 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual Result for the record. | `Sample Actual Result` |
| 393 | Qualification Result | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Qualification Result for the record. | `Pass` |
| 394 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 395 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Executed At for the record. | `2026-08-26 00:00:00` |
| 396 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 397 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 398 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 399 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 400 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 401 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## RTM

<a id="table-rtm_assessment"></a>
### 28. Requirements Traceability Matrix (`rtm_assessment`)

| Item | Definition |
|---|---|
| Description | Stores project traceability coverage and an approval-time RTM snapshot. |
| Primary Key | `rtm_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 402 | RTM ID | `rtm_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Requirements Traceability Matrix record. | `00000000-0000-0000-0000-000000000001` |
| 403 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 404 | RTM No | `rtm_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | RTM No for the record. | `RTM-VP-SYS-008-20260422` |
| 405 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 406 | Total URS Count | `total_urs_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Total URS Count for the record. | `8` |
| 407 | FRA Link Rate | `fra_link_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | FRA Link Rate for the record. | `88` |
| 408 | IQ Coverage Rate | `iq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | IQ Coverage Rate for the record. | `0` |
| 409 | OQ Coverage Rate | `oq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | OQ Coverage Rate for the record. | `44` |
| 410 | Avg Coverage Rate | `avg_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | Avg Coverage Rate for the record. | `44` |
| 411 | Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Version for the record. | `v1.0` |
| 412 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 413 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 414 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 415 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Status for the record. | `Sample Status` |
| 416 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 417 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 418 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 419 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 420 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-rtm_item"></a>
### 29. RTM Traceability Item (`rtm_item`)

| Item | Definition |
|---|---|
| Description | Stores URS-level traceability results and detailed approval-time snapshots. |
| Primary Key | `rtm_item_id` |
| Main References | `rtm_id, requirement_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 421 | RTM Item ID | `rtm_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the RTM Traceability Item record. | `00000000-0000-0000-0000-000000000001` |
| 422 | RTM ID | `rtm_id` | `uuid` | N | Y | `rtm_assessment.rtm_id` | Y | - | N | N | N | Y | References rtm_assessment.rtm_id. | `00000000-0000-0000-0000-000000000001` |
| 423 | Requirement ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | References requirement.requirement_id. | `00000000-0000-0000-0000-000000000001` |
| 424 | URS No | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | URS No for the record. | `URS-001` |
| 425 | Feature Name | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Feature Name for the record. | `Sample Feature Name` |
| 426 | Requirement Desc | `requirement_desc` | `text` | N | N | - | Y | - | N | N | N | Y | Requirement Desc for the record. | `Sample Requirement Desc` |
| 427 | Adoption Status | `adoption_status` | `varchar(10)` | N | N | - | Y | `O` | N | N | N | Y | Adoption Status for the record. | `O` |
| 428 | FRA Mapping | `fra_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | FRA Mapping for the record. | `N/A` |
| 429 | FDS Mapping | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | FDS Mapping for the record. | `Sample FDS Mapping` |
| 430 | DDS Mapping | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | DDS Mapping for the record. | `Sample DDS Mapping` |
| 431 | IQ Result | `iq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | IQ Result for the record. | `IQ-NEW-07` |
| 432 | OQ Result | `oq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | OQ Result for the record. | `Sample OQ Result` |
| 433 | PQ Result | `pq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | PQ Result for the record. | `N/A` |
| 434 | Item Coverage Rate | `item_coverage_rate` | `numeric(5,2)` | N | N | - | Y | `0` | N | N | N | Y | Item Coverage Rate for the record. | `50` |
| 435 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 436 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 437 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 438 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 439 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## VSR

<a id="table-vsr_assessment"></a>
### 30. Validation Summary Report (`vsr_assessment`)

| Item | Definition |
|---|---|
| Description | Stores the final validation conclusion and summary report. |
| Primary Key | `vsr_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 440 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the Validation Summary Report record. | `00000000-0000-0000-0000-000000000001` |
| 441 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 442 | VSR No | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR No for the record. | `VSR-VP-SYS-008-20260422` |
| 443 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Validation Summary Report` |
| 444 | Overall Conclusion | `overall_conclusion` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Overall Conclusion for the record. | `Sample Overall Conclusion` |
| 445 | Conclusion Remarks | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | Conclusion Remarks for the record. | `Sample Conclusion Remarks` |
| 446 | Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Version for the record. | `v1.0` |
| 447 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number for the record. | `1` |
| 448 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 449 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 450 | Status | `status` | `varchar(20)` | N | N | - | Y | `IN_REVIEW` | N | N | N | Y | Status for the record. | `Sample Status` |
| 451 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 452 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 453 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 454 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 455 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-vsr_item"></a>
### 31. VSR Activity Summary Item (`vsr_item`)

| Item | Definition |
|---|---|
| Description | Stores activity-level document, result, deviation, and approval summaries. |
| Primary Key | `vsr_item_id` |
| Main References | `vsr_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 456 | VSR Item ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Unique identifier for the VSR Activity Summary Item record. | `00000000-0000-0000-0000-000000000001` |
| 457 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | N | N | Y | References vsr_assessment.vsr_id. | `00000000-0000-0000-0000-000000000001` |
| 458 | Activity Code | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Activity Code for the record. | `URS` |
| 459 | Doc No | `doc_no` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Doc No for the record. | `VP-SYS-008-20260422 · URS` |
| 460 | Revision No | `revision_no` | `varchar(20)` | N | N | - | Y | `1` | N | N | N | Y | Revision No for the record. | `2.1` |
| 461 | Execution Date | `execution_date` | `date` | N | N | - | N | - | N | N | N | Y | Execution Date for the record. | `2024-02-15 00:00:00` |
| 462 | Pass Count | `pass_count` | `integer` | N | N | - | N | - | N | N | N | Y | Pass Count for the record. | `125` |
| 463 | Fail Count | `fail_count` | `integer` | N | N | - | N | - | N | N | N | Y | Fail Count for the record. | `0` |
| 464 | Deviation Info | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Deviation Info for the record. | - |
| 465 | Item Status | `item_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | N | N | Y | Item Status for the record. | `Sample Item Status` |
| 466 | Approver Name | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Approver Name for the record. | `Sample User` |
| 467 | Approval Date | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | Approval Date for the record. | `2024-02-20 00:00:00` |
| 468 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-26 00:00:00` |
| 469 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 470 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-26 00:00:00` |
| 471 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 472 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## Workflow

<a id="table-workflow_instance"></a>
### 32. Workflow Instance (`workflow_instance`)

| Item | Definition |
|---|---|
| Description | Stores document review and approval workflow progress. |
| Primary Key | `workflow_instance_id` |
| Main References | `requested_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 473 | Workflow Instance ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Workflow Instance record. | `00000000-0000-0000-0000-000000000001` |
| 474 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Target Table Name for the record. | `fds_spec` |
| 475 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | Target Record ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 476 | Target Version | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Target Version for the record. | `v1.0` |
| 477 | Requested By | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 478 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Requested At for the record. | `2026-08-31 15:00:00` |
| 479 | Workflow Status | `workflow_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Workflow Status for the record. | `IN_PROGRESS` |
| 480 | Current Step Order | `current_step_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Current Step Order for the record. | `1` |
| 481 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completed At for the record. | `2026-08-31 17:00:00` |
| 482 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-31 15:00:00` |
| 483 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 484 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-31 16:00:00` |
| 485 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 486 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-workflow_step"></a>
### 33. Workflow Step (`workflow_step`)

| Item | Definition |
|---|---|
| Description | Stores workflow step order, assignment, status, and completion data. |
| Primary Key | `workflow_step_id` |
| Main References | `workflow_instance_id, assignee_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 487 | Workflow Step ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Workflow Step record. | `00000000-0000-0000-0000-000000000001` |
| 488 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | References workflow_instance.workflow_instance_id. | `00000000-0000-0000-0000-000000000001` |
| 489 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | Step Order for the record. | `1` |
| 490 | Step Type | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Step Type for the record. | `REVIEW` |
| 491 | Step Name | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Step Name for the record. | `Sample Step Name` |
| 492 | Assignee ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 493 | Step Status | `step_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Step Status for the record. | `PENDING` |
| 494 | Due At | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Due At for the record. | `2026-09-02 18:00:00` |
| 495 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completed At for the record. | `2026-09-01 10:00:00` |
| 496 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-08-31 15:00:00` |
| 497 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 498 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-08-31 16:00:00` |
| 499 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 500 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-approval_action"></a>
### 34. Approval Action History (`approval_action`)

| Item | Definition |
|---|---|
| Description | Stores actual review, approval, rejection, submission, and cancellation actions. |
| Primary Key | `approval_action_id` |
| Main References | `workflow_step_id, actor_id, signature_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 501 | Approval Action ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Approval Action History record. | `00000000-0000-0000-0000-000000000001` |
| 502 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | References workflow_step.workflow_step_id. | `00000000-0000-0000-0000-000000000001` |
| 503 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Action Type for the record. | `APPROVE` |
| 504 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 505 | Action Comment | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | Action Comment for the record. | `Sample Action Comment` |
| 506 | Rejection Reason | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | Rejection Reason for the record. | `Sample Rejection Reason` |
| 507 | Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | References electronic_signature.signature_id. | `00000000-0000-0000-0000-000000000001` |
| 508 | Acted At | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Acted At for the record. | `2026-09-01 10:00:00` |
| 509 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-01 10:00:00` |

[↑ Back to top](#dvt-data-table-definition)

---

## Traceability

<a id="table-traceability_link"></a>
### 35. Traceability Link (`traceability_link`)

| Item | Definition |
|---|---|
| Description | Stores traceability relationships among URS, FDS, DDS, DQ, FRA, IQ, OQ, and PQ items. |
| Primary Key | `traceability_link_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 510 | Traceability Link ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Traceability Link record. | `00000000-0000-0000-0000-000000000001` |
| 511 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 512 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Source Entity Type for the record. | `REQUIREMENT` |
| 513 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Source Entity ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 514 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Target Entity Type for the record. | `IQ_ITEM` |
| 515 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Target Entity ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 516 | Link Type | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Link Type for the record. | `VERIFIED_BY` |
| 517 | Link Reason | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | Link Reason for the record. | `Sample Link Reason` |
| 518 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-01 10:00:00` |
| 519 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 520 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-01 10:00:00` |
| 521 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 522 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## DDS

<a id="table-dds_spec"></a>
### 41. Detailed Design Specification (`dds_spec`)

| Item | Definition |
|---|---|
| Description | Stores DDS document headers, versions, and approval statuses. |
| Primary Key | `dds_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 587 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Detailed Design Specification record. | `00000000-0000-0000-0000-000000000001` |
| 588 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 589 | DDS No | `dds_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DDS No for the record. | `DDS-VP-SYS-001` |
| 590 | Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Detailed Design Specification` |
| 591 | Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Version for the record. | `v1.0` |
| 592 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision Number for the record. | `1` |
| 593 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason for the record. | `Sample Revision Reason` |
| 594 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Current Version for the record. | `True` |
| 595 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Status for the record. | `APPROVED` |
| 596 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-01 10:00:00` |
| 597 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 598 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-01 10:00:00` |
| 599 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 600 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-dds_item"></a>
### 42. DDS Detail Item (`dds_item`)

| Item | Definition |
|---|---|
| Description | Stores database, interface, component, security, and batch design details. |
| Primary Key | `dds_item_id` |
| Main References | `dds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 601 | DDS Item ID | `dds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the DDS Detail Item record. | `00000000-0000-0000-0000-000000000001` |
| 602 | DDS ID | `dds_id` | `uuid` | N | Y | `dds_spec.dds_id` | Y | - | N | Y | N | Y | References dds_spec.dds_id. | `00000000-0000-0000-0000-000000000001` |
| 603 | Item No | `item_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Item No for the record. | `DDS-001` |
| 604 | Design Type | `design_type` | `varchar(30)` | N | N | - | Y | `COMPONENT` | N | Y | N | Y | Design Type for the record. | `DATABASE` |
| 605 | Design Name | `design_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Design Name for the record. | `Sample Design Name` |
| 606 | Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Description for the record. | `Sample Description` |
| 607 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Status for the record. | `APPROVED` |
| 608 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `09/01/2026 10:00:00` |
| 609 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 610 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `09/01/2026 10:00:00` |
| 611 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 612 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## Deviation

<a id="table-deviation"></a>
### 43. Deviation (`deviation`)

| Item | Definition |
|---|---|
| Description | Stores deviations, investigation, resolution, and closure approval status. |
| Primary Key | `deviation_id` |
| Main References | `project_id, resolved_by, approved_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 613 | Deviation ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Deviation record. | `00000000-0000-0000-0000-000000000001` |
| 614 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 615 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Source Entity Type for the record. | `OQ_ITEM` |
| 616 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Source Entity ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 617 | Deviation No | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Deviation No for the record. | `DEV-001` |
| 618 | Title | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 619 | Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Description for the record. | `Sample Description` |
| 620 | Severity | `severity` | `varchar(20)` | N | N | - | Y | `MINOR` | N | Y | N | Y | Severity for the record. | `MAJOR` |
| 621 | Deviation Status | `deviation_status` | `varchar(20)` | N | N | - | Y | `OPEN` | N | Y | N | Y | Deviation Status for the record. | `OPEN` |
| 622 | Resolution | `resolution` | `text` | N | N | - | N | - | N | N | N | Y | Resolution for the record. | `Sample Resolution` |
| 623 | Resolved At | `resolved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Resolved At for the record. | `09/03/2026 15:00:00` |
| 624 | Resolved By | `resolved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 625 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Approved At for the record. | `09/03/2026 17:00:00` |
| 626 | Approved By | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 627 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `09/01/2026 10:00:00` |
| 628 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 629 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `09/01/2026 10:00:00` |
| 630 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 631 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## Report

<a id="table-report_generation"></a>
### 44. Report Generation Job (`report_generation`)

| Item | Definition |
|---|---|
| Description | Stores report requests, execution status, failures, retries, and output files. |
| Primary Key | `report_generation_id` |
| Main References | `project_id, requested_by, result_file_id, report_schedule_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 632 | Report Generation ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Report Generation Job record. | `00000000-0000-0000-0000-000000000001` |
| 633 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 634 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Report Type for the record. | `AUDIT_TRAIL` |
| 635 | Report Name | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Report Name for the record. | `Sample Report Name` |
| 636 | Execution Type | `execution_type` | `varchar(20)` | N | N | - | Y | `ON_DEMAND` | N | Y | N | Y | Execution Type for the record. | `ON_DEMAND` |
| 637 | Period From | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Period From for the record. | `2026-08-01 00:00:00+00` |
| 638 | Period To | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Period To for the record. | `2026-08-31 23:59:59+00` |
| 639 | Report Parameters | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | Report Parameters for the record. | `{"action_types":["CREATE","UPDATE"]}` |
| 640 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | Output Format for the record. | `PDF` |
| 641 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Generation Status for the record. | `PENDING` |
| 642 | Requested By | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 643 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Requested At for the record. | `2026-09-02 15:00:00+00` |
| 644 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Started At for the record. | `2026-09-02 15:00:05+00` |
| 645 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completed At for the record. | `2026-09-02 15:01:30+00` |
| 646 | Result File ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | References file_asset.file_id. | `00000000-0000-0000-0000-000000000001` |
| 647 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Retry Count for the record. | `0` |
| 648 | Max Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Max Retry Count for the record. | `3` |
| 649 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next Retry At for the record. | `2026-09-02 15:10:00+00` |
| 650 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Error Code for the record. | `REPORT_FILE_CREATE_FAILED` |
| 651 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Error Message for the record. | `Sample Error Message` |
| 652 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 15:00:00+00` |
| 653 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 654 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 15:01:30+00` |
| 655 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 656 | Report Schedule ID | `report_schedule_id` | `uuid` | N | Y | `report_schedule.report_schedule_id` | N | - | N | Y | N | Y | References report_schedule.report_schedule_id. | `00000000-0000-0000-0000-000000000001` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-report_schedule"></a>
### 50. Report Schedule (`report_schedule`)

| Item | Definition |
|---|---|
| Description | Stores report frequency, period rules, output format, and next execution time. |
| Primary Key | `report_schedule_id` |
| Main References | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 744 | Report Schedule ID | `report_schedule_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Report Schedule record. | `00000000-0000-0000-0000-000000000001` |
| 745 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 746 | Schedule Name | `schedule_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Schedule Name for the record. | `Sample Schedule Name` |
| 747 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Report Type for the record. | `AUDIT_TRAIL` |
| 748 | Schedule Type | `schedule_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Schedule Type for the record. | `MONTHLY` |
| 749 | Schedule Expression | `schedule_expression` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Schedule Expression for the record. | `0 0 1 * *` |
| 750 | Period Type | `period_type` | `varchar(30)` | N | N | - | Y | `PREVIOUS_MONTH` | N | N | N | Y | Period Type for the record. | `PREVIOUS_MONTH` |
| 751 | Report Parameters | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | Report Parameters for the record. | `{"action_types":["CREATE","UPDATE"]}` |
| 752 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | Output Format for the record. | `PDF` |
| 753 | Next Run At | `next_run_at` | `timestamptz` | N | N | - | Y | - | N | Y | N | Y | Next Run At for the record. | `2026-10-01 00:00:00+00` |
| 754 | Last Run At | `last_run_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Last Run At for the record. | `2026-09-01 00:00:00+00` |
| 755 | Last Report Generation ID | `last_report_generation_id` | `uuid` | N | Y | `report_generation.report_generation_id` | N | - | N | Y | N | Y | References report_generation.report_generation_id. | `00000000-0000-0000-0000-000000000001` |
| 756 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Is Active for the record. | `True` |
| 757 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 18:00:00+00` |
| 758 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 759 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 18:00:00+00` |
| 760 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 761 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp; NULL when active. | - |

[↑ Back to top](#dvt-data-table-definition)

---

## AI

<a id="table-ai_generation_job"></a>
### 45. AI Generation Job (`ai_generation_job`)

| Item | Definition |
|---|---|
| Description | Stores AI generation requests, model configuration, execution status, failures, and retries. |
| Primary Key | `ai_job_id` |
| Main References | `project_id, requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 657 | AI Job ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the AI Generation Job record. | `00000000-0000-0000-0000-000000000001` |
| 658 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 659 | Job Type | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Job Type for the record. | `ITEM_GENERATION` |
| 660 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Target Entity Type for the record. | `REQUIREMENT` |
| 661 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Target Entity ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 662 | Model Name | `model_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Model Name for the record. | `GPT-5` |
| 663 | Input Parameters | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Input Parameters for the record. | `JSON` |
| 664 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Generation Status for the record. | `COMPLETED` |
| 665 | Requested By | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 666 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Requested At for the record. | `2026-09-02 00:00:00` |
| 667 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Started At for the record. | `2026-09-02 00:00:00` |
| 668 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completed At for the record. | `2026-09-02 00:00:00` |
| 669 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Retry Count for the record. | `0` |
| 670 | Max Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Max Retry Count for the record. | `3` |
| 671 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next Retry At for the record. | `2026-09-02 15:10:00` |
| 672 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Error Code for the record. | `LLM_TIMEOUT` |
| 673 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Error Message for the record. | `Sample Error Message` |
| 674 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 00:00:00` |
| 675 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 676 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 00:00:00` |
| 677 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-ai_generation_result"></a>
### 46. AI Generation Result (`ai_generation_result`)

| Item | Definition |
|---|---|
| Description | Stores AI-generated result sets and adoption status. |
| Primary Key | `ai_result_id` |
| Main References | `ai_job_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 678 | AI Result ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the AI Generation Result record. | `00000000-0000-0000-0000-000000000001` |
| 679 | AI Job ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | References ai_generation_job.ai_job_id. | `00000000-0000-0000-0000-000000000001` |
| 680 | Result Title | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | Result Title for the record. | `Sample Result Title` |
| 681 | Is Selected | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Is Selected for the record. | `True` |
| 682 | Is Applied | `is_applied` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Is Applied for the record. | `True` |
| 683 | Selected By | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 684 | Selected At | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Selected At for the record. | `2026-09-02 00:00:00` |
| 685 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 00:00:00` |
| 686 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 687 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 00:00:00` |
| 688 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |

[↑ Back to top](#dvt-data-table-definition)

---

<a id="table-ai_result_item"></a>
### 47. AI Result Item (`ai_result_item`)

| Item | Definition |
|---|---|
| Description | Stores detailed AI-generated document sections or validation items. |
| Primary Key | `ai_result_item_id` |
| Main References | `ai_result_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 689 | AI Result Item ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the AI Result Item record. | `00000000-0000-0000-0000-000000000001` |
| 690 | AI Result ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | References ai_generation_result.ai_result_id. | `00000000-0000-0000-0000-000000000001` |
| 691 | Item Order | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Item Order for the record. | `1` |
| 692 | Item Type | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Item Type for the record. | `REQUIREMENT` |
| 693 | Title | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Title for the record. | `Sample Title` |
| 694 | Content | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Content for the record. | `Sample Content` |
| 695 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Target Entity Type for the record. | `REQUIREMENT` |
| 696 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Target Entity ID for the record. | `00000000-0000-0000-0000-000000000001` |
| 697 | Is Selected | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Is Selected for the record. | `True` |
| 698 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 00:00:00` |
| 699 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 700 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 00:00:00` |
| 701 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |

[↑ Back to top](#dvt-data-table-definition)

---

## Notification

<a id="table-notification_delivery"></a>
### 48. Notification Delivery (`notification_delivery`)

| Item | Definition |
|---|---|
| Description | Stores notification recipients, delivery status, failures, and retry history. |
| Primary Key | `notification_delivery_id` |
| Main References | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion Principle | Retain for the applicable auditable period. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal or sensitive information

| No. | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 702 | Notification Delivery ID | `notification_delivery_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the Notification Delivery record. | `00000000-0000-0000-0000-000000000001` |
| 703 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | References validation_project.project_id. | `00000000-0000-0000-0000-000000000001` |
| 704 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id. | `00000000-0000-0000-0000-000000000001` |
| 705 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | N | - | N | Y | N | Y | References workflow_step.workflow_step_id. | `00000000-0000-0000-0000-000000000001` |
| 706 | Notification Type | `notification_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Notification Type for the record. | `OVERDUE` |
| 707 | Delivery Channel | `delivery_channel` | `varchar(20)` | N | N | - | Y | `EMAIL` | N | Y | N | Y | Delivery Channel for the record. | `EMAIL` |
| 708 | Recipient ID | `recipient_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | Y | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 709 | Notification Title | `notification_title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Notification Title for the record. | `Sample Notification Title` |
| 710 | Notification Content | `notification_content` | `text` | N | N | - | Y | - | N | N | N | Y | Notification Content for the record. | `Sample Notification Content` |
| 711 | Delivery Status | `delivery_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Delivery Status for the record. | `PENDING` |
| 712 | Scheduled At | `scheduled_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Scheduled At for the record. | `2026-09-02 18:00:00+00` |
| 713 | Sent At | `sent_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Sent At for the record. | `2026-09-02 18:00:05+00` |
| 714 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Retry Count for the record. | `0` |
| 715 | Max Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Max Retry Count for the record. | `3` |
| 716 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next Retry At for the record. | `2026-09-02 18:10:00+00` |
| 717 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Error Code for the record. | `EMAIL_SEND_TIMEOUT` |
| 718 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Error Message for the record. | `Sample Error Message` |
| 719 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp in UTC. | `2026-09-02 18:00:00+00` |
| 720 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |
| 721 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of the most recent record update in UTC. | `2026-09-02 18:00:05+00` |
| 722 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id. | `00000000-0000-0000-0000-000000000001` |

[↑ Back to top](#dvt-data-table-definition)

---

