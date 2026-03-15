---
name: paystack
description: "Integrate with the Paystack payment API for Nigerian and African commerce. Use this skill whenever the user wants to accept payments in Nigeria or West Africa, integrate Paystack into their app, handle NGN transactions, create payment links, verify transactions, manage transfers/payouts, set up subscription plans, or work with Paystack's API in any way. Also trigger when the user mentions 'Paystack', 'Nigerian payments', 'NGN checkout', 'accept payments in Nigeria', or needs to send money to Nigerian bank accounts."
---

# Paystack Integration Skill

Paystack is Nigeria's leading payment gateway, processing payments in NGN (Nigerian Naira) and other African currencies. It powers online and offline payments for businesses across Africa with a clean REST API. Stripe acquired Paystack in October 2020 to accelerate commerce across Africa.

## When to use this skill

You're building something that needs to accept payments in Nigeria or Africa — an e-commerce checkout, a subscription service, a marketplace payout system, or anything that touches money movement in the region. Paystack handles card payments, bank transfers, USSD, mobile money, and direct bank debits.

## Authentication

All Paystack API requests require a secret key passed as a Bearer token:

```
Authorization: Bearer sk_test_xxxxx
```

Keys come in pairs — `sk_test_` for sandbox, `sk_live_` for production. The user should store their key in an environment variable like `PAYSTACK_SECRET_KEY`. Never hardcode keys.

**Base URL:** `https://api.paystack.co`

## Core API Reference

### Initialize a Transaction

This is the starting point for collecting payments. You create a transaction, then redirect the user to Paystack's hosted checkout page.

```
POST /transaction/initialize
```

**Body:**
```json
{
  "email": "customer@email.com",
  "amount": 50000,
  "currency": "NGN",
  "reference": "unique-ref-12345",
  "callback_url": "https://yoursite.com/payment/callback",
  "metadata": {
    "order_id": "ORD-12345",
    "custom_fields": [
      { "display_name": "Cart Items", "variable_name": "cart_items", "value": "2 items" }
    ]
  }
}
```

**Important:** Amount is in **kobo** (smallest currency unit). ₦500 = 50000 kobo.

**Response:**
```json
{
  "status": true,
  "message": "Authorization URL created",
  "data": {
    "authorization_url": "https://checkout.paystack.com/xxxxx",
    "access_code": "xxxxx",
    "reference": "txn_ref_xxxxx"
  }
}
```

Redirect the customer to `authorization_url`. After payment, they'll be sent back to your `callback_url` with `?reference=txn_ref_xxxxx`.

### Verify a Transaction

Always verify server-side after payment. Never trust the client callback alone.

```
GET /transaction/verify/{reference}
```

**Response (success):**
```json
{
  "status": true,
  "message": "Verification successful",
  "data": {
    "id": 291553,
    "reference": "txn_ref_xxxxx",
    "amount": 50000,
    "currency": "NGN",
    "status": "success",
    "paid_at": "2025-01-15T14:30:00.000Z",
    "channel": "card",
    "gateway_response": "Successful",
    "customer": {
      "id": 123456,
      "email": "customer@email.com",
      "customer_code": "CUS_xxxxx"
    },
    "authorization": {
      "authorization_code": "AUTH_xxxxx",
      "card_type": "visa",
      "last4": "4081",
      "bank": "First Bank of Nigeria",
      "country_code": "NG",
      "brand": "Visa",
      "reusable": true
    }
  }
}
```

Check `data.status === "success"` and `data.amount` matches what you expect. The `authorization_code` can be stored (if `reusable: true`) for charging the customer again later without them re-entering card details.

### List Transactions

```
GET /transaction?perPage=50&page=1&status=success&from=2025-01-01&to=2025-01-31
```

Query params: `perPage` (max 100), `page`, `status` (success/failed/abandoned), `from`, `to` (ISO dates), `customer` (customer ID or email), `amount` (in kobo).

### Create a Customer

```
POST /customer
```

```json
{
  "email": "customer@email.com",
  "first_name": "Amina",
  "last_name": "Okafor",
  "phone": "+2348012345678"
}
```

