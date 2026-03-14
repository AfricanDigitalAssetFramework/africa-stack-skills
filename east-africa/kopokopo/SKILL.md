---
name: Kopokopo
description: "Integrate with Kopokopo's M-Pesa payment APIs to build payment collections, disbursements, and settlements for Kenya. Use this skill for STK Push payment requests, receiving M-Pesa payments, sending payouts to customers, and transferring funds to bank accounts. Also trigger when user mentions 'Kopokopo', 'M-Pesa payments', 'STK Push', 'merchant payments', or needs payment integration for Kenya."
---

# Kopokopo Integration Skill

Kopokopo is Kenya's leading M-Pesa API platform enabling merchants to collect payments directly from customers' phones via STK Push, send payouts to M-Pesa wallets, and settle funds to bank accounts. It powers payment collections, merchant payouts, and business automation across Kenya with a production-grade REST API and webhooks.

## When to use this skill

Use Kopokopo when you need to:
- **Collect payments** from M-Pesa users in Kenya (marketplace checkouts, invoice payments, utility bills)
- **Send payouts** to merchants or customers in M-Pesa wallets
- **Transfer funds** to bank accounts for settlement with automatic reconciliation
- **Build payment flows** that eliminate customer navigation—STK Push displays payment prompts directly
- **Set up real-time payment notifications** via webhooks for accounting or order fulfillment
- **Integrate with existing systems** (Laravel, Node.js, Python, Ruby SDKs available)

Kopokopo's STK Push eliminates friction: customers see a payment prompt on their phone—no need to manually enter till numbers or navigate menus. This increases payment success rates and reduces cart abandonment.

## Authentication

Kopokopo uses **OAuth 2.0 with Client Credentials flow** for server-to-server authentication.

### Step 1: Request an OAuth Token

Exchange your Client ID and Client Secret for a Bearer token:

**Sandbox:**
```http
POST https://sandbox.kopokopo.com/oauth/token
Content-Type: application/x-www-form-urlencoded
```

**Production:**
```http
POST https://app.kopokopo.com/oauth/token
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET
```

**Response:**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "payment:write transfer:write"
}
```

### Step 2: Use the Token in API Requests

Include the access token in the `Authorization` header for all subsequent requests:

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
```

### Important Security Notes

- **Store credentials securely**: Use environment variables (never hardcode credentials)
- **Token caching**: Tokens expire in 3600 seconds (1 hour). Cache and refresh tokens before expiry
- **Separate sandbox/production**: Maintain separate credentials for sandbox and production environments
- **Rate limiting**: Implement exponential backoff when rate-limited (HTTP 429)

```bash
# Example: Store credentials securely
export KOPOKOPO_CLIENT_ID="your_client_id"
export KOPOKOPO_CLIENT_SECRET="your_client_secret"
export KOPOKOPO_BASE_URL="https://sandbox.kopokopo.com"  # Sandbox
# export KOPOKOPO_BASE_URL="https://app.kopokopo.com"   # Production
```

## Core API Reference

### Authentication: Request OAuth Token

Obtain a Bearer token for all API requests.

**Endpoint:**
```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded
```

**Request:**
```
grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET
```

**Response (200 OK):**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "payment:write transfer:write"
}
```

**Error Response (400):**
```json
{
  "error": "invalid_client",
  "error_description": "Client authentication failed"
}
```

---

### Receive Payments: M-Pesa STK Push

Request payment from a customer via M-Pesa STK Push. The customer's phone displays a payment prompt—they enter their PIN to authorize.

**Endpoint:**
```http
POST /api/v1/incoming_payments
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request:**
```json
{
  "phone_number": "+254712345678",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "amount": 1500,
  "currency": "KES",
  "description": "Payment for Order #ORD-12345",
  "merchant_reference": "ORD-12345",
  "callback_url": "https://yourapp.com/webhooks/payment"
}
```

**Field Descriptions:**
- `phone_number`: Customer's M-Pesa phone number (format: +254XXXXXXXXX)
- `first_name`, `last_name`: Customer details (optional but recommended)
- `email`: Customer email for receipt (optional)
- `amount`: Integer amount in KES (no decimals, minimum 100, maximum 150,000)
- `currency`: Always "KES" (Kenyan Shilling)
- `description`: Customer-facing payment description
- `merchant_reference`: Your unique transaction ID for tracking (max 100 chars)
- `callback_url`: (Optional) Webhook URL for payment notifications

