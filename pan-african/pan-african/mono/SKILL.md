---
name: mono
description: "Integrate with Mono's financial data API for account linking, transaction history, income verification, and identity checks across Nigeria and other African markets. Use this skill whenever the user wants to connect bank accounts, pull transaction data, verify income or identity, build a personal finance app, implement open banking, do credit scoring, or work with Mono's API. Also trigger for 'Mono', 'Mono Connect', 'open banking Nigeria', 'account aggregation Africa', 'bank data API', 'financial data API Africa', 'pull bank transactions', 'income verification Nigeria', or 'KYC bank data'."
---

# Mono Integration Skill

Mono is Africa's leading open banking platform. It lets you securely connect to users' bank accounts to pull financial data — transactions, balances, income, identity, and spending analytics. Think of it as Plaid for Africa, operating across Nigeria, Ghana, Kenya, and South Africa with integrations across 30+ financial institutions.

## When to use this skill

You're building something that needs access to a user's bank data — a personal finance app, a lending platform that needs income verification, a credit scoring engine, an accounting tool that syncs with bank accounts, or any product that benefits from knowing a user's financial picture. Mono handles the bank connections so you don't have to integrate with each bank individually.

## Authentication

Mono uses a secret key for server-side requests:

```
mono-sec-key: live_sk_xxxxx
```

Sandbox: `test_sk_xxxxx`. Production: `live_sk_xxxxx`.

Store in `MONO_SECRET_KEY` env var.

**Base URL:** `https://api.withmono.com/v2`

**Webhook Secret:** Verify incoming webhooks using `mono-webhook-secret` header with HMAC-SHA512.

## The Mono Connect Flow

Before you can pull data, the user needs to link their bank account through Mono Connect (a drop-in widget). Here's how it works:

### Step 1: Initialize Mono Connect Widget

Client-side — embed the Mono Connect widget in your app:

```html
<script src="https://connect.mono.co/connect.js"></script>
<script>
  const connect = new Connect({
    key: "live_pk_xxxxx",  // Your PUBLIC key (not secret)
    onSuccess: function(response) {
      // response.code is a temporary auth code (expires in minutes)
      // Send it to your server to exchange for an account ID
      fetch('/api/mono/exchange', {
        method: 'POST',
        body: JSON.stringify({ code: response.code })
      });
    },
    onClose: function() {
      console.log('Widget closed');
    }
  });
  connect.open();
</script>
```

### Step 2: Exchange Code for Account ID

Server-side — exchange the temporary code for a permanent account ID:

```
POST /accounts/auth
mono-sec-key: {secret_key}
```

```json
{
  "code": "code_xxxxx"
}
```

**Response:**
```json
{
  "status": "successful",
  "message": "Account successfully linked",
  "data": {
    "id": "60d2a8a0xxxxx"
  }
}
```

Save `data.id` — this is the account ID you'll use for all subsequent data requests.

## Core API Reference

### Get Account Information

```
GET /accounts/{account_id}
mono-sec-key: {secret_key}
```

**Response:**
```json
{
  "status": "successful",
  "data": {
    "account": {
      "_id": "60d2a8a0xxxxx",
      "name": "Amina Okafor",
      "account_number": "0123456789",
      "currency": "NGN",
      "balance": 1250000,
      "type": "SAVINGS_ACCOUNT",
      "bvn": "22xxxxx789",
      "institution": {
        "name": "GTBank",
        "bank_code": "058",
        "type": "PERSONAL_BANKING"
      }
    }
  }
}
```

**Note:** `balance` is in the **minor unit** (kobo for NGN). 1,250,000 kobo = ₦12,500.

### Get Transactions

```
GET /accounts/{account_id}/transactions?start=2025-01-01&end=2025-01-31&type=debit&paginate=true&limit=50&page=1
mono-sec-key: {secret_key}
```

Query params:
- `start`, `end` — Date range (YYYY-MM-DD)
- `type` — `debit`, `credit`, or omit for both
- `narration` — Search by transaction description
- `paginate` — `true` for paginated results
- `limit` — Items per page (max 100)
- `page` — Page number