### Create a Transfer Recipient

Before you can send money to someone, register their bank account:

```
POST /transferrecipient
```

```json
{
  "type": "nuban",
  "name": "Amina Okafor",
  "account_number": "0123456789",
  "bank_code": "058",
  "currency": "NGN"
}
```

`type: "nuban"` is for Nigerian bank accounts (NUBAN = Nigeria Uniform Bank Account Number). Use `GET /bank` to look up `bank_code` values.

**Response includes `recipient_code`** — save this to initiate transfers.

### Initiate a Transfer (Payout)

```
POST /transfer
```

```json
{
  "source": "balance",
  "amount": 100000,
  "recipient": "RCP_xxxxx",
  "reason": "Monthly payout"
}
```

Amount is in kobo. `source` is always `"balance"` (from your Paystack balance). Requires transfer authorization — Paystack will send an OTP to the account owner for amounts above a threshold, unless you've disabled this in dashboard settings.

### Verify a Transfer

```
GET /transfer/{transfer_id}
```

Returns the status of a transfer, including whether it's pending, success, or failed.

### List Transfer Recipients

```
GET /transferrecipient?perPage=50&page=1
```

### Create a Subscription Plan

```
POST /plan
```

```json
{
  "name": "Pro Monthly",
  "interval": "monthly",
  "amount": 500000,
  "currency": "NGN",
  "description": "Pro plan — billed monthly"
}
```

Intervals: `hourly`, `daily`, `weekly`, `monthly`, `quarterly`, `biannually`, `annually`.

### Create a Subscription

After a customer authorizes a payment, create a subscription to charge them on schedule:

```
POST /subscription
```

```json
{
  "customer": "CUS_xxxxx",
  "plan": "PLN_xxxxx",
  "authorization": "AUTH_xxxxx"
}
```

### Direct Card Charge (PCI-DSS Servers Only)
> ⚠️ Only use this if your server is PCI-DSS compliant. For most integrations, use Initialize Transaction + hosted page instead.

```
POST /charge
Authorization: Bearer {secret_key}
Content-Type: application/json

{
  "email": "customer@example.com",
  "amount": 50000,
  "card": {
    "number": "4084084084084081",
    "cvv": "408",
    "expiry_month": "01",
    "expiry_year": "99"
  }
}
```
Response may require OTP or 3DS — handle `data.status` values: `send_otp`, `send_pin`, `failed`, `success`.

### Bulk Charges (Payroll / Mass Payouts)
```
POST /bulkcharge
Authorization: Bearer {secret_key}
Content-Type: application/json

[
  { "authorization": "AUTH_xxxxxxxxxx", "amount": 10000, "reference": "ref_001" },
  { "authorization": "AUTH_yyyyyyyyyy", "amount": 20000, "reference": "ref_002" }
]
```
Returns a batch `id`. Poll `GET /bulkcharge/{id}/charges` for individual charge statuses.

### Create a Refund
```
POST /refund
Authorization: Bearer {secret_key}
Content-Type: application/json

{
  "transaction": "ref_or_id_of_original_transaction",
  "amount": 5000
}
```
`amount` is optional — omit to refund the full transaction amount. Partial refunds supported.

### List Banks

```
GET /bank?country=nigeria&perPage=100
```

Returns bank names and codes needed for transfer recipients. Also supports `country=ghana`, `country=south-africa`, `country=kenya`.

## Webhooks

Paystack sends webhooks for transaction events. Set up a POST endpoint and verify the signature using HMAC SHA-512:

### Webhook Signature Verification

The `x-paystack-signature` header contains an HMAC SHA-512 hash of the request body signed with your secret key.

**JavaScript/Node.js:**
```javascript
const crypto = require('crypto');

app.post('/webhook', (req, res) => {
  const hash = crypto
    .createHmac('sha512', process.env.PAYSTACK_SECRET_KEY)
    .update(JSON.stringify(req.body))
    .digest('hex');

  if (hash !== req.headers['x-paystack-signature']) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook event
  const event = req.body;
  if (event.event === 'charge.success') {
    // Handle successful charge
  }

  res.status(200).json({ success: true });
});
```

