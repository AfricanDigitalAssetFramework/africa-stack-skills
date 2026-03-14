---
name: paymob
description: "Integrate with the Paymob (Accept) payment API for Egyptian payments. Use this skill whenever the user wants to accept payments in Egypt, integrate Paymob into their app, handle EGP transactions, create payment orders, generate payment keys, process card and wallet payments, manage refunds, or work with Paymob's API in any way. Also trigger when the user mentions 'Paymob', 'Accept Egypt', 'Egyptian payments', 'EGP checkout', 'mobile wallet payments', 'kiosk payments', 'cash collection', or needs payment solutions for Egypt."
---

# Paymob (Accept) Integration Skill

Paymob, operating as Accept, is Egypt's leading payment gateway, processing payments in EGP (Egyptian Pound) and supporting debit/credit cards, mobile wallets (Vodafone, Etisalat, Orange), bank transfers, kiosk payments, and various BNPL providers. It powers online and offline payments for businesses across Egypt with a comprehensive REST API.

## When to use this skill

You're building something that needs to accept payments in Egypt — an e-commerce checkout, a subscription service, a marketplace, digital services, invoicing platforms, or anything that touches money movement in Egypt. Paymob handles card payments, mobile wallet integration, bank transfers, kiosk payments, cash collection, and provides powerful order management capabilities.

## Authentication

Paymob uses a two-step authentication flow. First, you authenticate with your API key to get an auth token (valid for 1 hour), then use that token for subsequent API calls.

**Base URL:** `https://accept.paymob.com/api`

### How it works

1. Exchange your API key for an authentication token
2. Use the token in all subsequent API requests
3. Tokens expire after 60 minutes — request a new one when needed

Store your API key securely in environment variables like `PAYMOB_API_KEY`. Never hardcode API keys in your source code.

## Core API Reference

### Get Authentication Token

Exchange your API key for an auth token that's valid for one hour. This is the first step in every payment flow.

**Endpoint:**
```
POST /auth/tokens
```

**Request:**
```json
{
  "api_key": "your_api_key_here"
}
```

**Response:**
```json
{
  "profile": {
    "id": 12345,
    "created_at": "2023-01-01T00:00:00Z",
    "coins": 0,
    "staging": false
  },
  "token": "nJWzxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

Store the `token` value for use in subsequent requests. Implement caching to avoid requesting new tokens unnecessarily.

### Create an Order

Initialize an order for payment processing. This order holds transaction details until payment is processed.

**Endpoint:**
```
POST /ecommerce/orders
```

**Headers:**
```
Authorization: Bearer {token}
Content-Type: application/json
```

**Request:**
```json
{
  "auth_token": "{token}",
  "delivery_needed": false,
  "currency": "EGP",
  "amount_cents": 500000,
  "merchant_order_id": "ORD-2025-12345",
  "items": [
    {
      "name": "Premium Subscription",
      "amount_cents": 500000,
      "description": "1 month access",
      "quantity": "1"
    }
  ]
}
```

**Important:** All amounts in the API are in **cents**, not pounds. For example, 5000 EGP = 500,000 cents.

**Response:**
```json
{
  "id": 987654321,
  "merchant_order_id": "ORD-2025-12345",
  "amount_cents": 500000,
  "currency": "EGP",
  "paid": false,
  "pending": false,
  "delivery_status": null,
  "created_at": "2025-02-23T10:00:00Z"
}
```

Save the `id` from the response — you'll need it to generate a payment key.

### Generate Payment Key

Create a secure payment key that customers use to access the checkout. The payment key is valid for 30 minutes.

**Endpoint:**
```
POST /acceptance/payment_keys
```

**Headers:**
```
Authorization: Bearer {token}
Content-Type: application/json
```

**Request:**
```json
{
  "auth_token": "{token}",
  "order_id": 987654321,
  "billing_data": {
    "apartment": "Apt 5",
    "email": "customer@email.com",
    "floor": "2",
    "first_name": "Ahmed",
    "street": "123 Main St",
    "building": "20",
    "phone_number": "+201234567890",
    "postal_code": "11111",
    "city": "Cairo",
    "country": "EG",
    "last_name": "Hassan",
    "state": "Cairo"
  },
  "currency": "EGP",
  "integration_id": 123456,
  "amount_cents": 500000
}
```

**Key fields:**
- `integration_id`: The ID for your payment method (cards, wallet, etc.). Found in Paymob dashboard under Developers > Payment Integrations.
- `billing_data`: Customer information for the payment
- `amount_cents`: Must match the order amount exactly

**Response:**
```json
{
  "profile": {
    "id": 12345
  },
  "redirect_url": "https://accept.paymob.com/api/acceptance/iframes/...",
  "token": "ZXhhbXBsZV9wYXltZW50X2tleQ=="
}
```

Options for using the payment key:
1. **Redirect flow**: Send customer to `redirect_url` where they enter payment details
2. **Iframe flow**: Embed the iframe using the token in your own checkout page
3. **API flow**: Use the token with client-side or server-side payment processing

### Retrieve Order Details

Get the current state of an order and all associated payments.

**Endpoint:**
```
GET /ecommerce/orders/{order_id}
```

**Headers:**
```
Authorization: Bearer {token}
```

**Response:**
```json
{
  "id": 987654321,
  "merchant_order_id": "ORD-2025-12345",
  "amount_cents": 500000,
  "currency": "EGP",
  "paid": true,
  "pending": false,
  "payments": [
    {
      "id": 1234567890,
      "amount_cents": 500000,
      "currency": "EGP",
      "is_auth": true,
      "pending": false,
      "paid": true,
      "created_at": "2025-02-23T10:05:00Z"
    }
  ],
  "created_at": "2025-02-23T10:00:00Z"
}
```

### Refund a Payment

Issue a refund for a completed payment.

**Endpoint:**
```
POST /acceptance/payments/{payment_id}/refund
```

**Headers:**
```
Authorization: Bearer {token}
Content-Type: application/json
```

**Request:**
```json
{
  "auth_token": "{token}"
}
```

**Response:**
```json
{
  "id": 1234567890,
  "order": {
    "id": 987654321,
    "merchant_order_id": "ORD-2025-12345"
  },
  "amount_cents": 500000,
  "currency": "EGP",
  "is_refund": true,
  "pending": false,
  "created_at": "2025-02-23T10:10:00Z"
}
```

## Webhooks

Paymob sends webhooks to your endpoint whenever payment events occur. Configure your webhook URL in the Paymob dashboard (Settings > Webhook) and verify incoming requests with HMAC-SHA512 signature verification.

### Webhook Events

Paymob sends webhooks for various transaction events:
- `charge.success` — Payment completed successfully
- `charge.failed` — Payment attempt failed
- `charge.refunded` — Refund processed
- `payment.approved` — Payment approved by bank
- `payment.denied` — Payment denied by bank

### Webhook Signature Verification

Always verify the webhook signature to ensure the request came from Paymob. Use HMAC-SHA512 with your webhook secret key.

**Node.js Example:**
```javascript
const crypto = require('crypto');

