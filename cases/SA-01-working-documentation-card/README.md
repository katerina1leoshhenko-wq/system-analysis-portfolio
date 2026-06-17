# SA-01. Working Documentation Card in a Common Data Environment

## Educational System Analysis Portfolio Case

This case demonstrates how a working documentation approval process can be translated into a system-level description of a document card in a Common Data Environment: user roles, user scenarios, statuses, business rules, functional and non-functional requirements, acceptance criteria, data model, JSON examples, validations, error handling and audit log.

This is an educational portfolio case. It does not describe a real production system, does not contain confidential data, and is intended to demonstrate system analysis skills.

---

## 1. Case Overview

In infrastructure projects, working documentation usually goes through several stages: preparation, upload, review, return with comments, revision, approval, signing and transfer to the general contractor.

When the workflow is paper-based or poorly formalized, several problems may occur:

- it is unclear which document version is currently valid;
- comments may be stored separately from the document version;
- statuses may be tracked manually;
- it is difficult to see who performed an action and when;
- the general contractor may receive an outdated version;
- there is no transparent approval history;
- the signing process is difficult to control.

This case does not describe the entire Common Data Environment. It focuses on one key system object — the **working documentation card**.

The document card is used to store and display:

- basic information about the document;
- the current document status;
- the current file version;
- review comments;
- signing status;
- action history;
- user access to the document.

---

## 2. Case Objective

The objective of this case is to describe the system logic of a working documentation card in a Common Data Environment.

The case shows:

- which roles work with the document card;
- which user scenarios are performed;
- what data is stored in the card;
- which statuses the document goes through;
- which business rules restrict user actions;
- which functional requirements the system should support;
- which non-functional requirements are important for this logic;
- which acceptance criteria can be used to verify implementation;
- which entities and relationships can be reflected in an ERD;
- how the document card and related objects may look in JSON;
- which validations and errors the system should handle;
- which events should be recorded in the audit log.

---

## 3. Case Scope

### In Scope

The case covers:

- user roles;
- main user scenarios;
- working documentation card;
- document version;
- review comments;
- document status;
- signing status;
- business rules;
- functional requirements;
- non-functional requirements;
- acceptance criteria;
- data model / ERD;
- JSON examples;
- validations;
- error handling;
- audit log.

### Out of Scope

The case does not cover:

- full Common Data Environment architecture;
- production database implementation;
- full API specification;
- Swagger / OpenAPI specification;
- microservice architecture;
- real integration with an electronic signature service;
- production-level security;
- load testing;
- frontend/backend development;
- real system implementation.

---

## 4. Main User Roles

| Role | Purpose |
| --- | --- |
| Designer | Creates the document card, uploads the first file, sends the document for review, reviews comments and uploads a revised version. |
| Technical Customer / Client Representative | Reviews the document, adds comments, marks blocking comments, returns the document for revision, approves the document, sends it for signing and transfers the signed document to the general contractor. |
| General Contractor | Receives access to the current signed version of the document, views and downloads the document. |
| CDE Administrator | Manages users, roles, access rights and reference data. |
| Viewer | Views the document card without editing rights. |

The signatory may be the same user as the Technical Customer / Client Representative, or may be another authorized user within the signing procedure. In the data model, this user is represented by the `signer_id` field in the `Signature` entity.

---

## 5. Access Rights Matrix

The actual availability of an action depends not only on the user role, but also on the current document status.

| Action | Designer | Technical Customer / Client Representative | General Contractor | CDE Administrator | Viewer |
| --- | --- | --- | --- | --- | --- |
| View document card | Yes | Yes | Yes, if access is granted | Yes | Yes |
| Create document card | Yes | No | No | Yes | No |
| Edit draft card | Yes | No | No | Yes | No |
| Upload first file | Yes, if status is `Draft` | No | No | Yes | No |
| Send document for review | Yes, if status is `Uploaded` or `Revised` | No | No | Yes | No |
| Add comment | No | Yes, if status is `Sent for review` | No | No | No |
| Mark comment as blocking | No | Yes | No | No | No |
| Return document with comments | No | Yes | No | No | No |
| Upload revised version | Yes, if status is `Returned with comments` | No | No | Yes | No |
| Approve document | No | Yes, if there are no open blocking comments | No | No | No |
| Send document for signing | No | Yes, if status is `Approved` | No | Yes | No |
| Retry signing after error | No | Yes | No | Yes | No |
| Transfer signed document to contractor | No | Yes, if status is `Signed` | No | Yes | No |
| Download current version | Yes | Yes | Yes, if access is granted | Yes | Yes |
| View version history | Yes | Yes | Yes, if access is granted | Yes | Yes |
| View audit log | Limited | Yes | Limited | Yes | Limited |

---

## 6. Main User Scenarios

This section describes the main user scenarios performed by participants when working with the working documentation card.

