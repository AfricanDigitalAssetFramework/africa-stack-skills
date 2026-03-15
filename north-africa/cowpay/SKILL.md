---
name: cowpay
description: "Integrate with Cowpay Egyptian payment gateway for accepting payments via Fawry bill payment, credit cards, cash collection, and installments. Use this skill whenever integrating Cowpay, building Egyptian payment solutions, processing EGP transactions, creating Fawry charges, handling card payments, managing cash on delivery, checking payment status, processing refunds, or implementing payment webhooks. Trigger on 'Cowpay', 'Egyptian payment gateway', 'Fawry integration', 'pay at Fawry', 'EGP payments', or 'Cowpay API'."
---

# Cowpay Payment Gateway Integration

Cowpay is a flexible payment gateway enabling Egyptian businesses to accept multiple payment methods including Fawry bill payments, credit cards, cash collection (pay on delivery), and installment plans. The API supports full transaction management with webhooks, refunds, and real-time payment status tracking in Egyptian Pounds (EGP).

## When to Use This Skill

Use Cowpay when you need to:
- Accept payments from Egyptian customers across multiple payment methods
- Build e-commerce checkout systems for Egyptian merchants
- Implement Fawry bill payment integration (customers pay at kiosks)
- Process direct card payments with 3D Secure
- Enable cash on delivery with payment collection
- Handle installment payment plans
- Create payment tracking dashboards
- Manage refunds and payment disputes
- Set up webhook listeners for real-time payment updates

Perfect for Egyptian startups, e-commerce platforms, SaaS products, marketplaces, and any business needing flexible Egyptian payment options.

## Authentication

Cowpay uses **signature-based authentication** with merchant credentials and HMAC-SHA256 signing.

### Credentials Required

```
Merchant Code: YOUR_MERCHANT_CODE
Merchant Hash Key: YOUR_MERCHANT_HASH_KEY
API Key: YOUR_API_KEY (if applicable)
```

Store in environment variables:
```bash
COWPAY_MERCHANT_CODE=your_merchant_code
COWPAY_MERCHANT_HASH=your_merchant_hash_key
COWPAY_API_KEY=your_api_key
```

### Signature Generation

Generate HMAC-SHA256 signature for each request:

```javascript
const crypto = require('crypto');

function generateCowpaySignature(merchantCode, merchantRefId, customerId, paymentMethod, amount, merchantHash) {
  const data = merchantCode + merchantRefId + customerId + paymentMethod + amount + merchantHash;
  return crypto.createHash('sha256').update(data).digest('hex');
}

// Example
const signature = generateCowpaySignature(
  'MERCHANT123',
  'ORD-2025-12345',
  'CUST-001',
  'PAYATFAWRY',
  50000,
  'YOUR_MERCHANT_HASH'
);
```

### Base URLs

| Environment | URL |
|---|---|
| **Production** | `https://cowpay.me/api/` |
| **Staging** | `https://staging.cowpay.me/api/` |

Always use staging for testing before production deployment.

## Core API Reference

### Create Fawry Charge (Bill Payment)

Create a Fawry bill payment charge. Customer receives a reference number and pays at any Fawry kiosk nationwide.

```
POST /fawry/charge-request
Content-Type: application/x-www-form-urlencoded
```

**Request Parameters:**

```
merchant_code=MERCHANT123
merchant_reference_id=ORD-2025-12345
customer_merchant_profile_id=CUST-001
customer_name=Ahmed Hassan
customer_mobile=+201234567890
customer_email=customer@email.com
payment_method=PAYATFAWRY
amount=50000
currency_code=EGP
description=Electronics purchase
charge_items=[{"description":"Laptop","amount":45000,"quantity":1},{"description":"Shipping","amount":5000,"quantity":1}]
signature=<HMAC_SHA256_SIGNATURE>
```

**Response (Success):**

