# Trust Wallet ✌️ --- AI-Powered Banking Fraud Intelligence & Protection System

> **Project master brief for GitHub**
>
> This document is the source of truth for the project architecture,
> product requirements, integration rules, security boundaries, and
> implementation standards. Read it before changing or generating any
> part of the system.

## 1. Project mission

Build an AI-powered banking fraud intelligence and protection platform
that monitors transaction activity, detects suspicious behaviour,
explains risk signals, supports human investigations, and gives
customers clear security assistance.

The platform must be built as **three separate web applications**
sharing secure backend services:

1.  **Customer Website** --- customer banking experience, transaction
    services, account security, alerts, and AI Banking Assistant.
2.  **Bank Intelligence Dashboard** --- authorised bank investigators
    monitor risk, investigate suspicious transactions/accounts, review
    evidence, and manage cases.
3.  **Platform Administrator Dashboard** --- platform-level
    administration, participating banks, users, roles, configuration,
    and audit oversight.

These are three frontends, not three isolated systems. They must use
shared, properly secured backend services and consistent identifiers.

## 2. Non-negotiable project rules

-   Do not build only a visual mock-up and call it a working banking
    system.
-   Do not claim live bank connectivity, real balances, real
    fraud-database access, or successful payments unless a real
    authorised integration confirms them.
-   Use synthetic/demo customer, account, beneficiary, and transaction
    data until authorised integrations are available.
-   Do not let an AI model calculate the official numerical risk score.
    A deterministic Risk Engine owns numerical scoring.
-   AI may explain signals, summarise evidence, and assist
    investigators. AI must not independently declare a person guilty,
    freeze an account, or execute a transfer.
-   Only an authorised banking/payment integration or authorised human
    workflow may approve, hold, decline, or execute a financial
    transaction.
-   Do not expose webhook URLs, API keys, service credentials,
    authentication tokens, or private secrets in browser code or public
    GitHub files.
-   Every important user-initiated business request and every
    transaction lifecycle event must be routed through the central
    integration gateway described below.
-   Keep the user-provided Trust Wallet logo/brand asset unchanged. Do
    not invent a replacement logo or substitute an unrelated logo.

## 3. Brand and visual direction

**Customer-facing name:** Trust Wallet ✌️

Use an original interface with OPay-like ease of use, but do not copy
OPay's branding, exact layout, icons, or proprietary visual assets.

### Visual system

-   Deep black and dark charcoal surfaces.
-   Muted champagne gold / soft antique gold accents. Avoid bright
    yellow or harsh metallic gold.
-   White and soft-grey typography with clear contrast.
-   Premium, calm, trustworthy banking aesthetic.
-   Rounded cards, restrained shadows, subtle borders, clean spacing,
    and restrained gradients.
-   Mobile-first responsive design, especially for Android-sized
    screens, while supporting tablet and desktop.

### Logo requirements

-   Use the exact approved Trust Wallet logo asset supplied by the
    project owner.
-   Preserve its proportions, colours, symbol, and typography. Do not
    redraw, stylise, crop, recolour, or replace it.
-   Use the logo in the agreed brand positions: app header/navigation
    and authentication screens where appropriate.
-   Store the asset in a central asset location and reference it through
    one central configuration/component. Do not paste different logo
    copies into multiple pages.
-   If the exact approved logo file or URL is not present in the
    repository, leave a clearly identified asset slot and request the
    approved asset. **Do not fabricate a logo URL.**
-   The text wordmark may read `Trust Wallet ✌️` while the approved
    image asset is being added, but this must not be represented as the
    final logo.

Suggested asset convention once the real file is supplied:
`public/assets/trust-wallet-logo` with its actual file extension. Update
all references centrally.

## 4. System architecture

Use a monorepo initially, with clear application and service boundaries.
Adapt the folder names only if the existing repository already has a
consistent structure.