**Response:**
```json
{
  "status": "successful",
  "data": [
    {
      "_id": "txn_xxxxx",
      "type": "debit",
      "amount": 150000,
      "narration": "POS Purchase - ShopRite Ikeja",
      "date": "2025-01-15T14:30:00.000Z",
      "balance": 1100000,
      "currency": "NGN",
      "category": "groceries"
    }
  ],
  "meta": {
    "total": 245,
    "page": 1,
    "previous": null,
    "next": "https://api.withmono.com/v2/accounts/xxx/transactions?page=2"
  }
}
```

### Get Income Information

Pull verified income data — useful for lending decisions and credit scoring:

```
GET /accounts/{account_id}/income
mono-sec-key: {secret_key}
```

**Response:**
```json
{
  "status": "successful",
  "data": {
    "type": "SALARY",
    "amount": 850000,
    "employer": "Tech Company Ltd",
    "confidence": 0.95,
    "total_income": 10200000,
    "annual_income": 10200000,
    "monthly_income": 850000,
    "income_streams": [
      {
        "type": "SALARY",
        "amount": 850000,
        "frequency": "MONTHLY",
        "last_date": "2025-01-28",
        "average_amount": 840000,
        "number_of_occurrences": 12
      }
    ]
  }
}
```

**Note:** Income analysis may take time after account linking. You'll receive a `mono.events.account_income` webhook when analysis completes.

### Get Identity Information

Retrieve KYC data from the user's bank:

```
GET /accounts/{account_id}/identity
mono-sec-key: {secret_key}
```

**Response:**
```json
{
  "status": "successful",
  "data": {
    "fullName": "Amina Chinyere Okafor",
    "email": "amina@email.com",
    "phone": "+2348012345678",
    "gender": "Female",
    "dob": "1990-05-15",
    "bvn": "22xxxxx789",
    "maritalStatus": "Single",
    "addressLine1": "123 Allen Avenue, Ikeja",
    "state": "Lagos",
    "city": "Lagos"
  }
}
```

### Get Spending Analysis

Categorized breakdown of the user's spending patterns:

```
GET /accounts/{account_id}/spending?period=last_6_months
mono-sec-key: {secret_key}
```

**Response:**
```json
{
  "status": "successful",
  "data": {
    "total_spent": 4500000,
    "categories": [
      { "category": "food_and_dining", "amount": 1200000, "percentage": 26.7 },
      { "category": "transport", "amount": 800000, "percentage": 17.8 },
      { "category": "utilities", "amount": 600000, "percentage": 13.3 },
      { "category": "entertainment", "amount": 400000, "percentage": 8.9 },
      { "category": "transfers", "amount": 1500000, "percentage": 33.3 }
    ]
  }
}
```

### Lookup API — Instant Identity Verification

Verify identity information without requiring account linking. Endpoints include:

- **BVN Lookup** — Validate Bank Verification Numbers
- **NIN Lookup** — Validate National Identification Numbers
- **Driver's License Lookup** — Verify driver's license information
- **Account Number Lookup** — Validate account ownership
- **International Passport Lookup** — Verify passport details
- **House Address Verification** — Validate residential addresses

These enable KYC/KYB verification with minimal friction. Contact your Mono account manager to enable Lookup API access.

### DirectPay — Payment Initiation

Initiate one-time direct debits from connected accounts. Useful for payments, subscriptions, and checkout:

```
POST /directpay/initiate
mono-sec-key: {secret_key}
```

```json
{
  "account_id": "60d2a8a0xxxxx",
  "amount": 50000,
  "type": "ONE_TIME",
  "reference": "TXN_123456",
  "description": "Payment for services"
}
```

**Response:**
```json
{
  "status": "successful",
  "data": {
    "id": "payment_xxxxx",
    "reference": "TXN_123456",
    "status": "PENDING"
  }
}
```

DirectPay availability depends on account eligibility and bank support. Verify payment status via webhooks or the Verify endpoint.

### Revoke Account Access

When a user wants to disconnect their bank account:

```
POST /accounts/{account_id}/unlink
mono-sec-key: {secret_key}
```