// req.body contains the webhook payload
const hmacObject = req.body.obj;
const webhookSecret = process.env.PAYMOB_WEBHOOK_SECRET;

// Create HMAC signature
const hash = crypto
  .createHmac('sha512', webhookSecret)
  .update(JSON.stringify(hmacObject))
  .digest('hex');

// Compare signatures
if (hash === req.body.hmac) {
  // Webhook is valid — process the payment
  console.log('Valid webhook from Paymob');
  console.log('Transaction ID:', hmacObject.id);
  console.log('Order ID:', hmacObject.order.id);
  console.log('Amount:', hmacObject.amount_cents);
  console.log('Status:', hmacObject.paid ? 'Success' : 'Failed');
} else {
  // Invalid signature — reject the webhook
  console.error('Invalid webhook signature');
  return res.status(401).send('Unauthorized');
}
```

**Python Example:**
```python
import hmac
import hashlib
import json

# Extract webhook data
hmac_object = request.json['obj']
webhook_hmac = request.json['hmac']
webhook_secret = os.getenv('PAYMOB_WEBHOOK_SECRET')

# Create HMAC signature
calculated_hash = hmac.new(
    webhook_secret.encode(),
    json.dumps(hmac_object).encode(),
    hashlib.sha512
).hexdigest()

# Verify signature
if calculated_hash == webhook_hmac:
    # Valid webhook
    transaction_id = hmac_object['id']
    order_id = hmac_object['order']['id']
    amount = hmac_object['amount_cents']
    paid = hmac_object['paid']
else:
    # Invalid webhook
    return {'error': 'Invalid signature'}, 401
```

### Webhook Payload Example

```json
{
  "obj": {
    "id": 1234567890,
    "amount_cents": 500000,
    "currency": "EGP",
    "paid": true,
    "pending": false,
    "is_refund": false,
    "is_auth": true,
    "order": {
      "id": 987654321,
      "merchant_order_id": "ORD-2025-12345"
    },
    "created_at": "2025-02-23T10:05:00Z"
  },
  "hmac": "abc123..."
}
```

## Common Integration Patterns

### Standard E-Commerce Checkout Flow

Complete flow for processing card and wallet payments:

```
1. POST /auth/tokens
   ↓ Get auth token
2. POST /ecommerce/orders
   ↓ Create order with amount and items
3. POST /acceptance/payment_keys
   ↓ Generate payment key
4. Redirect customer to checkout URL
   ↓ Customer enters payment details
5. Listen for webhook from Paymob
   ↓ Verify HMAC signature
6. Update order status in your database
   ↓ Mark as paid/failed
