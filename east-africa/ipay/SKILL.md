---
name: iPay Kenya Payment Gateway
description: Kenyan payment gateway with support for M-Pesa, Airtel, EazzyPay, and cards using HMAC-based authentication. Includes C2B, B2B, and B2C APIs with STK push, USSD, and callback integration.
triggers:
  - iPay
  - iPay Kenya
  - iPay Africa
  - Kenyan payment gateway
  - KES mobile money payments
  - M-Pesa payment integration
---

# iPay Kenya Payment Gateway

iPay is a leading East African payment processing platform providing comprehensive payment solutions for businesses in Kenya, Uganda, Tanzania, Rwanda, and the Democratic Republic of the Congo. The platform enables customer-to-business (C2B), business-to-business (B2B), and business-to-customer (B2C) payments through multiple channels including mobile money (M-Pesa, Airtel Money, EazzyPay) and card payments.

The iPay API uses a secure two-step transaction flow with HMAC-SHA256 authentication for all requests, supporting modern payment initiation methods like STK push and USSD alongside traditional form-based integration.

## When to Use This Skill

Use iPay integration when you need to:

- **Collect payments from customers in Kenya** with support for M-Pesa, Airtel Money, EazzyPay, or card payments
- **Send disbursements** to customers' mobile wallets or bank accounts (B2C)
- **Pay other businesses** directly to their M-Pesa PayBill/Till (B2B)
- **Trigger STK push prompts** on customers' phones for frictionless M-Pesa payments (SIM Toolkit)
- **Enable USSD-based payments** for feature phone users and low-connectivity environments
- **Process recurring/subscription billing** with the Recurring Billing API
- **Verify payment status** after transactions complete via webhooks or manual search
- **Refund customer payments** after transaction completion

iPay is ideal for East African startups, e-commerce platforms, service providers, and fintech applications requiring local payment processing with minimal friction.

## Authentication & HMAC Signatures

All iPay API requests require HMAC-SHA256 authentication. The merchant must generate a cryptographic signature combining request parameters with their secret key to prove request authenticity.

### API Credentials

Obtain from your iPay merchant dashboard:
- **Live Base URL**: `https://apis.ipayafrica.com`
- **Staging Base URL**: `https://apis.staging.ipayafrica.com`
- **Merchant ID (vid)**: Your unique merchant identifier
- **Secret Key**: Private key for HMAC generation (keep secure)

### HMAC Generation Process

The hash is computed from a **concatenated datastring** of parameters in strict order, using HMAC-SHA256 with your secret key. Parameter order is critical and varies by endpoint.

