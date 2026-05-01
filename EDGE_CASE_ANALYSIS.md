# Edge Case Analysis: Engineering for Reliability

This document outlines the specific "edge cases" and technical hurdles addressed in the Membership Management Portal, specifically within the Change Request and Payment modules.

## 1. Membership Change Request Module

### A. Identity Collision Prevention
**Edge Case**: A member attempts to update their ID/Passport number to one that is already registered to another member.
**Solution**: During the approval process, the system performs a "Decryption-Compare" check. Since PII is encrypted at rest, the system retrieves potential duplicates, decrypts them in memory, and validates against the proposed change before committing.

### B. Atomic Profile Synchronization
**Edge Case**: A member's name and email are updated, but the database connection drops before their metadata (EAV) is saved.
**Solution**: All profile updates are wrapped in a `DB::transaction`. If any part of the user account, membership record, or metadata fails validation (e.g., duplicate email), the entire request is rolled back, and the previous state is preserved.

### C. Dynamic Fee Recalculation
**Edge Case**: A member changes their membership type (e.g., Annual to Life) mid-cycle.
**Solution**: The approval logic automatically triggers a fee recalculation based on the new `form_id`. It specifically accounts for the **50% gender-based discount** for female members, ensuring the ledger is always accurate regardless of the previous fee structure.

---

## 2. Global Payment & Financial Module (USD Only)

### A. Installment Integrity (The "Skipped Payment" Problem)
**Edge Case**: A member tries to pay for Installment #3 while Installment #2 is still pending.
**Solution**: The payment initialization logic identifies the `first unpaid payment` in the sequence. It locks all future installments, ensuring that the financial ledger follows a strict chronological order.

### B. Regional Stripe Routing
**Edge Case**: Funds from specific geographic chapters must go to a different bank account than other global regions.
**Solution**: The Stripe Service uses a dynamic configuration switch. Based on the Chapter assignment, it attaches an `on_behalf_of` or `transfer_data` destination to the Stripe Checkout session, routing USD funds to the correct regional legal entity automatically.

### C. Idempotent Webhook/Callback Handling
**Edge Case**: A user refreshes their browser after a successful Stripe payment, or Stripe sends multiple webhook notifications for the same transaction.
**Solution**: The `handleStripeCallback` and Webhook handlers perform a "State Check" first. If the payment is already marked as `verified`, subsequent calls are ignored, preventing duplicate transaction logs or accidental double-crediting.

### D. Multi-Interval Automated Reminders
**Edge Case**: Hundreds of payments become due on the same day, potentially timing out the email server.
**Solution**: Reminders are processed via a **Queued Job Architecture**. The system identifies payments due at specific intervals (1 month before, 14 days before, 1 month after) and dispatches them to a background queue, ensuring system stability during peak billing cycles.