**Response (201 Created):**
```json
{
  "data": {
    "id": "incoming_payment_8f9a7c3b2e1d4a5f",
    "status": "PENDING",
    "phone_number": "+254712345678",
    "first_name": "John",
    "last_name": "Doe",
    "amount": 1500,
    "currency": "KES",
    "merchant_reference": "ORD-12345",
    "description": "Payment for Order #ORD-12345",
    "created_at": "2025-02-24T10:30:00Z",
    "expires_at": "2025-02-24T10:31:00Z"
  }
}
```

**Timeline:**
- STK Push prompt appears on customer's phone immediately
- Customer has 30 seconds to enter their M-Pesa PIN
- After 30 seconds, STK expires and payment fails

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| 400 | `INVALID_PHONE` | Phone number format invalid (must be +254...) |
| 400 | `INVALID_AMOUNT` | Amount < 100 or > 150,000 KES |
| 401 | `UNAUTHORIZED` | Invalid or expired token |
| 403 | `INSUFFICIENT_BALANCE` | Your Kopokopo account has insufficient funds |
| 429 | `RATE_LIMITED` | Too many requests—implement exponential backoff |

---

### Check Payment Status

Retrieve the current status of a payment request.

**Endpoint:**
```http
GET /api/v1/incoming_payments/{id}
Authorization: Bearer {access_token}
```

**Response (200 OK) - Payment Pending:**
```json
{
  "data": {
    "id": "incoming_payment_8f9a7c3b2e1d4a5f",
    "status": "PENDING",
    "phone_number": "+254712345678",
    "amount": 1500,
    "currency": "KES",
    "merchant_reference": "ORD-12345",
    "created_at": "2025-02-24T10:30:00Z",
    "expires_at": "2025-02-24T10:31:00Z"
  }
}
```

**Response (200 OK) - Payment Completed:**
```json
{
  "data": {
    "id": "incoming_payment_8f9a7c3b2e1d4a5f",
    "status": "COMPLETED",
    "phone_number": "+254712345678",
    "amount": 1500,
    "currency": "KES",
    "merchant_reference": "ORD-12345",
    "mpesa_reference": "MU5C7H6YT2",
    "mpesa_receipt_number": "MPF12A45G78H",
    "payment_date": "2025-02-24T10:30:45Z",
    "created_at": "2025-02-24T10:30:00Z"
  }
}
```

**Response (200 OK) - Payment Failed:**
```json
{
  "data": {
    "id": "incoming_payment_8f9a7c3b2e1d4a5f",
    "status": "FAILED",
    "phone_number": "+254712345678",
    "amount": 1500,
    "currency": "KES",
    "merchant_reference": "ORD-12345",
    "failure_reason": "User Denied Transaction",
    "created_at": "2025-02-24T10:30:00Z"
  }
}
```

**Payment Statuses:**
- `PENDING`: Waiting for customer to authorize
- `COMPLETED`: Payment received and funds settled
- `FAILED`: Customer rejected or timeout (30 seconds elapsed)

---

### Send Money: Payout to M-Pesa

Send funds to a customer's M-Pesa wallet (referral bonuses, seller payouts, refunds).