| ID | Scenario | Main Role |
| --- | --- | --- |
| UC-01 | Create a working documentation card | Designer |
| UC-02 | Upload the first working documentation file | Designer |
| UC-03 | Send the document for review | Designer |
| UC-04 | Review the working documentation | Technical Customer / Client Representative |
| UC-05 | Add comments | Technical Customer / Client Representative |
| UC-06 | Return the document with comments | Technical Customer / Client Representative |
| UC-07 | Upload a revised version | Designer |
| UC-08 | Approve the document | Technical Customer / Client Representative |
| UC-09 | Send the document for signing | Technical Customer / Client Representative |
| UC-10 | Process the signing result | System / Signing service |
| UC-11 | Transfer the document to the contractor | Technical Customer / Client Representative |
| UC-12 | Receive approved documentation | General Contractor |
| UC-13 | View document history | All roles with access |

Functional requirements, validations and acceptance criteria are linked to the main user scenarios and business rules. This makes it possible to trace the logic from a user action to system behavior and result verification.

---

## 7. Document Lifecycle Logic

### Document Card Creation and First File Upload

```text
Create document card → Draft
Upload first file → Uploaded
Send for review → Sent for review
```

### Revision Cycle

```text
Sent for review
→ Returned with comments
→ Revised
→ Sent for review
```

### Approval, Signing and Transfer to Contractor

```text
Sent for review
→ Approved
→ Sent for signing
→ Signed
→ Transferred to contractor
→ Archived
```

---

## 8. Document Statuses

| Status | System Value | Meaning |
| --- | --- | --- |
| Draft | `draft` | The card has been created, but the first file has not been uploaded yet, or the document is not ready to be sent. |
| Uploaded | `uploaded` | The first file has been uploaded, but the document has not yet been sent for review. |
| Sent for review | `sent_for_review` | The document has been sent to the Technical Customer / Client Representative. |
| Returned with comments | `returned_with_comments` | The document has been returned to the Designer for revision. |
| Revised | `revised` | The Designer has uploaded a revised version. |
| Approved | `approved` | The document has been approved by the Technical Customer / Client Representative. |
| Sent for signing | `sent_for_signing` | The document has been sent to the signing procedure. |
| Signed | `signed` | The document has been successfully signed. |
| Transferred to contractor | `transferred_to_contractor` | The signed document has been transferred to the general contractor. |
| Archived | `archived` | The document has been moved to archive. |

---

## 9. Signing Statuses

The document status and the signing status are stored separately.

| Signing Status | System Value | Meaning |
| --- | --- | --- |
| Signing not started | `not_started` | The document has not yet been sent for signing. |
| Pending signing | `pending` | The document has been sent for signing and is waiting for the signatory’s action. |
| Signing in progress | `in_progress` | The signing procedure is in progress. |
| Signed | `signed` | The signing procedure has been completed successfully. |
| Rejected | `rejected` | The signatory refused to sign the document. |
| Signing error | `error` | A technical signing error occurred. |
| Expired | `expired` | The signing deadline has expired. |

Example:

```text
status = sent_for_signing
signing_status = error
```

This means that the document is still at the signing stage, but the signing procedure ended with a technical error.

---

## 10. Main Business Rules

| ID | Business Rule |
| --- | --- |
| BR-01 | To create a draft card, project, document code, document title and document type must be filled in. |
| BR-02 | After the card is created, the system sets `status = draft` and `signing_status = not_started`. |
| BR-03 | The first file can be uploaded only to a card with the `Draft` status. |
| BR-04 | After the first file is successfully uploaded, the status changes to `Uploaded`. |
| BR-05 | The document can be sent for review only from the `Uploaded` or `Revised` status. |
| BR-06 | When returning the document with comments, at least one comment must be added. |
| BR-07 | Comments must be linked to a specific document version. |
| BR-08 | The system does not determine comment criticality based on the comment text. |
| BR-09 | The Technical Customer / Client Representative specifies `severity` and `is_blocking` when creating a comment. |
| BR-10 | The document cannot be approved if the current version has an open blocking comment. |
| BR-11 | A revised version can be created only after the document has been returned with comments. |
| BR-12 | Previous document versions are not deleted. |
| BR-13 | When a new version is uploaded, the previous version is no longer current, and the new version becomes the current one. |
| BR-14 | A signed document cannot be edited within the current version. |
| BR-15 | The General Contractor can see only the signed or transferred version of the document. |
| BR-16 | All status changes are recorded in the audit log. |
| BR-17 | Document status and signing status are managed separately. |
| BR-18 | In case of a technical signing error, the document remains at the signing stage. |
| BR-19 | If the signatory rejects the document, `signing_status` becomes `rejected`, and the document status returns to `Approved`. |
| BR-20 | An archived document cannot be edited. |
| BR-21 | `is_archived` is used as a technical flag for archived documents. |