```json
{
  "type": "CHARGE",
  "reference_number": "FAWRY123456789",
  "merchant_reference_id": "ORD-2025-12345",
  "amount": 50000,
  "currency_code": "EGP",
  "status_code": 200,
  "status_description": "Request processed successfully",
  "payment_gateway_reference_id": "PG-2025-xxx",
  "expires_at": "2025-02-25T23:59:59Z",
  "created_at": "2025-02-23T10:00:00Z"
}
```

**Important:**
- Amount is in **piasters** (1 EGP = 100 piasters)
- 500 EGP = 50000 piasters
- Customer has 24-48 hours to pay
- `reference_number` is what customer sees and uses at Fawry kiosks

### Create Card Charge

Process direct credit card payment with tokenization for PCI compliance.

```
POST /card/charge-request
Content-Type: application/x-www-form-urlencoded
```

**Request Parameters:**

```
merchant_code=MERCHANT123
merchant_reference_id=ORD-2025-12346
customer_merchant_profile_id=CUST-002
customer_name=Fatima Hassan
customer_mobile=+201234567890
customer_email=customer@email.com
payment_method=CREDITCARD
amount=50000
currency_code=EGP
card_token=tok_visa_xxxxx
card_cvv=123
card_expiry_month=12
card_expiry_year=26
description=Subscription renewal
signature=<HMAC_SHA256_SIGNATURE>
```

**Response (Success):**

```json
{
  "type": "CHARGE",
  "reference_number": "CARD-123456789",
  "merchant_reference_id": "ORD-2025-12346",
  "amount": 50000,
  "currency_code": "EGP",
  "status_code": 200,
  "status_description": "Card charged successfully",
  "payment_gateway_reference_id": "PG-2025-yyy",
  "transaction_reference": "TXN-2025-001",
  "card_last_four": "4081",
  "card_type": "visa",
  "completed_at": "2025-02-23T10:05:00Z",
  "created_at": "2025-02-23T10:00:00Z"
}
```

**Security Notes:**
- Never handle raw card data directly
- Implement 3D Secure for card authentication
- Use tokenized cards whenever possible
- PCI DSS compliance is mandatory

### Create Cash Collection Charge (COD)

Create a cash collection order for pay-on-delivery scenarios.

```
POST /cashcollection/charge-request
Content-Type: application/x-www-form-urlencoded
```

**Request Parameters:**

```
merchant_code=MERCHANT123
merchant_reference_id=ORD-2025-12347
customer_merchant_profile_id=CUST-003
customer_name=Salma Mohamed
customer_mobile=+201234567890
customer_email=customer@email.com
payment_method=CASHCOLLECTION
amount=50000
currency_code=EGP
delivery_address=123 Main St, Cairo, 11111
delivery_governorate=Cairo
description=Electronics purchase with cash delivery
signature=<HMAC_SHA256_SIGNATURE>
```

**Response (Success):**

```json
{
  "type": "CHARGE",
  "reference_number": "COD-123456789",
  "merchant_reference_id": "ORD-2025-12347",
  "amount": 50000,
  "currency_code": "EGP",
  "status_code": 200,
  "status_description": "Cash collection order created",
  "payment_gateway_reference_id": "PG-2025-zzz",
  "collection_reference": "COLLECT-2025-001",
  "status": "pending_delivery",
  "delivery_address": "123 Main St, Cairo, 11111",
  "created_at": "2025-02-23T10:00:00Z"
}
```

**Flow:**
1. Create COD charge
2. Ship product to customer
3. Cash collected at delivery
4. Webhook confirms `payment.completed`
5. Mark order as fully paid

### Check Payment Status

Query the status of any charge by merchant reference number.

```
GET /payment/status
```

**Query Parameters:**

```
merchant_code=MERCHANT123
merchant_reference_id=ORD-2025-12345
signature=<HMAC_SHA256_SIGNATURE>
```

**Response:**

```json
{
  "type": "PAYMENT_STATUS",
  "merchant_reference_id": "ORD-2025-12345",
  "reference_number": "FAWRY123456789",
  "amount": 50000,
  "currency_code": "EGP",
  "status_code": 200,
  "status_description": "Success",
  "payment_status": "completed",
  "payment_method": "PAYATFAWRY",
  "payment_gateway_reference_id": "PG-2025-xxx",
  "paid_at": "2025-02-24T14:30:00Z",
  "created_at": "2025-02-23T10:00:00Z"
}
```