``` text
trust-wallet-fraud-intelligence/
├── apps/
│   ├── customer-web/
│   ├── bank-intelligence-web/
│   └── platform-admin-web/
├── services/
│   ├── backend-api/
│   ├── risk-engine/
│   └── ai-services/
├── integrations/
│   └── n8n/
├── packages/
│   ├── shared-types/
│   ├── shared-ui/
│   └── validation/
├── docs/
│   ├── architecture.md
│   ├── webhook-contract.md
│   ├── risk-scoring.md
│   └── security.md
├── .env.example
├── .gitignore
└── README.md
```

### Core components

-   **Frontend applications:** three independent web applications with
    role-appropriate interfaces.
-   **Authentication:** use the selected frontend authentication
    provider (Lovable-supported authentication where applicable), but
    validate sessions/tokens on the server for every protected
    operation.
-   **Backend API / integration gateway:** trusted server-side layer
    that authenticates requests, derives identity and role from verified
    sessions, validates payloads, enforces customer/bank data
    boundaries, applies rate limits, handles idempotency, and calls n8n.
-   **n8n:** workflow orchestration, event routing, notifications,
    AI-agent coordination, integrations, and audit/event processing.
-   **Python Risk Engine:** deterministic account and transaction risk
    calculations.
-   **AI services:** investigation assistance, customer-facing
    explanations, evidence summaries, and report generation.
-   **Persistent database:** secure storage for users, accounts,
    transactions, risk assessments, cases, reports, audit records, and
    configuration. Supabase has been ruled out for this project; select
    and configure another suitable secure persistence layer before
    relying on persistent production data.
-   **Payment/banking provider:** required for real transfers, debits,
    card activity, deposits, and balance confirmation. n8n webhook
    receipt by itself does not move money.

Do not assume that frontend authentication alone provides a secure
database, API, or role enforcement layer.

## 5. Central n8n webhook --- mandatory integration

### Configured webhook endpoint

The project owner supplied this central n8n webhook:

``` text
https://enemo.app.n8n.cloud/webhook/58c35268-18b6-4b99-9588-90c481b9ff0b
```

This is the configured endpoint for the current prototype. Keep it in
server-side environment/configuration, not in client-side source code.

Suggested server-side variable:

``` env
N8N_CENTRAL_WEBHOOK_URL=https://enemo.app.n8n.cloud/webhook/58c35268-18b6-4b99-9588-90c481b9ff0b
```

Do not commit real secrets or private credentials. If the endpoint must
later be rotated, update the server-side environment configuration only.

### Mandatory routing rule: every request

**Every customer-initiated business request/action must be sent through
the central backend integration gateway to this n8n webhook.** Do not
limit webhook calls to the AI chat.

This includes, at minimum:

-   Registration and account onboarding events.
-   Login, logout, session/security events where appropriate.
-   Profile and account-setting changes.
-   Balance enquiries.
-   Transaction-history requests.
-   Transaction-detail requests.
-   Transfers to another customer or bank.
-   Airtime and data purchases.
-   TV subscriptions and bill payments.
-   Card actions and card transaction events.
-   Deposits, withdrawals, refunds, reversals, and payment requests.
-   Beneficiary creation, update, and deletion.
-   Fraud reports and suspicious-activity reports.
-   Complaints, support requests, and complaint-status requests.
-   AI assistant messages and requests for account/transaction
    explanations.
-   Security-status enquiries, device/security changes, and alert
    acknowledgements.
-   Rewards, invitations, and other customer-facing service actions that
    require backend processing.

The Bank Intelligence and Platform Administrator applications must also
send important business operations, investigation actions, configuration
changes, user/role changes, and other security-relevant actions through
the same central event gateway, with the correct `source_app`, actor
role, and event type.

Purely local presentation changes (for example, opening a local menu,
changing a theme locally, or typing into an unsent form field) do not
need a webhook event. **Every request that reads or changes business
data, initiates a service, triggers an AI operation, or affects a
transaction must be routed and logged.**

### Required request flow

