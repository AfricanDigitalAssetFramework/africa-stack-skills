---
name: fawry
description: "Integrate with Fawry's payment API for Egyptian commerce. Use this skill whenever the user wants to accept payments in Egypt, process EGP transactions, handle Fawry reference number payments, integrate buy-now-pay-later (BNPL) in Egypt, generate invoices, process refunds, or work with Fawry in any way. Also trigger for 'Fawry', 'Egyptian payments', 'EGP checkout', 'pay at Fawry', 'Egyptian e-commerce', 'Fawry reference number', 'Egypt BNPL', 'FawryPay', or when the user needs to accept payments from Egyptian customers including cash payments at Fawry retail outlets."
---

# Fawry Integration Skill

Fawry is Egypt's largest electronic payment network with 250,000+ service points, 50M+ customers, and handles both online and offline payments. It's unique in Africa because it bridges digital payments with a massive physical cash collection network — customers can pay online OR generate a reference number and pay cash at any Fawry retail outlet (pharmacies, supermarkets, etc.).

## When to use this skill

You're building something that needs to accept payments in Egypt. Fawry is essential because a large portion of Egyptian consumers prefer cash-based payments. With Fawry, you get card payments, wallet payments, AND cash-at-retail — all through one API. It also offers BNPL (buy-now-pay-later) and invoicing.

## Authentication

Fawry uses a **Merchant Code** and **HMAC SHA-256 signature** for authentication. Every request is signed with your security key.

**Credentials:**
- `merchantCode` — Your Fawry merchant identifier
- `securityKey` — Used to compute HMAC signatures (never sent in requests)

Store in env vars: `FAWRY_MERCHANT_CODE`, `FAWRY_SECURITY_KEY`.

**Environment URLs:**
- Sandbox: `https://atfawry.fawrystaging.com`
- Production: `https://www.atfawry.com`

### Signature Computation

Each API call requires an HMAC SHA-256 signature computed from specific request fields concatenated together in a strict order. **Field order matters — incorrect ordering will result in signature verification failures.**

Fawry uses **plain SHA-256**, not HMAC-SHA256. The `securityKey` is appended to the string being hashed — it is NOT used as an HMAC key.

```javascript
const crypto = require('crypto');

function computeSignature(data) {
  return crypto.createHash('sha256')
    .update(data)
    .digest('hex');
}

// For payment initiation (charge):
// SHA-256(merchantCode + merchantRefNum + customerProfileId + paymentMethod + amount + securityKey)
// Note: amount must be formatted to 2 decimal places; securityKey appended at end
const signatureData = `${merchantCode}${merchantRefNum}${customerProfileId}${paymentMethod}${amount.toFixed(2)}${securityKey}`;
const signature = computeSignature(signatureData);

// For payment status check:
// SHA-256(merchantCode + merchantRefNum + securityKey)
const statusSignatureData = `${merchantCode}${merchantRefNum}${securityKey}`;
const statusSignature = computeSignature(statusSignatureData);

// For refund:
// SHA-256(merchantCode + referenceNumber + refundAmount + reason + securityKey)
const refundSignatureData = `${merchantCode}${referenceNumber}${refundAmount.toFixed(2)}${reason}${securityKey}`;
const refundSignature = computeSignature(refundSignatureData);
```

## Core API Reference

### Initiate a Payment

```
POST /ECommerceWeb/Fawry/payments/charge
Content-Type: application/json
```

```json
{
  "merchantCode": "YOUR_MERCHANT_CODE",
  "merchantRefNum": "order_ref_123",
  "customerName": "Ahmed Hassan",
  "customerMobile": "01012345678",
  "customerEmail": "ahmed@email.com",
  "customerProfileId": "cust_123",
  "paymentMethod": "PAYATFAWRY",
  "chargeItems": [
    {
      "itemId": "ITEM001",
      "description": "Premium Subscription",
      "price": 250.00,
      "quantity": 1
    }
  ],
  "currencyCode": "EGP",
  "paymentExpiry": 1737072000000,
  "signature": "computed_hmac_signature"
}
```

