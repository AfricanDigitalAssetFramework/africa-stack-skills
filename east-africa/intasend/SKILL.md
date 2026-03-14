---
name: intasend
description: "Integrate with IntaSend payment processing API for cards, M-Pesa, bank transfers, and crypto. Use this skill whenever the user wants to accept multiple payment methods, process card payments, collect M-Pesa via STK-Push, handle disbursements, manage wallets, or work with IntaSend's unified payment infrastructure. Trigger when the user mentions 'IntaSend', 'M-Pesa collection', 'payment disbursement', 'multi-currency payments', 'crypto payments', or needs to handle diverse payment methods in East Africa."
---

# IntaSend Integration Skill

IntaSend is a unified payment infrastructure platform for East Africa, enabling merchants and developers to accept cards, M-Pesa (including STK-Push collections), bank transfers, and cryptocurrencies through a single API. Perfect for businesses needing multi-channel payments with fraud protection, instant settlements, and rapid integration.

## When to use this skill

Use IntaSend when you need to:
- **Accept multiple payment methods** in a single checkout — card payments, M-Pesa STK-Push, bank transfers, PesaLink, or crypto
- **Send money at scale** — batch disbursements, payouts, commissions, or B2C/B2B transfers
- **Manage customer wallets** — track balances, intra-wallet transfers, and withdraw funds
- **Automate collections** — trigger M-Pesa STK-Push requests directly from your backend without customer friction
- **Support East Africa** — Kenya (KES), Uganda (UGX), Tanzania (TZS), Ghana (GHS) with local payment methods

IntaSend's unique value: reduce payment integration complexity from weeks to hours with unified APIs, built-in M-Pesa handling, sandbox environment for testing, and SDKs for Python, PHP, JavaScript, and Go.

## Authentication

IntaSend uses **Bearer token authentication** with environment-specific keys.

### API Keys

IntaSend provides **two types of keys** for every account:

1. **Public/Publishable Key** (Prefix: `ISPubKey_test_` or `ISPubKey_live_`)
   - Safe to use in frontend code
   - Used only to identify your business during checkout
   - Used in public-facing collection requests

2. **Secret Key** (Prefix: `ISSecretKey_test_` or `ISSecretKey_live_`)
   - Must be kept secure and only used in backend code
   - Grants full API access (refunds, transfers, wallet operations)
   - Never expose in frontend or version control
   - Store securely in environment variables (e.g., `INTASEND_SECRET_KEY`)

### Authorization Header

All API requests to protected resources require the Authorization header with Bearer token:

```
Authorization: Bearer ISSecretKey_test_xxxxx
Content-Type: application/json
```

### Base URL

**Sandbox (Testing):**
```
https://sandbox.intasend.com/api/v1
```

**Production (Live):**
```
https://payment.intasend.com/api/v1
```

### Environment Types

- **Test keys** (containing `test` in the prefix): Use in sandbox for development and testing
- **Live keys** (containing `live` in the prefix): Use only in production with real transactions

### Key Storage Best Practices

```python
# Example: Python with environment variables
import os
from dotenv import load_dotenv

load_dotenv()

INTASEND_SECRET_KEY = os.getenv("INTASEND_SECRET_KEY")
INTASEND_PUBLIC_KEY = os.getenv("INTASEND_PUBLIC_KEY")

# Use in headers
headers = {
    "Authorization": f"Bearer {INTASEND_SECRET_KEY}",
    "Content-Type": "application/json"
}
```

Never hardcode credentials. Always use the 12-factor app methodology to keep secrets in environment variables.

## Core API Reference

### 1. Create Payment Collection (M-Pesa STK-Push)

Automatically trigger an M-Pesa STK-Push payment request from your backend. The customer receives a prompt on their phone to enter their M-Pesa PIN without leaving your app.

**Endpoint:**
```
POST https://payment.intasend.com/api/v1/payment/collection/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
Content-Type: application/json
```

