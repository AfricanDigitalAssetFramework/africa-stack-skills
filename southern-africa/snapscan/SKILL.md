---
name: snapscan
description: "Integrate with SnapScan for QR code payment processing in South Africa. Use this skill whenever the user wants to accept QR code payments, build contactless payment solutions, generate SnapScan payment codes, verify transactions, query payment status, manage merchant operations, or work with SnapScan's merchant API in any way. Also trigger when the user mentions 'SnapScan', 'QR code payments South Africa', 'SnapScan checkout', 'contactless payments', or needs QR payment integration."
---

# SnapScan Integration Skill

SnapScan is South Africa's leading QR code payment platform, owned by Standard Bank. It enables contactless transactions through QR codes and provides a merchant API for generating dynamic QR codes, tracking payments, and integrating SnapScan payments into applications. SnapScan handles settlement instantly with live payment notifications.

## When to use this skill

You're building a solution that needs to accept contactless QR code payments in South Africa — a marketplace, a POS system, a bill payment platform, a retail operation, or any system requiring QR-based transactions. SnapScan provides instant settlement and real-time payment tracking.

## Authentication

SnapScan uses HTTP Basic Auth for API authentication:

**Authentication Method:**
```
Authorization: Basic base64(api_key:)
```

Where the API key is base64-encoded followed by an empty password (colon with nothing after it).

**JavaScript Example:**
```javascript
const apiKey = process.env.SNAPSCAN_API_KEY;
const auth = 'Basic ' + Buffer.from(apiKey + ':').toString('base64');
const headers = {
  'Authorization': auth,
  'Content-Type': 'application/json'
};
```

**cURL Example:**
```bash
curl -u your-api-key: "https://pos.snapscan.io/merchant/api/v1/payments"
```

**Ruby Example:**
```ruby
require 'net/http'
uri = URI('https://pos.snapscan.io/merchant/api/v1/payments')
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true
request = Net::HTTP::Get.new(uri.request_uri)
request.basic_auth(api_key, '')
```

**Important Security Notes:**
- Store your API key in environment variables (`SNAPSCAN_API_KEY`) — never hardcode credentials
- All requests must use HTTPS; HTTP requests will fail with 401 Unauthorized
- The password field in Basic Auth is empty; only include the API key

**Base URL:** `https://pos.snapscan.io/merchant/api/v1`

## Core API Reference

### Create a Payment (Dynamic QR Code)

Initiate a QR code payment request for a specific amount. Returns a QR code that expires after 60 minutes.

```
POST /payments
```

**Request Body:**
```json
{
  "amount": 19999,
  "currency": "ZAR",
  "description": "Purchase of product XYZ",
  "merchantReference": "ORDER-12345",
  "externalReference": "INV-12345",
  "notifyUrl": "https://yoursite.com/payment/notify",
  "redirectUrl": "https://yoursite.com/payment/redirect"
}
```

**Field Descriptions:**
- `amount` (required): Amount in cents (ZAR). Example: R199.99 = 19999 cents
- `currency` (required): "ZAR" (only supported currency)
- `description` (optional): Payment description shown to customer
- `merchantReference` (optional): Your internal transaction ID for reconciliation
- `externalReference` (optional): External reference like invoice or order number
- `notifyUrl` (optional): Webhook URL for payment notifications
- `redirectUrl` (optional): Where to redirect customer after payment

**Successful Response (201 Created):**
```json
{
  "id": "pay_abc123def456",
  "qrCodeUrl": "https://snapscan.io/pay/abc123def456",
  "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAMgCAIAAA...",
  "merchantReference": "ORDER-12345",
  "externalReference": "INV-12345",
  "amount": 19999,
  "currency": "ZAR",
  "description": "Purchase of product XYZ",
  "status": "PENDING",
  "expiresAt": "2026-02-24T12:00:00Z",
  "createdAt": "2026-02-24T11:00:00Z"
}
```

