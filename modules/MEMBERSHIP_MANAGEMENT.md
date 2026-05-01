# Module Documentation: Membership Management

## 1. Overview
The Membership Management module is the administrative core of the system. It manages the full lifecycle of a member—from activation to succession or termination. It handles complex relationships between users, chapters, and financial records while ensuring strict data privacy and regional isolation.

## 2. Core Functional Components

### A. High-Performance Search & Discovery
The system uses a custom-optimized **DataTables** implementation to handle large datasets across multiple joins.
- **Architecture**: A single `memberships_data_table` query joins `users`, `memberships`, `membership_types`, `chapters`, and `membership_statuses`.
- **Dynamic Scoping**: Queries are automatically scoped based on the administrator's **Chapter Access** and **Gender Access** permissions.
- **Formatted Identifiers**: Leverages `DB::raw` to format membership IDs with global prefixes (e.g., `ORG-XXXX`) directly in the database layer for maximum performance.

### B. The EAV Data Strategy
To support 15+ different membership types with varying documentation requirements, the system uses an **Entity-Attribute-Value (EAV)** model.
- **Hybrid Storage**: Core data (ID, Type, Status) is stored in normalized columns, while type-specific metadata is stored in `user_meta` and `membership_meta`.
- **Selective Decryption**: For performance, non-sensitive fields used for filtering (City, Country, Gender) are stored as plain text. PII (Passport, DOB, Phone) is decrypted only during the "View" action using `Helpers::decryptValue()`.

---

## 3. Advanced Succession & Transfer Logic
One of the most complex features is the **Trustee & Patron Succession** system, which manages the transfer of membership ownership across generations.

### The Transfer Lifecycle:
1.  **Nominee Assignment**: A primary member assigns a `Nominee` (successor).
2.  **Ownership History**: The system maintains a JSON-based audit trail (`ownership_history`) tracking every previous owner, their email, and the transfer timestamp.
3.  **Governance Constraints**:
    *   **Trustee Memberships**: Limited to **two** lifetime transfers.
    *   **Patron Memberships**: Limited to **one** lifetime transfer.
4.  **Atomic Swap**: During a transfer, the system atomically swaps the `user_id` of the membership record, archives the previous owner in the history log, and clears the nominee field for the next generation.

---

## 4. Membership Change Request Lifecycle
To maintain data integrity, Chapter Coordinators cannot modify member data directly. Instead, they use a structured **Change Request** workflow.

### The Workflow:
1.  **Initiation**: Coordinator proposes changes (e.g., upgrading a membership type or updating an email).
2.  **State Protection**: The membership record is marked `is_change_request = true`, locking it from further edits.
3.  **Drafting**: Proposed changes are stored in a `changes` JSON column in the `MembershipChangeRequest` table.
4.  **Notification**: The General Manager is automatically notified via an `ActionRequiredMembershipChangeRequestMail`.
5.  **Review & Audit**: The GM reviews the "Current vs. Proposed" data. Upon approval, the system atomically applies the changes to the `users` and `user_meta` tables and logs the entire diff.

---

## 5. Relationships & Linkages
The module manages several complex entity relationships:
- **Spouse Linking**: Connects "Individual Life" memberships to their spouses, allowing for joint benefit tracking.
- **Chapter Isolation**: Ensures that a Chapter Coordinator in "Region A" has no visibility into the records of "Region B."
- **Financial Status Linkage**: Real-time linkage to the **Payment Module**, where membership status (e.g., `Active` vs. `Pending`) is often driven by the verification of payment records.

---

## 6. Edge Case Handling

| Edge Case | Solution |
| :--- | :--- |
| **Email Collision on Update** | The Change Request validator cross-references all existing users (including encrypted records) before allowing a proposed email change. |
| **Max Transfers Reached** | The `get_nominee_data` method calculates history depth and disables the transfer UI if the jurisdictional limit (1 or 2 transfers) is reached. |
| **Gender-Restricted Access** | Chapter Coordinators assigned a "Female-only" access level are restricted via SQL `whereExists` clauses on `user_meta` during all search and view operations. |
| **Legacy Data Conflicts** | The system supports a `data_source` flag to distinguish between records created in the Portal vs. those imported from legacy migration staging. |
