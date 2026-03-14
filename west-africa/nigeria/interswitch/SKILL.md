---
name: interswitch
description: "Integrate with Interswitch APIs for payment processing, transfers, bill payments, and collections across Nigeria and Africa. Use this skill when building payment platforms, fintech solutions, merchant networks, or payment collection systems. Trigger on requests mentioning: Interswitch, Quickteller Business, payment switches, bank transfers, bill payments, payment collections, or merchant payment routing."
---

# Interswitch Integration Skill

Interswitch is Africa's largest payments switching and settlement infrastructure provider, operating a network connecting 3,000+ merchants, all major Nigerian banks, mobile operators, and utility service providers. The platform enables developers to build scalable payment solutions with unified APIs for transfers, collections, bill payments, and transaction management.

## When to use this skill

You're building a payment platform that needs to serve Nigeria's financial ecosystem with:

- **Bank transfers** to any Nigerian bank account using unified APIs instead of managing bank-specific integrations
- **Payment collections** at scale across merchants, marketplaces, and service providers
- **Bill payments** to over 3,000 billers (utilities, telecommunications, government services)
- **Airtime and data vending** across all Nigerian mobile networks
- **Payment routing** through Nigeria's largest merchant network
- **Real-time transaction settlement and reconciliation**

Interswitch abstracts away the complexity of Nigeria's fragmented banking infrastructure, providing a single integration point for accessing the entire financial network. This makes it ideal for fintech apps, enterprise payment systems, marketplaces, and e-commerce platforms.

## Authentication

Interswitch uses OAuth 2.0 client credentials flow combined with signature-based security headers.

### Step 1: Obtain Access Token

Use your Client ID and Secret Key (provided in the Interswitch Developer Console) to request an access token via OAuth 2.0.

**Endpoint:** `POST /passport/oauth/token` (Sandbox) or `POST https://api-gateway.interswitchng.com/passport/oauth/token` (Production)

**Headers:**
```
Authorization: Basic base64(client_id:secret_key)
Content-Type: application/x-www-form-urlencoded
```

**Body:**
```
grant_type=client_credentials
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read write"
}
```

**Important:** Tokens expire after 1 hour. Implement token caching and refresh logic before expiry.

### Step 2: Include in API Requests

All subsequent API requests must include the access token in the Authorization header:

```
Authorization: Bearer {access_token}
X-Access-Signature: {signature}
Timestamp: {unix_timestamp}
Nonce: {random_nonce}
SignatureMethod: SHA256
```

### Environment Setup

Store credentials securely in environment variables:
```bash
INTERSWITCH_CLIENT_ID=your_client_id
INTERSWITCH_SECRET_KEY=your_secret_key
INTERSWITCH_MERCHANT_CODE=your_merchant_code
```

### Base URLs

- **Sandbox:** `https://sandbox.interswitchng.com`
- **Production:** `https://api-gateway.interswitchng.com` and `https://live.interswitchng.com`

Authenticate separately for sandbox and production using different credentials.

---

## Core API Reference

### Initiate Bank Transfer

Transfer funds to any Nigerian bank account using NUBAN (11-digit Nigerian Uniform Bank Account Number).

**Endpoint:** `POST /api/v2/quickteller/payments/transfers`

**Headers:**
```
Authorization: Bearer {access_token}
X-Access-Signature: {signature}
Content-Type: application/json
Timestamp: {unix_timestamp}
Nonce: {random_nonce}
SignatureMethod: SHA256
```

**Request Body:**
```json
{
  "amount": 100000,
  "currency": "NGN",
  "beneficiary": {
    "name": "Chinedu Adeyemi",
    "bankCode": "058",
    "accountNumber": "0123456789"
  },
  "narration": "Payment for services rendered",
  "reference": "TRF-2025-001",
  "transactionRef": "REF-2025-001"
}
```

