---
name: ozow
description: "Integrate with Ozow's instant EFT payment API for South African payments. Use this skill whenever the user wants to accept EFT (Electronic Funds Transfer) payments in South Africa, integrate Ozow into their app, handle ZAR transactions, create payment requests, verify transactions, check payment status, or work with Ozow's API in any way. Also trigger when the user mentions 'Ozow', 'South African EFT payments', 'instant EFT', 'accept EFT in South Africa', or needs to process bank transfers."
---

# Ozow Integration Skill

Ozow is South Africa's leading instant EFT (Electronic Funds Transfer) payment gateway, enabling direct bank transfers through a simple REST API. It processes real-time ZAR payments through South African banking rails with instant settlement, making it ideal for e-commerce platforms, subscription services, marketplaces, bill payments, and any solution requiring reliable bank transfer payment processing in South Africa.

## When to use this skill

Build solutions that need to accept payments in South Africa using direct bank transfers. Whether you're building an e-commerce checkout, a subscription billing system, a marketplace with multiple sellers, an invoice payment system, or any application requiring ZAR payments—Ozow provides the infrastructure to accept EFT payments with minimal friction. The customer initiates payment through their own banking app and the transaction settles instantly, making it ideal for high-trust payment scenarios.

## Authentication

Ozow uses a hybrid authentication approach combining API keys with cryptographic hash signatures for request integrity verification.

### Authentication Components

All Ozow API requests require three key components:

- **SiteCode**: Your unique merchant site identifier (provided in Ozow merchant dashboard)
- **API Key**: Your API authentication key (provided in Ozow merchant dashboard)
- **Private Key**: Your secret private key used for generating SHA-512 request signatures (provided in Ozow merchant dashboard)

Store these securely in environment variables:
```bash
OZOW_SITE_CODE=YOUR_SITE_CODE
OZOW_API_KEY=YOUR_API_KEY
OZOW_PRIVATE_KEY=YOUR_PRIVATE_KEY
```

### Request Headers

All requests must include standard headers:

```
Content-Type: application/json
```

### Base URLs

- **Production**: `https://api.ozow.com`
- **Staging**: `https://stagingapi.ozow.com`

Use the staging environment for development and testing. Switch to production only when ready for live transactions.

### Request Signing (SHA-512 Hash)

Every POST request must include a `HashCheck` field computed from a SHA-512 hash of specific request fields concatenated with your private key.

**Hash Generation Algorithm:**

1. Concatenate the following fields IN THIS EXACT ORDER (as strings): `SiteCode`, `CountryCode`, `CurrencyCode`, `Amount`, `Reference`, `PrivateKey`
2. Convert the concatenated string to lowercase
3. Generate a SHA-512 hash of the lowercase string
4. Include the resulting hash as the `HashCheck` field in your request body

**Example (JavaScript):**
```javascript
const crypto = require('crypto');

function generateHashCheck(siteCode, countryCode, currencyCode, amount, reference, privateKey) {
  const concatenated = `${siteCode}${countryCode}${currencyCode}${amount}${reference}${privateKey}`;
  const lowercase = concatenated.toLowerCase();
  const hash = crypto.createHash('sha512').update(lowercase).digest('hex');
  return hash;
}

// Usage
const hashCheck = generateHashCheck(
  'MYSITE',
  'ZA',
  'ZAR',
  '19999',
  'ORDER-12345',
  'my-private-key-here'
);
```

**Example (Python):**
```python
import hashlib

def generate_hash_check(site_code, country_code, currency_code, amount, reference, private_key):
    concatenated = f"{site_code}{country_code}{currency_code}{amount}{reference}{private_key}"
    lowercase = concatenated.lower()
    hash_check = hashlib.sha512(lowercase.encode()).hexdigest()
    return hash_check

# Usage
hash_check = generate_hash_check(
    'MYSITE',
    'ZA',
    'ZAR',
    '19999',
    'ORDER-12345',
    'my-private-key-here'
)
```

## Core API Reference

