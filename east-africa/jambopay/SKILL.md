---
name: JamboPay
description: Kenya's leading payment gateway supporting M-Pesa, Airtel Money, cards, and bank transfers. Express Checkout and Redirect integration options available.
triggers:
  - JamboPay
  - Kenya payment gateway
  - Kenyan payments
  - KES checkout
  - accept payments Kenya
  - M-Pesa payment
  - Airtel Money payment
---

# JamboPay Payment Gateway

JamboPay is a Central Bank of Kenya-authorized Payment Service Provider (PSP) offering secure payment collection across multiple channels. It supports M-Pesa, Airtel Money, bank cards (Visa, MasterCard, Union Pay), bank transfers, and integration with Kenyan banking networks including KenSwitch and 32+ partner banks.

## When to use this skill

Use JamboPay when you need to:
- Accept payments in Kenyan Shillings (KES) from Kenyan customers
- Process M-Pesa or Airtel Money payments
- Support card payments (Visa, MasterCard, Union Pay) in Kenya
- Integrate with Kenyan banking infrastructure
- Accept payments with minimal PCI compliance burden (hosted checkout)
- Provide both embedded (Express Checkout) and redirect payment flows
- Receive instant payment notifications via IPN/webhooks

**Best suited for:** E-commerce, SaaS subscriptions, ticketing, bill payments, and any Kenya-focused payment collection.

## Authentication

JamboPay uses bearer token authentication for API requests. All API communication must include an Authorization header with a valid bearer token.

### Obtaining Bearer Token

```bash
# Request a token from the Token endpoint
curl -X POST https://api.checkoutv3.jambopay.com/Token \
  -H "Content-Type: application/json" \
  -d '{
    "username": "YOUR_USERNAME",
    "password": "YOUR_PASSWORD"
  }'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### API Request Headers

Include the bearer token in all API requests:

```bash
curl -X POST https://api.checkoutv3.jambopay.com/payment/request \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ ... }'
```

### Environment-Specific URLs

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| Production | `https://api.checkoutv3.jambopay.com/` | Live payment processing |
| Sandbox | `https://checkout-test.jambopay.co.ke/` | Testing and development |

### Merchant Credentials

You'll need the following credentials from your JamboPay merchant account:
- **ClientKey**: Unique identifier for your merchant application
- **BusinessEmail**: Email address registered with JamboPay
- **Bearer Token**: Obtained via the Token endpoint (see above)

## Core API Reference

### 1. Initiate Payment / Payment Request

Initiate a payment transaction that can be processed via Express Checkout or Redirect Checkout.

**Endpoint:** `POST /payment/request`

**Request Body:**
```json
{
  "orderId": "ORDER-12345",
  "customerEmail": "customer@example.com",
  "customerPhone": "+254712345678",
  "currency": "KES",
  "orderAmount": 5000.00,
  "businessEmail": "merchant@yourcompany.com",
  "clientKey": "YOUR_CLIENT_KEY",
  "callbackUrl": "https://yoursite.com/payment/callback",
  "cancelledUrl": "https://yoursite.com/payment/cancelled",
  "description": "Purchase of Product XYZ",
  "storeName": "Your Store Name",
  "supportEmail": "support@yourcompany.com",
  "supportPhone": "+254712345678",
  "useJPLogo": true
}
```

**Response:**
```json
{
  "orderId": "ORDER-12345",
  "checkoutUrl": "https://checkout.jambopay.com/pay/ORDER-12345",
  "merchantReference": "JAMBOPAY-ORDER-12345",
  "status": "PENDING",
  "amount": 5000.00,
  "currency": "KES"
}
```

### 2. Query Payment Status

Check the status of a previously initiated payment.

**Endpoint:** `GET /payment/query/{merchantReference}`