**Request Body:**
```json
{
  "public_key": "ISPubKey_test_xxxxx",
  "currency": "KES",
  "amount": 1500,
  "api_ref": "ORD-2026-001",
  "phone_number": "+254712345678",
  "name": "John Doe",
  "email": "john@example.com",
  "method": "M-PESA"
}
```

**Field Descriptions:**
- `public_key` (string, required): Your IntaSend publishable key
- `currency` (string, required): ISO 4217 currency code (KES, UGX, TZS, GHS, USD)
- `amount` (number, required): Payment amount in whole numbers (e.g., 1500 for KES 1,500)
- `api_ref` (string, required): Your unique transaction reference (used for idempotency)
- `phone_number` (string, required): Customer phone with country code (e.g., +254, +256)
- `name` (string, required): Customer's full name
- `email` (string, optional): Customer's email address
- `method` (string, required): Payment method ("M-PESA" for STK-Push)

**Response (Success - HTTP 200):**
```json
{
  "invoice": {
    "id": "XMSLWOS",
    "invoice_id": "XMSLWOS",
    "state": "PENDING",
    "provider": "M-PESA",
    "value": "1500.00",
    "account": "john@example.com",
    "api_ref": "ORD-2026-001",
    "failed_reason": null,
    "created_at": "2026-02-24T10:30:47.040822+03:00",
    "updated_at": "2026-02-24T10:30:47.040849+03:00"
  }
}
```

**Response Fields:**
- `id`: Unique invoice identifier (use for status checks)
- `state`: Current payment state (PENDING, COMPLETE, FAILED)
- `provider`: Payment method used (M-PESA, CARD, BANK, etc.)
- `value`: Payment amount
- `api_ref`: Your reference code (echoed back)
- `failed_reason`: Error message if payment fails

**Example Integration (Python):**
```python
import requests
import json

def create_mpesa_collection(phone, amount, reference):
    headers = {
        "Authorization": f"Bearer {INTASEND_SECRET_KEY}",
        "Content-Type": "application/json"
    }

    payload = {
        "public_key": INTASEND_PUBLIC_KEY,
        "currency": "KES",
        "amount": amount,
        "api_ref": reference,
        "phone_number": phone,
        "name": "Customer Name",
        "method": "M-PESA"
    }

    response = requests.post(
        "https://payment.intasend.com/api/v1/payment/collection/",
        headers=headers,
        json=payload
    )

    if response.status_code == 200:
        return response.json()["invoice"]
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return None
```

---

### 2. Get Payment/Invoice Status

Retrieve the current status of a payment collection or invoice.

**Endpoint:**
```
GET https://payment.intasend.com/api/v1/payment/{invoice_id}/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
```

**URL Parameters:**
- `invoice_id` (string, required): The invoice ID from collection response

**Response (Success - HTTP 200):**
```json
{
  "invoice": {
    "id": "XMSLWOS",
    "invoice_id": "XMSLWOS",
    "state": "COMPLETE",
    "provider": "M-PESA",
    "value": "1500.00",
    "account": "john@example.com",
    "api_ref": "ORD-2026-001",
    "failed_reason": null,
    "mpesa_reference": "LIK8OVW5YV",
    "created_at": "2026-02-24T10:30:47.040822+03:00",
    "updated_at": "2026-02-24T10:31:20.123456+03:00"
  }
}
```

**Possible States:**
- `PENDING`: Awaiting customer payment confirmation
- `COMPLETE`: Payment successfully received
- `FAILED`: Payment declined or timeout occurred

---

### 3. Initiate Send Money / Disbursement

Send money from your IntaSend wallet to another account (M-Pesa, Bank, or another wallet).

**Endpoint:**
```
POST https://payment.intasend.com/api/v1/send-money/initiate/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
Content-Type: application/json
```

