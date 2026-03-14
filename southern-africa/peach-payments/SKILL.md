---
name: peach-payments
description: "Integrate with Peach Payments for card, 3DS, and tokenization payments in Africa. Use this skill whenever the user wants to accept card payments with 3D Secure authentication, implement tokenization for recurring payments, process transactions via Peach Payments, manage payment registrations, or work with Peach's API in any way. Also trigger when the user mentions 'Peach Payments', 'card payments Africa', '3D Secure', 'payment tokenization', or needs multi-currency payment processing."
---

# Peach Payments Integration Skill

Peach Payments is a PCI-compliant payment gateway serving Africa, enabling secure card payments, 3D Secure authentication, and tokenization for recurring billing. Peach offers a modern REST API with multi-currency support, server-to-server integration, and integrated fraud protection across major African currencies.

## When to use this skill

You're building a payment system requiring secure card acceptance with 3D Secure fraud protection and recurring billing capability — an e-commerce platform, SaaS subscription service, marketplace, or fintech application. Peach Payments handles direct card processing, 3D Secure verification, card tokenization for recurring payments, and multi-currency transactions across Africa (ZAR, NGN, GHS, KES, and USD).

## Authentication

All Peach Payments API requests require Bearer token authentication via the Authorization header:

```
Authorization: Bearer YOUR_SECRET_KEY
```

The Entity ID header identifies your merchant account and determines which currencies and payment methods are available:

```
EntityId: YOUR_ENTITY_ID
```

Store credentials in environment variables:
```bash
PEACH_SECRET_KEY=your_bearer_token
PEACH_ENTITY_ID=your_entity_id
```

Never hardcode credentials. Retrieve API credentials from your Peach Payments Dashboard under the Payments API section.

**Base URL (Production):** `https://api.peachpayments.com/v1`

**Base URL (Sandbox Testing):** Refer to your dashboard for the sandbox API base URL. Use sandbox for testing before going live.

## Core API Reference

### 1. Create a Registration (Tokenize Card)

Store a customer's card securely without exposing the card number. Returns a registration ID for future charges.

```
POST /registrations
```

**Headers:**
```
Authorization: Bearer YOUR_SECRET_KEY
EntityId: YOUR_ENTITY_ID
```

**Request Body:**
```json
{
  "paymentBrand": "VISA",
  "holder": "John Doe",
  "number": "4111111111111111",
  "expiryMonth": "12",
  "expiryYear": "2027",
  "cvv": "123"
}
```

**Successful Response (result code 000.200.100):**
```json
{
  "id": "8a8294185c18ab2c015c48cc91e900a0",
  "paymentBrand": "VISA",
  "holder": "John Doe",
  "number": "411111XXXXXX1111",
  "expiryMonth": "12",
  "expiryYear": "2027",
  "resultCode": "000.200.100",
  "resultDescription": "Registration successfully created"
}
```

Save the `id` field as the registration token. The card is now tokenized and stored securely at Peach; you never retain the full card number.

### 2. Create a Payment

Charge a card directly using card details or a stored registration. All amounts use minor units (e.g., ZAR cents).

```
POST /payments
```

**Headers:**
```
Authorization: Bearer YOUR_SECRET_KEY
EntityId: YOUR_ENTITY_ID
```

#### Direct Card Payment with 3D Secure:

```json
{
  "amount": "1999",
  "currency": "ZAR",
  "paymentType": "DB",
  "paymentBrand": "VISA",
  "holder": "John Doe",
  "number": "4111111111111111",
  "expiryMonth": "12",
  "expiryYear": "2027",
  "cvv": "123",
  "merchantTransactionId": "ORDER-12345",
  "customer.email": "john@example.com",
  "customer.phone": "+27123456789",
  "3ds.channel": "ECOMMERCE",
  "3ds.initiator": "MERCHANT"
}
```

#### Charge a Stored Registration:

```json
{
  "amount": "1999",
  "currency": "ZAR",
  "paymentType": "DB",
  "registrationId": "8a8294185c18ab2c015c48cc91e900a0",
  "merchantTransactionId": "ORDER-67890",
  "recurringType": "INITIAL"
}
```

