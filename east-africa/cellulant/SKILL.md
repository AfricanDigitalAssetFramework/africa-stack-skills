---
name: cellulant
description: "Integrate with Cellulant Tingg pan-African checkout API for payment collection across 25+ African countries. Use for processing payments via mobile money (M-Pesa, MTN, Airtel), cards (Visa/Mastercard), bank transfers, USSD, and digital wallets. Trigger on keywords: 'Cellulant', 'Tingg', 'checkout', 'payment collection', 'bill payments', 'invoice collection', 'multi-channel payments', 'African payments'."
---

# Cellulant Tingg Checkout API Integration

Cellulant's Tingg is a pan-African payment collection platform providing unified checkout for 25+ African countries via 283+ payment methods including mobile money, cards, bank transfers, USSD, and digital wallets. Single integration reaches Kenya, Nigeria, Ghana, Tanzania, Uganda, Rwanda, Zambia, Botswana, Mozambique, and more.

## When to use this skill

You're building a payment collection system for Africa: e-commerce checkout, utility bill payments, insurance premiums, invoice collection, subscription billing, or donation collection. Tingg abstracts country-specific complexity with one API supporting local payment methods, multi-currency transactions, real-time settlement, and built-in fraud protection. Perfect for businesses operating across multiple African markets or requiring seamless cross-border payment collection.

## Authentication

Tingg v3 API uses OAuth 2.0 bearer token authentication with client credentials flow.

### Getting Credentials

