---
name: pesapal
description: "Integrate with Pesapal, East Africa's leading payment gateway. Use this skill to accept payments in Kenya (KES), Uganda (UGX), Tanzania (TZS), and Rwanda (RWF) via M-Pesa, Airtel Money, MTN Mobile Money, cards, bank transfers, and digital wallets. Trigger when handling payment processing, transaction verification, merchant integrations, or any Pesapal API work across East Africa."
---

# Pesapal Integration Skill

Pesapal is East Africa's leading payment gateway, processing payments across Kenya, Uganda, Tanzania, and Rwanda. It powers online and offline payments for businesses with a modern REST API 3.0 supporting multiple payment channels through a single integration: M-Pesa, Airtel Money, MTN Mobile Money, Visa, Mastercard, American Express, and bank transfers.

## When to use this skill

Use this skill when building payment solutions for East Africa:
- E-commerce checkouts requiring multi-currency support (KES, UGX, TZS, RWF)
- Subscription billing and recurring payment systems
- Marketplace payments with instant settlement
- Invoice and utility payment collection
- Mobile money integration across multiple carriers
- Any application accepting payments in Kenya, Uganda, Tanzania, or Rwanda

Pesapal abstracts away payment complexity by supporting 7+ payment methods through a single API, handling currency conversion, and providing real-time payment notifications via IPN webhooks.

## Authentication

Pesapal uses OAuth 2.0 Bearer Token authentication. Your Consumer Key and Consumer Secret (provided by Pesapal) are exchanged for a short-lived access token valid for 5 minutes.

### Obtaining Credentials