**Usage Notes:**
- Display the `qrCode` as an embedded PNG image or redirect to `qrCodeUrl`
- QR code remains valid for 60 minutes from creation
- Use `merchantReference` to prevent duplicate transactions
- Webhooks deliver real-time payment status updates

### Get Payment Status

Retrieve the current status of a specific payment.

```
GET /payments/{id}
```

**URL Parameters:**
- `{id}`: Payment ID from the create payment response (e.g., `pay_abc123def456`)

**Response (Pending Payment):**
```json
{
  "id": "pay_abc123def456",
  "merchantReference": "ORDER-12345",
  "externalReference": "INV-12345",
  "amount": 19999,
  "currency": "ZAR",
  "description": "Purchase of product XYZ",
  "status": "PENDING",
  "qrCodeUrl": "https://snapscan.io/pay/abc123def456",
  "expiresAt": "2026-02-24T12:00:00Z",
  "createdAt": "2026-02-24T11:00:00Z"
}
```

**Response (Completed Payment):**
```json
{
  "id": "pay_abc123def456",
  "merchantReference": "ORDER-12345",
  "externalReference": "INV-12345",
  "amount": 19999,
  "currency": "ZAR",
  "description": "Purchase of product XYZ",
  "status": "COMPLETED",
  "qrCodeUrl": "https://snapscan.io/pay/abc123def456",
  "paidAt": "2026-02-24T11:05:00Z",
  "createdAt": "2026-02-24T11:00:00Z",
  "payer": {
    "name": "John Doe",
    "phoneNumber": "+27712345678",
    "userId": "snap_user_789"
  }
}
```

**Possible Status Values:**
- `PENDING`: Waiting for customer to scan and pay
- `COMPLETED`: Payment successful
- `FAILED`: Payment declined or processing error
- `CANCELLED`: Customer cancelled the payment
- `EXPIRED`: QR code expired after 60 minutes without payment

### Query Payments by Reference

Search for payments using merchant or external reference with pagination support.

```
GET /payments?merchantReference={ref}&limit=20&offset=0
```

OR

```
GET /payments?externalReference={ref}&limit=20&offset=0
```

**Query Parameters:**
- `merchantReference` (optional): Your internal reference (e.g., ORDER-12345)
- `externalReference` (optional): External reference like invoice ID
- `limit` (optional): Results per page (default: 20, max: 100)
- `offset` (optional): Number of results to skip for pagination

**Response:**
```json
{
  "data": [
    {
      "id": "pay_abc123def456",
      "merchantReference": "ORDER-12345",
      "externalReference": "INV-12345",
      "amount": 19999,
      "currency": "ZAR",
      "status": "COMPLETED",
      "paidAt": "2026-02-24T11:05:00Z",
      "createdAt": "2026-02-24T11:00:00Z",
      "payer": {
        "name": "John Doe",
        "phoneNumber": "+27712345678"
      }
    }
  ],
  "paging": {
    "limit": 20,
    "offset": 0,
    "total": 1
  }
}
```

### Create Static QR Code

Generate a static, reusable QR code for fixed locations (like in-store displays). Customers can scan anytime to pay any amount.

```
POST /qr_codes
```

**Request Body:**
```json
{
  "description": "In-store payments - Counter 1",
  "destination": "https://yoursite.com/snapscan-webhook"
}
```

**Field Descriptions:**
- `description` (optional): Label for this QR code (for your reference)
- `destination` (optional): Webhook URL to receive payment notifications

**Response:**
```json
{
  "id": "qr_xyz789abc123",
  "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAMgCAIAAA...",
  "qrCodeUrl": "https://snapscan.io/qr/xyz789abc123",
  "description": "In-store payments - Counter 1",
  "destination": "https://yoursite.com/snapscan-webhook",
  "createdAt": "2026-02-24T11:00:00Z"
}
```

**Usage Notes:**
- Print or display the `qrCode` PNG in your physical store
- Static QR codes do NOT expire — they're permanent
- Customers can scan repeatedly and pay any amount
- Each scan triggers a new `payment.created` event with unique payment ID

### List Static QR Codes

Retrieve all static QR codes created for your merchant account.

