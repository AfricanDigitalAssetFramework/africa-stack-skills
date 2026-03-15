---
name: Seerbit Payment Gateway Integration
description: "Nigerian payment gateway API for accepting card, USSD, bank transfer, QR code, and virtual account payments. Seerbit payment API for card payments Nigeria, USSD payments Africa, bank transfer payments, recurring subscriptions, virtual accounts, and more. Accept payments in Nigeria with Seerbit."
---

# Seerbit Payment Gateway Integration

Seerbit is a comprehensive payment gateway that enables businesses to accept payments across multiple channels in Nigeria and West Africa. It provides APIs for card payments (Visa, Mastercard, Verve), USSD transfers, bank account transfers, QR code payments, virtual accounts, and mobile money solutions. With PCI-DSS certification and support for recurring payments, subscriptions, and tokenization, Seerbit is designed for developers building payment solutions across web, mobile, and backend systems.

## When to Use This Skill

Use Seerbit when you need to:
- Accept card payments (Visa, Mastercard, Verve) on your platform
- Process USSD-based payments (popular in Nigeria for feature phones)
- Implement bank transfer payments with automatic settlement
- Create virtual accounts for customers to deposit funds
- Generate QR codes for contactless payments
- Build subscription and recurring billing systems
- Tokenize cards for future charges without re-entering details
- Accept mobile money payments across West Africa
- Handle webhooks for payment notifications and status updates
- Implement 3D Secure authentication for high-value transactions
- Support international customer payments in multiple currencies

**Not suitable for**: Direct wallet transfers, payouts to customer accounts (use dedicated payout APIs), or cases where PCI compliance is not a priority.

## Authentication

Seerbit uses Bearer token authentication. You must obtain an encrypted key (bearer token) before making API requests.

### Obtaining an Encrypted Key

Generate a bearer token by sending your merchant keys to the encryption endpoint:

**Endpoint:** `POST https://seerbitapi.com/api/v2/encrypt/keys`

**Request:**
```json
{
  "clientId": "YOUR_MERCHANT_PUBLIC_KEY",
  "clientSecret": "YOUR_MERCHANT_SECRET_KEY"
}
```

**Response:**
```json
{
  "status": "SUCCESS",
  "data": {
    "code": "00",
    "EncryptedSecKey": {
      "encryptedKey": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    },
    "message": "Successful"
  }
}
```

### Using the Bearer Token

Include the `encryptedKey` in the Authorization header for all subsequent API requests:

```bash
curl -X POST https://seerbitapi.com/api/v2/payments/initiates \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{...}'
```

**JavaScript/Node.js Example:**
```javascript
const axios = require('axios');

async function getEncryptedKey() {
  const response = await axios.post('https://seerbitapi.com/api/v2/encrypt/keys', {
    clientId: process.env.SEERBIT_PUBLIC_KEY,
    clientSecret: process.env.SEERBIT_SECRET_KEY
  });

  return response.data.data.EncryptedSecKey.encryptedKey;
}

async function makePaymentRequest(bearerToken) {
  const config = {
    headers: {
      'Authorization': `Bearer ${bearerToken}`,
      'Content-Type': 'application/json'
    }
  };

  const paymentData = {
    publicKey: process.env.SEERBIT_PUBLIC_KEY,
    amount: '5000',
    currency: 'NGN',
    paymentType: 'CARD',
    reference: 'ref-' + Date.now()
  };

  const response = await axios.post(
    'https://seerbitapi.com/api/v2/payments/initiates',
    paymentData,
    config
  );

  return response.data;
}
```

