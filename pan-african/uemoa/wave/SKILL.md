---
name: Wave Mobile Money API
description: Senegal-based mobile money platform with business APIs for checkout, payouts, and balance management
triggers:
  - Wave
  - Wave mobile money
  - Senegal mobile money
  - Wave API
  - XOF mobile payments
  - Wave checkout
  - Wave payout
  - West Africa mobile money
---

# Wave Mobile Money API

Wave is a Senegal-based mobile money platform providing business APIs for payment collection, fund disbursement, and wallet management across West and East Africa. The platform supports XOF (West African Franc) transactions and operates in multiple countries including Senegal, Côte d'Ivoire, Mali, Burkina Faso, and The Gambia.

## When to Use Wave

- **Payment Collection**: Build checkout experiences for customer payments in West Africa
- **Payouts & Remittances**: Send money to recipients identified by phone numbers
- **Wallet Management**: Check account balances and transaction history
- **Business Integration**: Accept mobile money payments with native API integration
- **Multi-Country Expansion**: Expand across West African markets with single API

## Country and Currency Coverage

Not all Wave products are available in all markets. Reference the table below:

| Country | Currency | Checkout API | Payout API | Balance API |
|---------|----------|-------------|------------|-------------|
| 🇸🇳 Senegal | XOF | ✅ | ✅ | ✅ |
| 🇨🇮 Côte d'Ivoire | XOF | ✅ | ✅ | ✅ |
| 🇲🇱 Mali | XOF | ✅ | ✅ | ✅ |
| 🇧🇫 Burkina Faso | XOF | ✅ | ✅ | ✅ |
| 🇬🇲 The Gambia | GMD | ✅ | ⚠️ Limited | ✅ |

> ⚠️ **GMD (Gambian Dalasi):** The Gambia uses `"GMD"` as the currency code — NOT `"XOF"`. Passing `"XOF"` for Gambian customers will fail. The API key for Gambia is issued separately from UEMOA-region keys. Contact Wave to enable GMD on your account.

> ⚠️ **Onboarding barrier — Senegal business registration required.** Wave's Business Portal signup at `business.wave.com` requires a **Senegal-registered business phone number** to complete account creation. If you are an international developer or a non-Senegalese company, you will not be able to self-register. You must either (1) partner with a Senegalese registered business, (2) contact Wave directly at business@wave.com for enterprise onboarding, or (3) use a Wave-connected aggregator (e.g. Flutterwave or CinetPay) which may offer Wave as a channel without requiring a direct Wave account.

## Authentication

### API Key Setup