```
GET /qr_codes?limit=20&offset=0
```

**Query Parameters:**
- `limit` (optional): Results per page (default: 20, max: 100)
- `offset` (optional): Results to skip for pagination

**Response:**
```json
{
  "data": [
    {
      "id": "qr_xyz789abc123",
      "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAMgCAIAAA...",
      "qrCodeUrl": "https://snapscan.io/qr/xyz789abc123",
      "description": "In-store payments - Counter 1",
      "destination": "https://yoursite.com/snapscan-webhook",
      "createdAt": "2026-02-24T11:00:00Z"
    },
    {
      "id": "qr_abc456def789",
      "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAMgCAIAAA...",
      "qrCodeUrl": "https://snapscan.io/qr/abc456def789",
      "description": "In-store payments - Counter 2",
      "createdAt": "2026-02-24T10:30:00Z"
    }
  ],
  "paging": {
    "limit": 20,
    "offset": 0,
    "total": 2
  }
}
```

## Webhooks

SnapScan sends real-time payment notifications via webhooks. Validate webhook signatures to ensure authenticity.

**Webhook Events:**
- `payment.created` — Dynamic QR code generated (waiting for payment)
- `payment.completed` — Payment completed successfully
- `payment.failed` — Payment failed or QR expired
- `payment.cancelled` — Customer cancelled the payment

**Webhook Payload Example (payment.completed):**
```json
{
  "event": "payment.completed",
  "data": {
    "id": "pay_abc123def456",
    "merchantReference": "ORDER-12345",
    "externalReference": "INV-12345",
    "amount": 19999,
    "currency": "ZAR",
    "description": "Purchase of product XYZ",
    "status": "COMPLETED",
    "paidAt": "2026-02-24T11:05:00Z",
    "createdAt": "2026-02-24T11:00:00Z",
    "payer": {
      "name": "John Doe",
      "phoneNumber": "+27712345678",
      "userId": "snap_user_789"
    }
  },
  "timestamp": "2026-02-24T11:05:00Z"
}
```

### Webhook Signature Verification

SnapScan signs webhook payloads using HMAC-SHA256. The signature is sent in the `X-Snapscan-Signature` header as a hexadecimal string.

**Verification Process:**
1. Get the webhook payload (raw JSON body)
2. Get the signature from the `X-Snapscan-Signature` header
3. Compute HMAC-SHA256 of the raw JSON body using your webhook secret
4. Compare computed signature with header signature (use constant-time comparison)
5. If signatures match, the webhook is authentic

**JavaScript Example:**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(rawBody, signature, webhookSecret) {
  const computed = crypto
    .createHmac('sha256', webhookSecret)
    .update(rawBody)
    .digest('hex');

  // Use constant-time comparison to prevent timing attacks
  return crypto.timingSafeEqual(
    Buffer.from(computed),
    Buffer.from(signature)
  );
}