Register a merchant account at [Pesapal Dashboard](https://dashboard.pesapal.com). Your Consumer Key and Consumer Secret are generated automatically and displayed in your dashboard. Store these securely in environment variables:

```
PESAPAL_CONSUMER_KEY=your_consumer_key_here
PESAPAL_CONSUMER_SECRET=your_consumer_secret_here
```

### Request Token Endpoint

```
POST https://pay.pesapal.com/v3/api/Auth/RequestToken
Content-Type: application/json
```

**Request Body:**
```json
{
  "consumer_key": "your_consumer_key",
  "consumer_secret": "your_consumer_secret"
}
```

**Success Response (200 OK):**
```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwczovL3BheS5wZXNhcGFsLmNvbSIsImF1ZCI6IlBlc2FwYWwiLCJpYXQiOjE3MDg2NTk2MjcsImV4cCI6MTcwODY1OTkyN30.abc123xyz",
  "expiresIn": 300,
  "error": null
}
```

### Token Usage

Include the token as a Bearer token in subsequent requests:

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
```

Tokens expire after 300 seconds (5 minutes). Always handle 401 responses by requesting a fresh token. Implement token caching to avoid excessive token requests.

### Base URLs

- **Production:** `https://pay.pesapal.com/v3/api`
- **Sandbox/Testing:** `https://cybqa.pesapal.com/pesapalv3/api`

## Core API Reference

All endpoints require the `Authorization: Bearer {token}` header and `Content-Type: application/json`.

### 1. Request OAuth Token

Generate a Bearer token from your credentials.

**Endpoint:**
```
POST /Auth/RequestToken
```

**Request:**
```json
{
  "consumer_key": "YOUR_CONSUMER_KEY",
  "consumer_secret": "YOUR_CONSUMER_SECRET"
}
```

**Response:**
```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "expiresIn": 300,
  "error": null
}
```

Use the `token` value in all subsequent API calls. Cache this token and refresh when you receive a 401 response.

---

### 2. Register Webhook/IPN Notification URL

Before submitting order requests, register a webhook URL where Pesapal will send payment status notifications.

**Endpoint:**
```
POST /URLSetup/RegisterIPN
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "url": "https://yourdomain.com/api/pesapal/webhook",
  "ipn_notification_type": "GET"
}
```

**Response (200 OK):**
```json
{
  "url": "https://yourdomain.com/api/pesapal/webhook",
  "created_date": "2025-02-20T10:30:00Z",
  "ipn_id": "ca870eb0-3d31-4cba-b7e3-e53c1fb13fbd",
  "error": null,
  "status": "200"
}
```

**Important Notes:**
- The `ipn_id` returned is your IPN ID (notification_id) — save this; you'll reference it as `notification_id` in order requests
- The webhook URL must be publicly accessible and return a 200 OK response
- Pesapal will retry failed webhook deliveries
- During development, use an ngrok tunnel: `https://your-ngrok-url.ngrok-free.app/api/pesapal/webhook`

---

### 3. Submit Order Request (Initiate Payment)

Create a payment transaction and receive a checkout URL to redirect customers to.

**Endpoint:**
```
POST /Transactions/SubmitOrderRequest
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "id": "ORDER-20250220-001",
  "currency": "KES",
  "amount": 1500.00,
  "description": "Purchase of digital subscription",
  "callback_url": "https://yourdomain.com/payment/callback",
  "redirect_mode": "",
  "notification_id": "ca870eb0-3d31-4cba-b7e3-e53c1fb13fbd",
  "branch": "Online Store - Nairobi",
  "billing_address": {
    "email_address": "customer@example.com",
    "phone_number": "+254712345678",
    "country_code": "KE",
    "first_name": "John",
    "middle_name": "",
    "last_name": "Doe",
    "line_1": "123 Main Street",
    "line_2": "Apartment 4B",
    "city": "Nairobi",
    "state": "Nairobi",
    "postal_code": "00100",
    "zip_code": ""
  }
}
```

**Response (200 OK):**
```json
{
  "order_tracking_id": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
  "merchant_reference": "ORDER-20250220-001",
  "redirect_url": "https://pay.pesapal.com/checkout?reference=9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
  "error": null
}
```

**Field Details:**
- `id`: Your unique order reference (must be unique per transaction)
- `currency`: Payment currency — KES, UGX, TZS, or RWF
- `amount`: Payment amount in decimal format (e.g., 1500.00 for 1500 KES)
- `notification_id`: IPN ID from RegisterIPNURL endpoint (mandatory)
- `callback_url`: URL customer returns to after payment (gets order_tracking_id as query param)
- `redirect_mode`: Optional; specify preferred payment method or leave blank for all methods

**Usage Flow:**
1. Create the order request
2. Redirect customer to the `redirect_url`
3. Customer selects payment method and completes payment
4. Pesapal redirects to `callback_url` with `?order_tracking_id=9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b`
5. Call GetTransactionStatus to verify the payment server-side

---

### 4. Get Transaction Status

Verify payment completion server-side. Call this after redirect callback and when receiving webhook notifications.

**Endpoint:**
```
GET /Transactions/GetTransactionStatus?order_tracking_id=9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "12345",
  "reference": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
  "merchant_reference": "ORDER-20250220-001",
  "amount": 1500.00,
  "currency": "KES",
  "status": "COMPLETED",
  "description": "Purchase of digital subscription",
  "payment_method": "MPESA",
  "payment_status_description": "Transaction has been processed successfully",
  "created_date": "2025-02-20T10:30:00Z",
  "completion_date": "2025-02-20T10:32:15Z",
  "confirmation_code": "PESAPAL_CONF_ABC123XYZ"
}
```

**Status Values:**
- `COMPLETED`: Payment successful; funds received
- `PENDING`: Payment initiated; awaiting completion
- `FAILED`: Payment declined or failed; customer can retry
- `INVALID`: Invalid or cancelled transaction

**Validation Logic:**
```javascript
// Pseudo-code for payment verification
if (response.status === "COMPLETED" && response.amount === expectedAmount && response.currency === expectedCurrency) {
  // Payment verified — fulfill order
  fulfillOrder(response.merchant_reference);
} else {
  // Payment incomplete or mismatched — do not fulfill
  logError(`Payment verification failed for ${response.merchant_reference}`);
}
```

---

### 5. Get Transaction List

Retrieve paginated transaction history for reporting and reconciliation.

**Endpoint:**
```
GET /Transactions/GetTransactions?pageNumber=1&pageSize=50&startDate=2025-01-01&endDate=2025-12-31
Authorization: Bearer {token}
```

**Query Parameters:**
- `pageNumber`: Pagination page (1-indexed, default: 1)
- `pageSize`: Records per page (1-100, default: 50)
- `startDate`: ISO 8601 date filter (optional)
- `endDate`: ISO 8601 date filter (optional)

**Response:**
```json
{
  "data": [
    {
      "id": "12345",
      "reference": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
      "merchant_reference": "ORDER-20250220-001",
      "amount": 1500.00,
      "currency": "KES",
      "status": "COMPLETED",
      "payment_method": "MPESA",
      "created_date": "2025-02-20T10:30:00Z",
      "completion_date": "2025-02-20T10:32:15Z"
    }
  ],
  "pageNumber": 1,
  "pageSize": 50,
  "totalRecords": 1250
}
```

Use this endpoint for transaction reconciliation, reporting, and audit trails.

---

### 6. Refund Request

Issue full or partial refunds for completed transactions. Refunds are sent to the original payment method.

**Endpoint:**
```
POST /Transactions/RefundRequest
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "reference": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
  "amount": 500.00,
  "reason": "Customer requested refund due to duplicate charge",
  "username": "merchant_username"
}
```

**Response (200 OK):**
```json
{
  "status": "PENDING",
  "refund_id": "REF-20250220-001",
  "reference": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
  "amount": 500.00,
  "reason": "Customer requested refund due to duplicate charge",
  "created_date": "2025-02-20T11:45:00Z"
}
```

**Refund Constraints:**
- Can only refund COMPLETED transactions
- Cannot refund more than the original transaction amount
- Partial refunds allowed for card payments only (mobile money refunds must be full amount)
- Only one refund per transaction
- Refunds are performed in the original transaction currency
- Refund approval required by merchant (not instant)

**Refund Timeline:**
- Card refunds: 3-5 business days to customer's card
- Mobile money refunds: 1-2 hours to customer's wallet

---

### 7. Get Registered IPNs

Retrieve all registered webhook URLs for your merchant account.

**Endpoint:**
```
GET /URLSetup/GetIpnList
Authorization: Bearer {token}
```

**Response:**
```json
{
  "data": [
    {
      "id": "ca870eb0-3d31-4cba-b7e3-e53c1fb13fbd",
      "url": "https://yourdomain.com/api/pesapal/webhook",
      "active": true,
      "created_date": "2025-02-20T10:30:00Z"
    }
  ]
}
```

Use this to verify registered webhooks or manage multiple notification URLs.

---

### 8. Cancel Order

Cancel a pending or failed order request.

**Endpoint:**
```
POST /Transactions/CancelOrder
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "order_tracking_id": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b"
}
```

**Response:**
```json
{
  "order_tracking_id": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
  "status": "CANCELLED"
}
```

Only orders in PENDING or FAILED status can be cancelled.

---

## Webhooks and IPN (Instant Payment Notifications)

### IPN Flow

When a customer completes or fails a payment, Pesapal sends a POST request to your registered webhook URL with transaction details.

**Webhook Request to Your Server:**
```
POST https://yourdomain.com/api/pesapal/webhook
Content-Type: application/json

{
  "id": "12345",
  "reference": "9c4c5b3e6f2a1d8e7c9b4a5f6d3e2c1b",
  "merchant_reference": "ORDER-20250220-001",
  "status": "COMPLETED",
  "amount": 1500.00,
  "currency": "KES",
  "payment_method": "MPESA",
  "payment_status_description": "Transaction has been processed successfully",
  "created_date": "2025-02-20T10:30:00Z",
  "confirmation_code": "PESAPAL_CONF_ABC123XYZ"
}
```

### Webhook Best Practices

1. **Always verify server-side:** Never trust webhook data alone. Always call `GetTransactionStatus` to verify the payment:

```javascript
// Example: Node.js webhook handler
app.post('/api/pesapal/webhook', async (req, res) => {
  const { reference } = req.body;

  try {
    // Verify transaction on Pesapal servers
    const response = await fetch(
      `https://pay.pesapal.com/v3/api/Transactions/GetTransactionStatus?order_tracking_id=${reference}`,
      {
        headers: {
          'Authorization': `Bearer ${accessToken}`,
          'Content-Type': 'application/json'
        }
      }
    );

    const transaction = await response.json();

    if (transaction.status === 'COMPLETED') {
      // Update database and fulfill order
      await db.orders.update(
        { pesapal_reference: reference },
        { status: 'paid', completed_at: new Date() }
      );
      await fulfillOrder(transaction.merchant_reference);
    }

    // Always return 200 OK to acknowledge receipt
    res.status(200).json({ success: true });
  } catch (error) {
    console.error('Webhook processing failed:', error);
    res.status(500).json({ error: 'Webhook processing failed' });
  }
});
```

2. **Return 200 OK immediately:** Pesapal expects a 200 response within 5 seconds. Process asynchronously if needed.

3. **Handle retries:** Pesapal retries failed webhooks; use idempotency keys to prevent duplicate processing.

4. **Log all webhooks:** Store webhook payloads for audit trails and debugging.

### IPN Registration in Dashboard

Alternatively, register webhooks directly in your Pesapal Dashboard:
1. Navigate to Settings → API Notifications
2. Enter your webhook URL
3. Click "Register"
4. Save the returned notification_id

---

## Common Integration Patterns

### Pattern 1: Basic Payment Checkout

```
1. Customer clicks "Pay Now"
   ↓