**Key Parameters:**
- `amount`: Integer in minor units (cents for ZAR, kobo for NGN, pesewas for GHS, cents for KES/USD). R19.99 = 1999.
- `paymentType`: **DB** (debit/charge) or **RF** (refund)
- `merchantTransactionId`: Unique transaction identifier; Peach de-duplicates based on this field (idempotency)
- `recurringType`: **INITIAL** for first transaction, **REPEATED** for subsequent recurring charges (skips 3DS)
- `3ds.channel`: **ECOMMERCE** for card-not-present transactions
- `3ds.initiator`: **MERCHANT** for merchant-initiated transactions

**Successful Response (result code 000.100.110):**
```json
{
  "id": "8a8294195c18ab2c015c48cc91e900b1",
  "paymentType": "DB",
  "paymentBrand": "VISA",
  "amount": "1999",
  "currency": "ZAR",
  "merchantTransactionId": "ORDER-12345",
  "registrationId": "8a8294185c18ab2c015c48cc91e900a0",
  "resultCode": "000.100.110",
  "resultDescription": "Request successfully processed",
  "risk.fraudScore": 3,
  "timestamp": "2025-02-23T10:30:00Z"
}
```

**3D Secure Challenge Response (result code 100.380.401):**

If the card issuer requires 3D Secure authentication, you'll receive:

```json
{
  "id": "8a8294195c18ab2c015c48cc91e900b1",
  "resultCode": "100.380.401",
  "resultDescription": "3D Secure authentication required",
  "3ds.eci": "02",
  "redirect.url": "https://acs.example.com/auth?session=..."
}
```

**Action:** Redirect the customer's browser to the `redirect.url` for authentication. After successful authentication, Peach will notify you via webhook or you can poll the payment status.

### 3. Get Payment Details

Retrieve the current status and full details of a payment.

```
GET /payments/{payment_id}
```

**Headers:**
```
Authorization: Bearer YOUR_SECRET_KEY
EntityId: YOUR_ENTITY_ID
```

**Response:**
```json
{
  "id": "8a8294195c18ab2c015c48cc91e900b1",
  "paymentType": "DB",
  "paymentBrand": "VISA",
  "amount": "1999",
  "currency": "ZAR",
  "merchantTransactionId": "ORDER-12345",
  "registrationId": "8a8294185c18ab2c015c48cc91e900a0",
  "status": "CAPTURED",
  "resultCode": "000.100.110",
  "resultDescription": "Request successfully processed",
  "customer.email": "john@example.com",
  "3ds.eci": "05",
  "3ds.cavvAlgorithm": "3",
  "timestamp": "2025-02-23T10:30:00Z",
  "risk.fraudScore": 3
}
```

**Status Values:** `INITIAL`, `PENDING`, `CAPTURED`, `VOIDED`, `CHARGEBACK`, `REFUNDED`, `FAILED`.

### 4. Create Refund

Refund a previously captured payment in full or partially.

```
POST /payments/{payment_id}/refunds
```

**Headers:**
```
Authorization: Bearer YOUR_SECRET_KEY
EntityId: YOUR_ENTITY_ID
```

**Request Body (Partial Refund):**
```json
{
  "amount": "999",
  "currency": "ZAR",
  "merchantTransactionId": "REFUND-12345"
}
```

Omit `amount` to refund the entire original payment.

**Response (result code 000.100.110):**
```json
{
  "id": "8a8294195c18ab2c015c48cc91e900c2",
  "parentId": "8a8294195c18ab2c015c48cc91e900b1",
  "paymentType": "RF",
  "amount": "999",
  "currency": "ZAR",
  "merchantTransactionId": "REFUND-12345",
  "resultCode": "000.100.110",
  "resultDescription": "Request successfully processed",
  "timestamp": "2025-02-23T10:35:00Z"
}
```

### 5. Charge a Stored Registration (Recurring)

Initiate a recurring payment from a previously tokenized registration.

```
POST /registrations/{registration_id}/payments
```

**Headers:**
```
Authorization: Bearer YOUR_SECRET_KEY
EntityId: YOUR_ENTITY_ID
```

**Request Body:**
```json
{
  "amount": "1999",
  "currency": "ZAR",
  "paymentType": "DB",
  "merchantTransactionId": "RECURRING-67890",
  "recurringType": "REPEATED"
}
```