7. Fulfill order or notify customer
```

### Card Payment Integration

For accepting credit and debit cards (Visa, Mastercard, Amex):

1. Set `integration_id` to your card payment integration ID from dashboard
2. Include full `billing_data` with customer address and phone
3. Use redirect flow or iframe for customer to enter card details
4. Verify payment via webhook before fulfilling

```json
{
  "integration_id": 123456,
  "billing_data": {
    "first_name": "Ahmed",
    "last_name": "Hassan",
    "email": "customer@example.com",
    "phone_number": "+201234567890",
    "country": "EG",
    "city": "Cairo"
  }
}
```

### Mobile Wallet Payment Integration

For accepting mobile wallet payments (Vodafone Cash, Etisalat Money, Orange Money):

1. Set `integration_id` to your wallet integration ID
2. Wallet payments require phone number in `billing_data`
3. Customer completes payment through their mobile wallet app
4. Confirmation comes via webhook

```json
{
  "integration_id": 789012,
  "billing_data": {
    "phone_number": "+201234567890"
  }
}
```

### Kiosk Payment Integration

For kiosk payments (Aman, Masary):

1. Generate payment key with kiosk integration ID
2. Provide customer with unique kiosk reference code
3. Customer pays at kiosk location
4. Payment confirmed via webhook when customer completes kiosk transaction

### Cash Collection Integration

For cash-on-delivery and cash collection:

1. Create order with `delivery_needed: true`
2. Use cash collection integration ID
3. Driver collects payment in cash
4. Webhook confirms when cash is collected

## Error Handling

Paymob returns errors with HTTP status codes and error details in the response.

### HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 400 | Bad Request | Check your request format and required fields |
| 401 | Unauthorized | Token expired or invalid — get a new auth token |
| 403 | Forbidden | Integration ID doesn't match or permission denied |
| 404 | Not Found | Order/payment/integration not found — check IDs |
| 500 | Server Error | Retry with exponential backoff (Paymob is experiencing issues) |

### Error Response Format

```json
{
  "detail": "Amount is required",
  "code": "VALIDATION_ERROR"
}
```

### Common Error Scenarios

**Invalid Token:**
```json
{
  "detail": "Token is invalid or expired"
}
```
Solution: Get a new auth token with `POST /auth/tokens`

**Order Not Found:**
```json
{
  "detail": "Order with ID 999999999 not found"
}
```
Solution: Verify the order ID matches the order you created

**Insufficient Amount:**
```json
{
  "detail": "Amount must be greater than zero"
}
```
Solution: Ensure `amount_cents` is a positive integer

**Invalid Integration ID:**
```json
{
  "detail": "Integration ID is not valid for this account"
}
```
Solution: Verify integration ID in dashboard under Developers > Payment Integrations

## Important Notes & Gotchas

### Amount Format

All amounts in the Paymob API are in **cents** (piasters), not pounds.
- 1 EGP = 100 cents
- To charge 500 EGP, use `amount_cents: 50000`
- Always use integers, never decimals

### Token Expiration

Auth tokens expire after 60 minutes. Options:
- Request a new token for each API call (simplest but slower)
- Cache tokens and refresh before expiration (recommended)
- Implement token pooling for high-volume integrations

### Integration IDs

Different payment methods require different integration IDs:
- Card payments: One integration ID
- Wallet payments: Separate integration ID
- Kiosk: Different integration ID
- Find all your integration IDs in Paymob dashboard: Developers > Payment Integrations

To support multiple payment methods, generate separate payment keys with different integration IDs and let customer choose their preferred method.

### Payment Key Validity

Payment keys are valid for only 30 minutes. If a customer's session times out, generate a new payment key before asking them to pay again.

### Iframe vs Redirect vs API

**Redirect Flow (Simplest)**
- Generate payment key
- Redirect customer to `redirect_url`
- Paymob hosts the checkout page
- Best for simple integrations

**Iframe Flow (More Control)**
- Generate payment key
- Embed Paymob iframe in your checkout page using the token
- Keeps customer on your site
- Requires iframe implementation
- Better UX for custom designs

**API Flow (Most Complex)**
- Handle payment directly through API
- Requires PCI compliance for card data
- Use only for mobile apps or special cases
- Not recommended for web applications

### Webhook Reliability

- Webhooks may arrive out of order
- Webhooks may arrive multiple times (implement idempotency)
- Always verify HMAC signature
- Always verify `amount_cents` matches your expected amount
- Store webhook data for audit trail

### Billing Data

Include as complete billing data as possible:
- Required: `first_name`, `last_name`, `email`, `phone_number`, `country`, `city`
- Recommended: `street`, `building`, `floor`, `apartment`, `postal_code`
- 3D Secure and fraud checks use this data

### Test vs Production

- Use sandbox API key from dashboard for testing
- Switch to live API key for production
- Test card numbers provided in Paymob dashboard
- HMAC verification works the same in test and production

## Useful Links

- **Official Paymob Developer Portal:** https://developers.paymob.com/
- **Egypt-specific Documentation:** https://developers.paymob.com/hub/egypt
- **API Documentation:** https://docs.paymob.com/
- **Checkout API Guide:** https://developers.paymob.com/guides/checkout-api
- **Transaction Webhooks:** https://docs.paymob.com/docs/transaction-webhooks
- **Test Cards:** https://docs.paymob.com/testing
- **Dashboard (Login):** https://accept.paymob.com/
- **Accept Payment Solutions:** https://paymob.com/en/online-payment