**Request:**
```bash
curl -X GET "https://api.checkoutv3.jambopay.com/payment/query/JAMBOPAY-ORDER-12345" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Response:**
```json
{
  "merchantReference": "JAMBOPAY-ORDER-12345",
  "orderId": "ORDER-12345",
  "status": "COMPLETED",
  "amount": 5000.00,
  "currency": "KES",
  "paymentMethod": "MPESA",
  "transactionId": "MPESA-TRANS-123456",
  "processedDate": "2024-02-24T14:30:00Z",
  "customerPhone": "+254712345678"
}
```

### 3. Refund Transaction

Refund a completed payment transaction.

**Endpoint:** `POST /payment/refund`

**Request Body:**
```json
{
  "merchantReference": "JAMBOPAY-ORDER-12345",
  "refundAmount": 5000.00,
  "reason": "Customer requested cancellation",
  "refundReference": "REFUND-12345"
}
```

**Response:**
```json
{
  "refundReference": "REFUND-12345",
  "merchantReference": "JAMBOPAY-ORDER-12345",
  "refundAmount": 5000.00,
  "status": "PROCESSING",
  "processedDate": "2024-02-24T14:35:00Z"
}
```

## Integration Methods

### Express Checkout (Embedded)

Customer completes payment within your website using an embedded form or iframe. Requires additional frontend integration with JamboPay's payment widgets.

**Advantages:**
- Better conversion rates (no redirect)
- Full UI control
- Faster checkout experience

**Implementation:**
```html
<!-- Include JamboPay checkout script -->
<script src="https://checkout.jambopay.com/js/checkout.js"></script>

<!-- Checkout container -->
<div id="jambopay-checkout"></div>

<script>
  var checkout = new JamboPay({
    merchantKey: "YOUR_CLIENT_KEY",
    orderId: "ORDER-12345",
    amount: 5000,
    email: "customer@example.com",
    phone: "+254712345678",
    currency: "KES",
    onSuccess: function(data) {
      console.log("Payment successful:", data);
    },
    onError: function(error) {
      console.error("Payment failed:", error);
    }
  });

  checkout.render("#jambopay-checkout");
</script>
```

### Redirect Checkout

Customer is redirected to JamboPay's hosted payment page. Simpler integration with minimal frontend requirements.

**Advantages:**
- Simplest integration
- PCI-DSS compliance handled by JamboPay
- No custom payment form needed

**Implementation:**
```javascript
// After initiating payment via API and receiving checkoutUrl
window.location.href = response.checkoutUrl;
```

## Webhooks and Callbacks

JamboPay sends payment notifications to your callback URL via Instant Payment Notification (IPN).

### Callback Configuration

Configure your callback URL in the JamboPay merchant dashboard:
1. Log in to [JamboPay Client Portal](https://client.jambopay.com/)
2. Navigate to Settings/Integrations
3. Set your **Callback URL** and **Cancel URL**

### IPN Callback Payload

JamboPay sends a POST request to your callback URL with the following structure:

```json
{
  "merchantReference": "JAMBOPAY-ORDER-12345",
  "orderId": "ORDER-12345",
  "status": "COMPLETED",
  "amount": 5000.00,
  "currency": "KES",
  "paymentMethod": "MPESA",
  "transactionId": "MPESA-TXN-123456789",
  "customerPhone": "+254712345678",
  "customerEmail": "customer@example.com",
  "processedDate": "2024-02-24T14:30:00Z",
  "description": "Purchase of Product XYZ",
  "metadata": {
    "customField1": "value1"
  }
}
```

### Handling Callbacks Securely

**Best Practices:**
1. Always validate the callback is from JamboPay (implement signature verification if available)
2. Log all callbacks for auditing
3. Return HTTP 200 immediately upon receipt
4. Process payment asynchronously (don't hold the callback response)
5. Implement idempotency to handle duplicate callbacks
6. Never rely solely on callbacks—always query the payment status

**Example Node.js Handler:**
```javascript
app.post('/payment/callback', (req, res) => {
  const payload = req.body;

  // Log for auditing
  console.log('Callback received:', payload);

  // Return 200 immediately
  res.status(200).json({ status: 'received' });

  // Process payment asynchronously
  processPayment(payload).catch(error => {
    console.error('Callback processing error:', error);
    // Retry logic here
  });
});

async function processPayment(payload) {
  // Verify payment with query endpoint
  const status = await queryPaymentStatus(payload.merchantReference);

  if (status === 'COMPLETED') {
    // Update order in your database
    await updateOrderStatus(payload.orderId, 'PAID');
    // Send confirmation email
    await sendConfirmationEmail(payload.customerEmail);
  }
}
```

### Cancelled Payment Callback

When a customer cancels a transaction, JamboPay redirects to the `cancelledUrl` with parameters:

```
https://yoursite.com/payment/cancelled?orderId=ORDER-12345&reason=USER_CANCELLED
```

## Common Integration Patterns

### Pattern 1: Simple E-commerce Checkout

```javascript
// 1. User clicks "Checkout"
async function initiateCheckout(orderDetails) {
  const response = await fetch('https://api.checkoutv3.jambopay.com/payment/request', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      orderId: orderDetails.id,
      customerEmail: orderDetails.email,
      customerPhone: orderDetails.phone,
      currency: 'KES',
      orderAmount: orderDetails.totalAmount,
      businessEmail: 'merchant@yourcompany.com',
      clientKey: 'YOUR_CLIENT_KEY',
      callbackUrl: 'https://yoursite.com/api/payment/callback',
      cancelledUrl: 'https://yoursite.com/checkout?cancelled=true',
      description: `Order #${orderDetails.id}`
    })
  });

  const data = await response.json();

  // 2. Redirect to JamboPay
  window.location.href = data.checkoutUrl;
}

