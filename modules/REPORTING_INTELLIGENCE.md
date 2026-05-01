# Module Documentation: Reporting & Business Intelligence

## 1. Overview
The Reporting module provides the organizational leadership with the data required for financial auditing, regional growth analysis, and governance compliance. It transforms unstructured membership and payment data into actionable business intelligence through a multi-faceted filtering and aggregation engine.

## 2. Advanced Reporting Engines

### A. Aging Accounts Payable (AP) Report
The system implements a sophisticated aging analysis to track "Dues at Risk."
- **Aging Intervals**: Dues are automatically bucketed into: `Current`, `1-30`, `31-90`, `91-180`, `181-365`, and `365+` days.
- **Logic**: For every membership, the system iterates through `pending` payments, calculates the `diffInDays` between the `billing_date` and the target `to_date`, and aggregates the totals into the corresponding aging bucket.
- **Institutional Utility**: This report is used by regional treasurers to manage collection priorities and identify long-term arrears.

### B. Voter Eligibility List (Governance Report)
A critical governance tool used for regional and global elections.
- **Criteria**: 
    1.  Member must belong to a "Paid" membership type (fee > 0).
    2.  Member must have at least one payment record.
    3.  Member must have **zero** unpaid payments with a `billing_date` on or before the current date.
- **Integrity**: Ensures only "Members in Good Standing" are granted voting rights in the system.

### C. Membership Balance Statements
A personalized financial statement for individual members.
- **Data Points**: Displays a full ledger of transactions, including transaction references, bank details, and payment channels.
- **Calculation**: Dynamically calculates `Total Paid` vs. `Total Pending` against the `targetMembershipFee` (after gender/spouse discounts).

---

## 3. Data Integrity & Access Control
The Reporting module adheres to the same strict **Data Isolation** rules as the operational modules:

- **Regional Privacy**: A Coordinator can only generate reports for their assigned Chapters.
- **Gender Isolation**: If an admin's access is restricted to "Male" or "Female," all reports (including financial totals) will only reflect data for that specific gender.
- **EAV Mapping**: Reports dynamically map metadata keys (Profession, Phone, DOB) from the `user_meta` table into a flattened tabular format for export.

---

## 4. Export & Distribution Strategy

The system is integrated with the **Maatwebsite/Excel** library to provide high-fidelity data exports.

- **Standardization**: Exports are standardized to `.xlsx` format with pre-formatted headers and calculated totals.
- **Memory Management**: For large datasets (e.g., Global Member Master List), the system uses eager loading and specific `whereIn` scoping to minimize the SQL footprint.
- **Security**: Activity logs track whenever an administrator generates or exports a sensitive report, ensuring an audit trail for data exfiltration protection.

---

## 5. Report Types Summary

| Report Name | Frequency | Target Audience | Key Metric |
| :--- | :--- | :--- | :--- |
| **Membership Master** | On-Demand | Operations | Growth by Chapter/Date |
| **Payments Summary** | Daily/Weekly | Treasury | Actual Cash Flow (USD) |
| **Aging AP** | Monthly | Treasurers | Arrears Bucketing |
| **Activity Log** | Audits | IT/Internal Audit | System Integrity/Changes |
| **Voter List** | Election Cycles | Election Commission | Compliance/Standing |