**Python Example:**
```python
import requests
import os

def get_encrypted_key():
    url = 'https://seerbitapi.com/api/v2/encrypt/keys'
    payload = {
        'clientId': os.getenv('SEERBIT_PUBLIC_KEY'),
        'clientSecret': os.getenv('SEERBIT_SECRET_KEY')
    }
    response = requests.post(url, json=payload)
    return response.json()['data']['EncryptedSecKey']['encryptedKey']

def initiate_payment(bearer_token):
    url = 'https://seerbitapi.com/api/v2/payments/initiates'
    headers = {
        'Authorization': f'Bearer {bearer_token}',
        'Content-Type': 'application/json'
    }
    payload = {
        'publicKey': os.getenv('SEERBIT_PUBLIC_KEY'),
        'amount': '5000',
        'currency': 'NGN',
        'paymentType': 'CARD',
        'reference': f'ref-{int(time.time())}'
    }
    response = requests.post(url, json=payload, headers=headers)
    return response.json()
```

## Core API Reference

### Base URL
```
https://seerbitapi.com/api/v2/
```

### Payment Initialization Endpoint

**Endpoint:** `POST /payments/initiates`

**Description:** Initiate a payment transaction with various payment methods.

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `publicKey` | string | Yes | Your Seerbit public key |
| `amount` | string | Yes | Payment amount in the smallest currency unit (e.g., kobo for NGN) |
| `currency` | string | Yes | Currency code (NGN, GHS, KES, UGX, etc.) |
| `paymentType` | string | Yes | Payment method: `CARD`, `USSD`, `TRANSFER`, `QR`, `ACCOUNTFUNDING`, `MOBILEMONEY` |
| `reference` | string | Yes | Unique transaction reference (max 50 chars) |
| `email` | string | Yes | Customer email address |
| `phoneNumber` | string | No | Customer phone number |
| `fullname` | string | No | Customer full name |
| `productId` | string | No | Product/service identifier |
| `productDescription` | string | No | Description of product/service |
| `mobileNumber` | string | Conditional | Required for USSD and MOBILEMONEY |
| `bankCode` | string | Conditional | Required for TRANSFER payment type |
| `accountNumber` | string | Conditional | Required for TRANSFER payment type |
| `redirectUrl` | string | No | URL to redirect after payment |
| `callbackUrl` | string | No | Webhook callback URL for status updates |

**Card Payment Request Example:**
```json
{
  "publicKey": "pk_live_xxxxxxxxxxxxx",
  "amount": "50000",
  "currency": "NGN",
  "paymentType": "CARD",
  "reference": "ref-1234567890",
  "email": "customer@example.com",
  "phoneNumber": "+2348012345678",
  "fullname": "John Doe",
  "productId": "PROD001",
  "productDescription": "Premium Subscription"
}
```

**USSD Payment Request Example:**
```json
{
  "publicKey": "pk_live_xxxxxxxxxxxxx",
  "amount": "10000",
  "currency": "NGN",
  "paymentType": "USSD",
  "reference": "ref-1234567890",
  "email": "customer@example.com",
  "mobileNumber": "+2348012345678",
  "fullname": "John Doe"
}
```

**Bank Transfer Payment Request Example:**
```json
{
  "publicKey": "pk_live_xxxxxxxxxxxxx",
  "amount": "25000",
  "currency": "NGN",
  "paymentType": "TRANSFER",
  "reference": "ref-1234567890",
  "email": "customer@example.com",
  "bankCode": "011",
  "accountNumber": "1234567890"
}
```

**Response (Successful):**
```json
{
  "code": "00",
  "status": "SUCCESS",
  "message": "Successful",
  "data": {
    "redirectUrl": "https://checkout.seerbit.com/?token=...",
    "reference": "ref-1234567890",
    "paymentStatus": "pending",
    "amount": "50000",
    "currency": "NGN",
    "transactionReference": "SR123456789"
  }
}
```

**Response (Error):**
```json
{
  "code": "01",
  "status": "FAILED",
  "message": "Invalid API key",
  "data": null
}
```

### Charge Token Endpoint

**Endpoint:** `POST /payments/charge-token`

**Description:** Charge a previously tokenized card (recurring payments).

**Request:**
```json
{
  "publicKey": "pk_live_xxxxxxxxxxxxx",
  "amount": "5000",
  "currency": "NGN",
  "reference": "ref-recurring-1234567890",
  "authorizationCode": "TOKEN_FROM_FIRST_PAYMENT",
  "email": "customer@example.com"
}
```

