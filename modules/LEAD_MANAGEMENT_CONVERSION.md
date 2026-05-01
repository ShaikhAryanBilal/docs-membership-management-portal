# Module Documentation: Lead Management & Conversion Engine

## 1. Overview
The Lead Management module is the entry point for all membership applications. It bridges the gap between external application captures (WordPress/FluentForms) and the core Laravel membership database. This module is responsible for fetching, filtering, refining, and atomically converting raw "leads" into structured "memberships."

## 2. Technical Architecture

### A. Lead Sourcing (External API Integration)
Leads are fetched from a remote WordPress instance via a dedicated API bridge.
- **Transport**: `Illuminate\Support\Facades\Http` (cURL wrapper).
- **Authentication**: Custom headers provided via `Helpers::wpApiCallHeaders()`.
- **Caching Strategy**: API responses are cached using `Cache::remember` with a MD5-hashed key derived from the request payload (Form IDs, Date Ranges). This significantly reduces latency and minimizes load on the WordPress database.
- **Payload Handling**: Supports filtering by `start_date`, `end_date`, and specific `form_id`s.

### B. Data Refinement Layer (Factory Pattern)
Because the system handles multiple form types (Individual, Corporate, Trustee, Youth, etc.), a **FormRefinerFactory** is used to normalize data.
- **Factory**: `App\Services\FormRefiners\FormRefinerFactory`
- **Logic**: Each form type has a corresponding "Refiner" class that maps unstructured FluentForm field keys to normalized system keys (e.g., mapping `organisation_email` vs `email` into a standard `refined['applicant']['email']`).

---

## 3. The Conversion Engine (Atomic Transactions)

Converting a lead to a membership is a high-stakes operation involving multiple database tables. To ensure data integrity, the system uses **Laravel Database Transactions**.

### The Conversion Workflow:
1.  **Duplicate Check**: Verifies `entry_id` hasn't been processed before.
2.  **Identity Verification**: Checks if the target email is already registered to a `User`.
3.  **User Provisioning**: Creates a `User` account with a randomized, secure password and assigned the `MEMBER` role.
4.  **PII Encryption**: Sensitive fields (Passport IDs, Phone numbers, etc.) are encrypted at rest using `Helpers::encryptValue()` before being stored in `user_meta`.
5.  **Sub-Entity Handling**: 
    *   **Spouse Provisioning**: If a spouse email is provided, the system automatically checks for an existing user or creates a secondary "Spouse Member" record.
    *   **Nominee Linkage**: Automatically creates and links a Nominee user record for succession tracking.
6.  **EAV Storage**: All non-core fields are stored in an **Entity-Attribute-Value (EAV)** format in `user_meta` and `membership_meta`, allowing for infinite field flexibility without schema migrations.
7.  **Financial Initialization**: Calculates initial fees based on the `MembershipType` and applies dynamic logic (e.g., 50% discount for female applicants).

---

## 4. Multi-Layered Access Control (RBAC)

The module implements strict isolation to ensure that regional coordinators only see data relevant to them.

- **Chapter Scoping**: Chapter Coordinators are restricted to leads belonging to their assigned `Chapter`.
- **Membership Type Scoping**: Coordinators are restricted to specific `MembershipTypes` assigned to their role.
- **Gender-Based Access Control**: A unique governance feature where specific administrators are only permitted to view and process leads of a specific gender (e.g., "Male-only" or "Female-only" access).
- **Audit Logging**: Every initiation, validation failure, and successful conversion is logged using **Spatie Activity Log**, including details on the admin who performed the action and the entry ID involved.

---

## 5. Edge Cases & Resilience

| Edge Case | Solution |
| :--- | :--- |
| **Duplicate API Submission** | The `entry_id` is indexed and checked pre-transaction to prevent double-conversion. |
| **Email Collision** | A dedicated pre-flight check catches existing emails and reverts with a user-friendly "Email already registered" error instead of a database crash. |
| **API Timeout** | The HTTP client is configured with a 60-second timeout and fails gracefully, logging the error and notifying the admin. |
| **Nested Data Mapping** | A recursive `flattenArray` method handles complex multi-dimensional data from the WordPress API, ensuring no data is lost during the EAV conversion. |
| **Selective Encryption** | The system maintains a "Non-encrypted Key" list (e.g., City, Country) to allow for reporting and filtering, while keeping PII (Passport, Address) secure. |
