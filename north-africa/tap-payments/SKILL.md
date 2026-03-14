---
name: Tap Payments
description: MENA payment gateway for accepting credit cards, mada, KNET, Apple Pay, Google Pay, BNPL and alternative payment methods in Egypt and the region. RESTful API with support for tokenization, recurring payments, and webhooks.
triggers:
  - "Tap Payments"
  - "Tap"
  - "Egypt payment gateway"
  - "MENA payments"
  - "EGP checkout"
  - "accept payments Egypt"
  - "mada payments"
  - "KNET payments"
  - "Tap API"
  - "tap.company"
---

# Tap Payments

Tap is the leading payment gateway for the Middle East and North Africa (MENA) region, with extensive support for local and international payment methods in Egypt and across the region. It provides developers with a robust RESTful API for accepting payments, managing customers, and handling recurring transactions.

## When to Use This Skill

Use Tap Payments when you need to:

- **Accept online payments in Egypt**: Credit/debit cards (Visa, Mastercard, American Express)
- **Support local Egyptian payment methods**: mada, KNET, Fawry, Meeza, Benefit, STC Pay
- **Offer modern payment options**: Apple Pay, Google Pay
- **Support Buy Now Pay Later (BNPL)**: Tabby, deema, and other BNPL providers
- **Build recurring/subscription billing**: Automate recurring charges with tokenized cards
- **Save customer payment methods**: Securely store and reuse card tokens
- **Process payments via API**: Direct server-to-server integration without PCI compliance burden
- **Authorize and capture payments**: Implement authorize-capture workflows for compliance
- **Handle webhooks and async operations**: Receive real-time payment status updates
- **Support multi-currency transactions**: Process payments in multiple currencies across MENA

## Authentication

Tap Payments uses **HTTP Bearer Token Authentication**. All requests must include your API secret key in the Authorization header.

### Authentication Method

```
Authorization: Bearer YOUR_SECRET_API_KEY
```

### Getting Your API Key

1. Log in to your Tap Payments dashboard
2. Navigate to API Keys or Developer Settings
3. Generate separate keys for Test and Live modes
4. Use the Test key for development/testing
5. Switch to Live key for production

### Code Examples

#### JavaScript/Node.js

```javascript
const axios = require('axios');

const tapApiKey = 'sk_test_abc123...'; // Your secret API key

const headers = {
  'Authorization': `Bearer ${tapApiKey}`,
  'Content-Type': 'application/json'
};

// Example API call
const response = await axios.post(
  'https://api.tap.company/v2/charges',
  {
    amount: 5000,
    currency: 'EGP',
    source: {
      id: 'src_card_123'
    }
  },
  { headers }
);
```

#### cURL

```bash
curl -X POST https://api.tap.company/v2/charges \
  -H "Authorization: Bearer sk_test_abc123..." \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 5000,
    "currency": "EGP",
    "source": {
      "id": "src_card_123"
    }
  }'
```

#### Python

```python
import requests

headers = {
    'Authorization': 'Bearer sk_test_abc123...',
    'Content-Type': 'application/json'
}

payload = {
    'amount': 5000,
    'currency': 'EGP',
    'source': {
        'id': 'src_card_123'
    }
}

response = requests.post(
    'https://api.tap.company/v2/charges',
    json=payload,
    headers=headers
)
```

## Base URL

```
https://api.tap.company/v2/
```

All endpoints are relative to this base URL. Always use HTTPS for security.

## Core API Reference

### Charges

Process direct payments via card tokens, saved cards, or customer sources.

#### Create a Charge

**POST** `/charges`

```json
{
  "amount": 5000,
  "currency": "EGP",
  "customer_id": "cus_123abc",
  "source": {
    "id": "src_card_456"
  },
  "reference": "ref_unique_order_id",
  "description": "Order #1001",
  "metadata": {
    "order_id": "1001",
    "user_email": "customer@example.com"
  }
}
```

**Response (Success - 200)**

```json
{
  "id": "chg_TS123ABC",
  "object": "charge",
  "amount": 5000,
  "currency": "EGP",
  "status": "CAPTURED",
  "reference": "ref_unique_order_id",
  "customer": {
    "id": "cus_123abc"
  },
  "source": {
    "id": "src_card_456",
    "object": "card",
    "brand": "VISA",
    "last_four": "4242"
  },
  "created": 1687856400
}
```

#### Retrieve a Charge

**GET** `/charges/{charge_id}`

Returns full details of a specific charge by ID.