``` text
Customer / Bank Investigator / Platform Administrator
                         |
                         v
                Frontend application
                         |
                         v
             Authenticated Backend API
                         |
            Validate session, role, input,
          permissions, ownership, and request
                         |
                         v
               Central Event Gateway
                         |
                         v
                 n8n Webhook Node
                         |
       +-----------------+------------------+
       |                 |                  |
       v                 v                  v
  Risk Engine        AI Agents        Notifications /
  (deterministic)    (explain/assist)  Other workflows
       |                 |                  |
       +-----------------+------------------+
                         |
                         v
             Structured backend response
                         |
                         v
                 Correct frontend UI
```

Do not call the webhook directly from browser code. The backend gateway
must make the server-to-server request so secrets and customer identity
cannot be forged by editing browser requests.

### Common event envelope

Every event sent to n8n should use a consistent, versioned structure.
Extend it only when necessary; do not send passwords, OTPs, PINs, raw
authentication tokens, API keys, or unnecessary sensitive information.

``` json
{
  "schema_version": "1.0",
  "event_id": "unique-uuid",
  "event_type": "balance_enquiry",
  "occurred_at": "ISO-8601 timestamp",
  "source_app": "customer-web",
  "actor": {
    "user_ref": "server-derived-user-reference",
    "role": "customer"
  },
  "request": {
    "request_id": "unique-request-id",
    "correlation_id": "correlation-id",
    "idempotency_key": "where-applicable"
  },
  "subject": {
    "account_ref": "pseudonymous-account-reference"
  },
  "data": {},
  "context": {
    "device_ref": "non-secret-device-reference",
    "session_ref": "non-secret-session-reference"
  }
}
```

Rules: - Derive `user_ref`, role, and account ownership from the
verified server-side session. Never trust a browser-supplied customer ID
or role. - Use pseudonymous references where possible. Do not send full
account/card numbers unless a specific authorised integration requires
them. - Include a unique `event_id` and `request_id`; use idempotency
keys for operations where duplicate execution would be harmful. -
Include timestamps in ISO-8601 UTC format. - Validate all inputs and
enforce payload size limits. - Record delivery status, correlation IDs,
and safe error details. - Never log passwords, OTPs, transaction PINs,
raw tokens, or secrets.

## 6. Transaction lifecycle and fraud gate

Every transaction must be represented by a unique transaction record and
sent to the central webhook at initiation and at every meaningful status
change.

### Transaction event types

Use event types such as:

-   `transfer_initiated`
-   `internal_transfer_initiated`
-   `airtime_purchase_initiated`
-   `data_purchase_initiated`
-   `tv_subscription_initiated`
-   `bill_payment_initiated`
-   `card_transaction_initiated`
-   `withdrawal_initiated`
-   `deposit_received`
-   `payment_initiated`
-   `refund_initiated`
-   `debit_transaction`
-   `credit_transaction`
-   `transaction_completed`
-   `transaction_failed`
-   `transaction_cancelled`
-   `transaction_reversed`
-   `transaction_expired`
-   `transaction_reviewed`
-   `transaction_declined`

Choose the event type that accurately describes the operation. Do not
label a transaction completed until the authorised payment/banking
provider confirms completion.

### Transaction event data

Where applicable, include:

-   Unique `transaction_id`.
-   Event type and transaction type.
-   Direction (`debit` or `credit`).
-   Amount and currency.
-   Pseudonymous source-account reference.
-   Beneficiary reference and beneficiary bank where relevant.
-   Beneficiary-new/known indicator where available.
-   Initiation timestamp.
-   Device reference and session reference.
-   Channel and safe contextual metadata.
-   Current transaction status.
-   Correlation and idempotency identifiers.

Never include a transaction PIN, OTP, password, raw access token, or API
secret.

### Required decision sequence

For transactions that require fraud assessment:

1.  Create a pending transaction intent with a unique transaction ID.
2.  Send the initiation event through the backend gateway to the central
    n8n webhook.
3.  n8n obtains a deterministic risk assessment from the Risk Engine and
    returns a structured decision.
