---
triggers:
  - "PawaPay"
  - "mobile money aggregation"
  - "African mobile money API"
  - "pan-African payouts"
  - "mobile money deposits"
  - "mobile money refunds"
  - "Africa payment aggregator"
  - "sub-Saharan mobile money"
---

# PawaPay API Integration Guide

## Overview

PawaPay is a leading pan-African mobile money aggregation platform providing a unified API to process deposits, payouts, and refunds across 19+ countries and 15+ mobile money operators in Sub-Saharan Africa. Instead of integrating with individual mobile money operators separately, PawaPay provides a single API endpoint for all your mobile money payment needs across the continent.

**Key Statistics:**
- Covers 19+ African countries
- Integrates 15+ mobile money operators
- Processes 3+ million transactions daily
- Single API, single dashboard

## When to Use PawaPay

Use PawaPay when you need to:

- **Collect payments** from customers via mobile money (deposits/collections)
- **Distribute funds** to users via mobile money (payouts/disbursements)
- **Process refunds** to customer mobile money wallets
- **Reach customers** across multiple African countries without managing separate integrations
- **Perform bulk transfers** to many recipients efficiently
- **Build cross-border payment solutions** within Africa
- **Offer remittance services** across African markets
- **Scale pan-African operations** with a single API instead of multiple provider integrations

**Not suitable for:**
- Non-financial transactions
- Users requiring instant (non-async) transaction processing
- Markets outside Sub-Saharan Africa (not currently supported)

## Authentication

PawaPay uses Bearer token authentication with optional request signing for enhanced security.

### Basic Bearer Token Authentication

All API requests (except MMO Availability check) require an `Authorization` header:

```bash
curl -X POST https://api.sandbox.pawapay.io/deposits \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

### Getting Your API Token

1. Log in to your [PawaPay Dashboard](https://dashboard.pawapay.io/)
2. Navigate to Settings → API Token
3. Copy your Bearer token
4. Store it securely (environment variables, secrets manager)

### Optional: Request Signing (RFC-9421)

For enhanced security, PawaPay supports signed requests using ECDSA P-256 SHA-256:

**Headers required:**
- `Content-Digest`: SHA-256 or SHA-512 hash of request body
- `Signature-Date`: Timestamp of request
- `Signature`: Digital signature
- `Signature-Input`: Parameters used to generate signature

**Example with SHA-256 signing:**

```javascript
const crypto = require('crypto');

function generateContentDigest(body) {
  const hash = crypto.createHash('sha256').update(body).digest('base64');
  return `sha-256=:${hash}:`;
}

const body = JSON.stringify({
  depositId: 'dep-123',
  amount: 1000,
  correspondent: 'MTN_MW'
});

const headers = {
  'Authorization': 'Bearer YOUR_API_TOKEN',
  'Content-Digest': generateContentDigest(body),
  'Content-Type': 'application/json',
  'Signature-Date': new Date().toISOString()
};
```

**Reference:** See [PawaPay signatures-node-example](https://github.com/pawaPay/signatures-node-example) for complete implementation.

## Environment URLs

| Environment | Base URL |
|------------|----------|
| **Sandbox** | `https://api.sandbox.pawapay.io` |
| **Production** | `https://api.pawapay.io` |

> **Note:** Only the base URL and API token differ between environments. All endpoint paths are identical.

## Core API Reference

### 1. Deposits (Collections)

Initiate a request to collect funds from a customer's mobile money wallet to your PawaPay account.

**Endpoint:** `POST /deposits`

**Request:**
```json
{
  "depositId": "dep-20240220-001",
  "amount": 5000,
  "correspondent": "MTN_MW",
  "depositor": {
    "type": "MSISDN",
    "address": "265999999999"
  },
  "metadata": {
    "orderRef": "ORDER-123",
    "customerId": "CUST-456"
  }
}
```

**Response (202 Accepted):**
```json
{
  "depositId": "dep-20240220-001",
  "status": "SUBMITTED"
}
```

**Status Values:**
- `SUBMITTED`: Request accepted and sent to mobile operator
- `WAITING_FOR_CUSTOMER_INPUT`: Awaiting customer PIN confirmation
- `COMPLETED`: Deposit successful
- `FAILED`: Deposit failed

