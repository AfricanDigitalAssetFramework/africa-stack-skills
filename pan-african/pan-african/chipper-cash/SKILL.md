---
name: chipper-cash
description: "Integrate with Chipper Cash Network API to accept payments and send payouts across 7+ African countries. Use this skill when building payment solutions that need to accept payments from verified customers, send payouts across Africa, or integrate a 'Pay With Chipper' checkout. Trigger for cross-border P2P payments, marketplace payouts, remittances, or multi-currency transactions in Nigeria, Kenya, Uganda, Ghana, Tanzania, Rwanda, or South Africa. Also trigger on mentions of 'Chipper Cash', 'Chipper for Business', 'Chipper Network API', or cross-border African payments."
---

# Chipper Cash Integration Skill

Chipper Cash is a pan-African fintech platform providing payment solutions for businesses and individuals. The **Chipper Network API** is the business offering that enables merchants to accept payments from verified customers and send payouts across multiple African markets with a simple REST API.

## When to use this skill

You're building payment infrastructure for African markets — an e-commerce platform needing to accept payments, a marketplace paying sellers across multiple countries, a remittance service, a business application handling international vendor payments, or any platform needing multi-currency transactions. Chipper Cash provides access to 5M+ KYC-verified customers and supports 40+ markets through direct licensing and partnerships.

**Important:** Chipper Cash is primarily a consumer-focused payment app. The Network API is their business/merchant solution. If you need direct P2P transfers via the consumer app, consider partner integrations or direct merchant onboarding instead of a public API.

## Authentication

Chipper Network API uses API Key authentication. Credentials consist of:
- **Chipper User ID** - Your merchant account identifier
- **Chipper Network API Key** - Your API secret key

Include these in request headers:

```
Authorization: Bearer YOUR_API_KEY
X-Chipper-User-ID: YOUR_USER_ID
```

Keys are provisioned in sandbox and production environments. Store them securely in environment variables like `CHIPPER_API_KEY` and `CHIPPER_USER_ID`. Never hardcode credentials.

**Base URL (Sandbox):** `https://sandbox-api.chippercash.com/v1`
**Base URL (Production):** `https://api.chippercash.com/v1`

## Getting Access

The Network API is not open to all developers. To integrate:

1. Visit https://enterprise.chippercash.com/ or https://www.chippercash.com/api
2. Complete the merchant application form
3. Chipper's team will contact you to verify your business and complete onboarding
4. Upon approval, you'll receive API credentials and gain access to sandbox environment
5. Test thoroughly in sandbox before requesting production access

## Core API Reference

### Create a Payment Order

Initiate a payment order that customers can complete via their Chipper account or linked payment methods.

```
POST /v1/orders
```

**Headers:**
```
Authorization: Bearer YOUR_API_KEY
X-Chipper-User-ID: YOUR_USER_ID
Content-Type: application/json
```

**Body:**
```json
{
  "amount": 5000,
  "currency": "NGN",
  "customer_email": "customer@example.com",
  "description": "Payment for Order #12345",
  "reference": "ORDER_12345_001",
  "redirect_url": "https://yourapp.com/payment-success",
  "metadata": {
    "order_id": "12345",
    "customer_id": "cust_789"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "order_id": "CHI_ORD_xxxxx",
    "payment_link": "https://chipper.cash/pay/CHI_ORD_xxxxx",
    "amount": 5000,
    "currency": "NGN",
    "status": "pending",
    "created_at": "2026-02-24T10:30:00Z",
    "expires_at": "2026-02-25T10:30:00Z"
  }
}
```

Share the `payment_link` with customers. They authenticate with Chipper and complete payment via the hosted page.

### Create a Payout

Send funds directly to a recipient's Chipper account or mobile money wallet.

```
POST /v1/payouts
```

**Body:**
```json
{
  "recipient_identifier": "+254701234567",
  "recipient_type": "phone",
  "amount": 2500,
  "currency": "KES",
  "reason": "Vendor payment",
  "reference": "PAYOUT_VND_001",
  "metadata": {
    "vendor_id": "vnd_123"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payout_id": "CHI_PAYOUT_xxxxx",
    "recipient_identifier": "+254701234567",
    "amount": 2500,
    "currency": "KES",
    "status": "pending",
    "estimated_delivery": "2026-02-24T11:00:00Z",
    "fees": 50,
    "net_amount": 2450,
    "reference": "PAYOUT_VND_001",
    "created_at": "2026-02-24T10:30:00Z"
  }
}
```

