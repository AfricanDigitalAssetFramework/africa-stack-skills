---
name: payfast
description: "Integrate with PayFast for payment gateway processing in South Africa. Use this skill whenever the user wants to accept payments via PayFast, process subscriptions, manage refunds, handle merchant payments, verify transactions, or work with PayFast's API in any way. Also trigger when the user mentions 'PayFast', 'South African payments', 'PayFast subscriptions', 'merchant payments', or needs payment processing with subscription support."
---

# PayFast Integration Skill

PayFast (now part of Network International) is South Africa's longest-running payment processor. It handles online payments, subscriptions, and merchant services with both form-based redirect flows and REST API endpoints. PayFast supports card payments, bank transfers, and recurring billing for millions of South African customers.

## When to use this skill

You're building a payment system that needs to accept payments in South Africa with strong subscription support — an e-commerce checkout, a SaaS platform with recurring billing, a marketplace, or any solution requiring ZAR payment processing. PayFast is ideal for businesses targeting the South African market with excellent local market penetration and support for card, bank transfer, and alternative payment methods.

## Authentication

PayFast uses merchant credentials and signature-based authentication for security:

**Authentication Components:**
- **Merchant ID:** Your PayFast merchant ID (found in your dashboard)
- **Merchant Key:** Your PayFast merchant key (found in your dashboard)
- **Passphrase:** Optional PayFast passphrase (set in dashboard security settings)
- **API Signature:** MD5 or SHA256 HMAC signature for request verification

> ⚠️ **Use SHA256 for new integrations, not MD5.** PayFast supports both MD5 (legacy) and SHA256 (recommended). MD5 is documented first because it's most common in older integrations, but SHA256 is significantly more secure. For new integrations, use SHA256 throughout. PayFast's ITN (Instant Transaction Notification) verification should always use SHA256.

> ℹ️ **Rebrand note:** PayFast was acquired by Network International in 2023. The product continues to operate as "PayFast by Network International." API endpoints and credentials are unchanged, but support channels, contracts, and billing may route through Network International for new merchants.

Store credentials in environment variables: `PAYFAST_MERCHANT_ID`, `PAYFAST_MERCHANT_KEY`, and `PAYFAST_PASSPHRASE`. Never hardcode credentials.

**Environments:**
- **Sandbox:** `https://sandbox.payfast.co.za` (for testing)
- **Production:** `https://www.payfast.co.za` and `https://api.payfast.co.za` (for live transactions)

## Core API Reference

### 1. Create a Payment (Form Redirect)

Initiate a payment by redirecting the customer to PayFast's hosted payment form. This is the most common integration method.

**Endpoint:**
```
POST https://www.payfast.co.za/eng/process
```

(Sandbox: `POST https://sandbox.payfast.co.za/eng/process`)

**Content-Type:** `application/x-www-form-urlencoded`

**Form Parameters:**
```
merchant_id=10001234
merchant_key=abc123xyz
return_url=https://yoursite.com/payment/success
cancel_url=https://yoursite.com/payment/cancel
notify_url=https://yoursite.com/payment/notify
email_address=customer@example.com
item_name=Premium Subscription
item_description=1 Month Premium Access
amount=199.99
custom_str1=ORDER-12345
custom_str2=user_metadata_here
signature=MD5_HASH_HERE
```

**Important Notes:**
- **Amount** is in ZAR decimal format (major units). R199.99 = "199.99", NOT "19999" cents
- **Signature** computation (MD5): Concatenate fields in order, then apply MD5:
  ```
  MD5(merchant_id&merchant_key&return_url&cancel_url&notify_url&email_address&item_name&item_description&amount&custom_str1&custom_str2&PASSPHRASE)
  ```
  Note: Only include `custom_str1` and `custom_str2` if you're using them. If passphrase is empty, still include the `&` separator.

**Example HTML Form:**
```html
<form method="POST" action="https://www.payfast.co.za/eng/process">
  <input type="hidden" name="merchant_id" value="10001234">
  <input type="hidden" name="merchant_key" value="abc123xyz">
  <input type="hidden" name="return_url" value="https://yoursite.com/payment/success">
  <input type="hidden" name="cancel_url" value="https://yoursite.com/payment/cancel">
  <input type="hidden" name="notify_url" value="https://yoursite.com/payment/notify">
  <input type="hidden" name="email_address" value="customer@example.com">
  <input type="hidden" name="item_name" value="Premium Subscription">
  <input type="hidden" name="item_description" value="1 Month Premium Access">
  <input type="hidden" name="amount" value="199.99">
  <input type="hidden" name="custom_str1" value="ORDER-12345">
  <input type="hidden" name="signature" value="abc123def456xyz789">
  <button type="submit">Pay with PayFast</button>
</form>
```