2. Backend: POST /Auth/RequestToken → get Bearer token
   ↓
3. Backend: POST /Transactions/SubmitOrderRequest → get redirect_url
   ↓
4. Frontend: Redirect customer to redirect_url
   ↓
5. Customer: Selects payment method and pays
   ↓
6. Pesapal: Redirects to callback_url with ?order_tracking_id=XXX
   ↓
7. Frontend: GET /Transactions/GetTransactionStatus from backend
   ↓
8. Backend: Verify status === "COMPLETED" and amount matches
   ↓
9. Fulfill order (email confirmation, download link, shipment, etc.)
   ↓
10. (Async) Pesapal IPN webhook provides redundant confirmation
```

### Pattern 2: Webhook-Driven Order Fulfillment

```
1. Customer completes payment on Pesapal's page
   ↓
2. Pesapal sends IPN to your webhook with transaction details
   ↓
3. Your webhook handler calls GetTransactionStatus to verify
   ↓
4. If verified, update database and trigger fulfillment
   ↓
5. Return 200 OK to Pesapal
```

### Pattern 3: Order Tracking Page

```
1. After redirect callback, customer sees "Processing..." page
   ↓
2. Page polls GET /Transactions/GetTransactionStatus every 2-3 seconds
   ↓
3. On status change to COMPLETED, display confirmation
   ↓
