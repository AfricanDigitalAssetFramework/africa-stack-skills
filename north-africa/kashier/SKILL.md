---
name: kashier
description: "Integrate with Kashier, Egypt's leading payment processor. Use this skill whenever you need to process payments in Egypt, accept EGP transactions, create payment orders, verify payments, handle refunds, or integrate Kashier's API into Egyptian apps. Trigger when user mentions 'Kashier', 'Egyptian payment processing', 'EGP payments', 'payment gateway Egypt', or needs payment solutions for Egypt-based businesses."
---

# Kashier Payment Processor Integration

Kashier is Egypt's leading digital payment platform, enabling businesses to accept online payments in EGP (Egyptian Pound) and multiple currencies. It provides REST API endpoints for creating payment orders, handling redirects/iFrames, verifying transactions, and managing refunds with webhook support.

## When to Use This Skill

Use Kashier integration when you need to:
- Process payments from Egyptian customers or businesses
- Accept EGP and multi-currency payments (USD, GBP, EUR)
- Build online checkout flows with hosted pages or iFrame integration
- Handle refunds and payment reconciliation
- Integrate payment processing into Egyptian e-commerce, SaaS, or marketplace platforms
- Set up webhook listeners for payment notifications

Kashier supports multiple payment methods: credit cards, debit cards, mobile wallets, and bank installments.

## Authentication

Kashier uses API key-based authentication with credentials obtained from the merchant dashboard:

**Credentials Required:**
- **Merchant ID (MID)**: Unique identifier for your merchant account
- **Payment API Key**: Used for order hashing and signature generation (HMAC-SHA256)
- **Secret Key**: Used for API request authentication and webhook verification

**Authentication Header:**
```
Authorization: {SECRET_KEY}
Content-Type: application/json
```

Store credentials in environment variables:
```
KASHIER_MERCHANT_ID=your_merchant_id
KASHIER_API_KEY=your_payment_api_key
KASHIER_SECRET_KEY=your_secret_key
```

**Base URLs:**
- Test (Sandbox): `https://test-api.kashier.io`, `https://test-iframe.kashier.io`
- Production (Live): `https://api.kashier.io`, `https://iframe.kashier.io`