**Response (Success):**
```json
{
  "responseCode": "00",
  "responseMessage": "Transfer initiated successfully",
  "data": {
    "transactionId": "TXN_12345abcde",
    "reference": "TRF-2025-001",
    "amount": 100000,
    "currency": "NGN",
    "status": "Processing",
    "timestamp": "2026-02-24T14:30:00Z",
    "beneficiary": {
      "accountNumber": "0123456789",
      "bankCode": "058",
      "name": "Chinedu Adeyemi"
    }
  }
}
```

**Important Notes:**
- Amount is in **kobo** (₦500 = 50,000 kobo). Never send in naira.
- Bank code is the 3-digit CBN code. Always validate against the banks list endpoint.
- `reference` and `transactionRef` must be globally unique per transaction.
- Transfers typically complete within 1-5 minutes but may take longer during off-hours.
- Monitor status via webhook or polling the status endpoint.

---

### Get Transfer Status

Poll the status of a submitted transfer transaction.

**Endpoint:** `GET /api/v2/quickteller/payments/transfers/status`

**Query Parameters:**
```
reference=TRF-2025-001
```

**Response:**
```json
{
  "responseCode": "00",
  "responseMessage": "Transfer status retrieved",
  "data": {
    "reference": "TRF-2025-001",
    "transactionId": "TXN_12345abcde",
    "amount": 100000,
    "currency": "NGN",
    "status": "Completed",
    "completedAt": "2026-02-24T14:31:00Z",
    "beneficiaryAccount": "0123456789",
    "beneficiaryBank": "Guaranty Trust Bank",
    "beneficiaryAccountName": "Chinedu Adeyemi"
  }
}
```

**Status Values:**
- `Pending` - Transfer submitted, awaiting processing
- `Processing` - Transfer in progress
- `Completed` - Transfer succeeded
- `Failed` - Transfer failed; check error details

---

### Verify Bank Account

Validate a bank account before initiating a transfer. Returns the registered account holder name.

**Endpoint:** `GET /api/v2/quickteller/banks/account/verify`

**Query Parameters:**
```
bankCode=058
accountNumber=0123456789
```

**Response (Success):**
```json
{
  "responseCode": "00",
  "responseMessage": "Account verified",
  "data": {
    "accountName": "Chinedu Adeyemi",
    "accountNumber": "0123456789",
    "bankCode": "058",
    "bankName": "Guaranty Trust Bank"
  }
}
```

**Response (Failed):**
```json
{
  "responseCode": "05",
  "responseMessage": "Account validation failed - invalid account",
  "data": null
}
```

**Important Notes:**
- Always verify accounts before transferring to prevent sending money to incorrect recipients.
- Compare returned `accountName` against expected recipient name.
- Cache results for 24 hours to minimize API calls.

---

### Get List of Supported Banks

Retrieve all banks and their CBN codes supported by Interswitch for transfers.

**Endpoint:** `GET /api/v2/quickteller/banks`

**Response:**
```json
{
  "responseCode": "00",
  "responseMessage": "Banks retrieved successfully",
  "data": [
    {
      "bankCode": "033",
      "bankName": "United Bank for Africa (UBA)",
      "bankSlug": "uba",
      "isActive": true
    },
    {
      "bankCode": "058",
      "bankName": "Guaranty Trust Bank",
      "bankSlug": "gtb",
      "isActive": true
    },
    {
      "bankCode": "044",
      "bankName": "Access Bank",
      "bankSlug": "access-bank",
      "isActive": true
    },
    {
      "bankCode": "050",
      "bankName": "Ecobank",
      "bankSlug": "ecobank",
      "isActive": true
    }
  ]
}
```

**Important Notes:**
- Cache this list locally and refresh every 24 hours.
- Use for validating user bank selection and populating dropdown menus.
- Only use banks where `isActive: true`.

---

### Initialize Payment Collection

Create a payment collection session. Direct customer to the checkout URL to complete payment via card, bank transfer, or USSD.