**Python:**
```python
import hmac
import hashlib
import json

@app.route('/webhook', methods=['POST'])
def webhook():
    secret_key = os.environ.get('PAYSTACK_SECRET_KEY')
    signature = request.headers.get('x-paystack-signature')
    body = request.get_data()

    computed_hash = hmac.new(
        secret_key.encode(),
        body,
        hashlib.sha512
    ).hexdigest()

    if computed_hash != signature:
        return {'error': 'Invalid signature'}, 401

    event = request.get_json()
    if event['event'] == 'charge.success':
        # Handle successful charge
        pass

    return {'success': True}, 200
```

**PHP:**
```php
$secret = getenv('PAYSTACK_SECRET_KEY');
$input = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_PAYSTACK_SIGNATURE'] ?? '';

$computed = hash_hmac('sha512', $input, $secret);

if ($computed !== $signature) {
    http_response_code(401);
    exit('Invalid signature');
}

$event = json_decode($input, true);
if ($event['event'] === 'charge.success') {
    // Handle successful charge
}

http_response_code(200);
```

### Common Webhook Events

- **charge.success** — Customer payment succeeded
- **charge.failed** — Customer payment failed
- **transfer.success** — Payout to recipient succeeded
- **transfer.failed** — Payout failed
- **transfer.reversed** — Transfer was reversed
- **subscription.create** — New subscription created
- **subscription.disable** — Subscription disabled
- **invoice.create** — Invoice created (subscription billing)
- **invoice.update** — Invoice updated
- **invoice.payment_failed** — Invoice payment failed

## Common Integration Patterns

### E-commerce checkout flow
1. `POST /transaction/initialize` → get checkout URL with unique reference
2. Redirect customer → they pay on Paystack's page
3. Customer returns to callback URL
4. `GET /transaction/verify/{reference}` → confirm payment server-side
5. Fulfill order

### Recurring billing (subscriptions)
1. First payment via `POST /transaction/initialize`
2. On success, create `POST /subscription` with stored authorization
3. Paystack charges customer on the plan interval automatically
4. Listen for `invoice.payment_failed` webhook to handle failures

### Marketplace payouts
1. Register sellers with `POST /transferrecipient`
2. After collecting payments, `POST /transfer` to pay sellers
3. Track with `GET /transfer/{id}` or listen for `transfer.success` webhook

## Error Handling

Paystack returns consistent error shapes:

```json
{
  "status": false,
  "message": "Description of what went wrong"
}
```

### HTTP Status Codes and Common Errors

| Status | Scenario | Solution |
|--------|----------|----------|
| **400** | Bad Request — validation error | Check request body format, required fields, amount in kobo |
| **401** | Unauthorized — invalid/missing API key | Verify `Authorization: Bearer sk_test_...` header is correct |
| **403** | Forbidden — action not allowed | Permission issue (e.g., transfer not enabled), contact support |
| **404** | Not Found — resource doesn't exist | Bad reference, recipient_code, or customer ID |
| **429** | Too Many Requests — rate limited | Back off and retry (Paystack allows ~100 requests/second) |
| **500** | Internal Server Error | Paystack issue, retry with exponential backoff |

### Common Error Messages

- `"Invalid key"` — Secret key is missing or malformed
- `"Customer with this email does not exist"` — Customer not found
- `"Reference has already been used"` — Duplicate reference, use a unique one
- `"Amount is not supported for this currency"` — Amount too low or invalid for currency
- `"Recipient with this code does not exist"` — Transfer recipient not found

## Important Notes / Gotchas

### 1. Amounts Are Always in Subunits (Kobo)

All amounts in the Paystack API are in the smallest currency unit:
- **NGN (Nigeria):** 1 kobo = 1/100 NGN. Send 50000 for ₦500.
- **GHS (Ghana):** 1 pesewa = 1/100 GHS. Send 50000 for GHS 500.
- **ZAR (South Africa):** 1 cent = 1/100 ZAR. Send 5000 for R50.
- **KES (Kenya):** 1 cent = 1/100 KES. Send 5000 for Ksh50.