**Check Deposit Status:**
```bash
GET /deposits/{depositId}
```

### 2. Payouts (Disbursements)

Send funds from your PawaPay account to a customer's mobile money wallet.

**Endpoint:** `POST /payouts`

**Request:**
```json
{
  "payoutId": "payout-20240220-001",
  "amount": 2500,
  "correspondent": "MTN_MW",
  "recipient": {
    "type": "MSISDN",
    "address": "265999888888"
  },
  "metadata": {
    "reason": "salary",
    "employeeId": "EMP-789"
  }
}
```

**Response (202 Accepted):**
```json
{
  "payoutId": "payout-20240220-001",
  "status": "SUBMITTED"
}
```

**Status Values:**
- `SUBMITTED`: Request sent to mobile operator
- `WAITING_FOR_CUSTOMER_INPUT`: Recipient action required
- `COMPLETED`: Payout successful
- `FAILED`: Payout failed

**Check Payout Status:**
```bash
GET /payouts/{payoutId}
```

### 3. Bulk Payouts

Send funds to multiple recipients in a single operation.

**Endpoint:** `POST /bulk-payouts`

**Request:**
```json
{
  "bulkPayoutId": "bulk-20240220-001",
  "payouts": [
    {
      "payoutId": "payout-001",
      "amount": 2000,
      "correspondent": "MTN_MW",
      "recipient": {
        "type": "MSISDN",
        "address": "265999111111"
      }
    },
    {
      "payoutId": "payout-002",
      "amount": 3000,
      "correspondent": "AIRTEL_TZ",
      "recipient": {
        "type": "MSISDN",
        "address": "255755222222"
      }
    }
  ]
}
```

**Response (202 Accepted):**
```json
{
  "bulkPayoutId": "bulk-20240220-001",
  "status": "SUBMITTED",
  "payoutCount": 2
}
```

**Check Bulk Payout Status:**
```bash
GET /bulk-payouts/{bulkPayoutId}
```

### 4. Refunds

Reverse a completed transaction by refunding to the original source.

**Endpoint:** `POST /refunds`

**Request:**
```json
{
  "refundId": "refund-20240220-001",
  "originalTransactionId": "payout-20240220-001",
  "amount": 2500,
  "reason": "customer_request",
  "correspondent": "MTN_MW"
}
```

**Response (202 Accepted):**
```json
{
  "refundId": "refund-20240220-001",
  "status": "SUBMITTED"
}
```

**Check Refund Status:**
```bash
GET /refunds/{refundId}
```

## Webhook Integration

PawaPay sends callbacks to your configured webhook URL when a transaction reaches a final state (completed or failed).

### Configuration

1. Go to PawaPay Dashboard → Settings → Webhooks
2. Add your callback URL (must be HTTPS with valid CA certificate)
3. Select which events to receive: deposits, payouts, refunds

### Webhook Payload Examples

**Deposit Callback:**
```json
{
  "depositId": "dep-20240220-001",
  "amount": 5000,
  "correspondent": "MTN_MW",
  "depositor": {
    "type": "MSISDN",
    "address": "265999999999"
  },
  "status": "COMPLETED",
  "timestamp": "2024-02-20T14:30:45Z",
  "metadata": {
    "orderRef": "ORDER-123"
  }
}
```

**Payout Callback (Failed):**
```json
{
  "payoutId": "payout-20240220-001",
  "amount": 2500,
  "correspondent": "MTN_MW",
  "recipient": {
    "type": "MSISDN",
    "address": "265999888888"
  },
  "status": "FAILED",
  "failureCode": "RECIPIENT_NOT_FOUND",
  "failureMessage": "Recipient phone number not on MTN network",
  "timestamp": "2024-02-20T14:35:20Z"
}
```

### Webhook Requirements & Best Practices

**Your endpoint must:**
- Accept HTTP POST requests
- Return HTTP 200 OK to confirm receipt
- Be idempotent (handle duplicate callbacks safely)
- Use HTTPS with trusted CA certificate
- Complete processing within 30 seconds

**Retry behavior:**
- PawaPay retries failed callbacks for 15 minutes
- Use `/resend-callback` endpoint to manually request resend

