# Module Documentation: Global Payment Management

## 1. Overview
The Global Payment Management module is a multi-jurisdictional financial engine designed to handle international membership dues. It supports both manual (Bank Transfer/Cash) and automated (Stripe) payments, manages annual installment plans, and enforces complex regional financial rules.

## 2. Hierarchical Financial Architecture

The system uses a two-tier model to handle partial payments and overpayments:

### A. The Payment Record (The Obligation)
Represents a specific financial milestone (e.g., "Year 1 Installment" or "Full Membership Fee").
- **Types**: `one-time` (Full) or `five-years` (Installments).
- **Control Logic**: Installments are sequential. The system enforces a **Sequential Integrity Rule** where Payment #N cannot be verified unless Payment #N-1 is `verified` or `waived`.

### B. The Payment Line Item (The Transaction)
Represents the actual flow of funds.
- **Multiplicity**: A single `Payment` obligation can be satisfied by multiple `LineItems` (e.g., paying $5,000 for a $10,000 obligation in two $2,500 transactions).
- **Overpayment Handling**: If a Line Item amount exceeds the current `Payment` amount, the system can credit the surplus to the next chronological installment.

---

## 3. Dynamic Stripe Integration

The system implements a sophisticated regional routing logic for online payments via Stripe.

### Regional Connected Accounts:
To comply with regional financial regulations, payments are routed to different Stripe Connected Accounts based on the member's Chapter:
- **Region A (America & Canada)**: Routed via `on_behalf_of` to the jurisdictional Stripe Account.
- **Region B (Global/UK)**: Routed via `transfer_data` to the international treasury account.
- **Normalization**: Regardless of the region, all transactions are processed and stored in **USD**.

### Online Payment Portal:
- **Token-Based Access**: Members access their payment portal via a unique `payment_token`, requiring no login for immediate dues settlement.
- **Payable Constraint**: The portal dynamically calculates the `nextPayable` installment. Members cannot "skip ahead" to pay future dues if current ones are outstanding.

---

## 4. Regional Financial Rules & Discounts

The engine automatically calculates dues based on several organizational governance rules:

| Rule | Implementation |
| :--- | :--- |
| **Gender-Based Discount** | 50% waiver is automatically applied to the `fees` base for female applicants. |
| **Spouse Discount** | A $1,000 credit is applied to members linked as a spouse of an "Individual Life" member. |
| **Complementary Status** | Specific spouse membership types (Form ID 9) are marked as free, disabling the payment UI entirely. |

---

## 5. Verification & Audit Workflow

### The Verification Loop:
1.  **Submission**: Admin uploads a transaction slip (stored on a `private` disk).
2.  **Review**: Status moves to `in-review`.
3.  **Finalization**: Upon setting status to `verified`, the record is **Locked** (immutable).
4.  **Invoicing**: The system generates a formatted invoice number (`INV-XXXXXX`).
5.  **Notifications**: An automated `PaymentConfirmationMail` is dispatched to the **Member** and the **Global CFO** simultaneously.

### Resilience Features:
- **Private Storage**: Transaction slips are stored outside the public directory, served via a controlled controller method (`viewSlip`) to ensure only authorized admins can view financial proof.
- **Idempotency**: Callback handlers check if a payment is already `verified` before applying Stripe webhooks to prevent double-crediting.
- **Automated Dunning**: A scheduled job (`SendPaymentReminderJob`) monitors `billing_date` and sends reminders 30 days before, 14 days before, and 30 days after a due date.
