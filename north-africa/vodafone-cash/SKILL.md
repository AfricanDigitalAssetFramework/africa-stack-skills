---
name: vodafone-cash
description: "Integrate Vodafone Cash mobile money payments in Egypt through payment aggregators. Use this skill when the user wants to accept Vodafone Cash payments, integrate mobile money into their app, process payments via Vodafone Cash, handle mobile money transactions in Egypt, or work with Vodafone Cash payment integration. Trigger when the user mentions 'Vodafone Cash', 'Vodafone Egypt payments', 'mobile money Egypt', or needs payment solutions for Egypt's 40M+ Vodafone Cash users."
---

# Vodafone Cash Integration Skill

Vodafone Cash is Egypt's leading mobile money service with 40+ million users. It enables customers to send, receive, and pay directly from their Vodafone accounts without needing a separate bank account. For merchants, Vodafone Cash integration is achieved through payment aggregators rather than direct API access.

## When to use this skill

You're building an Egyptian e-commerce platform, digital services marketplace, or utilities billing system that needs to accept mobile money payments. Vodafone Cash reaches nearly 40% of Egypt's adult population and represents the most accessible payment method for unbanked and underbanked customers. Integration through aggregators provides secure, standardized payment processing with built-in compliance and fraud detection.

## Integration Architecture

Vodafone Cash uses an **aggregator-based model** for merchant integration. There is no publicly available direct merchant REST API from Vodafone. Instead, merchants integrate through payment gateway aggregators that handle the complexity of Vodafone Cash's backend while providing a standardized API interface.

**Key Aggregators Supporting Vodafone Cash in Egypt:**
- **Paymob** (most popular for Egypt)
- **Fawry**
- **PaymentWall**
- **Yuno**
- **Various PSPs** (Payment Service Providers)

This approach ensures compliance, reduces onboarding complexity, and provides unified payment handling across multiple methods.

## Authentication via Aggregator (Paymob Example)

Most aggregators use API keys for authentication. Here's the typical flow using Paymob as an example:

```
POST https://accept.paymob.com/api/auth/tokens
Body:
{
  "api_key": "YOUR_PAYMOB_API_KEY"
}

Response:
{
  "token": "YOUR_AUTH_TOKEN"
}
```

Store your API keys securely in environment variables. Never hardcode credentials.

**Base URL (Paymob):** `https://accept.paymob.com/api`

## Core Integration Patterns

### Pattern 1: Paymob Payment Collection Flow

Paymob is the most comprehensive aggregator for Vodafone Cash in Egypt.

#### 1. Authenticate

```
POST /auth/tokens
Body:
{
  "api_key": "YOUR_API_KEY"
}

Response:
{
  "token": "YOUR_AUTH_TOKEN"
}
```

#### 2. Create Order

```
POST /ecommerce/orders
Headers:
Authorization: Bearer {auth_token}
Content-Type: application/json

Body:
{
  "auth_token": "YOUR_AUTH_TOKEN",
  "delivery_needed": false,
  "amount_cents": 5000,
  "currency": "EGP",
  "items": [
    {
      "name": "Product Name",
      "amount_cents": 5000,
      "quantity": 1,
      "description": "Product description"
    }
  ],
  "shipping_data": {
    "apartment": "NA",
    "email": "customer@email.com",
    "floor": "NA",
    "first_name": "Ahmed",
    "street": "NA",
    "phone_number": "+201234567890",
    "postal_code": "NA",
    "city": "NA",
    "country": "EG",
    "last_name": "Hassan",
    "building": "NA"
  }
}

Response:
{
  "id": 123456,
  "created_at": "2025-02-24T10:00:00.000Z",
  "delivery_needed": false,
  "amount_cents": 5000,
  "currency": "EGP",
  "merchant_order_id": null,
  "items": [...]
}
```

**Important:** Amount is in **piasters** (Egyptian currency cents). 50 EGP = 5000 piasters.

#### 3. Create Payment Key (for payment processing)

```
POST /acceptance/payment_keys
Headers:
Content-Type: application/json

Body:
{
  "auth_token": "YOUR_AUTH_TOKEN",
  "amount_cents": 5000,
  "expiration": 3600,
  "order_id": 123456,
  "billing_data": {
    "apartment": "NA",
    "email": "customer@email.com",
    "floor": "NA",
    "first_name": "Ahmed",
    "street": "NA",
    "phone_number": "+201234567890",
    "postal_code": "NA",
    "city": "NA",
    "country": "EG",
    "last_name": "Hassan",
    "state": "Cairo"
  },
  "currency": "EGP",
  "integration_id": YOUR_INTEGRATION_ID,
  "lock_order_when_paid": true
}

Response:
{
  "token": "PAYMENT_KEY_TOKEN",
  "order_id": 123456,
  "amount_cents": 5000,
  "expiration": 3600,
  "success": true
}
```