**Endpoint:**
```http
POST /api/v1/pay
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request:**
```json
{
  "phone_number": "+254712345678",
  "amount": 1000,
  "currency": "KES",
  "reference": "PAYOUT-SELLER-001",
  "description": "Seller payout for completed orders",
  "callback_url": "https://yourapp.com/webhooks/payout"
}
```

**Field Descriptions:**
- `phone_number`: Recipient's M-Pesa phone (format: +254XXXXXXXXX)
- `amount`: Integer amount in KES (minimum 100, maximum 150,000)
- `currency`: Always "KES"
- `reference`: Your unique payout ID for tracking
- `description`: Customer-facing payout description (shown in M-Pesa receipt)
- `callback_url`: (Optional) Webhook URL for payout notifications

**Response (201 Created):**
```json
{
  "data": {
    "id": "payout_b4f2a8e7d1c9g5h3",
    "status": "PROCESSING",
    "phone_number": "+254712345678",
    "amount": 1000,
    "currency": "KES",
    "reference": "PAYOUT-SELLER-001",
    "description": "Seller payout for completed orders",
    "created_at": "2025-02-24T10:35:00Z"
  }
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| 400 | `INVALID_PHONE` | Phone number format invalid |
| 400 | `INVALID_AMOUNT` | Amount < 100 or > 150,000 KES |
| 401 | `UNAUTHORIZED` | Invalid or expired token |
| 403 | `INSUFFICIENT_BALANCE` | Your Kopokopo account has insufficient funds for payout |
| 429 | `RATE_LIMITED` | Too many requests |

**Payout Timeline:**
- Status changes to `PROCESSING` immediately
- Funds queued for M-Pesa network transmission (typically 1-5 seconds)
- Webhook fired when `COMPLETED` or `FAILED`
- Maximum processing time: 30 seconds

---

### Check Payout Status

Retrieve the current status of a payout.

**Endpoint:**
```http
GET /api/v1/pay/{id}
Authorization: Bearer {access_token}
```

**Response (200 OK) - Payout Completed:**
```json
{
  "data": {
    "id": "payout_b4f2a8e7d1c9g5h3",
    "status": "COMPLETED",
    "phone_number": "+254712345678",
    "amount": 1000,
    "currency": "KES",
    "reference": "PAYOUT-SELLER-001",
    "mpesa_reference": "MU5C7H6YT2",
    "mpesa_transaction_id": "MPF12A45G78H",
    "completed_at": "2025-02-24T10:35:03Z",
    "created_at": "2025-02-24T10:35:00Z"
  }
}
```

**Response (200 OK) - Payout Failed:**
```json
{
  "data": {
    "id": "payout_b4f2a8e7d1c9g5h3",
    "status": "FAILED",
    "phone_number": "+254712345678",
    "amount": 1000,
    "currency": "KES",
    "reference": "PAYOUT-SELLER-001",
    "failure_reason": "Invalid M-Pesa account",
    "created_at": "2025-02-24T10:35:00Z"
  }
}
```

**Payout Statuses:**
- `PROCESSING`: Queued for transmission
- `COMPLETED`: Funds delivered to recipient's M-Pesa account
- `FAILED`: Delivery failed (invalid number, M-Pesa errors, etc.)

---

### List Incoming Payments

Retrieve a paginated list of all payment requests (STK Push transactions).

**Endpoint:**
```http
GET /api/v1/incoming_payments?page=1&per_page=20&status=COMPLETED&from_date=2025-02-01&to_date=2025-02-28
Authorization: Bearer {access_token}
```

**Query Parameters:**
- `page`: Page number (1-indexed, default: 1)
- `per_page`: Records per page (max 100, default: 20)
- `status`: Filter by status (PENDING, COMPLETED, FAILED)
- `from_date`: ISO 8601 date (YYYY-MM-DD)
- `to_date`: ISO 8601 date (YYYY-MM-DD)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "incoming_payment_8f9a7c3b2e1d4a5f",
      "status": "COMPLETED",
      "phone_number": "+254712345678",
      "amount": 1500,
      "currency": "KES",
      "merchant_reference": "ORD-12345",
      "mpesa_reference": "MU5C7H6YT2",
      "mpesa_receipt_number": "MPF12A45G78H",
      "payment_date": "2025-02-24T10:30:45Z",
      "created_at": "2025-02-24T10:30:00Z"
    },
    {
      "id": "incoming_payment_7e8c6b4a3f2d1e9h",
      "status": "COMPLETED",
      "phone_number": "+254798765432",
      "amount": 2000,
      "currency": "KES",
      "merchant_reference": "ORD-12346",
      "mpesa_reference": "MU5C7H6YT3",
      "mpesa_receipt_number": "MPF12A45G78I",
      "payment_date": "2025-02-24T11:15:20Z",
      "created_at": "2025-02-24T11:14:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 147,
    "total_pages": 8
  }
}
```

---

### List Payouts

Retrieve a paginated list of all payouts sent.

**Endpoint:**
```http
GET /api/v1/pay?page=1&per_page=20&status=COMPLETED&from_date=2025-02-01&to_date=2025-02-28
Authorization: Bearer {access_token}
```

**Query Parameters:** (Same as incoming payments)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "payout_b4f2a8e7d1c9g5h3",
      "status": "COMPLETED",
      "phone_number": "+254712345678",
      "amount": 1000,
      "currency": "KES",
      "reference": "PAYOUT-SELLER-001",
      "mpesa_reference": "MU5C7H6YT2",
      "completed_at": "2025-02-24T10:35:03Z",
      "created_at": "2025-02-24T10:35:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 52,
    "total_pages": 3
  }
}
```