### 1. Post Payment Request

Initiate a new payment request. The customer will be redirected to complete payment through their bank's EFT interface.

```
POST /PostPaymentRequest
```

**Request Body:**
```json
{
  "SiteCode": "YOUR_SITE_CODE",
  "CountryCode": "ZA",
  "CurrencyCode": "ZAR",
  "Amount": 19999,
  "Reference": "ORDER-12345",
  "BankReference": "Order Ref: ORD-12345",
  "IsTest": false,
  "HashCheck": "a7f3e8b2c1d9f4e6a8b3c2d1e9f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6",
  "OptionalField1": "Customer metadata",
  "OptionalField2": "Additional info",
  "OptionalField3": "",
  "OptionalField4": "",
  "OptionalField5": "",
  "Customer": {
    "Title": "Mr",
    "FirstName": "John",
    "LastName": "Doe",
    "EmailAddress": "john@example.com",
    "CellNumber": "+27123456789",
    "CompanyName": ""
  },
  "SuccessUrl": "https://yoursite.com/payment/success",
  "FailureUrl": "https://yoursite.com/payment/failure",
  "ErrorUrl": "https://yoursite.com/payment/error",
  "NotifyUrl": "https://yoursite.com/webhook/ozow"
}
```

**Field Definitions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| SiteCode | string | Yes | Your unique merchant site code |
| CountryCode | string | Yes | Always "ZA" for South Africa |
| CurrencyCode | string | Yes | Always "ZAR" for South African Rand |
| Amount | integer | Yes | Payment amount in **cents** (e.g., 19999 = R199.99) |
| Reference | string | Yes | Your unique transaction reference (must be unique per transaction) |
| BankReference | string | No | Description shown to customer during bank transfer |
| IsTest | boolean | No | Set to `true` for testing in staging environment |
| HashCheck | string | Yes | SHA-512 signature for request authentication |
| OptionalField1-5 | string | No | Custom metadata fields for your system |
| Customer | object | No | Customer details object |
| SuccessUrl | string | Yes | URL to redirect after successful payment |
| FailureUrl | string | Yes | URL to redirect after failed payment |
| ErrorUrl | string | Yes | URL to redirect if error occurs |
| NotifyUrl | string | No | Webhook URL for payment status notifications |

**Response (Success):**
```json
{
  "IsTest": false,
  "SiteCode": "YOUR_SITE_CODE",
  "TransactionId": "txn_abc123def456",
  "Reference": "ORDER-12345",
  "Amount": 19999,
  "Status": "Created",
  "RedirectUrl": "https://secure.ozow.com/checkout?TransactionId=txn_abc123def456",
  "ErrorCode": 0,
  "ErrorMessage": "Request processed successfully"
}
```

**Critical Details:**

- **Amount in cents**: R199.99 must be sent as `19999` (cents)
- **RedirectUrl**: Redirect the customer to this URL immediately. They will complete the payment through their bank's EFT interface
- **Customer return**: After payment completion (success or failure), the customer is redirected back to your success/failure URL with `TransactionId` as a query parameter
- **Webhook notification**: Listen for webhook notifications on your `NotifyUrl` for final payment confirmation (recommended over relying on client-side redirects)

### 2. Get Transaction by Reference

Query the current status of a payment transaction using your merchant reference.