4.  Apply the decision:
    -   `APPROVE`: continue to the authorised payment provider. This is
        not proof that the payment succeeded.
    -   `REVIEW`: keep the transaction pending and do not execute until
        the required authorised review/verification is completed.
    -   `DECLINE`: reject the transaction and record the reason.
    -   `UNAVAILABLE`: fail safely. Do not bypass fraud checks or
        pretend the transaction is safe.
5.  If approved for processing, submit to the authorised payment
    provider.
6.  Wait for the provider's verified result.
7.  Send the corresponding outcome event to the same central webhook.
8.  Update the transaction record and customer interface from the
    verified result.

### Example risk response contract

The actual n8n workflow and backend must agree on the final contract.
This is an illustrative target, not a claim that the current workflow
already returns it.

``` json
{
  "success": true,
  "request_id": "request-uuid",
  "transaction_id": "transaction-uuid",
  "risk_assessment": {
    "risk_score": 82,
    "risk_level": "critical",
    "decision": "REVIEW",
    "signals": [
      "Unusual transaction amount",
      "New beneficiary",
      "Unrecognised device"
    ]
  },
  "transaction_status": "pending_review",
  "requires_human_review": true
}
```

### Reliability requirements

-   Use idempotency and duplicate-event detection.
-   Use bounded timeouts and safe retries for event delivery.
-   Never blindly retry a financial transaction after a timeout. First
    reconcile the transaction status with the authorised provider.
-   Maintain transaction state transitions and an auditable event
    history.
-   If n8n or the Risk Engine is unavailable for a transaction requiring
    assessment, do not silently bypass the check.
-   Distinguish webhook acceptance, risk approval, provider acceptance,
    and final transaction success. They are different states.

## 7. Risk intelligence and scoring

Keep two separate scores:

1.  **Transaction Risk Score:** risk associated with one transaction.
2.  **Account Risk Score:** broader, continuously updated risk indicator
    for an account.

Potential signals include:

-   Transaction amount and deviation from normal behaviour.
-   Transaction frequency and velocity.
-   Time-of-day and access-pattern changes.
-   New or unusual beneficiaries.
-   Rapid movement of incoming funds.
-   Transfer chains and linked-account patterns.
-   Device changes and unusual access patterns.
-   Verified historical reports and case intelligence.

Illustrative score bands (0--100): - 0--29: Low - 30--59: Medium -
60--79: High - 80--100: Critical

These are prototype bands, not validated production thresholds. Weights
and thresholds must be tested against labelled scenarios and reviewed
before real-world deployment.

The Risk Engine must: - Be deterministic and explainable. - Return
score, level, reason codes/signals, assessment time, and data
sufficiency/confidence indicators. - Handle new accounts with
insufficient history without inventing certainty. - Support signal
ageing/decay, corrections, and disputes. - Keep score history for audit
and trend analysis.

The LLM must not invent risk scores, evidence, account links, or
historical reports.

## 8. Beneficiary intelligence and evidence quality

For the prototype, use synthetic case data and reports entered by
authorised demo administrators.

For future deployment, use only lawful, authorised data-sharing
arrangements and approved integrations.

Track the status of intelligence separately: - `unverified` -
`under_review` - `verified` - `rejected` - `disputed` / correction
requested

A single allegation must never automatically label a beneficiary a
confirmed fraudster. Store source/authority, case reference, timestamp,
evidence summary, review status, and correction/dispute history.
Restrict access by role and maintain audit logs.

## 9. Customer Website requirements

### Authentication and account access

-   Sign-up, sign-in, sign-out, password recovery, and profile
    management.
-   Use the selected authentication provider, with server-side
    session/token validation.
-   Demo OTP/PIN flows must be clearly labelled as simulated. Never
    collect real banking credentials in a prototype.
-   Support secure logout and session expiry.

### Home dashboard

-   Approved logo and Trust Wallet ✌️ wordmark.
-   Greeting, notifications, support, and scan/QR entry point where
    implemented.
-   Balance card with show/hide control, account reference/number
    display only where appropriate, and copy action.
-   Add Money and Transaction History actions.
-   Account security overview with risk level, recent alerts, and
    security details.