Get your `integration_id` from your Paymob merchant dashboard.

#### 4. Initiate Payment (Customer Completes Payment)

Redirect customer to Paymob hosted payment page or use iframe:

```
https://accept.paymob.com/api/acceptance/iframes/{your_iframe_id}?payment_token={PAYMENT_KEY_TOKEN}
```

Customer selects Vodafone Cash as payment method, enters phone number, receives SMS with OTP, confirms payment.

#### 5. Verify Payment Status

```
GET /acceptance/transactions/{transaction_id}
Headers:
Authorization: Bearer {auth_token}

Response:
{
  "id": 999999,
  "success": true,
  "is_refunded": false,
  "is_amount_cents": true,
  "amount_cents": 5000,
  "fees_cents": 150,
  "integration_id": 12345,
  "order": {
    "id": 123456,
    "amount_cents": 5000,
    "status": "PAID"
  },
  "created_at": "2025-02-24T10:05:00.000Z",
  "currency": "EGP",
  "source_data": {
    "type": "vodafone_cash",
    "pan": "****1234",
    "sub_type": "wallet"
  }
}
```

### Pattern 2: Fawry Integration (Alternative Aggregator)

Fawry is another popular aggregator for Vodafone Cash in Egypt.

```
POST https://www.atfawry.com/ECommerceWeb/Fawry/payments/charge
Content-Type: application/json

Body:
{
  "merchantCode": "YOUR_MERCHANT_CODE",
  "merchantRefNum": "ORD-2025-12345",
  "customerProfileId": "CUSTOMER_ID",
  "paymentMethod": "MWALLET",
  "customerMobile": "+201234567890",
  "customerEmail": "customer@email.com",
  "customerName": "Ahmed Hassan",
  "chargeItems": [
    {
      "itemId": "PROD-123",
      "description": "Product",
      "price": 50.00,
      "quantity": 1
    }
  ],
  "currencyCode": "EGP",
  "paymentExpiry": 1737072000000,
  "signature": "CALCULATED_SIGNATURE"
}
```

Signature is plain **SHA-256** (not HMAC) of: `merchantCode + merchantRefNum + customerProfileId + paymentMethod + amount.toFixed(2) + securityKey`

## Real Payment Flow: SMS Authorization

The actual Vodafone Cash payment flow works as follows:

1. **Customer initiates payment** on aggregator's payment page or app
2. **Selects Vodafone Cash** as payment method
3. **Enters phone number** associated with Vodafone account
4. **Vodafone sends SMS with OTP** (One-Time Password) to customer's phone
5. **Customer enters OTP** to confirm and authorize payment
6. **Payment is processed** and customer receives confirmation SMS
7. **Merchant receives webhook notification** (through aggregator)

**No OAuth, JWT, or complex authentication required** for the customer. The flow is entirely SMS-based and designed for simplicity on low-bandwidth networks.

## Webhooks / Callbacks

Aggregators send webhooks when payment status changes. Verify webhook signatures for security:

**Paymob Webhook Example:**

```
POST https://yoursite.com/webhook
Headers:
Content-Type: application/json
X-HmacSHA256: SIGNATURE_VERIFICATION_HASH

Body:
{
  "type": "TRANSACTION",
  "obj": {
    "id": 999999,
    "created_at": "2025-02-24T10:05:00.000Z",
    "amount_cents": 5000,
    "success": true,
    "is_refunded": false,
    "order": {
      "id": 123456,
      "amount_cents": 5000,
      "status": "PAID"
    },
    "source_data": {
      "type": "vodafone_cash",
      "sub_type": "wallet"
    }
  }
}
```

Verify signature:

```javascript
const crypto = require('crypto');
const hmacSignature = crypto
  .createHmac('sha256', YOUR_HMAC_SECRET)
  .update(JSON.stringify(req.body))
  .digest('hex');

if (hmacSignature === req.headers['x-hmac-sha256']) {
  // Valid webhook
  console.log('Payment confirmed:', req.body.obj);
}
```

## Refunds

Refunds are processed through the aggregator:

**Paymob Refund:**

```
POST /acceptance/refunds
Headers:
Authorization: Bearer {auth_token}
Content-Type: application/json

Body:
{
  "transaction_id": 999999,
  "amount_cents": 5000
}

Response:
{
  "id": 1111111,
  "transaction_id": 999999,
  "amount_cents": 5000,
  "reason": "Customer requested refund",
  "created_at": "2025-02-24T10:10:00.000Z",
  "status": "ACCEPTED"
}
```