> ⚠️ **HMAC parameter order varies by flow.** The C2B collection and B2C disbursement APIs use different parameter concatenation orders. Using the C2B hash logic for a B2C request (or vice versa) will result in all requests being rejected with a signature error. Always refer to the endpoint-specific documentation at [dev.ipayafrica.com](https://dev.ipayafrica.com) for the exact field order for each API type.

> ⚠️ **B2C disbursements have separate authentication.** The B2C payout API (sending money to customers' mobile wallets) uses a different API key and may require separate merchant activation. Contact iPay support (support@ipayafrica.com) to enable B2C on your account — it is not automatically enabled with C2B access.

> ℹ️ **C2B integration: API vs form redirect.** iPay supports two C2B integration models: (1) **API-initiated** — your server calls the iPay API to initiate STK push or USSD; (2) **Form redirect** — you POST an HTML form to the iPay checkout page and the customer completes payment there. The form redirect model does not require HMAC on the frontend, but your backend callback verification must still validate the iPay response signature. Choose API-initiated for embedded UX; form redirect for quickest integration.

#### C2B Transaction Registration Example (PHP)

```php
<?php
// C2B transaction registration parameters
$amount = "1000.00";
$phone = "254712345678";
$reference = "ORDER-12345";
$redirect_url = "https://yoursite.com/callback";
$secret = "your_secret_key_from_dashboard";

// Construct datastring - ORDER MATTERS!
$datastring = $amount . $phone . $reference;

// Generate HMAC-SHA256 hash
$hash = hash_hmac('sha256', $datastring, $secret, false);

// Request payload
$payload = [
    "amount" => $amount,
    "phone" => $phone,
    "reference" => $reference,
    "redirect_url" => $redirect_url,
    "hash" => $hash
];

echo json_encode($payload);
?>
```

#### M-Pesa STK Push Example (Node.js)

```javascript
const crypto = require('crypto');

const amount = '1000.00';
const phone = '254712345678';
const vid = 'your_merchant_id';
const account = 'ACCT-001';
const secret = 'your_secret_key_from_dashboard';

// STK push datastring - different order
const datastring = phone + vid + amount + account;

// Generate HMAC-SHA256
const hash = crypto
  .createHmac('sha256', secret)
  .update(datastring)
  .digest('hex');

const payload = {
  phone,
  vid,
  amount,
  account,
  hash
};

console.log(JSON.stringify(payload, null, 2));
```

#### Key Points

- **No spaces** between concatenated values
- **Parameter order** is endpoint-specific and must not vary
- **Hash is hexadecimal** lowercase string
- **Secret key** should never be exposed in client-side code or URLs
- Use **HTTPS only** for all production requests

## Core API Reference

### Base URLs

```
Live:    https://apis.ipayafrica.com/payments/v2
Staging: https://apis.staging.ipayafrica.com/payments/v2
```

### C2B Transaction Registration

Register a transaction before processing payment. Returns a session ID (sid) used in subsequent transact calls.

**Endpoint**: `POST /c2b/register`

**Request Example**:
```json
{
  "amount": "1000.00",
  "phone": "254712345678",
  "reference": "ORDER-12345",
  "redirect_url": "https://yoursite.com/callback",
  "comment": "Payment for order 12345",
  "hash": "abc123def456..."
}
```

**Response Example**:
```json
{
  "status": 200,
  "sid": "xf8a2b1c3d4e5f6g7h8i9j0k",
  "reference": "ORDER-12345"
}
```

### Mobile Money Transact (M-Pesa, Airtel, EazzyPay)

Process payment after customer authorization through their mobile money app. Can be triggered by customer action or via STK push.

**Endpoint**: `POST /transact`

**Request Example**:
```json
{
  "amount": "1000.00",
  "phone": "254712345678",
  "sid": "xf8a2b1c3d4e5f6g7h8i9j0k",
  "channel": "mpesa",
  "hash": "abc123def456..."
}
```

**Response Example**:
```json
{
  "status": "pending",
  "transaction_id": "TXN-987654321",
  "message": "Transaction initiated. Awaiting payment confirmation.",
  "amount": "1000.00"
}
```

### M-Pesa STK Push

Trigger an automatic STK (SIM Toolkit) prompt on the customer's phone to enter their M-Pesa PIN. No customer app interaction required.

**Endpoint**: `POST /transact/push/mpesa`

**Request Example**:
```json
{
  "phone": "254712345678",
  "vid": "your_merchant_id",
  "amount": "1000.00",
  "account": "ACCT-001",
  "hash": "abc123def456..."
}
```

**Response Example**:
```json
{
  "status": "pending",
  "transaction_id": "TXN-654321987",
  "message": "STK prompt sent to customer's phone"
}
```

**Note**: Datastring for STK hash is `phone + vid + amount + account` (different from C2B).

### Card Payment Transact

Process credit/debit card payments via the transaction flow.

**Endpoint**: `POST /transact`

**Request Example**:
```json
{
  "amount": "1000.00",
  "card_number": "4111111111111111",
  "card_holder": "John Doe",
  "exp_month": "12",
  "exp_year": "2025",
  "cvv": "123",
  "sid": "xf8a2b1c3d4e5f6g7h8i9j0k",
  "channel": "card",
  "hash": "abc123def456..."
}
```

**Response Example**:
```json
{
  "status": "pending",
  "transaction_id": "TXN-111222333",
  "message": "Card transaction processing",
  "amount": "1000.00"
}
```

### Transaction Search & Verification

Query transaction status by reference or transaction ID.

**Endpoint**: `POST /transaction/search`

**Request Example**:
```json
{
  "reference": "ORDER-12345",
  "transaction_id": "TXN-987654321",
  "hash": "abc123def456..."
}
```

**Response Example**:
```json
{
  "status": "success",
  "transaction_id": "TXN-987654321",
  "reference": "ORDER-12345",
  "amount": "1000.00",
  "payment_status": "paid",
  "paid_amount": "1000.00",
  "channel": "mpesa",
  "paid_date": "2026-02-24 10:30:45"
}
```

### Transaction Refund

Reverse a completed transaction. Refunded amount returns to customer's payment method.

**Endpoint**: `POST /transaction/refund`

**Request Example**:
```json
{
  "transaction_id": "TXN-987654321",
  "reference": "ORDER-12345",
  "amount": "1000.00",
  "hash": "abc123def456..."
}
```

**Response Example**:
```json
{
  "status": "success",
  "refund_id": "REFUND-123456789",
  "transaction_id": "TXN-987654321",
  "amount": "1000.00",
  "message": "Refund processed successfully"
}
```

## Callbacks & Webhooks

iPay sends payment status notifications to your callback URL as HTTP POST requests. Validate the webhook signature using the included `hash` parameter.

### Callback Request Format

```json
{
  "transaction_id": "TXN-987654321",
  "reference": "ORDER-12345",
  "amount": "1000.00",
  "phone": "254712345678",
  "channel": "mpesa",
  "status": "success",
  "payment_status": "paid",
  "paid_amount": "1000.00",
  "paid_date": "2026-02-24 10:30:45",
  "psp_response_code": "00",
  "psp_response": "Transaction successful",
  "hash": "abc123def456..."
}
```

### Validating Callback Signatures (PHP)

```php
<?php
// Receive webhook POST
$callback = json_decode(file_get_contents('php://input'), true);

$secret = "your_secret_key_from_dashboard";

// Construct validation string - order matters!
$datastring = $callback['amount'] . $callback['phone'] . $callback['reference'];

// Compute expected hash
$expected_hash = hash_hmac('sha256', $datastring, $secret, false);

// Verify signature
if ($callback['hash'] === $expected_hash) {
    // Signature valid - process payment
    echo json_encode(['status' => 'ok', 'message' => 'Payment processed']);
    http_response_code(200);
} else {
    // Signature mismatch - reject callback
    echo json_encode(['status' => 'error', 'message' => 'Invalid signature']);
    http_response_code(403);
}
?>
```

### Callback Status Values

- **success**: Payment completed and confirmed by mobile network or card processor
- **pending**: Payment initiated but awaiting customer authorization or network confirmation
- **failed**: Payment declined by customer, network, or processor
- **cancelled**: Customer aborted the transaction
- **invalid**: Transaction rejected due to validation errors

### Best Practices

1. **Always validate signature** before processing callback
2. **Return HTTP 200** within 5 seconds of receiving callback
3. **Implement idempotency**: Process callback only once per `transaction_id`
4. **Verify amount and reference** match your original request
5. **Log all callbacks** for audit and debugging
6. **Handle duplicate callbacks** (iPay may retry if no 200 response received)

## Common Integration Patterns

### Pattern 1: E-Commerce Checkout (Form Redirect)

Traditional web flow redirecting to iPay payment page:

1. Customer completes product selection and initiates checkout
2. Backend calls `POST /c2b/register` to create transaction, receives `sid`
3. Backend redirects browser to iPay payment form with `sid` parameter
4. Customer selects payment method (M-Pesa, Airtel, Card) and enters credentials
5. iPay authenticates payment and redirects to your `redirect_url`
6. Callback webhook confirms payment status to your backend
7. Backend updates order status and fulfills order

```javascript
// Backend: Create transaction
app.post('/api/checkout', async (req, res) => {
  const { amount, phone, orderId } = req.body;

  // Register transaction with iPay
  const response = await fetch('https://apis.ipayafrica.com/payments/v2/c2b/register', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      amount,
      phone,
      reference: orderId,
      redirect_url: 'https://yoursite.com/payment-complete',
      hash: generateHash(amount, phone, orderId)
    })
  });

  const { sid } = await response.json();

  // Redirect to iPay payment page
  res.json({
    payment_url: `https://apis.ipayafrica.com/payment/${sid}`
  });
});

