# Module Documentation: Executive Dashboard & Analytics

## 1. Overview
The Executive Dashboard serves as the command center for both global administrators and regional coordinators. It provides a real-time, high-level visualization of the organization's financial health, membership growth, and regional performance.

## 2. Intelligence Engine: Dynamic Data Scoping
The dashboard is powered by a **Role-Aware Aggregation Engine**. It ensures that the "Reality" presented to a user is strictly bound by their jurisdictional permissions.

- **Global View (Admin/GM)**: Aggregates data across all chapters and membership types.
- **Regional View (Chapter Coordinator)**: Scopes every KPI—revenue, member counts, and growth charts—to only include the chapters and membership types assigned to the coordinator.
- **Implementation**: Uses Laravel's `whereIn` clauses on `chapter_id` and `form_id` across all Eloquent and Raw SQL queries.

---

## 3. Key Performance Indicators (KPIs)

### A. Monthly Revenue Heatmap
- **Logic**: Aggregates `SUM(amount)` from the `payments` table grouped by month for the current year.
- **Integrity**: Explicitly excludes `failed`, `refunded`, and `waived` payments to ensure financial accuracy.
- **Currency**: Standardized to **USD** globally.

### B. Membership Composition (The Pie Chart)
- **Logic**: Groups total active members by their `MembershipType` (Form ID).
- **Formatting**: Uses `Helpers::membership_type_name_formatted` to ensure user-friendly labels (e.g., "Life Member" instead of "individual_life_form").

### C. Regional Density (The Chapter Bar)
- **Logic**: Groups membership counts by `chapter_id`.
- **Insight**: Identifies the most active regional hubs and jurisdictions with potential for growth.

---

## 4. Multi-Dimensional Growth Analytics

The dashboard includes a sophisticated **3-Year Growth Matrix** that compares membership adoption across two dimensions: **Time** and **Geography**.

- **Dimension 1 (Time)**: Compares the current year against the previous two fiscal years.
- **Dimension 2 (Geography)**: Breaks down counts for each `Chapter`.
- **Dimension 3 (Type)**: Further segments the data by `MembershipType`.
- **Performance**: Pre-aggregates the matrix in the controller to minimize the number of frontend DOM operations.

---

## 5. Financial Health Monitoring
A dedicated "Payment Status" visualization tracks the lifecycle of organizational dues:
- **Verified**: Confirmed revenue in the treasury.
- **In-Review**: Bank transfers awaiting admin verification.
- **Pending**: Outstanding dues (Aging).
- **Waived/Refunded**: Adjustments made for administrative or humanitarian reasons.

---

## 6. Technical Implementation Details

| Feature | Technology |
| :--- | :--- |
| **Data Fillers** | PHP-based array padding to ensure all 12 months are represented even with zero data. |
| **Chart Strategy** | Controller-level flattening of multidimensional arrays into simple lists for easy injection into Chart.js/ApexCharts. |
| **Optimization** | Leverages `pluck()` and `DB::raw()` for high-speed aggregation, bypassing heavy Eloquent model hydration for bulk statistics. |
| **Permissions** | Shared authorization logic with core modules via `isCoordinatorGroup()` and `hasAccessToChapter()`. |
