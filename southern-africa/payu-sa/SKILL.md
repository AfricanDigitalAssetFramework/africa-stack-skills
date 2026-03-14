---
name: PayU South Africa
description: REST-based payment gateway supporting cards, instant EFT, eBucks, Discovery Miles, and recurring payments in South Africa. Part of the Prosus group.
triggers:
  - PayU
  - PayU South Africa
  - PayU SA
  - South Africa payment gateway
  - ZAR card payments
  - eBucks payments
  - Instant EFT payments
  - PayU subscription
---

# PayU South Africa

PayU South Africa is a full-featured payment gateway operated by Prosus, providing local payment methods and global card processing for e-commerce merchants in South Africa. The platform offers both hosted (RPP) and server-to-server (Enterprise API) integration options with support for multiple payment methods including credit/debit cards, instant EFT, eBucks, Discovery Miles, and recurring/subscription payments.

## When to use this skill

- Building payment processing for South African e-commerce stores requiring ZAR transactions
- Integrating multiple payment methods (cards, EFT, loyalty programs) in a single checkout
- Implementing recurring/subscription billing with customer consent
- Processing payments with 3D Secure authentication requirements
- Handling instant EFT payments from South African banks (Nedbank, Standard Bank, FNB, ABSA)
- Requiring PCI DSS compliance without handling card details directly (use RPP)
- Building a custom checkout experience with full payment control (Enterprise API)

## Authentication

PayU South Africa uses a **hash-based authentication** model with the following credentials:

### Credentials Required
- **Merchant Key**: API key for identifying your merchant account
- **Merchant Salt**: Secret string used to generate secure hashes
- **SOAP Username** (for legacy integrations): Available in your merchant dashboard

### Hash Generation (SHA512)
For Redirect Payment Page (RPP) integration:
```
hash = SHA512(merchant_key + security_key + amount + currency + order_id + customer_email)
```

For server-to-server API calls, include:
- Merchant Key in request headers or as first parameter
- SHA512 hashes for request verification