// Express.js middleware example
app.post('/webhook/snapscan', (req, res) => {
  const signature = req.headers['x-snapscan-signature'];
  const rawBody = req.rawBody; // Store raw body before JSON parsing

  if (!verifyWebhookSignature(rawBody, signature, process.env.SNAPSCAN_WEBHOOK_SECRET)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const webhook = JSON.parse(rawBody);

  if (webhook.event === 'payment.completed') {
    // Process successful payment
    console.log(`Payment ${webhook.data.id} completed for ${webhook.data.amount} ZAR`);
  }

  res.json({ success: true });
});
```

**Ruby Example:**
```ruby
require 'openssl'
require 'digest'

def verify_webhook_signature(raw_body, signature, webhook_secret)
  computed = OpenSSL::HMAC.hexdigest(
    OpenSSL::Digest::SHA256.new,
    webhook_secret,
    raw_body
  )

  Rack::Utils.secure_compare(computed, signature)
end

# Rails example
class WebhooksController < ApplicationController
  skip_forgery_protection

  def snapscan
    signature = request.headers['X-Snapscan-Signature']
    raw_body = request.raw_post

    unless verify_webhook_signature(raw_body, signature, ENV['SNAPSCAN_WEBHOOK_SECRET'])
      render json: { error: 'Invalid signature' }, status: :unauthorized
      return
    end

    webhook = JSON.parse(raw_body)

    if webhook['event'] == 'payment.completed'
      # Process successful payment
      payment = webhook['data']
      Order.find_by(reference: payment['merchantReference']).mark_paid!
    end

    render json: { success: true }
  end
end
```

**Python Example:**
```python
import hmac
import hashlib
import json

def verify_webhook_signature(raw_body, signature, webhook_secret):
    computed = hmac.new(
        webhook_secret.encode(),
        raw_body if isinstance(raw_body, bytes) else raw_body.encode(),
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(computed, signature)

# Flask example
from flask import request, jsonify

@app.route('/webhook/snapscan', methods=['POST'])
def snapscan_webhook():
    signature = request.headers.get('X-Snapscan-Signature')
    raw_body = request.get_data()

    if not verify_webhook_signature(raw_body, signature, os.getenv('SNAPSCAN_WEBHOOK_SECRET')):
        return jsonify({'error': 'Invalid signature'}), 401

    webhook = json.loads(raw_body)

    if webhook['event'] == 'payment.completed':
        payment = webhook['data']
        order = Order.query.filter_by(reference=payment['merchantReference']).first()
        if order:
            order.mark_paid()

    return jsonify({'success': True})
```

**Important Security Notes:**
- Always use raw request body (before JSON parsing) for signature computation
- Store webhook secret securely in environment variables
- Use constant-time comparison (`timingSafeEqual`, `secure_compare`, `hmac.compare_digest`)
- Webhooks without valid signatures should be rejected
- Never trust webhook data without signature verification — they're an untrusted event stream

## Common Integration Patterns

### Pattern 1: In-Store Static QR Code Payment

For a fixed retail location where customers scan a permanent QR code.

**Setup:**
1. Create static QR code: `POST /qr_codes`
2. Print/display the QR code image in store
3. Set up webhook endpoint to receive notifications

**Flow:**
```
Customer scans printed QR code
↓
Opens SnapScan app (or web) to enter amount
↓
Completes payment
↓
Your webhook receives payment.completed event
↓
Fulfill order/ring register
```

**Code Example (Node.js):**
```javascript
const express = require('express');
const app = express();

// 1. Create static QR code once during setup
async function setupInStoreQR() {
  const response = await fetch(
    'https://pos.snapscan.io/merchant/api/v1/qr_codes',
    {
      method: 'POST',
      headers: {
        'Authorization': `Basic ${Buffer.from(apiKey + ':').toString('base64')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        description: 'In-store payments - Till 1',
        destination: 'https://yoursite.com/webhook/snapscan'
      })
    }
  );

  const qrData = await response.json();
  console.log('QR Code URL:', qrData.qrCodeUrl);
  // Print qrData.qrCode (PNG image) to display in store
}

// 2. Receive webhook when customer pays
app.post('/webhook/snapscan', (req, res) => {
  const webhook = req.body;

  if (webhook.event === 'payment.completed') {
    const payment = webhook.data;
    console.log(`Received payment: ${payment.amount} ZAR from ${payment.payer.name}`);
    // Update POS system, ring register, fulfill order
  }

  res.json({ received: true });
});
```

### Pattern 2: Online Checkout with Dynamic QR Code

For e-commerce where each order gets a unique, time-limited QR code.

**Flow:**
```
Customer adds items to cart
↓
Proceeds to checkout
↓
You create dynamic QR code: POST /payments with amount
↓
Display QR + "Scan with SnapScan app"
↓
Customer scans QR on phone
↓
SnapScan app shows payment screen
↓
Customer confirms payment
↓
Webhook notifies you (or poll status)
↓
Redirect to success page
```

**Code Example (Node.js + React):**
```javascript
// Backend: Create dynamic QR code
app.post('/api/checkout', async (req, res) => {
  const { amount, orderId } = req.body;

  const response = await fetch(
    'https://pos.snapscan.io/merchant/api/v1/payments',
    {
      method: 'POST',
      headers: {
        'Authorization': `Basic ${Buffer.from(apiKey + ':').toString('base64')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        amount: amount * 100, // Convert to cents
        currency: 'ZAR',
        merchantReference: orderId,
        description: `Order ${orderId}`,
        notifyUrl: 'https://yoursite.com/webhook/snapscan'
      })
    }
  );

  const payment = await response.json();

  res.json({
    paymentId: payment.id,
    qrCode: payment.qrCode,
    expiresAt: payment.expiresAt
  });
});

// Frontend: Display QR code
function CheckoutPage({ orderId, amount }) {
  const [qrCode, setQrCode] = useState(null);
  const [status, setStatus] = useState('pending');

  useEffect(() => {
    // Create payment
    fetch('/api/checkout', {
      method: 'POST',
      body: JSON.stringify({ amount, orderId })
    })
      .then(r => r.json())
      .then(data => {
        setQrCode(data.qrCode);

        // Poll for payment status
        const pollInterval = setInterval(async () => {
          const statusRes = await fetch(`/api/payment/${data.paymentId}`);
          const payment = await statusRes.json();

          if (payment.status === 'COMPLETED') {
            setStatus('completed');
            clearInterval(pollInterval);
            // Redirect to success
          }
        }, 2000);
      });
  }, [orderId, amount]);

  return (
    <div>
      <h2>Scan with SnapScan</h2>
      {qrCode && <img src={qrCode} alt="SnapScan QR Code" />}
      <p>Amount: R{(amount / 100).toFixed(2)}</p>
      <p>Status: {status}</p>
    </div>
  );
}
```

### Pattern 3: Invoice/Bill Payment

Include QR code on invoices so customers can scan and pay directly.

**Flow:**
```
Generate invoice for customer
↓
Create dynamic QR code with invoice amount
↓
Embed QR code image on PDF invoice
↓
Send to customer via email/print
↓
Customer scans QR code
↓
Payment amount pre-filled in SnapScan
↓
Customer completes payment
↓
Webhook marks invoice as paid
```

**Code Example (Node.js with PDF generation):**
```javascript
const PDFDocument = require('pdfkit');

async function generateInvoice(invoiceData) {
  const { invoiceId, amount, customerName } = invoiceData;

  // Create dynamic QR code
  const paymentResponse = await fetch(
    'https://pos.snapscan.io/merchant/api/v1/payments',
    {
      method: 'POST',
      headers: {
        'Authorization': `Basic ${Buffer.from(apiKey + ':').toString('base64')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        amount: Math.round(amount * 100),
        currency: 'ZAR',
        merchantReference: invoiceId,
        externalReference: `INV-${invoiceId}`,
        description: `Invoice ${invoiceId}`
      })
    }
  );

  const payment = await paymentResponse.json();

  // Create PDF with QR code
  const doc = new PDFDocument();
  doc.pipe(fs.createWriteStream(`invoice-${invoiceId}.pdf`));

  doc.fontSize(20).text(`Invoice ${invoiceId}`);
  doc.fontSize(12).text(`Customer: ${customerName}`);
  doc.text(`Amount: R${(amount).toFixed(2)}`);

  // Add QR code to PDF
  const qrBuffer = Buffer.from(payment.qrCode.split(',')[1], 'base64');
  doc.image(qrBuffer, 100, 200, { width: 200 });
  doc.text('Scan to pay with SnapScan', 100, 450);

  doc.end();
}
```

### Pattern 4: Polling-Based Confirmation (No Webhooks)

For scenarios where webhooks aren't available; periodically check payment status.

**Flow:**
```
Create dynamic QR code: POST /payments
↓
Display QR to customer
↓
Start polling payment status every 2 seconds
↓
When status = COMPLETED, fulfill order
↓
Timeout after 10 minutes if not completed
```

**Code Example (Node.js):**
```javascript
async function pollPaymentStatus(paymentId, maxWaitSeconds = 600) {
  const startTime = Date.now();
  const pollInterval = 2000; // 2 seconds

  return new Promise((resolve, reject) => {
    const interval = setInterval(async () => {
      try {
        const response = await fetch(
          `https://pos.snapscan.io/merchant/api/v1/payments/${paymentId}`,
          {
            headers: {
              'Authorization': `Basic ${Buffer.from(apiKey + ':').toString('base64')}`
            }
          }
        );

        const payment = await response.json();

        if (payment.status === 'COMPLETED') {
          clearInterval(interval);
          resolve(payment);
        } else if (payment.status === 'FAILED' || payment.status === 'CANCELLED') {
          clearInterval(interval);
          reject(new Error(`Payment ${payment.status}`));
        } else if (Date.now() - startTime > maxWaitSeconds * 1000) {
          clearInterval(interval);
          reject(new Error('Payment timeout'));
        }
      } catch (error) {
        clearInterval(interval);
        reject(error);
      }
    }, pollInterval);
  });
}

