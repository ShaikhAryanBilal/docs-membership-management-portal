# Module Documentation: Administrative User Management & RBAC

## 1. Overview
The User Management module governs the administrative hierarchy of the portal. It implements a sophisticated **Role-Based Access Control (RBAC)** system combined with **Jurisdictional Mapping** to ensure that regional coordinators operate only within their assigned geographical and topical boundaries.

## 2. Hierarchical Role Architecture

The system manages four primary administrative roles, distinct from the standard "Member" role:
- **Super Admin**: Full system access, including global settings and user management.
- **General Manager (GM)**: High-level oversight with approval authority for Change Requests.
- **Chapter Coordinator**: Regional authority restricted to specific Chapters and Membership Types.
- **Office Bearer**: Similar to Coordinator, often used for regional leadership with viewing or limited processing rights.

---

## 3. Jurisdictional Permission Scoping

For the `Coordinator` and `Office Bearer` groups, the system enforces a three-dimensional permission matrix:

### A. Geographical Scoping (Chapters)
Coordinators are linked to specific **Chapters** via a many-to-many relationship. This scoping is enforced at the database level across the Membership, Lead, and Payment modules.

### B. Topical Scoping (Membership Types)
Coordinators are restricted to managing specific **Membership Types** (e.g., a "Youth Coordinator" may only have access to "Youth Membership" records).

### C. Demographic Scoping (Gender Access)
A unique governance constraint where an administrator can be restricted to viewing only **Male**, **Female**, or **Both** genders. This is enforced via `whereExists` clauses on the `user_meta` table during data retrieval.

---

## 4. Security & Access Governance

### A. Atomic Role Transitions
To prevent "Permission Leakage," the system implements an **Atomic Detachment** logic. If an administrator's role is changed away from a Coordinator group, the system automatically detaches all assigned Chapters and Membership Types in a single transaction, ensuring no residual access remains.

### B. Forced Session Termination
When an account is deactivated (`is_active = 0`), the system performs a multi-step security sweep:
1.  **Token Revocation**: Deletes all Sanctum/Passport API tokens.
2.  **Session Purge**: Directly deletes all records for that `user_id` from the `sessions` table, forcing an immediate logout across all active browser instances.
3.  **Audit Event**: Logs the deactivation event under a specialized `user_deactivation` audit log.

---

## 5. Audit & Accountability

The module captures high-fidelity audit trails for administrative changes:
- **`chapters` Log**: A specialized audit log that records the **Symmetric Difference** between old and new chapter assignments.
- **Data Discovery**: The User Management interface leverages DataTables with a "Self-Exclusion" rule, ensuring administrators cannot accidentally modify their own roles or suspend their own accounts.

## 6. Permission Matrix Summary

| Role | Chapters | Mem. Types | Gender Access | Change Requests |
| :--- | :--- | :--- | :--- | :--- |
| **Super Admin** | Global | Global | Both | Full Access |
| **General Manager** | Global | Global | Both | Review & Approve |
| **Coordinator** | Assigned Only | Assigned Only | Restricted | Propose Only |
| **Office Bearer** | Assigned Only | Assigned Only | Restricted | View Only |