**Important details:**
- `paymentAmount` is in **major currency units** (EGP, not piasters). EGP 250 = `250.00`.
- `paymentExpiry` is a Unix timestamp in **milliseconds**.
- Phone numbers: Egyptian format `01XXXXXXXXX` (11 digits).
- `customerProfileId` can be any unique identifier for the customer in your system (email, user ID, etc.)

**Payment methods:**
- `PAYATFAWRY` — Cash at Fawry outlets (generates reference number for customer)
- `CARD` — Credit/debit card
- `MWALLET` — Mobile wallet (Vodafone Cash, Orange Money, Etisalat Cash, WE Pay, CIB Smart Wallet)
- `CASHONDELIVERY` — Cash on delivery
- `VALU` — ValU BNPL (installment payment)

**Response (PAYATFAWRY):**
```json
{
  "type": "ChargeResponse",
  "referenceNumber": "123456789",
  "merchantRefNumber": "order_ref_123",
  "orderAmount": 250.00,
  "fawryFees": 5.00,
  "paymentAmount": 255.00,
  "orderStatus": "NEW",
  "paymentMethod": "PAYATFAWRY",
  "expirationTime": 1737072000000,
  "statusCode": 200,
  "statusDescription": "Operation done successfully"
}
```

The customer takes the `referenceNumber` to any Fawry outlet and tells the cashier "I want to pay Fawry bill [reference number]". Once paid, you get a webhook notification.

**Response (CARD):**
```json
{
  "type": "ChargeResponse",
  "referenceNumber": "987654321",
  "merchantRefNumber": "order_ref_123",
  "paymentAmount": 250.00,
  "orderStatus": "PAID",
  "paymentMethod": "CARD",
  "statusCode": 200,
  "statusDescription": "Operation done successfully"
}
```

**Response (MWALLET):**
```json
{
  "type": "ChargeResponse",
  "referenceNumber": "555123456",
  "merchantRefNumber": "order_ref_123",
  "paymentAmount": 250.00,
  "orderStatus": "PENDING",
  "paymentMethod": "MWALLET",
  "statusCode": 200,
  "statusDescription": "Operation done successfully"
}
```

### Get Payment Status

```
GET /ECommerceWeb/Fawry/payments/status/v2?merchantCode={merchantCode}&merchantRefNumber={merchantRefNum}&signature={signature}
```

**Signature for status (critical field order):**
```
merchantCode + merchantRefNum + securityKey
```

**Response:**
```json
{
  "type": "PaymentStatusResponse",
  "referenceNumber": "123456789",
  "merchantRefNumber": "order_ref_123",
  "orderAmount": 250.00,
  "paymentAmount": 255.00,
  "fawryFees": 5.00,
  "orderStatus": "PAID",
  "paymentMethod": "PAYATFAWRY",
  "paymentTime": 1737100000000,
  "statusCode": 200,
  "statusDescription": "Operation done successfully"
}
```

**Order statuses:**
- `NEW` — Created, awaiting payment
- `PAID` — Successfully paid
- `CANCELLED` — Cancelled
- `EXPIRED` — Payment window expired
- `REFUNDED` — Refunded
- `FAILED` — Payment failed
- `PENDING` — Awaiting customer action (e.g., MWALLET confirmation)

### Initiate BNPL (Buy Now, Pay Later)

Fawry integrates with ValU for BNPL in Egypt:

```
POST /ECommerceWeb/Fawry/payments/charge
```

```json
{
  "merchantCode": "YOUR_MERCHANT_CODE",
  "merchantRefNum": "bnpl_ref_123",
  "customerName": "Ahmed Hassan",
  "customerMobile": "01012345678",
  "customerEmail": "ahmed@email.com",
  "customerProfileId": "cust_123",
  "paymentMethod": "VALU",
  "chargeItems": [
    {
      "itemId": "ITEM001",
      "description": "Laptop",
      "price": 15000.00,
      "quantity": 1
    }
  ],
  "currencyCode": "EGP",
  "valuTenureInMonths": 12,
  "signature": "computed_hmac_signature"
}
```

