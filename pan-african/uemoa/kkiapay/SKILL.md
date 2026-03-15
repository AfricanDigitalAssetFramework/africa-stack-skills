---
name: KKiaPay
description: Francophone West African payment aggregator supporting mobile money (MTN, Moov, Orange Money, Wave) and card payments in FCFA currencies
triggers:
  - KKiaPay
  - Benin payments
  - Francophone payment gateway
  - FCFA mobile money
  - West Africa checkout
  - Togo payments
  - Senegal payments
  - Côte d'Ivoire payments
  - Wave money transfer
  - MTN Mobile Money payment
  - Moov Money payment
---

# KKiaPay Payment Gateway

**KKiaPay** is a payment aggregator platform enabling businesses in Francophone West Africa (Benin, Côte d'Ivoire, Togo, Senegal, Niger) to accept payments via mobile money and card payments in West African CFA francs (FCFA/XOF). The platform supports multiple payment methods including MTN Mobile Money, Moov Money, Orange Money, Free Money, Celtiis Cash, Wave, Visa, and Mastercard through a unified API and widget-based integration.

## When to Use This Skill

- Building payment checkout experiences for West African markets
- Integrating mobile money (MTN, Moov, Orange Money) payment methods
- Processing FCFA (XOF) currency transactions
- Accepting card payments (Visa/Mastercard) alongside mobile money
- Implementing Wave money transfer functionality
- Setting up real-time transaction webhooks and status callbacks
- Developing multi-country payment solutions for FCFA zone countries
- Creating merchant-facing transaction dashboards
- Handling payment refunds and transaction verification

## Authentication & API Keys

KKiaPay uses three types of API keys, all obtained from the KKiaPay Dashboard Developer section:

### Key Types

1. **Public Key** - Used in client-side integrations (JavaScript widget)
2. **Private Key** - Used for server-side transaction verification and admin operations
3. **Secret Key** - Used for webhook signature verification and sensitive operations

### Environment Setup

```javascript
// JavaScript SDK - Browser/Frontend
const k = Kkiapay({
  publicKey: "YOUR_PUBLIC_KEY",
  sandbox: true  // Set to false for production
});

// Node.js Admin SDK - Server-side
const Kkiapay = require('@kkiapay-org/nodejs-sdk');
const k = Kkiapay({
  publickey: "YOUR_PUBLIC_KEY",
  privatekey: "YOUR_PRIVATE_KEY",
  secretkey: "YOUR_SECRET_KEY",
  sandbox: true
});

// PHP SDK - Server-side
$k = new Kkiapay([
    'public_key' => 'YOUR_PUBLIC_KEY',
    'private_key' => 'YOUR_PRIVATE_KEY',
    'secret' => 'YOUR_SECRET_KEY',
    'sandbox' => true
]);
```

**Security Notes:**
- Never expose private or secret keys in client-side code
- Store keys in environment variables, not in source code
- Rotate keys periodically through the KKiaPay dashboard
- Always use HTTPS for API communications

## REST API Base URL

While KKiaPay is primarily used via their JavaScript widget and official SDKs, the underlying REST API can be called directly:

| Environment | Base URL |
|-------------|----------|
| **Production** | `https://api.kkiapay.me/v1` |
| **Sandbox** | `https://sandbox-api.kkiapay.me/v1` |

**Authentication for direct REST calls:** Include your private key in the `X-Private-Key` header. Example:

```bash
curl -X GET https://api.kkiapay.me/v1/transactions/txn_abc123456789 \
  -H "X-Private-Key: your_private_key"
```

## Transaction Status Values

When verifying a transaction (via SDK or REST), the `status` field returns one of:

| Status | Meaning |
|--------|---------|
| `COMPLETE` | Payment successfully completed |
| `FAILED` | Payment failed (see `failureCode` for reason) |
| `PENDING` | Payment initiated but not yet completed — poll again |
| `REFUNDED` | Payment was successfully refunded |
| `CANCELLED` | Transaction was cancelled before completion |

Common `failureCode` values: `insufficient_fund`, `wrong_pin`, `timeout`, `cancelled_by_user`, `network_error`.

## Core API Reference

### Transaction Endpoints

#### Initialize a Payment Request (Widget-Based)

The primary integration method uses KKiaPay's JavaScript widget, which handles the payment UI:

```javascript
// Browser-side widget integration
const k = Kkiapay({
  publicKey: "pk_live_xxxxxxx",
  sandbox: false
});

// Trigger payment widget
const promise = k.openKkiapayWidget({
  amount: 5000,              // Amount in FCFA (XOF)
  reason: "Product Purchase", // Transaction description
  key: "pk_live_xxxxxxx",    // Public key
  callback: "https://yoursite.com/payment-callback",
  metadata: {
    orderId: "12345",
    userId: "user789"
  }
});

promise
  .then((response) => {
    console.log("Payment successful:", response);
    // response: { transactionId: "txn_xxx", account: "data", ... }
  })
  .catch((error) => {
    console.log("Payment failed:", error);
  });
```

#### Direct Debit Request (Mobile Money)

```javascript
// JavaScript SDK - Mobile money debit
k.debit(
  "22967434270",  // Phone number (include country code)
  5000,           // Amount in FCFA
  {
    reason: "Product Purchase",
    firstname: "John",
    lastname: "Doe",
    email: "john@example.com",
    callback: "https://yoursite.com/callback"
  }
)
.then((response) => {
  console.log("Transaction ID:", response.transactionId);
  // Webhook will also be sent to your configured URL
})
.catch((error) => {
  console.log("Debit request failed:", error.failureCode);
});
```

#### Verify Transaction (Server-Side)

```javascript
// Node.js - Verify transaction status
const k = Kkiapay({
  privatekey: "sk_live_xxxxxxx",
  publickey: "pk_live_xxxxxxx",
  secretkey: "secret_xxxxxxx",
  sandbox: false
});

k.verify("txn_abc123456789")
  .then((response) => {
    if (response.isPaymentSucces) {
      console.log("Transaction verified - Payment successful");
      console.log("Amount:", response.amount);
      console.log("Method:", response.method); // MOBILE_MONEY, CARD, WALLET
    } else {
      console.log("Payment failed:", response.failureCode);
    }
  })
  .catch((error) => {
    console.error("Verification error:", error);
  });
```

#### Refund Transaction (Mobile Money Only)

```javascript
// Node.js - Refund a successful transaction
k.refund("txn_abc123456789", {
  reason: "Customer requested refund"
})
  .then((response) => {
    console.log("Refund processed:", response);
  })
  .catch((error) => {
    console.log("Refund failed:", error);
  });
```

### Response Format

**Successful Transaction Response:**
```json
{
  "transactionId": "txn_abc123456789",
  "isPaymentSucces": true,
  "method": "MOBILE_MONEY",
  "account": "+22967434270",
  "amount": 5000,
  "fees": 25,
  "label": "Product Purchase",
  "partnerId": "partner_123",
  "performedAt": "2024-02-24T10:30:45Z",
  "stateData": {
    "orderId": "12345"
  }
}
```

**Failed Transaction Response:**
```json
{
  "transactionId": "txn_def456789012",
  "isPaymentSucces": false,
  "method": "MOBILE_MONEY",
  "account": "+22967434270",
  "amount": 5000,
  "failureCode": "insufficient_fund",
  "failureMessage": "Account has insufficient balance for this transaction",
  "label": "Product Purchase",
  "performedAt": "2024-02-24T10:35:12Z"
}
```

## Webhooks

### Webhook Configuration

Configure your webhook endpoint in the KKiaPay Dashboard under **Developers > Webhooks**. KKiaPay will POST transaction updates to your URL.

### Webhook Security

All webhook requests include a signature header for verification:

```javascript
// Node.js - Verify webhook signature
const crypto = require('crypto');
const express = require('express');

app.post('/webhook/kkiapay', (req, res) => {
  const signature = req.headers['x-kkiapay-secret'];
  const secretKey = process.env.KKIAPAY_SECRET_KEY;

  // Create expected signature
  const payload = JSON.stringify(req.body);
  const expectedSignature = crypto
    .createHmac('sha256', secretKey)
    .update(payload)
    .digest('hex');

  // Verify signature
  if (signature !== expectedSignature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook
  const transaction = req.body;
  if (transaction.isPaymentSucces) {
    // Fulfill order
    console.log('Payment confirmed:', transaction.transactionId);
  } else {
    // Log payment failure
    console.log('Payment failed:', transaction.failureCode);
  }

  res.status(200).json({ success: true });
});
```

### Webhook Payload Events

KKiaPay sends webhooks for the following events:

**transaction.success** - Payment completed successfully
```json
{
  "event": "transaction.success",
  "transactionId": "txn_abc123456789",
  "isPaymentSucces": true,
  "method": "MOBILE_MONEY",
  "account": "+22967434270",
  "amount": 5000,
  "fees": 25,
  "label": "Product Purchase",
  "partnerId": "partner_123",
  "performedAt": "2024-02-24T10:30:45Z",
  "stateData": {}
}
```

**transaction.failed** - Payment failed
```json
{
  "event": "transaction.failed",
  "transactionId": "txn_def456789012",
  "isPaymentSucces": false,
  "method": "MOBILE_MONEY",
  "failureCode": "insufficient_fund",
  "failureMessage": "Account has insufficient balance",
  "performedAt": "2024-02-24T10:35:12Z"
}
```

**transaction.timeout** - Transaction exceeded 90-second timeout
```json
{
  "event": "transaction.timeout",
  "transactionId": "txn_ghi789012345",
  "failureCode": "timeout",
  "performedAt": "2024-02-24T10:40:00Z"
}
```

### Webhook Best Practices

- Always verify the `x-kkiapay-secret` signature
- Implement idempotency handling (transactions may be retried)
- Store webhook payloads in your database for audit trails
- Return HTTP 200 immediately; process async
- Set a timeout on webhook processing (avoid long-running operations)
- Implement retry logic for failed webhook deliveries

## Common Integration Patterns

### Pattern 1: Web Checkout with JavaScript Widget

```html
<!-- HTML -->
<button id="paymentBtn">Complete Payment</button>

<script src="https://cdn.jsdelivr.net/npm/kkiapay/index.js"></script>
<script>
  const k = Kkiapay({
    publicKey: "pk_live_xxxxxxx",
    sandbox: false
  });

  document.getElementById('paymentBtn').addEventListener('click', () => {
    const promise = k.openKkiapayWidget({
      amount: 10000,
      reason: "Order #12345",
      key: "pk_live_xxxxxxx",
      callback: "https://yoursite.com/payment-success",
      metadata: {
        orderId: "12345",
        customerId: "cust_789"
      }
    });

    promise
      .then((response) => {
        // Update UI with payment confirmed
        // Verify server-side before fulfilling order
        document.getElementById('paymentBtn').disabled = true;
      })
      .catch((error) => {
        alert('Payment failed: ' + error.failureCode);
      });
  });
</script>
```

### Pattern 2: Mobile Money Direct Debit (SDK)

```javascript
// Node.js server-side
const Kkiapay = require('@kkiapay-org/nodejs-sdk');

const k = Kkiapay({
  publickey: process.env.KKIAPAY_PUBLIC_KEY,
  privatekey: process.env.KKIAPAY_PRIVATE_KEY,
  secretkey: process.env.KKIAPAY_SECRET_KEY,
  sandbox: false
});

async function requestMobileMoneyPayment(phoneNumber, amount, orderId) {
  try {
    const response = await k.debit(phoneNumber, amount, {
      reason: `Order ${orderId}`,
      firstname: "Customer",
      callback: `https://yoursite.com/webhook/kkiapay`
    });

    return {
      transactionId: response.transactionId,
      status: 'pending' // Wait for webhook confirmation
    };
  } catch (error) {
    console.error('Debit request failed:', error);
    throw error;
  }
}
```

### Pattern 3: Card Payment via Widget

```javascript
// Card payments are handled through the same widget interface
const k = Kkiapay({
  publicKey: "pk_live_xxxxxxx",
  sandbox: false
});