**Possible Payment Statuses:**
- `pending` - Charge created, awaiting payment
- `processing` - Payment in progress
- `completed` - Payment received and confirmed
- `failed` - Payment failed (declined card, expired Fawry, etc.)
- `cancelled` - Transaction cancelled by merchant or customer
- `expired` - Fawry reference expired (24-48 hours)
- `refunded` - Refund processed

### Refund Payment

Issue full or partial refund for completed charge.

```
POST /refund/charge-request
Content-Type: application/x-www-form-urlencoded
```

**Request Parameters:**

```
merchant_code=MERCHANT123
original_merchant_reference_id=ORD-2025-12345
refund_merchant_reference_id=REF-2025-12345
amount=50000
currency_code=EGP
reason=Customer requested refund
refund_items=[{"description":"Product refund","amount":50000,"quantity":1}]
signature=<HMAC_SHA256_SIGNATURE>
```

**Response (Success):**

```json
{
  "type": "REFUND",
  "reference_number": "REFUND-123456789",
  "refund_merchant_reference_id": "REF-2025-12345",
  "original_merchant_reference_id": "ORD-2025-12345",
  "amount": 50000,
  "currency_code": "EGP",
  "status_code": 200,
  "status_description": "Refund processed successfully",
  "payment_gateway_reference_id": "PG-2025-ref",
  "refund_status": "completed",
  "completed_at": "2025-02-23T10:10:00Z",
  "created_at": "2025-02-23T10:10:00Z"
}
```

**Notes:**
- Omit `amount` to refund full transaction
- Include `amount` for partial refunds (in piasters)
- Refunds can only be issued for completed charges
- Processing takes 1-3 business days for Fawry refunds

## Webhooks and Callbacks

Cowpay sends real-time notifications for payment events. Configure callback URL in dashboard.

### Webhook Signature Verification

Always verify webhook signatures to prevent spoofing:

```javascript
const crypto = require('crypto');

function verifyCowpayWebhook(payload, signature, merchantHash) {
  // Reconstruct the data that was signed
  const data = payload.merchant_code +
               payload.merchant_reference_id +
               payload.reference_number +
               payload.amount +
               merchantHash;

  const calculated = crypto.createHash('sha256').update(data).digest('hex');

  return calculated === signature;
}

// Express.js example
app.post('/webhooks/cowpay', (req, res) => {
  const signature = req.headers['x-request-signature'] || req.body.signature;

  if (!verifyCowpayWebhook(req.body, signature, process.env.COWPAY_MERCHANT_HASH)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook
  handleCowpayEvent(req.body);
  res.json({ success: true });
});
```

### Webhook Event Types

**charge.completed**

```json
{
  "event_type": "charge.completed",
  "merchant_code": "MERCHANT123",
  "merchant_reference_id": "ORD-2025-12345",
  "reference_number": "FAWRY123456789",
  "payment_gateway_reference_id": "PG-2025-xxx",
  "amount": 50000,
  "currency_code": "EGP",
  "payment_method": "PAYATFAWRY",
  "status": "completed",
  "paid_at": "2025-02-24T14:30:00Z",
  "signature": "abc123..."
}
```

**charge.failed**

```json
{
  "event_type": "charge.failed",
  "merchant_code": "MERCHANT123",
  "merchant_reference_id": "ORD-2025-12346",
  "reference_number": "CARD-123456789",
  "amount": 50000,
  "currency_code": "EGP",
  "payment_method": "CREDITCARD",
  "status": "failed",
  "failure_reason": "Card declined by issuer",
  "failure_code": "CARD_DECLINED",
  "failed_at": "2025-02-23T10:05:00Z",
  "signature": "abc123..."
}
```

**collection.completed**