Always multiply your display amount by 100 before sending to the API.

### 2. Transaction References Must Be Unique and Idempotent

- Provide a unique, non-random reference for each transaction (e.g., `order-123-attempt-1` not just a timestamp).
- If you retry a failed request with the same reference, Paystack won't create a duplicate charge.
- If you don't provide a reference, Paystack auto-generates one.
- Once a reference is used, it cannot be reused on the same integration.
- **Best practice:** Use deterministic references tied to your business logic, e.g., `charging-cus10-day5` for recurring charges.

### 3. Single Currency per Account (Exception: Nigeria + Kenya)

A single Paystack account supports only one currency, except for:
- **Nigeria:** Can support NGN + USD simultaneously
- **Kenya:** Can support KES + USD simultaneously

If you need to support multiple currencies (e.g., both NGN and GHS), you must create separate Paystack business accounts.

### 4. Webhook Signature Verification Is Critical

- Always verify the `x-paystack-signature` header using HMAC SHA-512 (not SHA-256).
- Use your **secret key** to compute the hash, not your public key.
- Hash the **raw request body** as a string, not parsed JSON.
- Webhook events are not guaranteed to arrive in order — check timestamps.
- Implement IP whitelisting (dashboard: API Keys & Webhooks → IP Whitelist) as an additional security layer.

### 5. Supported Payment Channels Vary by Region

Not all payment methods are available in all countries:

| Channel | Nigeria | Ghana | Kenya | South Africa | Côte d'Ivoire |
|---------|---------|-------|-------|--------------|---------------|
| Card (Visa, Mastercard, Verve, AmEx) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Bank Account Transfer | ✓ | ✓ | - | - | - |
| USSD | ✓ | - | - | - | - |
| Mobile Money | - | ✓ | ✓ | - | ✓ |
| Apple Pay | ✓ | ✓ | ✓ | ✓ | ✓ |
| QR Code (Visa) | ✓ | ✓ | ✓ | ✓ | ✓ |

### 6. Transfer Authorization (OTP) Is Required

When initiating transfers above a certain threshold (usually ₦50,000), Paystack requires account owner authorization via OTP. You can disable this in dashboard settings, but it's a security best practice.

### 7. Bank Codes Are Country-Specific

Use `GET /bank?country=nigeria` to fetch valid bank codes. Bank codes differ between countries, so always fetch fresh codes if supporting multiple regions.

### 8. Authorization Codes Enable Recurring Charges

When a customer's authorization is marked `reusable: true` (after first successful payment), you can store the `authorization_code` and charge them later without requiring them to re-enter card details:

```
POST /transaction/charge_authorization
```

```json
{
  "authorization_code": "AUTH_xxxxx",
  "email": "customer@email.com",
  "amount": 50000,
  "currency": "NGN"
}
```

### 9. Metadata Is Useful but Not Persisted Automatically

Custom fields in metadata are displayed to users during checkout but are not automatically returned in verification responses. Store important metadata in your own database.

### 10. Paystack Is Now Part of Stripe

Paystack was acquired by Stripe in October 2020. While Paystack operates independently, you can now manage both Stripe and Paystack payments through a single account. However, the APIs remain separate and have different authentication methods.

## Currency Notes

Paystack operates in multiple African currencies, but each account is limited to one (plus USD in Nigeria and Kenya):

- **NGN (Nigeria):** Primary currency, amounts in kobo
- **GHS (Ghana):** Ghanaian Cedi, amounts in pesewas
- **ZAR (South Africa):** South African Rand, amounts in cents
- **KES (Kenya):** Kenyan Shilling, amounts in cents
- **USD:** Only available in Nigeria and Kenya accounts

## Useful Links

- API Docs: https://paystack.com/docs/api/
- Dashboard: https://dashboard.paystack.com
- Test Cards: https://paystack.com/docs/payments/test-payments
- Webhooks: https://paystack.com/docs/payments/webhooks/
- Payment Channels: https://paystack.com/docs/payments/payment-channels/
- Error Codes: https://paystack.com/docs/api/errors/
