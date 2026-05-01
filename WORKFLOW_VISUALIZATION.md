# Workflow Visualization: The Membership Journey

This flowchart illustrates the end-to-end lifecycle of a member, from the initial external application (Lead) to the final financial verification (Payment).

```mermaid
graph TD
    %% Lead Phase
    subgraph "Phase 1: Lead Acquisition"
        A[External FluentForm Submission] -->|WP API Bridge| B(Data Fetching & Caching)
        B --> C{FormRefinerFactory}
        C -->|Normalize| D[Staging Lead Area]
    end

    %% Conversion Phase
    subgraph "Phase 2: Conversion Engine"
        D --> E{Admin Review & Convert}
        E -->|Atomic Transaction| F[User Provisioning]
        F --> G[PII Encryption & Meta Storage]
        G --> H[Financial Initialization]
        H -->|Apply Discounts| I[Membership Created]
    end

    %% Payment Phase
    subgraph "Phase 3: Financial Settlement"
        I --> J{Payment Method}
        
        %% Online Path
        J -->|Online / Stripe| K[Payment Portal Access via Token]
        K --> L[Stripe Checkout]
        L -->|Webhook| M{Verified Status}
        
        %% Manual Path
        J -->|Manual / Bank Transfer| N[Bank Details Exposure]
        N --> O[Admin Uploads Transaction Slip]
        O --> P[Status: In-Review]
        P -->|Verification| M
    end

    %% Finalization Phase
    subgraph "Phase 4: Activation & Audit"
        M --> Q[Invoice Generation]
        Q --> R[Confirmation Emails Sent]
        R --> S[Membership Activated]
        S --> T[Activity Audit Log Updated]
    end

    %% Styling
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style I fill:#bbf,stroke:#333,stroke-width:2px
    style S fill:#bfb,stroke:#333,stroke-width:2px
    style M fill:#ff9,stroke:#333,stroke-width:2px
```

## Journey Breakdown

### 1. Lead Acquisition
The journey begins outside the portal on a WordPress site. The **API Bridge** handles the secure transport of data, while the **Refiner Factory** ensures that diverse form structures are flattened into a consistent schema for the portal.

### 2. Conversion Engine
This is the most critical technical step. The system atomically creates the User identity, encrypts sensitive data (PII), and initializes the financial obligations. If a spouse or nominee is included, they are provisioned simultaneously to maintain relational integrity.

### 3. Financial Settlement
The system supports a dual-path payment architecture:
- **Automated**: Leverages Stripe with regional routing and webhook callbacks for instant verification.
- **Manual**: Provides jurisdictional bank details and a secure "In-Review" workflow for admin-led verification of bank transfers.

### 4. Activation & Audit
Activation is the final state change. Once verified, the system generates a unique invoice number, notifies all stakeholders (Member and Global CFO), and locks the financial record to prevent future tampering.
