# Abstract System Architecture: Membership Management Portal

These diagrams visualize the core logic and engineering patterns of the Membership Management Portal. They are designed for technical portfolios to demonstrate system design and workflow management.

## 1. Membership Lifecycle & State Machine
This diagram shows the complex transition from an external lead to a verified member, including the multi-level approval and nomination succession logic.

```mermaid
stateDiagram-v2
    [*] --> Lead_Captured: External Integration (Fluent Forms)
    Lead_Captured --> Review_Pending: System Validation
    
    state Review_Pending {
        [*] --> Data_Verification
        Data_Verification --> Compliance_Check
    }
    
    Review_Pending --> Member_Active: Admin Approval
    Review_Pending --> Rejected: Documentation Gap
    
    state Member_Active {
        [*] --> Active_Status
        Active_Status --> Change_Request_Pending: Update Initiated
        Change_Request_Pending --> Active_Status: Approval
        Active_Status --> Nomination_Flow: Succession Event
        Nomination_Flow --> Active_Status: Nominee Conversion
    }
    
    Member_Active --> Archive: Termination
```

## 2. Event-Driven Payment Architecture (Stripe Integration)
This diagram illustrates the robust, idempotent handling of online and offline payments, ensuring financial integrity.

```mermaid
sequenceDiagram
    participant User
    participant Laravel_Portal
    participant Stripe_Gateway
    participant Bank_Staging
    participant Event_Dispatcher

    User->>Laravel_Portal: Initiates Payment
    alt Online (Stripe)
        Laravel_Portal->>Stripe_Gateway: Generate Checkout Session
        Stripe_Gateway-->>User: Secure Payment Form
        User->>Stripe_Gateway: Submit Payment
        Stripe_Gateway->>Laravel_Portal: Webhook (payment_intent.succeeded)
        Laravel_Portal->>Event_Dispatcher: Trigger "PaymentReceived" Event
    else Offline (Bank)
        User->>Laravel_Portal: Upload Deposit Slip
        Laravel_Portal->>Bank_Staging: Log Pending Transaction
        Admin->>Laravel_Portal: Verify & Approve Slip
        Laravel_Portal->>Event_Dispatcher: Trigger "PaymentReceived" Event
    end

    Event_Dispatcher->>Financial_Ledger: Update Member Balance
    Event_Dispatcher->>Activity_Log: Record Transaction Audit
    Event_Dispatcher->>Mailer: Send Receipt
```

## 3. High-Level System Decoupling (Service Layer)
A visualization of the Clean Architecture used to separate concerns, making the system scalable and testable.

```mermaid
graph TD
    UI[Frontend: Custom CSS / Blade] --> Controllers[Controllers: Thin Layer]
    
    subgraph "Application Core"
        Controllers --> Services[Business Logic Services]
        Services --> Repositories[Data Access Layer]
    end
    
    subgraph "External Integrations"
        Services --> Stripe[Stripe API]
        Services --> WordPress[WordPress API Integration]
        Services --> Migration[Legacy Data Staging]
    end
    
    Repositories --> DB[(MySQL Normalized Schema)]
```

## 4. Global Multi-Regional Architecture
Visualizes how the system handles international scale by partitioning data and administration across global chapters and mapped countries.

```mermaid
graph TD
    Central[Central Administrative Portal] --> ChapterA["Chapter: Region A"]
    Central --> ChapterB["Chapter: Region B"]
    Central --> ChapterC["Chapter: Region C"]
    
    subgraph "Regional Partitioning"
        ChapterA --> CountriesA["Mapped Jurisdictions"]
        ChapterA --> RegionalBanksA["Regional Accounts: USD"]
        ChapterA --> CoordinatorA["Chapter Coordinators"]
        
        ChapterB --> CountriesB["Mapped Jurisdictions"]
        ChapterB --> RegionalBanksB["Regional Accounts: USD"]
        ChapterB --> CoordinatorB["Chapter Coordinators"]
    end
    
    CoordinatorA --> RegionalDataA["Regional Memberships & Financials"]
    CoordinatorB --> RegionalDataB["Regional Memberships & Financials"]
```