1. Log into the [Wave Business Portal](https://business.wave.com)
2. Navigate to Developer section (Admin access required — requires Senegal-registered business phone number; see onboarding note above)
3. Create and manage API keys
4. Each key is bound to a single business wallet
5. Define specific API access per key (Checkout, Payout, Balance)

### Authorization Header

All requests must include an HTTPS Authorization header with the Bearer scheme:

```bash
Authorization: Bearer wave_sn_prod_YOUR_API_KEY_HERE
```

Example with cURL:

```bash
curl -X POST https://api.wave.com/v1/checkout/sessions \
  -H "Authorization: Bearer wave_sn_prod_YhUNb9d...i4bA6" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": "1000",
    "currency": "XOF"
  }'
```

Example with Python:

```python
import requests

api_key = "wave_sn_prod_YOUR_API_KEY_HERE"
headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}

response = requests.post(
    "https://api.wave.com/v1/checkout/sessions",
    headers=headers,
    json={
        "amount": "1000",
        "currency": "XOF",
        "success_url": "https://example.com/success",
        "error_url": "https://example.com/error"
    }
)
```

Example with Node.js:

```javascript
const axios = require('axios');

const apiKey = 'wave_sn_prod_YOUR_API_KEY_HERE';

axios.post('https://api.wave.com/v1/checkout/sessions', {
  amount: '1000',
  currency: 'XOF',
  success_url: 'https://example.com/success',
  error_url: 'https://example.com/error'
}, {
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  }
})
.then(response => console.log(response.data))
.catch(error => console.error(error.response.data));
```

## Core API Reference

### Base URL

```
https://api.wave.com
```

All endpoints are relative to this base URL. All requests must use HTTPS.

### Checkout API

Create payment sessions for customers to pay your business via mobile money.

**Endpoint**: `POST /v1/checkout/sessions`

**Request Body**:

```json
{
  "amount": "1000",
  "currency": "XOF",
  "description": "Payment for Order #123",
  "success_url": "https://example.com/order/success",
  "error_url": "https://example.com/order/error",
  "customer_reference": "CUST_12345"
}
```

**Response (201 Created)**:

```json
{
  "id": "checkout_session_abc123def456",
  "status": "pending",
  "amount": "1000",
  "currency": "XOF",
  "created_at": "2026-02-24T10:30:00Z",
  "expiration_time": "2026-02-24T11:30:00Z",
  "checkout_url": "https://checkout.wave.com/session/abc123def456"
}
```

**Get Checkout Session**: `GET /v1/checkout/sessions/:id`

```bash
curl -X GET https://api.wave.com/v1/checkout/sessions/checkout_session_abc123def456 \
  -H "Authorization: Bearer wave_sn_prod_YhUNb9d...i4bA6"
```

**Expire Session Early**: `POST /v1/checkout/sessions/:id/expire`

```json
{
  "reason": "order_cancelled"
}
```

**Refund Payment**: `POST /v1/checkout/sessions/:id/refund`

```json
{
  "amount": "1000",
  "reason": "customer_request"
}
```

### Payout API

Transfer money from your business wallet to a recipient identified by phone number.

**Endpoint**: `POST /v1/payout`

**Request Body**:

```json
{
  "amount": "50000",
  "currency": "XOF",
  "recipient_phone_number": "+221701234567",
  "recipient_name": "Amadou Diallo",
  "description": "Monthly salary payment",
  "client_reference": "PAY_20260224_001"
}
```

**Response (200 OK)**:

```json
{
  "id": "payout_xyz789abc012",
  "status": "succeeded",
  "amount": "50000",
  "currency": "XOF",
  "recipient_phone_number": "+221701234567",
  "created_at": "2026-02-24T10:30:00Z",
  "effective_date": "2026-02-24"
}
```

**Get Payout Status**: `GET /v1/payout/:id`

```bash
curl -X GET https://api.wave.com/v1/payout/payout_xyz789abc012 \
  -H "Authorization: Bearer wave_sn_prod_YhUNb9d...i4bA6"
```

**Batch Payouts**: `POST /v1/payout-batch`

```json
{
  "currency": "XOF",
  "payouts": [
    {
      "amount": "10000",
      "recipient_phone_number": "+221701234567",
      "client_reference": "BATCH_001"
    },
    {
      "amount": "15000",
      "recipient_phone_number": "+221709876543",
      "client_reference": "BATCH_002"
    }
  ]
}
```

**Reverse Payout**: `POST /v1/payout/:id/reverse`

```json
{
  "reason": "duplicate_entry"
}
```

### Balance API

Check your business wallet balance.

**Endpoint**: `GET /v1/balance`

**Response (200 OK)**:

```json
{
  "balance": "1500000",
  "currency": "XOF",
  "available_balance": "1450000",
  "pending_balance": "50000"
}
```

## Webhooks

Wave sends webhook notifications for key events in your transactions. Configure webhook endpoints in the Business Portal to receive real-time updates.

### Webhook Configuration

1. Go to Developer section → Webhooks
2. Register webhook URL and select event types
3. Save the webhook secret provided by Wave
4. Verify webhook signatures on incoming requests

### Webhook Signature Verification

Each webhook request includes a `Wave-Signature` header with timestamp and signature:

```
Wave-Signature: t=1640179200,v1=signature1
```

**Python Verification Example**:

```python
import hmac
import hashlib
import time

def verify_webhook_signature(request_body, signature_header, webhook_secret):
    # Parse signature header
    parts = {}
    for part in signature_header.split(','):
        key, value = part.split('=')
        parts[key] = value

    timestamp = int(parts['t'])
    signature = parts['v1']

    # Prevent replay attacks (within 5 minutes)
    if abs(time.time() - timestamp) > 300:
        return False

    # Construct signed content
    signed_content = f"{timestamp}.{request_body}"

    # Compute HMAC-SHA256
    expected_signature = hmac.new(
        webhook_secret.encode(),
        signed_content.encode(),
        hashlib.sha256
    ).hexdigest()

    # Constant-time comparison
    return hmac.compare_digest(signature, expected_signature)
```

**Node.js Verification Example**:

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(requestBody, signatureHeader, webhookSecret) {
  const parts = {};
  signatureHeader.split(',').forEach(part => {
    const [key, value] = part.split('=');
    parts[key] = value;
  });

  const timestamp = parseInt(parts.t);
  const signature = parts.v1;

  // Prevent replay attacks
  if (Math.abs(Date.now() / 1000 - timestamp) > 300) {
    return false;
  }

  // Construct signed content
  const signedContent = `${timestamp}.${requestBody}`;

  // Compute HMAC-SHA256
  const expectedSignature = crypto
    .createHmac('sha256', webhookSecret)
    .update(signedContent)
    .digest('hex');

  // Constant-time comparison
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

### Webhook Event Types

Common webhook event types:

- `checkout.session.completed`: Payment session completed successfully
- `checkout.session.expired`: Checkout session expired before payment
- `payout.succeeded`: Payout completed successfully
- `payout.failed`: Payout transaction failed
- `payout.reversed`: Payout was reversed
- `balance.updated`: Wallet balance changed

### Webhook Event Structure

```json
{
  "id": "webhook_event_123456",
  "type": "checkout.session.completed",
  "created_at": "2026-02-24T10:35:00Z",
  "data": {
    "checkout_session_id": "checkout_session_abc123def456",
    "amount": "1000",
    "currency": "XOF",
    "customer_reference": "CUST_12345",
    "customer_phone_number": "+221701234567",
    "transaction_id": "txn_12345678"
  }
}
```

### Webhook Retry Policy

Wave retries failed webhook deliveries with exponential backoff:

- First retry: 1 minute
- Second retry: 5 minutes
- Third retry: 30 minutes
- Fourth retry: 2 hours
- Final retry: 24 hours

Return HTTP 2xx status to acknowledge receipt.

## Common Integration Patterns

### Pattern 1: Simple Checkout Integration

```python
# 1. Create checkout session
response = requests.post(
    'https://api.wave.com/v1/checkout/sessions',
    headers={'Authorization': f'Bearer {api_key}'},
    json={
        'amount': '5000',
        'currency': 'XOF',
        'success_url': 'https://yoursite.com/order/confirm',
        'error_url': 'https://yoursite.com/order/error'
    }
)

session = response.json()

# 2. Redirect customer to checkout
checkout_url = session['checkout_url']
# Redirect to checkout_url

# 3. Handle webhook notification
@app.route('/webhook/wave', methods=['POST'])
def handle_wave_webhook():
    if not verify_webhook_signature(
        request.data,
        request.headers.get('Wave-Signature'),
        webhook_secret
    ):
        return {'error': 'Invalid signature'}, 403

    event = request.json

    if event['type'] == 'checkout.session.completed':
        order_id = event['data']['customer_reference']
        amount = event['data']['amount']
        # Mark order as paid
        db.orders.update_one(
            {'id': order_id},
            {'$set': {'status': 'paid'}}
        )

    return {'success': True}, 200
```

### Pattern 2: Batch Payroll Processing

```javascript
// Prepare payroll data
const payrollData = [
  { phone: '+221701111111', amount: '100000', name: 'Employee 1' },
  { phone: '+221702222222', amount: '100000', name: 'Employee 2' },
  { phone: '+221703333333', amount: '100000', name: 'Employee 3' }
];

// Create batch payout
const batchPayload = {
  currency: 'XOF',
  payouts: payrollData.map((emp, idx) => ({
    amount: emp.amount,
    recipient_phone_number: emp.phone,
    recipient_name: emp.name,
    client_reference: `PAYROLL_${Date.now()}_${idx}`
  }))
};

const response = await axios.post(
  'https://api.wave.com/v1/payout-batch',
  batchPayload,
  {
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    }
  }
);

console.log(`Batch ID: ${response.data.id}`);
```

### Pattern 3: Balance Monitoring

```bash
#!/bin/bash

API_KEY="wave_sn_prod_YOUR_KEY"
WEBHOOK_SECRET="your_webhook_secret"

# Check balance hourly
while true; do
  BALANCE=$(curl -s https://api.wave.com/v1/balance \
    -H "Authorization: Bearer $API_KEY" | jq '.balance')

  TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
  echo "$TIMESTAMP - Current balance: $BALANCE XOF"

  # Alert if balance below threshold
  if (( $(echo "$BALANCE < 100000" | bc -l) )); then
    echo "WARNING: Balance below 100000 XOF"
    # Send alert email/SMS
  fi

  sleep 3600
done
```

## B2B Payment Request Flow

Wave supports business-to-business payment requests — you send a payment request to another Wave business wallet, and they approve it from their Wave app. This is distinct from the checkout (C2B) and payout (B2C) flows.

### Create a B2B Payment Request
```bash
POST https://api.wave.com/v1/business-payment-requests
Authorization: Bearer wave_sn_prod_YOUR_API_KEY
Content-Type: application/json

{
  "amount": "50000",
  "currency": "XOF",
  "payment_from_business_id": "wbiz_target_business_id",
  "short_description": "Invoice #INV-2024-047 — Services rendered February 2024",
  "client_reference": "INV-2024-047"
}
```

`payment_from_business_id` is the Wave Business ID of the business you're requesting payment from — they must be a registered Wave business with an active business wallet.

**Response:**
```json
{
  "id": "wbpr_xxxxxxxxxxxx",
  "status": "pending",
  "amount": "50000",
  "currency": "XOF",
  "payment_from_business_id": "wbiz_target_business_id",
  "short_description": "Invoice #INV-2024-047",
  "created_at": "2024-02-24T11:00:00Z",
  "expires_at": "2024-02-25T11:00:00Z"
}
```

The receiving business gets a notification in their Wave app. They can approve or decline. Wave fires a webhook to your endpoint when the status changes.

### Get B2B Payment Request Status
```bash
GET https://api.wave.com/v1/business-payment-requests/{id}
Authorization: Bearer wave_sn_prod_YOUR_API_KEY
```

**Status values:** `pending` → `completed` or `declined` or `expired`

## Error Handling

### HTTP Status Codes

- **200 OK**: Request successful
- **201 Created**: Resource created successfully
- **400 Bad Request**: Invalid request parameters
- **401 Unauthorized**: Missing or invalid API key
- **403 Forbidden**: API key lacks permission for endpoint
- **404 Not Found**: Resource not found
- **429 Too Many Requests**: Rate limit exceeded
- **500 Internal Server Error**: Server error
- **503 Service Unavailable**: Service temporarily down

### Error Response Format

```json
{
  "error": {
    "code": "insufficient_balance",
    "message": "Your business wallet does not have sufficient balance for this payout",
    "details": {
      "required": "50000",
      "available": "25000"
    }
  }
}
```

### Common Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| `invalid_amount` | Amount format or value incorrect | Validate amount is positive number |
| `invalid_phone` | Phone number format invalid | Ensure E.164 format (+221XXXXXXXXX) |
| `insufficient_balance` | Insufficient funds | Ensure wallet has adequate balance |
| `invalid_currency` | Currency not supported | Use XOF for West African transactions |
| `rate_limit_exceeded` | Too many requests | Implement exponential backoff |
| `invalid_api_key` | API key invalid/expired | Verify key in Business Portal |
| `webhook_secret_mismatch` | Webhook signature invalid | Verify webhook secret and signing |
| `recipient_not_found` | Phone number not registered | Verify recipient is Wave user |
| `duplicate_reference` | Client reference already used | Use unique references |
| `payout_timeout` | Payout took too long to process | Wait and check status later |

### Retry Strategy

```python
import time
import random

def with_exponential_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise

            wait_time = (2 ** attempt) + random.uniform(0, 1)
            print(f"Attempt {attempt + 1} failed. Retrying in {wait_time:.2f}s...")
            time.sleep(wait_time)

# Usage
def send_payout():
    return requests.post(
        'https://api.wave.com/v1/payout',
        headers={'Authorization': f'Bearer {api_key}'},
        json=payout_data
    )

response = with_exponential_backoff(send_payout)
```

## Important Notes & Gotchas

### 1. API Key Security

- **Never** commit API keys to version control
- **Never** expose keys in client-side code
- Use environment variables or secure vaults (AWS Secrets Manager, HashiCorp Vault)
- Rotate keys regularly and revoke unused ones
- API keys can perform any action, including moving money

### 2. Phone Number Format

Wave requires phone numbers in E.164 format: `+[country_code][number]`

```
Correct:   +221701234567 (Senegal)
Incorrect: 0701234567
Incorrect: 701234567
```

Always validate and normalize phone numbers before API requests.

### 3. Amount Precision

Wave works with string amounts to avoid floating-point precision errors:

```python
# Correct
amount = "1000"  # 1000 XOF

# Incorrect
amount = 1000.00  # Float can cause precision issues
amount = 10.50    # Float
```

Always send amounts as strings in API requests.

### 4. Webhook Timestamp Validation

The 5-minute replay attack window is strict. Ensure server clocks are synchronized with NTP:

```bash
# Linux: check NTP sync
timedatectl status

# Docker: add NTP sync
docker run --cap-add SYS_TIME -v /etc/ntp.conf:/etc/ntp.conf ...
```

### 5. Checkout Session Expiration

Checkout sessions expire after 1 hour. For long-form purchase flows, create the session only when customer is ready to pay:

```python
# Bad: Create session at page load
session = create_checkout_session()

# Good: Create session just before payment
@app.route('/api/checkout', methods=['POST'])
def start_checkout():
    # Only create when customer submits form
    return create_checkout_session()
```

### 6. Idempotency with Client References

Use `client_reference` to ensure exactly-once semantics for payouts:

```python
# If request fails and you retry with same client_reference,
# Wave returns the original payout instead of creating duplicate
response1 = send_payout(client_reference='PAY_001')
# Network error - safe to retry
response2 = send_payout(client_reference='PAY_001')
# Same payout returned
```

### 7. Rate Limiting & Backoff

Wave applies rate limits during high traffic:

```
429 Too Many Requests
Retry-After: 60
```

Implement jittered exponential backoff to avoid thundering herd:

```python
base_wait = 2 ** attempt
jitter = random.uniform(0, base_wait)
wait_time = base_wait + jitter
```

### 8. Currency & Country Restrictions

- **Senegal**: XOF only
- **Côte d'Ivoire**: XOF only
- **Other countries**: Check latest docs for supported currencies
- Cannot mix currencies in batch operations
- Conversion rates not provided by API (use market data)

### 9. Testing vs Production

Wave provides separate API key prefixes:

- `wave_sn_test_*` for sandbox/testing
- `wave_sn_prod_*` for production

Create separate Business Portal accounts for testing and never use production keys in development.

### 10. Webhook Delivery Guarantees

Wave webhooks use at-least-once delivery:

```python
# Your webhook handler MUST be idempotent
@app.route('/webhook/wave', methods=['POST'])
def handle_webhook():
    event_id = request.json['id']

    # Check if we've already processed this event
    if db.processed_events.find_one({'event_id': event_id}):
        return {'success': True}, 200

    # Process the event
    process_event(request.json)

    # Mark as processed
    db.processed_events.insert_one({'event_id': event_id})

    return {'success': True}, 200
```

Webhooks may be delivered multiple times. Store processed event IDs to prevent duplicate processing.

## Useful Links

- [Wave Official Website](https://wave.com)
- [Wave Business API Reference](https://docs.wave.com/business)
- [Wave Checkout API Documentation](https://docs.wave.com/checkout)
- [Wave Payout API Documentation](https://docs.wave.com/payout)
- [Wave Balance API Documentation](https://docs.wave.com/balance-api)
- [Wave Webhook Documentation](https://docs.wave.com/webhook)
- [Wave Business Portal](https://business.wave.com)
- [Wave API Integration Guide](https://rollout.com/integration-guides/wave/quick-guide-to-implementing-webhooks-in-wave)
- [Y Combinator - Wave Profile](https://www.ycombinator.com/companies/wave)
- [Wave GitHub Custom API Example](https://github.com/gabrielmuteki/wave_custom_api)
