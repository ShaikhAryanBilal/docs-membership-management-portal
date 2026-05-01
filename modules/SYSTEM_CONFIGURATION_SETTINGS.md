# Module Documentation: System Configuration & Global Settings

## 1. Overview
The System Configuration module serves as the central nervous system of the portal. It allows Super Administrators to define the structural, geographical, and financial parameters that govern how the entire platform operates. This module ensures that regional hubs (Chapters) are correctly mapped, financial channels (Banks) are operational, and global notification routing is optimized.

## 2. Global System Options
The platform leverages a dynamic key-value storage system (`portal_options`) to manage global configurations without requiring code deployments.

### A. Weekly Report Recipients
- **Purpose**: Automates the distribution of the "Weekly Membership & Financial Pulse" report.
- **Implementation**: Administrators can assign specific email addresses to receive automated PDF summaries of global KPIs.
- **Routing**: The system supports multiple recipients per region, ensuring that global leadership and regional GMs are synchronized on a weekly cadence.

---

## 3. Jurisdictional Mapping (Chapters & Countries)
The portal implements a strict **Geographical Isolation** model. The Chapter configuration defines the boundaries of administrative authority.

### A. Chapter Management
- **Hierarchy**: Chapters represent the primary organizational units (e.g., Africa, Middle East, Europe).
- **Membership Association**: Every member is linked to a Chapter, which dictates their currency, banking options, and assigned coordinator.

### B. Country Mapping
- **Logic**: Chapters are mapped to specific ISO countries via a JSON-encoded relationship.
- **Dynamic Filtering**: When an applicant selects their country of residence, the system automatically routes the application to the corresponding Chapter's workflow.

---

## 4. Regional Financial Infrastructure (Banks)
To support international manual payments (Bank Transfers), the system maintains a robust banking directory.

### A. Multi-Chapter Banking
- **Assignment**: Each Chapter can manage multiple bank accounts.
- **Metadata**: Stores comprehensive wire transfer details including IBAN, SWIFT codes, Branch Codes, and Account Titles.
- **Redundancy Rule**: The system enforces a **Safety Constraint**—at least one active bank account must exist for a chapter. This prevents "Payment Deadlocks" where a member is unable to find a valid payment destination.

### B. Transactional Visibility
- **Security**: Bank details are only exposed to members during the "Submit Payment" phase of their specific jurisdictional workflow.

---

## 5. Membership Taxonomy (Types & Fees)
The platform manages a dynamic catalog of membership tiers, each with distinct financial obligations.

### A. Fee Governance
- **Tiers**: Manages diverse types such as "Patron," "Individual Life," "Youth," and "Corporate."
- **Financial Mapping**: Each type is assigned a base `fee` (USD), which serves as the anchor for all discount calculations (e.g., Gender or Spouse discounts).

### B. Form ID Integration
- **Structural Link**: Each membership type is mapped to a `form_id`, ensuring that data collected via the Fluent Forms API is correctly categorized into the portal's internal schema.

---

## 6. Security & Governance
Given the sensitive nature of system configurations, the module implements high-level security protocols:

### A. Super Admin Exclusivity
- **Access Control**: Only the `super_admin` role is granted access to the `/settings` sub-menu. All other roles, including General Managers, are restricted from modifying these parameters.

### B. Activity Auditing
- **Immutable Logs**: Every change to a Chapter name, Bank status, or Membership Fee is captured by the `Spatie ActivityLog` engine.
- **Traceability**: Logs record the "Before" and "After" state of the configuration, the performing administrator, and the timestamp of the event.

---

## 7. Configuration Matrix

| Component | Entity | Key Governance Rule |
| :--- | :--- | :--- |
| **Global Settings** | `portal_options` | JSON-encoded routing for multi-recipient notifications. |
| **Chapters** | `chapters` | Strictly maps ISO countries to administrative hubs. |
| **Banks** | `banks` | Requires 1+ active record per chapter to enable manual payments. |
| **Membership Types**| `membership_types` | Anchor for fee calculations and Fluent Forms integration. |