// Receive callback
app.post('/api/callback', (req, res) => {
  const callback = req.body;

  // Validate signature
  if (!validateSignature(callback)) {
    return res.status(403).json({ error: 'Invalid signature' });
  }

  // Update order status
  if (callback.status === 'success') {
    updateOrderStatus(callback.reference, 'PAID');
  }

  res.status(200).json({ status: 'ok' });
});
```

### Pattern 2: STK Push for Mobile Apps & Progressive Web Apps

Prompt M-Pesa PIN entry without requiring app launch:

1. User enters amount in your app
2. App calls `POST /transact/push/mpesa` with user's phone number
3. Customer's phone displays STK prompt ("Enter your M-Pesa PIN")
4. Customer enters PIN directly in prompt
5. Payment processes in background
6. App polls transaction status or receives webhook notification
7. Transaction confirmed and order fulfilled

```javascript
// Mobile/PWA: Initiate STK push
async function initiateSTKPush(amount, phone, orderId) {
  const hash = generateSTKHash(phone, merchantId, amount, account);

  const response = await fetch(
    'https://apis.ipayafrica.com/payments/v2/transact/push/mpesa',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        phone,
        vid: merchantId,
        amount,
        account: orderId,
        hash
      })
    }
  );

  const { transaction_id, status } = await response.json();

  // Poll for completion (check every 3 seconds)
  pollTransactionStatus(transaction_id, orderId);
}