**Response:**
```json
{
  "status": "successful",
  "message": "Account unlinked successfully"
}
```

## Webhooks

Mono sends webhooks for account events. Verify with your webhook secret:

Mono sends a static `mono-webhook-secret` header value that you compare directly against your configured webhook secret — it is **not** a computed HMAC. Verify as follows:

```javascript
const incomingSecret = req.headers['mono-webhook-secret'];
if (incomingSecret !== process.env.MONO_WEBHOOK_SECRET) {
  return res.status(401).end();
}
```

<!-- TODO: confirm with https://docs.mono.co/docs/financial-data/webhook-introduction — if Mono has added HMAC signing for webhooks in a newer version, update accordingly -->

### Key Webhook Events

**Financial Data Events:**
- `mono.events.account_connected` — User successfully linked an account
- `mono.events.account_updated` — New data available (transactions, balance, etc.). Includes `meta.data_status` field: `available`, `processing`, or `failed`
- `mono.events.account_income` — Income analysis complete (sent after account linking and analysis finishes)
- `mono.events.account_reauthorized` — User re-authenticated (MFA accounts require reauth for fresh data)
- `mono.events.account_unlinked` — Account disconnected

**Payment Events (DirectPay):**
- `direct_debit.payment_successful` — Payment completed successfully
- `direct_debit.payment_failed` — Payment failed

Each webhook payload includes `account_id`, timestamp, and event-specific data.

## Data Sync & Reauthorization

**Important:** Accounts with Multi-Factor Authentication (MFA) require periodic reauthorization before fresh data can be retrieved.

### Reauth Flow

When MFA reauth is needed:
1. Call the Reauth Endpoint with the account ID
2. Generate a reauth link
3. User completes MFA (OTP, security question, or CAPTCHA)
4. `mono.events.account_reauthorized` webhook fires
5. Data becomes available again

**Rate Limiting:** One Data Sync call per account every 2 minutes. Plan data refresh schedules accordingly.

## Error Handling

All errors follow this format:

```json
{
  "status": "failed",
  "message": "Description of what went wrong"
}
```

### Common Error Codes & Solutions

| Code | Error | Cause | Solution |
|------|-------|-------|----------|
| 400 | Bad Request | Invalid params, malformed request, bad date format | Validate all parameters; use YYYY-MM-DD for dates |
| 401 | Unauthorized | Invalid, expired, or missing `mono-sec-key` | Verify secret key in environment; check sandbox vs. production |
| 404 | Not Found | Account ID invalid or account unlinked | Verify account ID exists; check if user unlinked account |
| 429 | Too Many Requests | Rate limit exceeded | Implement exponential backoff; respect Data Sync limits (1 call/2 min per account) |
| 503 | Service Unavailable | Bank API temporarily down or Mono infrastructure issue | Retry with exponential backoff; check Mono status page |

## Important Notes / Gotchas

### Account Reauthorization & Data Freshness

- **MFA accounts** require reauthorization every 30-90 days (varies by bank) before fresh data can be pulled. If reauth is needed and skipped, your API calls will return stale data or error.
- Monitor for `mono.events.account_reauthorized` webhooks and update your user experience accordingly.
- After reauth, data status is `processing` briefly, then transitions to `available`. Poll or wait for the `account_updated` webhook before calling data endpoints.
- Implement a reauth flow in your UI: detect when an account needs reauth and prompt the user to reconnect via the Reauth Widget.

### Data Freshness & Availability

- Transactions and balance data may be **1-24 hours stale** depending on the bank. Real-time data is not guaranteed.
- Some banks may take longer to sync data; plan refresh intervals accordingly (daily is typical).
- Income analysis is not instant; it runs asynchronously after account linking. Always wait for the `mono.events.account_income` webhook before serving income data to users.
- Spending categories are generated from transaction history; more transactions = more accurate categorization.

### Bank Coverage Limitations

- **Nigeria:** 30+ institutions including GTBank, Access, First Bank, UBA, Zenith, Stanbic IBTC, FCMB, Wema, Kuda, and more.
- **Ghana, Kenya, South Africa:** Growing coverage but not all banks are supported. Always gracefully handle account linking failures for unsupported banks.
- Coverage expands regularly; check Mono's website for the latest list.
- Some institutions may have unstable open banking implementations; build retry logic and error handling.