---

## 11. Main Data Model Entities

| Entity | Purpose |
| --- | --- |
| Project | Project to which the working documentation belongs. |
| User | System user. |
| Role | User role. |
| UserRole | Relationship between user and role. |
| Document | Working documentation card. |
| DocumentVersion | Document version. |
| Comment | Comment linked to a specific document version. |
| Signature | Information about the signing procedure. |
| AuditLog | Document action log. |

---

## 12. ERD Model in Text Format

| Entity 1 | Relationship | Entity 2 | Description |
| --- | --- | --- | --- |
| Project | 1 → many | Document | One project can contain many documents. |
| Document | 1 → many | DocumentVersion | One document can have many versions. |
| DocumentVersion | 1 → many | Comment | One document version can have many comments. |
| Document | 1 → many | Comment | One document can have many comments through versions. |
| Document | 1 → many | Signature | One document can have one or more signing attempts. |
| Document | 1 → many | AuditLog | One document can have many audit log entries. |
| User | 1 → many | Document | One user can create many document cards. |
| User | 1 → many | DocumentVersion | One user can upload many document versions. |
| User | 1 → many | Comment | One user can create many comments. |
| User | 1 → many | AuditLog | One user can perform many logged actions. |
| User | many → many | Role | One user can have several roles, and one role can be assigned to many users. |
| User | 1 → many | UserRole | One user can have several user-role records. |
| Role | 1 → many | UserRole | One role can be assigned to many users. |

---

## 13. Document Entity / Document Card

| Field | System Name | Data Type | Required | Description |
| --- | --- | --- | --- | --- |
| Document ID | `document_id` | UUID / string | Yes | Unique document card identifier. |
| Project ID | `project_id` | UUID / string | Yes | Project to which the document belongs. |
| Document code | `document_code` | string | Yes | User-readable working documentation code within the project. |
| Document title | `document_title` | string | Yes | Document name. |
| Document type | `document_type` | enum | Yes | Document type. |
| Project stage | `project_stage` | enum / string | No | Project lifecycle stage. |
| Documentation section | `document_section` | enum / string | No | Section or part of the working documentation package. |
| Current document status | `status` | enum | Yes | General document status. |
| Signing status | `signing_status` | enum | Yes | Status of the signing procedure. |
| Current version | `current_version_id` | UUID / string | No | Reference to the current document version. |
| Comments count | `comments_count` | integer | Yes | Total number of comments. |
| Open comments count | `open_comments_count` | integer | Yes | Number of open comments. |
| Open blocking comments count | `open_blocking_comments_count` | integer | Yes | Number of open comments that block approval. |
| Created by | `created_by` | UUID / string | Yes | User who created the card. |
| Created at | `created_at` | datetime | Yes | Card creation date and time. |
| Updated by | `updated_by` | UUID / string | No | User who made the last change. |
| Updated at | `updated_at` | datetime | Yes | Last update date and time. |
| Transferred at | `transferred_at` | datetime | No | Date and time when the document was transferred to the contractor. |
| Archive flag | `is_archived` | boolean | Yes | Technical flag indicating that the document is archived. |
| Document note | `document_note` | string | No | Additional note. |

### Explanation of `document_code`

`document_code` is not a technical system ID. It is a user-readable document code.

Example:

```text
RD-A08-001
```

Difference between document ID, code and title:

| Field | Meaning | Example |
| --- | --- | --- |
| `document_id` | Technical identifier in the system | DOC-1001 |
| `document_code` | User-readable document code | RD-A08-001 |
| `document_title` | Document name | Working documentation for a road section |

### Explanation of `project_stage` and `document_section`

`project_stage` represents the project stage.

Examples:

- `survey`;
- `design`;
- `construction`;
- `operation`.

`document_section` represents the section or part of the documentation package.

Examples:

- `road_works`;
- `artificial_structures`;
- `traffic_management`;
- `cost_estimate`;
- `other`.

### Explanation of `is_archived`

`is_archived` is a technical flag that helps quickly identify whether the document is archived.

The main source of the document state is still the `status` field.

Example:

```json
{
  "status": "archived",
  "is_archived": true
}
```

This means that the document is archived.

Example:

```json
{
  "status": "uploaded",
  "is_archived": false
}
```

This means that the document is not archived and is currently in the `Uploaded` status.

Rule:

```text
If status = archived, then is_archived = true.
If status is not archived, then is_archived = false.
```

---

## 14. DocumentVersion Entity / Document Version