**Response:**
```json
{
  "code": "00",
  "status": "SUCCESS",
  "message": "Transaction Successful",
  "data": {
    "amount": "5000",
    "transactionReference": "SR987654321",
    "paymentStatus": "successful",
    "authorizationCode": "TOKEN_FOR_FUTURE_USE"
  }
}
```

### Query Transaction Status

**Endpoint:** `GET /transactions/{reference}`

**Description:** Check the status of a payment transaction.

**Response:**
```json
{
  "code": "00",
  "status": "SUCCESS",
  "message": "Successful",
  "data": {
    "reference": "ref-1234567890",
    "transactionReference": "SR123456789",
    "paymentStatus": "successful",
    "gatewayCode": "00",
    "gatewayMessage": "Approved by Financial Institution",
    "amount": "50000",
    "currency": "NGN",
    "paymentType": "CARD",
    "createdAt": "2024-02-15T10:30:00Z",
    "authorizationCode": "AUTH_CODE_FOR_CARD"
  }
}
```

## Webhooks

Seerbit sends real-time webhook notifications when payment events occur. Configure your webhook endpoint on the Seerbit merchant dashboard.

### Webhook Configuration

1. Log in to your Seerbit dashboard
2. Navigate to **Settings > Webhooks**
3. Enter your webhook URL (must be publicly accessible HTTPS)
4. Select events you want to receive notifications for

### Webhook Payload Structure

**Transaction Success Webhook:**
```json
{
  "notificationItems": [
    {
      "notificationRequestItem": {
        "eventType": "transaction",
        "eventDate": "2024-02-15T10:35:22Z",
        "eventId": "evt_123456789",
        "data": {
          "reference": "ref-1234567890",
          "transactionReference": "SR123456789",
          "paymentStatus": "successful",
          "amount": "50000",
          "currency": "NGN",
          "paymentType": "CARD",
          "email": "customer@example.com",
          "phoneNumber": "+2348012345678",
          "fullname": "John Doe",
          "gatewayCode": "00",
          "gatewayMessage": "Approved by Financial Institution",
          "authorizationCode": "AUTH_CODE_12345",
          "processorResponse": "Successful",
          "processorCode": "00"
        }
      }
    }
  ]
}
```

**Recurring Transaction Webhook:**
```json
{
  "notificationItems": [
    {
      "notificationRequestItem": {
        "eventType": "transaction.recurrent",
        "eventDate": "2024-02-16T10:35:22Z",
        "eventId": "evt_987654321",
        "data": {
          "reference": "ref-recurring-1234567890",
          "transactionReference": "SR987654321",
          "paymentStatus": "successful",
          "amount": "5000",
          "currency": "NGN",
          "authorizationCode": "TOKEN_FROM_FIRST_PAYMENT"
        }
      }
    }
  ]
}
```

**Refund Webhook:**
```json
{
  "notificationItems": [
    {
      "notificationRequestItem": {
        "eventType": "refund",
        "eventDate": "2024-02-16T11:00:00Z",
        "eventId": "evt_refund_123",
        "data": {
          "originalReference": "ref-1234567890",
          "refundReference": "refund-123456",
          "amount": "50000",
          "currency": "NGN",
          "status": "successful",
          "reason": "Customer request"
        }
      }
    }
  ]
}
```

### Webhook Handling Best Practices

1. **Always respond with HTTP 200** to acknowledge receipt within 5 seconds
2. **Verify webhook authenticity** using the signature header (if provided by Seerbit)
3. **Use the reference field** to match incoming webhooks to your orders
4. **Handle idempotency**: Webhooks may be retried up to 7 days, so check if you've already processed the event
5. **Process asynchronously**: Offload webhook processing to a background job to avoid timeouts
6. **Log all webhooks** for debugging and reconciliation