**Example Node.js handler:**
```javascript
const express = require('express');
const app = express();

app.post('/pawapay-webhook', express.json(), async (req, res) => {
  const { transactionId, status, amount } = req.body;

  try {
    // Log the callback for idempotency check
    const exists = await db.webhookLogs.findOne({ transactionId });
    if (exists) {
      return res.status(200).json({ message: 'Already processed' });
    }

    // Process transaction status update
    await db.transactions.updateOne(
      { id: transactionId },
      { status, updatedAt: new Date() }
    );

    // Log for audit trail
    await db.webhookLogs.create({ transactionId, status, amount });

    res.status(200).json({ received: true });
  } catch (error) {
    console.error('Webhook error:', error);
    res.status(200).json({ received: true }); // Return 200 anyway to prevent retries
  }
});
```

## Common Integration Patterns

### Pattern 1: Simple One-Time Payout

```javascript
const axios = require('axios');

async function sendPayout(phoneNumber, amount, country) {
  const correspondent = getCorrespondent(country); // e.g., 'MTN_MW'

  try {
    const response = await axios.post(
      'https://api.pawapay.io/payouts',
      {
        payoutId: `payout-${Date.now()}`,
        amount,
        correspondent,
        recipient: {
          type: 'MSISDN',
          address: phoneNumber
        }
      },
      {
        headers: {
          'Authorization': `Bearer ${process.env.PAWAPAY_TOKEN}`,
          'Content-Type': 'application/json'
        }
      }
    );

    return { success: true, payoutId: response.data.payoutId };
  } catch (error) {
    console.error('Payout failed:', error.response?.data);
    return { success: false, error: error.message };
  }
}
```

### Pattern 2: Polling for Status (Fallback)