-   Quick services: Cards, Airtime, Data, TV, Invitation/Rewards,
    Transfer, To Bank, Security, and See All.
-   Recent transactions with description, debit/credit, amount,
    date/time, reference, and verified status.
-   Bottom navigation: Home, Rewards, Insights, Cards, Me.
-   Floating AI Banking Assistant at the bottom-right, replacing the
    floating security-icon position.

### Services

Build coherent screens and flows for: - Internal transfers and bank
transfers. - Airtime and data purchases. - TV and bill payments. - Card
management (clearly demo-only unless issuer integration exists). -
Deposits and withdrawals (clearly demo-only unless provider integration
exists). - Beneficiaries. - Rewards and invitations. - Security
monitoring and alerts. - Transaction details and receipts.

Every business action must use the central backend gateway and n8n event
flow. Do not show a success state until the backend and relevant
provider confirm it.

### AI Banking Assistant

The assistant is part of the Customer Website, not a fourth website.

It can: - Answer general banking-support questions. - Retrieve the
authenticated customer's permitted balance and transaction
information. - Explain why a transaction or account generated a risk
alert. - Help submit a fraud report or complaint. - Track
complaint/report status. - Give security guidance.

Requirements: - Send every assistant request through the central gateway
to n8n. - Use authenticated customer context and enforce strict customer
data isolation. - Do not expose another customer's data. - Require
additional verification for sensitive actions. - Do not execute
transfers or change security-critical settings based only on
conversational intent. - Clearly distinguish general AI guidance from
verified account data. - If the risk service or backend fails, say that
the information is unavailable; do not invent an answer.

## 10. Bank Intelligence Dashboard requirements

For authorised bank investigators: - Overview of transaction and account
risk. - Search and filter suspicious transactions/accounts. - Separate
transaction and account risk scores. - Explainable risk signals and data
sufficiency. - Connected-account/network investigation view where
supported by evidence. - Case management, evidence timeline,
investigator notes, and status changes. - Review queue for transactions
awaiting human review. - Reports, trends, and export controls subject to
permissions. - Audit history of investigator actions.

All sensitive actions must be permission-checked server-side and sent
through the central event gateway. No investigator should see data
outside their authorised bank/tenant scope.

## 11. Platform Administrator Dashboard requirements

For authorised platform administrators: - Manage participating banks and
organisations. - Manage users, invitations, roles, and permissions. -
Configure platform-level settings and integration status. - View
platform activity and security audit logs. - Manage demo intelligence
records and test scenarios. - Monitor webhook delivery, workflow health,
and service errors without exposing secrets. - Support access revocation
and account deactivation.

Platform administrators must not automatically gain unrestricted access
to all customer banking data. Enforce least privilege and explicit
permissions.

## 12. Security, privacy, and compliance

-   HTTPS/TLS for all network traffic.
-   Encrypt sensitive data at rest.
-   Validate every protected request server-side.
-   Enforce role-based access control and customer/bank/tenant
    isolation.
-   Keep secrets in server-side environment variables or a secrets
    manager.
-   Apply rate limits, input validation, request-size limits, and abuse
    protection.
-   Use audit logs for sensitive reads and all important writes/actions.
-   Do not log passwords, OTPs, PINs, raw tokens, or API keys.
-   Do not store raw biometric data in the prototype.
-   Use synthetic data for demos and competition testing.
-   Before real Nigerian deployment, obtain legal/security review for
    the Nigeria Data Protection Act and applicable CBN requirements,
    plus any other relevant payment, banking, and data-sharing
    obligations.

This project brief is not legal or regulatory approval to process real
customer banking data.

## 13. GitHub and engineering standards

-   Use GitHub as the source-control home for the project.
-   Keep a clear `main` branch and use feature branches for substantial
    work.
-   Make small, descriptive commits.
-   Review changes before merging.
-   Never commit `.env` files, API keys, webhook secrets, real customer
    data, production exports, or private credentials.
-   Maintain `.env.example` with variable names and safe placeholder
    values only.
-   Keep API contracts and shared types versioned.
-   Add tests for authentication/authorization, transaction state
    transitions, idempotency, webhook failure handling, and risk
    scoring.