// 3. Receive callback and verify
app.post('/api/payment/callback', async (req, res) => {
  const { merchantReference, orderId, status } = req.body;

  res.status(200).send('OK');

  // Verify status
  const verified = await verifyPaymentStatus(merchantReference);
  if (verified.status === 'COMPLETED') {
    await updateOrderToPaid(orderId);
  }
});
```

### Pattern 2: M-Pesa Payment Flow

```javascript
// M-Pesa payments flow through the same API
// Customer selects M-Pesa as payment method on checkout
async function processM2bPayment(orderDetails) {
  const response = await fetch('https://api.checkoutv3.jambopay.com/payment/request', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      orderId: orderDetails.id,
      customerPhone: orderDetails.phone, // M-Pesa registered number
      orderAmount: orderDetails.amount,
      currency: 'KES',
      businessEmail: 'merchant@yourcompany.com',
      clientKey: 'YOUR_CLIENT_KEY',
      callbackUrl: 'https://yoursite.com/api/payment/callback',
      description: `M-Pesa Payment for Order #${orderDetails.id}`
    })
  });

  const data = await response.json();
  return data.checkoutUrl; // Redirect user to complete M-Pesa STK push
}
```

### Pattern 3: Subscription/Recurring Billing

For subscriptions, initiate payment for each billing cycle:

```javascript
async function processSubscriptionPayment(subscription) {
  const nextBillingDate = new Date(subscription.nextBillingDate);

  const response = await fetch('https://api.checkoutv3.jambopay.com/payment/request', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      orderId: `SUB-${subscription.id}-${Date.now()}`,
      customerEmail: subscription.customerEmail,
      customerPhone: subscription.customerPhone,
      orderAmount: subscription.billingAmount,
      currency: 'KES',
      businessEmail: 'merchant@yourcompany.com',
      clientKey: 'YOUR_CLIENT_KEY',
      callbackUrl: 'https://yoursite.com/api/subscription/payment-callback',
      description: `Subscription renewal: ${subscription.planName}`
    })
  });

  // Store the merchant reference for reconciliation
  await saveSubscriptionPayment(subscription.id, await response.json());
}
```

## Error Handling

### Common Error Responses

#### 401 Unauthorized
```json
{
  "error": "Unauthorized",
  "message": "Invalid or expired bearer token"
}
```
**Solution:** Refresh your access token via the Token endpoint.

#### 400 Bad Request
```json
{
  "error": "INVALID_REQUEST",
  "message": "Missing required field: customerEmail",
  "field": "customerEmail"
}
```
**Solution:** Validate all required fields before submitting.

#### 402 Payment Required
```json
{
  "error": "INSUFFICIENT_FUNDS",
  "message": "Customer account has insufficient funds"
}
```
**Solution:** Inform customer to top up their M-Pesa/Airtel account.

#### 404 Not Found
```json
{
  "error": "ORDER_NOT_FOUND",
  "message": "Order JAMBOPAY-ORDER-12345 does not exist"
}
```
**Solution:** Verify the merchant reference is correct.

#### 429 Too Many Requests
```json
{
  "error": "RATE_LIMITED",
  "message": "Too many requests. Please retry after 60 seconds"
}
```
**Solution:** Implement exponential backoff retry logic.

#### 500 Internal Server Error
```json
{
  "error": "INTERNAL_ERROR",
  "message": "An unexpected error occurred processing your request"
}
```
**Solution:** Retry request after a delay. Log the error for investigation.

### Implementing Retry Logic

```javascript
async function makeJamboPayRequest(endpoint, body, retries = 3) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      const response = await fetch(`https://api.checkoutv3.jambopay.com${endpoint}`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${accessToken}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(body)
      });

      if (response.status === 429 || response.status >= 500) {
        // Retry with exponential backoff
        if (attempt < retries) {
          const delay = Math.pow(2, attempt - 1) * 1000;
          await new Promise(resolve => setTimeout(resolve, delay));
          continue;
        }
      }

      if (!response.ok) {
        const error = await response.json();
        throw new Error(`JamboPay error: ${error.message}`);
      }

      return await response.json();
    } catch (error) {
      if (attempt === retries) throw error;
    }
  }
}
```

## Important Notes and Gotchas

1. **Bearer Token Expiration**: Access tokens expire after a set duration (typically 1 hour). Implement token refresh logic or request new tokens for each batch of requests. Store tokens securely and never expose them to the client-side.

2. **Callback URL Must Be Publicly Accessible**: Your callback URL must be accessible from JamboPay's servers. If you're testing locally, use tunneling tools like ngrok to expose your localhost to the internet. Avoid using private IP addresses or localhost URLs.

3. **Phone Number Format**: Customer phone numbers must include the country code. For Kenya, use the format `+254XXXXXXXXX`. Avoid formats like `07XXXXXXXX` which will be rejected.

4. **Idempotent Callback Handling**: Implement idempotency keys or track processed callbacks by merchant reference. JamboPay may send duplicate callbacks for the same transaction, and your system must handle this gracefully without creating duplicate orders.

5. **Currency is Always KES**: JamboPay only supports Kenyan Shillings (KES). Do not attempt to use other currency codes. Convert any non-KES amounts to KES before submission.

6. **Order ID Uniqueness**: OrderId values should be unique for each payment attempt. Reusing the same OrderId may cause unexpected behavior or conflicts in JamboPay's system. Consider including timestamps or random tokens in order IDs.

7. **Webhook Signature Verification**: Although not always enforced, validate that callbacks originate from JamboPay. Implement signature verification if JamboPay provides webhook signing capabilities, or whitelist JamboPay's IP addresses in your firewall.

8. **Test Environment Limitations**: The sandbox environment (`checkout-test.jambopay.co.ke`) uses test credentials and may have limited features. Always test critical flows thoroughly before deploying to production. Use sandbox phone numbers and test card details provided by JamboPay.

9. **Merchant Reference Tracking**: The `merchantReference` returned by JamboPay is essential for status queries and refunds. Store this reference alongside your OrderId in your database for easy reconciliation and debugging.

10. **PCI-DSS Compliance**: Even with JamboPay's hosted checkout, ensure your website meets PCI-DSS compliance standards. Never store credit card details on your servers. Use HTTPS for all pages handling payment information.

11. **Callback Timing**: Payment callbacks may not arrive immediately. There can be delays ranging from seconds to minutes. Do not assume a payment was unsuccessful if the callback hasn't arrived within a few seconds. Always query the payment status as the authoritative source.

12. **Customer Email Requirement**: CustomerEmail is required for most payment methods and is used for payment confirmations. Ensure this field is always provided and valid. Some payment methods may require this for compliance reporting.

## Useful Links

- [JamboPay Official Website](https://jambopay.com/)
- [JamboPay Developer Documentation](https://backoffice.jambopay.com/developer.aspx)
- [JamboPay Checkout API Docs](https://apidocs.jambopay.co.ke/checkoutv3/integratingCheckout.html)
- [JamboPay API Services Portal](https://apidocs.jambopay.co.ke/)
- [JamboPay Client Portal (Merchant Dashboard)](https://client.jambopay.com/)
- [JamboPay npm Package](https://www.npmjs.com/package/jambopay)
- [JamboPay PHP SDK](https://packagist.org/packages/bixbyte/jambopay)
- [JamboPay Postman Collection](https://www.postman.com/cloudy-space-8697/workspace/jambopay/)

## Additional Resources

**Payment Methods Supported:**
- M-Pesa (Safaricom)
- Airtel Money
- Visa/MasterCard (Credit & Debit)
- Union Pay
- Bank Transfers (KenSwitch, 32+ partner banks)
- YU-CASH

**Compliance & Security:**
- Central Bank of Kenya (CBK) Authorized
- PCI-DSS Compliant
- SSL/TLS Encryption
- Two-Factor Authentication (available)

**Support Channels:**
- Email: support@jambopay.com
- Phone: +254 (0) 20 2665000
- Dashboard: https://client.jambopay.com/ (submit support tickets)