**Node.js Webhook Handler Example:**
```javascript
const express = require('express');
const app = express();

app.post('/webhook/seerbit', express.json(), (req, res) => {
  // Always respond immediately
  res.status(200).json({ success: true });

  // Process webhook asynchronously
  processWebhookAsync(req.body).catch(err => {
    console.error('Webhook processing error:', err);
  });
});

async function processWebhookAsync(payload) {
  const items = payload.notificationItems || [];

  for (const item of items) {
    const { eventType, data } = item.notificationRequestItem;

    // Check if already processed (idempotency)
    const processed = await db.webhooks.findOne({
      eventId: item.notificationRequestItem.eventId
    });

    if (processed) {
      console.log('Webhook already processed:', item.notificationRequestItem.eventId);
      continue;
    }

    // Log the webhook
    await db.webhooks.create({
      eventId: item.notificationRequestItem.eventId,
      eventType: eventType,
      payload: data,
      createdAt: new Date()
    });

    // Handle different event types
    if (eventType === 'transaction') {
      await handleTransactionSuccess(data);
    } else if (eventType === 'transaction.recurrent') {
      await handleRecurringTransaction(data);
    } else if (eventType === 'refund') {
      await handleRefund(data);
    }
  }
}

async function handleTransactionSuccess(data) {
  console.log('Payment successful:', data.reference);

  // Update order status
  await db.orders.updateOne(
    { reference: data.reference },
    {
      paymentStatus: 'completed',
      transactionRef: data.transactionReference,
      authCode: data.authorizationCode
    }
  );

  // Send confirmation email
  // await sendConfirmationEmail(data.email);
}
```

**Python Webhook Handler Example:**
```python
from flask import Flask, request, jsonify
import asyncio
from queue import Queue

app = Flask(__name__)
webhook_queue = Queue()

@app.route('/webhook/seerbit', methods=['POST'])
def handle_webhook():
    # Acknowledge receipt immediately
    webhook_queue.put(request.json)
    return jsonify({'success': True}), 200

def process_webhook_queue():
    while True:
        payload = webhook_queue.get()
        try:
            process_webhook_payload(payload)
        except Exception as e:
            print(f"Error processing webhook: {e}")

def process_webhook_payload(payload):
    items = payload.get('notificationItems', [])

    for item in items:
        request_item = item.get('notificationRequestItem', {})
        event_type = request_item.get('eventType')
        data = request_item.get('data', {})
        event_id = request_item.get('eventId')

        # Check idempotency
        if db.webhooks.find_one({'eventId': event_id}):
            print(f"Webhook already processed: {event_id}")
            continue

        # Log webhook
        db.webhooks.insert_one({
            'eventId': event_id,
            'eventType': event_type,
            'payload': data,
            'createdAt': datetime.now()
        })

        # Route to handler
        if event_type == 'transaction':
            handle_transaction_success(data)
        elif event_type == 'transaction.recurrent':
            handle_recurring_transaction(data)
        elif event_type == 'refund':
            handle_refund(data)
```

## Common Integration Patterns

### Pattern 1: Simple Card Payment Flow