// Poll transaction status
async function pollTransactionStatus(transactionId, orderId) {
  let attempts = 0;
  const maxAttempts = 20; // ~60 seconds timeout

  const pollInterval = setInterval(async () => {
    const { payment_status } = await searchTransaction(transactionId);

    if (payment_status === 'paid') {
      clearInterval(pollInterval);
      completeOrder(orderId);
      showSuccessMessage('Payment successful!');
    } else if (payment_status === 'failed') {
      clearInterval(pollInterval);
      showErrorMessage('Payment failed. Please try again.');
    }

    attempts++;
    if (attempts >= maxAttempts) {
      clearInterval(pollInterval);
      showErrorMessage('Payment timeout. Please try again.');
    }
  }, 3000);
}
```

### Pattern 3: B2B Payouts (Direct Bank/M-Pesa Transfers)

Send payments directly to business partner accounts:

```javascript
// Backend: Initiate B2B payout
async function payBusiness(vendorPhone, amount, reference) {
  const hash = generateHash(vendorPhone, amount, reference);

  const response = await fetch('https://apis.ipayafrica.com/payments/v2/b2b', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      phone: vendorPhone,
      amount,
      reference,
      hash
    })
  });

  return await response.json();
}
```

### Pattern 4: Card Payment with 3DS Authentication

Securely process credit/debit card payments:

```javascript
// Frontend: Collect card via secure form
function processCardPayment(cardDetails, amount, reference) {
  const hash = generateCardHash(cardDetails, amount, reference);

  // Send to backend - NEVER expose card data to client
  fetch('/api/process-card', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      amount,
      reference,
      sid: sessionId // from C2B registration
    })
  });
}