// Widget automatically shows card option alongside mobile money
k.openKkiapayWidget({
  amount: 25000,
  reason: "Premium Subscription - Monthly",
  key: "pk_live_xxxxxxx",
  callback: "https://yoursite.com/subscription-callback"
});

// Widget will accept Visa, Mastercard, and other supported cards
// User selects payment method in widget UI
```

### Pattern 4: Transaction Verification Workflow

```javascript
// Complete server-side verification workflow
const express = require('express');
const app = express();

// Webhook endpoint - receives payment notifications
app.post('/webhook/kkiapay', async (req, res) => {
  const { transactionId, isPaymentSucces } = req.body;

  // Verify signature first
  const signature = req.headers['x-kkiapay-secret'];
  if (!verifySignature(req.body, signature)) {
    return res.status(401).send('Invalid signature');
  }

  // For webhook success notification
  if (isPaymentSucces) {
    // Update order status
    await updateOrderStatus(transactionId, 'PAID');
    // Send confirmation email
    // Trigger fulfillment
  } else {
    // Mark order as payment failed
    await updateOrderStatus(transactionId, 'PAYMENT_FAILED');
  }

  res.status(200).json({ received: true });
});

// Explicit transaction verification endpoint
app.get('/api/verify-transaction/:id', async (req, res) => {
  const k = Kkiapay({
    privatekey: process.env.KKIAPAY_PRIVATE_KEY,
    sandbox: false
  });

  try {
    const transaction = await k.verify(req.params.id);
    res.json(transaction);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

## Error Handling

### Common Failure Codes

| Code | Meaning | Action |
|------|---------|--------|
| `insufficient_fund` | Customer account balance is too low | Prompt user to add funds |
| `invalid_phone` | Phone number format is invalid | Validate phone number format |
| `operator_error` | Mobile operator service is unavailable | Retry after delay |
| `timeout` | Customer took >90 seconds to confirm | Request new payment attempt |
| `user_rejected` | Customer declined the payment prompt | Allow retry |
| `processing_error` | Temporary processing error | Retry transaction |
| `network_error` | Network connectivity issue | Check webhook delivery |
| `invalid_amount` | Amount is below minimum or above maximum | Adjust amount |

### Error Handling Example

```javascript
async function handlePaymentError(failureCode) {
  const errorMessages = {
    'insufficient_fund': 'Your account balance is too low. Please add funds.',
    'invalid_phone': 'Please provide a valid phone number.',
    'operator_error': 'Mobile operator is temporarily unavailable. Please try again.',
    'timeout': 'Payment request timed out. Please try again.',
    'user_rejected': 'You declined the payment. Please try again if needed.',
    'processing_error': 'A temporary error occurred. Please try again.',
    'invalid_amount': 'The payment amount is invalid. Please check and try again.'
  };

  const message = errorMessages[failureCode] || 'Payment failed. Please try again.';

  // Log for debugging
  console.error(`Payment failed with code: ${failureCode}`);

  // Return user-friendly message
  return {
    success: false,
    message: message,
    retry: true,
    code: failureCode
  };
}
```

### HTTP Status Codes

- **200 OK** - Request successful
- **400 Bad Request** - Invalid request format or parameters
- **401 Unauthorized** - Invalid or missing API credentials
- **422 Unprocessable Entity** - Validation error (invalid phone, amount, etc.)
- **500 Server Error** - KKiaPay server error (retry with backoff)

## Important Notes & Gotchas

1. **Phone Number Format Required** - Always include country code in phone numbers (e.g., `+229`, `+221`, `+225`, `+228` for Benin, Senegal, Côte d'Ivoire, Togo respectively). Without it, requests will fail.

2. **Widget vs. Direct API** - The JavaScript SDK primarily uses the widget-based payment flow. Direct REST API calls are typically handled through the Node.js/PHP admin SDKs for server-side verification, not for initiating payments.

3. **Webhook Signature Verification is Essential** - Always verify the `x-kkiapay-secret` header on webhook calls. Never trust webhook data without signature validation, as transactions can be fraudulently injected.

4. **90-Second Payment Timeout** - Customers have exactly 90 seconds to confirm the payment prompt on their phone. After this, the transaction times out and you'll receive a `transaction.timeout` webhook event. Plan your UI accordingly.

5. **Sandbox Testing Required** - Always test integrations in sandbox mode (`sandbox: true`) with test phone numbers provided by KKiaPay before going live. Production data should never be tested against.

6. **Fees Are Deducted from Merchant Account** - The `fees` field in responses indicates the commission KKiaPay deducts. Your merchant account is credited with `amount - fees`. This is automatic and non-negotiable at API level.

7. **Transaction IDs are Unique Per Attempt** - Each payment attempt generates a new transaction ID. If a customer retries a failed payment, it's a completely different transaction. Don't create duplicates in your database.

8. **Mobile Money Refunds Have Limitations** - Refunds can only be processed for successful mobile money transactions. Card transactions (Visa/Mastercard) cannot be refunded through the API—they must be refunded through your card processor or chargeback system.

9. **Multiple Webhook Delivery Attempts** - KKiaPay may retry webhook delivery if your endpoint doesn't return HTTP 200. Implement idempotency to handle duplicate webhook events gracefully (check `transactionId` in your database before processing).

10. **FCFA Currency Only** - All amounts are in West African CFA francs (XOF/FCFA). Currency conversion must happen client-side if your application uses a different currency. No automatic currency conversion is provided.

11. **Account Verification Required** - Your KKiaPay merchant account must be fully verified (ID, business documents) before processing live transactions. This takes up to 24 hours from submission.

12. **Rate Limiting and Throttling** - KKiaPay may rate-limit your API requests during high-traffic periods. Implement exponential backoff retry logic. Check response headers for rate-limit information.

## Useful Links

- **Official KKiaPay Website**: https://kkiapay.me/
- **API Documentation Portal**: https://docs.kkiapay.me/
- **JavaScript SDK Repository**: https://github.com/kkiapay/js-sdk
- **Node.js Admin SDK**: https://github.com/kkiapay/nodejs-sdk
- **PHP SDK**: https://github.com/kkiapay/php-sdk
- **Android SDK**: https://github.com/kkiapay/android-sdk
- **Flutter SDK**: https://github.com/kkiapay/kkiapay-flutter-sdk
- **React Hook**: https://github.com/kkiapay/kkiapay-react
- **Sandbox Test Guide**: https://docs.kkiapay.me/v1/en-1.0.0/compte/kkiapay-sandbox-guide-de-test
- **Webhook Documentation**: https://docs.kkiapay.me/v1/en-1.0.0/tableau-de-bord/webhook
- **Transactions Documentation**: https://docs.kkiapay.me/v1/en-1.0.0/paiements/transactions
- **NPM Package** (@kkiapay-org/nodejs-sdk): https://www.npmjs.com/package/@kkiapay-org/nodejs-sdk
- **Support Email**: support@kkiapay.me
- **Status Page**: https://transactions-db.kkiapay.me/