```javascript
// 1. Client-side: Create payment button click handler
async function handlePaymentClick() {
  try {
    // Get bearer token from backend
    const tokenResponse = await fetch('/api/seerbit/token', { method: 'POST' });
    const { bearerToken } = await tokenResponse.json();

    // Initiate payment
    const paymentResponse = await fetch('/api/seerbit/payment', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        amount: 50000,
        currency: 'NGN',
        paymentType: 'CARD',
        email: document.getElementById('email').value,
        fullname: document.getElementById('fullname').value
      })
    });

    const { data } = await paymentResponse.json();

    // Redirect to Seerbit checkout
    window.location.href = data.redirectUrl;
  } catch (error) {
    console.error('Payment initiation failed:', error);
  }
}

// 2. Server-side: Handle payment initialization
app.post('/api/seerbit/payment', async (req, res) => {
  const bearerToken = await getEncryptedKey();
  const reference = `ref-${Date.now()}`;

  const response = await axios.post(
    'https://seerbitapi.com/api/v2/payments/initiates',
    {
      publicKey: process.env.SEERBIT_PUBLIC_KEY,
      amount: String(req.body.amount),
      currency: req.body.currency,
      paymentType: req.body.paymentType,
      reference: reference,
      email: req.body.email,
      fullname: req.body.fullname,
      redirectUrl: `${process.env.BASE_URL}/payment/callback`,
      callbackUrl: `${process.env.BASE_URL}/api/webhook/seerbit`
    },
    {
      headers: {
        'Authorization': `Bearer ${bearerToken}`,
        'Content-Type': 'application/json'
      }
    }
  );

  // Store order
  await db.orders.create({
    reference: reference,
    amount: req.body.amount,
    currency: req.body.currency,
    email: req.body.email,
    status: 'pending'
  });

  res.json(response.data);
});

// 3. Handle webhook notification
app.post('/api/webhook/seerbit', express.json(), (req, res) => {
  res.status(200).json({ success: true });

  const eventData = req.body.notificationItems[0].notificationRequestItem.data;

  if (eventData.paymentStatus === 'successful') {
    // Mark order as paid
    db.orders.updateOne(
      { reference: eventData.reference },
      { status: 'paid', transactionRef: eventData.transactionReference }
    );

    // Send confirmation
    sendOrderConfirmation(eventData.email, eventData.reference);
  }
});
```

### Pattern 2: USSD Payment (Mobile-Friendly)

```javascript
async function initiateUSSDPayment(phoneNumber, amount) {
  const bearerToken = await getEncryptedKey();
  const reference = `ref-ussd-${Date.now()}`;

  const response = await axios.post(
    'https://seerbitapi.com/api/v2/payments/initiates',
    {
      publicKey: process.env.SEERBIT_PUBLIC_KEY,
      amount: String(amount),
      currency: 'NGN',
      paymentType: 'USSD',
      reference: reference,
      mobileNumber: phoneNumber,
      email: 'customer@example.com',
      productDescription: 'Mobile Payment',
      callbackUrl: `${process.env.BASE_URL}/api/webhook/seerbit`
    },
    {
      headers: {
        'Authorization': `Bearer ${bearerToken}`,
        'Content-Type': 'application/json'
      }
    }
  );

  // USSD payment returns USSD code
  const ussdCode = response.data.data.ussdCode;
  console.log(`USSD Code: ${ussdCode}`);

  return ussdCode;
}
```

### Pattern 3: Bank Transfer / Virtual Account

```javascript
async function createVirtualAccount(customerId) {
  const bearerToken = await getEncryptedKey();

  const response = await axios.post(
    'https://seerbitapi.com/api/v2/virtual-accounts',
    {
      publicKey: process.env.SEERBIT_PUBLIC_KEY,
      customerId: customerId,
      email: 'customer@example.com',
      fullname: 'John Doe'
    },
    {
      headers: {
        'Authorization': `Bearer ${bearerToken}`,
        'Content-Type': 'application/json'
      }
    }
  );

  // Return virtual account details
  return {
    accountNumber: response.data.data.accountNumber,
    bankCode: response.data.data.bankCode,
    bankName: response.data.data.bankName,
    amount: response.data.data.amount
  };
}
```

### Pattern 4: Recurring Billing / Subscriptions