### Connect Widget vs. API

- **Mono Connect** (the widget) is for account linking only. It returns a temporary `code`, not direct data.
- **Mono API** (server-side) is where you pull data using the account ID. The widget and API are separate — don't confuse them.
- Widget keys are public; secret keys are for server-side only. Never expose secret keys in client-side code.

### DirectPay Availability

- DirectPay is not available for all accounts. Some users may not be eligible for one-time debits depending on their bank and account type.
- Gracefully fall back to alternative payment methods (e.g., card payment) if DirectPay fails.
- DirectPay success depends on bank infrastructure; failures are common. Always provide retry options.

### Rate Limiting & Data Sync

- **Data Sync:** 1 call per account every 2 minutes. Aggressive polling will cause rate limit errors (429).
- **Webhook delays:** Webhooks may arrive slightly delayed; don't rely on immediate delivery for critical flows.
- Plan batch operations during off-peak hours if syncing many accounts.
- Respect 429 responses; implement exponential backoff (start with 2s, double each retry, max 32s).

### Sandbox vs. Production

- **Sandbox keys** start with `test_sk_`; **production keys** start with `live_sk_`.
- Sandbox data is mock data; transactions and balances are not real.
- Test all MFA flows, reauth scenarios, and error states in sandbox before production.

### Webhook Security

- Always verify webhook signatures using the `mono-webhook-secret` header and HMAC-SHA512.
- Webhooks may be retried; implement idempotency using the event ID or transaction reference.
- Never trust webhook payloads without signature verification — treat as untrusted until verified.

### Breaking Changes & Versioning

- Always use `/v2` endpoints (newer and more stable than `/v1`).
- Monitor Mono's changelog for deprecations and breaking changes.
- Income API v2 includes new fields; ensure your code handles both old and new response formats for compatibility.

## Common Integration Patterns

### Personal Finance App

1. User connects bank via Mono Connect widget
2. Exchange code → get account ID
3. `GET /accounts/{id}/transactions` to pull transaction history
4. `GET /accounts/{id}/spending` for categorized insights
5. Set up webhooks for `account_updated` to sync new transactions
6. Handle `account_reauthorized` events for MFA accounts

### Lending / Credit Scoring

1. User links account during loan application
2. `GET /accounts/{id}/income` — verify and analyze income
3. `GET /accounts/{id}/transactions` — analyze cash flow patterns (6-12 months)
4. `GET /accounts/{id}/identity` — verify KYC data
5. Use Mono Lookup API for additional identity verification
6. Compute credit score / affordability based on income, expenses, and transaction behavior
7. Handle account reauth for stale data scenarios

### Accounting Integration

1. Business connects bank accounts
2. Periodically pull transactions via `GET /accounts/{id}/transactions`
3. Auto-categorize using Mono's categories + custom rules
4. Reconcile against invoices and expected transfers
5. Use webhooks to stay in sync with new transactions

### Payment Processing (DirectPay)

1. Initiate payment via DirectPay API
2. User authenticates and approves payment in their bank
3. Monitor `direct_debit.payment_successful` webhook
4. Verify payment status if webhook doesn't arrive (reliability fallback)
5. Handle failures gracefully; offer alternative payment methods

## Useful Links

- **API Docs:** https://docs.mono.co
- **Dashboard:** https://app.mono.co
- **Mono Connect Widget:** https://docs.mono.co/docs/financial-data/connect-link
- **Webhook Events:** https://docs.mono.co/docs/financial-data/webhook-introduction
- **Income API v2:** https://docs.mono.co/docs/financial-data/income
- **Lookup API:** https://docs.mono.co/docs/lookup/overview
- **DirectPay Overview:** https://docs.mono.co/docs/payments/overview
- **Data Sync Guide:** https://docs.mono.co/docs/financial-data/data-sync
- **Error Reference:** https://docs.mono.co/docs/errors
- **Bank Coverage:** https://mono.co/coverage
