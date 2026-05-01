# ORG Membership Management Portal - User Roles & Permissions

This document outlines the various user roles within the system and their corresponding responsibilities and access levels. This guide is intended for the QA team to facilitate testing of authorization and system workflows.

---

## 1. Role Overview Matrix

| Feature / Action | Super Admin | General Manager | Chapter Coordinator | Office Bearer |
| :--- | :---: | :---: | :---: | :---: |
| **Data Scope** | Global | Global | Chapter Assigned | Chapter Assigned |
| **Manage Users** | ✅ | ❌ | ❌ | ❌ |
| **Manage Chapters/Types** | ✅ | ❌ | ❌ | ❌ |
| **Convert Leads** | ✅ | ✅ | ✅ | ❌ |
| **Add/Edit Payments** | ✅ | ✅ | ✅ | ❌ |
| **View Reports** | ✅ | ✅ | ❌ | ✅ |
| **Change Requests (Update)** | ✅ | ✅ | ❌ | ❌ |
| **Data Migration** | ✅ | ✅ | ✅ | ❌ |
| **Activity Logs** | ✅ | ❌ | ❌ | ❌ |

---

## 2. Detailed Role Definitions

### 🔑 Super Admin (`super_admin`)
The **Super Admin** is the highest-level user with unrestricted access to the entire system.
*   **User Management**: Create, edit, activate/deactivate, and delete users.
*   **System Configuration**: Manage Chapters (Regions), Membership Types, and Bank details.
*   **Audit Control**: Access to system-wide Activity Logs to monitor all changes.
*   **Global Visibility**: Can see leads, memberships, and payments across all chapters.
*   **Reporting**: Access to all high-level reports.

### 🏢 General Manager (`general_manager`)
The **General Manager** oversees global operations but without administrative control over user accounts or system settings.
*   **Global Visibility**: Like the Super Admin, they can see data from all chapters.
*   **Lead & Membership Management**: Full capability to convert leads, transfer memberships, and manage member profiles.
*   **Change Requests**: Authorized to review and update the status of membership change requests.
*   **Reporting**: Full access to operational reports (Membership, Payments, Aging AP, etc.).
*   **Financials**: Can record and edit payments and line items.

### 📍 Chapter Coordinator (`chapter_coordinator`)
The **Chapter Coordinator** is a regional role restricted to the chapters and membership types specifically assigned to them.
*   **Assigned Scope**: Can only view and manage data (Leads, Memberships, Payments) for their assigned Chapters.
*   **Operational Control**: Can convert leads and manage memberships within their scope.
*   **Payments**: Authorized to collect and record payments for members in their regional scope.
*   **Restricted Access**: Cannot access Global Reports or Change Request management.

### 👔 Office Bearer (`office_bearer`)
The **Office Bearer** is a regional stakeholder role with **Read-Only** access for operational oversight.
*   **View-Only Access**: Can view leads, memberships, and payments within their assigned chapters.
*   **Reporting**: Can view and export regional reports for transparency.
*   **Major Restrictions**: System-enforced restriction from performing any "Write" actions (e.g., cannot convert leads, cannot record payments, cannot delete records).
*   **No Status Management**: Can view change requests but cannot update or approve them.

---

## 3. Data Sensitivity & Access Control

### Chapter & Type Restrictions
For **Chapter Coordinators** and **Office Bearers**, access is filtered at the database level:
- **Chapter Access**: Users are linked to specific chapters in a many-to-many relationship.
- **Membership Type Access**: Users are linked to specific forms (Individual vs Corporate types).
- **Combined Logic**: A coordinator can only see a member if they have access to *both* the member's Chapter AND the member's Membership Type.

### Write Protection (`role_except:office_bearer`)
The system uses a specific middleware to block **Office Bearers** from sensitive actions. If an Office Bearer attempts to access a protected route (like `POST` or `PUT` actions on memberships/payments), the system will return a `403 Unauthorized` error.

---

## 4. Other Roles
*   **Member**: Intended for end-users. Currently has minimal interaction with the management portal.
*   **Viewer**: A generic role intended for read-only access (currently used less frequently than Office Bearer).

---

## 5. QA Testing Checklist
When testing these roles, ensure:
1.  [ ] **Super Admin** can see "Users" and "Settings" in the sidebar.
2.  [ ] **General Manager** can see all change requests and reports, but NOT "Users" or "Settings".
3.  [ ] **Chapter Coordinator** can ONLY see members belonging to their assigned chapters.
4.  [ ] **Office Bearer** can see the "Add Payment" button/link but receives a `403` error or has the button hidden if they try to use it.
5.  [ ] **Data Isolation**: A Coordinator for 'Chapter A' should never see leads or memberships from 'Chapter B'.