---

### Transfer Funds to Bank Account (Settle)

Transfer funds from your Kopokopo wallet to a pre-registered bank settlement account.

**Endpoint:**
```http
POST /api/v1/settlement_transfers
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request:**
```json
{
  "settlement_account_id": "settlement_acc_1a2b3c4d5e6f7g8h",
  "amount": 50000,
  "currency": "KES",
  "reference": "SETTLE-BANK-FEB-24",
  "description": "Monthly settlement to business account"
}
```

**Field Descriptions:**
- `settlement_account_id`: ID of pre-verified bank account (created via dashboard)
- `amount`: Integer amount in KES
- `currency`: Always "KES"
- `reference`: Your unique settlement ID for tracking
- `description`: Internal description for accounting

**Response (201 Created):**
```json
{
  "data": {
    "id": "settlement_transfer_9x8y7z6w5v4u3t2s",
    "status": "PROCESSING",
    "settlement_account_id": "settlement_acc_1a2b3c4d5e6f7g8h",
    "amount": 50000,
    "currency": "KES",
    "reference": "SETTLE-BANK-FEB-24",
    "created_at": "2025-02-24T14:00:00Z",
    "settlement_fee": 50
  }
}
```

**Settlement Details:**
- **Fee**: KSh 50 per transfer (fixed)
- **Processing time**: 1-3 business days (depends on receiving bank)
- **Minimum amount**: KSh 1,000
- **Pre-requirement**: Bank account must be verified in dashboard

---

### Check Settlement Transfer Status

**Endpoint:**
```http
GET /api/v1/settlement_transfers/{id}
Authorization: Bearer {access_token}
```

**Response (200 OK) - Settlement Completed:**
```json
{
  "data": {
    "id": "settlement_transfer_9x8y7z6w5v4u3t2s",
    "status": "COMPLETED",
    "settlement_account_id": "settlement_acc_1a2b3c4d5e6f7g8h",
    "amount": 50000,
    "currency": "KES",
    "settlement_fee": 50,
    "reference": "SETTLE-BANK-FEB-24",
    "bank_transaction_id": "BANK-TX-2025-02-24-001",
    "completed_at": "2025-02-26T10:30:00Z",
    "created_at": "2025-02-24T14:00:00Z"
  }
}
```

---

## Webhooks

Kopokopo sends webhooks to your registered URLs whenever payment or payout events occur. This enables real-time updates for order fulfillment, accounting, and notifications.

### Registering Webhook Endpoints

Webhooks are registered in your Kopokopo dashboard under **Settings > Webhooks**. Provide:
- **Webhook URL**: HTTPS endpoint that receives POST requests
- **Event types**: Select which events trigger webhooks (incoming_payment.completed, pay.completed, etc.)

### Verifying Webhook Signatures

Every webhook includes a signature for verification. Verify the webhook before processing:

**Webhook Headers:**
```http
X-Kopokopo-Signature: sha256=abcdef123456789...
X-Kopokopo-Request-Id: webhook_req_1a2b3c4d5e6f7g8h
X-Kopokopo-Timestamp: 2025-02-24T10:30:00Z
```

**Node.js Webhook Verification:**
```javascript
const crypto = require('crypto');

function verifyKopokokoWebhook(body, signature, webhookSecret) {
  const hash = crypto
    .createHmac('sha256', webhookSecret)
    .update(JSON.stringify(body))
    .digest('hex');

  const expectedSignature = `sha256=${hash}`;
  return crypto.timingSafeEqual(expectedSignature, signature);
}

// Express.js example
app.post('/webhooks/kopokopo', express.json(), (req, res) => {
  const signature = req.headers['x-kopokopo-signature'];
  const webhookSecret = process.env.KOPOKOPO_WEBHOOK_SECRET;

  if (!verifyKopokokoWebhook(req.body, signature, webhookSecret)) {
    return res.status(401).json({ error: 'Signature verification failed' });
  }

  // Process webhook
  handleKopokokoEvent(req.body);
  res.json({ status: 'received' });
});
```

**Python Webhook Verification:**
```python
import hmac
import hashlib
import json

def verify_kopokopo_webhook(body, signature, webhook_secret):
    hash_obj = hmac.new(
        webhook_secret.encode(),
        body.encode(),
        hashlib.sha256
    )
    expected_signature = f"sha256={hash_obj.hexdigest()}"
    return hmac.compare_digest(expected_signature, signature)