Payouts typically settle within minutes. Monitor via webhooks for `payout.completed` or `payout.failed` events.

### Get Exchange Rates

Retrieve current exchange rates between supported currency pairs for rate quotes.

```
GET /v1/rates?from=NGN&to=KES
```

**Response:**
```json
{
  "success": true,
  "data": {
    "from_currency": "NGN",
    "to_currency": "KES",
    "rate": 0.45,
    "timestamp": "2026-02-24T10:30:00Z",
    "rate_expires_in_seconds": 60
  }
}
```

Rates are valid for 60 seconds. Request fresh rates before each transaction.

### Retrieve Transaction Details

```
GET /v1/transactions/{transaction_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "transaction_id": "CHI_TXN_xxxxx",
    "type": "payment",
    "amount": 5000,
    "currency": "NGN",
    "status": "completed",
    "reference": "ORDER_12345_001",
    "created_at": "2026-02-24T10:30:00Z",
    "completed_at": "2026-02-24T10:31:15Z"
  }
}
```

### List Transactions

```
GET /v1/transactions?limit=20&offset=0&status=completed&start_date=2026-02-01&end_date=2026-02-28
```

**Query Parameters:**
- `limit` - Maximum 100 (default: 20)
- `offset` - For pagination
- `status` - Filter: `pending`, `completed`, `failed`
- `start_date`, `end_date` - ISO 8601 format
- `type` - Filter: `payment`, `payout`

## Webhooks

Chipper Cash sends webhooks for transaction events. Verify webhook signatures using HMAC-SHA256:

```javascript
const crypto = require('crypto');
const signature = req.headers['x-chipper-signature'];
const body = req.rawBody; // Raw request body, not parsed
const expectedSignature = crypto
  .createHmac('sha256', YOUR_CHIPPER_SECRET)
  .update(body)
  .digest('hex');

if (crypto.timingSafeEqual(signature, expectedSignature)) {
  // Valid webhook — process the event
} else {
  // Invalid signature — reject
  return res.status(401).json({ error: 'Invalid signature' });
}
```

**Common webhook events:**
- `payment.completed` - Payment received successfully
- `payment.failed` - Payment failed or was cancelled
- `payout.completed` - Payout delivered
- `payout.failed` - Payout delivery failed
- `payout.pending` - Payout initiated but not yet settled

**Webhook payload example:**
```json
{
  "event": "payment.completed",
  "timestamp": "2026-02-24T10:31:15Z",
  "data": {
    "order_id": "CHI_ORD_xxxxx",
    "amount": 5000,
    "currency": "NGN",
    "status": "completed",
    "reference": "ORDER_12345_001"
  }
}
```

Configure your webhook endpoint in the Chipper dashboard. Chipper will retry failed webhook deliveries with exponential backoff for up to 72 hours.

## Common Integration Patterns

### Accept Payment Checkout Flow

1. Customer clicks "Pay with Chipper" button on your checkout page
2. `POST /v1/orders` → receive `payment_link`
3. Redirect customer to payment link
4. Customer authenticates with Chipper and completes payment
5. Chipper redirects customer to your `redirect_url` with payment status
6. Listen for `payment.completed` webhook to confirm and fulfill order
7. Update order status in your database

### Marketplace Seller Payouts

1. Buyers purchase products from sellers on your platform
2. Collect payments in buyer's local currency via Network API
3. Accumulate funds in your merchant wallet
4. At settlement time, `POST /v1/payouts` to each seller in their country
5. Track payout status with `GET /v1/transactions/{id}`
6. Listen for `payout.completed` webhook to notify sellers

### Remittance Service

1. User enters recipient phone number and amount to send
2. `GET /v1/rates?from=NGN&to=KES` → show recipient receives amount after fees
3. Show fee breakdown before confirmation
4. `POST /v1/payouts` → initiate payout
5. Listen for `payout.completed` webhook
6. Notify sender of successful delivery

### Multi-Currency Settlement

1. Accept payments in multiple currencies via Network API
2. Store the exchange rate used at transaction time
3. For payouts, `GET /v1/rates` → quote current rate
4. Execute payouts in recipient's preferred currency
5. Reconcile fx gains/losses in accounting