```
GET /GetTransactionByReference
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| SiteCode | string | Yes | Your site code |
| Reference | string | Yes | Your transaction reference |
| TransactionId | string | No | Ozow's transaction ID (optional, for additional specificity) |

**Response (Success):**
```json
{
  "IsTest": false,
  "SiteCode": "YOUR_SITE_CODE",
  "TransactionId": "txn_abc123def456",
  "Reference": "ORDER-12345",
  "Amount": 19999,
  "Status": "Complete",
  "StatusMessage": "Payment completed successfully",
  "TransactionType": "Payment",
  "BankName": "Nedbank",
  "BankReference": "Order Ref: ORD-12345",
  "CreatedDate": "2025-02-23T10:30:00Z",
  "CompletedDate": "2025-02-23T10:35:00Z",
  "ErrorCode": 0,
  "ErrorMessage": "Transaction retrieved successfully"
}
```

**Status Values:**

- `Created` — Payment request created, awaiting customer action
- `Pending` — Customer initiated payment, awaiting bank confirmation
- `Complete` — Payment successfully completed
- `Failed` — Payment failed
- `Abandoned` — Customer abandoned the payment without completing
- `Cancelled` — Merchant cancelled the transaction

### 3. Get Transaction List

Retrieve a paginated list of recent transactions for your merchant site.

```
GET /GetTransactionList
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| SiteCode | string | Yes | Your site code |
| Skip | integer | No | Number of records to skip (default: 0) |
| Take | integer | No | Number of records to return (default: 20, max: 100) |
| FromDate | string | No | Start date filter (ISO 8601 format, e.g., "2025-02-01") |
| ToDate | string | No | End date filter (ISO 8601 format, e.g., "2025-02-28") |

**Response:**
```json
{
  "Data": [
    {
      "TransactionId": "txn_abc123def456",
      "Reference": "ORDER-12345",
      "Amount": 19999,
      "Status": "Complete",
      "CreatedDate": "2025-02-23T10:30:00Z",
      "CompletedDate": "2025-02-23T10:35:00Z",
      "BankName": "FNB",
      "StatusMessage": "Payment completed"
    },
    {
      "TransactionId": "txn_xyz789uvw012",
      "Reference": "ORDER-12346",
      "Amount": 5000,
      "Status": "Pending",
      "CreatedDate": "2025-02-23T11:00:00Z",
      "CompletedDate": null,
      "BankName": "Standard Bank",
      "StatusMessage": "Awaiting customer confirmation"
    }
  ],
  "ErrorCode": 0,
  "ErrorMessage": "Transactions retrieved successfully"
}
```

### 4. Cancel Transaction

Cancel a pending payment transaction that hasn't been completed yet.

```
POST /CancelTransaction
```

**Request Body:**
```json
{
  "SiteCode": "YOUR_SITE_CODE",
  "TransactionId": "txn_abc123def456",
  "Reference": "ORDER-12345"
}
```

**Response (Success):**
```json
{
  "TransactionId": "txn_abc123def456",
  "Reference": "ORDER-12345",
  "Status": "Cancelled",
  "ErrorCode": 0,
  "ErrorMessage": "Transaction cancelled successfully"
}
```

### 5. Generate Hash Check

Generate a SHA-512 hash signature for request verification (useful for testing hash generation in your integration).

```
POST /GenerateHashCheck
```

**Request Body:**
```json
{
  "SiteCode": "YOUR_SITE_CODE",
  "CountryCode": "ZA",
  "CurrencyCode": "ZAR",
  "Amount": 19999,
  "Reference": "ORDER-12345",
  "PrivateKey": "YOUR_PRIVATE_KEY"
}
```

**Response:**
```json
{
  "Hash": "a7f3e8b2c1d9f4e6a8b3c2d1e9f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6",
  "ErrorCode": 0,
  "ErrorMessage": "Hash generated successfully"
}
```

## Webhooks

Ozow sends webhook notifications to your application when payment status changes. Webhooks are essential for reliable payment confirmation and should be prioritized over client-side redirect verification.

### Webhook Events

Ozow sends POST requests to your `NotifyUrl` endpoint with the following payload structure:

**Webhook Payload:**
```json
{
  "TransactionId": "txn_abc123def456",
  "Reference": "ORDER-12345",
  "Amount": 19999,
  "Status": "Complete",
  "StatusMessage": "Payment completed successfully",
  "BankName": "Nedbank",
  "BankReference": "Order Ref: ORD-12345",
  "CreatedDate": "2025-02-23T10:30:00Z",
  "CompletedDate": "2025-02-23T10:35:00Z",
  "TransactionType": "Payment",
  "Hash": "b8g4f9i3d0k7e5h2c1j6a8m3l9n4o1p5q2r6s3t7u8v1w2x3y4z5a6b7c8d9e0f1g2h3i4j5k6l7m8n9o0p1q2r3s4t5u6v7w8x9y0z1a2b3c4d5e6f7"
}
```