// Usage
app.post('/api/create-payment', async (req, res) => {
  const { amount, orderId } = req.body;

  const paymentResponse = await fetch(
    'https://pos.snapscan.io/merchant/api/v1/payments',
    {
      method: 'POST',
      headers: {
        'Authorization': `Basic ${Buffer.from(apiKey + ':').toString('base64')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        amount: amount * 100,
        currency: 'ZAR',
        merchantReference: orderId
      })
    }
  );

  const payment = await paymentResponse.json();

  // Start polling in background
  pollPaymentStatus(payment.id)
    .then(completedPayment => {
      console.log('Payment completed:', completedPayment);
      // Fulfill order
    })
    .catch(error => {
      console.error('Payment failed:', error);
      // Handle failure
    });

  res.json({
    paymentId: payment.id,
    qrCode: payment.qrCode
  });
});
```

## Error Handling

SnapScan returns HTTP status codes and error details in JSON format.

**Error Response Format:**
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Invalid amount",
    "details": "Amount must be greater than 0 and less than 999999999"
  }
}
```

**HTTP Status Codes and Meanings:**

| Status | Code | Meaning | Action |
|--------|------|---------|--------|
| 400 | `INVALID_REQUEST` | Validation error in request body | Check `details` field and correct the request |
| 401 | `UNAUTHORIZED` | Invalid or missing API key | Verify `SNAPSCAN_API_KEY` environment variable |
| 403 | `FORBIDDEN` | API key lacks required permissions | Contact SnapScan support |
| 404 | `NOT_FOUND` | Payment or QR code doesn't exist | Verify the ID is correct |
| 429 | `RATE_LIMITED` | Too many requests | Implement exponential backoff and retry |
| 500 | `SERVER_ERROR` | SnapScan API error | Retry with exponential backoff |

**Common Validation Errors:**
- `amount` must be integer > 0 and < 999999999
- `currency` must be "ZAR"
- `merchantReference` must be unique (prevent duplicates)
- `notifyUrl` must be valid HTTPS URL

**Payment Status Failures:**
- `EXPIRED`: QR code exceeded 60-minute lifetime
- `FAILED`: Payment declined, insufficient funds, or processing error
- `CANCELLED`: Customer explicitly cancelled payment

**Error Handling Best Practices:**

```javascript
async function createPaymentWithRetry(amount, orderId, maxRetries = 3) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(
        'https://pos.snapscan.io/merchant/api/v1/payments',
        {
          method: 'POST',
          headers: {
            'Authorization': `Basic ${Buffer.from(apiKey + ':').toString('base64')}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            amount: amount * 100,
            currency: 'ZAR',
            merchantReference: orderId
          })
        }
      );

      if (response.status === 429 || response.status >= 500) {
        // Rate limited or server error — retry with backoff
        const delay = Math.pow(2, attempt - 1) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }

      if (!response.ok) {
        const error = await response.json();
        if (response.status === 400 || response.status === 403) {
          // Client error — don't retry
          throw new Error(`${error.error.code}: ${error.error.message}`);
        }
      }

      return await response.json();
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${attempt} failed:`, error.message);
    }
  }

  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}
```

## Important Implementation Notes

### 1. Amount Precision
Always use integers for cents to avoid floating-point errors. Never use decimals.
```javascript
const amountInRands = 199.99;
const amountInCents = Math.round(amountInRands * 100); // 19999 (correct)
```

### 2. Static vs Dynamic QR Codes

**Static QR Codes** (`POST /qr_codes`):
- Created once and reused indefinitely
- Don't expire (permanent)
- Customer enters amount at payment time
- Ideal for in-store, printed displays
- Used for "pay any amount" scenarios
- Each scan creates a new `payment.created` event

**Dynamic QR Codes** (`POST /payments`):
- Created per transaction with specific amount
- Expire after 60 minutes
- Amount pre-filled in payment screen
- Ideal for invoices, online checkout, bills
- Customer scans to confirm and pay that specific amount
- Single-use per QR code

### 3. Idempotency and Duplicate Prevention

Use `merchantReference` to prevent duplicate charges from network failures or retries:
```javascript
const merchantReference = `ORDER-${orderId}-${Date.now()}`;
// If request fails and retries, the same reference ensures idempotent behavior
```

### 4. Webhook Validation

Never trust webhook data without signature verification:
```javascript
// WRONG — unsafe, trusts untrusted data
app.post('/webhook', (req, res) => {
  const payment = req.body.data;
  fulfillOrder(payment); // Dangerous!
});

// CORRECT — verify signature first
app.post('/webhook', (req, res) => {
  if (!verifyWebhookSignature(req.rawBody, req.headers['x-snapscan-signature'])) {
    return res.status(401).send('Invalid signature');
  }
  const payment = req.body.data;
  fulfillOrder(payment); // Safe
});
```

### 5. Polling Timeout

If polling payment status, always set a timeout (typically 10 minutes) to avoid indefinite waits:
```javascript
const MAX_POLL_SECONDS = 600; // 10 minutes
const POLL_INTERVAL_MS = 2000;

// Check payment status with timeout
```

### 6. Mobile-First Design

SnapScan is optimized for mobile payments:
- QR codes should display prominently on mobile screens
- Ensure checkout pages are mobile-responsive
- Test QR code scannability with various phone cameras
- Avoid very small QR code sizes (minimum 100x100 pixels)

### 7. Currency Support

Currently only **ZAR (South African Rand)** is supported. Amounts are always in cents (1 ZAR = 100 cents).

### 8. Rate Limiting

SnapScan API implements rate limiting. Respect 429 responses with exponential backoff:
```javascript
// Backoff: 1s, 2s, 4s, 8s, 16s...
const delay = Math.pow(2, retryCount - 1) * 1000;
```

### 9. Settlement and Fees

- SnapScan charges per transaction (check merchant dashboard for current rates)
- Settlement typically daily to linked bank account
- No additional fees for static QR codes
- Real-time payment notifications via webhooks

### 10. Test Merchant Credentials

For testing before production:
1. Create a test merchant account in SnapScan sandbox
2. Use test API keys for all requests
3. Payments in test mode are simulated — no real money transfers
4. Test mode is disabled in production environment

## Useful Links

- **Official API Documentation**: [SnapScan Merchant API](https://developer.getsnapscan.com/)
- **SnapScan Developer Portal**: [developer.snapscan.co.za](https://developer.snapscan.co.za/)
- **Merchant Dashboard**: [merchant.snapscan.io](https://merchant.snapscan.io/)
- **SnapScan Status Page**: [status.snapscan.io](https://status.snapscan.io/)
- **API Tracker**: [SnapScan on API Tracker](https://apitracker.io/a/snapscan-za)
- **Standard Bank Integration**: [Standard Bank API Marketplace](https://developer.standardbank.com/APIMarketplace/s/communityapi/a6tTr00000002OPIAY/api-marketplacesnapscanqrcodepayments)