#### Refund a Charge

**POST** `/charges/{charge_id}/refunds`

```json
{
  "amount": 5000,
  "reason": "customer_request"
}
```

#### List Charges

**GET** `/charges?limit=10&skip=0`

Returns paginated list of charges with optional filtering by date, status, or customer.

### Authorize

Authorize payments without immediate capture (useful for manual capture workflows).

#### Create an Authorization

**POST** `/authorizations`

```json
{
  "amount": 5000,
  "currency": "EGP",
  "customer_id": "cus_123abc",
  "source": {
    "id": "src_card_456"
  },
  "auto": {
    "type": "CAPTURE",
    "time_period": 0
  }
}
```

**Response (Success - 200)**

```json
{
  "id": "auth_TS123ABC",
  "object": "authorization",
  "amount": 5000,
  "currency": "EGP",
  "status": "AUTHORIZED",
  "auto": {
    "type": "CAPTURE",
    "time_period": 0
  },
  "created": 1687856400
}
```

#### Capture an Authorization

**POST** `/authorizations/{auth_id}/capture`

```json
{
  "amount": 5000
}
```

#### Void an Authorization

**POST** `/authorizations/{auth_id}/void`

Releases the authorization hold.

### Tokens

Create and manage reusable payment tokens for customers.

#### Create a Token

**POST** `/tokens`

```json
{
  "customer": {
    "id": "cus_123abc"
  },
  "card": {
    "id": "card_456def"
  },
  "type": "SUBSCRIPTION"
}
```

**Response (Success - 200)**

```json
{
  "id": "tok_TS123ABC",
  "object": "token",
  "type": "SUBSCRIPTION",
  "customer": {
    "id": "cus_123abc"
  },
  "card": {
    "id": "card_456def",
    "brand": "VISA",
    "last_four": "4242"
  },
  "created": 1687856400
}
```

#### Retrieve a Token

**GET** `/tokens/{token_id}`

Returns token details including associated customer and card.

### Customers

Manage customer profiles for saved payments and recurring billing.

#### Create a Customer

**POST** `/customers`

```json
{
  "first_name": "Ahmed",
  "last_name": "Hassan",
  "email": "ahmed@example.com",
  "phone": {
    "country_code": "+20",
    "number": "1012345678"
  },
  "metadata": {
    "user_id": "usr_123"
  }
}
```

**Response (Success - 200)**

```json
{
  "id": "cus_123abc",
  "object": "customer",
  "first_name": "Ahmed",
  "last_name": "Hassan",
  "email": "ahmed@example.com",
  "phone": {
    "country_code": "+20",
    "number": "1012345678"
  },
  "created": 1687856400
}
```

#### Retrieve a Customer

**GET** `/customers/{customer_id}`

Returns customer profile and associated payment methods.

#### Update a Customer

**PUT** `/customers/{customer_id}`

Update customer name, email, phone, or metadata.

#### List Customers

**GET** `/customers?limit=10&skip=0`

Returns paginated list of customers.

### Invoices

Create and manage invoices with automatic payment reminders and recurring billing.

#### Create an Invoice

**POST** `/invoices`

```json
{
  "customer": {
    "id": "cus_123abc"
  },
  "amount": 5000,
  "currency": "EGP",
  "due_date": "2024-12-31",
  "items": [
    {
      "description": "Monthly Subscription",
      "amount": 5000
    }
  ],
  "recurring": {
    "interval": "MONTH",
    "count": 12
  }
}
```

**Response (Success - 200)**

```json
{
  "id": "inv_TS123ABC",
  "object": "invoice",
  "customer": {
    "id": "cus_123abc"
  },
  "amount": 5000,
  "currency": "EGP",
  "status": "DRAFT",
  "due_date": "2024-12-31",
  "created": 1687856400
}
```

#### Send an Invoice

**POST** `/invoices/{invoice_id}/send`

Send invoice to customer via email.

#### Retrieve an Invoice

**GET** `/invoices/{invoice_id}`

Returns full invoice details.

#### List Invoices

**GET** `/invoices?limit=10&skip=0`

Returns paginated list of invoices.

## Webhooks

Tap Payments sends webhooks for important payment events. Your application must have an active HTTPS endpoint to receive and validate them.

### Webhook Structure

```json
{
  "id": "evt_TS123ABC",
  "object": "event",
  "type": "charge.captured",
  "created": 1687856400,
  "data": {
    "object": {
      "id": "chg_TS456DEF",
      "object": "charge",
      "amount": 5000,
      "currency": "EGP",
      "status": "CAPTURED",
      "reference": "ref_order_123"
    }
  }
}
```