## Error Handling

Chipper Cash returns standardized error responses:

```json
{
  "success": false,
  "error": {
    "code": "INVALID_AMOUNT",
    "message": "Amount must be greater than minimum transaction limit"
  }
}
```

**Common error codes:**
- **401 Unauthorized** - Invalid or missing API credentials
- **400 Bad Request** - Validation error (invalid amount, unsupported currency, invalid phone format)
- **404 Not Found** - Resource not found (order ID, payout ID, etc.)
- **429 Too Many Requests** - Rate limited; back off and retry after 60 seconds
- **503 Service Unavailable** - Temporary service issue; retry with exponential backoff

**Validation Rules:**
- Phone numbers must include country code (e.g., +254 for Kenya, +234 for Nigeria)
- Amounts must be positive numbers with up to 2 decimal places
- Currency codes must be valid ISO 4217 codes for supported countries
- References should be unique per merchant to prevent duplicate processing

## Supported Countries & Currencies

Chipper Cash supports direct operations in:

| Country | Currency | Code |
|---------|----------|------|
| Nigeria | Nigerian Naira | NGN |
| Kenya | Kenyan Shilling | KES |
| Uganda | Ugandan Shilling | UGX |
| Ghana | Ghanaian Cedi | GHS |
| Tanzania | Tanzanian Shilling | TZS |
| Rwanda | Rwandan Franc | RWF |
| South Africa | South African Rand | ZAR |

Plus access to 40+ additional markets through partnerships and licensed corridors.

## Transaction Limits & Fees

**Transaction Limits:**
- Minimum transaction: Varies by corridor, typically $1 USD equivalent
- Maximum transaction: Depends on account type and regulatory requirements
- Check your merchant dashboard for current limits

**Fees:**
- Vary by transaction type, corridor (currency pair), and amount
- Always displayed in API responses via the `fees` field
- Display fees to users before payment/payout confirmation
- Fees are deducted from payout amounts (not added to customer payment)

## Important Notes & Gotchas

**Consumer App vs. Network API:**
- Chipper Cash app is primarily for peer-to-peer transfers between individuals
- The Network API is the merchant/business product for accepting payments and sending payouts
- Don't confuse consumer app functionality with API capabilities
- P2P transfers via the app use a different system than the Network API

**Merchant Approval Required:**
- Unlike some payment APIs, Network API access requires merchant vetting
- Signup is not instant — expect 1-5 business days for approval
- Have your business documentation ready (registration, proof of address, etc.)
- High-risk categories may require additional underwriting

**Production Access:**
- You must thoroughly test in the sandbox environment first
- Sandbox credentials and production credentials are completely separate
- Production access requires explicit request and may require additional security review
- Do not use production credentials in development environments

**Idempotency:**
- Use unique `reference` values to prevent duplicate transactions
- Chipper will reject duplicate references within a time window
- If a request times out, retry with the same reference to safely check status

**Rate Limits:**
- API has rate limiting (exact limits vary by account tier)
- Implement exponential backoff for retries
- Store exchange rates locally for 60 seconds to reduce API calls

**Webhook Reliability:**
- Always implement your own transaction status polling as backup
- Webhooks may be delayed or delivered out of order
- Store webhook delivery IDs to prevent processing duplicates
- Implement idempotent webhook handlers

**Currency Conversion:**
- Exchange rates are locked at transaction initiation time
- The rate shown in responses is valid for 60 seconds only
- FX costs and spreads are included in the rate
- No guarantee of specific spreads or rates beyond the 60-second window

## Useful Links

- **Main API Page:** https://www.chippercash.com/api
- **Chipper for Business:** https://enterprise.chippercash.com/
- **Help Center - Network API:** https://support.chippercash.com/en/collections/3354823-network-api
- **Help Center - API Access FAQ:** https://support.chippercash.com/en/articles/6019594-what-is-the-chipper-cash-network-api-faqs
- **Help Center - Getting Started:** https://support.chippercash.com/en/articles/6004640-as-a-merchant-how-do-i-get-access-to-the-chipper-cash-payment-api
- **Postman Collection:** https://documenter.getpostman.com/view/18693070/UVz1Psag
- **GitHub:** https://github.com/ChipperCash
- **Support:** https://support.chippercash.com/en/