| Field | System Name | Data Type | Required | Description |
| --- | --- | --- | --- | --- |
| Version ID | `document_version_id` | UUID / string | Yes | Unique version identifier. |
| Document ID | `document_id` | UUID / string | Yes | Document to which the version belongs. |
| Version number | `version_number` | integer | Yes | Sequential version number. |
| Revision | `revision` | string | Yes | Revision identifier, for example R1 or R2. |
| File name | `file_name` | string | Yes | Version file name. |
| File URL | `file_url` | string / URL | Yes | Link to the file in storage. |
| File format | `file_format` | enum / string | Yes | File format. |
| File size | `file_size` | integer | No | File size. |
| File checksum | `file_checksum` | string | No | Technical value used to check file integrity. |
| Uploaded by | `uploaded_by` | UUID / string | Yes | User who uploaded the version. |
| Uploaded at | `uploaded_at` | datetime | Yes | Upload date and time. |
| Is current | `is_current` | boolean | Yes | Flag indicating whether the version is current. |
| Version status | `version_status` | enum | No | Current, outdated or archived. |

### Explanation of `file_checksum`

`file_checksum` is an optional technical field.

It is used to check file integrity. In simple terms, it is a “digital fingerprint” of the file. If the file changes or is damaged, the checksum will be different.

### Explanation of `is_current`

`is_current` shows whether the document version is currently valid.

Example:

| Version | File | `is_current` |
| --- | --- | --- |
| Version 1 | RD-A08-001_R1.pdf | false |
| Version 2 | RD-A08-001_R2.pdf | true |

Rule:

1. When the first version is uploaded, the system sets `is_current = true`.
2. When a new version is uploaded, the system sets `is_current = false` for the previous version.
3. The system sets `is_current = true` for the new version.
4. The `current_version_id` field in the document card must refer to the current version.

---

## 15. Comment Entity / Review Comment

A comment belongs to a specific document and a specific document version.

The system does not determine comment criticality automatically based on the text. The blocking flag is set by the Technical Customer / Client Representative when creating the comment.

| Field | System Name | Data Type | Required | Description |
| --- | --- | --- | --- | --- |
| Comment ID | `comment_id` | UUID / string | Yes | Unique comment identifier. |
| Document ID | `document_id` | UUID / string | Yes | Document to which the comment belongs. |
| Document version ID | `document_version_id` | UUID / string | Yes | Document version to which the comment belongs. |
| Comment text | `comment_text` | string | Yes | Comment content. |
| Severity | `severity` | enum | Yes | Comment severity level. |
| Blocks approval | `is_blocking` | boolean | Yes | Indicates whether the comment blocks document approval. |
| Comment status | `status` | enum | Yes | Comment status. |
| Created by | `created_by` | UUID / string | Yes | User who created the comment. |
| Created at | `created_at` | datetime | Yes | Comment creation date and time. |
| Closed at | `closed_at` | datetime | No | Comment closing date and time. |

### Possible `severity` values

- `blocking` — blocking comment;
- `major` — major comment;
- `minor` — minor comment;
- `info` — informational comment.

### Possible `status` values

- `open` — comment is open;
- `resolved` — correction has been proposed;
- `closed` — comment is closed.

---

## 16. Signature Entity / Signing

The `Signature` entity stores information about a signing attempt or signing procedure.

The signatory may be a user with the “Technical Customer / Client Representative” role, or another authorized user assigned the right to sign the document.

| Field | System Name | Data Type | Required | Description |
| --- | --- | --- | --- | --- |
| Signature ID | `signature_id` | UUID / string | Yes | Unique signing procedure identifier. |
| Document ID | `document_id` | UUID / string | Yes | Document sent for signing. |
| Document version ID | `document_version_id` | UUID / string | Yes | Document version being signed. |
| Signing status | `signing_status` | enum | Yes | Status of the signing procedure. |
| Signatory | `signer_id` | UUID / string | No | User who should sign the document. |
| Sent at | `sent_at` | datetime | No | Date and time when the document was sent for signing. |
| Started at | `started_at` | datetime | No | Date and time when the signatory started the signing procedure. |
| Completed at | `completed_at` | datetime | No | Date and time when signing was successfully completed. |
| Rejected at | `rejected_at` | datetime | No | Date and time when the signatory rejected signing. |
| Error at | `error_at` | datetime | No | Date and time when a technical signing error occurred. |
| Expired at | `expired_at` | datetime | No | Date and time when the signing deadline expired. |
| Rejection reason | `rejection_reason` | string | No | Reason for signing rejection. |
| Error message | `error_message` | string | No | Description of the technical error. |

---

## 17. AuditLog Entity / Audit Log

The audit log records all significant events related to the document card.