### Common Webhook Events

| Event Type | Description |
|-----------|-------------|
| `charge.captured` | Charge successfully captured |
| `charge.failed` | Charge failed |
| `charge.refunded` | Charge refunded |
| `charge.voided` | Charge voided |
| `authorization.created` | Authorization created |
| `authorization.captured` | Authorization captured |
| `authorization.voided` | Authorization voided |
| `invoice.created` | Invoice created |
| `invoice.payment_succeeded` | Invoice payment succeeded |
| `invoice.payment_failed` | Invoice payment failed |
| `customer.created` | Customer created |
| `customer.updated` | Customer updated |

### Webhook Signature Verification

Tap includes a signature header for verification. Always validate the signature before processing the webhook.

#### Verification Steps

1. Extract the signature from the `X-Tap-Signature` header
2. Get the raw request body as a string
3. Hash the body using HMAC-SHA256 with your API secret key
4. Compare the computed hash with the signature

#### JavaScript/Node.js Example

```javascript
const crypto = require('crypto');

function verifyTapWebhook(rawBody, signature, apiSecret) {
  const computedSignature = crypto
    .createHmac('sha256', apiSecret)
    .update(rawBody)
    .digest('hex');

  return signature === computedSignature;
}

// Express middleware example
app.post('/webhook/tap', (req, res, next) => {
  const signature = req.headers['x-tap-signature'];
  const rawBody = req.rawBody; // Store raw body before parsing

  if (!verifyTapWebhook(rawBody, signature, process.env.TAP_API_SECRET)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook
  const event = req.body;
  handleWebhookEvent(event);
  res.json({ success: true });
});
```

#### Python Example

```python
import hmac
import hashlib

def verify_tap_webhook(raw_body, signature, api_secret):
    computed_signature = hmac.new(
        api_secret.encode(),
        raw_body.encode() if isinstance(raw_body, str) else raw_body,
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(signature, computed_signature)

# Flask example
@app.route('/webhook/tap', methods=['POST'])
def handle_webhook():
    signature = request.headers.get('X-Tap-Signature')
    raw_body = request.get_data(as_text=True)

    if not verify_tap_webhook(raw_body, signature, os.environ['TAP_API_SECRET']):
        return {'error': 'Invalid signature'}, 401

    event = request.json
    handle_webhook_event(event)
    return {'success': True}
```

### Setting Up Webhook Endpoints

1. Create an HTTPS endpoint that accepts POST requests
2. Validate the signature for security
3. Return HTTP 200 to confirm receipt
4. Process the event asynchronously (don't block the response)
5. Store the webhook in your database for audit logs
6. Configure the endpoint URL in your Tap dashboard

## Common Integration Patterns

### Pattern 1: Simple Payment with Card Source

```javascript
async function processPayment(amount, cardSourceId, customerId) {
  const response = await axios.post(
    'https://api.tap.company/v2/charges',
    {
      amount: amount * 100, // Convert to smallest unit (piasters for EGP, fils for Gulf currencies)
      currency: 'EGP',
      source: {
        id: cardSourceId
      },
      customer_id: customerId,
      reference: generateOrderId()
    },
    {
      headers: {
        'Authorization': `Bearer ${process.env.TAP_API_KEY}`,
        'Content-Type': 'application/json'
      }
    }
  );

  return response.data;
}
```

### Pattern 2: Saving a Card for Future Use

```javascript
async function saveCardForCustomer(customerId, cardToken, shouldChargeNow = false) {
  // First, create a customer if they don't exist
  let customer = await getOrCreateCustomer({
    id: customerId,
    email: 'customer@example.com'
  });

  // Save the card to the customer
  const card = await axios.post(
    'https://api.tap.company/v2/customers',
    {
      id: customer.id,
      cards: [cardToken]
    },
    { headers: authHeaders }
  );

  // Optionally charge immediately
  if (shouldChargeNow) {
    return await processPayment(5000, card.id, customer.id);
  }

  return card;
}
```

### Pattern 3: Recurring/Subscription Billing

```javascript
async function setupRecurringPayment(customerId, cardId, monthlyAmount) {
  // Create a token for recurring use
  const token = await axios.post(
    'https://api.tap.company/v2/tokens',
    {
      customer: { id: customerId },
      card: { id: cardId },
      type: 'SUBSCRIPTION'
    },
    { headers: authHeaders }
  );

  // Create an invoice with recurring settings
  const invoice = await axios.post(
    'https://api.tap.company/v2/invoices',
    {
      customer: { id: customerId },
      amount: monthlyAmount * 100,
      currency: 'EGP',
      due_date: getNextMonthDate(),
      recurring: {
        interval: 'MONTH',
        count: 12 // 12 months
      }
    },
    { headers: authHeaders }
  );

  // Send invoice to customer
  await axios.post(
    `https://api.tap.company/v2/invoices/${invoice.data.id}/send`,
    {},
    { headers: authHeaders }
  );

  return invoice.data;
}
```

### Pattern 4: Authorize and Capture (Manual Approval)

```javascript
async function authorizePayment(amount, cardSourceId, customerId) {
  // Step 1: Authorize the payment
  const auth = await axios.post(
    'https://api.tap.company/v2/authorizations',
    {
      amount: amount * 100,
      currency: 'EGP',
      source: { id: cardSourceId },
      customer_id: customerId,
      auto: {
        type: 'VOID',
        time_period: 432000 // 5 days in seconds
      }
    },
    { headers: authHeaders }
  );

  // Step 2: Store authorization ID in database
  await db.authorizations.save({
    auth_id: auth.data.id,
    order_id: orderId,
    status: 'AUTHORIZED'
  });

  return auth.data;
}