### Webhook Signature Verification

**CRITICAL:** Always verify the `Hash` field in webhook payloads to prevent spoofing and ensure request authenticity.

**Hash Verification Algorithm:**

1. Extract the `Hash` value from the webhook payload
2. Concatenate the following fields IN THIS EXACT ORDER: `TransactionId`, `Reference`, `Amount`, `Status`, `BankName`, `BankReference`, `CreatedDate`, `CompletedDate`, `PrivateKey`
3. Convert the concatenated string to lowercase
4. Generate a SHA-512 hash of the lowercase string
5. Compare your generated hash with the received `Hash` value
6. **Only process the webhook if hashes match**

**Example (Node.js):**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(webhook, privateKey) {
  // Extract the hash from the webhook
  const receivedHash = webhook.Hash;

  // Reconstruct the signature
  const fields = [
    webhook.TransactionId,
    webhook.Reference,
    webhook.Amount.toString(),
    webhook.Status,
    webhook.BankName,
    webhook.BankReference,
    webhook.CreatedDate,
    webhook.CompletedDate,
    privateKey
  ];

  const concatenated = fields.join('');
  const lowercase = concatenated.toLowerCase();
  const calculatedHash = crypto.createHash('sha512').update(lowercase).digest('hex');

  // Verify the hash
  return calculatedHash === receivedHash;
}