```json
{
  "event_type": "collection.completed",
  "merchant_code": "MERCHANT123",
  "merchant_reference_id": "ORD-2025-12347",
  "reference_number": "COD-123456789",
  "collection_reference": "COLLECT-2025-001",
  "amount": 50000,
  "currency_code": "EGP",
  "status": "completed",
  "collected_at": "2025-02-25T16:45:00Z",
  "signature": "abc123..."
}
```

**refund.completed**

```json
{
  "event_type": "refund.completed",
  "merchant_code": "MERCHANT123",
  "original_merchant_reference_id": "ORD-2025-12345",
  "refund_merchant_reference_id": "REF-2025-12345",
  "reference_number": "REFUND-123456789",
  "amount": 50000,
  "currency_code": "EGP",
  "status": "completed",
  "refund_completed_at": "2025-02-24T10:00:00Z",
  "signature": "abc123..."
}
```

### Webhook Best Practices

1. **Idempotency:** Use `merchant_reference_id` to prevent duplicate processing
2. **Timeouts:** Respond with 200 OK within 5 seconds, process async if needed
3. **Retry Logic:** Cowpay retries failed webhooks (exponential backoff)
4. **Logging:** Log all webhook events with timestamps for debugging
5. **Dead Letter Queue:** Store unprocessable webhooks for manual review

## Common Integration Patterns

### Fawry Payment Flow

```
1. Customer adds items to cart
   ↓
2. POST /fawry/charge-request
   ↓
3. Display fawry reference_number to customer
   ↓
4. Customer pays at any Fawry kiosk within 24-48 hours
   ↓
5. Receive charge.completed webhook
   ↓
6. Fulfill order (ship or deliver digital good)
   ↓
7. Mark order as paid in system
```

**Code Example:**

```javascript
async function createFawryCharge(order) {
  const signature = generateCowpaySignature(
    process.env.COWPAY_MERCHANT_CODE,
    order.id,
    order.customerId,
    'PAYATFAWRY',
    order.amount * 100, // Convert to piasters
    process.env.COWPAY_MERCHANT_HASH
  );

  const response = await fetch('https://cowpay.me/api/fawry/charge-request', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      merchant_code: process.env.COWPAY_MERCHANT_CODE,
      merchant_reference_id: order.id,
      customer_merchant_profile_id: order.customerId,
      customer_name: order.customerName,
      customer_mobile: order.customerPhone,
      customer_email: order.customerEmail,
      payment_method: 'PAYATFAWRY',
      amount: order.amount * 100,
      currency_code: 'EGP',
      description: 'Order payment',
      charge_items: JSON.stringify(order.items),
      signature: signature
    })
  });

  const data = await response.json();

  if (data.status_code === 200) {
    // Save reference number
    await savePaymentReference(order.id, data.reference_number);
    return { success: true, reference: data.reference_number };
  }

  return { success: false, error: data.status_description };
}
```

### Card Payment Flow

```
1. Customer enters card details
   ↓
2. Tokenize card client-side (optional but recommended)
   ↓
3. POST /card/charge-request with card_token
   ↓
4. Check response status immediately
   ↓
5. If 3D Secure required, redirect to bank verification
   ↓
6. Process result and fulfill order
```

**Code Example:**

```javascript
async function createCardCharge(order, cardToken) {
  const signature = generateCowpaySignature(
    process.env.COWPAY_MERCHANT_CODE,
    order.id,
    order.customerId,
    'CREDITCARD',
    order.amount * 100,
    process.env.COWPAY_MERCHANT_HASH
  );

  const response = await fetch('https://cowpay.me/api/card/charge-request', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      merchant_code: process.env.COWPAY_MERCHANT_CODE,
      merchant_reference_id: order.id,
      customer_merchant_profile_id: order.customerId,
      customer_name: order.customerName,
      customer_mobile: order.customerPhone,
      customer_email: order.customerEmail,
      payment_method: 'CREDITCARD',
      amount: order.amount * 100,
      currency_code: 'EGP',
      card_token: cardToken,
      description: 'Product purchase',
      signature: signature
    })
  });

  const data = await response.json();

  if (data.status_code === 200 && data.status === 'completed') {
    await fulfillOrder(order.id);
    return { success: true, transactionId: data.transaction_reference };
  }

  return { success: false, error: data.status_description };
}
```