### Obtaining Credentials
1. Log into your [PayU South Africa Merchant Dashboard](https://merchant.payumea.com/)
2. Navigate to **Settings** → **API Keys** or **Developer Settings**
3. Generate/copy your Merchant Key and Merchant Salt
4. For sandbox testing, use test credentials provided in your dashboard

## Core API Reference

### Base URLs
- **Production**: `https://secure.payu.co.za/` (varies by integration method)
- **Sandbox/Test**: `https://sandbox.payumea.com/` (check PayU documentation for test endpoints)

### Authentication Header
```
Authorization: Bearer [access_token]
```
Or include merchant key as first parameter in request body.

### Payment Methods Supported
- Credit/Debit Cards (Visa, Mastercard)
- Instant EFT (Smart EFT, EFT Pro)
- eBucks
- Discovery Miles
- PayU Subscription (recurring)
- PayPal (where available)

### Key Endpoints

#### 1. Create Authorization
Authorize a payment without capturing funds immediately.

**Method**: POST
**Endpoint**: `/payments/authorize` (Redirect) or REST API endpoint

**Request Parameters**:
```json
{
  "merchant_key": "your_merchant_key",
  "amount": 100.00,
  "currency": "ZAR",
  "order_id": "ORDER_12345",
  "customer_email": "customer@example.com",
  "customer_name": "John Doe",
  "payment_method": "CREDITCARD",
  "payment_method_details": {
    "card_number": "4111111111111111",
    "expiry_month": "12",
    "expiry_year": "2025",
    "cvv": "123"
  },
  "return_url": "https://yourdomain.com/payment-return",
  "notify_url": "https://yourdomain.com/payment-ipn",
  "hash": "SHA512_COMPUTED_HASH"
}
```

#### 2. Create Charge (Authorization + Capture)
Authorize and immediately capture funds.

**Method**: POST
**Endpoint**: `/payments/charge`

**Request Parameters**:
```json
{
  "merchant_key": "your_merchant_key",
  "amount": 100.00,
  "currency": "ZAR",
  "order_id": "ORDER_12345",
  "customer_email": "customer@example.com",
  "payment_method": "CREDITCARD",
  "provider_specific_data": {
    "magellan": {
      "additional_details": {
        "is3ds": "true",
        "merchant_site_url": "https://yourdomain.com"
      }
    }
  },
  "hash": "SHA512_COMPUTED_HASH"
}
```

#### 3. Capture Authorized Payment
Capture funds from a previously authorized transaction.

**Method**: POST
**Endpoint**: `/payments/capture`

**Request Parameters**:
```json
{
  "original_transaction_id": "AUTH_TRANS_ID",
  "merchant_key": "your_merchant_key",
  "amount": 100.00,
  "currency": "ZAR"
}
```

#### 4. Refund Transaction
Refund a captured payment (full or partial).

**Method**: POST
**Endpoint**: `/payments/refund`

**Request Parameters**:
```json
{
  "original_transaction_id": "CHARGE_TRANS_ID",
  "merchant_key": "your_merchant_key",
  "amount": 50.00,
  "currency": "ZAR",
  "reason": "Customer requested refund"
}
```

#### 5. Instant EFT Payment
Redirect customer to instant EFT provider selection.

**Flow**:
1. Customer selects Instant EFT on your checkout
2. Redirect to PayU with payment request
3. Customer selects their bank (Nedbank, Standard Bank, FNB, ABSA)
4. Customer logs into online banking and approves transfer
5. Receive instant IPN notification on success

**Request Parameters**:
```json
{
  "merchant_key": "your_merchant_key",
  "amount": 100.00,
  "currency": "ZAR",
  "order_id": "ORDER_12345",
  "customer_email": "customer@example.com",
  "payment_method": "EFT",
  "return_url": "https://yourdomain.com/payment-return",
  "notify_url": "https://yourdomain.com/payment-ipn"
}
```

#### 6. Recurring/Subscription Payment
Set up recurring charges with customer consent.

**Method**: POST
**Endpoint**: `/subscription/create`

**Request Parameters**:
```json
{
  "merchant_key": "your_merchant_key",
  "customer_email": "customer@example.com",
  "customer_name": "John Doe",
  "initial_amount": 100.00,
  "recurring_amount": 100.00,
  "frequency": "MONTHLY",
  "start_date": "2026-03-01",
  "end_date": "2026-12-31",
  "currency": "ZAR",
  "payment_method": "CREDITCARD",
  "order_id": "SUB_12345",
  "hash": "SHA512_COMPUTED_HASH"
}
```

## Webhooks / IPN (Instant Payment Notification)

### IPN Configuration
Configure your notification URL in the PayU Merchant Dashboard:
1. Log into [Merchant Dashboard](https://merchant.payumea.com/)
2. Go to **Settings** → **Notification URLs** or **IPN Settings**
3. Enter your callback URL (must be publicly accessible HTTPS endpoint)
4. PayU will POST transaction results to this URL

### IPN Request Format
PayU sends POST requests to your notification URL with transaction details:

```json
{
  "m_payment_id": "ORDER_12345",
  "pf_payment_id": "1234567890",
  "payment_status": "COMPLETE",
  "item_name": "Product Name",
  "item_description": "Order description",
  "amount_gross": "100.00",
  "amount_fee": "2.50",
  "amount_net": "97.50",
  "custom_str1": "reference_code",
  "custom_str2": "customer_id",
  "custom_str3": "order_details",
  "custom_str4": "metadata",
  "custom_str5": "tracking_info",
  "status_code": "00001",
  "reason_code": "1",
  "reason_text": "Successful transaction",
  "transaction_id": "1234567890",
  "proof_id": "proof_code_12345",
  "on01": "bank_name",
  "on02": "account_number",
  "pf_signature": "SHA512_SIGNATURE_HASH"
}
```

### IPN Response Requirements
Your endpoint **MUST** respond with:
- **HTTP 200 OK** status code within 20 seconds
- Plain text response: `OK`
- Parse and verify the `pf_signature` hash before processing

### IPN Signature Verification
Verify the signature to ensure PayU sent the notification:

```
expected_signature = SHA512(
  passphrase +
  merchant_id +
  m_payment_id +
  amount_gross +
  item_name +
  item_description +
  custom_str1 +
  status_code +
  reason_code +
  transaction_id +
  pf_payment_id
)
```

Compare `expected_signature` with received `pf_signature`.

### Payment Status Values
- `COMPLETE`: Payment successful and funds captured
- `FAILED`: Payment failed
- `PENDING`: Payment pending (e.g., awaiting fraud review or 3DS completion)
- `CANCELLED`: Payment cancelled by customer
- `TIMEOUT`: Customer session expired

### Retry Policy
If PayU cannot reach your endpoint:
- **Attempt 1**: Immediately
- **Attempt 2**: 5 minutes later
- **Attempt 3**: 1 hour later
- After 3 failures: Email notification sent to merchant

## Common Integration Patterns

### Pattern 1: Hosted Payment Page (RPP) - Lowest PCI Burden
Best for quick integration with minimal PCI compliance requirements.

```html
<form action="https://secure.payu.co.za/rpp.php" method="POST">
  <input type="hidden" name="merchant_id" value="10000001">
  <input type="hidden" name="merchant_key" value="your_merchant_key">
  <input type="hidden" name="return_url" value="https://yourdomain.com/return">
  <input type="hidden" name="notify_url" value="https://yourdomain.com/ipn">
  <input type="hidden" name="amount" value="100.00">
  <input type="hidden" name="item_name" value="Product Name">
  <input type="hidden" name="item_description" value="Description">
  <input type="hidden" name="custom_str1" value="ORDER_12345">
  <input type="hidden" name="email_address" value="customer@example.com">
  <input type="hidden" name="signature" value="SHA512_COMPUTED_HASH">
  <button type="submit">Pay with PayU</button>
</form>
```

### Pattern 2: Server-to-Server API - Full Custom Control
Full checkout control with higher PCI requirements or use tokenization.

```javascript
// Node.js example
const crypto = require('crypto');
const https = require('https');

function createCharge(orderData) {
  const payload = {
    merchant_key: process.env.PAYU_MERCHANT_KEY,
    amount: orderData.amount,
    currency: 'ZAR',
    order_id: orderData.orderId,
    customer_email: orderData.email,
    payment_method: 'CREDITCARD',
    payment_method_details: {
      card_number: orderData.cardNumber,
      expiry_month: orderData.expiryMonth,
      expiry_year: orderData.expiryYear,
      cvv: orderData.cvv
    },
    return_url: `https://yourdomain.com/payment/return/${orderData.orderId}`,
    notify_url: 'https://yourdomain.com/payment/ipn'
  };

  const hash = crypto
    .createHash('sha512')
    .update(JSON.stringify(payload) + process.env.PAYU_MERCHANT_SALT)
    .digest('hex');

  payload.hash = hash;

  // POST to PayU API endpoint
  return postToPayU('/payments/charge', payload);
}
```

### Pattern 3: 3D Secure Authentication
Required for high-value transactions and fraud prevention.

```json
{
  "merchant_key": "your_merchant_key",
  "amount": 5000.00,
  "currency": "ZAR",
  "order_id": "HIGH_VALUE_ORDER_123",
  "customer_email": "customer@example.com",
  "payment_method": "CREDITCARD",
  "provider_specific_data": {
    "magellan": {
      "additional_details": {
        "is3ds": "true",
        "merchant_site_url": "https://yourdomain.com/checkout"
      }
    }
  },
  "hash": "SHA512_COMPUTED_HASH"
}
```

Customer will be redirected to 3DS authentication flow. Transaction remains PENDING for up to 40 minutes awaiting completion. After timeout, status transitions to FAILED.

### Pattern 4: Instant EFT Integration
Leveraging South African bank connections for real-time transfers.

```javascript
// Redirect customer to EFT selection
const efiParams = {
  merchant_key: process.env.PAYU_MERCHANT_KEY,
  amount: orderData.amount,
  currency: 'ZAR',
  order_id: orderData.orderId,
  customer_email: orderData.email,
  payment_method: 'EFT',
  return_url: 'https://yourdomain.com/payment-return',
  notify_url: 'https://yourdomain.com/payment-ipn'
};