```javascript
async function waitForPayoutCompletion(payoutId, maxRetries = 30) {
  for (let i = 0; i < maxRetries; i++) {
    const response = await axios.get(
      `https://api.pawapay.io/payouts/${payoutId}`,
      {
        headers: {
          'Authorization': `Bearer ${process.env.PAWAPAY_TOKEN}`
        }
      }
    );

    const { status } = response.data;

    if (status === 'COMPLETED' || status === 'FAILED') {
      return response.data;
    }

    // Wait 2 seconds before next check
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  throw new Error('Payout status timeout');
}
```

### Pattern 3: Bulk Salary Distribution

```javascript
async function distributeSalaries(employees) {
  const bulkPayoutId = `bulk-salary-${Date.now()}`;

  const payouts = employees.map(emp => ({
    payoutId: `sal-${emp.id}-${Date.now()}`,
    amount: emp.salary,
    correspondent: getCorrespondent(emp.country),
    recipient: {
      type: 'MSISDN',
      address: emp.phoneNumber
    },
    metadata: {
      employeeId: emp.id,
      payrollPeriod: 'Feb-2024'
    }
  }));

  const response = await axios.post(
    'https://api.pawapay.io/bulk-payouts',
    {
      bulkPayoutId,
      payouts
    },
    {
      headers: {
        'Authorization': `Bearer ${process.env.PAWAPAY_TOKEN}`,
        'Content-Type': 'application/json'
      }
    }
  );

  return response.data;
}
```

### Pattern 4: Handling Webhooks with Database

```javascript
async function handlePayoutCallback(callbackData) {
  const { payoutId, status, failureCode } = callbackData;

  const transaction = await db.transactions.findOne({ id: payoutId });

  if (!transaction) {
    // Log orphaned callback
    console.warn(`Received callback for unknown payout: ${payoutId}`);
    return;
  }

  // Update transaction status
  await db.transactions.updateOne(
    { id: payoutId },
    {
      status,
      failureCode: failureCode || null,
      completedAt: new Date()
    }
  );

  // Trigger side effects (notifications, etc.)
  if (status === 'FAILED') {
    await notifyUser(transaction.userId, `Payout failed: ${failureCode}`);
  } else if (status === 'COMPLETED') {
    await notifyUser(transaction.userId, 'Funds received successfully');
  }
}
```

## Error Handling

### Transaction Failure Codes

| Code | Meaning | Action |
|------|---------|--------|
| `RECIPIENT_NOT_FOUND` | Phone number not on specified network | Verify phone number and correspondent |
| `BALANCE_INSUFFICIENT` | Insufficient funds in PawaPay account | Top up account balance |
| `RECIPIENT_NOT_ALLOWED_TO_RECEIVE` | Recipient wallet limit exceeded | Ask recipient to verify account or wait for limit reset |
| `CORRESPONDENT_TEMPORARILY_UNAVAILABLE` | Mobile operator experiencing outage | Retry after network recovers |
| `INVALID_REQUEST_FORMAT` | Request body malformed | Validate JSON structure |
| `DUPLICATE_TRANSACTION_ID` | Transaction ID already used | Use unique transaction IDs |
| `OTHER_ERROR` | Unspecified error | Check `failureMessage` for details |

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| `200` | Request successful |
| `202` | Request accepted, processing asynchronously |
| `400` | Bad request (validation error) |
| `401` | Unauthorized (invalid/missing token) |
| `403` | Forbidden (account not authorized) |
| `404` | Resource not found |
| `429` | Rate limit exceeded |
| `500` | Server error (retry with exponential backoff) |

### Error Response Example

```json
{
  "error": {
    "code": "RECIPIENT_NOT_FOUND",
    "message": "The specified phone number does not belong to the correspondent network",
    "details": {
      "phoneNumber": "265999888888",
      "correspondent": "MTN_MW"
    }
  }
}
```

### Recommended Error Handling

```javascript
async function robustPayout(payoutConfig, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await axios.post(
        'https://api.pawapay.io/payouts',
        payoutConfig,
        { headers: { 'Authorization': `Bearer ${token}` } }
      );

      return { success: true, payoutId: response.data.payoutId };
    } catch (error) {
      const status = error.response?.status;
      const errorCode = error.response?.data?.error?.code;

      // Don't retry for validation errors
      if (status === 400) {
        console.error('Invalid request:', error.response.data);
        return { success: false, permanent: true };
      }

      // Retry for temporary issues
      if (status === 500 || status === 429 || errorCode === 'CORRESPONDENT_TEMPORARILY_UNAVAILABLE') {
        if (attempt < maxRetries) {
          const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
          await new Promise(r => setTimeout(r, delay));
          continue;
        }
      }

      // Final error
      return { success: false, error: error.message, permanent: false };
    }
  }
}
```

## SDK & Libraries

### Official/Community SDKs

| Language | Repository | Status |
|----------|-----------|--------|
| **Node.js** | [dave-evans-pawapay/developer-guide-node](https://github.com/dave-evans-pawapay/developer-guide-node) | Official example |
| **PHP** | [katorymnd/pawa-pay-integration](https://github.com/katorymnd/pawa-pay-integration) | Community (Composer) |
| **Flutter/Dart** | [pawa_pay_flutter](https://pub.dev/packages/pawa_pay_flutter) | Community |
| **Go** | [Uchencho/pawapay](https://github.com/Uchencho/pawapay) | Community |

### PHP SDK Example

```bash
composer require katorymnd/pawa-pay-integration
```

```php
<?php
use PawaPay\Integration\PayaPay;

$payaPay = new PayaPay([
    'apiToken' => 'YOUR_API_TOKEN',
    'environment' => 'sandbox' // or 'production'
]);

$payout = $payaPay->payout([
    'payoutId' => 'payout-123',
    'amount' => 5000,
    'correspondent' => 'MTN_MW',
    'recipient' => [
        'type' => 'MSISDN',
        'address' => '265999999999'
    ]
]);