### Cash Collection (COD) Flow

```
1. Customer selects cash on delivery at checkout
   ↓
2. POST /cashcollection/charge-request
   ↓
3. Generate shipping label with COD amount
   ↓
4. Ship product with courier
   ↓
5. Courier collects cash from customer
   ↓
6. Receive collection.completed webhook
   ↓
7. Update order status to fully paid
   ↓
8. Courier remits payment to merchant
```

**Code Example:**

```javascript
async function createCashCollectionCharge(order) {
  const signature = generateCowpaySignature(
    process.env.COWPAY_MERCHANT_CODE,
    order.id,
    order.customerId,
    'CASHCOLLECTION',
    order.amount * 100,
    process.env.COWPAY_MERCHANT_HASH
  );

  const response = await fetch('https://cowpay.me/api/cashcollection/charge-request', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      merchant_code: process.env.COWPAY_MERCHANT_CODE,
      merchant_reference_id: order.id,
      customer_merchant_profile_id: order.customerId,
      customer_name: order.customerName,
      customer_mobile: order.customerPhone,
      customer_email: order.customerEmail,
      payment_method: 'CASHCOLLECTION',
      amount: order.amount * 100,
      currency_code: 'EGP',
      delivery_address: order.shippingAddress,
      delivery_governorate: order.governorate,
      description: 'Cash on delivery order',
      signature: signature
    })
  });

  const data = await response.json();

  if (data.status_code === 200) {
    await createShippingLabel(order.id, data.collection_reference);
    return { success: true, collectionRef: data.collection_reference };
  }

  return { success: false, error: data.status_description };
}
```

## Error Handling

Cowpay returns errors with HTTP status codes and descriptive messages.

### HTTP Status Codes

| Status | Meaning | Action |
|---|---|---|
| **200** | Success | Process response data |
| **400** | Bad Request | Validation error - check parameters |
| **401** | Unauthorized | Invalid credentials or signature |
| **403** | Forbidden | Merchant not authorized for operation |
| **404** | Not Found | Charge/payment not found |
| **422** | Unprocessable Entity | Business logic error (duplicate, expired, etc.) |
| **500** | Server Error | Retry with exponential backoff |
| **503** | Service Unavailable | Temporary outage - retry later |

### Error Response Format

```json
{
  "status_code": 400,
  "status_description": "Validation failed",
  "error_code": "INVALID_AMOUNT",
  "error_message": "Amount must be greater than 100 piasters",
  "details": {
    "field": "amount",
    "value": 50,
    "constraint": "minimum"
  }
}
```

### Common Error Codes

| Code | Description | Solution |
|---|---|---|
| `INVALID_AMOUNT` | Amount is invalid (too small, negative, etc.) | Ensure amount ≥ 100 piasters |
| `INVALID_PHONE` | Phone number format incorrect | Use format: +201234567890 |
| `INVALID_EMAIL` | Email format invalid | Use valid email address |
| `INVALID_SIGNATURE` | HMAC signature mismatch | Regenerate signature with correct hash key |
| `MERCHANT_NOT_FOUND` | Merchant code not registered | Verify merchant code is correct |
| `DUPLICATE_TRANSACTION` | Duplicate merchant_reference_id | Use unique reference numbers |
| `CHARGE_NOT_FOUND` | Referenced charge doesn't exist | Verify charge ID/reference number |
| `CHARGE_EXPIRED` | Fawry reference expired | Create new charge request |
| `REFUND_NOT_ELIGIBLE` | Charge cannot be refunded | Only completed charges can be refunded |
| `INSUFFICIENT_BALANCE` | Insufficient funds (cash collection) | Ensure payment collected first |

### Retry Strategy