# Flask example
from flask import request

@app.route('/webhooks/kopokopo', methods=['POST'])
def handle_kopokopo_webhook():
    signature = request.headers.get('X-Kopokopo-Signature')
    webhook_secret = os.getenv('KOPOKOPO_WEBHOOK_SECRET')

    if not verify_kopokopo_webhook(request.get_data(), signature, webhook_secret):
        return {'error': 'Signature verification failed'}, 401

    body = request.get_json()
    handle_kopokopo_event(body)
    return {'status': 'received'}, 200
```

### Webhook Payload Examples

**Incoming Payment Completed:**
```json
{
  "event": "incoming_payment.completed",
  "request_id": "webhook_req_1a2b3c4d5e6f7g8h",
  "timestamp": "2025-02-24T10:30:45Z",
  "data": {
    "id": "incoming_payment_8f9a7c3b2e1d4a5f",
    "status": "COMPLETED",
    "phone_number": "+254712345678",
    "first_name": "John",
    "last_name": "Doe",
    "amount": 1500,
    "currency": "KES",
    "merchant_reference": "ORD-12345",
    "mpesa_reference": "MU5C7H6YT2",
    "mpesa_receipt_number": "MPF12A45G78H",
    "payment_date": "2025-02-24T10:30:45Z",
    "created_at": "2025-02-24T10:30:00Z"
  }
}
```

**Incoming Payment Failed:**
```json
{
  "event": "incoming_payment.failed",
  "request_id": "webhook_req_2b3c4d5e6f7g8h9i",
  "timestamp": "2025-02-24T10:31:00Z",
  "data": {
    "id": "incoming_payment_8f9a7c3b2e1d4a5f",
    "status": "FAILED",
    "phone_number": "+254712345678",
    "amount": 1500,
    "currency": "KES",
    "merchant_reference": "ORD-12345",
    "failure_reason": "User Denied Transaction",
    "created_at": "2025-02-24T10:30:00Z"
  }
}
```

**Payout Completed:**
```json
{
  "event": "pay.completed",
  "request_id": "webhook_req_3c4d5e6f7g8h9i0j",
  "timestamp": "2025-02-24T10:35:03Z",
  "data": {
    "id": "payout_b4f2a8e7d1c9g5h3",
    "status": "COMPLETED",
    "phone_number": "+254712345678",
    "amount": 1000,
    "currency": "KES",
    "reference": "PAYOUT-SELLER-001",
    "mpesa_reference": "MU5C7H6YT2",
    "mpesa_transaction_id": "MPF12A45G78H",
    "completed_at": "2025-02-24T10:35:03Z",
    "created_at": "2025-02-24T10:35:00Z"
  }
}
```

**Payout Failed:**
```json
{
  "event": "pay.failed",
  "request_id": "webhook_req_4d5e6f7g8h9i0j1k",
  "timestamp": "2025-02-24T10:35:05Z",
  "data": {
    "id": "payout_b4f2a8e7d1c9g5h3",
    "status": "FAILED",
    "phone_number": "+254712345678",
    "amount": 1000,
    "currency": "KES",
    "reference": "PAYOUT-SELLER-001",
    "failure_reason": "Invalid M-Pesa account",
    "created_at": "2025-02-24T10:35:00Z"
  }
}
```

### Webhook Best Practices

- **Idempotency**: Process webhooks idempotently using `request_id` as a deduplication key (Kopokopo may retry failed deliveries)
- **Response timing**: Respond with 200 OK immediately; process asynchronously
- **Retry logic**: Kopokopo retries failed webhooks exponentially (1s, 5s, 30s, 5m, 30m)
- **Timeout**: Respond within 30 seconds
- **HTTPS only**: Webhook URLs must use HTTPS
- **Logging**: Log all webhook requests and responses for debugging

---

## Common Integration Patterns

### Pattern 1: E-Commerce Payment Collection

Collect payments for online purchases with order confirmation and shipment integration.

**Flow:**
1. Customer clicks "Pay" on checkout page
2. Backend requests OAuth token
3. Backend initiates STK Push with order details
4. Customer's phone displays M-Pesa payment prompt
5. Customer enters PIN and confirms
6. Webhook fires with payment confirmation
7. Backend marks order as paid and triggers shipment

**Code Example (Node.js):**
```javascript
const axios = require('axios');