**Request Body:**
```json
{
  "amount": 2500,
  "currency": "KES",
  "phone_number": "+254712345678",
  "reason": "Commission payout",
  "api_ref": "PAYOUT-2026-001"
}
```

**Field Descriptions:**
- `amount` (number, required): Disbursement amount in whole numbers
- `currency` (string, required): ISO 4217 currency code
- `phone_number` (string, required): Recipient's phone number (for M-Pesa) with country code
- `reason` (string, required): Purpose of transfer (appears in transaction record)
- `api_ref` (string, required): Your unique reference for idempotency

**Response (Success - HTTP 200):**
```json
{
  "transfer_id": "TXN-2026-abcd1234",
  "phone_number": "+254712345678",
  "amount": 2500,
  "currency": "KES",
  "status": "PROCESSING",
  "reason": "Commission payout",
  "api_ref": "PAYOUT-2026-001",
  "created_at": "2026-02-24T10:35:00.000000+03:00"
}
```

**Response Fields:**
- `transfer_id`: Unique identifier for this disbursement
- `status`: Processing status (PROCESSING, COMPLETED, FAILED)
- `created_at`: Timestamp of initiation

**Note:** IntaSend may require approval for disbursements depending on your account. Check for approval endpoint if needed.

---

### 4. Get Disbursement/Transfer Status

Check the status of a send money transaction.

**Endpoint:**
```
GET https://payment.intasend.com/api/v1/send-money/{transfer_id}/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
```

**Response (Success - HTTP 200):**
```json
{
  "transfer_id": "TXN-2026-abcd1234",
  "phone_number": "+254712345678",
  "amount": 2500,
  "currency": "KES",
  "status": "COMPLETED",
  "reason": "Commission payout",
  "api_ref": "PAYOUT-2026-001",
  "completed_at": "2026-02-24T10:36:45.000000+03:00"
}
```

---

### 5. Create Wallet

Create a customer wallet for balance management and intra-wallet transfers.

**Endpoint:**
```
POST https://payment.intasend.com/api/v1/wallets/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
Content-Type: application/json
```

**Request Body:**
```json
{
  "phone_number": "+254712345678",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com"
}
```

**Response (Success - HTTP 200):**
```json
{
  "wallet_id": "WALL-2026-xyz789",
  "phone_number": "+254712345678",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "balance": 0,
  "currency": "KES",
  "created_at": "2026-02-24T10:30:00.000000+03:00"
}
```

---

### 6. Get Wallet Balance

Retrieve a wallet's current balance.

**Endpoint:**
```
GET https://payment.intasend.com/api/v1/wallets/{wallet_id}/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
```

**Response (Success - HTTP 200):**
```json
{
  "wallet_id": "WALL-2026-xyz789",
  "phone_number": "+254712345678",
  "balance": 15000,
  "currency": "KES",
  "available_balance": 15000
}
```

---

### 7. List Wallets

Retrieve all wallets associated with your account.

**Endpoint:**
```
GET https://payment.intasend.com/api/v1/wallets/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
```

**Query Parameters:**
- `page` (integer, optional): Page number for pagination (default: 1)
- `page_size` (integer, optional): Results per page (default: 20, max: 100)

**Response (Success - HTTP 200):**
```json
{
  "results": [
    {
      "wallet_id": "WALL-2026-xyz789",
      "phone_number": "+254712345678",
      "balance": 15000,
      "currency": "KES"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_count": 1
  }
}
```

---

### 8. Intra-Wallet Transfer

Transfer funds between wallets/sub-accounts you own (useful for reconciliation across business units).

**Endpoint:**
```
POST https://payment.intasend.com/api/v1/wallets/{source_wallet_id}/intra_transfer/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
Content-Type: application/json
```

**Request Body:**
```json
{
  "destination_wallet_id": "WALL-2026-abc123",
  "amount": 5000,
  "reference": "TRANSFER-001"
}
```