```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      // Only retry on server errors (5xx)
      if (error.status >= 500) {
        const delay = Math.pow(2, i) * 1000; // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }

      // Don't retry client errors (4xx)
      throw error;
    }
  }
}
```

## Important Notes and Gotchas

### Amount Formatting
- **All amounts are in piasters** (smallest unit of EGP)
- 1 EGP = 100 piasters
- When accepting user input in EGP, multiply by 100 before sending to API
- Always validate amount is ≥ 100 piasters (minimum 1 EGP)

```javascript
// User enters: 500 EGP
const amountInPiasters = 500 * 100; // 50000
```

### HMAC Signature Requirements
- Generate for every request (except GET endpoints that may vary)
- Exact order matters: `merchant_code + merchant_reference_id + customer_id + payment_method + amount + merchant_hash`
- Use SHA256 hashing algorithm only
- Never expose merchant hash in client-side code
- Rotate merchant hash periodically for security

> **⚠️ SIGNATURE ORDER — VERIFY WITH COWPAY**
>
> Cowpay's signature algorithm concatenates fields in a non-standard order and their public documentation is limited. The order documented above (`merchant_code + merchant_reference_id + customer_id + payment_method + amount + merchant_hash`) is based on reported integrations. If signatures fail validation, **contact Cowpay's integration team** to confirm the exact concatenation order and whether it differs per payment method (Fawry vs card vs COD). Wrong field order = signature mismatch on every request.

### Merchant Reference Numbers
- Must be **unique per transaction**
- Use format: `ORD-{TIMESTAMP}-{RANDOM}` to ensure uniqueness
- Used as idempotency key - same reference won't create duplicate charges
- Store mapping between your order ID and merchant reference ID
- Required in status checks and refunds

```javascript
const merchantRefId = `ORD-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
```

### Fawry Reference Numbers
- Expire after **24-48 hours**
- Always check expiration time in response
- Display reference prominently to customer
- Send payment reminders before expiry
- Cannot reuse expired references

### Phone Number Format
- Must use **international format with +20 country code**
- ✓ Correct: `+201234567890`
- ✗ Wrong: `01234567890`, `201234567890`
- Validate format before submission

### Email Validation
- Must be valid email format
- Required for payment confirmation emails
- Webhook notifications sent to stored email
- Verify email domain is reachable

### Webhook Security
1. Always verify `x-request-signature` header
2. Check merchant_code matches your account
3. Process webhooks asynchronously
4. Don't trust webhook content without verification
5. Respond to webhook within 5 seconds
6. Implement idempotency with database constraints

### Testing and Staging
- Always test on staging endpoint: `https://staging.cowpay.me/api/`
- Use test merchant credentials for staging
- Fawry kiosks won't process staging transactions
- Card payments fail on staging with real cards
- Staging and production databases are separate

### PCI Compliance
- Never store raw card data
- Always tokenize cards before transmission
- Implement 3D Secure when available
- Use HTTPS/TLS for all connections
- Store only last 4 digits and card type
- Follow PCI DSS compliance guidelines

### Rate Limiting
- Cowpay enforces rate limits (typical: 100 requests/minute)
- Implement exponential backoff on 429 status
- Cache payment status when possible
- Batch refund operations

### Timezone Handling
- All timestamps from Cowpay are in UTC
- Responses include `created_at`, `paid_at`, `completed_at` timestamps
- Store timestamps in UTC in your database
- Convert to local time for display only

## Useful Links

- **Official Website:** https://cowpay.me/
- **Dashboard:** https://cowpay.me/dashboard/
- **WooCommerce Plugin:** https://github.com/cowpay/woocomerce_plugin
- **Contact Support:** https://cowpay.me/contact or support@cowpay.me
- **Staging API:** https://staging.cowpay.me/api/

## Related Skills

- **Fawry Integration** - For advanced Fawry bill payment features
- **Payment Analytics** - For tracking Egyptian payment trends
- **Webhook Management** - For handling real-time payment events
- **Egypt Tax Integration** - For invoice and VAT compliance