**Response:**
```json
{
  "id": "8a8294195c18ab2c015c48cc91e900d3",
  "registrationId": "8a8294185c18ab2c015c48cc91e900a0",
  "paymentType": "DB",
  "amount": "1999",
  "currency": "ZAR",
  "merchantTransactionId": "RECURRING-67890",
  "resultCode": "000.100.110",
  "resultDescription": "Request successfully processed",
  "status": "CAPTURED",
  "recurringType": "REPEATED",
  "timestamp": "2025-02-23T10:45:00Z"
}
```

## Webhooks

Peach Payments sends webhooks for payment state changes. Configure a POST endpoint in your dashboard to receive these events.

**Webhook Events:**
- `payment.created` — New payment initiated
- `payment.succeeded` — Payment captured successfully
- `payment.failed` — Payment declined or failed
- `payment.chargeback` — Chargeback initiated by cardholder
- `registration.created` — Card tokenized and stored
- `refund.succeeded` — Refund processed successfully
- `refund.failed` — Refund failed

**Webhook Payload:**
```json
{
  "id": "webhook_evt_123",
  "event": "payment.succeeded",
  "payment": {
    "id": "8a8294195c18ab2c015c48cc91e900b1",
    "paymentType": "DB",
    "amount": "1999",
    "currency": "ZAR",
    "status": "CAPTURED",
    "merchantTransactionId": "ORDER-12345",
    "registrationId": "8a8294185c18ab2c015c48cc91e900a0"
  },
  "timestamp": "2025-02-23T10:30:00Z"
}
```

**Signature Verification:**

Peach signs all webhooks using HMAC-SHA256. The signature is in the `x-webhook-signature` header.

**Verification Steps:**

1. Extract headers: `x-webhook-timestamp` and `x-webhook-signature`
2. Construct the message: `${timestamp}.${url}.${payload}` where `url` is the webhook endpoint URL and `payload` is the raw request body
3. Calculate HMAC: `crypto.createHmac('sha256', YOUR_SECRET_KEY).update(message).digest('hex')`
4. Compare calculated signature with the received `x-webhook-signature`

**Node.js Example:**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(secret, timestamp, url, payload, signature) {
  const message = `${timestamp}.${url}.${payload}`;
  const calculated = crypto.createHmac('sha256', secret).update(message).digest('hex');
  return calculated === signature;
}
```

Always verify webhook signatures to ensure authenticity and prevent replay attacks.

## Common Integration Patterns

### Pattern 1: E-Commerce Checkout Flow

Process one-time card payments with 3D Secure protection.

```
1. Customer enters card details on your checkout form
2. POST /payments → submit card details + 3DS parameters
3. Check resultCode:
   - 000.100.110 → Payment succeeded, fulfill order
   - 100.380.401 → 3DS required, redirect customer to issuer URL
   - 800.100.199+ → Payment failed, show error, retry allowed
4. GET /payments/{id} → confirm final status after 3DS challenge
5. Fulfill order and send confirmation email
```

**Key Points:**
- Always enable 3DS for card-not-present transactions
- Use `merchantTransactionId` for idempotency (prevent double charges if requests retry)
- Handle soft declines (result code 200-299) with retry logic; hard declines (300-399) are permanent

### Pattern 2: Recurring Subscription Billing

Charge customers automatically on a schedule using stored cards.

```
1. Signup: POST /registrations → tokenize card once
2. Store registrationId in your database linked to customer account
3. Billing cycle:
   - POST /registrations/{id}/payments with recurringType=REPEATED
   - Set up retry logic for failures (exponential backoff, max 3 attempts)
4. Handle webhooks:
   - payment.succeeded → Update subscription status, send receipt
   - payment.failed → Retry later or notify customer to update card
5. Logout/Cancellation: Store registration is retained; stop scheduling payments
```

**Important:** Subsequent charges with `recurringType=REPEATED` may skip 3DS if enabled by Peach for your account. Contact Peach support to confirm 3DS bypass availability for recurring payments.

### Pattern 3: Marketplace with Buyer Payments

Accept payments from buyers; settle seller funds through separate banking integration.

```
1. Buyer checkout: POST /payments → capture buyer card payment immediately
2. Order created with seller assignment
3. Store payment ID and seller information in your database
4. Payout scheduler (daily/weekly):
   - Calculate seller balances from successful payments
   - Use separate banking API (e.g., EFT, ACH) to transfer funds to seller accounts
   - Update seller account with payout status