**Response (Success - HTTP 200):**
```json
{
  "transfer_id": "INTXFER-2026-001",
  "source_wallet_id": "WALL-2026-xyz789",
  "destination_wallet_id": "WALL-2026-abc123",
  "amount": 5000,
  "status": "COMPLETED",
  "created_at": "2026-02-24T10:40:00.000000+03:00"
}
```

---

### 9. Checkout Links (Hosted Payment Page)

Generate a secure hosted checkout page where customers can select their preferred payment method.

**Endpoint:**
```
POST https://payment.intasend.com/api/v1/checkout/
```

**Headers:**
```
Authorization: Bearer ISSecretKey_test_xxxxx
Content-Type: application/json
```

**Request Body:**
```json
{
  "amount": 5000,
  "currency": "KES",
  "first_name": "Jane",
  "last_name": "Smith",
  "email": "jane@example.com",
  "phone_number": "+254712345679",
  "redirect_url": "https://yoursite.com/success",
  "webhook_url": "https://yoursite.com/webhook/intasend",
  "reference": "ORDER-12345",
  "comments": "Payment for subscription"
}
```

**Response (Success - HTTP 200):**
```json
{
  "success": true,
  "checkout_id": "CHK-2026-abc123",
  "checkout_url": "https://checkout.intasend.com/CHECKOUT_ID_HERE",
  "amount": 5000,
  "currency": "KES",
  "status": "PENDING"
}
```

Redirect users to the `checkout_url`. They can pay via card, M-Pesa, bank transfer, or crypto, then return to your `redirect_url`.

---

## Webhooks

IntaSend sends real-time webhooks to notify your system of payment events. **Always verify webhook signatures** to ensure authenticity.

### Webhook Signature Verification

IntaSend signs webhooks with HMAC-SHA256. Verify the signature using your secret key:

**Header:** `X-Intasend-Signature`

**Verification (Node.js):**
```javascript
const crypto = require('crypto');

function verifyIntaSendWebhook(payload, signature, secretKey) {
  const hash = crypto
    .createHmac('sha256', secretKey)
    .update(JSON.stringify(payload))
    .digest('hex');

  // Use constant-time comparison to prevent timing attacks
  return crypto.timingSafeEqual(
    Buffer.from(hash),
    Buffer.from(signature)
  );
}

app.post('/webhook/intasend', (req, res) => {
  const signature = req.headers['x-intasend-signature'];

  if (!verifyIntaSendWebhook(req.body, signature, INTASEND_SECRET_KEY)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook
  console.log('Valid webhook received:', req.body);
  res.json({ success: true });
});
```

**Verification (Python):**
```python
import hmac
import hashlib
import json

def verify_intasend_webhook(payload, signature, secret_key):
    hash_obj = hmac.new(
        secret_key.encode(),
        payload.encode(),
        hashlib.sha256
    )
    expected_signature = hash_obj.hexdigest()
    return hmac.compare_digest(expected_signature, signature)

@app.route('/webhook/intasend', methods=['POST'])
def webhook():
    signature = request.headers.get('X-Intasend-Signature')
    payload = request.get_data(as_text=True)

    if not verify_intasend_webhook(payload, signature, INTASEND_SECRET_KEY):
        return {'error': 'Invalid signature'}, 401

    data = request.get_json()
    print(f"Valid webhook: {data['event']}")
    return {'success': True}
```

### Webhook Events

**Payment Complete:**
```json
{
  "event": "payment.complete",
  "data": {
    "id": "XMSLWOS",
    "invoice_id": "XMSLWOS",
    "state": "COMPLETE",
    "provider": "M-PESA",
    "value": "1500.00",
    "api_ref": "ORD-2026-001",
    "mpesa_reference": "LIK8OVW5YV",
    "created_at": "2026-02-24T10:30:47.040822+03:00",
    "updated_at": "2026-02-24T10:31:20.123456+03:00"
  }
}
```