**Response:** Customer is redirected to PayFast's payment page. After payment, they're redirected to your `return_url` with query parameters including payment status.

### 2. Subscription Payments (Recurring Billing)

Create recurring payment subscriptions via form redirect.

**Endpoint:** Same as regular payments
```
POST https://www.payfast.co.za/eng/process
```

**Additional Parameters for Subscriptions:**
```
subscription=1
frequency=3
billing_date=2025-03-23
recurring_amount=99.99
initial_payment=0
cycle_period=3
```

**Frequency Values (`frequency` field):**
- `3` = Monthly
- `4` = Quarterly
- `5` = Semi-annually (bi-annually)
- `6` = Annually

Note: Values 1 and 2 represent per-hour and daily billing (uncommon). `12` is not a valid frequency code.

**Example Subscription Form:**
```html
<form method="POST" action="https://www.payfast.co.za/eng/process">
  <!-- Standard payment fields -->
  <input type="hidden" name="merchant_id" value="10001234">
  <input type="hidden" name="merchant_key" value="abc123xyz">
  <input type="hidden" name="email_address" value="customer@example.com">
  <input type="hidden" name="item_name" value="Monthly Premium">
  <input type="hidden" name="amount" value="0">

  <!-- Subscription fields -->
  <input type="hidden" name="subscription" value="1">
  <input type="hidden" name="recurring_amount" value="99.99">
  <input type="hidden" name="frequency" value="1">
  <input type="hidden" name="billing_date" value="2025-03-23">
  <input type="hidden" name="cycle_period" value="1">

  <input type="hidden" name="signature" value="MD5_HASH">
  <button type="submit">Subscribe Now</button>
</form>
```

**Response:** PayFast returns a subscription token in the webhook. Store this token to manage the subscription later.

### 3. Get Subscription Status

Fetch the current status of a subscription using its token.

**Endpoint:**
```
GET https://api.payfast.co.za/subscriptions/{token}/fetch
```

**Headers:**
```
merchant-id: YOUR_MERCHANT_ID
version: v1
timestamp: 2025-02-24T10:30:00
signature: COMPUTED_SIGNATURE
```

**Signature Generation (subscription endpoints):**
Create a sorted query string from the request parameters, then compute:
```
SHA256(merchant_id + version + timestamp + signature + PASSPHRASE)
```

**Response (success):**
```json
{
  "id": "sub_123456789",
  "m_subscription": "SUB-12345",
  "merchant_id": "10001234",
  "frequency": "1",
  "billing_date": "2025-03-23",
  "recurring_amount": "99.99",
  "status": "Active",
  "status_reason": "",
  "cycles": "0",
  "cycles_complete": "2",
  "amount": "99.99",
  "next_billing_date": "2025-03-23"
}
```

**Status Values:** `Active`, `Paused`, `Cancelled`, `Completed`, `Failed`

### 4. Pause a Subscription

Temporarily pause a subscription without cancelling it.

**Endpoint:**
```
PUT https://api.payfast.co.za/subscriptions/{token}/pause
```

**Headers:**
```
merchant-id: YOUR_MERCHANT_ID
version: v1
timestamp: 2025-02-24T10:30:00
signature: COMPUTED_SIGNATURE
```

**Body (form-encoded or JSON):**
```json
{
  "token": "subscription_token",
  "cycles": 1
}
```

**Response (success):**
```json
{
  "success": true,
  "data": {
    "status": "Paused",
    "message": "Subscription paused successfully",
    "token": "subscription_token"
  }
}
```

**Note:** The `cycles` parameter determines how many billing cycles to pause (default: 1). The subscription end date is automatically extended by PayFast.

### 5. Unpause a Subscription

Resume a previously paused subscription.

**Endpoint:**
```
PUT https://api.payfast.co.za/subscriptions/{token}/unpause
```

**Headers:**
```
merchant-id: YOUR_MERCHANT_ID
version: v1
timestamp: 2025-02-24T10:30:00
signature: COMPUTED_SIGNATURE
```

**Body:**
```json
{
  "token": "subscription_token"
}
```

**Response (success):**
```json
{
  "success": true,
  "data": {
    "status": "Active",
    "message": "Subscription resumed successfully",
    "token": "subscription_token"
  }
}
```

**Note:** Unpausing early does not adjust the next billing date — billing resumes after the full pause duration.

### 6. Process a Refund

Issue a refund for a previously successful payment.

**Endpoint:**
```
POST https://api.payfast.co.za/refunds
```

**Headers:**
```
merchant-id: YOUR_MERCHANT_ID
version: v1
timestamp: 2025-02-24T10:30:00
signature: COMPUTED_SIGNATURE
Content-Type: application/json
```

**Body:**
```json
{
  "m_payment_id": "12345",
  "pf_payment_id": "1234567890",
  "amount": "199.99"
}
```