// Backend: Handle card securely
app.post('/api/process-card', async (req, res) => {
  // Card details should ONLY be accepted from secure form
  // Never send card data client-side to iPay
  const hash = generateHash(
    cardDetails.number,
    cardDetails.cvv,
    amount,
    reference
  );

  const response = await fetch(
    'https://apis.ipayafrica.com/payments/v2/transact',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        amount,
        sid: req.body.sid,
        channel: 'card',
        hash,
        // Card details here
      })
    }
  );

  // Handle 3DS redirect if required
  const result = await response.json();
  if (result.requires_3ds) {
    res.json({ redirect_url: result.acs_url });
  }
});
```

## Error Handling

### Common HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process response normally |
| 400 | Invalid parameters | Check request format and parameter values |
| 401 | Unauthorized | Verify API credentials and HMAC hash |
| 403 | Forbidden | Confirm merchant account is active |
| 404 | Endpoint not found | Check API endpoint URL |
| 429 | Rate limited | Implement exponential backoff retry |
| 500 | Server error | Retry request after 30 seconds |
| 503 | Service unavailable | Retry with exponential backoff |

### Transaction-Level Error Codes

When `status` is `failed` in response or callback, the `psp_response_code` contains error details:

| Code | Description | User Action |
|------|-------------|-------------|
| 00 | Success | N/A - transaction complete |
| 02 | Declined by issuer | Contact your bank |
| 05 | Insufficient funds | Add funds and retry |
| 08 | Authentication failed | Verify credentials and retry |
| 12 | Invalid transaction | Check amount and try again |
| 13 | Invalid amount | Amount outside allowed range |
| 14 | Invalid card number | Verify card number |
| 15 | No such issuer | Card network not supported |
| 19 | Re-enter transaction | Retry transaction |
| 25 | Transaction not allowed | Contact support |
| 28 | Timeout / Network error | Check connection and retry |
| 39 | Invalid merchant | Verify merchant account |
| 54 | Expired card | Use valid card |
| 55 | PIN attempts exceeded | Retry after 24 hours |
| 62 | Restricted card | Contact issuer |
| 89 | Cryptographic error | Verify hash calculation |
| 91 | Network unavailable | Retry later |

### Retry Logic

```javascript
// Implement exponential backoff
async function retryRequest(requestFn, maxRetries = 5) {
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      return await requestFn();
    } catch (error) {
      if (error.status === 429 || error.status >= 500) {
        const delay = Math.pow(2, attempt) * 1000; // 1s, 2s, 4s, 8s, 16s
        await sleep(delay);
        attempt++;
      } else {
        throw error; // Non-retryable error
      }
    }
  }

  throw new Error('Max retries exceeded');
}
```

### Validation Errors in Requests

```json
{
  "status": 400,
  "errors": [
    {
      "field": "amount",
      "message": "Amount must be greater than 1 KES"
    },
    {
      "field": "phone",
      "message": "Invalid phone format. Use 254712345678"
    }
  ]
}
```

## Important Notes & Gotchas

### 1. **HMAC Hash Parameter Order is Critical**

Different endpoints require parameters in different orders. **Parameter order is not alphabetical** and must match iPay's specification:
- C2B: `amount + phone + reference`
- STK Push: `phone + vid + amount + account`
- Callback validation: `amount + phone + reference`

Verify the exact order in iPay documentation for each endpoint. A single misplaced parameter breaks signature validation.

### 2. **Callback Hash Validation Must Use Request Parameters, Not Response Data**

When validating callbacks, construct the datastring from the incoming callback data, not your stored request data. Amounts may differ due to fees or currency conversion:

```javascript
// CORRECT - use callback parameters
const datastring = callback.amount + callback.phone + callback.reference;

// WRONG - using stored request data
const datastring = originalRequest.amount + phone + reference;
```

### 3. **Phone Numbers Must Use E.164 Format**

Phone numbers must be formatted as `254712345678` (country code 254 for Kenya with leading zero removed):

```javascript
// CORRECT formats
254712345678  // Kenya Safaricom
254720000000  // Kenya Airtel
254706000000  // Kenya EazzyPay

// INCORRECT formats
+254712345678 // Extra plus sign
0712345678    // Missing country code
712345678     // Missing country code AND leading zero
```

### 4. **Callback Notifications May Arrive Out of Order**

iPay may send multiple callbacks if network issues occur. Implement idempotency by checking `transaction_id`:

```javascript
// Store processed transaction IDs
const processedTransactions = new Set();