**Payment Failed:**
```json
{
  "event": "payment.failed",
  "data": {
    "id": "XMSLWOS",
    "state": "FAILED",
    "provider": "M-PESA",
    "value": "1500.00",
    "api_ref": "ORD-2026-001",
    "failed_reason": "Insufficient balance",
    "updated_at": "2026-02-24T10:35:00.000000+03:00"
  }
}
```

**Transfer Complete:**
```json
{
  "event": "transfer.complete",
  "data": {
    "transfer_id": "TXN-2026-abcd1234",
    "phone_number": "+254712345678",
    "amount": 2500,
    "status": "COMPLETED",
    "api_ref": "PAYOUT-2026-001",
    "completed_at": "2026-02-24T10:36:45.000000+03:00"
  }
}
```

**Best Practices:**
- Always verify webhook signatures before processing
- Implement idempotency by tracking webhook IDs (log received IDs)
- Return HTTP 200 immediately; process asynchronously
- Retry failed webhook deliveries (IntaSend will retry on 5xx errors)
- Use a webhook logging service like Hookdeck for debugging

---

## Common Integration Patterns

### Pattern 1: M-Pesa STK-Push Collection Flow

Optimal for mobile apps where you want automatic M-Pesa prompts:

```
1. User initiates payment in your app
2. POST /payment/collection/ with phone_number and amount
3. M-Pesa STK-Push appears on customer's phone
4. Customer enters PIN
5. Receive webhook payment.complete
6. GET /payment/{invoice_id}/ to verify status
7. Fulfill order/service
```

**Code Example:**
```python
# 1. Create collection
invoice = create_mpesa_collection("+254712345678", 1500, "ORD-2026-001")
invoice_id = invoice['id']

# 2. Poll for status (optional, webhooks preferred)
import time
for attempt in range(30):  # Poll for 5 minutes
    time.sleep(10)
    status = get_invoice_status(invoice_id)
    if status['state'] == 'COMPLETE':
        print(f"Payment complete! Reference: {status['mpesa_reference']}")
        fulfill_order("ORD-2026-001")
        break
    elif status['state'] == 'FAILED':
        print(f"Payment failed: {status['failed_reason']}")
        break
```

---

### Pattern 2: Hosted Checkout (Multi-Method)

Best for web and e-commerce where customers choose payment method:

```
1. Create checkout link with POST /checkout/
2. Redirect customer to checkout_url
3. Customer selects payment method and pays
4. Customer redirected to your redirect_url
5. Webhook notifies of payment completion
6. GET /checkout/{checkout_id}/ to verify (optional)
7. Fulfill order
```

**Code Example:**
```python
# 1. Create checkout
checkout = create_checkout(
    amount=5000,
    currency="KES",
    reference="ORDER-12345",
    redirect_url="https://yoursite.com/success"
)

# 2. Share checkout_url with customer (in email, SMS, QR code)
# 3. Listen for webhook payment.complete
# 4. Fulfill order
```

---

### Pattern 3: Batch Payouts / Disbursements

Scale commission payouts, affiliate payments, or refunds:

```
1. Collect all payouts needed (array of {phone, amount, reference})
2. For each payout: POST /send-money/initiate/
3. Track transfer_id for each
4. Listen for transfer.complete webhooks
5. Update accounting/ledger
6. Optional: Retry failed transfers with exponential backoff
```

**Code Example:**
```python
payouts = [
    {"phone": "+254712345678", "amount": 2500, "ref": "AFFILIATE-001"},
    {"phone": "+254798765432", "amount": 3000, "ref": "AFFILIATE-002"},
    {"phone": "+254787654321", "amount": 1500, "ref": "AFFILIATE-003"},
]

transfer_ids = []
for payout in payouts:
    result = initiate_send_money(
        phone_number=payout['phone'],
        amount=payout['amount'],
        reason="Affiliate commission",
        api_ref=payout['ref']
    )
    transfer_ids.append(result['transfer_id'])
    print(f"Started transfer: {result['transfer_id']}")

# Listen for webhooks to confirm completion
```