// In your webhook handler:
app.post('/webhook/ozow', (req, res) => {
  const isValid = verifyWebhookSignature(req.body, process.env.OZOW_PRIVATE_KEY);

  if (!isValid) {
    console.error('Invalid webhook signature - potential spoofing attempt');
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process the webhook
  const { TransactionId, Reference, Status, Amount } = req.body;

  switch (Status) {
    case 'Complete':
      console.log(`Payment complete: ${Reference} for R${(Amount / 100).toFixed(2)}`);
      // Update order status, send confirmation email, etc.
      break;
    case 'Failed':
      console.log(`Payment failed: ${Reference}`);
      // Handle payment failure
      break;
    case 'Abandoned':
      console.log(`Payment abandoned: ${Reference}`);
      // Handle abandoned payment
      break;
    case 'Pending':
      console.log(`Payment pending: ${Reference}`);
      // Update order status to pending
      break;
  }

  res.status(200).json({ success: true });
});
```

**Example (Python):**
```python
import hashlib
from flask import Flask, request, jsonify

app = Flask(__name__)

def verify_webhook_signature(webhook_data, private_key):
    """Verify the webhook signature"""
    received_hash = webhook_data.get('Hash')

    # Reconstruct the signature
    fields = [
        webhook_data.get('TransactionId', ''),
        webhook_data.get('Reference', ''),
        str(webhook_data.get('Amount', '')),
        webhook_data.get('Status', ''),
        webhook_data.get('BankName', ''),
        webhook_data.get('BankReference', ''),
        webhook_data.get('CreatedDate', ''),
        webhook_data.get('CompletedDate', ''),
        private_key
    ]

    concatenated = ''.join(fields)
    lowercase = concatenated.lower()
    calculated_hash = hashlib.sha512(lowercase.encode()).hexdigest()

    return calculated_hash == received_hash

@app.route('/webhook/ozow', methods=['POST'])
def handle_ozow_webhook():
    webhook_data = request.get_json()
    private_key = os.getenv('OZOW_PRIVATE_KEY')

    if not verify_webhook_signature(webhook_data, private_key):
        return jsonify({'error': 'Invalid signature'}), 401

    transaction_id = webhook_data.get('TransactionId')
    reference = webhook_data.get('Reference')
    status = webhook_data.get('Status')
    amount = webhook_data.get('Amount')

    if status == 'Complete':
        print(f"Payment complete: {reference} for R{amount / 100:.2f}")
        # Update order, send confirmation, etc.
    elif status == 'Failed':
        print(f"Payment failed: {reference}")
    elif status == 'Abandoned':
        print(f"Payment abandoned: {reference}")

    return jsonify({'success': True})
```

### Important Webhook Considerations

- **Duplicate notifications**: Ozow may occasionally send duplicate notifications for the same transaction. Your system must be idempotent to prevent double-crediting
- **Asynchronous processing**: Process webhooks asynchronously and return a 200 OK response immediately to prevent timeouts
- **Retry logic**: Ozow retries failed webhook deliveries; ensure your endpoint is reliable
- **Reference uniqueness**: Always use unique references for each transaction for reliable webhook matching

## Common Integration Patterns

### Pattern 1: E-Commerce Checkout Flow

The most common integration pattern for online stores and marketplaces.

```
1. Customer clicks "Pay with Bank Transfer"
   ↓
2. POST /PostPaymentRequest with order details and customer info
   ↓
3. Receive RedirectUrl in response
   ↓
4. Redirect customer to RedirectUrl (Ozow's secure checkout)
   ↓
5. Customer completes EFT payment through their bank app
   ↓
6. Customer redirected to your success/failure URL with TransactionId
   ↓
7. (Recommended) Listen for webhook notification on NotifyUrl
   ↓
8. On webhook status=Complete: Fulfill order, send confirmation email
   ↓
9. (Backup) If no webhook received, periodic polling: GET /GetTransactionByReference
```

**Implementation:**
```javascript
// Step 1: Create payment request
async function initiatePayment(orderData) {
  const amount = Math.round(orderData.total * 100); // Convert to cents
  const reference = `ORDER-${orderData.orderId}`;

  const hashCheck = generateHashCheck(
    process.env.OZOW_SITE_CODE,
    'ZA',
    'ZAR',
    amount.toString(),
    reference,
    process.env.OZOW_PRIVATE_KEY
  );

  const response = await fetch('https://api.ozow.com/PostPaymentRequest', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      SiteCode: process.env.OZOW_SITE_CODE,
      CountryCode: 'ZA',
      CurrencyCode: 'ZAR',
      Amount: amount,
      Reference: reference,
      HashCheck: hashCheck,
      IsTest: false,
      Customer: {
        FirstName: orderData.customerName.split(' ')[0],
        LastName: orderData.customerName.split(' ')[1],
        EmailAddress: orderData.customerEmail,
        CellNumber: orderData.customerPhone
      },
      SuccessUrl: `https://yoursite.com/checkout/success?orderId=${orderData.orderId}`,
      FailureUrl: `https://yoursite.com/checkout/failure?orderId=${orderData.orderId}`,
      ErrorUrl: `https://yoursite.com/checkout/error?orderId=${orderData.orderId}`,
      NotifyUrl: 'https://yoursite.com/webhook/ozow'
    })
  });

  const paymentRequest = await response.json();
  return paymentRequest.RedirectUrl;
}