| Field | System Name | Data Type | Required | Description |
| --- | --- | --- | --- | --- |
| Audit log ID | `audit_log_id` | UUID / string | Yes | Unique audit log entry identifier. |
| Document ID | `document_id` | UUID / string | Yes | Document related to the action. |
| Document version ID | `document_version_id` | UUID / string | No | Document version, if the action is related to a version. |
| Actor type | `actor_type` | enum | Yes | Who initiated the event: user, system or external service. |
| User ID | `user_id` | UUID / string | No | User who performed the action; may be empty for system events. |
| Action type | `action_type` | enum / string | Yes | Type of event in the audit log. |
| Previous document status | `previous_status` | enum | No | Document status before the action. |
| New document status | `new_status` | enum | No | Document status after the action. |
| Previous signing status | `previous_signing_status` | enum | No | Signing status before the action. |
| New signing status | `new_signing_status` | enum | No | Signing status after the action. |
| Comment | `comment` | string | No | Additional action description. |
| Created at | `created_at` | datetime | Yes | Event date and time. |

### Possible `actor_type` values

- `user` — action performed by a user;
- `system` — action performed by the system;
- `external_service` — event received from an external service, for example a signing service.

### Possible `action_type` values

- `document_created` — card created;
- `first_file_uploaded` — first file uploaded;
- `document_sent_for_review` — document sent for review;
- `comment_added` — comment added;
- `document_returned_with_comments` — document returned with comments;
- `revised_version_uploaded` — revised version uploaded;
- `document_approved` — document approved;
- `document_sent_for_signing` — document sent for signing;
- `signing_started` — signing started;
- `signing_completed` — signing completed successfully;
- `signing_rejected` — signing rejected;
- `signing_error` — signing error;
- `signing_expired` — signing deadline expired;
- `document_transferred_to_contractor` — document transferred to contractor;
- `document_archived` — document archived.

---

## 18. Functional Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-01 | The system shall allow a user with the `Designer` role to create a working documentation card. | High |
| FR-02 | The system shall automatically assign a unique `document_id` to the card. | High |
| FR-03 | The system shall allow the user to fill in the required draft card fields: project, document code, document title and document type. | High |
| FR-04 | After the card is created, the system shall set the document status to `Draft`. | High |
| FR-05 | After the card is created, the system shall set `signing_status = not_started`. | High |
| FR-06 | The system shall allow the Designer to upload the first working documentation file to a card with the `Draft` status. | High |
| FR-07 | After the first file is successfully uploaded, the system shall create the first document version. | High |
| FR-08 | After the first file is successfully uploaded, the system shall change the document status to `Uploaded`. | High |
| FR-09 | The system shall store the file name, format, size, file link and checksum. | Medium |
| FR-10 | The system shall allow the Designer to send the document for review if the document status is `Uploaded` or `Revised`. | High |
| FR-11 | Before sending the document for review, the system shall check that required data and the file are present. | High |
| FR-12 | After the document is successfully sent for review, the system shall change the document status to `Sent for review`. | High |
| FR-13 | The system shall allow the Technical Customer / Client Representative to add comments to the current document version. | High |
| FR-14 | Each comment shall be linked to a specific document and a specific document version. | High |
| FR-15 | When creating a comment, the Technical Customer / Client Representative shall specify `severity` and `is_blocking`. | High |
| FR-16 | The system shall prohibit document approval if the current version has open blocking comments. | High |
| FR-17 | The system shall allow the document to be returned with comments only if at least one comment exists. | High |
| FR-18 | The system shall allow the Designer to upload a revised version only if the document status is `Returned with comments`. | High |
| FR-19 | After a revised version is uploaded, the system shall create a new document version. | High |
| FR-20 | The system shall preserve previous document versions in the version history. | High |
| FR-21 | The system shall allow the Technical Customer / Client Representative to approve the document if there are no open blocking comments. | High |
| FR-22 | After approval, the system shall change the document status to `Approved`. | High |
| FR-23 | The system shall allow the Technical Customer / Client Representative to send an approved document for signing. | High |
| FR-24 | When the document is sent for signing, the system shall change the document status to `Sent for signing` and the signing status to `pending`. | High |
| FR-25 | The system shall store document status separately from signing status. | High |
| FR-26 | When signing is completed successfully, the system shall change the document status to `Signed` and the signing status to `signed`. | High |
| FR-27 | If the signatory rejects the document, the system shall set `signing_status = rejected` and return the document status to `Approved`. | Medium |
| FR-28 | If a technical signing error occurs, the system shall set `signing_status = error` without automatically returning the document for revision. | High |
| FR-29 | The system shall allow the document to be sent for signing again after a technical error or signing expiration. | Medium |
| FR-30 | The system shall allow only signed documents to be transferred to the contractor. | High |
| FR-31 | After the document is transferred to the contractor, the system shall change the document status to `Transferred to contractor`. | High |
| FR-32 | The system shall record all significant actions in the audit log. | High |

---

## 19. Non-Functional Requirements