---

### Pattern 4: Wallet-Based Ecosystem

For platforms managing customer balances (escrow, prepaid, earned balances):

```
1. POST /wallets/ to create customer wallet
2. Customer pays via checkout → funds land in wallet
3. GET /wallets/{wallet_id}/ to check balance
4. POST /wallets/{wallet_id}/intra_transfer/ to move between accounts
5. POST /send-money/initiate/ to cash out to M-Pesa
6. Track all via webhooks
```

**Code Example:**
```python
# Create wallet for customer
wallet = create_wallet(
    phone_number="+254712345678",
    first_name="John",
    email="john@example.com"
)
wallet_id = wallet['wallet_id']

# Later: Check balance
balance = get_wallet_balance(wallet_id)
print(f"Wallet balance: KES {balance['balance']}")

# Payout wallet funds to M-Pesa
payout = initiate_send_money(
    phone_number="+254712345678",
    amount=5000,
    reason="Wallet withdrawal",
    api_ref="WITHDRAW-001"
)
```

---

## Error Handling

IntaSend returns standard HTTP status codes with descriptive error messages.

### Response Format

**Error Response:**
```json
{
  "success": false,
  "message": "Invalid phone number format"
}
```

### HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process response normally |
| 400 | Bad Request | Validation error (check field names, formats, required fields) |
| 401 | Unauthorized | Invalid or missing authentication header/credentials |
| 403 | Forbidden | Insufficient permissions or wallet balance for operation |
| 404 | Not Found | Payment/transfer/wallet ID doesn't exist |
| 429 | Too Many Requests | Rate limited; implement exponential backoff |
| 500 | Server Error | IntaSend server error; retry with backoff |

### Common Errors & Solutions

**Error: Invalid phone number format**
```json
{
  "success": false,
  "message": "Invalid phone number format"
}
```
**Solution:** Ensure phone includes country code: `+254712345678` (not `0712345678` or `254712345678`)

**Error: Invalid API credentials**
```json
{
  "success": false,
  "message": "Unauthorized"
}
```
**Solution:** Check your secret key in Authorization header. Ensure it's from the correct environment (test vs. live).

**Error: Insufficient wallet balance**
```json
{
  "success": false,
  "message": "Insufficient wallet balance for this operation"
}
```
**Solution:** Fund your IntaSend wallet via collections or deposits before sending money.

**Error: Duplicate request**
```json
{
  "success": false,
  "message": "Duplicate api_ref"
}
```
**Solution:** Each transaction needs a unique `api_ref`. IntaSend uses this for idempotency — retrying with the same ref returns the original response without duplicate charges.

**Error: Rate limited (HTTP 429)**
```json
{
  "success": false,
  "message": "Rate limit exceeded"
}
```
**Solution:** Implement exponential backoff. Wait 1s, 2s, 4s, etc. before retrying.

### Retry Strategy

```python
import time
import random

def call_intasend_with_retry(func, max_retries=3, base_wait=1):
    """Exponential backoff retry for IntaSend API calls"""
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise

            # Exponential backoff with jitter
            wait_time = base_wait * (2 ** attempt) + random.uniform(0, 1)
            print(f"Attempt {attempt + 1} failed. Retrying in {wait_time:.2f}s...")
            time.sleep(wait_time)
```

---

## Important Notes and Gotchas

### Phone Number Format

- **Always include country code**: `+254712345678` ✓, not `0712345678` ✗
- Supported countries: Kenya (+254), Uganda (+256), Tanzania (+255), Ghana (+233)
- Validate format before sending to API

### Amount Handling

- **Amounts are whole numbers** — no decimals: `1500` ✓, not `1500.50` ✗
- Minimum amount varies by currency and payment method (typically KES 100 minimum)
- Currency must match account locale (KES for Kenya, UGX for Uganda, etc.)