// Step 8: Handle webhook notification
app.post('/webhook/ozow', async (req, res) => {
  if (!verifyWebhookSignature(req.body, process.env.OZOW_PRIVATE_KEY)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const { Reference, Status, Amount, TransactionId } = req.body;

  if (Status === 'Complete') {
    const orderId = Reference.replace('ORDER-', '');

    // Update order status
    await updateOrderStatus(orderId, 'completed', TransactionId);

    // Send confirmation email
    await sendOrderConfirmationEmail(orderId, Amount / 100);

    // Trigger fulfillment
    await fulfillOrder(orderId);
  }

  res.status(200).json({ success: true });
});
```

### Pattern 2: Invoice / Utility Bill Payment

For payment links sent via email or SMS where customers pay individual invoices.

```
1. Generate unique payment link for invoice
2. Customer receives email/SMS with payment link
3. Link redirects to payment confirmation page
4. Click "Pay Now" triggers payment request
5. Customer redirected to Ozow to complete payment
6. Webhook confirms payment completion
7. Invoice marked as paid, receipt issued
```

**Implementation:**
```javascript
// Generate payment link
app.get('/invoice/:invoiceId/pay', async (req, res) => {
  const invoice = await getInvoiceById(req.params.invoiceId);

  if (invoice.paid) {
    return res.render('invoice-paid', { invoiceId: invoice.id });
  }

  res.render('invoice-payment', {
    invoiceId: invoice.id,
    amount: (invoice.amount / 100).toFixed(2),
    customerName: invoice.customerName
  });
});

// Handle "Pay Now" button
app.post('/invoice/:invoiceId/initiate-payment', async (req, res) => {
  const invoice = await getInvoiceById(req.params.invoiceId);
  const reference = `INV-${invoice.id}`;

  const hashCheck = generateHashCheck(
    process.env.OZOW_SITE_CODE,
    'ZA',
    'ZAR',
    invoice.amount.toString(),
    reference,
    process.env.OZOW_PRIVATE_KEY
  );

  const response = await fetch('https://api.ozow.com/PostPaymentRequest', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      SiteCode: process.env.OZOW_SITE_CODE,
      CountryCode: 'ZA',
      CurrencyCode: 'ZAR',
      Amount: invoice.amount,
      Reference: reference,
      BankReference: `Invoice ${invoice.id}`,
      HashCheck: hashCheck,
      Customer: {
        FirstName: invoice.customerName.split(' ')[0],
        LastName: invoice.customerName.split(' ')[1],
        EmailAddress: invoice.customerEmail
      },
      SuccessUrl: `https://yoursite.com/invoice/${invoice.id}/success`,
      FailureUrl: `https://yoursite.com/invoice/${invoice.id}/failure`,
      NotifyUrl: 'https://yoursite.com/webhook/ozow'
    })
  });

  const paymentRequest = await response.json();
  res.json({ redirectUrl: paymentRequest.RedirectUrl });
});
```

### Pattern 3: Subscription / Recurring Billing

For subscription services requiring periodic billing (manual recurring model).

```
1. Customer sets up subscription
2. Every billing period: Create new payment request for subscription fee
3. Send payment link to customer email
4. Customer completes EFT payment
5. Webhook confirms payment
6. Subscription renewed, service access renewed
7. Repeat next billing period
```

**Note**: Ozow doesn't support tokenized recurring billing. Each payment requires a new transaction.

### Pattern 4: Multi-Seller Marketplace

For marketplaces collecting payments that need to be distributed to multiple sellers.

```
1. Multiple sellers' products in customer's cart
2. Customer initiates checkout
3. Create single payment request for total amount (all seller fees)
4. Ozow processes payment (settle to merchant account)
5. Webhook confirms payment completion
6. Distribute funds to sellers based on order split
7. Update seller payouts queue
```

## Error Handling

Ozow returns consistent error structures in all responses. Always check the `ErrorCode` and `ErrorMessage` fields.

### Error Response Structure

```json
{
  "ErrorCode": 400,
  "ErrorMessage": "Invalid Amount - must be greater than 0"
}
```

### Common Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | No action needed |
| 100 | Invalid SiteCode | Verify SiteCode in environment variables |
| 101 | Invalid CountryCode | Must be "ZA" for South Africa |
| 102 | Invalid CurrencyCode | Must be "ZAR" for South African Rand |
| 200 | Invalid Amount | Amount must be > 0 and in cents |
| 300 | Invalid Reference | Reference must be unique per transaction |
| 400 | Hash check failed | SHA-512 hash signature is incorrect; verify hash generation |
| 401 | Authentication failed | Check API Key and credentials |
| 500 | Server error | Transient error; implement exponential backoff retry |
| 503 | Service unavailable | Ozow API temporarily unavailable; retry with backoff |

### Recommended Error Handling

```javascript
async function callOzowAPI(endpoint, payload) {
  let retries = 3;
  let backoffMs = 1000;

  while (retries > 0) {
    try {
      const response = await fetch(`https://api.ozow.com${endpoint}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
        timeout: 10000
      });

      const data = await response.json();

      if (data.ErrorCode === 0) {
        return data;
      }

      // Non-recoverable errors
      if ([100, 101, 102, 200, 300, 400, 401].includes(data.ErrorCode)) {
        throw new Error(`Ozow API Error ${data.ErrorCode}: ${data.ErrorMessage}`);
      }

      // Recoverable errors (5xx)
      if ([500, 503].includes(data.ErrorCode) && retries > 1) {
        console.warn(`Ozow API error ${data.ErrorCode}, retrying in ${backoffMs}ms...`);
        await sleep(backoffMs);
        backoffMs *= 2;
        retries--;
        continue;
      }

      throw new Error(`Ozow API Error ${data.ErrorCode}: ${data.ErrorMessage}`);

    } catch (error) {
      if (retries > 1 && error.code === 'ETIMEDOUT') {
        console.warn(`Ozow API timeout, retrying in ${backoffMs}ms...`);
        await sleep(backoffMs);
        backoffMs *= 2;
        retries--;
        continue;
      }
      throw error;
    }
  }

  throw new Error('Ozow API failed after 3 retries');
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

## Important Notes & Gotchas

### Critical Implementation Details

1. **Amount in cents**: Always convert currency amounts to cents. R199.99 = 19999. Use integers to avoid floating-point precision errors.

2. **Unique references**: Every payment request MUST have a unique `Reference`. Ozow uses this to match transactions. Duplicate references can cause lookup issues.

3. **Hash signature is mandatory**: The `HashCheck` field is required for all POST requests. Compute it exactly as specified—order matters, case matters (must be lowercase before hashing).

4. **Webhook signature verification is critical**: Always verify the `Hash` in incoming webhooks. Never process unverified webhooks as you're vulnerable to spoofing attacks.

5. **Idempotent webhooks**: Design your webhook handler to be idempotent because Ozow may send duplicate notifications. Use database unique constraints or deduplication logic.

6. **Don't rely on client-side redirects**: Customers may close their browser after payment completes. Always verify payment status server-side via webhooks or API polling.

7. **Redirect URLs are required**: `SuccessUrl`, `FailureUrl`, and `ErrorUrl` are all mandatory fields in PostPaymentRequest.

8. **EFT processing times**: Instant EFT typically completes within minutes during business hours (08:00-17:00 SAST weekdays), but may take longer after hours or on weekends.

9. **Test mode in staging**: Use `IsTest: true` in staging environment to process test transactions without actual bank transfers.

10. **Handle abandoned payments**: A customer may initiate payment but abandon it in their banking app. The transaction status becomes `Abandoned`. Monitor these and send reminder emails.

11. **Bank reference visibility**: The `BankReference` field appears in the customer's banking app. Use descriptive text like "Invoice INV-123" or "Order ORD-456" for clarity.

12. **SiteCode, CountryCode, CurrencyCode are constants**: For South African operations, these are always "YOUR_SITE_CODE", "ZA", and "ZAR" respectively.

13. **OptionalFields for metadata**: Use OptionalField1-5 to store custom data (like customer ID, order notes, etc.) that gets returned with transaction queries.

14. **Concurrent payment safety**: Ozow prevents creating multiple simultaneous payments for the same reference. If a transaction is pending and you create another with the same reference, you'll get an error.

15. **No partial refunds**: Ozow EFT payments don't support partial refunds. For partial refunds, you'll need to process them through separate banking channels.

## Useful Links

- **Official Ozow Website**: https://ozow.com/
- **API Documentation**: https://developer.ozow.com/
- **Developer Training Portal**: https://training.ozow.com/
- **Merchant Dashboard**: https://dashboard.ozow.com
- **API Tracker Reference**: https://apitracker.io/a/ozow
- **Support**: support@ozow.com
- **Sandbox Testing**: Use `https://stagingapi.ozow.com` with test credentials from your Ozow dashboard