| ID | Requirement | Category | Priority |
| --- | --- | --- | --- |
| NFR-01 | Access to the document card shall be determined by the user role and project access rights. | Security / Access | High |
| NFR-02 | The user shall see only documents to which they have access. | Security / Access | High |
| NFR-03 | The General Contractor shall not see documents before access is granted to the signed or transferred version. | Security / Access | High |
| NFR-04 | All actions affecting document status shall be logged. | Audit | High |
| NFR-05 | All actions affecting signing status shall be logged. | Audit | High |
| NFR-06 | Document version history shall be preserved and available to users with appropriate access rights. | Data integrity | High |
| NFR-07 | Comments shall be stored in relation to the document version. | Data integrity | High |
| NFR-08 | The system shall prevent manual document status changes outside allowed transitions. | Process control | High |
| NFR-09 | The system shall prevent manual signing status changes outside the allowed signing logic. | Process control | High |
| NFR-10 | Error messages shall be understandable to users and explain why an action is unavailable. | Usability | Medium |
| NFR-11 | In case of a technical signing error, the system shall preserve the current document state and display the option to retry signing. | Reliability / Error handling | High |
| NFR-12 | A signed document shall not be editable within the current version. | Data integrity | High |
| NFR-13 | An archived document shall not be editable. | Data integrity | High |
| NFR-14 | The audit log shall store either the user or the actor type that initiated the event. | Audit | High |

---

## 20. Acceptance Criteria

### AC-01. Document Card Creation

1. A user with the `Designer` role can create a working documentation card.
2. The system displays the required fields: project, document code, document title and document type.
3. If a required field is not filled in, the system displays an error message.
4. After the card is saved, the system assigns a unique `document_id`.
5. The new card receives the `Draft` status.
6. The signing status is set to `not_started`.
7. The working documentation file is not required to create a draft card.
8. The card creation action is recorded in the audit log.

### AC-02. First Working Documentation File Upload

1. A user with the `Designer` role can upload the first file to the document card.
2. First file upload is available only if the card is in the `Draft` status.
3. After the file is successfully uploaded, the system stores the file name, format, size and file link.
4. After the first file is successfully uploaded, the system creates the first document version.
5. After the first file is successfully uploaded, the system changes the document status from `Draft` to `Uploaded`.
6. If the file format is not supported, the system displays an error message and does not change the document status.
7. The file upload action is recorded in the audit log.

### AC-03. Sending the Document for Review

1. The `Send for review` action is available only to a user with the `Designer` role.
2. The action is available only if the document status is `Uploaded` or `Revised`.
3. The system checks that required data and the working documentation file are present.
4. If required data is missing, the system does not change the document status and displays an error message.
5. After the document is successfully sent for review, the status changes to `Sent for review`.
6. The system creates an audit log entry.
7. The Technical Customer / Client Representative receives the document for review.

### AC-04. Returning the Document with Comments

1. The `Return with comments` action is available only to a user with the `Technical Customer / Client Representative` role.
2. The action is available only for a document in the `Sent for review` status.
3. The system requires at least one comment to be added.
4. When adding a comment, the Technical Customer / Client Representative must fill in the comment text.
5. When adding a comment, the Technical Customer / Client Representative must specify the `severity` level.
6. When adding a comment, the Technical Customer / Client Representative must specify whether the comment blocks approval using the `is_blocking` flag.
7. After the document is returned, the document status changes to `Returned with comments`.
8. The comment is stored in relation to the document and the current document version.
9. The action is recorded in the audit log.

### AC-05. Revised Version Upload

1. The `Upload revised version` action is available only to a user with the `Designer` role.
2. The action is available only if the document status is `Returned with comments`.
3. After upload, the system creates a new document version.
4. The previous version remains available in the version history.
5. Comments linked to the previous version remain available in the history.
6. The current version number is increased.
7. The new version becomes the current version.
8. The document status changes to `Revised`.
9. The revised version upload is recorded in the audit log.

### AC-06. Document Approval

1. The `Approve` action is available only to a user with the `Technical Customer / Client Representative` role.
2. The action is available only for a document in the `Sent for review` status.
3. The system checks whether the current document version has open blocking comments.
4. If at least one open comment with `is_blocking = true` exists, the system does not allow document approval.
5. If there are no blocking comments, the system allows document approval.
6. The system does not determine comment criticality automatically based on the text.
7. After approval, the document status changes to `Approved`.
8. The system records the approval in the audit log.

### AC-07. Sending the Document for Signing

1. The `Send for signing` action is available only to a user with the `Technical Customer / Client Representative` role.
2. The action is available only for a document in the `Approved` status.
3. After the document is sent, the document status changes to `Sent for signing`.
4. The signing status changes to `pending`.
5. The system creates an audit log entry.

### AC-08. Successful Document Signing

1. The system receives an event that signing has been completed successfully.
2. The signing status changes to `signed`.
3. The document status changes to `Signed`.
4. The document card displays signing information.
5. The signing event is recorded in the audit log.
6. After signing, the document is no longer editable within the current version.

