---
name: kuda
description: "Integrate with the Kuda Bank API for Nigerian digital banking. Use this skill when building fintech apps, marketplaces, or business tools that need virtual accounts, transfers, bill payments, or banking features. Trigger on mentions of 'Kuda', 'virtual accounts Nigeria', 'Kuda transfers', 'digital banking API', or fintech applications."
---

# Kuda Bank Integration Skill

Kuda is Nigeria's first mobile-only bank (licensed by CBN) offering open banking APIs for account creation, transfers, bill payments, and fund management. Embed banking features directly into your application with virtual accounts, instant settlement, and comprehensive transaction management.

## When to use this skill

You're building a fintech platform, marketplace, or business tool that needs banking infrastructure — creating customer accounts, collecting payments into virtual accounts, distributing payouts, managing savings, or handling bill payments. Kuda provides the underlying banking backbone for embedded financial services.

## Authentication

Kuda uses **API key authentication** (not OAuth). Obtain your API key from the Kuda Business dashboard at **business.kuda.com**:

1. Log in to business.kuda.com with your Kuda Business account
2. Navigate to Settings > API Keys
3. Generate a new API key (keep it secret)
4. Toggle between **Sandbox** and **Live** modes in the dashboard

Include the API key in all requests:

```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

Store keys in environment variables like `KUDA_API_KEY`. Never hardcode credentials or commit them to version control.

**Base URLs:**
- Sandbox: `https://kuda-openapi-sandbox.kuda.com`
- Live: `https://kuda-openapi.kuda.com`

## Core API Reference

### Create a Virtual Account