### API Reference / Idempotency

- Every transaction (`api_ref`) must be unique within your account
- Retrying with the same `api_ref` returns the original response (idempotent)
- Use UUIDs or incrementing counters: `ORD-2026-001`, `PAYOUT-20260224-001`, etc.
- **Never change amount or phone for the same api_ref** — create a new transaction instead

### Webhook Considerations

- **Always verify signatures** — anyone can POST to your webhook URL
- IntaSend will retry failed webhook deliveries (HTTP 5xx errors)
- Return HTTP 200 quickly; process async to avoid timeout
- Webhooks may arrive out of order — check timestamps
- Handle duplicate webhook events (idempotency) — store webhook event IDs
- Use webhook delivery services like Hookdeck to test and debug

### Testing

- **Test keys** (containing "test" prefix): Use https://sandbox.intasend.com
- IntaSend provides test phone numbers and card details in your dashboard
- M-Pesa test requires approval from IntaSend support
- Always test both success and failure scenarios
- Full end-to-end: create payment → trigger webhook in sandbox → verify receipt

### Rate Limiting

- Standard rate limit: ~100-200 requests/minute (check docs for current limits)
- Implement exponential backoff for retries
- Batch operations where possible (send multiple payouts in sequence, not parallel)

### Settlements

- Payments received → IntaSend balance updates immediately
- IntaSend → Your bank account: typically within 24 hours
- Disbursements you send → Recipient receives within minutes for M-Pesa
- Check your settlement schedule in the dashboard

### Security Best Practices

1. **Never hardcode API keys** — use environment variables
2. **Never log full API keys** — log last 4 characters only: `ISSecretKey_test_...xxxx`
3. **Keep secret keys server-side only** — never expose in frontend or client apps
4. **Use HTTPS only** for all requests (HTTP will be rejected)
5. **Rotate API keys** periodically in production
6. **Monitor webhook authenticity** — validate signatures always
7. **Implement CSRF protection** on webhook endpoints

### Account Limits & Tiers

- Limits vary by account tier and verification level
- Higher tiers get better rates, higher limits, priority support
- Check your dashboard for current limits
- Contact IntaSend support to upgrade account tier

### SDK Availability

IntaSend provides official SDKs:
- **Python**: `pip install intasend`
- **PHP**: `composer require intasend/intasend-php`
- **JavaScript/Node.js**: Available via npm
- **Go**: Available via GitHub

Using SDKs handles authentication headers and response parsing automatically.

---

## Useful Links

- **Official Website:** https://intasend.com/
- **Developer Portal:** https://developers.intasend.com/
- **API Documentation:** https://developers.intasend.com/docs/
- **Authentication Docs:** https://developers.intasend.com/docs/authentication
- **Send Money API:** https://developers.intasend.com/docs/send-money
- **Checkout Links:** https://developers.intasend.com/docs/checkout-links
- **Wallets API:** https://developers.intasend.com/docs/wallets
- **Error Codes:** https://developers.intasend.com/docs/errors-references
- **API Testing & Sandbox:** https://developers.intasend.com/docs/api-testing-and-sandbox
- **GitHub Documentation:** https://github.com/IntaSend/documentation
- **Python SDK:** https://github.com/IntaSend/intasend-python
- **PHP SDK:** https://github.com/IntaSend/intasend-php
- **Dashboard:** https://dashboard.intasend.com/
- **Status Page:** https://status.intasend.com/

---

## Support & Resources

- **Email:** support@intasend.com
- **Slack Community:** Join through developer portal
- **Discord:** Check IntaSend social channels
- **X/Twitter:** @IntaSend
- **Documentation Issues:** GitHub Issues on IntaSend/documentation

---

## Version History

- **v2.0** (2026-02-24): Major revision with production-verified endpoints, Bearer token auth, comprehensive examples
- **v1.0** (Earlier): Initial skill documentation
