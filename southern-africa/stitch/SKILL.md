---
name: stitch
description: "Integrate with Stitch's payment and data API for South African commerce. Use this skill whenever the user wants to accept payments in South Africa, process EFT or instant payments, link bank accounts, access financial data, handle ZAR transactions, or work with Stitch in any way. Also trigger for 'Stitch', 'Stitch Money', 'South African payments', 'ZAR checkout', 'instant EFT South Africa', 'LinkPay', 'pay by bank South Africa', 'account linking South Africa', or when the user needs to move money in or pull financial data from South Africa."
---

# Stitch Integration Skill

Stitch is South Africa's leading pay-by-bank and financial data platform. It enables instant EFT payments (no card network fees), bank account linking, transaction data access, and payouts — all via a modern GraphQL API. It's the go-to for businesses building in the South African financial ecosystem.

## When to use this skill

You're building something that touches the South African financial system — accepting ZAR payments, pulling bank data for lending or accounting, processing payouts, or building open banking features. Stitch is especially strong for pay-by-bank (instant EFT) which avoids card fees and has higher success rates in SA.

## Authentication

Stitch uses OAuth 2.0 with client credentials:

### Get Access Token

<!-- TODO: verify auth method — Stitch documentation recommends JWT client assertion
(RS256 signed JWT using a Stitch-issued X.509 certificate + private key) for
client credentials, not a plain client_secret. If your account uses client_secret,
this works; if not, see https://docs.stitch.money/authentication/client-tokens
for the certificate-based flow -->

```
POST https://secure.stitch.money/connect/token
Content-Type: application/x-www-form-urlencoded
```

```
grant_type=client_credentials
&client_id=your_client_id
&client_secret=your_client_secret
&scope=client_paymentrequest client_bankaccountverification
```

**Response:**
```json
{
  "access_token": "xxxxx",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "client_paymentrequest"
}
```

Store credentials in env vars: `STITCH_CLIENT_ID`, `STITCH_CLIENT_SECRET`.