`valuTenureInMonths`: 6, 12, 18, 24, or 36 months.

### Validate BNPL Eligibility

Check if a customer is eligible for BNPL before initiating:

```
POST /ECommerceWeb/Fawry/payments/valu/eligibility
```

```json
{
  "merchantCode": "YOUR_MERCHANT_CODE",
  "customerMobile": "01012345678",
  "amount": 15000.00,
  "signature": "computed_hmac_signature"
}
```

**Signature for eligibility check:**
```
merchantCode + customerMobile + amount + securityKey
```

**Response:**
```json
{
  "statusCode": 200,
  "eligible": true,
  "downPayment": 1500.00,
  "tenureOptions": [
    { "months": 6, "monthlyInstallment": 2500.00, "adminFees": 200.00 },
    { "months": 12, "monthlyInstallment": 1350.00, "adminFees": 350.00 },
    { "months": 24, "monthlyInstallment": 750.00, "adminFees": 500.00 }
  ]
}
```

### Generate an Invoice

```
POST /ECommerceWeb/Fawry/invoices
```

```json
{
  "merchantCode": "YOUR_MERCHANT_CODE",
  "merchantRefNum": "inv_ref_123",
  "customerName": "Ahmed Hassan",
  "customerMobile": "01012345678",
  "customerEmail": "ahmed@email.com",
  "invoiceItems": [
    {
      "itemId": "SVC001",
      "description": "Web Development - Phase 1",
      "price": 8000.00,
      "quantity": 1
    },
    {
      "itemId": "SVC002",
      "description": "UI Design",
      "price": 3000.00,
      "quantity": 1
    }
  ],
  "currencyCode": "EGP",
  "paymentExpiry": 1738281600000,
  "signature": "computed_hmac_signature"
}
```

**Signature for invoice:**
```
merchantCode + merchantRefNum + totalAmount + securityKey
```

**Response includes an invoice URL** the customer can use to pay, and a reference number for cash payment at Fawry outlets.

### Get Invoice Status

```
GET /ECommerceWeb/Fawry/invoices/status?merchantCode={merchantCode}&merchantRefNumber={merchantRefNum}&signature={signature}
```

### Process a Refund

```
POST /ECommerceWeb/Fawry/payments/refund
```

```json
{
  "merchantCode": "YOUR_MERCHANT_CODE",
  "referenceNumber": "123456789",
  "refundAmount": 250.00,
  "reason": "Customer returned item",
  "signature": "computed_hmac_signature"
}
```

**Signature for refund (critical field order):**
```
merchantCode + referenceNumber + refundAmount (formatted to 2 decimal places) + reason + securityKey
```

Partial refunds are supported — set `refundAmount` to less than the original amount.

## Webhooks / Callbacks

Fawry sends server-to-server callbacks when payment status changes. Configure your callback URL in the Fawry dashboard.

### Webhook Payload Structure

```json
{
  "requestId": "xxxxx",
  "fawryRefNumber": "123456789",
  "merchantRefNumber": "order_ref_123",
  "orderAmount": 250.00,
  "paymentAmount": 255.00,
  "fawryFees": 5.00,
  "orderStatus": "PAID",
  "paymentMethod": "PAYATFAWRY",
  "messageSignature": "verification_hash",
  "paymentTime": 1737100000000
}
```

### Webhook Signature Verification