For full refunds, omit the `amount` field. For partial refunds, specify the amount less than the original transaction.

**Response (success):**
```json
{
  "success": true,
  "data": {
    "refund_id": "refund_xyz789",
    "m_payment_id": "12345",
    "pf_payment_id": "1234567890",
    "amount": "199.99",
    "status": "Pending",
    "created_at": "2025-02-24T10:35:00Z"
  }
}
```

**Refund Status:** `Pending`, `Processing`, `Completed`, `Failed`

Refunds typically appear in the customer's bank account within 5-10 business days.

## Webhooks (Instant Transaction Notification - ITN)

PayFast sends webhooks (ITN messages) to your `notify_url` when payment status changes. Always verify webhook authenticity before processing.

**Webhook Payload (form-encoded POST):**
```
m_payment_id=12345
pf_payment_id=1234567890
payment_status=COMPLETE
item_name=Premium Subscription
item_description=1 Month Premium Access
amount_gross=199.99
amount_fee=-5.00
amount_net=194.99
custom_str1=ORDER-12345
custom_str2=user_metadata
email_address=customer@example.com
merchant_id=10001234
signature=MD5_HASH_HERE
```

**Payment Status Values:**
- `COMPLETE` — Payment successful
- `FAILED` — Payment declined or failed
- `PENDING` — Payment processing (rare in webhooks)
- `CANCELLED` — Customer cancelled payment

### Webhook Verification Flow

**Step 1: Verify the Signature Locally**

Compute the expected signature from the webhook payload:

```javascript
const crypto = require('crypto');

function verifyPayFastSignature(data, passphrase) {
  // Extract and sort the fields in the exact order PayFast uses
  const fields = [
    data.m_payment_id,
    data.pf_payment_id,
    data.payment_status,
    data.item_name,
    data.item_description,
    data.amount_gross,
    data.amount_fee,
    data.amount_net,
    data.custom_str1 || '',
    data.custom_str2 || '',
    data.email_address,
    data.merchant_id,
    passphrase
  ];

  const payloadString = fields.join('');
  const expectedSignature = crypto
    .createHash('md5')
    .update(payloadString)
    .digest('hex');

  return expectedSignature === data.signature;
}

// In your webhook handler:
app.post('/payment/notify', (req, res) => {
  const { body } = req;
  const passphrase = process.env.PAYFAST_PASSPHRASE;

  if (!verifyPayFastSignature(body, passphrase)) {
    console.error('Invalid signature');
    return res.status(401).send('Unauthorized');
  }

  // Signature is valid, proceed
  if (body.payment_status === 'COMPLETE') {
    // Update order status, unlock access, etc.
  }

  res.status(200).send('OK');
});
```

**Step 2: Validate with PayFast Server (Optional but Recommended)**

Post the webhook data back to PayFast to confirm authenticity:

```
POST https://www.payfast.co.za/eng/query/validate
```

(Sandbox: `POST https://sandbox.payfast.co.za/eng/query/validate`)

**Body (form-encoded):**
```
cmd=_notify-validate
merchant_id=10001234
merchant_key=abc123xyz
m_payment_id=12345
pf_payment_id=1234567890
```

**Response:**
- `VALID` — Webhook is authentic, process it
- `INVALID` — Webhook is forged, discard it

**Example Validation Code:**
```javascript
async function validateWebhookWithPayFast(data) {
  const params = new URLSearchParams();
  params.append('cmd', '_notify-validate');
  params.append('merchant_id', process.env.PAYFAST_MERCHANT_ID);
  params.append('merchant_key', process.env.PAYFAST_MERCHANT_KEY);
  params.append('m_payment_id', data.m_payment_id);
  params.append('pf_payment_id', data.pf_payment_id);

  const response = await fetch(
    'https://www.payfast.co.za/eng/query/validate',
    { method: 'POST', body: params }
  );

  const text = await response.text();
  return text.trim() === 'VALID';
}
```

## Common Integration Patterns

### Pattern 1: E-commerce One-Time Payment

```
1. Customer completes checkout on your site
2. Display PayFast payment form (POST to https://www.payfast.co.za/eng/process)
3. Customer redirected to PayFast, enters payment details
4. PayFast processes payment and redirects to your return_url
5. Webhook sent to notify_url with payment status
6. Verify webhook signature (locally and/or with PayFast server)
7. If payment_status === 'COMPLETE', fulfill order
8. Update order status in database and send confirmation email
```

### Pattern 2: SaaS Recurring Subscription