```javascript
async function createRecurringPayment(cardToken, billingPeriod) {
  const bearerToken = await getEncryptedKey();

  // First authorize the card (if not already tokenized)
  const authResponse = await axios.post(
    'https://seerbitapi.com/api/v2/payments/charge-token',
    {
      publicKey: process.env.SEERBIT_PUBLIC_KEY,
      amount: '0', // Authorization only
      currency: 'NGN',
      reference: `ref-auth-${Date.now()}`,
      authorizationCode: cardToken,
      email: 'customer@example.com'
    },
    {
      headers: {
        'Authorization': `Bearer ${bearerToken}`,
        'Content-Type': 'application/json'
      }
    }
  );

  // Store subscription with tokenized card
  const subscription = await db.subscriptions.create({
    customerId: customerId,
    authorizationCode: cardToken,
    billingPeriod: billingPeriod, // 'daily', 'weekly', 'monthly', 'yearly'
    amount: 5000,
    currency: 'NGN',
    status: 'active',
    nextBillingDate: calculateNextBillingDate(billingPeriod)
  });

  return subscription;
}

// Scheduled job to charge recurring payments
async function processRecurringBillings() {
  const subscriptions = await db.subscriptions.find({
    status: 'active',
    nextBillingDate: { $lte: new Date() }
  });

  for (const sub of subscriptions) {
    const bearerToken = await getEncryptedKey();

    try {
      const response = await axios.post(
        'https://seerbitapi.com/api/v2/payments/charge-token',
        {
          publicKey: process.env.SEERBIT_PUBLIC_KEY,
          amount: String(sub.amount),
          currency: sub.currency,
          reference: `ref-recurring-${sub._id}-${Date.now()}`,
          authorizationCode: sub.authorizationCode,
          email: sub.customerEmail
        },
        {
          headers: {
            'Authorization': `Bearer ${bearerToken}`,
            'Content-Type': 'application/json'
          }
        }
      );

      if (response.data.data.paymentStatus === 'successful') {
        // Update next billing date
        const nextDate = calculateNextBillingDate(sub.billingPeriod);
        await db.subscriptions.updateOne(
          { _id: sub._id },
          { nextBillingDate: nextDate }
        );
      }
    } catch (error) {
      console.error(`Recurring billing failed for subscription ${sub._id}:`, error);
      // Optionally disable subscription after multiple failures
    }
  }
}
```

## Webhook Signature Verification

SeerBit signs webhook payloads so you can verify they are genuinely from SeerBit and not spoofed. The signature is sent in the `X-Seerbit-Signature` header.

```javascript
const crypto = require('crypto');

function verifySeerbitWebhook(payload, signature, secretKey) {
  // SeerBit signs: HMAC-SHA512 of the raw request body using your secret key
  const expectedSignature = crypto
    .createHmac('sha512', secretKey)
    .update(JSON.stringify(payload))   // use raw body string if possible
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expectedSignature, 'hex')
  );
}

app.post('/webhook/seerbit', express.json(), (req, res) => {
  const signature = req.headers['x-seerbit-signature'];
  const secretKey = process.env.SEERBIT_SECRET_KEY;

  if (!verifySeerbitWebhook(req.body, signature, secretKey)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process verified webhook
  res.status(200).json({ success: true });
});
```

> ⚠️ If SeerBit does not send an `X-Seerbit-Signature` header in your integration, use your transaction reference to look up and verify the transaction status via `GET /api/v2/transactions/{reference}` instead of trusting the webhook payload directly.

## Payment Links (Checkout Page)

SeerBit payment links allow you to collect payments without any frontend code — share a hosted checkout URL with your customer.

### Create a Payment Link
```
POST /api/v2/payments/standard/payment-link
Authorization: Bearer {bearer_token}
Content-Type: application/json

{
  "amount": "50000",
  "currency": "NGN",
  "description": "Payment for Order #12345",
  "email": "customer@example.com",
  "fullName": "Adaeze Obi",
  "phoneNumber": "+2348012345678",
  "reference": "unique_ref_" + Date.now(),
  "callbackUrl": "https://yourapp.com/payment-complete",
  "publicKey": "SBTESTPUBK_your_public_key"
}
```
Response includes `checkoutUrl` — redirect the customer to this URL to complete payment. No card data ever touches your server.

## Error Handling

### Common Error Codes