**Endpoint:** `POST /collections/api/v1/pay`

**Headers:**
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "amount": 50000,
  "customerName": "Amina Okafor",
  "customerEmail": "customer@example.com",
  "customerMobile": "+2348012345678",
  "narration": "Invoice INV-2025-001 payment",
  "paymentReference": "PAY-2025-001",
  "currency": "NGN",
  "redirectUrl": "https://yoursite.com/payment/callback",
  "merchantCode": "your_merchant_code",
  "payItemId": "your_payitem_id"
}
```

**Response:**
```json
{
  "responseCode": "00",
  "responseMessage": "Payment session initiated successfully",
  "data": {
    "checkoutUrl": "https://checkout.interswitch.com/pay/v3/abc123def456",
    "sessionId": "SESSION_XYZ789",
    "paymentReference": "PAY-2025-001",
    "amount": 50000,
    "currency": "NGN",
    "status": "Initiated",
    "expiresAt": "2026-02-25T14:30:00Z"
  }
}
```

**Important Notes:**
- Amount is in kobo.
- `paymentReference` must be unique and never reused.
- Customer is redirected to `checkoutUrl` to complete payment.
- After payment, customer is redirected to `redirectUrl` or you receive a webhook notification.
- Payment sessions expire after 24 hours.

---

### Get Transaction Details / Verify Payment

Retrieve full transaction details to verify payment completion.

**Endpoint:** `GET /collections/api/v1/gettransaction.json`

**Query Parameters:**
```
transactionRef=PAY-2025-001
amount=50000
```

**Response (Successful Payment):**
```json
{
  "isSuccessful": true,
  "message": "Transaction Successful",
  "responseCode": "00",
  "data": {
    "transactionRef": "PAY-2025-001",
    "amount": 50000,
    "transactionStatus": "Paid",
    "paymentMethod": "Card",
    "paymentMethodDetails": {
      "cardType": "Visa",
      "cardLast4": "1234",
      "cardBrand": "VISA"
    },
    "datePaid": "2026-02-24T15:45:00Z",
    "customerEmail": "customer@example.com",
    "customerName": "Amina Okafor",
    "transactionHash": "hash_abc123def456"
  }
}
```

**Response (Failed Payment):**
```json
{
  "isSuccessful": false,
  "message": "Transaction Failed",
  "responseCode": "01",
  "data": {
    "transactionRef": "PAY-2025-001",
    "amount": 50000,
    "transactionStatus": "Failed",
    "failureReason": "Card declined by issuer"
  }
}
```

**Important Notes:**
- Always verify the `amount` in response matches your expected amount (prevents amount manipulation).
- Check `transactionStatus: "Paid"` for successful payments.
- Cache verification results to minimize API calls (important for high-volume merchants).

---

### Get Billers (for Bill Payments)

Retrieve available billers for bill payment services (electricity, water, government services, etc.).

**Endpoint:** `GET /quickteller/bills/billers`

**Query Parameters (Optional):**
```
categoryId=1     (Optional: filter by category)
pageSize=100
pageNumber=1
```

**Response:**
```json
{
  "responseCode": "00",
  "responseMessage": "Billers retrieved successfully",
  "data": {
    "billers": [
      {
        "billerId": "407",
        "billerName": "AEDC (Abuja Distribution Company)",
        "billerCode": "AEDC",
        "categoryId": "1",
        "categoryName": "Electricity",
        "customerFieldName": "Meter Number",
        "logo": "https://s3.amazonaws.com/aedc-logo.png",
        "surcharge": 0,
        "isActive": true
      },
      {
        "billerId": "415",
        "billerName": "IKEDC (Ikeja Distribution Company)",
        "billerCode": "IKEDC",
        "categoryId": "1",
        "categoryName": "Electricity",
        "customerFieldName": "Meter Number",
        "logo": "https://s3.amazonaws.com/ikedc-logo.png",
        "surcharge": 0,
        "isActive": true
      }
    ],
    "totalCount": 150,
    "pageSize": 100,
    "pageNumber": 1
  }
}
```

**Important Notes:**
- Over 3,000 billers available across all major categories.
- Cache biller list and refresh daily.
- Use `billerId` in bill payment requests.

---

### Send Bill Payment

Process a bill payment to a biller.

**Endpoint:** `POST /quickteller/bills/pay`

**Request Body:**
```json
{
  "billerId": "407",
  "amount": 10000,
  "currency": "NGN",
  "customerRef": "METER123456",
  "phoneNumber": "+2348012345678",
  "email": "customer@example.com",
  "reference": "BILL-2025-001"
}
```

**Response:**
```json
{
  "responseCode": "00",
  "responseMessage": "Bill payment processed successfully",
  "data": {
    "transactionId": "BILL_TXN_12345",
    "reference": "BILL-2025-001",
    "billerId": "407",
    "billerName": "AEDC",
    "amount": 10000,
    "status": "Processing",
    "timestamp": "2026-02-24T16:00:00Z"
  }
}
```

---

### Airtime and Data Purchase

Purchase airtime or data bundles for mobile networks.

**Endpoint:** `POST /quickteller/airtime/recharge`

**Request Body:**
```json
{
  "phoneNumber": "+2348012345678",
  "amount": 1000,
  "networkCode": "1",
  "reference": "AIRTIME-2025-001",
  "serviceType": "Airtime"
}
```

**Response:**
```json
{
  "responseCode": "00",
  "responseMessage": "Airtime purchase successful",
  "data": {
    "transactionId": "AIRTIME_TXN_12345",
    "phoneNumber": "+2348012345678",
    "amount": 1000,
    "networkName": "MTN",
    "status": "Completed",
    "reference": "AIRTIME-2025-001"
  }
}
```

**Network Codes:**
- `1` - MTN
- `2` - Airtel
- `3` - GLO
- `4` - 9Mobile

---

## Webhooks

Interswitch sends real-time webhook notifications for transaction events. Always verify webhook signatures before processing.

### Webhook Verification

All webhooks are signed with HmacSHA512 using your secret key. Verify before processing:

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secretKey) {
  const hash = crypto
    .createHmac('sha512', secretKey)
    .update(JSON.stringify(payload))
    .digest('hex');

  return hash === signature;
}

// Express.js example
app.post('/webhooks/interswitch', (req, res) => {
  const signature = req.headers['x-interswitch-signature'];
  const payload = req.body;

  if (!verifyWebhookSignature(payload, signature, process.env.INTERSWITCH_SECRET_KEY)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook
  handleWebhookEvent(payload);

  // Return 200 to acknowledge receipt
  res.status(200).json({ success: true });
});
```