Always verify webhook signatures to ensure the callback is genuinely from Fawry:

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(
  fawryRefNumber,
  merchantRefNumber,
  paymentAmount,
  orderStatus,
  messageSignature,
  securityKey
) {
  // Fawry uses plain SHA-256; securityKey is appended to the data string (not used as HMAC key)
  const signatureData = `${fawryRefNumber}${merchantRefNumber}${paymentAmount.toFixed(2)}${orderStatus}${securityKey}`;
  const expectedSignature = crypto.createHash('sha256')
    .update(signatureData)
    .digest('hex');

  // Compare with provided signature
  return expectedSignature === messageSignature;
}

// Usage:
if (!verifyWebhookSignature(
  payload.fawryRefNumber,
  payload.merchantRefNumber,
  payload.paymentAmount,
  payload.orderStatus,
  payload.messageSignature,
  process.env.FAWRY_SECURITY_KEY
)) {
  throw new Error('Invalid webhook signature - potential security threat');
}
```

**CRITICAL:** Always verify webhook signatures. Do not process payments based on unverified webhooks.

## Common Integration Patterns

### E-commerce with cash option
1. Customer checks out → offer payment methods (Card, Fawry Cash, Wallet)
2. If PAYATFAWRY: `POST /payments/charge` → get reference number
3. Display: "Pay EGP 255 at any Fawry outlet using reference: 123456789"
4. Customer goes to nearest pharmacy/supermarket → pays cash
5. Webhook notifies you → fulfill order
6. This flow is critical in Egypt where many consumers don't have cards

### BNPL for high-value purchases
1. Check eligibility: `POST /payments/valu/eligibility`
2. Show tenure options to customer (6, 12, 24 months)
3. Customer selects plan
4. `POST /payments/charge` with `paymentMethod: "VALU"` and `valuTenureInMonths`
5. Customer completes ValU authentication
6. Deliver product — ValU handles installment collection

### Invoice-based billing
1. Generate invoice: `POST /invoices`
2. Send invoice URL to customer via email/SMS
3. Customer pays via card, wallet, or cash at Fawry
4. Webhook confirms payment → mark invoice as paid

## Error Handling

Fawry API returns structured error responses with status codes and descriptions. Always check both the HTTP status code and the `statusCode` field in the response.

### Common Error Response Format

```json
{
  "type": "ChargeResponse",
  "statusCode": 9901,
  "statusDescription": "Invalid merchant code"
}
```

### HTTP Status Codes

- **200**: Successful request
- **400**: Bad request (invalid parameters, malformed JSON, etc.)
- **401**: Unauthorized (invalid credentials)
- **500**: Server error
- **503**: Service unavailable

### Fawry-Specific Status Codes

| Code | Description | Action |
|------|-------------|--------|
| 200 | Operation done successfully | Process payment normally |
| 9901 | Invalid merchant code | Verify merchantCode in env vars |
| 9902 | Merchant not active | Contact Fawry support |
| 9903 | Merchant password not valid | Verify securityKey |
| 9904 | Merchant account not found | Verify merchant registration |
| 9905 | Duplicate merchant reference | Use unique merchantRefNum for each transaction |
| 9906 | Invalid request format | Check JSON structure and required fields |
| 9907 | Invalid amount | Verify amount is in EGP (not piasters), formatted to 2 decimals |
| 9908 | Currency code not supported | Use "EGP" for Egyptian transactions |
| 9931 | Invalid signature | **Verify field order in signature computation** (merchantCode + merchantRefNum + customerProfileId + paymentMethod + amount + securityKey) |
| 9932 | Payment amount mismatch | Ensure amount calculation is consistent |
| 9933 | Customer not found | Verify customerProfileId |
| 9946 | Expired payment | Payment window has expired; check paymentExpiry timestamp |
| 9950 | Payment method not available | Verify paymentMethod is one of: PAYATFAWRY, CARD, MWALLET, CASHONDELIVERY, VALU |
| 9952 | Customer not eligible for BNPL | Check eligibility before initiating VALU payment |
| 9953 | Invalid tenure option | Use one of: 6, 12, 18, 24, or 36 months for BNPL |
| 9960 | Reference number not found | Verify referenceNumber exists in the system |
| 9961 | Invalid refund amount | Ensure refund amount ≤ original payment amount |
| 9962 | Refund already processed | Cannot refund the same transaction twice |
| 9999 | General system error | Retry request; contact Fawry support if persists |

### Handling Different Error Scenarios

```javascript
async function handlePaymentError(response) {
  const { statusCode, statusDescription } = response;

  switch (statusCode) {
    case 9931: // Invalid signature
      console.error('Signature mismatch. Check field order and format.');
      throw new Error('Payment signature verification failed');

    case 9932: // Amount mismatch
      console.error('Amount mismatch between request and calculation.');
      throw new Error('Payment amount validation failed');

    case 9946: // Expired payment
      console.error('Payment expired. User must initiate a new payment.');
      throw new Error('Payment window expired. Please try again.');

    case 9952: // Not eligible for BNPL
      console.error('Customer not eligible for BNPL/ValU');
      throw new Error('You are not eligible for installment payments');

    case 9907: // Invalid amount
      console.error('Invalid payment amount format');
      throw new Error('Amount must be in EGP and formatted correctly');

    default:
      console.error(`Fawry Error ${statusCode}: ${statusDescription}`);
      throw new Error(`Payment failed: ${statusDescription}`);
  }
}
```

## Important Notes / Gotchas

### Critical Issues That Cause Integration Failures

#### 1. Signature Field Order (MOST COMMON ISSUE)
Fawry requires fields in a **specific order** for SHA-256 computation (plain SHA-256, not HMAC). The order changes by endpoint:
- **Charge**: `merchantCode + merchantRefNum + customerProfileId + paymentMethod + amount + securityKey`
- **Status Check**: `merchantCode + merchantRefNum + securityKey`
- **Refund**: `merchantCode + referenceNumber + refundAmount + reason + securityKey`
- **Webhook Verification**: `fawryRefNumber + merchantRefNumber + paymentAmount + orderStatus + securityKey`

Incorrect ordering results in status code **9931** (Invalid signature). Always verify field order when implementing.

#### 2. Amount Format and Currency
- Amounts must be in **Egyptian Pounds (EGP)**, not piasters
- EGP 250 = `250.00` (use 2 decimal places always)
- The format matters for signature computation: `amount.toFixed(2)`
- Sending amounts as integers (e.g., `25000` instead of `250.00`) causes status code **9907**

#### 3. Fawry Reference Number vs. Merchant Reference Number
- **Merchant Reference Number** (`merchantRefNum`): You create this; must be unique per transaction
- **Fawry Reference Number** (`referenceNumber`): Fawry generates this in the response; customer uses for PAYATFAWRY cash payment
- Do NOT confuse these in API calls (use correct one for status checks, refunds, etc.)

#### 4. PAYATFAWRY Cash Payments - 48-Hour Window
- PAYATFAWRY payments have a limited time window (typically 48 hours from creation)
- Set `paymentExpiry` appropriately using Unix timestamp in milliseconds
- If payment is not completed within the window, status becomes **EXPIRED**
- Always communicate the payment window to customers clearly

#### 5. 250K+ Retail Points Scalability
- Fawry has massive distribution (250,000+ collection points)
- For high-volume PAYATFAWRY transactions, ensure your system can:
  - Generate unique merchant reference numbers reliably
  - Handle webhook callbacks reliably (implement idempotency with webhooks)
  - Store and track Fawry reference numbers for customer service

#### 6. Mobile Wallet (MWALLET) Specific Behavior
- MWALLET payments may return status `PENDING` instead of `PAID`
- Customer must complete authentication/confirmation on their wallet app
- Always check status endpoint to confirm final payment status
- Do NOT fulfill orders on `PENDING` status

#### 7. BNPL/ValU Minimum and Maximum Amounts
- Minimum amount typically required for BNPL (usually around EGP 1,500-2,000)
- Always call eligibility endpoint before showing BNPL options to customer
- Check response for down payment requirement and tenure options

#### 8. Webhook Signature Verification is MANDATORY
- **ALWAYS verify webhook signatures** before processing payments
- Never trust webhook data without signature verification
- Store the securityKey securely and never log it
- Implement webhook signature verification in all callback handlers

#### 9. Customer Mobile Number Format
- Must be exactly 11 digits starting with `01` (Egyptian format)
- Examples: `01012345678`, `01112345678`, `01212345678`
- Sending invalid formats causes validation errors

#### 10. Unique Merchant Reference Numbers
- Each transaction must have a unique `merchantRefNum`
- Duplicate references result in status code **9905**
- Implement UUID or timestamp-based generation to avoid collisions
- Maintain uniqueness across staging and production if you use the same code

#### 11. Webhook Callback Failure Handling
- Implement retry logic for webhook processing
- Use idempotent operations (webhook may be sent multiple times)
- Log all webhook processing, especially signature verification failures
- Store webhook payload for audit trail

#### 12. Production vs. Staging
- Always test thoroughly in staging (`https://atfawry.fawrystaging.com`) first
- Use separate merchant codes for staging and production
- Do NOT send staging transactions to production or vice versa
- Each environment has different fee structures and processing rules