Refunds typically complete within 24-48 hours and appear as credits to the customer's Vodafone Cash wallet.

## Common Integration Patterns

### E-Commerce Store
1. Customer adds items to cart
2. Checkout page presents payment methods including Vodafone Cash
3. Customer enters phone number and authorizes via OTP
4. Webhook confirms payment completion
5. Fulfill order

### Bill Payment / Subscription
1. Display payment options to customer
2. Customer chooses Vodafone Cash
3. System initiates charge through aggregator
4. Customer confirms via SMS OTP
5. Update subscription/billing status on webhook

### Withdrawal (Reverse Flow)
1. Customer requests withdrawal/cash-out
2. System calls aggregator cash-out API with customer's Vodafone number
3. Amount credited to Vodafone Cash wallet
4. Customer receives SMS confirmation

## Important Notes / Gotchas

### Critical Limitations

**No Direct API:**
Vodafone does not provide a public merchant REST API for Vodafone Cash. All integration must go through aggregators. This is the industry standard in Egypt.

**Aggregator Dependency:**
Your payment flow depends entirely on your chosen aggregator's reliability, uptime, and feature availability. If Paymob is down, payments fail until restored.

**Phone Number Verification:**
Customers must have an active Vodafone line. The system verifies the phone number is associated with an active Vodafone account before processing payment.

**SMS Dependency:**
The entire authorization flow depends on SMS delivery, which can be delayed or fail in areas with poor network coverage. Always implement timeout handling and retry logic.

**User Base is Large but Not Universal:**
While Vodafone Cash has 40M+ users, not all Egyptian customers have Vodafone accounts. Offer multiple payment methods.

**Piaster Currency Units:**
All amounts are in piasters (cents). 1 EGP = 100 piasters. Never confuse these units.

**Rate Limiting:**
Aggregators enforce rate limits on API calls. Implement exponential backoff for retries.

**Settlement Delays:**
Funds may take 2-7 business days to settle to your merchant account depending on the aggregator.

**Compliance & KYC:**
Vodafone Cash transactions are subject to Egypt's mobile money regulations. Merchants may need to provide KYC documentation.

## Error Handling

**Common Aggregator Errors:**

- **400 Bad Request**: Invalid parameters (wrong phone format, missing fields)
- **401 Unauthorized**: Invalid API key or expired token
- **402 Payment Required**: Insufficient funds in customer's Vodafone Cash wallet
- **404 Not Found**: Transaction ID not found
- **429 Too Many Requests**: Rate limit exceeded, implement exponential backoff
- **500 Server Error**: Aggregator issue, retry with backoff

Always log full error responses for debugging. Implement user-friendly error messages (don't expose raw API errors).

## Choosing an Aggregator

| Aggregator | Coverage | Documentation | Fees | Best For |
|-----------|----------|----------------|------|----------|
| **Paymob** | Egypt, Africa | Excellent | ~2.9% | High-volume e-commerce |
| **Fawry** | Egypt, Sudan | Good | ~2.5% | Bill payments, utilities |
| **PaymentWall** | Global + Egypt | Comprehensive | ~3.5% | International + Egypt |
| **Yuno** | LatAm, Africa | Good | Variable | Cross-border merchants |

For Egypt-focused businesses, **Paymob** is the most recommended.

## Setup Checklist

- [ ] Register merchant account with chosen aggregator (Paymob, Fawry, etc.)
- [ ] Obtain API keys and integration IDs from merchant dashboard
- [ ] Store credentials in secure environment variables
- [ ] Set up webhook endpoint with HTTPS
- [ ] Implement signature verification for webhooks
- [ ] Test payment flow in sandbox environment
- [ ] Implement error handling and retry logic
- [ ] Add customer support documentation for Vodafone Cash payment method
- [ ] Test with real Vodafone line (if aggregator permits)
- [ ] Go live with production credentials

## Useful Links

- [Vodafone Developer Marketplace](https://developer.vodafone.com/)
- [Paymob Developer Portal](https://developers.paymob.com/)
- [Paymob Egypt Developer Hub](https://developers.paymob.com/hub/egypt)
- [Fawry API Documentation](https://fawaterak-api.readme.io/)
- [PaymentWall Mobile Wallets Documentation](https://docs.paymentwall.com/payment-method/mobilewalletseg)
- [Vodafone Cash Official Service](https://web.vodafone.com.eg/en/vodafone-cash)
- [GitHub: Payment Gateway Integrations](https://github.com/Nafezly/payments)