### AC-09. Signing Error, Rejection or Expiration

1. If a technical error occurs, the signing status changes to `error`.
2. In case of a technical error, the document status remains `Sent for signing`.
3. The system displays a clear error message to the user.
4. The system allows signing to be retried if the error is technical.
5. If the signatory rejects signing, the signing status changes to `rejected`.
6. If the signatory rejects signing, the document status returns to `Approved`.
7. If the signing deadline expires, the signing status changes to `expired`.
8. If the signing deadline expires, the document status returns to `Approved`.
9. The error, rejection or expiration event is recorded in the audit log.

### AC-10. Document Transfer to Contractor

1. The `Transfer to contractor` action is available only for a document in the `Signed` status.
2. After transfer, the document status changes to `Transferred to contractor`.
3. The General Contractor receives access to the current signed document version.
4. The General Contractor can view and download the document.
5. The transfer event is recorded in the audit log.
6. The transfer date and time are stored in the document card.

---

## 21. Validations

| ID | Action | Validation | Error Message |
| --- | --- | --- | --- |
| VAL-01 | Card creation | Project is filled in | Select a project |
| VAL-02 | Card creation | Document code is filled in | Fill in the document code |
| VAL-03 | Card creation | Document title is filled in | Fill in the document title |
| VAL-04 | Card creation | Document type is selected | Select the document type |
| VAL-05 | User action | User is active | User is inactive or access is blocked |
| VAL-06 | First file upload | Card is in the `Draft` status | The first file can be uploaded only to a card with the Draft status |
| VAL-07 | First file upload | File is selected | Upload the working documentation file |
| VAL-08 | First file upload | File format is supported | File format is not supported |
| VAL-09 | First file upload | Revision is specified | Specify the document revision |
| VAL-10 | Sending for review | Document status is `Uploaded` or `Revised` | The document cannot be sent for review from the current status |
| VAL-11 | Sending for review | File is uploaded | Upload the working documentation file |
| VAL-12 | Return with comments | At least one comment has been added | Add at least one comment before returning the document |
| VAL-13 | Adding comment | Comment text is filled in | Fill in the comment text |
| VAL-14 | Adding comment | Severity level is specified | Specify the comment severity level |
| VAL-15 | Adding comment | `is_blocking` flag is specified | Specify whether the comment blocks document approval |
| VAL-16 | Approval | There are no open blocking comments for the current version | The document cannot be approved: there are open blocking comments |
| VAL-17 | Revised version upload | Document status is `Returned with comments` | A revised version can be uploaded only after the document has been returned with comments |
| VAL-18 | Sending for signing | Document status is `Approved` | Only an approved document can be sent for signing |
| VAL-19 | Transfer to contractor | Document status is `Signed` | Only a signed document can be transferred to the contractor |
| VAL-20 | Document editing | Document is not signed | A signed document cannot be changed |
| VAL-21 | Editing archived document | Document must not be archived: `is_archived = false` | An archived document cannot be edited |

---

## 22. Error Handling

### Data Entry Errors

| Situation | System Behavior |
| --- | --- |
| Required field is empty | The system does not save the action and displays an error message. |
| File is not selected | The system does not upload the file and asks the user to select a file. |
| Revision is not specified | The system does not upload the version and asks the user to specify the revision. |
| Comment text is empty | The system does not save the comment. |
| Blocking flag is not specified | The system does not save the comment. |

### Access Errors

| Situation | System Behavior |
| --- | --- |
| User is inactive | The system prohibits actions. |
| User does not have the required role | The system hides the action or prohibits execution. |
| General Contractor tries to open the document before signing | The system does not show the document or restricts access. |
| Viewer tries to edit the card | The system prohibits the action. |
| User tries to change access rights without administrator role | The system prohibits the action. |

### Status Errors

| Situation | System Behavior |
| --- | --- |
| User tries to send a draft for review without a file | The system prohibits the action. |
| User tries to approve a document with an open blocking comment | The system prohibits approval. |
| User tries to upload a revised version when the document is not in the `Returned with comments` status | The system prohibits the action. |
| User tries to transfer an unsigned document to the contractor | The system prohibits the action. |
| User tries to edit a signed document | The system prohibits the action. |
| User tries to edit an archived document | The system prohibits the action. |

### Signing Errors

| Situation | System Behavior |
| --- | --- |
| Signing service is unavailable | The system records the error, displays a message and allows retry. |
| Signatory rejects signing | The system sets `signing_status = rejected`, returns `status = approved` and suggests further actions. |
| Signing deadline expires | The system sets `signing_status = expired`, returns `status = approved` and allows retry. |
| Signing is successful | The system sets `status = signed` and `signing_status = signed`. |

---

## 23. JSON Example: Document Card After First File Upload

