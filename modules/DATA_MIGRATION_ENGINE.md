# Module Documentation: Data Migration Engine

## 1. Overview
The Data Migration Engine is a high-integrity pipeline designed to transition legacy membership and financial data into the portal's modern schema. To prevent data corruption, the engine implements a **Two-Stage Staging Architecture**, where data is first ingested into a sandbox environment for review before being atomically committed to production.

## 2. Phase 1: Excel Ingestion & Staging
The system utilizes `Maatwebsite/Excel` to process bulk data imports for three primary entities:
- **Individual Memberships**: Personal details, contact information, and professional data.
- **Corporate Memberships**: Organization details and primary contact mapping.
- **Payment History**: Historical transaction records and installment logs.

### Staging Tables:
Data is stored in `individual_membership_staging`, `corporate_membership_staging`, and `payment_staging`. This allows the system to:
- Perform preliminary validation without affecting production records.
- Handle multi-file imports where payments may be imported separately from membership records.
- Track processing status via the `is_processed` flag.

---

## 3. Phase 2: Review & Jurisdictional Filtering
Administrators interact with the staging data through a specialized review interface. This phase is governed by the portal's **Jurisdictional Mapping** rules:
- **Scoped Visibility**: Coordinators only see staging records belonging to their assigned Chapters and Membership Types.
- **Gender Restrictions**: Demographic scoping is enforced even at the staging level, ensuring female coordinators restricted to female records cannot see male staging data.
- **Audit Discovery**: High-fidelity logs track every view and discard action performed on staging records.

---

## 4. Phase 3: Atomic Finalization (Services)
The final transition from staging to production is handled by dedicated service classes: `IndividualMigration` and `CorporateMigration`.

### The Atomic Transaction:
The migration of a single record executes as a single database transaction encompassing:
1.  **Identity Creation**: Generates a `User` account with a randomized, secure password.
2.  **Metadata Encryption**: Sensitive personal data (IDs, Phone Numbers, Addresses) is encrypted using the portal's `encryptValue` helper before storage in `user_meta`.
3.  **Membership Linkage**: Associates the user with a `Membership` record, generating a unique `payment_token` for future dues.
4.  **Meta Serialization**: Saves complex attributes (consents, verified status) into `membership_meta`.
5.  **Payment Reconciliation**: Reconstructs the financial history from staging payments.

---

## 5. Financial Integrity & Installment Logic
A critical component of the engine is its ability to handle complex payment structures during migration.

### A. Automatic Fee Calculation
The engine automatically calculates the expected `fees` for the new membership record, applying **Gender-Based Discounts** (50% waiver for females) dynamically during the migration process.

### B. Installment Reconstruction
For "Five-Year" installment plans, the engine:
- Matches staging payments to the correct installment number.
- **Gap Filling**: If a member has paid for 2 years, the engine automatically generates "Pending" payment records for the remaining 3 years.
- **Balance Logic**: It calculates the `remainingFeeAmount` by subtracting the sum of migrated payments from the total membership fee, ensuring the financial balance is accurate in the new system.

---

## 6. Security & Auditability
- **Bulk Constraints**: To ensure system stability, bulk migration is limited to 50 records per request.
- **Idempotency**: The `is_processed` flag prevents duplicate migrations of the same staging record.
- **Activity Logging**: 
    - `data_migration_import`: Logs the start and result of Excel imports.
    - `bulk-migrate-individual-membership`: Logs every successful or failed record migration.
    - `data_of_migration_discarded`: Logs the intentional removal of staging data.

---

## 7. Migration Pipeline Summary

| Step | Component | Primary Responsibility |
| :--- | :--- | :--- |
| **Ingestion** | `Excel::import` | Bulk data loading from XLSX/CSV. |
| **Verification** | Staging UI | Human-in-the-loop review with scoped access. |
| **Transformation** | `IndividualMigration` | Data cleaning, encryption, and relational mapping. |
| **Financials** | `processPayments` | Installment gap filling and fee reconciliation. |
| **Commit** | DB Transaction | Atomic finalization and staging record marking. |