async function createOrder(orderId, customerPhone, totalAmount) {
  // Get access token
  const tokenResponse = await axios.post(
    'https://sandbox.kopokopo.com/oauth/token',
    `grant_type=client_credentials&client_id=${process.env.KOPOKOPO_CLIENT_ID}&client_secret=${process.env.KOPOKOPO_CLIENT_SECRET}`,
    { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } }
  );

  const accessToken = tokenResponse.data.access_token;

  // Send STK Push for payment
  const paymentResponse = await axios.post(
    'https://sandbox.kopokopo.com/api/v1/incoming_payments',
    {
      phone_number: customerPhone,
      amount: totalAmount,
      currency: 'KES',
      merchant_reference: orderId,
      description: `Payment for Order ${orderId}`,
      callback_url: 'https://yourapp.com/webhooks/payment'
    },
    { headers: { 'Authorization': `Bearer ${accessToken}` } }
  );

  // Save payment request ID for tracking
  const paymentId = paymentResponse.data.data.id;
  await db.updateOrder(orderId, { kopokopo_payment_id: paymentId, status: 'AWAITING_PAYMENT' });

  return paymentId;
}

// Webhook handler
app.post('/webhooks/payment', express.json(), async (req, res) => {
  const { data: payment } = req.body;

  if (payment.status === 'COMPLETED') {
    const orderId = payment.merchant_reference;
    await db.updateOrder(orderId, { status: 'PAID', mpesa_reference: payment.mpesa_reference });
    await triggerShipment(orderId);
  } else if (payment.status === 'FAILED') {
    const orderId = payment.merchant_reference;
    await db.updateOrder(orderId, { status: 'PAYMENT_FAILED' });
  }

  res.json({ status: 'received' });
});
```

### Pattern 2: Marketplace Seller Payout

Collect buyer payments, deduct commission, and payout seller's share.

**Flow:**
1. Buyer pays via STK Push (e.g., 10,000 KES)
2. Platform deducts commission (10%, 1,000 KES)
3. Seller receives 9,000 KES via payout
4. Payout webhook confirms delivery
5. Seller's wallet updated in real-time

**Code Example (Python):**
```python
import requests
import os
from datetime import datetime

def process_marketplace_transaction(buyer_phone, seller_phone, order_amount):
    # Get access token
    token_response = requests.post(
        'https://sandbox.kopokopo.com/oauth/token',
        data={
            'grant_type': 'client_credentials',
            'client_id': os.getenv('KOPOKOPO_CLIENT_ID'),
            'client_secret': os.getenv('KOPOKOPO_CLIENT_SECRET')
        }
    )
    access_token = token_response.json()['access_token']
    headers = {'Authorization': f'Bearer {access_token}'}

    # Step 1: Collect payment from buyer
    payment_response = requests.post(
        'https://sandbox.kopokopo.com/api/v1/incoming_payments',
        json={
            'phone_number': buyer_phone,
            'amount': order_amount,
            'currency': 'KES',
            'merchant_reference': f'ORDER-{datetime.now().timestamp()}',
            'description': f'Payment for marketplace order'
        },
        headers=headers
    )

    payment_id = payment_response.json()['data']['id']

    # Calculate seller payout (after 10% commission)
    commission_rate = 0.10
    commission = int(order_amount * commission_rate)
    seller_payout = order_amount - commission

    # Step 2: Send payout to seller (will be triggered by payment webhook)
    payout_response = requests.post(
        'https://sandbox.kopokopo.com/api/v1/pay',
        json={
            'phone_number': seller_phone,
            'amount': seller_payout,
            'currency': 'KES',
            'reference': f'SELLER-PAYOUT-{payment_id}',
            'description': f'Order proceeds after {commission_rate*100}% commission'
        },
        headers=headers
    )

    return {
        'payment_id': payment_id,
        'payout_id': payout_response.json()['data']['id'],
        'order_amount': order_amount,
        'commission': commission,
        'seller_payout': seller_payout
    }