> ⚠️ **JWT Client Assertion (Recommended for Production):** The `client_secret` flow above works for some account types, but Stitch recommends **JWT client assertion** for production integrations. This uses an RS256-signed JWT with your X.509 certificate and private key instead of a `client_secret`. To use it, replace `client_secret=...` with `client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&client_assertion={signed_jwt}`. Generate the JWT using your private key (RS256) with claims: `iss`, `sub` (both = `client_id`), `aud` (`https://secure.stitch.money/connect/token`), `jti`, `iat`, `exp`. See [Stitch auth docs](https://docs.stitch.money/authentication/client-tokens) for certificate setup. If your integration fails with `client_secret`, this is the likely required path.

**API URL:** `https://api.stitch.money/graphql` (GraphQL endpoint)

All requests are GraphQL queries/mutations sent as POST to this single endpoint.

## Important: GraphQL API

Unlike the other Africa Stack skills which use REST, Stitch uses **GraphQL**. All requests go to one endpoint:

```
POST https://api.stitch.money/graphql
Authorization: Bearer {access_token}
Content-Type: application/json
```

```json
{
  "query": "mutation { ... }",
  "variables": { ... }
}
```

## Core API Reference

### Create a Payment Request (Pay by Bank)

This initiates an instant EFT payment — the user selects their bank and completes payment via their banking app.

```graphql
mutation CreatePaymentRequest(
  $amount: MoneyInput!,
  $payerReference: String!,
  $beneficiaryReference: String!,
  $externalReference: String,
  $expiresAt: DateTime
) {
  clientPaymentInitiationRequestCreate(input: {
    amount: $amount,
    payerReference: $payerReference,
    beneficiaryReference: $beneficiaryReference,
    externalReference: $externalReference,
    expiresAt: $expiresAt
  }) {
    paymentInitiationRequest {
      id
      url
      amount {
        quantity
        currency
      }
      payerReference
      beneficiaryReference
      expiresAt
      status {
        __typename
        ... on PaymentInitiationRequestCompleted {
          date
          amount { quantity currency }
          payer { bankId accountNumber }
        }
        ... on PaymentInitiationRequestPending {
          paymentInitiationRequest { id }
        }
        ... on PaymentInitiationRequestExpired {
          date
        }
        ... on PaymentInitiationRequestCancelled {
          date
          reason
        }
      }
    }
  }
}
```

**Variables:**
```json
{
  "amount": {
    "quantity": 150.00,
    "currency": "ZAR"
  },
  "payerReference": "Order #123",
  "beneficiaryReference": "MyStore-123",
  "externalReference": "ext_ref_123",
  "expiresAt": "2025-02-28T23:59:59Z"
}
```

**Important:** Amounts are in **major currency units** (Rands, not cents). ZAR 150.00 = `150.00`.

**Response includes `url`** — redirect the customer there to complete the payment. They select their bank, authenticate with their banking app, and approve the payment.

**Expiry:** If `expiresAt` is provided, the payment request automatically transitions to `PaymentInitiationRequestExpired` at that time if not completed. No webhook is required — the state is determined when you query it.

### Create a LinkPay Payment

LinkPay creates a shareable payment link:

```graphql
mutation CreateLinkPay($input: LinkPayCreateInput!) {
  linkPayCreate(input: $input) {
    linkPay {
      id
      url
      amount { quantity currency }
      status
      expiresAt
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "amount": { "quantity": 250.00, "currency": "ZAR" },
    "title": "Invoice #456",
    "description": "Consulting services - January 2025",
    "externalReference": "inv_456",
    "expiresAt": "2025-02-28T23:59:59Z"
  }
}
```

### Check Payment Status

```graphql
query GetPaymentRequestStatus($id: ID!) {
  node(id: $id) {
    ... on PaymentInitiationRequest {
      id
      amount { quantity currency }
      expiresAt
      status {
        __typename
        ... on PaymentInitiationRequestCompleted {
          date
          amount { quantity currency }
        }
        ... on PaymentInitiationRequestCancelled {
          date
          reason
        }
        ... on PaymentInitiationRequestExpired {
          date
        }
        ... on PaymentInitiationRequestPending {
          paymentInitiationRequest { id }
        }
      }
    }
  }
}
```

Status types: `PaymentInitiationRequestPending`, `PaymentInitiationRequestCompleted`, `PaymentInitiationRequestCancelled`, `PaymentInitiationRequestExpired`.

### Link a Bank Account (Account Linking)

Allow users to connect their bank accounts for data access:

```graphql
mutation CreateAccountLinkingRequest($input: ClientAccountLinkingRequestCreateInput!) {
  clientAccountLinkingRequestCreate(input: $input) {
    accountLinkingRequest {
      id
      url
      status
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "accountFilter": {
      "bankIds": ["absa", "fnb", "standardbank", "nedbank", "capitec", "investec", "tymebank", "discovery_bank"]
    }
  }
}
```

Redirect user to `url` — they authenticate with their bank and authorize data access. The linked account ID comes back via webhook (`account.linked` event).

### Get Linked Accounts

```graphql
query GetLinkedAccounts {
  user {
    bankAccounts {
      edges {
        node {
          id
          name
          bankId
          accountNumber
          accountType
          currentBalance {
            quantity
            currency
          }
          availableBalance {
            quantity
            currency
          }
        }
      }
    }
  }
}
```

### Get Account Transactions

```graphql
query GetTransactions($accountId: ID!, $first: Int, $after: String) {
  node(id: $accountId) {
    ... on BankAccount {
      transactions(first: $first, after: $after) {
        edges {
          node {
            id
            amount { quantity currency }
            date
            description
            reference
            runningBalance { quantity currency }
            debitOrCredit
            category
          }
        }
        pageInfo {
          hasNextPage
          endCursor
        }
      }
    }
  }
}
```

**Variables:**
```json
{
  "accountId": "account_xxxxx",
  "first": 50,
  "after": null
}
```

Uses cursor-based pagination. Pass `pageInfo.endCursor` as `after` for next page.

### Get Account Balance

```graphql
query GetBalance($accountId: ID!) {
  node(id: $accountId) {
    ... on BankAccount {
      currentBalance {
        quantity
        currency
      }
      availableBalance {
        quantity
        currency
      }
      name
      bankId
      accountType
    }
  }
}
```

### Verify Bank Account Details

Before disbursing funds, verify the bank account exists and belongs to the stated owner:

```graphql
query VerifyBankAccount($input: VerifyBankAccountDetailsInput!) {
  verifyBankAccountDetails(input: $input) {
    accountHolder {
      accountHolderName
      accountHolderIdentifier
      accountHolderType
    }
    bankAccount {
      bankId
      accountNumber
      accountType
    }
    verified
  }
}
```

**Variables:**
```json
{
  "input": {
    "bankId": "fnb",
    "accountNumber": "1234567890",
    "accountHolderIdentifier": "9001010000000",
    "accountHolderName": "John Dlamini"
  }
}
```

### Create a Payout (Disbursement)

Send money to a bank account:

```graphql
mutation CreateDisbursement($input: ClientDisbursementCreateInput!) {
  clientDisbursementCreate(input: $input) {
    disbursement {
      id
      amount { quantity currency }
      status {
        __typename
        ... on DisbursementInitiated {
          initiatedAt
        }
        ... on DisbursementCompleted {
          completedAt
          reference
        }
        ... on DisbursementFailed {
          failedAt
          reason
        }
      }
      bankBeneficiary {
        bankAccountNumber
        bankId
        name
        accountType
      }
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "amount": { "quantity": 5000.00, "currency": "ZAR" },
    "bankBeneficiary": {
      "bankAccountNumber": "1234567890",
      "bankId": "fnb",
      "name": "John Dlamini",
      "accountType": "current"
    },
    "nonce": "unique_payout_ref_123",
    "externalReference": "payout_jan_001"
  }
}
```

**Bank IDs:** `absa`, `fnb` (First National Bank), `standardbank`, `nedbank`, `capitec`, `investec`, `tymebank`, `discovery_bank`.

## Webhooks

Configure webhooks in the Stitch dashboard. Verify the signature:

```javascript
const crypto = require('crypto');
const signature = req.headers['x-stitch-signature'];
const payload = JSON.stringify(req.body);
const expected = crypto.createHmac('sha256', webhookSecret).update(payload).digest('hex');
if (signature !== expected) {
  return res.status(401).end();
}
```

**Webhook Events:**

- `payment_initiation_request.authorisation_required` — User needs to authorize payment
- `payment_initiation_request.completed` — Payment successfully processed
- `payment_initiation_request.cancelled` — Payment cancelled by user
- `payment_initiation_request.expired` — Payment request expired before completion
- `account_linking_request.completed` — Bank account successfully linked
- `account_linking_request.cancelled` — Account linking cancelled
- `disbursement.initiated` — Payout sent to bank
- `disbursement.completed` — Payout settled in beneficiary account
- `disbursement.failed` — Payout failed

**Webhook Payload Structure:**
```json
{
  "event_type": "payment_initiation_request.completed",
  "event_id": "evt_xxxxx",
  "created_at": "2025-02-24T10:30:00Z",
  "resource": {
    "id": "payment_xxxxx",
    "amount": { "quantity": 150.00, "currency": "ZAR" },
    "status": "completed",
    "payer": { "bankId": "fnb", "accountNumber": "1234567890" }
  }
}
```

## Common Integration Patterns

### E-commerce pay-by-bank checkout
1. `CreatePaymentRequest` mutation → get redirect URL
2. Customer selects bank → approves payment in banking app
3. Webhook fires `payment_initiation_request.completed`
4. Query status to confirm → fulfill order
5. Lower fees than card payments (no Visa/Mastercard interchange)

### Lending / credit assessment
1. `CreateAccountLinkingRequest` → user links bank
2. `GetLinkedAccounts` → get account IDs
3. `GetTransactions` → analyze cash flow
4. Build credit score from transaction patterns
5. Income verification from salary deposits

### Marketplace payouts
1. Collect payments via payment requests
2. Calculate commissions
3. `CreateDisbursement` to pay each seller
4. Track via webhooks

## Error Handling

GraphQL errors come in the response body. Even successful HTTP responses (200) can contain GraphQL errors:

```json
{
  "data": {
    "clientPaymentInitiationRequestCreate": null
  },
  "errors": [
    {
      "message": "Invalid amount",
      "extensions": {
        "code": "VALIDATION_ERROR",
        "path": ["clientPaymentInitiationRequestCreate"]
      }
    }
  ]
}
```

**Always check the `errors` array** — a 200 response doesn't mean success in GraphQL.

**Common error codes:**
- `UNAUTHORIZED` — Invalid or expired token, or missing scope
- `INVALID_SCOPE` — Token missing required scope for operation
- `VALIDATION_ERROR` — Invalid input (amount, references, etc.)
- `PAYMENT_REQUEST_NOT_FOUND` — Bad payment request ID
- `BANK_ACCOUNT_NOT_FOUND` — Bad account ID or user not authorized
- `INSUFFICIENT_FUNDS` — Not enough balance for disbursement
- `INVALID_BANK_ACCOUNT` — Bad account number or bank ID (verify with `verifyBankAccountDetails` first)
- `RATE_LIMITED` — Too many requests (implement exponential backoff)
- `INTERNAL_SERVER_ERROR` — Stitch server error (retry with backoff)

**Error Handling Pattern:**
```javascript
async function executeGraphQL(query, variables) {
  const response = await fetch('https://api.stitch.money/graphql', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ query, variables })
  });

  const result = await response.json();

  // Check both HTTP status AND GraphQL errors
  if (!response.ok || result.errors) {
    const error = result.errors?.[0];
    console.error('GraphQL Error:', error?.message);
    console.error('Error Code:', error?.extensions?.code);
    throw new Error(error?.message || 'GraphQL request failed');
  }

  return result.data;
}
```

## Important Notes / Gotchas

### GraphQL is not REST
- All Stitch API requests use GraphQL, not REST. There is a single endpoint: `https://api.stitch.money/graphql`
- HTTP 200 responses can still contain GraphQL errors in the `errors` array — always check both
- Use variables for all dynamic data; inline values in queries are error-prone
- Fragments with `__typename` (like `... on PaymentInitiationRequestCompleted`) are required to handle union types

### South African Bank Support
- Stitch supports all major South African banks: ABSA, FNB, Standard Bank, Nedbank, Capitec, Investec, Tymebank, Discovery Bank
- Not all account types are supported for account linking — cheque/current accounts with active internet banking only
- Credit facilities, savings accounts, and rewards wallets cannot be linked
- Bank EAP (Electronic Account Payment) limits are set by each bank and constrain disbursement amounts — Stitch doesn't enforce limits

### EFT Settlement Times
- **Pay by Bank (incoming):** T+1 (typically 12-24 hours, depends on receiving bank)
- **Payouts (outgoing):** Stitch supports multiple payment rails:
  - **RTC (Real-Time Credit):** Instant settlement, 24/7, 365 days/year
  - **RPP (Rapid Payment Platform):** Instant settlement, business hours
  - **Standard EFT:** T+1 (batch processing, cheaper)
- Payouts settle 24/7, including weekends and public holidays
- Verify actual settlement timing with Stitch — it depends on your specific account configuration

### Payment Request Expiry
- If you set `expiresAt` on a payment request, it automatically transitions to `PaymentInitiationRequestExpired` when the deadline passes
- **No webhook is fired on expiry** — the expiry state is determined when you query the payment status
- Always query payment status before taking action, especially for critical flows
- Expired payments cannot be resumed — create a new payment request if needed

### Client Assertion JWT
- Stitch currently uses **client secrets** for OAuth 2.0 client credentials flow, not JWT client assertions
- If you need client assertion JWT support, contact Stitch support — it may be available as an enterprise option
- Always store `client_secret` securely (environment variables, not hardcoded)
- Token lifetime is 1 hour (`expires_in: 3600`) — cache tokens and refresh before expiry

### Scope-Based Access Control
- Stitch uses OAuth 2.0 scopes to control which operations your token can perform
- Common scopes: `client_paymentrequest`, `client_bankaccountverification`, `client_disbursement`
- If a field returns `null` or an error like "insufficient scope", add the missing scope to your token request
- The API response will tell you which scopes are required for specific fields

### Account Linking Requires User Authorization
- Redirecting a user to `accountLinkingRequest.url` is required — there's no programmatic account linking
- The account ID comes back via webhook (`account.linked` event) after user authorization
- Linked accounts are user-scoped — your app doesn't get access until the user grants it
- A single user can link multiple accounts; iterate through `user.bankAccounts.edges` to get all of them

### Transaction History & Data Access
- Transaction data is only available for accounts the user has linked in your app
- Historical transaction data depends on what the user's bank exposes — typically 6-24 months
- Category classification (`category` field) is provided by Stitch, not the bank
- Pagination is cursor-based (`after`, `endCursor`) — don't use offset-based pagination

### Disbursement Verification
- **Always verify bank account details before disbursing** using `verifyBankAccountDetails` query
- Verification prevents fraud and accounts opened under false names
- The `accountHolderIdentifier` must match the identity or business registration number
- If verification fails, reject the payout — don't attempt it anyway
- Verification does not guarantee the account is correct; it confirms the account exists and the owner matches

### Amount Precision
- Amounts use `MoneyInput` with `quantity` (decimal) and `currency` (e.g., "ZAR")
- **Use major units (Rands), not cents:** 150.00 ZAR = `"quantity": 150.00`, not `15000`
- Stitch has no upper limit for payment amounts, but individual banks may (check EAP limits)
- There's a transaction minimum (typically R1.00), but no documented maximum in public docs — contact Stitch for limits

### Webhook Reliability & Idempotency
- Webhooks can be delivered multiple times — implement idempotency with `event_id`
- Store `event_id` in your database and skip duplicate webhook deliveries
- Don't rely solely on webhooks for critical flows — always query state to confirm
- Webhook signature verification is **mandatory** for production

### Rate Limiting
- Stitch imposes rate limits on GraphQL requests
- If you hit a rate limit, you'll receive a `RATE_LIMITED` error
- Implement exponential backoff (e.g., start with 1s, double each retry, max 60s)
- For high-volume operations, contact Stitch about enterprise rate limits

## South African Bank Details

When working with SA bank accounts, you'll encounter:
- **Bank ID:** Short identifier (`fnb`, `absa`, `standardbank`, `nedbank`, `capitec`, `investec`, `tymebank`, `discovery_bank`)
- **Account Number:** 10-11 digit number
- **Branch Code:** 6-digit universal branch code (Stitch handles this automatically via bank ID)
- **Account Type:** `current`, `savings`, `cheque`

## Useful Links

- **API Docs:** https://docs.stitch.money
- **Client Tokens Reference:** https://docs.stitch.money/authentication/client-tokens
- **Error Handling:** https://docs.stitch.money/graphql/error-handling
- **Webhook Events:** https://docs.stitch.money/webhooks
- **Bank Account Verification:** https://docs.stitch.money/payment-products/bank-account-verification/integration-process
- **Pay by Bank Integration:** https://docs.stitch.money/payment-products/payins/paybybank/integration-process
- **Disbursements/Payouts:** https://docs.stitch.money/payment-products/payouts/disbursements
- **GraphQL Explorer:** https://stitch.money/docs/graphql-explorer
- **Dashboard:** https://app.stitch.money
- **Support:** https://support.stitch.money