| Code | Message | Meaning | Resolution |
|------|---------|---------|-----------|
| 00 | Successful / Approved by Financial Institution | Transaction processed successfully | No action needed |
| 01 | Invalid API key | Authentication failed | Verify public/secret keys |
| 02 | Merchant not found | Invalid merchant account | Check merchant credentials |
| 03 | Invalid amount | Amount is invalid or zero | Ensure amount > 0 |
| 04 | Invalid reference | Reference format incorrect or duplicate | Use unique, valid reference |
| 05 | Invalid email | Email format is incorrect | Provide valid email |
| 06 | Invalid phone number | Phone number format is invalid | Use E.164 format |
| 10 | Transaction pending | Payment still processing | Wait for webhook confirmation |
| 15 | Insufficient funds | Customer account insufficient balance | Request new payment attempt |
| 20 | Transaction declined | Card/account declined by processor | Try alternative payment method |
| 25 | Invalid card details | Card number, CVV, or expiry invalid | Verify card information |
| 30 | 3D Secure required | Additional authentication needed | Complete 3D Secure verification |
| 50 | Network error | Connection failed | Retry request |
| 99 | Generic error | Unexpected error occurred | Check logs and retry |

### Error Response Example

```json
{
  "code": "03",
  "status": "FAILED",
  "message": "Invalid amount",
  "data": null
}
```

### Best Practices for Error Handling

```javascript
async function makePaymentWithErrorHandling(paymentData) {
  const bearerToken = await getEncryptedKey();

  try {
    const response = await axios.post(
      'https://seerbitapi.com/api/v2/payments/initiates',
      paymentData,
      {
        headers: {
          'Authorization': `Bearer ${bearerToken}`,
          'Content-Type': 'application/json'
        },
        timeout: 10000 // 10 second timeout
      }
    );

    if (response.data.code === '00') {
      return { success: true, data: response.data.data };
    } else {
      // Handle Seerbit error codes
      return {
        success: false,
        code: response.data.code,
        message: response.data.message,
        userMessage: getCustomErrorMessage(response.data.code)
      };
    }
  } catch (error) {
    if (error.response) {
      // HTTP error response
      console.error('HTTP Error:', error.response.status, error.response.data);
      return {
        success: false,
        error: 'Payment gateway error',
        details: error.response.data
      };
    } else if (error.code === 'ECONNABORTED') {
      // Timeout
      return {
        success: false,
        error: 'Request timeout',
        retry: true
      };
    } else {
      // Network or other error
      console.error('Request Error:', error.message);
      return {
        success: false,
        error: 'Network error',
        retry: true
      };
    }
  }
}

function getCustomErrorMessage(code) {
  const messages = {
    '01': 'Payment gateway authentication failed. Please contact support.',
    '03': 'Invalid payment amount. Please check and try again.',
    '04': 'Transaction reference already exists. Please refresh and try again.',
    '05': 'Invalid email address. Please check and try again.',
    '15': 'Insufficient funds. Please try another payment method.',
    '20': 'Payment declined. Please try another card.',
    '25': 'Invalid card details. Please check and try again.',
    '30': 'Additional verification required. Please complete 3D Secure.',
    '50': 'Network error. Please try again.',
    '99': 'An error occurred. Please try again or contact support.'
  };

  return messages[code] || 'Payment failed. Please try again.';
}
```

## Important Notes / Gotchas

### 1. **Bearer Token Expiration**
The encrypted key (bearer token) may expire after a certain period. Implement token refresh logic to avoid "Invalid API key" errors. Store the token timestamp and refresh periodically (recommend every 30 minutes).

```javascript
let cachedToken = null;
let tokenExpiredAt = null;

async function getValidBearerToken() {
  const now = Date.now();

  if (cachedToken && tokenExpiredAt && now < tokenExpiredAt) {
    return cachedToken;
  }

  const newToken = await getEncryptedKey();
  cachedToken = newToken;
  tokenExpiredAt = now + (30 * 60 * 1000); // 30 minutes

  return newToken;
}
```

### 2. **Reference Uniqueness**
The `reference` field must be unique for every transaction. Duplicates will fail. Use timestamps, UUIDs, or database IDs to ensure uniqueness.

```javascript
// Good
const reference = `ref-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

// Bad - can have duplicates
const reference = `ref-${customerId}`;
```

### 3. **Amount Precision**
Amounts should be in the smallest currency unit (kobo for NGN, pesewas for GHS). Don't use decimals; convert to integers.

```javascript
// Good: 5000 NGN = 500000 kobo
const amountInKobo = parseInt(5000 * 100);