### Webhook Payload Examples

**Event: Transaction Completed**
```json
{
  "eventId": "evt_abc123def456",
  "eventType": "TRANSACTION.COMPLETED",
  "timestamp": "2026-02-24T16:30:00Z",
  "data": {
    "transactionRef": "PAY-2025-001",
    "paymentReference": "PAY-2025-001",
    "amount": 50000,
    "currency": "NGN",
    "responseCode": "00",
    "responseMessage": "Transaction successful",
    "transactionStatus": "Paid",
    "customerEmail": "customer@example.com",
    "paymentMethod": "Card",
    "merchantCode": "MX123",
    "dateCompleted": "2026-02-24T16:30:00Z"
  }
}
```

**Event: Transaction Failed**
```json
{
  "eventId": "evt_xyz789abc123",
  "eventType": "TRANSACTION.FAILED",
  "timestamp": "2026-02-24T16:35:00Z",
  "data": {
    "transactionRef": "PAY-2025-002",
    "paymentReference": "PAY-2025-002",
    "amount": 75000,
    "currency": "NGN",
    "responseCode": "05",
    "responseMessage": "Transaction failed",
    "failureReason": "Card declined by issuer",
    "customerEmail": "customer@example.com",
    "merchantCode": "MX123"
  }
}
```

**Event: Transfer Completed**
```json
{
  "eventId": "evt_transfer_123",
  "eventType": "TRANSFER.COMPLETED",
  "timestamp": "2026-02-24T16:45:00Z",
  "data": {
    "reference": "TRF-2025-001",
    "transactionId": "TXN_12345abcde",
    "amount": 100000,
    "currency": "NGN",
    "status": "Completed",
    "beneficiary": {
      "accountNumber": "0123456789",
      "bankCode": "058",
      "bankName": "Guaranty Trust Bank"
    }
  }
}
```