5. Webhooks: Monitor payment.chargeback to handle disputes and adjust seller balance
6. Reconciliation: Use GET /payments/{id} to verify payment status before finalizing payouts
```

### Pattern 4: Payment with Optional Tokenization

Allow customers to save a card for future one-click checkout.

```
1. Checkout form: "Save this card for next time?" checkbox
2. POST /payments → Submit card payment with 3DS
3. On success:
   - If checkbox=true: POST /registrations → tokenize the card
   - Save registrationId for future orders
4. Next order:
   - Offer "Pay with saved card" → POST /payments with registrationId
   - Customer skips entering card details (faster checkout)
```

## Error Handling

Peach Payments returns structured responses with result codes following the format `XXX.XXX.XXX`.

**Result Code Ranges:**

| Range | Category | Meaning | Action |
|-------|----------|---------|--------|
| **000.xxx.xxx** | Success | Transaction processed successfully | Fulfill order |
| **100.xxx.xxx** | Processing Error | Authorization or processing issue | Retry allowed; check logs |
| **200.xxx.xxx** | Soft Decline | Soft decline (issuer issue) | Retry with backoff |
| **300.xxx.xxx** | Hard Decline | Fraud/lost/stolen card | Do not retry; prompt customer |
| **800.xxx.xxx** | Validation Error | Invalid request format or merchant config | Fix request; check Entity ID |
| **900.xxx.xxx** | System Error | Peach system error | Retry with exponential backoff |

**Common Result Codes:**

| Code | Meaning | Action |
|------|---------|--------|
| `000.100.110` | Success — request processed | Fulfill order |
| `000.200.100` | Registration created successfully | Store registration ID |
| `100.380.401` | 3DS authentication required | Redirect to `redirect.url` |
| `100.100.101` | Invalid merchant configuration | Verify Entity ID; contact Peach support |
| `100.150.100` | CVV verification failed | Prompt customer to re-enter CVV |
| `200.100.101` | Issuer unavailable (soft decline) | Retry with backoff |
| `300.100.100` | Insufficient funds / hard decline | Show error; ask customer to use different card |
| `300.180.401` | Card declined (fraud) | Hard decline; do not retry |
| `800.100.199` | Authorization failed (invalid request) | Fix request parameters; check documentation |
| `800.100.200` | Bad merchant request (missing parameters) | Validate all required fields before sending |
| `800.120.101` | Currency not supported for Entity ID | Verify currency is enabled; check dashboard |

**Error Response Example:**
```json
{
  "id": "8a8294195c18ab2c015c48cc91e900d4",
  "resultCode": "800.100.199",
  "resultDescription": "Authorization failed"
}
```

**Recommended Retry Logic:**

- **Soft declines (200-299):** Exponential backoff; max 3 retries over 24 hours
- **Hard declines (300-399, 800.xxx.xxx):** No retry; show error to customer
- **System errors (900.xxx.xxx):** Exponential backoff; max 5 retries over 72 hours
- **3DS auth required (100.380.401):** Redirect customer; no automatic retry

## Important Implementation Notes

### 1. PCI Compliance
Never store raw card data in your database. Always use Peach's tokenization (registrations) for recurring charges. Card details should be transmitted directly to Peach APIs or use hosted payment forms. Your PCI scope is minimal when delegating card handling to Peach.

### 2. 3D Secure (3DS) & Liability Shift
3D Secure provides strong customer authentication and reduces fraud liability. Peach Payments supports 3DS 2.0, which performs risk-based authentication:
- Low-risk transactions may complete without customer challenge
- High-risk transactions prompt the customer to verify with their issuing bank

Enable 3DS for all card-not-present (e-commerce) transactions. Include `3ds.channel: "ECOMMERCE"` and `3ds.initiator: "MERCHANT"` in payment requests.

**Important:** For recurring payments (subsequent charges), some Peach merchant accounts allow 3DS bypass. Contact Peach support to determine if 3DS is mandatory or optional for your account's recurring transactions.

### 3. Amount Precision
Use integers for minor units to avoid floating-point rounding errors:
- ZAR: 1999 = R19.99 (2 decimal places)
- NGN: 50000 = ₦500.00 (2 decimal places)
- GHS: 2499 = GHS 24.99 (2 decimal places)
- KES: 10000 = KES 100.00 (2 decimal places)
- USD: 2999 = $29.99 (2 decimal places)

### 4. Idempotency
Use `merchantTransactionId` (your unique transaction reference) to prevent duplicate charges. If your API client retries a request, Peach will de-duplicate based on this field. Use a UUID or unique order number; this field is required.

### 5. Webhook Validation
Always verify webhook signatures using HMAC-SHA256 to ensure authenticity. Process webhooks asynchronously; respond with HTTP 200 within 30 seconds and handle the event in a background queue. Implement webhook retry logic — Peach may resend webhooks if your endpoint returns a non-2xx status code.

### 6. Soft vs Hard Declines
- **Soft declines (200-299):** Issuer issue (unavailable, temporary restriction). Safe to retry with exponential backoff.
- **Hard declines (300-399):** Fraud detection, lost/stolen card, insufficient funds. Do not retry; ask customer to use a different card.

### 7. Entity ID & Currency Support
Your Entity ID determines which currencies and payment methods are enabled. Verify your Entity ID in the Peach Dashboard and ensure the currency is active for your merchant account. Unsupported currencies will return `800.120.101`.

### 8. Testing in Sandbox
Use sandbox credentials for integration testing:
- Retrieve sandbox API base URL from your dashboard
- Use test card numbers (e.g., 4111111111111111 for Visa)
- 3D Secure challenges are simulated; test redirection logic
- Webhooks are sent to your sandbox webhook endpoint (configure in dashboard)

Transition to production by:
1. Updating the API base URL to `https://api.peachpayments.com/v1`
2. Using production Bearer token and Entity ID
3. Testing with real payment methods (small amounts) to verify live environment
4. Enabling webhook verification in production