async function captureAuthorization(authId, amount) {
  // Step 3: Later, capture the authorized amount
  const capture = await axios.post(
    `https://api.tap.company/v2/authorizations/${authId}/capture`,
    {
      amount: amount * 100
    },
    { headers: authHeaders }
  );

  return capture.data;
}
```

### Pattern 5: Handling Webhook Events

```javascript
async function handleWebhookEvent(event) {
  switch (event.type) {
    case 'charge.captured':
      await handlePaymentSuccess(event.data.object);
      break;

    case 'charge.failed':
      await handlePaymentFailure(event.data.object);
      break;

    case 'charge.refunded':
      await handleRefund(event.data.object);
      break;

    case 'invoice.payment_succeeded':
      await handleRecurringPaymentSuccess(event.data.object);
      break;

    case 'invoice.payment_failed':
      await handleRecurringPaymentFailure(event.data.object);
      break;

    default:
      console.log(`Unhandled event type: ${event.type}`);
  }
}

async function handlePaymentSuccess(charge) {
  // Update order status
  await db.orders.updateOne(
    { reference: charge.reference },
    { status: 'PAID', charge_id: charge.id }
  );

  // Send confirmation email
  await sendOrderConfirmation(charge.reference);
}
```

## Error Handling

Tap Payments returns error responses with appropriate HTTP status codes and error details.

### HTTP Status Codes

| Status | Description |
|--------|-------------|
| 200 | Success |
| 400 | Bad Request (invalid parameters) |
| 401 | Unauthorized (invalid API key) |
| 402 | Payment Required (payment failed) |
| 404 | Not Found (resource doesn't exist) |
| 422 | Unprocessable Entity (validation error) |
| 429 | Too Many Requests (rate limited) |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

### Common Error Response

```json
{
  "errors": [
    {
      "code": "INVALID_AMOUNT",
      "description": "Amount must be greater than 0",
      "field": "amount"
    }
  ]
}
```

### Charge Response Codes

| Code | Status | Description |
|------|--------|-------------|
| 000 | CAPTURED | Successfully captured |
| 100 | INITIATED | Payment initiated |
| 200 | AUTHORIZED | Successfully authorized |
| 301 | ABANDONED | Payment abandoned |
| 302 | CANCELLED | Payment cancelled |
| 303 | DEFERRED | Payment deferred |
| 304 | EXPIRED | Payment expired |
| 401 | FAILED | Payment failed |
| 402 | FAILED_INVALID_PARAM | Invalid parameters |
| 403 | FAILED_DUPLICATE | Duplicate transaction |
| 404 | FAILED_LOCKED | Transaction locked |
| 405 | FAILED_INVALID_CARD_NO | Invalid card number |
| 406 | FAILED_INVALID_EXPIRY | Invalid expiry date |
| 407 | FAILED_EXPIRED_CARD | Card expired |
| 408 | FAILED_UNSPECIFIED | Unspecified failure |

### Error Handling Example

```javascript
async function processPaymentWithErrorHandling(amount, cardSourceId) {
  try {
    const response = await axios.post(
      'https://api.tap.company/v2/charges',
      {
        amount: amount * 100,
        currency: 'EGP',
        source: { id: cardSourceId }
      },
      { headers: authHeaders }
    );

    return { success: true, charge: response.data };

  } catch (error) {
    if (error.response) {
      const status = error.response.status;
      const errors = error.response.data.errors || [];

      if (status === 400) {
        return {
          success: false,
          error: 'Invalid request parameters',
          details: errors
        };
      } else if (status === 402) {
        return {
          success: false,
          error: 'Payment failed',
          details: errors
        };
      } else if (status === 401) {
        return {
          success: false,
          error: 'Invalid API key'
        };
      } else {
        return {
          success: false,
          error: `HTTP ${status}`,
          details: errors
        };
      }
    }

    return {
      success: false,
      error: error.message
    };
  }
}
```

## Important Notes / Gotchas

1. **Amounts in Smallest Unit**: All amounts in API requests are in the smallest currency unit. For EGP that is **piasters** (not fils — fils is a Gulf denomination). Multiply by 100: `50 EGP = 5000 piasters`. For KWD/BHD/AED: use fils (1 KWD = 1000 fils).

2. **HTTPS Required**: All API calls must use HTTPS. HTTP connections will be rejected. Card SDK also requires HTTPS for browser security policies.

3. **Test vs Live Mode**: Separate API keys for test and live modes. Test keys start with `pk_test_` or `sk_test_`. Live keys are production-only. Never commit live keys to version control.

4. **API Key Security**: Always use environment variables or secure vaults for API keys. Never hardcode keys in source code. Rotate keys periodically and revoke old ones.

5. **Save Card Activation Required**: To use the save card feature (storing customer payment methods), you must contact your Tap account manager to enable this feature on your account first.

6. **3D Secure Authentication**: Tap automatically includes 3D Secure authentication in the checkout flow for mada, Visa, Mastercard, and AMEX. No additional configuration needed, but ensure your customer's bank supports it.

7. **Rate Limiting**: Tap implements rate limiting (typically 100 requests per minute). Implement exponential backoff retry logic for handling rate limit errors (429).

8. **Webhook Signature Validation**: Always validate webhook signatures using HMAC-SHA256 before processing. Use your API secret key, not the public key. Reject any webhook with an invalid or missing signature.

9. **Idempotency**: Use the `reference` field with unique values to ensure idempotent requests. If you retry a request with the same reference, you'll get the same response without duplicate charges.

10. **Customer and Card ID Storage**: When saving cards, store both the `customer_id` and `card_id` in your database. You need both to create tokens for recurring payments. Do not store raw card numbers or CVVs.

11. **Token Expiration**: Tokens created for one-time use expire after a period. Use `type: "SUBSCRIPTION"` when creating tokens for recurring payments to ensure they remain valid for multiple uses.

12. **Invoice Due Dates**: When creating invoices with recurring payments, ensure the due_date is in ISO 8601 format (`YYYY-MM-DD`). Tap will automatically attempt payment on the due date.

13. **Metadata Support**: Use the `metadata` object in charges, customers, and invoices to store custom application data. This data is returned in responses and webhooks for easy reconciliation with your system.

14. **Partial Refunds**: You can refund less than the full charged amount. Ensure the refund amount is less than or equal to the original charge amount. Multiple partial refunds are supported for a single charge.

15. **Currency Support**: While EGP is the primary currency for Egypt, Tap supports multi-currency transactions. Always explicitly specify the currency in your requests to avoid confusion.

## Useful Links

- [Tap Payments Official Website](https://www.tap.company/)
- [Tap Payments Egypt](https://www.tap.company/en-eg)
- [API Documentation](https://developers.tap.company/)
- [Authentication Guide](https://developers.tap.company/docs/authentication)
- [Charges API Reference](https://developers.tap.company/reference/charges)
- [Tokens & Saved Cards](https://developers.tap.company/reference/create-a-token-from-saved-card)
- [Recurring Payments Guide](https://developers.tap.company/docs/recurring-payments)
- [Webhook Documentation](https://developers.tap.company/docs/webhook)
- [Payment Methods Overview](https://developers.tap.company/docs/payment-methods)
- [Web Card SDK V2](https://developers.tap.company/docs/card-sdk-web-v2)
- [Checkout SDK for Mobile](https://blog.tap.company/checkout-payment-sdks-mena/)
- [Error Codes Reference](https://developers.tap.company/reference/charge-response-codes)
- [Tap GitHub Repositories](https://github.com/Tap-Payments)