echo $payout['status']; // SUBMITTED
?>
```

## Important Notes & Gotchas

1. **Asynchronous Processing**: All financial operations (deposits, payouts, refunds) are asynchronous. Use webhooks or polling, not immediate response status.

2. **Phone Number Format**: Use international format with country code (e.g., `265999999999` for Malawi). Include the `+` prefix for clarity but it's optional in the API.

3. **Correspondent Code Matters**: Each country has specific correspondent codes (e.g., `MTN_MW`, `AIRTEL_TZ`). Using wrong code causes `RECIPIENT_NOT_FOUND` errors. Verify the correspondent list for target country.

4. **Idempotency**: Always use unique transaction IDs. Submitting the same transaction ID twice results in an error. Consider including timestamp: `payout-${userId}-${Date.now()}`.

5. **Webhook Idempotency**: Your webhook handler MUST be idempotent as PawaPay may resend callbacks. Check if transaction was already processed before updating state.

6. **Sufficient Balance Required**: Ensure your PawaPay account has sufficient balance for payouts. Account top-ups happen through separate dashboard or API (contact sales for details).

7. **Network Operator Outages**: Mobile operators may temporarily go offline. Implement retry logic with exponential backoff. Some failures are temporary and will succeed on retry.

8. **Customer PIN Required**: For deposits, the customer must authorize by entering their mobile money PIN. If they don't respond within ~5 minutes, deposit times out and fails.

9. **Callback URL HTTPS Requirement**: Webhook callback URLs must use HTTPS with a valid, trusted CA certificate. Self-signed certificates are rejected.

10. **Rate Limiting**: PawaPay implements rate limiting. Implement backoff strategy. Receiving 429 status means you've exceeded limits; back off exponentially.

11. **Timeout Configuration**: Keep reasonable timeouts (5-30 seconds) when polling status. Don't poll indefinitely. Set a maximum retry count and flag for manual investigation.

12. **Money Is Real**: Test thoroughly in sandbox before production. Every payout in production moves real money from your account to recipient's mobile wallet.

13. **Metadata Not Searchable**: Metadata fields are for your records only. You cannot query or filter by metadata; use it for internal reconciliation and logging.

14. **Partial Bulk Failures**: Bulk payouts may partially succeed. One payout failure doesn't stop others. Check individual payout statuses, not just bulk status.

15. **Timezone Handling**: Timestamps in callbacks use ISO 8601 format (UTC). Ensure your system handles timezone conversions correctly for reconciliation.

## Supported Countries & Operators

PawaPay supports mobile money across 19+ countries. Key operators include:

| Country | Operators |
|---------|-----------|
| **Malawi** | MTN Mobile Money |
| **Tanzania** | Airtel Money, Vodacom M-Pesa |
| **Uganda** | MTN Mobile Money, Airtel Money |
| **Rwanda** | Airtel Money, MTN Mobile Money |
| **Zambia** | Airtel Money, MTN Zambia |
| **Benin** | MTN Mobile Money |
| **Congo (DRC)** | Airtel Money |
| **Gabon** | Airtel Money |

For complete current list, check the [PawaPay Payment Providers page](https://www.pawapay.io/payment-providers).

## Testing in Sandbox

PawaPay provides a sandbox environment for safe testing:

- **Dashboard**: https://dashboard.sandbox.pawapay.io
- **API Base URL**: https://api.sandbox.pawapay.io
- **Funds**: Test funds provided; no real money involved
- **Isolation**: Completely separate from production

### Test Phone Numbers

Sandbox provides specific phone numbers for different test scenarios. Check your sandbox dashboard for current test numbers and expected outcomes.

## Useful Links

- **Official Documentation**: https://docs.pawapay.io/
- **PawaPay Website**: https://www.pawapay.io/
- **API Reference**: https://docs.pawapay.io/v2/api-reference/
- **Postman Collection**: https://www.postman.com/pawapay-4106/pawapay/
- **Sandbox Dashboard**: https://dashboard.sandbox.pawapay.io/
- **Node.js Example**: https://github.com/dave-evans-pawapay/developer-guide-node
- **Signature Implementation**: https://github.com/pawaPay/signatures-node-example
- **PHP SDK**: https://packagist.org/packages/katorymnd/pawa-pay-integration
- **Status Page**: https://pawapay.zendesk.com/hc/
- **Support**: Contact your PawaPay account manager or support@pawapay.io

## Summary Table

| Aspect | Details |
|--------|---------|
| **API Type** | REST (asynchronous) |
| **Authentication** | Bearer token + optional RFC-9421 signing |
| **Base URL (Sandbox)** | https://api.sandbox.pawapay.io |
| **Base URL (Production)** | https://api.pawapay.io |
| **Core Operations** | Deposits, Payouts, Bulk Payouts, Refunds |
| **Callback Support** | Yes, with retry mechanism |
| **Coverage** | 19+ African countries |
| **Operators** | 15+ mobile money operators |
| **SDKs** | Node.js, PHP, Flutter/Dart, Go |
| **Rate Limiting** | Yes (429 status code) |
| **SLA** | Varies by operator, typically minutes to hours |

---

**Document Version:** 1.0
**Last Updated:** February 2024
**Status:** Production-Ready