4. Webhook provides server-side confirmation for fulfillment
```

### Pattern 4: Multi-Currency Marketplace

```
// Pesapal automatically converts prices
const submitOrder = async (order) => {
  const currency = {
    'KE': 'KES',
    'UG': 'UGX',
    'TZ': 'TZS',
    'RW': 'RWF'
  }[order.country];

  const payload = {
    id: order.id,
    currency: currency,
    amount: order.amount,
    billing_address: {
      ...order.billing_data,
      country_code: order.country
    },
    // ... other fields
  };

  return await submitOrderRequest(payload);
};
```

---

## Error Handling

### HTTP Status Codes

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | Success | Process response |
| 400 | Bad Request | Validate request fields and retry |
| 401 | Unauthorized | Request fresh token and retry |
| 404 | Not Found | Verify order_tracking_id exists |
| 500 | Server Error | Implement exponential backoff retry |
| 503 | Service Unavailable | Retry after 30+ seconds |

### Common Error Scenarios

Pesapal wraps all errors in an `error` object with `type`, `code`, and `message` fields:

```json
{
  "error": {
    "type": "error_type",
    "code": "error_code",
    "message": "Detailed error message"
  }
}
```

**1. Invalid Token (401)**
```json
{
  "error": {
    "type": "unauthorized",
    "code": "401",
    "message": "Invalid or expired token"
  }
}
```
**Solution:** Request a new token via `POST /Auth/RequestToken`

**2. Missing Required Fields (400)**
```json
{
  "error": {
    "type": "bad_request",
    "code": "400",
    "message": "notification_id is required"
  }
}
```
**Solution:** Ensure all required fields in request body are present

**3. Duplicate Order ID (400)**
```json
{
  "error": {
    "type": "bad_request",
    "code": "400",
    "message": "Order with id ORDER-20250220-001 already exists"
  }
}
```
**Solution:** Ensure order IDs are globally unique; use timestamps or UUIDs

**4. Order Not Found (404)**
```json
{
  "error": {
    "type": "not_found",
    "code": "404",
    "message": "Transaction not found"
  }
}
```
**Solution:** Verify the order_tracking_id is correct

### Retry Strategy

Implement exponential backoff for transient errors:

```javascript
const retryWithBackoff = async (fn, maxRetries = 3) => {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      if (![500, 503].includes(error.status)) throw error; // Don't retry client errors

      const delayMs = Math.pow(2, attempt) * 1000; // 1s, 2s, 4s
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
  }
};