```
1. Customer selects monthly/yearly subscription plan
2. Display PayFast subscription form with subscription=1 parameter
3. Customer redirected to PayFast, enters payment details
4. PayFast processes first payment and creates subscription
5. Webhook includes subscription token (store in your database)
6. PayFast automatically charges recurring_amount on each billing cycle
7. Webhook sent each billing cycle confirming charge
8. Monitor subscription status via GET /subscriptions/{token}/fetch
9. Use PUT /subscriptions/{token}/pause to temporarily pause
10. Use PUT /subscriptions/{token}/unpause to resume after pause
```

### Pattern 3: Refund Processing

```
1. Customer requests refund (within your return policy)
2. POST to https://api.payfast.co.za/refunds with payment details
3. PayFast responds with refund_id and status "Pending"
4. Store refund_id in database for tracking
5. Periodically check refund status (webhook updates)
6. Once status becomes "Completed", notify customer refund arrived
7. Refund appears in customer's bank account within 5-10 business days
```

### Pattern 4: Manual Subscription Management

```
1. User signs up for subscription (token stored in database)
2. Implement admin panel showing subscription status
3. Admin can manually pause subscription (PUT .../pause)
4. Paused subscription doesn't charge, but can be resumed later
5. Admin can unpause subscription (PUT .../unpause)
6. Show subscription details via GET .../fetch
7. Allow users to cancel from settings (via API or manual support)
```

## Error Handling

**Common HTTP Status Codes:**
- `200 OK` — Request successful
- `400 Bad Request` — Invalid parameters (check field names, types, signature)
- `401 Unauthorized` — Invalid merchant ID, key, or signature
- `403 Forbidden` — Insufficient permissions for this operation
- `404 Not Found` — Payment or subscription not found
- `500 Server Error` — PayFast server error (retry with exponential backoff)

**Signature/Authentication Errors:**
If you receive a 401 "Merchant Authorization Failed":
1. Verify merchant_id and merchant_key are correct
2. Check field order in signature computation (order matters!)
3. Ensure amount format is decimal (199.99 not 19999)
4. Verify passphrase is identical on PayFast dashboard and in code
5. Check that whitespace is not included in the signature string

**Common Issues:**
- **Signature mismatch**: Most common cause is wrong field order or including/excluding passphrase inconsistently
- **Amount format**: Use decimal (199.99) not cents (19999)
- **Empty passphrase**: Still include the `&` separator in signature computation
- **Webhook validation fails**: Confirm you're using exact field order and not adding/removing custom fields

## Important Notes and Gotchas

1. **Always Verify Webhooks:** Never trust webhooks without signature verification. Always verify signature locally AND/OR POST back to PayFast's validate endpoint.

2. **Amount Format is Decimal:** Use ZAR decimal format (e.g., "199.99"), NOT cents. This is different from many other payment gateways.

3. **Field Order in Signatures:** MD5 signature computation is extremely sensitive to field order. Check the official documentation for the exact order required.

4. **Passphrase Handling:** If your PayFast account has no passphrase set, still include the `&` separator in the signature string, but leave the passphrase value empty.

5. **Subscription Tokens:** Always store subscription tokens in your database. These are required for pause/unpause/fetch operations.

6. **Webhook Retry Logic:** PayFast may retry webhooks multiple times if you don't respond with HTTP 200. Respond quickly and process asynchronously if needed.

7. **Status Polling:** Don't rely solely on customer redirects to determine payment status. Always verify via webhooks or API before fulfilling orders.

8. **Sandbox Testing:** Use `https://sandbox.payfast.co.za` for testing. Test with real-looking merchant credentials (your sandbox account credentials).

9. **PCI Compliance:** PayFast is PCI DSS Level 1 compliant. Never send card data through your own server — always use form redirects or hosted fields.

10. **Custom Fields:** The `custom_str1` and `custom_str2` fields are returned in webhooks. Use these to correlate payments with your orders (e.g., order ID).

11. **API Endpoint Changes:** The API base URL is `https://api.payfast.co.za` for all API operations (subscriptions, refunds, etc.). The form redirect endpoint is `https://www.payfast.co.za/eng/process`.

12. **Subscription Pause Duration:** Pausing a subscription extends its end date automatically. Unpausing early doesn't adjust the next billing date — billing still resumes after the full pause duration.

13. **Refund Timeline:** Refunds are processed within 5-10 business days. Monitor refund status but don't expect instant refunds.

## Useful Links

- **Official Developer Portal:** https://developers.payfast.co.za/
- **API Documentation:** https://developers.payfast.co.za/api
- **PayFast Dashboard:** https://www.payfast.co.za/
- **Sandbox Environment:** https://sandbox.payfast.co.za/
- **Support & FAQs:** https://payfast.io/faq/merchant-faqs/
- **PayFast by Network:** https://payfast.io/
- **PHP SDK:** https://github.com/PayFast/payfast-php-sdk
- **Status Page:** https://status.payfast.io/