```json
{
  "document_id": "DOC-1001",
  "project_id": "PRJ-2026-01",
  "document_code": "RD-A08-001",
  "document_title": "Working documentation for a road section",
  "document_type": "working_documentation",
  "project_stage": "construction",
  "document_section": "road_works",
  "status": "uploaded",
  "signing_status": "not_started",
  "current_version_id": "DOCV-2001",
  "comments_count": 0,
  "open_comments_count": 0,
  "open_blocking_comments_count": 0,
  "created_by": "USER-203",
  "created_at": "2026-04-15T10:20:00Z",
  "updated_by": "USER-203",
  "updated_at": "2026-04-15T10:30:00Z",
  "transferred_at": null,
  "is_archived": false,
  "document_note": "Initial upload of the working documentation package",
  "current_version": {
    "document_version_id": "DOCV-2001",
    "version_number": 1,
    "revision": "R1",
    "file_name": "RD-A08-001_R1.pdf",
    "file_format": "PDF",
    "file_size": 15400000,
    "file_checksum": "a4f8c2d9e1",
    "file_url": "https://cde.example.com/files/RD-A08-001_R1.pdf",
    "uploaded_by": "USER-203",
    "uploaded_at": "2026-04-15T10:30:00Z",
    "is_current": true
  }
}
```

---

## 24. JSON Example: Blocking Comment

```json
{
  "comment_id": "COM-3001",
  "document_id": "DOC-1001",
  "document_version_id": "DOCV-2001",
  "comment_text": "The required approval sheet is missing",
  "severity": "blocking",
  "is_blocking": true,
  "status": "open",
  "created_by": "USER-315",
  "created_at": "2026-04-16T09:15:00Z",
  "closed_at": null
}
```

This comment blocks approval because:

```text
is_blocking = true
status = open
```

---

## 25. JSON Example: Audit Log Entry

```json
{
  "audit_log_id": "LOG-7001",
  "document_id": "DOC-1001",
  "document_version_id": "DOCV-2001",
  "actor_type": "user",
  "user_id": "USER-203",
  "action_type": "document_sent_for_review",
  "previous_status": "uploaded",
  "new_status": "sent_for_review",
  "previous_signing_status": "not_started",
  "new_signing_status": "not_started",
  "comment": "The document was sent to the Technical Customer / Client Representative for review",
  "created_at": "2026-04-15T10:45:00Z"
}
```

---

## 26. JSON Example: Technical Signing Error

```json
{
  "audit_log_id": "LOG-7011",
  "document_id": "DOC-1001",
  "document_version_id": "DOCV-2002",
  "actor_type": "external_service",
  "user_id": null,
  "action_type": "signing_error",
  "previous_status": "sent_for_signing",
  "new_status": "sent_for_signing",
  "previous_signing_status": "in_progress",
  "new_signing_status": "error",
  "comment": "The signing service is temporarily unavailable",
  "created_at": "2026-04-18T14:05:00Z"
}
```

---

## 27. Recommended Repository Structure

```text
system-analysis-portfolio/
│
├── README.md
│
└── cases/
    └── SA-01-working-documentation-card/
        ├── README.md
        └── examples/
            ├── document_card_uploaded.json
            ├── blocking_comment.json
            ├── audit_log_entry.json
            └── signing_error_log.json
```

---

## 28. What This Case Demonstrates

This case demonstrates the following system analysis skills:

- identifying a system entity;
- describing an object card;
- describing user roles;
- describing access rights;
- describing main user scenarios;
- describing the document lifecycle;
- describing statuses and transitions;
- defining business rules;
- separating document status from signing status;
- writing functional requirements;
- writing non-functional requirements;
- defining acceptance criteria;
- describing an ERD model;
- preparing JSON examples;
- describing validations;
- describing error handling;
- describing audit log logic;
- understanding the relationship between document, version, comment, signing and action history.

---

## 29. Possible Next Steps

The case can be extended in future versions by adding:

- API specification for main actions;
- sequence diagram for the signing scenario;
- separate notification scenario;
- document lifecycle diagram;
- graphical ERD diagram;
- JSON examples as separate files;
- PDF presentation or portfolio page.

---

## 30. Final Value of the Case

The **SA-01. Working Documentation Card in a Common Data Environment** case shows how a document approval business process can be transformed into a system-level description.

It demonstrates the analyst’s ability to:

- understand the domain context;
- identify a system object;
- describe roles and scenarios;
- define data and statuses;
- formulate business rules;
- prepare requirements;
- define acceptance criteria;
- connect entities in a data model;
- prepare JSON examples;
- describe validations, errors and audit log.

This case can be used as a portfolio artifact for the following roles:

- Junior Business Analyst;
- Junior System Analyst;
- Business / System Analyst;
- Requirements Analyst;
- IT Analyst.