### Webhook Retry Policy

- Interswitch retries failed webhook deliveries up to **5 times**.
- Retry delays increase exponentially (backoff strategy).
- Always respond with HTTP 200 to confirm receipt.
- Any response code other than 200 triggers a retry.

### Configuring Webhooks

Set your webhook URL in the Interswitch Dashboard under Integration Settings:

1. Log in to [Interswitch Developer Console](https://developer.interswitchgroup.com/)
2. Navigate to Webhook Settings
3. Add your endpoint: `https://yoursite.com/webhooks/interswitch`
4. Select event types to subscribe to
5. Copy your webhook secret key for signature verification

---

## Common Integration Patterns

### Pattern 1: Payment Collection Flow

Collect payments from customers via Interswitch checkout:

```
1. POST /collections/api/v1/pay
   ↓ Response with checkoutUrl and sessionId
2. Redirect customer to checkoutUrl
   ↓ Customer completes payment
3. Listen for webhook TRANSACTION.COMPLETED
   ↓ OR
   Poll GET /collections/api/v1/gettransaction.json
   ↓ Verify transactionStatus: "Paid"
4. Fulfill order/service
5. Confirm to customer
```

**Implementation Checklist:**
- Generate unique `paymentReference` for each session
- Store `sessionId` for reconciliation
- Implement idempotent webhook handling (duplicate webhooks possible)
- Handle both webhook and polling verification
- Implement 30-second polling fallback if webhook delivery fails

---

### Pattern 2: Secure Bank Transfer Flow

Transfer funds to customer or vendor accounts:

```
1. Collect destination bank details from user
2. GET /api/v2/quickteller/banks/account/verify
   ↓ Verify accountName matches expected recipient
3. Prompt user to confirm ("Send ₦500 to Chinedu Adeyemi?")
4. POST /api/v2/quickteller/payments/transfers
   ↓ Store transactionId and reference
5. Listen for webhook TRANSFER.COMPLETED
   ↓ OR
   Poll GET /api/v2/quickteller/payments/transfers/status
   ↓ Check status transitions (Processing → Completed/Failed)
6. Update user dashboard with final status
7. Send confirmation notification
```

**Security Considerations:**
- Always verify account names before transfer
- Implement user confirmation step for amounts over threshold
- Log all transfer requests for audit
- Monitor for duplicate transfer attempts (use unique references)
- Implement rate limiting for transfer endpoints

---

### Pattern 3: Multi-Biller Payment Processing

Build a bill payment aggregator:

```
1. GET /quickteller/bills/billers (cache daily)
2. Display billers grouped by category
3. User selects biller and enters meter/account number
4. POST /collections/api/v1/pay with billerId and amount
5. Redirect to checkout OR auto-deduct if enabled
6. Webhook notifies of payment completion
7. Auto-submit payment to biller using Interswitch's backend integration
8. Update customer's service (electricity restored, balance updated, etc.)
```

**Optimization Tips:**
- Cache biller list in Redis/Memcached with 24-hour TTL
- Pre-validate customer references with biller before payment
- Implement bulk bill payment for corporate customers
- Use webhooks exclusively (polling causes rate limit issues at scale)

---

### Pattern 4: Recurring Subscription Payments

Set up subscription billing:

```
1. Initialize first payment via /collections/api/v1/pay
2. Store card reference or customer token from successful payment
3. On subscription renewal date:
   POST /collections/api/v1/pay with stored card reference
4. Webhook notifies of payment success/failure
5. If failed, retry 3 times over 7 days
6. Send notification to customer
7. Resume/suspend service based on payment status
```

---

## Error Handling

### Standard Response Codes

All Interswitch API responses include a `responseCode` field. Always check this first:

| Code | Meaning | Action |
|------|---------|--------|
| `00` | Success | Proceed with transaction |
| `01` | Invalid credentials / Unauthorized | Check API keys and token expiry |
| `02` | Invalid request / Bad format | Validate request body against schema |
| `03` | Transaction not found | Verify transaction reference |
| `04` | Insufficient funds | Retry or inform customer to add funds |
| `05` | Account validation failed | Confirm account details with user |
| `06` | Duplicate transaction | Use unique references; check for previous attempt |
| `07` | Daily limit exceeded | Inform customer of transaction limits |
| `09` | Parameter/data missing | Check all required fields are provided |
| `13` | Merchant not configured | Verify merchant code in dashboard |
| `99` | Server error / Timeout | Retry with exponential backoff |

### Error Response Example

```json
{
  "responseCode": "05",
  "responseMessage": "Account validation failed - account not found",
  "data": {
    "errorDetails": "The specified account number does not exist",
    "suggestedAction": "Verify account number and bank code"
  }
}
```

### Retry Strategy

Implement exponential backoff for retryable errors (`99` and some `04` responses):

```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries || !isRetryable(error)) {
        throw error;
      }
      const delay = Math.pow(2, attempt - 1) * 1000; // 1s, 2s, 4s
      await sleep(delay);
    }
  }
}

function isRetryable(error) {
  const retryableCodes = ['99', '04'];
  return retryableCodes.includes(error.responseCode);
}
```

---

## Important Notes and Gotchas

### Critical Implementation Details

**1. Amount Units (Kobo, Not Naira)**
- ALL amounts are in **kobo** (₦1 = 100 kobo)
- Common mistake: Sending ₦500 as `500` instead of `50000`
- Always multiply naira amounts by 100 in your code
- Double-check amount conversions in integration tests

```javascript
const naira = 500;
const kobo = naira * 100; // 50000
```

**2. Bank Code Validation**
- Always validate bank codes against `/api/v2/quickteller/banks` list
- Invalid codes cause transfer failures with no recovery
- Cache the bank list locally; refresh every 24 hours
- Use in dropdowns to prevent user error

**3. Unique References (Critical)**
- Every transaction needs a globally unique reference
- Never reuse references across transactions
- Use UUID or timestamp-based references: `REF-${Date.now()}-${randomId()}`
- Duplicate references may return cached result from previous attempt

**4. Account Verification Before Transfer**
- Always call `/api/v2/quickteller/banks/account/verify` before transfer
- Prevents transferring to wrong accounts (irreversible)
- Compare returned `accountName` against user's expected recipient
- Expired accounts return verification errors; don't retry immediately

**5. Token Management**
- Access tokens expire after **1 hour**
- Implement token caching with refresh before expiry
- Store in Redis or memory cache
- Don't store tokens in client-side localStorage (security risk)
- Separate token per environment (sandbox vs production)

**6. Webhook Security**
- Always verify webhook signatures using HmacSHA512
- Never process unverified webhooks
- Respond with HTTP 200 within 30 seconds
- Implement idempotency: webhooks may be delivered multiple times
- Store processed webhook IDs to detect duplicates

**7. Response Code Checking (Always!)**
- Check `responseCode: "00"` before processing any response
- Don't assume success without explicit verification
- Handle error codes explicitly (don't just check for truthy data)
- Log all non-00 responses for debugging

**8. Sandbox vs Production Differences**
- Separate credentials and base URLs
- Test thoroughly in sandbox before production
- Production uses real money; test edge cases extensively
- Some features may be rate-limited in production (contact support)
- Production has higher transaction limits

**9. Polling vs Webhooks**
- Use webhooks for real-time updates (strongly recommended)
- Polling causes rate limit issues at scale
- Minimum polling interval: 5 seconds (but not recommended)
- For critical confirmations: combine polling + webhooks
- Implement exponential backoff in polling: 5s → 10s → 30s

**10. Customer Name Verification**
- For transfers, always verify account names
- Scammers often use slightly different names
- Implement strict matching; ask customer to confirm
- Log mismatches for fraud investigation
- Flag transfers to accounts with generic names ("Account Holder")

### Common Failure Scenarios

**Scenario: "Insufficient funds" error during transfer**
- Check merchant account balance (dashboard)
- Transfer from wrong account? Verify using Interswitch portal
- Daily/transaction limits exceeded? Contact support
- Sandbox use? Test credentials may have limited balance

**Scenario: "Duplicate transaction" error**
- Reference already exists in system
- Check transaction history in dashboard
- Use different reference for retry
- Don't repeat identical request within 5 minutes

**Scenario: Account verification returns not found**
- Account number or bank code incorrect
- Account doesn't exist or is closed
- CBN code changes? Update from `/api/v2/quickteller/banks`
- Non-NUBAN account? Use bank-specific verification

**Scenario: Webhook not received but payment succeeded**
- Webhook delivery may be delayed (max 5 retries over ~1 hour)
- Implement polling as fallback
- Check transaction status via verification endpoint
- Store webhook IDs to avoid reprocessing
- Check firewall/DNS for webhook URL accessibility

---

## Useful Links

**Official Documentation:**
- [Interswitch Developer Console](https://developer.interswitchgroup.com/)
- [Interswitch API Documentation](https://docs.interswitchgroup.com/)
- [Interswitch DocBase (Comprehensive Guides)](https://sandbox.interswitchng.com/docbase/)
- [Authentication Guide](https://docs.interswitchgroup.com/docs/authentication)
- [Webhooks Configuration](https://docs.interswitchgroup.com/docs/webhooks)
- [Bill Payments API](https://docs.interswitchgroup.com/docs/bills-payment-1)
- [Card Payments API](https://docs.interswitchgroup.com/docs/payment-api)

**Sandbox Testing:**
- [Sandbox Dashboard](https://sandbox.interswitchng.com/)
- Test Card: `5555555555554444` (Visa)
- Test OTP: Any 6-digit number
- Sandbox tokens have different expiry

**Security:**
- [Interswitch Security Headers](https://developer.interswitch.com/interswitch-security-headers/)
- [Response Codes & Error Handling](https://docs.interswitchgroup.com/docs/payment-response-codes)

**Community & Support:**
- Email: developer@interswitchgroup.com
- Support portal in Interswitch Developer Console
- Check status page for incident updates

---

## Integration Checklist

Before going live with Interswitch:

- [ ] Registered on Interswitch Developer Console
- [ ] Created app and obtained credentials
- [ ] Tested all endpoints in sandbox
- [ ] Implemented OAuth 2.0 token refresh
- [ ] Added webhook signature verification
- [ ] Implemented error handling for all response codes
- [ ] Bank account verification before transfers
- [ ] Unique reference generation per transaction
- [ ] Token caching with 1-hour refresh
- [ ] Kobo conversion tests (₦ to kobo)
- [ ] Webhook endpoint receives POST requests
- [ ] Proper logging for debugging
- [ ] Rate limiting implementation
- [ ] Fallback polling for failed webhooks
- [ ] Production credentials configured
- [ ] Compliance with Interswitch TOS
- [ ] Load testing in sandbox
- [ ] Reconciliation process implemented
- [ ] Customer support processes for transaction issues
