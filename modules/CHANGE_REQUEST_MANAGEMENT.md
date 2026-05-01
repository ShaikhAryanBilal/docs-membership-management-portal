# Module Documentation: Change Request Management

## 1. Overview
The Change Request Management module implements a rigorous **Maker-Checker (Dual Control)** governance pattern. It ensures that sensitive member data (including PII and Financial statuses) cannot be modified by regional coordinators directly. Instead, all modifications must be proposed as a "Change Request" and approved by a central authority (General Manager).

## 2. The Maker-Checker Architecture

### A. The "Maker" (Chapter Coordinator)
Coordinators initiate changes via the `MembershipController@storeChangeRequest` method.
- **Drafting State**: Changes are not applied to the record. Instead, the current record is marked as `is_change_request = true` (locked), and the proposed changes are serialized into a JSON object in the `membership_change_requests` table.
- **Notification**: Upon submission, the system triggers a background email to the General Manager to alert them of a pending review.

### B. The "Checker" (General Manager)
The GM reviews the request via the `MembershipChangeRequestController@show` view, which presents a side-by-side comparison of "Current Data" vs. "Proposed Changes."

---

## 3. The Atomic Approval Engine
When the GM clicks "Approve," the system executes a complex, multi-table update logic within the `update` method.

### Logical Sequence:
1.  **Identity Re-Validation**: The system performs a final check for email and ID/Passport collisions (see Section 4).
2.  **User Table Update**: Updates core `users` table if `name` or `email` was changed.
3.  **Membership Core Update**: Updates `form_id`, `chapter_id`, and `is_active` status.
4.  **Automatic Fee Recalculation**: If the `form_id` (Membership Type) changed, the system automatically re-calculates the `fees` column, applying regional rules like the 50% gender discount.
5.  **Metadata (EAV) Sync**: Iterates through the proposed changes and updates `user_meta` and `membership_meta`.
6.  **Selective Encryption**: Each metadata field is checked against an encryption whitelist. PII is encrypted on-the-fly before being committed to the database.

---

## 4. Integrity & Validation Logic

The system handles several high-stakes edge cases during the approval phase:

### A. Decryption-Based Duplicate Check
Since PII (like Passport numbers) is encrypted, a standard SQL `unique` constraint is insufficient.
- **Pattern**: The system retrieves all existing metadata for that key, decrypts each value in memory, and compares it against the proposed change.
- **Resilience**: This prevents two different members from inadvertently being assigned the same government ID number.

### B. Email Collision Handling
When updating the core `users` table, the system wraps the update in a `try-catch` block.
- **Specific Error Catching**: It specifically looks for SQL error code `23000` (Integrity Constraint Violation).
- **Graceful Failure**: If a collision is detected, it reverts the transaction and provides the GM with a user-friendly error message, rather than a system crash.

### C. Membership "Movement" Tracking
The system uses a custom helper, `Helpers::membershipMovement`, to analyze the transition between membership types.
- **Upgrades vs. Downgrades**: It logs whether a member is moving "up" (e.g., Annual to Trustee) or "down," which is vital for organizational reporting.

---

## 5. Automated Communication Loops

The module acts as a communications hub, ensuring all stakeholders are informed:
- **GM Notification**: Notifies the central office of new requests.
- **Approval Email**: Sends `MembershipApprovedMail` with custom chapter details upon approval.
- **Rejection Email**: Sends `MembershipRejectedMail` with the GM's comments if the request is denied.

## 6. Audit & Accountability
Leveraging **Spatie Activity Log**, the module records a "Deep Diff" of every approval.
- **Properties Logged**: `existingData` vs. `changedData`.
- **Metadata Visibility**: Captured both at the User level (profile changes) and Membership level (status/checklist changes).