// Bad: floating point
const amountInKobo = 5000.50 * 100; // Results in imprecise number
```

### 4. **USSD Codes Are Time-Limited**
USSD payment codes expire after a few minutes. Don't store them for later use; generate them only when customer is ready to pay immediately.

### 5. **Webhook Retry Strategy**
Seerbit retries webhooks for 7 days at increasing intervals (2 mins, 5 mins, 10 mins, 15 mins, 30 mins, 1 hour, 2 hours, 4 hours). Always implement idempotency to handle potential duplicates.

```javascript
// Check if event already processed
const existingEvent = await db.webhooks.findOne({ eventId: data.eventId });
if (existingEvent) {
  console.log('Duplicate webhook, skipping');
  return;
}
```

### 6. **3D Secure Authentication**
High-value transactions or international cards may require 3D Secure. The response may include a `redirectUrl` for customer authentication. Don't assume all transactions complete immediately; handle the pending state.

### 7. **Testing Environment**
Test with Seerbit test keys and test card numbers before going live. Documentation is available for different card scenarios (Visa, Mastercard, Verve, etc.). Use test phone numbers for USSD testing.

### 8. **Currency Support Varies by Payment Type**
Not all payment methods support all currencies. Verify supported currencies:
- **CARD**: NGN, GHS, KES, UGX, ZAR, USD, EUR
- **USSD**: NGN, GHS
- **TRANSFER**: NGN, GHS
- **MOBILEMONEY**: Varies by country

### 9. **Merchant Account Status**
Your Seerbit merchant account must be verified and settled before processing live transactions. Test transactions may work in sandbox, but live will fail if account is not properly configured.

### 10. **Card Tokenization Limitations**
- Tokens are valid for future charges but may expire after extended periods of non-use
- Always maintain error handling for failed token charges (customer may have cancelled/updated card)
- Implement a grace period (e.g., retry after 3 days) before disabling recurring billing

### 11. **Refund Processing Delays**
Refunds may take 2-5 business days to appear in customer accounts depending on the card issuer. Webhook notifications will confirm refund status, but settlement timing varies.

### 12. **PCI Compliance**
Never log, store, or transmit full card details. Let Seerbit handle card processing via their checkout or SDKs. Only store authorization codes for future charges.

## Useful Links

- **Official Documentation:** https://doc.seerbit.com/
- **API Reference (OpenAPI):** https://seerbit.github.io/openapi/
- **Authentication Guide:** https://apis.seerbit.com/authentication
- **Card Payment Docs:** https://doc.seerbit.com/payment-method/card-payment
- **USSD Payment Docs:** https://doc.seerbit.com/payment-method/ussd-payment
- **Bank Transfer Docs:** https://doc.seerbit.com/payment-method/transfer
- **Virtual Accounts:** https://doc.seerbit.com/online-payment/payment-features/virtual-accounts
- **Webhooks Documentation:** https://doc.seerbit.com/online-payment/after-payment/webhook-events
- **Subscriptions Guide:** https://doc.seerbit.com/online-payment/payment-features/subscription
- **Card Tokenization:** https://doc.seerbit.com/online-payment/payment-features/card-tokenisation
- **Status Codes Reference:** https://doc.seerbit.com/development-resources/seerbit-status-code
- **GitHub - Seerbit Repositories:** https://github.com/seerbit
- **Python SDK:** https://github.com/seerbit/seerbit-python-api-library
- **PHP SDK:** https://github.com/seerbit/seerbit-php-sdk
- **Node.js Integration Guide:** https://dev.to/seerbit_developers/integrating-seerbit-nodejs-sdk-in-your-application-445k
- **Release Notes:** https://releasenotes.seerbitapi.com/
- **Seerbit Developers Portal:** https://www.seerbit.com/developers
- **Recurring Payments Guide:** https://blog.seerbit.com/en/recurring-payments-made-simple-tools-for-subscription-based-businesses-in-africa