app.post('/api/callback', (req, res) => {
  const { transaction_id, status } = req.body;

  // Skip if already processed
  if (processedTransactions.has(transaction_id)) {
    return res.status(200).json({ status: 'ok' });
  }

  // Process payment
  updateOrderStatus(transaction_id, status);
  processedTransactions.add(transaction_id);

  res.status(200).json({ status: 'ok' });
});
```

### 5. **M-Pesa STK Push Requires Specific Data in Correct Order**

The `account` field in STK push must match your stored reference for the transaction. The hash is computed from `phone + vid + amount + account` in that exact order. Mixing up `reference` and `account` will cause hash mismatches.

### 6. **Transaction Statuses Change Asynchronously**

Transactions begin as `pending` after initiation. Status changes to `success` or `failed` when payment completes. Check callback webhooks or poll the search endpoint - do NOT assume immediate status change:

```javascript
// WRONG - assume immediate completion
const response = await registerTransaction();
console.log(response.status); // Still "pending"!

// CORRECT - wait for callback or poll
await waitForCallback(transactionId);
```

### 7. **Staging and Live Use Different Endpoints and Credentials**

Ensure you're using the correct environment:
- Staging: `https://apis.staging.ipayafrica.com` (for testing)
- Live: `https://apis.ipayafrica.com` (for production)

Merchants have separate credentials in each environment. Using staging credentials with live endpoint results in authentication failures.

### 8. **HTTPS Only - All Requests Must Be Encrypted**

iPay only accepts HTTPS requests. HTTP requests are rejected. For development, use the staging endpoint to avoid SSL certificate issues on localhost. Never send card data or API keys over unencrypted connections.

### 9. **Idempotent Request IDs Not Supported**

Unlike some payment APIs, iPay does not support idempotent request headers. If you retry a failed request, ensure sufficient time has passed before retry to avoid duplicate transactions. Include your own reference ID and deduplication logic.

### 10. **Rate Limiting and Timeouts**

- API rate limit: ~100 requests per minute per merchant
- Request timeout: 30 seconds (implement client-side timeouts)
- STK push timeout: ~120 seconds (customer has 2 minutes to enter PIN)
- Callback retry period: iPay retries up to 10 times over 24 hours

### 11. **Limited Error Details in Production**

Production responses provide minimal error information for security. Error codes like `89 (Cryptographic error)` typically indicate HMAC hash issues. Enable detailed logging and test thoroughly in staging.

### 12. **Transaction Search Endpoint Delays**

Transactions may take 5-10 seconds to appear in the transaction search endpoint after completion. Implement a retry loop with delays when immediately searching for transactions.

## Useful Links

- **iPay Official Website**: https://www.ipayafrica.com/
- **C2B API Documentation**: https://dev.ipayafrica.com/C2B.html
- **B2B API Documentation**: https://dev.ipayafrica.com/B2B.html
- **B2C API Documentation**: https://dev.ipayafrica.com/B2C.html
- **Billing API Documentation**: https://dev.ipayafrica.com/billing.html
- **Developer Portal**: https://dev.ipayafrica.com/
- **GitHub PHP Library**: https://github.com/SmoDav/ipay
- **GitHub Node.js Library**: https://github.com/frankizzo/ipayafrica
- **NPM Package**: https://www.npmjs.com/package/ipayafrica
- **Strapi Integration Guide**: https://strapi.io/blog/how-to-integrate-i-pay-africa-ap-is-into-your-strapi-application
- **eLipa API Documentation**: https://elipa.global/dev/Kenya/C2B/
- **Merchant Support Email**: Contact through iPay dashboard
- **Status Page**: Monitor platform uptime via status dashboard

---

**Documentation Quality Note**: This SKILL.md is based on publicly available documentation and search results as of February 2026. Official iPay documentation at dev.ipayafrica.com provides authoritative reference material. Always verify endpoint URLs, parameter requirements, and error codes against current official documentation before implementing production integrations, as payment APIs frequently update endpoints and requirements.