// Usage
const transaction = await retryWithBackoff(() =>
  getTransactionStatus(order_tracking_id)
);
```

---

## Supported Currencies and Payment Methods

### Kenya (KES)
- **Mobile Money:** M-Pesa (Safaricom)
- **Cards:** Visa, Mastercard, American Express
- **Bank Transfers:** Equity Bank, KCB, Standard Chartered
- **Digital:** Airtel Money

### Uganda (UGX)
- **Mobile Money:** MTN Mobile Money, Airtel Money
- **Cards:** Visa, Mastercard
- **Bank Transfers:** Stanbic, Standard Chartered, Uganda Commercial Bank
- **Digital:** Pesapal Sabi (POS terminal for Visa/MC)

### Tanzania (TZS)
- **Mobile Money:** Tigo Pesa, Vodacom M-Pesa, Airtel Money
- **Cards:** Visa, Mastercard
- **Bank Transfers:** NMB Bank, CRDB Bank, Equity Bank

### Rwanda (RWF)
- **Mobile Money:** MTN Mobile Money, Airtel Money
- **Cards:** Visa, Mastercard
- **Bank Transfers:** BPR Bank, Equity Bank, KCB Rwanda

---

## Important Notes and Gotchas

### Token Management
- Tokens expire after 300 seconds (5 minutes)
- Cache tokens and reuse within the 5-minute window to reduce requests
- Always handle 401 responses by requesting a fresh token
- Never hardcode tokens; always request fresh tokens on startup

### Order IDs
- Must be unique per transaction across your entire system
- Use UUID v4 or `timestamp-random` format to ensure uniqueness
- Cannot contain special characters; use alphanumeric and hyphens only
- 50-character maximum length

### Payment Verification
- Never fulfill orders based on frontend callbacks alone
- Always verify `status === "COMPLETED"` and `amount === expectedAmount` server-side
- Implement idempotent order fulfillment to handle duplicate webhooks
- Use database transactions to prevent double-fulfillment

### IPN Webhooks
- Webhook URLs must be publicly accessible (no localhost/private IPs)
- Return 200 OK within 5 seconds; process fulfillment asynchronously
- Pesapal retries failed webhooks; implement idempotency
- During development, use ngrok or similar tunneling service
- Test webhook registration before going live

### Currency Handling
- Amounts must be in decimal format (1500.00, not 150000 cents)
- Pesapal automatically handles local currency conversions
- Card payments may be in USD; mobile money in local currency
- Specify correct currency code (KES, UGX, TZS, RWF) or request will fail

### Refunds
- Only COMPLETED transactions can be refunded
- Cannot refund more than the original amount
- Partial refunds only supported for card payments
- Mobile money refunds must be full amount
- Only one refund per transaction allowed
- Merchant approval required; not instant

### PCI-DSS Compliance
- Pesapal is PCI-DSS Level 1 certified
- Never capture or store card data yourself
- All card data handled by Pesapal's secure page
- Your application never receives raw card information

### Testing
- Use sandbox credentials from [Pesapal Dashboard](https://dashboard.pesapal.com)
- Sandbox URL: `https://cybqa.pesapal.com/pesapalv3/api`
- Test with various payment methods before going live
- Sandbox transactions don't incur fees

---

## Useful Links

- **Developer Documentation:** [https://developer.pesapal.com/](https://developer.pesapal.com/)
- **API Reference:** [https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/api-reference](https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/api-reference)
- **Authentication Guide:** [https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/authentication](https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/authentication)
- **SubmitOrderRequest:** [https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/submitorderrequest](https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/submitorderrequest)
- **GetTransactionStatus:** [https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/gettransactionstatus](https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/gettransactionstatus)
- **RegisterIPNURL:** [https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/registeripnurl](https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/registeripnurl)
- **RefundRequest:** [https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/refund-request](https://developer.pesapal.com/how-to-integrate/e-commerce/api-30-json/refund-request)
- **Merchant Dashboard:** [https://dashboard.pesapal.com](https://dashboard.pesapal.com)
- **Forum & Support:** [https://developer.pesapal.com/forum](https://developer.pesapal.com/forum)