```

### Pattern 3: Real-Time Accounting & Reconciliation

Every payment and payout updates ledger and balance in real-time.

**Flow:**
1. Payment webhook received and verified
2. Debit/credit transaction recorded in journal
3. Account balance updated instantly
4. Reconciliation report automatically regenerated
5. Financial dashboard reflects latest state

**Database Schema Example (SQL):**
```sql
CREATE TABLE transactions (
  id VARCHAR(64) PRIMARY KEY,
  type ENUM('PAYMENT', 'PAYOUT', 'SETTLEMENT') NOT NULL,
  status ENUM('PENDING', 'COMPLETED', 'FAILED') NOT NULL,
  amount INT NOT NULL,
  currency VARCHAR(3) NOT NULL,
  phone_number VARCHAR(20),
  merchant_reference VARCHAR(100),
  mpesa_reference VARCHAR(50),
  kopokopo_timestamp TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  webhook_request_id VARCHAR(64) UNIQUE,  -- Deduplication key
  INDEX idx_merchant_ref (merchant_reference),
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);

CREATE TABLE account_balance (
  account_id VARCHAR(64) PRIMARY KEY,
  balance_ksh BIGINT NOT NULL,
  last_transaction_id VARCHAR(64),
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

**Webhook Handler (PHP):**
```php
<?php
function handleKopokokoWebhook($event_data, $request_id) {
    // Idempotency check
    $existing = db_query("SELECT id FROM transactions WHERE webhook_request_id = ?", [$request_id]);
    if ($existing) {
        return ['status' => 'already_processed'];
    }

    $payment = $event_data['data'];
    $type = strpos($event_data['event'], 'incoming_payment') ? 'PAYMENT' : 'PAYOUT';

    // Record transaction
    db_insert('transactions', [
        'id' => $payment['id'],
        'type' => $type,
        'status' => $payment['status'],
        'amount' => $payment['amount'],
        'currency' => $payment['currency'],
        'phone_number' => $payment['phone_number'],
        'merchant_reference' => $payment['merchant_reference'] ?? null,
        'mpesa_reference' => $payment['mpesa_reference'] ?? null,
        'kopokopo_timestamp' => $payment['payment_date'] ?? $payment['completed_at'],
        'webhook_request_id' => $request_id
    ]);

    // Update balance if completed
    if ($payment['status'] === 'COMPLETED') {
        $adjustment = ($type === 'PAYMENT') ? $payment['amount'] : -$payment['amount'];
        db_query("UPDATE account_balance SET balance_ksh = balance_ksh + ? WHERE account_id = ?",
                 [$adjustment, 'primary_account']);
    }

    return ['status' => 'processed'];
}
?>
```

### Pattern 4: Bill Payment Collection (Utilities, Subscriptions)

Collect recurring or one-time bill payments with automated reconciliation.

**Flow:**
1. Monthly bill created for customer
2. Automated notification sent to customer (SMS/Email) with payment instruction
3. Admin initiates STK Push for bill amount
4. Customer pays; webhook triggers
5. Webhook marks bill as paid
6. Invoice automatically generated and emailed
7. Overdue bills list updated

---

## Error Handling

Kopokopo returns structured error responses. Always handle errors gracefully.

### HTTP Status Codes & Error Responses

| Status | Error Code | Meaning | Action |
|--------|-----------|---------|--------|
| 400 | `INVALID_PHONE` | Phone number format invalid | Validate phone format (+254XXXXXXXXX) |
| 400 | `INVALID_AMOUNT` | Amount out of range (< 100 or > 150,000) | Adjust amount or inform user |
| 400 | `INVALID_CURRENCY` | Only KES supported | Use KES currency |
| 400 | `MISSING_REQUIRED_FIELD` | Required field missing in request | Check request payload |
| 401 | `UNAUTHORIZED` | Invalid or expired Bearer token | Request new token |
| 401 | `INVALID_CLIENT` | Client ID/Secret authentication failed | Verify credentials |
| 403 | `INSUFFICIENT_BALANCE` | Kopokopo account balance too low | Add funds to account |
| 403 | `FORBIDDEN` | Insufficient permissions for operation | Check account settings |
| 404 | `NOT_FOUND` | Payment/Payout ID not found | Verify transaction ID |
| 429 | `RATE_LIMITED` | Too many requests in short period | Implement exponential backoff |
| 500 | `INTERNAL_ERROR` | Server error | Retry with exponential backoff |

### Error Response Format

```json
{
  "status": "error",
  "code": "INVALID_PHONE",
  "message": "Phone number must be in format +254XXXXXXXXX",
  "request_id": "req_1a2b3c4d5e6f7g8h"
}
```

### Retry Strategy

Implement exponential backoff for transient failures (5xx, 429):

```javascript
async function retryableRequest(method, url, data, maxRetries = 3) {
  let attempt = 0;
  while (attempt < maxRetries) {
    try {
      return await axios({ method, url, data });
    } catch (error) {
      if (error.response?.status >= 500 || error.response?.status === 429) {
        const delayMs = Math.pow(2, attempt) * 1000 + Math.random() * 1000;
        await new Promise(resolve => setTimeout(resolve, delayMs));
        attempt++;
      } else {
        throw error; // Don't retry 4xx errors (except 429)
      }
    }
  }
  throw new Error(`Max retries exceeded after ${maxRetries} attempts`);
}
```

---

## Important Notes & Gotchas

### Phone Numbers
- **Format required**: Must include country code (+254 for Kenya)
- **Invalid formats**: 0712345678 (missing +254), +1234 (wrong country), 254712345678 (missing +)
- **Test numbers**: Sandbox provides test phone numbers for development—check dashboard

### Amounts
- **No decimals**: Amounts must be integers (1500, not 1500.50)
- **Whole KES only**: All amounts in Kenyan Shillings
- **Range limits**: Minimum 100 KES, Maximum 150,000 KES per transaction
- **Account balance**: Ensure sufficient funds before sending payouts

### STK Push Behavior
- **Expiration**: 30-second timeout if customer doesn't enter PIN
- **One-time use**: Each STK Push is single-use (failed push doesn't prompt again)
- **Network timing**: May take 1-2 seconds for prompt to appear on customer's phone
- **Test mode**: Use sandbox test phone numbers provided in dashboard

### Webhooks
- **Idempotency**: Webhooks may be retried multiple times; process idempotently using `request_id`
- **Delivery guarantees**: At-least-once delivery (not exactly-once)
- **HTTPS required**: Only HTTPS URLs accepted for webhooks
- **Timeout**: Respond within 30 seconds
- **Network failures**: If your endpoint is unreachable, Kopokopo retries exponentially

### Authentication
- **Token caching**: Tokens valid for 1 hour; cache and reuse to avoid per-request auth overhead
- **Token refresh**: Pre-emptively refresh tokens near expiry (e.g., at 50 minutes)
- **Never hardcode**: Store Client ID/Secret in environment variables, never in code
- **Sandbox vs Production**: Maintain separate credentials for development and production

### Rate Limiting
- **Limits**: Contact Kopokopo for specific rate limits (typically 100+ requests/minute)
- **Headers**: Check `X-RateLimit-*` headers in responses
- **Backoff strategy**: Implement exponential backoff starting at 1 second
- **Monitoring**: Log 429 responses to identify rate limit issues

### Settlement & Bank Transfers
- **Pre-registration**: Bank accounts must be verified in dashboard before settlement API calls
- **Settlement fee**: Fixed 50 KSh per transfer (plus potential bank fees)
- **Processing time**: 1-3 business days for funds to appear in bank account
- **Minimum amount**: 1,000 KES per transfer

### Common Pitfalls to Avoid
1. **Hardcoding credentials**: Always use environment variables
2. **Ignoring webhook signatures**: Always verify `X-Kopokopo-Signature`
3. **Not deduplicating webhooks**: Webhooks can be retried; use `request_id` for idempotency
4. **Phone format errors**: Test phone validation thoroughly
5. **Expired tokens**: Implement automatic token refresh
6. **Not handling 429 errors**: Implement proper rate limit backoff
7. **Assuming synchronous transactions**: Always listen for webhooks; don't poll status repeatedly

---

## Useful Links

- **Official API Docs**: [https://developers.kopokopo.com/](https://developers.kopokopo.com/)
- **API Reference**: [https://api-docs.kopokopo.com/](https://api-docs.kopokopo.com/)
- **Dashboard**: [https://app.kopokopo.com/](https://app.kopokopo.com/)
- **Sandbox Dashboard**: [https://sandbox.kopokopo.com/](https://sandbox.kopokopo.com/)
- **Support & FAQs**: [https://kopokopoinc.zendesk.com/](https://kopokopoinc.zendesk.com/)
- **GitHub SDKs**: [https://github.com/kopokopo](https://github.com/kopokopo)
  - [Node.js SDK](https://github.com/kopokopo/k2-connect-node)
  - [Python SDK](https://github.com/kopokopo/k2-connect-python)
  - [PHP SDK](https://github.com/kopokopo/k2-connect-php)
  - [Ruby SDK](https://github.com/kopokopo/k2-connect-ruby)
  - [Flutter SDK](https://github.com/kopokopo/k2-connect-flutter)