### 9. Recurring Payment Merchants
If your Peach account is configured for recurring payments (subscriptions, standing orders):
- Initial payment: `recurringType: "INITIAL"` (may require 3DS)
- Subsequent charges: `recurringType: "REPEATED"` (3DS may be bypassed by account configuration)
- Contact Peach support to confirm your account's 3DS requirements for recurring transactions

## Currency Support

Peach Payments supports the following African currencies:

| Currency | Code | Decimal Places | Minor Unit |
|----------|------|-----------------|------------|
| South African Rand | ZAR | 2 | Cents |
| Nigerian Naira | NGN | 2 | Kobo |
| Ghanaian Cedi | GHS | 2 | Pesewas |
| Kenyan Shilling | KES | 2 | Cents |
| US Dollar | USD | 2 | Cents |

All amounts must be submitted as integers in minor units (e.g., ZAR 19.99 = 1999).

## Useful Links

- **Official Documentation:** [https://developer.peachpayments.com/](https://developer.peachpayments.com/)
- **Payments API Overview:** [https://developer.peachpayments.com/docs/payments-api-overview](https://developer.peachpayments.com/docs/payments-api-overview)
- **API Reference:** [https://developer.peachpayments.com/reference/payment](https://developer.peachpayments.com/reference/payment)
- **Merchant Dashboard:** [https://merchant.peachpayments.com](https://merchant.peachpayments.com)
- **Support Center - Result Codes:** [https://support.peachpayments.com/support/solutions/articles/47001098650-most-relevant-result-codes-for-transaction-processing](https://support.peachpayments.com/support/solutions/articles/47001098650-most-relevant-result-codes-for-transaction-processing)
- **Support Center - Test Cards:** [https://support.peachpayments.com/support/solutions/articles/47001098775-test-cards-to-use-for-sandbox-testing](https://support.peachpayments.com/support/solutions/articles/47001098775-test-cards-to-use-for-sandbox-testing)
- **Postman Collection:** [https://www.postman.com/peachpayments/peach-payments-public-workspace/overview](https://www.postman.com/peachpayments/peach-payments-public-workspace/overview)
- **3D Secure FAQ:** [https://support.peachpayments.com/support/solutions/articles/47001098824-3d-secure-3ds-faq-s](https://support.peachpayments.com/support/solutions/articles/47001098824-3d-secure-3ds-faq-s)