### Best Practices

1. **Signature Computation**: Write tests specifically for signature computation in each endpoint. Signature issues are the #1 cause of integration failures.

2. **Amount Handling**: Always use `toFixed(2)` for amount formatting in signatures and API requests to avoid floating-point precision issues.

3. **Error Recovery**: Implement comprehensive logging of all Fawry responses, especially errors. Log the exact request and response for debugging.

4. **Webhook Idempotency**: Design your payment confirmation logic to be idempotent — same webhook can be processed multiple times safely.

5. **Customer Communication**: Always display the Fawry reference number prominently for PAYATFAWRY payments, with clear instructions on where to pay.

6. **Timeout Handling**: For MWALLET and BNPL, don't immediately mark as failed if status is PENDING; implement polling or wait for webhook confirmation.

7. **Refund Tracking**: Keep detailed records of refund requests and their status. Some refunds may be asynchronous.

8. **Currency Conversion**: If accepting other currencies, convert to EGP on the merchant backend, never in the API request.

## Egyptian Market Notes

- Currency: **EGP** (Egyptian Pound). No piasters in API amounts — use decimal (250.50).
- Phone format: `01XXXXXXXXX` (11 digits, starting with 01)
- Fawry has 250,000+ retail collection points — pharmacies, supermarkets, kiosks
- ValU BNPL is popular for electronics and high-value purchases
- Mobile wallets: Vodafone Cash, Orange Money, Etisalat Cash, WE Pay, CIB Smart Wallet
- Cash payments are preferred by ~60% of Egyptian online shoppers
- Fawry fees typically range from 1-3% depending on payment method

## Testing Checklist

- [ ] Test signature computation for each endpoint separately
- [ ] Verify webhook callback signature verification
- [ ] Test PAYATFAWRY flow end-to-end with cash payment
- [ ] Test CARD payment flow
- [ ] Test MWALLET payment and polling for status
- [ ] Test BNPL eligibility check and payment
- [ ] Test refund with partial and full amounts
- [ ] Test error scenarios (expired payment, invalid amount, etc.)
- [ ] Test duplicate merchant reference number handling
- [ ] Verify unique reference number generation strategy
- [ ] Test webhook idempotency (process same webhook twice)
- [ ] Test timeout and retry logic

## Useful Links

- Developer Portal: https://developer.fawrystaging.com
- API Docs: https://developer.fawrystaging.com/docs
- Error Codes Reference: https://developer.fawrystaging.com/docs/error-codes/error-codes
- Dashboard: https://dashboard.atfawry.com
- Community Library (Node.js): https://github.com/fawry-api/fawry-node
- Fawry Homepage: https://www.atfawry.com
