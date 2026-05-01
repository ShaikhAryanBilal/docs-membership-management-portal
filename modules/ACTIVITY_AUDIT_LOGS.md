# Module Documentation: Administrative Activity Audit Logs

## 1. Overview
The Activity Audit module is the system's "Black Box," providing an immutable record of all administrative actions. It is designed to ensure maximum accountability across regional chapters and to provide a forensic trail for financial and data-integrity investigations.

## 2. Multi-Dimensional Logging Strategy

The system utilizes **Spatie Activity Log** to capture events across three primary dimensions:

### A. Core Action Logs
Captures standard CRUD operations on primary entities:
- **`member_update`**: Tracks changes to the core `users` and `memberships` tables.
- **`membership_transfer`**: Specialized log for succession events (Trustee/Patron transfers).
- **`membership_change_request`**: Records the lifecycle of proposals (Approved/Rejected/Requested).

### B. Communication & System Logs
Tracks the digital footprint of the system's automated processes:
- **`mail_sent` / `mail_failed`**: Essential for verifying that payment links and approval notices reached the members.
- **`payment_reminder_process`**: Tracks the execution of scheduled dunning jobs.
- **`corporate-lead-to-membership`**: Logs the atomic transaction details during lead conversion.

### C. EAV Metadata Audits
Because much of the member profile is stored in EAV (Meta) tables, the system uses custom logging to capture "Property Diffs":
- **`member_meta` / `membership_meta`**: Logs the state change of individual profile keys (e.g., changing a Passport number or Gender).

---

## 3. The Dynamic Diff-Formatting Engine

To make JSON audit trails human-readable, the `ActivityLogsController` implements a specialized **Log Inspector** logic.

### Logic Flow:
1.  **JSON Decoding**: Safely extracts the `properties` column.
2.  **Structure Identification**:
    *   **EAV Pattern**: If `existingData` and `changedData` are found, it generates a list of "Changed Keys."
    *   **Model Pattern**: If `attributes` and `old` are found, it identifies direct model attribute updates.
    *   **Permission Pattern**: Specifically for Chapter assignments, it calculates the **Symmetric Difference** to identify which jurisdictions were "Added" or "Removed" from a coordinator's profile.
3.  **Sanitization**: All output is escaped (`e()`) to prevent XSS during audit viewing.

---

## 4. Accountability & Search Discovery

The module provides high-speed discovery tools for system auditors:

- **Causer Tracking**: Every log is linked to a `causer_id` (the Admin who performed the action). If the action was triggered by a cron job or webhook, it is marked as `System`.
- **Subject Indexing**: Logs are indexed by `subject_id`, allowing auditors to find the entire history of a single Membership Record across multiple years.
- **Regional Filtering**: In alignment with the system's RBAC, Chapter Coordinators are restricted to viewing logs relevant to their assigned jurisdictions.

---

## 5. Audit Use Cases

| Scenario | Targeted Log Name | Key Property to Inspect |
| :--- | :--- | :--- |
| **Financial Discrepancy** | `payment_update` | `amount` or `status` in attributes |
| **Succession Dispute** | `membership_transfer` | `from_user` vs `to_user` |
| **Communication Failure** | `mail_failed` | `error` or `recipient_email` |
| **Permission Escalation** | `user_chapters_updated` | `Added` vs `Removed` chapters |
| **Lead Processing Check** | `corporate-lead-to-membership` | `entry_id` or `new_user_email` |