**Getting Your Keys:**
1. Log in to [Kashier Dashboard](https://portal.kashier.io/)
2. Navigate to "Integrate now" → "Payment Api Keys"
3. Generate test and live keys separately
4. Toggle "Live Data" switch to switch between test and production environments

## Core API Reference

### Payment Initialization (Order Hash Generation)

Before initiating any payment, generate an order hash using HMAC-SHA256. This validates the order data and prevents tampering.

**Order Hash Generation:**
```javascript
const crypto = require('crypto');

function generateOrderHash(merchantId, orderId, amount, apiKey) {
  const hashString = `${merchantId}${orderId}${amount}`;
  return crypto
    .createHmac('sha256', apiKey)
    .update(hashString)
    .digest('hex');
}

// Example
const hash = generateOrderHash(
  'MERCHANT123',
  'ORD-2025-12345',
  50000,
  'your_api_key'
);
```

**Request Body (backend):**
```json
{
  "amount": 50000,
  "currency": "EGP",
  "orderId": "ORD-2025-12345",
  "merchantId": "MERCHANT123",
  "orderHash": "generated_hash_above",
  "customerEmail": "customer@example.com",
  "customerPhone": "+201001234567",
  "customerName": "Ahmed Hassan",
  "items": [
    {
      "name": "Product Name",
      "description": "Product Description",
      "quantity": 1,
      "unitPrice": 50000
    }
  ]
}
```

### Payment UI (iFrame) Integration

Embed a checkout popup within your page for a seamless payment experience.

**iFrame Script Implementation (client-side):**
```html
<script src="https://test-iframe.kashier.io/kashier.js"></script>

<button
  class="kashier-button"
  data-merchant-id="MERCHANT123"
  data-order-id="ORD-2025-12345"
  data-amount="50000"
  data-currency="EGP"
  data-description="Order description"
  data-order-hash="generated_hash_above"
  data-merchant-redirect="https://yoursite.com/payment/success"
  data-display="hidden">
  Pay Now
</button>
```

**iFrame Parameters:**
- `data-merchant-id`: Your merchant ID
- `data-order-id`: Unique order identifier
- `data-amount`: Amount in piasters (see conversion below)
- `data-currency`: EGP, USD, GBP, or EUR
- `data-description`: Order description for display
- `data-order-hash`: HMAC-SHA256 hash from above
- `data-merchant-redirect`: Success callback URL
- `data-display`: hidden | visible (button display mode)

**Response Flow:**
Customer clicks the button → iFrame popup appears → Customer completes payment → Browser redirects to `data-merchant-redirect` with parameters:
- `id`: Payment transaction ID
- `status`: Payment status

### Hosted Payment Page (Redirect)

Generate a redirect URL to Kashier's hosted checkout page for a full-page payment experience.

**Backend Endpoint:**
```
POST /api/initPayment
```

**Request (backend):**
```json
{
  "merchantId": "MERCHANT123",
  "orderId": "ORD-2025-12345",
  "amount": 50000,
  "currency": "EGP",
  "orderHash": "generated_hash_above",
  "customerEmail": "customer@example.com",
  "customerPhone": "+201001234567",
  "customerName": "Ahmed Hassan",
  "callbackUrl": "https://yoursite.com/payment/callback"
}
```

**Response:**
```json
{
  "success": true,
  "paymentUrl": "https://iframe.kashier.io/pay?id=UNIQUE_PAYMENT_ID&redirectUrl=...",
  "paymentId": "pay_xxxxx"
}
```

**Client-side (redirect):**
```javascript
// Redirect customer to payment URL
window.location.href = response.paymentUrl;
```

### Payment Status Verification

Query payment status after customer returns from checkout.

**GET Request:**
```
GET /api/payment?merchantId=MERCHANT123&orderId=ORD-2025-12345
Authorization: {SECRET_KEY}
```

**Response:**
```json
{
  "success": true,
  "id": "pay_xxxxx",
  "orderId": "ORD-2025-12345",
  "amount": 50000,
  "currency": "EGP",
  "status": "completed",
  "paid": true,
  "paidAmount": 50000,
  "paymentMethod": "card",
  "cardType": "visa",
  "cardLast4": "4081",
  "transactionId": "txn_ref_xxxxx",
  "createdAt": "2025-02-24T10:00:00Z",
  "completedAt": "2025-02-24T10:05:00Z"
}
```

**Status Values:**
- `pending`: Payment awaiting processing
- `completed`: Payment successful
- `failed`: Payment declined
- `refunded`: Payment refunded

### Refund Payment

Issue full or partial refunds for completed payments.

**POST Request:**
```
POST /api/refund
Authorization: {SECRET_KEY}
Content-Type: application/json
```

**Request:**
```json
{
  "merchantId": "MERCHANT123",
  "orderId": "ORD-2025-12345",
  "paymentId": "pay_xxxxx",
  "amount": 50000,
  "reason": "Customer requested refund",
  "refundId": "REF-2025-001"
}
```

Omit `amount` field for full refund. Include specific amount (in piasters) for partial refund.

**Response:**
```json
{
  "success": true,
  "refundId": "ref_xxxxx",
  "paymentId": "pay_xxxxx",
  "amount": 50000,
  "status": "completed",
  "createdAt": "2025-02-24T10:15:00Z"
}
```

### List Transactions

Retrieve paginated list of transactions with filtering.

**GET Request:**
```
GET /api/transactions?merchantId=MERCHANT123&status=completed&from=2025-02-01&to=2025-02-28&page=1&pageSize=50
Authorization: {SECRET_KEY}
```

**Query Parameters:**
- `status`: pending, completed, failed, refunded
- `from`, `to`: ISO date format (YYYY-MM-DD)
- `page`: Page number (default: 1)
- `pageSize`: Records per page (default: 50, max: 100)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "pay_xxxxx",
      "orderId": "ORD-2025-12345",
      "amount": 50000,
      "currency": "EGP",
      "status": "completed",
      "paymentMethod": "card",
      "createdAt": "2025-02-24T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "total": 150
  }
}
```

## Webhooks

Kashier sends real-time notifications for payment events to your configured webhook URL.

**Webhook Configuration:**
1. Log into Kashier Dashboard
2. Go to Settings → Webhooks
3. Add your webhook endpoint URL
4. Kashier will POST events to this URL

**Webhook Events:**
- `transaction.completed`: Payment successful
- `transaction.failed`: Payment failed
- `transaction.refunded`: Refund processed
- `payment.pending`: Payment pending confirmation

**Webhook Payload:**
```json
{
  "event": "transaction.completed",
  "timestamp": "2025-02-24T10:05:00Z",
  "data": {
    "id": "pay_xxxxx",
    "orderId": "ORD-2025-12345",
    "amount": 50000,
    "currency": "EGP",
    "status": "completed",
    "paid": true,
    "paymentMethod": "card",
    "transactionId": "txn_ref_xxxxx"
  }
}
```

**Webhook Verification (Node.js):**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secretKey) {
  const hash = crypto
    .createHmac('sha256', secretKey)
    .update(JSON.stringify(payload))
    .digest('hex');
  return hash === signature;
}

// In your webhook handler
app.post('/webhook/kashier', (req, res) => {
  const signature = req.headers['x-kashier-signature'];

  if (!verifyWebhookSignature(req.body, signature, process.env.KASHIER_SECRET_KEY)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook
  console.log('Valid webhook:', req.body.event);
  res.status(200).json({ received: true });
});
```