-   Document setup, deployment, environment variables, and known
    limitations.
-   Do not claim a feature is complete until it has been tested
    end-to-end.

## 14. Environment configuration

Use server-side environment variables. Example names:

``` env
N8N_CENTRAL_WEBHOOK_URL=
N8N_WEBHOOK_AUTH_SECRET=
DATABASE_URL=
AUTH_PROVIDER_SERVER_SECRET=
OPENAI_API_KEY=
PAYMENT_PROVIDER_SECRET=
APP_ENV=development
```

The `.env.example` file must contain blank or clearly fake placeholders.
Actual values must be configured privately in the deployment
environment. Never place server secrets in `VITE_*`, `NEXT_PUBLIC_*`, or
any other browser-exposed variable.

## 15. Minimum end-to-end acceptance tests

Do not mark the first customer website complete until these checks pass:

1.  Customer can register/sign in using the configured demo
    authentication flow.
2.  Server verifies the session and derives the customer identity.
3.  A balance enquiry produces a correctly typed event at the central
    n8n webhook.
4.  A transaction-history request produces an event and returns only
    that customer's permitted data.
5.  Every transfer, debit, credit, bill payment, airtime/data purchase,
    card event, refund, reversal, deposit, and withdrawal lifecycle
    event is routed through the gateway.
6.  A transaction initiation receives a risk decision before execution
    where assessment is required.
7.  `REVIEW`, `DECLINE`, and `UNAVAILABLE` are handled safely.
8.  A provider-confirmed success is the only condition that produces a
    completed/successful transaction state.
9.  Duplicate requests do not cause duplicate financial execution.
10. A timeout triggers status reconciliation, not a blind payment retry.
11. AI assistant requests reach n8n and use the authenticated customer's
    context.
12. Customer A cannot access Customer B's account or transaction data.
13. Bank investigators cannot access another bank's data without
    explicit permission.
14. Admin and investigator actions are recorded in audit logs.
15. No secret or private credential is present in browser bundles or Git
    history.
16. The approved logo asset is used consistently and remains unchanged.
17. Mobile and desktop layouts are tested.
18. If n8n, the Risk Engine, or the database is unavailable, the
    interface reports the actual failure state without fabricating
    success or safety.

## 16. Recommended implementation order

1.  Create the GitHub repository and add this master brief.
2.  Confirm the approved logo asset and place it in the shared asset
    location.
3.  Establish the secure backend/API gateway and persistent database
    choice (without Supabase).
4.  Configure the central n8n webhook integration server-side and agree
    on request/response contracts.
5.  Build and test the Customer Website.
6.  Implement the deterministic Python Risk Engine and test scenarios.
7.  Connect AI explanation/investigation workflows without giving the
    LLM decision authority.
8.  Build the Bank Intelligence Dashboard.
9.  Build the Platform Administrator Dashboard.
10. Run end-to-end security, reliability, and demo acceptance tests.
11. Prepare competition demo materials using synthetic data and clearly
    stated prototype limitations.

## 17. Current known integration and open items

  -----------------------------------------------------------------------
  Item                                Current position
  ----------------------------------- -----------------------------------
  Central n8n webhook                 Supplied and recorded in Section 5

  Logo                                Must use the owner's approved
                                      original asset; exact asset
                                      URL/file must be added to the
                                      repository

  Frontend applications               Three separate websites

  Customer website                    First application to complete and
                                      verify

  Risk scoring                        Deterministic Python Risk Engine

  AI                                  Investigation, explanation, and
                                      assistant workflows

  Supabase                            Not to be used

  Real banking/payment connectivity   Not assumed; requires authorised
                                      provider integrations

  Demo data                           Synthetic until authorised data
                                      access exists
  -----------------------------------------------------------------------

------------------------------------------------------------------------

**Implementation instruction:** Treat this file as the project-wide
specification. If a generated feature conflicts with these requirements,
follow this document and explain the conflict instead of silently
weakening the architecture or security rules.