1. Sign up at [Tingg Dashboard](https://dashboard.tingg.africa)
2. Create an API application in your merchant account
3. Receive `client_id` and `client_secret`
4. Generate access tokens via authentication endpoint

### Generate Access Token

```
POST https://api.tingg.africa/v3/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Store the `access_token` and use for subsequent API calls. Tokens expire after 1 hour; regenerate as needed.

### API Headers

Include these headers in all requests:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

Store credentials in environment variables:
```bash
TINGG_CLIENT_ID=your_client_id
TINGG_CLIENT_SECRET=your_client_secret
```

### Base URLs

- **Production:** `https://api.tingg.africa/v3`
- **Sandbox/Approval:** `https://api-approval.tingg.africa/v3`

## Core API Reference

### 1. Initiate Checkout Request

Create a payment checkout session and redirect customer to hosted payment page.

```
POST /checkout-api/checkout/request
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "merchant_transaction_id": "TXN-2025022401",
  "merchant_reference": "ORD-98765",
  "account_number": "ACC-12345",
  "service_code": "GenericService",
  "currency_code": "KES",
  "country_code": "KE",
  "request_amount": 1500,
  "customer_email": "customer@example.com",
  "customer_first_name": "John",
  "customer_last_name": "Doe",
  "msisdn": "+254712345678",
  "callback_url": "https://yoursite.com/webhooks/tingg",
  "fail_redirect_url": "https://yoursite.com/checkout/failed",
  "success_redirect_url": "https://yoursite.com/checkout/success",
  "metadata": {
    "order_id": "ORD-98765",
    "product": "Service Package",
    "description": "Premium subscription"
  }
}
```

**Field Reference:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| merchant_transaction_id | string | Yes | Unique transaction ID in your system (max 50 chars). Must be unique per request. |
| merchant_reference | string | Yes | Human-readable reference like order/invoice number (max 50 chars) |
| account_number | string | No | Customer account number in your system |
| service_code | string | Yes | Service identifier (use "GenericService" for basic payments) |
| currency_code | string | Yes | 3-letter ISO currency code (KES, NGN, GHS, UGX, TZS, RWF, ZMW, BWP) |
| country_code | string | Yes | 2-letter ISO country code (KE, NG, GH, UG, TZ, RW, ZM, BW) |
| request_amount | integer | Yes | Amount in smallest currency unit (cents/pesewa). Whole numbers only. <!-- TODO: verify — M-Pesa in KES does not use sub-unit "cents"; confirm with Tingg docs whether KES amounts are in whole shillings or in cents --> |
| customer_email | string | Yes | Payer's email address |
| customer_first_name | string | Yes | Payer's first name |
| customer_last_name | string | Yes | Payer's last name |
| msisdn | string | Yes | Payer's phone number with country code (e.g., +254712345678) |
| callback_url | string | Yes | Your endpoint to receive payment webhooks (must be HTTPS) |
| fail_redirect_url | string | No | Redirect customer here if payment fails/cancelled |
| success_redirect_url | string | No | Redirect customer here after successful payment |
| metadata | object | No | Custom fields (max 10, up to 500 chars total) |

**Response:**
```json
{
  "status_code": 200,
  "status_description": "Checkout request created successfully",
  "merchant_transaction_id": "TXN-2025022401",
  "transaction_id": "TINGG_TXN_a1b2c3d4e5",
  "checkout_url": "https://checkout.tingg.africa/modal/pay/TINGG_TXN_a1b2c3d4e5",
  "expiry_time": "2025-02-24T18:30:00Z"
}
```

**Integration Steps:**
1. Call this endpoint to initiate payment session
2. Redirect customer to `checkout_url` (opens Tingg's hosted payment page)
3. Customer selects payment method and completes payment
4. On completion, Tingg redirects to `success_redirect_url` or `fail_redirect_url`
5. Receive webhook callback at `callback_url` (reliable; customer redirect may fail)
6. Acknowledge the webhook to confirm receipt

### 2. Get Checkout Status

Query the current status of a checkout request.

```
GET /checkout-api/checkout/status/{merchant_transaction_id}
Authorization: Bearer {access_token}
```

**Response (Payment Completed):**
```json
{
  "status_code": 200,
  "status_description": "Success",
  "merchant_transaction_id": "TXN-2025022401",
  "transaction_id": "TINGG_TXN_a1b2c3d4e5",
  "request_amount": 1500,
  "currency_code": "KES",
  "status": "SUCCESS",
  "payment_method": "MPESA",
  "payment_reference": "MPESA_REF_abc123",
  "merchant_reference": "ORD-98765",
  "timestamp": "2025-02-24T14:25:00Z"
}
```

**Possible Status Values:**
- `PENDING` - Payment awaiting customer action
- `PROCESSING` - Payment processing
- `SUCCESS` - Payment completed successfully
- `FAILED` - Payment failed (invalid card, insufficient funds, etc.)
- `CANCELLED` - Customer cancelled the payment

### 3. Acknowledge Payment Webhook

After receiving a webhook, acknowledge it to Tingg (required for reliability—unacknowledged webhooks retry for 24 hours).

```
POST /checkout-api/acknowledgement/request
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "merchant_transaction_id": "TXN-2025022401",
  "status": "SUCCESS"
}
```

**Response:**
```json
{
  "status_code": 200,
  "status_description": "Acknowledgement received successfully"
}
```

Always respond to Tingg with HTTP 200 after acknowledging. If acknowledge fails or times out, Tingg will retry the webhook.

### 4. Query Transactions

Retrieve transaction history with filtering and pagination.

```
GET /checkout-api/transactions?page=1&page_size=50&status=SUCCESS&country_code=KE
Authorization: Bearer {access_token}
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| page | integer | Page number (1-indexed), default 1 |
| page_size | integer | Results per page (1-100), default 20 |
| status | string | Filter by status: SUCCESS, FAILED, CANCELLED, PENDING |
| country_code | string | Filter by country (e.g., KE, NG, GH) |
| from_date | string | ISO 8601 date (e.g., 2025-02-01T00:00:00Z) |
| to_date | string | ISO 8601 date (e.g., 2025-02-28T23:59:59Z) |
| merchant_reference | string | Filter by your reference |

**Response:**
```json
{
  "status_code": 200,
  "status_description": "Success",
  "transactions": [
    {
      "transaction_id": "TINGG_TXN_a1b2c3d4e5",
      "merchant_transaction_id": "TXN-2025022401",
      "merchant_reference": "ORD-98765",
      "request_amount": 1500,
      "currency_code": "KES",
      "country_code": "KE",
      "status": "SUCCESS",
      "payment_method": "MPESA",
      "payment_reference": "MPESA_REF_abc123",
      "customer_email": "customer@example.com",
      "created_at": "2025-02-24T14:25:00Z",
      "paid_at": "2025-02-24T14:26:30Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 50,
    "total_count": 250,
    "total_pages": 5
  }
}
```

### 5. Fetch Available Payment Methods

Get payment methods available for a country/currency combination (optional pre-validation).

```
GET /checkout-api/payment-methods?country_code=KE&currency_code=KES
Authorization: Bearer {access_token}
```

**Response:**
```json
{
  "status_code": 200,
  "status_description": "Success",
  "payment_methods": [
    {
      "code": "MPESA",
      "name": "M-Pesa",
      "type": "MOBILE_MONEY",
      "supported_currencies": ["KES"],
      "min_amount": 10,
      "max_amount": 150000
    },
    {
      "code": "VISA",
      "name": "Visa Card",
      "type": "CARD",
      "supported_currencies": ["KES", "USD"],
      "min_amount": 50,
      "max_amount": 500000
    },
    {
      "code": "BANK_TRANSFER",
      "name": "Bank Transfer",
      "type": "BANK",
      "supported_currencies": ["KES"],
      "min_amount": 100,
      "max_amount": 1000000
    },
    {
      "code": "USSD",
      "name": "USSD",
      "type": "USSD",
      "supported_currencies": ["KES"],
      "min_amount": 10,
      "max_amount": 70000
    }
  ]
}
```

## Webhooks

### Payment Webhook Callback

When payment status changes, Tingg sends an HTTP POST to your `callback_url` with payment details.

**Webhook Request:**
```json
{
  "merchant_transaction_id": "TXN-2025022401",
  "transaction_id": "TINGG_TXN_a1b2c3d4e5",
  "merchant_reference": "ORD-98765",
  "request_amount": 1500,
  "currency_code": "KES",
  "country_code": "KE",
  "status": "SUCCESS",
  "payment_method": "MPESA",
  "payment_reference": "MPESA_REF_abc123",
  "customer_email": "customer@example.com",
  "timestamp": "2025-02-24T14:26:30Z"
}
```

### Webhook Security

Tingg does NOT currently use signature verification on webhooks. However, follow these practices:

1. **Verify merchant_transaction_id** - Ensure it matches a pending transaction in your system
2. **Verify amounts** - Confirm request_amount matches your records
3. **Check timestamp** - Ensure webhook is recent (not replayed)
4. **Store webhook data** - Log all webhooks for reconciliation
5. **Implement idempotency** - Handle duplicate webhooks safely (same merchant_transaction_id received twice)

### Webhook Handling Best Practices

```javascript
// Example webhook handler (Node.js)
app.post('/webhooks/tingg', async (req, res) => {
  const { merchant_transaction_id, status, request_amount } = req.body;

  try {
    // 1. Verify transaction exists in your system
    const transaction = await db.transactions.findOne({ id: merchant_transaction_id });
    if (!transaction) {
      console.error(`Unknown transaction: ${merchant_transaction_id}`);
      return res.status(400).json({ error: 'Invalid transaction' });
    }

    // 2. Verify amount matches
    if (transaction.amount !== request_amount) {
      console.error(`Amount mismatch for ${merchant_transaction_id}`);
      return res.status(400).json({ error: 'Amount mismatch' });
    }

    // 3. Check for duplicate webhook (idempotency)
    if (transaction.webhook_received) {
      console.log(`Duplicate webhook for ${merchant_transaction_id}`);
      return res.status(200).json({ received: true }); // Ack duplicate
    }

    // 4. Process payment
    if (status === 'SUCCESS') {
      await db.transactions.update(merchant_transaction_id, {
        status: 'COMPLETED',
        paid_at: new Date(req.body.timestamp),
        webhook_received: true
      });
      // Fulfill order, send receipt, etc.
    } else {
      await db.transactions.update(merchant_transaction_id, {
        status: 'FAILED',
        webhook_received: true
      });
    }

    // 5. Acknowledge webhook
    await tinggClient.acknowledge({
      merchant_transaction_id,
      status
    });

    // 6. Return 200 to confirm receipt
    return res.status(200).json({ received: true });

  } catch (error) {
    console.error('Webhook error:', error);
    return res.status(500).json({ error: 'Processing error' });
  }
});
```

**Important:** Always respond with HTTP 200 within 30 seconds. Tingg will retry unacknowledged webhooks with exponential backoff (up to 24 hours).

## Common Integration Patterns

### Pattern 1: Basic Checkout Flow

```
1. Customer initiates payment on your site
   → POST /checkout-api/checkout/request
   ← Receive checkout_url

2. Redirect customer to checkout_url
   → Customer selects payment method & completes payment

3. Customer redirected back (check success_redirect_url)
   → May fail to redirect (network issues, etc.)

4. Receive webhook at callback_url
   → POST /checkout-api/acknowledgement/request to confirm

5. Fulfill order (update database, send email, etc.)
```

### Pattern 2: Hosted Payment Page (Express Checkout)

Use Tingg's pre-built modal/page for quick integration:

```javascript
// Include Tingg SDK
<script src="https://cdn.tingg.africa/tingg-checkout-sdk.min.js"></script>

// Initialize checkout on button click
document.getElementById('payBtn').addEventListener('click', () => {
  const checkout = new Tingg.Checkout({
    key: 'YOUR_API_KEY',
    redirectUrl: 'https://yoursite.com/checkout/success',
    onSuccess: (response) => {
      console.log('Payment successful:', response);
      // Handle success
    },
    onError: (error) => {
      console.error('Payment error:', error);
      // Handle error
    }
  });

  checkout.open({
    amount: 1500,
    currency: 'KES',
    email: 'customer@example.com',
    phone: '+254712345678',
    firstName: 'John',
    lastName: 'Doe'
  });
});
```

### Pattern 3: Transaction Reconciliation

End-of-day verification:

```javascript
// Fetch all transactions from Tingg for a date range
const tinggTxns = await tinggClient.queryTransactions({
  from_date: '2025-02-24T00:00:00Z',
  to_date: '2025-02-24T23:59:59Z'
});

// Compare with your database
const localTxns = await db.transactions.findByDate('2025-02-24');

const tinggMap = new Map(tinggTxns.map(t => [t.merchant_transaction_id, t]));
const localMap = new Map(localTxns.map(t => [t.id, t]));

// Find discrepancies
for (const [id, localTx] of localMap) {
  const tinggTx = tinggMap.get(id);

  if (!tinggTx) {
    console.warn(`Missing in Tingg: ${id}`);
    // Contact support if customer claims they paid
  } else if (tinggTx.status !== localTx.status) {
    console.warn(`Status mismatch for ${id}: Tingg=${tinggTx.status}, Local=${localTx.status}`);
    // Update local record
  } else if (tinggTx.request_amount !== localTx.amount) {
    console.error(`Amount mismatch for ${id}`);
    // Escalate
  }
}
```

### Pattern 4: Subscription Billing

For recurring payments:

```javascript
async function createRecurringPayment(customerId, amount, currency) {
  // Note: Tingg Checkout is one-time by default
  // For subscriptions, create multiple one-time checkout sessions

  const merchant_transaction_id = `SUB-${customerId}-${Date.now()}`;

  const response = await tinggClient.checkout({
    merchant_transaction_id,
    merchant_reference: `Subscription-${customerId}`,
    request_amount: amount,
    currency_code: currency,
    customer_email: customer.email,
    // ... other fields
  });

  // Store subscription info
  await db.subscriptions.create({
    customer_id: customerId,
    merchant_transaction_id,
    amount,
    currency,
    frequency: 'MONTHLY',
    next_billing_date: new Date(Date.now() + 30*86400*1000)
  });

  return response.checkout_url;
}
```

## Error Handling

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad request (invalid fields, validation error) |
| 401 | Unauthorized (invalid/expired token) |
| 403 | Forbidden (merchant account issue) |
| 404 | Resource not found (invalid transaction_id) |
| 429 | Rate limited (too many requests) |
| 500 | Server error (temporary, retry with backoff) |

### Response Status Codes

Tingg includes `status_code` in all responses:

| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Bad request (missing/invalid fields) |
| 401 | Authentication error (invalid token or credentials) |
| 403 | Forbidden (merchant account inactive/suspended) |
| 404 | Transaction not found |
| 409 | Conflict (transaction already processed) |
| 429 | Rate limited |
| 500 | Server error |
| 503 | Service temporarily unavailable |

### Error Response Example

```json
{
  "status_code": 400,
  "status_description": "Invalid request: customer_email is required",
  "errors": [
    {
      "field": "customer_email",
      "message": "Email address is required and must be valid"
    }
  ]
}
```

### Retry Strategy

Implement exponential backoff for retryable errors:

```javascript
async function retryTinggRequest(fn, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (error.statusCode === 429 || error.statusCode >= 500) {
        if (attempt < maxRetries) {
          const delayMs = Math.pow(2, attempt) * 1000; // 2s, 4s, 8s
          console.log(`Retry attempt ${attempt} after ${delayMs}ms`);
          await new Promise(r => setTimeout(r, delayMs));
          continue;
        }
      }
      throw error;
    }
  }
}

// Usage
const result = await retryTinggRequest(() =>
  tinggClient.checkout({ /* params */ })
);
```

## Country & Currency Coverage

### Supported Countries and Currencies

Tingg operates across 25+ African countries with 283+ payment methods:

| Country | Code | Currency | Code | Payment Methods |
|---------|------|----------|------|-----------------|
| Kenya | KE | Kenyan Shilling | KES | M-Pesa, Cards, Bank Transfer, USSD |
| Nigeria | NG | Nigerian Naira | NGN | Cards, Bank Transfer, USSD |
| Ghana | GH | Ghanaian Cedi | GHS | Mobile Money, Cards, Bank Transfer |
| Uganda | UG | Ugandan Shilling | UGX | MTN Money, Airtel, Cards |
| Tanzania | TZ | Tanzanian Shilling | TZS | M-Pesa, Cards, Bank Transfer |
| Rwanda | RW | Rwandan Franc | RWF | Mobile Money, Cards, Bank Transfer |
| Zambia | ZM | Zambian Kwacha | ZMW | Mobile Money, Cards, Bank Transfer |
| Botswana | BW | Botswana Pula | BWP | Cards, Bank Transfer |
| Mozambique | MZ | Mozambican Metical | MZN | Mobile Money, Cards |
| Malawi | MW | Malawian Kwacha | MWK | Mobile Money, Cards |
| Namibia | NA | Namibian Dollar | NAD | Cards, Bank Transfer |
| Zimbabwe | ZW | Zimbabwean Dollar | ZWL | Cards, Bank Transfer |
| South Africa | ZA | South African Rand | ZAR | Cards, Bank Transfer |

*Full list of 25+ countries available in [Tingg documentation](https://docs.tingg.africa/docs/checkout-v3-countries)*

### Payment Methods by Country

**Mobile Money (Primary in East/Central Africa):**
- M-Pesa (Kenya, Tanzania)
- MTN Mobile Money (Uganda, Ghana, Rwanda, Zambia)
- Airtel Money (Kenya, Uganda, Tanzania, Rwanda)
- Orange Money (multiple countries)
- Vodafone Mobile Money
- Eco Cash (Zimbabwe)

**Cards (All countries):**
- Visa
- Mastercard
- All 3D Secure compatible cards

**Bank Transfer/Direct Debit:**
- Direct bank transfers via SWIFT
- Local bank transfers via country-specific systems
- USSD bank transfers (available in most countries)

**Digital Wallets:**
- Regional and local digital wallet solutions
- Country-specific e-wallet providers

## Important Notes and Gotchas

### Critical Implementation Details

1. **Merchant Transaction ID Uniqueness**
   - Must be unique per checkout request
   - Tingg will reject duplicate merchant_transaction_ids within 24 hours
   - Use timestamp + random suffix: `TXN-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`

2. **Amount Precision**
   - Amounts are integers (no decimals)
   - KES: verify whether 1500 means KES 15.00 (cents) or KES 1,500 (whole shillings); Tingg may use whole shilling units for mobile money channels
   - GHS 0.50 = 50 pesewas
   <!-- TODO: confirm exact KES unit with https://docs.tingg.africa before production -->

3. **Phone Number Format**
   - Must include country code with + prefix
   - Examples: `+254712345678`, `+233201234567`, `+256701234567`
   - Tingg validates format; invalid numbers will fail

4. **Webhook Reliability**
   - Webhooks may arrive out of order (payment success after failure)
   - Implement idempotency using merchant_transaction_id
   - Always acknowledge webhooks within 30 seconds
   - Tingg retries for 24 hours if not acknowledged

5. **Token Expiration**
   - Access tokens expire after 1 hour
   - Regenerate before expiry in high-volume integrations
   - Store token expiry time: `expires_at = now + expires_in`

6. **Production Readiness**
   - Test extensively in sandbox (api-approval.tingg.africa)
   - Use Tingg's test phone numbers for testing
   - Coordinate with Tingg support before going live
   - Enable webhook signature verification if available (check docs)

7. **Customer Redirect Failures**
   - Customer may not be redirected to success/fail URLs
   - Always rely on webhooks for payment confirmation, not redirects
   - Check transaction status if no webhook received after 5 minutes

8. **Currency Conversion**
   - Tingg does NOT auto-convert currencies
   - Must use currency supported by customer's country
   - Different countries support different currencies

9. **Settlement**
   - Settlements typically occur within 24-48 hours
   - Check your Tingg dashboard for settlement details
   - Some payment methods (mobile money) may have longer settlement times

10. **Rate Limiting**
    - Specific limits not publicly documented
    - Implement exponential backoff for errors
    - Contact Tingg support for tier-specific limits

### Common Integration Mistakes

**Mistake 1: Not acknowledging webhooks**
→ Causes Tingg to retry for 24 hours, creating duplicate processing

**Mistake 2: Using redirect URL for confirmation**
→ Network failures prevent redirect; always use webhooks

**Mistake 3: Not validating webhook data**
→ Missing amount/transaction_id verification allows fraud

**Mistake 4: Storing plain API credentials**
→ Use environment variables or secrets management

**Mistake 5: Not handling duplicate webhooks**
→ Same merchant_transaction_id may be received twice due to retries

**Mistake 6: Assuming webhook signature verification**
→ Tingg does not currently sign webhooks; verify by comparing stored transaction data

**Mistake 7: Using soft-deleted transactions**
→ If allowing transaction cancellation, mark as canceled in DB, don't delete

**Mistake 8: Not testing in sandbox first**
→ Use api-approval.tingg.africa before production deployment

## Testing

### Sandbox Credentials

```
Base URL: https://api-approval.tingg.africa/v3
Client ID: (provided by Tingg for sandbox)
Client Secret: (provided by Tingg for sandbox)
```

### Test Phone Numbers

Tingg provides test phone numbers in sandbox that trigger specific payment flows:

- Contact Tingg support for test credentials and phone numbers
- Use these for testing different payment methods and error scenarios
- Test webhooks using services like ngrok or localtunnel

### Test Scenarios

```javascript
// Test successful payment
const testSuccess = {
  customer_first_name: 'Test',
  customer_last_name: 'Success',
  msisdn: '+254789000001', // Example test number
  // ... other fields
};

// Test failed payment
const testFailed = {
  customer_first_name: 'Test',
  customer_last_name: 'Failed',
  msisdn: '+254789000002', // Example test number
  // ... other fields
};

// Always use sandbox environment for testing
const client = new TinggClient({
  clientId: process.env.TINGG_CLIENT_ID_SANDBOX,
  clientSecret: process.env.TINGG_CLIENT_SECRET_SANDBOX,
  baseUrl: 'https://api-approval.tingg.africa/v3'
});
```

## Useful Links

- **Official Documentation:** [Tingg Checkout API v3 Docs](https://docs.tingg.africa)
- **Supported Countries:** [Tingg v3 Countries](https://docs.tingg.africa/docs/checkout-v3-countries)
- **Developer Portal:** [Tingg Dev Portal](https://dev-portal.tingg.africa)
- **Payment Options:** [Available Payment Methods](https://docs.tingg.africa/docs/checkout-v3-payment-options)
- **SDK & UI Library:** [Tingg SDKs](https://docs.tingg.africa/docs/checkout-v3-sdks-ui-library)
- **Dashboard:** [Tingg Merchant Dashboard](https://dashboard.tingg.africa)
- **Status Page:** [Tingg System Status](https://status.tingg.africa)
- **Support:** [Tingg Support Portal](https://cellulant-tingg.freshdesk.com)
- **Blog & Resources:** [Cellulant Blog](https://www.cellulant.io)
- **GitHub Examples:** [Cellulant GitHub](https://github.com/CellulantCorp)

## Changelog

### v3 (Current)
- OAuth 2.0 bearer token authentication
- 283+ payment methods across 25+ countries
- Express checkout and custom checkout options
- Enhanced webhook handling
- Improved error messages

### v2 (Legacy)
- HMAC-SHA256 signature authentication
- Limited payment method coverage
- Being phased out—migrate to v3

**Last Updated:** February 2025
**API Version:** v3
**Status:** Production Ready