Generate a new virtual account for a customer. Each account receives a unique Nigerian bank account number (Kuda's 090287 bank code).

```
POST /Account/CreateVirtualAccount
```

**Body:**
```json
{
  "email": "customer@example.com",
  "firstName": "Chinedu",
  "lastName": "Okafor",
  "phoneNumber": "+2348012345678",
  "businessName": "Chinedu Logistics",
  "trackingRef": "USER-2025-0001"
}
```

**Critical:** `trackingRef` must be globally unique per request (use UUIDs or timestamps). Kuda uses this for idempotency — sending duplicate requests with the same `trackingRef` returns the same account without creating duplicates.

**Response:**
```json
{
  "success": true,
  "data": {
    "accountNumber": "1234567890",
    "bankName": "Kuda Bank",
    "bankCode": "090287",
    "accountName": "Chinedu Okafor",
    "customerId": "CUST_a7f8d9e2",
    "trackingRef": "USER-2025-0001"
  }
}
```

Share `accountNumber` and `bankCode` with customers to receive transfers. This is a real Nigerian bank account.

### Get Account Balance

Retrieve current balance and ledger balance (available funds for transfers).

```
POST /Account/GetBalance
```

**Body:**
```json
{
  "accountNumber": "1234567890",
  "trackingRef": "BAL-CHECK-2025-001"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "accountNumber": "1234567890",
    "balance": 500000,
    "currency": "NGN",
    "ledgerBalance": 500000
  }
}
```

Amounts are in **kobo** (₦1 = 100 kobo). `ledgerBalance` reflects available funds after pending transactions.

### Send Single Transfer

Transfer funds from a virtual account to any Nigerian bank account.

```
POST /Transfer/SingleFund
```

**Body:**
```json
{
  "sourceAccountNumber": "1234567890",
  "destinationAccountNumber": "9876543210",
  "destinationBankCode": "058",
  "amount": 50000,
  "reference": "TRF-ORDER-2025-001",
  "narration": "Payment for order #5432",
  "trackingRef": "TRF-2025-0001"
}
```

**Key details:**
- `amount` is in kobo (50000 = ₦500)
- `reference` is idempotent — resubmit the same reference if the first request fails without duplicating transfers
- `destinationBankCode` is the receiving bank's code (058=GTB, 033=FirstBank, 011=First City, etc.)
- `trackingRef` must be unique per request

**Response:**
```json
{
  "success": true,
  "data": {
    "reference": "TRF-ORDER-2025-001",
    "transactionId": "TXN_5f8a9c2d",
    "status": "success",
    "amount": 50000,
    "fee": 10,
    "sourceAccountNumber": "1234567890",
    "destinationAccountNumber": "9876543210"
  }
}
```

**Gotcha:** The response shows `success: true`, but transfers settle asynchronously. Listen for `transfer.completed` webhooks or poll status for confirmation before marking transfers as complete.

### Send Bulk Transfers

Distribute funds to multiple accounts in a single API call.

```
POST /Transfer/BulkFund
```

**Body:**
```json
{
  "sourceAccountNumber": "1234567890",
  "transfers": [
    {
      "destinationAccountNumber": "9876543210",
      "destinationBankCode": "058",
      "amount": 50000,
      "narration": "Payout to vendor 1"
    },
    {
      "destinationAccountNumber": "5555555555",
      "destinationBankCode": "033",
      "amount": 75000,
      "narration": "Payout to vendor 2"
    }
  ],
  "trackingRef": "BULK-PAY-2025-001"
}
```

Each transfer in the array must have its own destination details. Use for marketplace payouts, salary distributions, or batch settlements.

### Get Transaction History

Retrieve account transactions within a date range.

```
POST /Account/GetTransactions
```

**Body:**
```json
{
  "accountNumber": "1234567890",
  "startDate": "2025-01-01",
  "endDate": "2025-02-28",
  "limit": 100
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "transactions": [
      {
        "transactionId": "TXN_5f8a9c2d",
        "type": "credit",
        "amount": 100000,
        "reference": "INCOMING-001",
        "narration": "Transfer from customer",
        "timestamp": "2025-02-24T14:30:00Z",
        "status": "success"
      },
      {
        "transactionId": "TXN_7c3e1f9a",
        "type": "debit",
        "amount": 50000,
        "reference": "TRF-ORDER-2025-001",
        "narration": "Payment for order",
        "timestamp": "2025-02-24T10:15:00Z",
        "status": "success"
      }
    ]
  }
}
```

Use for reconciliation, customer statements, and audit trails.

### Pay Bills and Airtime

Integrate bill payments, airtime top-ups, and data purchases.

```
POST /Bill/Pay
```

**Body:**
```json
{
  "billerId": "airtel-airtime",
  "customerReference": "08012345678",
  "amount": 5000,
  "sourceAccountNumber": "1234567890",
  "reference": "AIRTIME-2025-001",
  "trackingRef": "BILL-2025-0001"
}
```

Supported billers: Airtel, MTN, GLO, 9Mobile (airtime/data), DSTV, GoTV, Smile (internet), electricity providers, etc.

### Verify Account Number

Validate account details before transferring to prevent sending funds to wrong recipients.

```
POST /Account/VerifyAccount
```

**Body:**
```json
{
  "accountNumber": "9876543210",
  "bankCode": "058"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "accountNumber": "9876543210",
    "accountName": "Chinedu Adeyemi",
    "bankName": "Guaranty Trust Bank",
    "valid": true
  }
}
```

**Essential:** Always verify destination accounts. Transfers cannot be reversed once sent, and account name mismatches are your only safety net.

## Webhooks

Kuda sends real-time events for transfers and account activity. Register your webhook URL in the Kuda Business dashboard at business.kuda.com > Settings > Webhooks.

**Webhook verification:**
```javascript
const crypto = require('crypto');

app.post('/webhook/kuda', (req, res) => {
  const signature = req.headers['x-kuda-signature'];
  const hash = crypto
    .createHmac('sha256', process.env.KUDA_WEBHOOK_SECRET)
    .update(JSON.stringify(req.body))
    .digest('hex');

  if (hash !== signature) {
    return res.status(401).send('Invalid signature');
  }

  // Process event
  const event = req.body.event;
  const data = req.body.data;

  if (event === 'transfer.completed') {
    console.log(`Transfer ${data.reference} completed`);
    // Update order status, mark payout as sent
  } else if (event === 'account.credited') {
    console.log(`Account ${data.accountNumber} received ₦${data.amount / 100}`);
    // Update customer balance, process payment
  }

  res.json({ received: true });
});
```

**Key webhook events:**
- `transfer.completed` — transfer successfully settled
- `transfer.failed` — transfer failed (insufficient funds, invalid account, etc.)
- `account.credited` — funds received in virtual account
- `account.debited` — funds sent from account
- `account.created` — new virtual account provisioned

Webhooks may arrive **after** your API response. Don't immediately mark transfers as complete — always confirm via webhooks or status polling.

## Common Integration Patterns

### Marketplace Seller Onboarding
```
1. Seller registers → POST /Account/CreateVirtualAccount
2. Store accountNumber and bankCode
3. Display account details to seller
4. Buyer transfers money to seller's account
5. Listen for account.credited webhook
6. POST /Account/GetBalance to confirm receipt
7. Initiate marketplace payout or hold escrow
```

### Payment Collection & Distribution
```
1. Create master collection account → POST /Account/CreateVirtualAccount
2. Customers transfer to collection account
3. Listen for account.credited webhooks
4. POST /Account/GetBalance to check available balance
5. POST /Transfer/BulkFund to distribute to vendors/staff
6. Listen for transfer.completed webhooks for confirmation
```

### Savings & Wallet Management
```
1. Create user savings account → POST /Account/CreateVirtualAccount
2. User deposits funds into savings account
3. Poll or listen for account.credited events
4. POST /Account/GetTransactions for savings history
5. POST /Transfer/SingleFund for withdrawals
6. POST /Account/GetBalance to show savings progress
```

## Error Handling

Kuda returns structured error responses with status codes and actionable messages:

```json
{
  "success": false,
  "message": "Insufficient balance in source account",
  "statusCode": 400
}
```

**Common errors and handling:**
- **400 Bad Request**: Validation error (invalid account format, negative amount, missing fields). Check request structure.
- **401 Unauthorized**: Invalid or expired API key. Verify credentials in Kuda Business dashboard.
- **403 Forbidden**: API key lacks permission (e.g., live transfers with sandbox key). Check environment and dashboard settings.
- **404 Not Found**: Account doesn't exist. Verify accountNumber or create account first.
- **429 Too Many Requests**: Rate limited. Implement exponential backoff (1s, 2s, 4s...).
- **500 Server Error**: Kuda service issue. Retry with exponential backoff. Check status.kuda.com.

For transfers, **always check the `status` field** in response data. A `success: true` response might still have `status: "pending"` or `status: "failed"`.

## Important Notes and Gotchas

**1. Kobo units:** All amounts are in kobo (100 kobo = ₦1). ₦500 = 50000 kobo. Double-check conversions.

**2. Bank codes:** Standard CBN codes must be exact (058=GTB, 033=FirstBank, 011=First City, 035=Wema, 050=ecoBank). Invalid codes fail transfers silently.

**3. Tracking references:** `trackingRef` enables idempotency — keep them unique globally per request. Reuse for retries, but changing `trackingRef` creates new operations.

**4. Transfer settlement:** Transfers show `success: true` immediately but settle asynchronously. **Always listen for webhooks** to confirm completion before marking transfers as done.

**5. Account verification:** Destination account names should match recipients. Mismatches indicate fraud or wrong accounts — abort the transfer. Kuda cannot reverse sent transfers.

**6. Virtual account limits:** Regulatory limits apply (CBN compliance). Check current transfer limits in Kuda Business dashboard before initiating bulk transfers.

**7. Webhook delays:** Webhooks may arrive 30+ seconds after API response. Don't poll excessively; webhooks are your source of truth. Implement timeout handling for edge cases.

**8. API key rotation:** Rotate API keys periodically in the dashboard. Old keys remain valid until explicitly revoked.

**9. Sandbox vs. Live:** Sandbox uses test data and fake settlement. Switch to **Live** mode in the dashboard and use live API URLs only in production.

**10. Bill payment billers:** Biller IDs and requirements vary by region. Fetch available billers from the Kuda dashboard or `/Bill/GetBillers` endpoint before integrating.

## Useful Links

- Official Documentation: https://docs.kuda.com
- GitBook API Reference: https://kudabank.gitbook.io/kudabank
- Developer Portal: https://developer.kuda.com
- Kuda Business Dashboard: https://business.kuda.com
- Status Page: https://status.kuda.com