**Webhook Response:**
Always respond with HTTP 200 within 30 seconds. Kashier will retry failed webhooks up to 5 times with exponential backoff.

## Common Integration Patterns

### E-commerce Checkout Flow

1. **Backend: Generate Order Hash**
   ```javascript
   const hash = generateOrderHash(merchantId, orderId, amount, apiKey);
   // Store hash in session or pass to frontend
   ```

2. **Frontend: Initiate Payment (iFrame)**
   ```html
   <button
     class="kashier-button"
     data-merchant-id="MERCHANT123"
     data-order-id="ORD-2025-12345"
     data-amount="50000"
     data-order-hash="generated_hash">
     Pay with Kashier
   </button>
   ```

3. **Backend: Verify Payment on Callback**
   ```javascript
   app.get('/payment/success', async (req, res) => {
     const { id, status } = req.query;
     const payment = await verifyPayment(id);

     if (payment.status === 'completed') {
       // Fulfill order
       fulfillOrder(orderId);
     }
     res.redirect('/order/confirmation');
   });
   ```

### Hosted Page Payment Flow

1. **Backend: Create Payment Session**
   ```javascript
   const hash = generateOrderHash(merchantId, orderId, amount, apiKey);
   const paymentUrl = `https://iframe.kashier.io/pay?merchantId=${merchantId}&orderId=${orderId}&amount=${amount}&hash=${hash}&redirectUrl=...`;
   ```

2. **Frontend: Redirect Customer**
   ```javascript
   window.location.href = paymentUrl;
   ```

3. **Backend: Handle Callback**
   ```javascript
   app.post('/payment/callback', (req, res) => {
     const { orderId, status } = req.body;

     if (status === 'completed') {
       updateOrderStatus(orderId, 'paid');
     }
     res.status(200).send('OK');
   });
   ```

### Handle Refunds

```javascript
async function issueRefund(orderId, paymentId, refundAmount) {
  const response = await fetch('https://api.kashier.io/api/refund', {
    method: 'POST',
    headers: {
      'Authorization': process.env.KASHIER_SECRET_KEY,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      merchantId: process.env.KASHIER_MERCHANT_ID,
      orderId: orderId,
      paymentId: paymentId,
      amount: refundAmount // omit for full refund
    })
  });

  const data = await response.json();
  if (data.success) {
    updateOrderStatus(orderId, 'refunded');
  }
  return data;
}
```

## Error Handling

Kashier returns error responses with HTTP status codes and error details.

**Error Response Format:**
```json
{
  "success": false,
  "error": "Description of what went wrong",
  "code": "ERROR_CODE"
}
```

**Common Error Codes:**

| HTTP | Code | Description | Action |
|------|------|-------------|--------|
| 400 | INVALID_PARAMETERS | Missing or invalid request fields | Validate request body |
| 400 | INVALID_HASH | Order hash doesn't match | Regenerate hash with correct credentials |
| 401 | UNAUTHORIZED | Invalid API key or secret | Verify credentials in dashboard |
| 404 | NOT_FOUND | Payment or order not found | Check orderId and paymentId |
| 422 | MERCHANT_ERROR | Business logic error (e.g., duplicate orderId) | Check order exists, use unique IDs |
| 429 | RATE_LIMIT | Too many requests | Implement exponential backoff |
| 500 | SERVER_ERROR | Kashier server error | Retry with exponential backoff |

**Handling Errors:**
```javascript
try {
  const response = await verifyPayment(orderId);
} catch (error) {
  if (error.code === 'INVALID_HASH') {
    // Regenerate hash
  } else if (error.code === 'NOT_FOUND') {
    // Order doesn't exist yet - redirect to create payment
  } else if (error.statusCode === 429) {
    // Wait before retrying
  } else {
    // Log and notify user
  }
}
```

## Important Notes / Gotchas

### Amount Format
- **All amounts are in piasters** (smallest unit of EGP)
- 1 EGP = 100 piasters
- Example: 50 EGP = 5,000 piasters
- Always convert before sending to API: `amountInEGP * 100 = amountInPiasters`

### Currency Support
Kashier supports: **EGP** (Egyptian Pound), **USD**, **GBP**, **EUR**

### Order ID Uniqueness
- Order IDs must be unique per merchant
- Kashier rejects duplicate orderId requests with `MERCHANT_ERROR`
- Implement idempotency checks on your backend

### Customer Phone Format
- Use international format: `+201XXXXXXXXX` (Egypt +20 country code)
- Local numbers like `01XXXXXXXXX` may not work reliably

### Hash Generation Security
- **NEVER** generate hashes on the client-side
- Always hash on the backend with your Payment API Key
- Include Merchant ID, Order ID, and Amount in hash (exact order matters)
- Use HMAC-SHA256 algorithm

### Test vs Production
- Test keys use `https://test-api.kashier.io` and `https://test-iframe.kashier.io`
- Production keys use `https://api.kashier.io` and `https://iframe.kashier.io`
- Generate separate test and live keys in dashboard
- Test with test cards first before going live

### Webhook Security
- Always verify webhook signatures using the Secret Key
- Never trust webhook data without signature verification
- Implement webhook retry logic (Kashier retries up to 5 times)
- Respond with 200 status within 30 seconds

### Common Issues
- **Hash mismatch**: Ensure hash string format is `{merchantId}{orderId}{amount}` with no separators
- **Unauthorized errors**: Double-check API key and Secret Key are not swapped
- **Payment not reflecting**: Check webhook is configured and receiving events
- **iFrame not loading**: Verify merchant ID and order hash are correct

## Useful Links

- **Official Site**: https://www.kashier.io/
- **Developer Portal**: https://developers.kashier.io/
- **Merchant Dashboard**: https://portal.kashier.io/
- **Integration Guide**: https://www.kashier.io/docs/integration-guide
- **API Keys Setup**: https://developers.kashier.io/gettingstarted/apikeys/
- **Webhooks Docs**: https://developers.kashier.io/payment/webhook/
- **Postman Collection**: https://www.postman.com/kashierpayments/kashier-api
- **GitHub Examples**: https://github.com/Kashier-payments
- **Support Email**: techsupport@kashier.io