// Compute hash and redirect to PayU
const redirectUrl = buildPayURedirectURL(efiParams);
res.redirect(redirectUrl);

// Customer selects bank → logs in → approves → instant notification sent to your server
```

### Pattern 5: Recurring Subscriptions with Consent
Handling monthly/annual recurring charges.

```javascript
async function setupRecurringPayment(customerData) {
  const subscriptionPayload = {
    merchant_key: process.env.PAYU_MERCHANT_KEY,
    customer_email: customerData.email,
    customer_name: customerData.name,
    initial_amount: 99.99,        // First charge
    recurring_amount: 99.99,       // Monthly charge
    frequency: 'MONTHLY',
    start_date: new Date().toISOString().split('T')[0],
    end_date: '2027-12-31',
    currency: 'ZAR',
    payment_method: 'CREDITCARD',
    order_id: `SUB_${customerData.customerId}`,
    description: 'Monthly subscription',
    hash: computeHash(subscriptionPayload)
  };

  // Redirect to PayU subscription form
  // Customer authorizes and PayU handles recurring charges
}
```

## Error Handling

### Common Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| 00001 | Successful | No action needed |
| 00002 | Payment Failed | Retry with customer confirmation |
| 00003 | Payment Pending | Wait for final status update via IPN |
| 00004 | Payment Cancelled | Customer cancelled; prompt retry |
| 00005 | Invalid Merchant | Check credentials in PayU dashboard |
| 00006 | Invalid Hash | Verify SHA512 hash computation |
| 00007 | Invalid Amount | Amount format/value incorrect (ZAR cents) |
| 00008 | Invalid Email | Invalid customer email format |
| 00009 | Duplicate Transaction | Order ID already processed; use new ID |
| 00010 | 3DS Required | Customer card requires 3D Secure |
| 00011 | 3DS Failed | Customer failed 3D Secure authentication |
| 00012 | Bank Error | Bank declined transaction; see bank_error_code |
| 00013 | Fraud Alert | Transaction flagged; manual review pending |

### 3D Secure Specific Errors
- **is3ds not provided**: 3DS configuration missing when required by card issuer
- **merchant_site_url missing**: Cannot complete 3DS authentication without return URL
- **3DS timeout**: Customer didn't complete within 40 minutes; transaction becomes FAILED

### Response Hash Verification Error
- **Signature mismatch**: Indicates potential tampering or wrong salt
- **Verification failed**: Recalculate hash with exact parameters in correct order

### EFT-Specific Errors
- **Invalid bank**: Customer selected unsupported bank
- **Session expired**: Customer took too long to complete EFT; retry payment
- **Insufficient funds**: Bank declined due to account balance; customer must retry

### Handling 3DS Pending Transactions
```javascript
// After receiving PENDING status
setTimeout(async () => {
  const status = await checkTransactionStatus(transactionId);

  if (status === 'COMPLETE') {
    // Process order
    fulfillOrder();
  } else if (status === 'FAILED') {
    // Cancel order, notify customer
    cancelOrder('3DS authentication failed');
  }
}, 40 * 60 * 1000); // Check after 40 minutes max
```

## Important Notes / Gotchas

1. **Hash Computation Precision**: SHA512 hashes are extremely sensitive to input order and encoding. Test hash generation extensively in sandbox before production. Include exact field order and UTF-8 encoding.

2. **Amount Format in ZAR**: Always use amount in South African Rand with two decimal places (cents). Amount `100.00` = 100 ZAR. Never include currency symbols or thousand separators in the actual field value.

3. **3D Secure Timeout Behavior**: Transactions requiring 3DS will remain PENDING for up to 40 minutes while awaiting customer authentication. After 40 minutes without customer action, status automatically transitions to FAILED. Don't treat PENDING as final; implement polling or use webhooks for status updates.

4. **IPN Not Guaranteed Delivery**: Although PayU retries 3 times, webhooks can be lost. Implement transaction status reconciliation polls (e.g., daily) against PayU's transaction API to catch missed notifications.

5. **Merchant Site URL Required for 3DS**: Omitting `merchant_site_url` in provider_specific_data will cause 3DS flows to fail silently. This is critical for high-value card transactions where 3DS is mandatory.

6. **Recurring Payment Consent**: South African regulations require explicit customer consent before setting up recurring payments. Document consent and store proof. PayU may request evidence of consent during disputes.

7. **Instant EFT Bank Coverage**: Smart EFT only works with 4 major banks (Nedbank, Standard Bank, FNB, ABSA). If customer uses another bank, payment fails. Implement fallback to credit card or show supported banks during checkout.

8. **Sandbox vs Production Credentials**: Test and production environments use completely different merchant keys and salts. Deploying with test credentials to production will cause all payments to fail. Verify credentials match environment before going live.

9. **Notification URL HTTPS Requirement**: PayU will only POST to HTTPS endpoints. HTTP URLs are rejected. Ensure your notification endpoint has valid SSL/TLS certificate.

10. **Response Hash Signature Verification**: Always verify the `pf_signature` from IPN notifications using the exact parameter order and your merchant salt. Skipping verification leaves you vulnerable to forged payment notifications.

## Useful Links

- [PayU South Africa Developer Hub](https://southafrica.payu.com/developer-hub/)
- [PayU South Africa Merchant Dashboard](https://merchant.payumea.com/)
- [PayU Africa Developer Documentation](https://corporate.payu.com/developer-documentation-africa/)
- [PayU South Africa Payment Gateway](https://southafrica.payu.com/payment-gateway/)
- [PayU South Africa Subscription Payments](https://southafrica.payu.com/subscription-payments/)
- [PayU Instant EFT Solution](https://southafrica.payu.com/ozow/)
- [PayU South Africa FAQs](https://southafrica.payu.com/faqs/)
- [PayU Global Corporate Site](https://corporate.payu.com/)
- [PayU MEA Knowledge Base](https://help.payu.co.za/)
- [Prosus Group](https://www.prosus.com/)
