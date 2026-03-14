---
name: yoco
description: "Integrate with Yoco's payment API for card payments and POS solutions in South Africa. Use this skill whenever the user wants to accept card payments, build a checkout, manage transactions, handle refunds, process payments via Yoco, integrate POS systems, or work with Yoco's API in any way. Also trigger when the user mentions 'Yoco', 'South African card payments', 'Yoco checkout', 'accept cards in South Africa', or needs payment processing with POS capabilities."
---

# Yoco Integration Skill

Yoco is South Africa's leading fintech payment solution for both online card payments and physical POS terminals. It offers a modern REST API for processing payments, managing transactions, issuing refunds, and building custom checkout experiences. Yoco accepts international cards and settles in ZAR to your South African bank account.

## When to use this skill

You're building a payment system that needs to accept cards in South Africa — an e-commerce checkout, a POS integration, a subscription service, or any solution requiring ZAR payment processing. Yoco handles card payments for both online and offline merchants with a single, unified API.

## Authentication

All Yoco API requests require Bearer token authentication passed in the Authorization header:

```
Authorization: Bearer sk_test_xxxxx
```

Keys come in pairs:
- `sk_test_xxxxx` — Sandbox environment for testing
- `sk_live_xxxxx` — Live production environment

Store your secret key securely in an environment variable like `YOCO_SECRET_KEY`. Never hardcode keys or commit them to version control.

**Base URLs:**
- Live: `https://api.yoco.com`
- Sandbox: `https://api.yocosandbox.com`

**Important:** Live API keys will not work against the sandbox environment and vice versa.

## Core API Reference

### Create a Checkout

Initialize a payment checkout for a customer to complete via card. Customers are redirected to a hosted payment page.

```
POST https://payments.yoco.com/api/checkouts
```

**Request:**
```json
{
  "amount": 1999,
  "currency": "ZAR",
  "metadata": {
    "orderId": "ORDER-12345",
    "customerId": "CUST-67890"
  },
  "successRedirectUrl": "https://yoursite.com/payment/success",
  "failureRedirectUrl": "https://yoursite.com/payment/failure",
  "cancelRedirectUrl": "https://yoursite.com/payment/cancel",
  "description": "Purchase of product XYZ"
}
```

**Parameters:**
- `amount` (required, integer): Amount in cents. R19.99 = 1999 cents.
- `currency` (required, string): "ZAR" (only currency supported)
- `metadata` (optional, object): Custom key-value pairs for tracking (max 10 fields, values must be strings)
- `successRedirectUrl` (required, string): URL to redirect after successful payment
- `failureRedirectUrl` (optional, string): URL to redirect after failed payment
- `cancelRedirectUrl` (optional, string): URL to redirect if customer cancels
- `description` (optional, string): Payment description shown to customer

**Response (success):**
```json
{
  "id": "checkout_abc123def456",
  "redirectUrl": "https://checkout.yoco.com/checkout_abc123def456",
  "status": "pending",
  "amount": 1999,
  "currency": "ZAR",
  "metadata": {
    "orderId": "ORDER-12345",
    "customerId": "CUST-67890"
  },
  "description": "Purchase of product XYZ",
  "createdAt": "2026-02-24T10:30:00Z"
}
```

**Important Notes:**
- Redirect the customer to the `redirectUrl` to complete payment
- The checkout ID is returned to your redirect URLs as a query parameter: `?checkoutId=checkout_abc123def456`
- Do not rely solely on client-side redirects; verify payment status via API or webhook before fulfilling orders
- Use metadata to track orders idempotently and prevent duplicate processing

### Get Checkout Details

Retrieve status and details of a checkout.

```
GET https://payments.yoco.com/api/checkouts/{id}
```

**Path Parameters:**
- `id` (required): The checkout ID (e.g., `checkout_abc123def456`)

**Response:**
```json
{
  "id": "checkout_abc123def456",
  "status": "completed",
  "amount": 1999,
  "currency": "ZAR",
  "metadata": {
    "orderId": "ORDER-12345",
    "customerId": "CUST-67890"
  },
  "description": "Purchase of product XYZ",
  "createdAt": "2026-02-24T10:30:00Z",
  "completedAt": "2026-02-24T10:32:00Z",
  "redirectUrl": "https://checkout.yoco.com/checkout_abc123def456"
}
```

**Status values:** `pending`, `completed`, `failed`, `cancelled`

### Get Transaction Details

Retrieve the full status and details of a payment transaction.

```
GET https://api.yoco.com/v1/transactions/{id}
```

**Path Parameters:**
- `id` (required): The transaction ID (e.g., `txn_abc123def456`)

**Response:**
```json
{
  "id": "txn_abc123def456",
  "checkoutId": "checkout_abc123def456",
  "status": "succeeded",
  "amount": 1999,
  "currency": "ZAR",
  "description": "Purchase of product XYZ",
  "metadata": {
    "orderId": "ORDER-12345",
    "customerId": "CUST-67890"
  },
  "cardNumberLastFour": "4242",
  "cardBrand": "VISA",
  "cardHolder": "John Doe",
  "fee": 40,
  "net": 1959,
  "createdAt": "2026-02-24T10:30:00Z",
  "completedAt": "2026-02-24T10:32:00Z"
}
```

**Status values:** `pending`, `succeeded`, `failed`, `cancelled`, `refunded`

**Important fields:**
- `fee`: Yoco's transaction fee in cents
- `net`: Amount you receive after fees (in cents)

### List Transactions

Retrieve a paginated list of transactions for your account.

```
GET https://api.yoco.com/v1/transactions?limit=50&skip=0&status=succeeded
```

**Query Parameters:**
- `limit` (optional, integer): Records per page (max: 100, default: 50)
- `skip` (optional, integer): Records to skip for pagination (default: 0)
- `status` (optional, string): Filter by status — `pending`, `succeeded`, `failed`, `cancelled`
- `createdAfter` (optional, ISO 8601 date): Filter transactions created after this date
- `createdBefore` (optional, ISO 8601 date): Filter transactions created before this date

**Response:**
```json
{
  "data": [
    {
      "id": "txn_abc123def456",
      "checkoutId": "checkout_abc123def456",
      "status": "succeeded",
      "amount": 1999,
      "currency": "ZAR",
      "fee": 40,
      "net": 1959,
      "cardBrand": "VISA",
      "cardNumberLastFour": "4242",
      "createdAt": "2026-02-24T10:30:00Z",
      "completedAt": "2026-02-24T10:32:00Z"
    }
  ],
  "count": 1,
  "totalCount": 150,
  "hasMore": true
}
```

### Refund a Payment

Refund a previously succeeded transaction in full or partially.

```
POST https://payments.yoco.com/api/checkouts/{id}/refund
```

**Path Parameters:**
- `id` (required): The checkout ID

**Request:**
```json
{
  "amount": 999,
  "description": "Customer requested refund"
}
```

**Parameters:**
- `amount` (optional, integer): Amount in cents to refund. Omit for full refund.
- `description` (optional, string): Refund reason

**Response (success):**
```json
{
  "id": "refund_xyz789abc123",
  "checkoutId": "checkout_abc123def456",
  "status": "succeeded",
  "amount": 1999,
  "currency": "ZAR",
  "description": "Customer requested refund",
  "createdAt": "2026-02-24T10:35:00Z",
  "completedAt": "2026-02-24T10:36:00Z"
}
```

**Important Notes:**
- Refunds process within 5-10 business days to the customer's card
- Only succeededtransactions can be refunded
- Refunds typically only available within 90 days of original transaction
- Test mode refunds are not supported; test refunds using live mode

### List Refunds

Retrieve a list of refunds processed on your account.

```
GET https://api.yoco.com/v1/refunds?limit=50&skip=0
```

**Query Parameters:**
- `limit` (optional, integer): Records per page (max: 100, default: 50)
- `skip` (optional, integer): Records to skip
- `checkoutId` (optional, string): Filter by specific checkout

**Response:**
```json
{
  "data": [
    {
      "id": "refund_xyz789abc123",
      "checkoutId": "checkout_abc123def456",
      "status": "succeeded",
      "amount": 1999,
      "currency": "ZAR",
      "createdAt": "2026-02-24T10:35:00Z",
      "completedAt": "2026-02-24T10:36:00Z"
    }
  ],
  "count": 1,
  "totalCount": 5,
  "hasMore": false
}
```

## Webhooks

Yoco sends webhooks for payment events. Configure a secure POST endpoint to receive and validate webhook payloads.

### Webhook Events

- `payment.succeeded` — Payment successfully processed
- `payment.failed` — Payment failed
- `refund.succeeded` — Refund completed
- `refund.failed` — Refund failed

### Webhook Payload

```json
{
  "type": "payment.succeeded",
  "id": "evt_abc123def456",
  "timestamp": "2026-02-24T10:32:00Z",
  "data": {
    "id": "txn_abc123def456",
    "checkoutId": "checkout_abc123def456",
    "status": "succeeded",
    "amount": 1999,
    "currency": "ZAR",
    "cardBrand": "VISA",
    "cardNumberLastFour": "4242",
    "description": "Purchase of product XYZ",
    "fee": 40,
    "net": 1959,
    "metadata": {
      "orderId": "ORDER-12345"
    }
  }
}
```

### Signature Verification

Yoco signs each webhook with `X-Yoco-Signature` header using HMAC-SHA256. Verify signatures to ensure webhooks are authentic.

**Signed Content Format:**
```
{webhook-id}.{webhook-timestamp}.{request-body}
```

**Python Implementation:**
```python
import hmac
import hashlib
import base64
import time
import json

def verify_webhook_signature(signature_header, request_body, webhook_secret):
    """
    Verify Yoco webhook signature.

    Args:
        signature_header: Value from X-Yoco-Signature header (format: "v1,signature")
        request_body: Raw request body (string or bytes)
        webhook_secret: Your webhook secret key (remove whsec_ prefix if present)

    Returns:
        bool: True if signature is valid, False otherwise
    """
    # Parse signature header: "v1,base64_encoded_signature"
    parts = signature_header.split(',')
    if len(parts) != 2 or parts[0] != 'v1':
        return False

    received_signature = parts[1]

    # Extract webhook metadata from request body
    try:
        if isinstance(request_body, bytes):
            request_body = request_body.decode('utf-8')

        body_data = json.loads(request_body)
        webhook_id = body_data.get('id')
        webhook_timestamp = body_data.get('timestamp')

        if not webhook_id or not webhook_timestamp:
            return False
    except (json.JSONDecodeError, UnicodeDecodeError):
        return False

    # Verify timestamp is within acceptable threshold (3 minutes recommended)
    current_time = time.time()
    webhook_time = int(webhook_timestamp.timestamp()) if hasattr(webhook_timestamp, 'timestamp') else int(webhook_timestamp)
    if abs(current_time - webhook_time) > 180:  # 3 minutes in seconds
        return False

    # Remove whsec_ prefix from secret if present
    secret = webhook_secret.replace('whsec_', '') if webhook_secret.startswith('whsec_') else webhook_secret

    # Construct signed content
    signed_content = f"{webhook_id}.{webhook_timestamp}.{request_body}"

    # Generate expected signature
    expected_signature = base64.b64encode(
        hmac.new(
            secret.encode('utf-8'),
            signed_content.encode('utf-8'),
            hashlib.sha256
        ).digest()
    ).decode('utf-8')

    # Constant-time comparison to prevent timing attacks
    return hmac.compare_digest(received_signature, expected_signature)

# Flask example
from flask import request

@app.route('/webhook', methods=['POST'])
def yoco_webhook():
    signature_header = request.headers.get('X-Yoco-Signature')
    webhook_secret = os.getenv('YOCO_WEBHOOK_SECRET')

    if not verify_webhook_signature(signature_header, request.get_data(as_text=True), webhook_secret):
        return {'error': 'Invalid signature'}, 401

    data = request.json
    event_type = data.get('type')

    if event_type == 'payment.succeeded':
        handle_payment_success(data['data'])
    elif event_type == 'payment.failed':
        handle_payment_failed(data['data'])
    elif event_type == 'refund.succeeded':
        handle_refund_success(data['data'])

    return {'status': 'received'}, 200
```

**Node.js Implementation:**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(signatureHeader, requestBody, webhookSecret) {
  /**
   * Verify Yoco webhook signature.
   *
   * @param {string} signatureHeader - Value from X-Yoco-Signature header
   * @param {string|Buffer} requestBody - Raw request body
   * @param {string} webhookSecret - Your webhook secret (remove whsec_ prefix if present)
   * @returns {boolean} - True if signature is valid
   */

  // Parse signature header: "v1,base64_encoded_signature"
  const [version, receivedSignature] = signatureHeader.split(',');
  if (version !== 'v1' || !receivedSignature) {
    return false;
  }

  // Convert body to string if necessary
  const bodyString = typeof requestBody === 'string' ? requestBody : requestBody.toString('utf-8');

  // Extract webhook metadata
  try {
    const bodyData = JSON.parse(bodyString);
    const webhookId = bodyData.id;
    const webhookTimestamp = bodyData.timestamp;

    if (!webhookId || !webhookTimestamp) {
      return false;
    }

    // Verify timestamp is within threshold (3 minutes)
    const currentTime = Math.floor(Date.now() / 1000);
    const webhookTime = Math.floor(new Date(webhookTimestamp).getTime() / 1000);
    if (Math.abs(currentTime - webhookTime) > 180) {
      return false;
    }

    // Remove whsec_ prefix from secret if present
    const secret = webhookSecret.startsWith('whsec_')
      ? webhookSecret.slice(6)
      : webhookSecret;

    // Construct signed content
    const signedContent = `${webhookId}.${webhookTimestamp}.${bodyString}`;

    // Generate expected signature
    const expectedSignature = crypto
      .createHmac('sha256', secret)
      .update(signedContent)
      .digest('base64');

    // Constant-time comparison
    return crypto.timingSafeEqual(
      Buffer.from(receivedSignature),
      Buffer.from(expectedSignature)
    );
  } catch (error) {
    return false;
  }
}

// Express example
app.post('/webhook', (req, res) => {
  const signatureHeader = req.headers['x-yoco-signature'];
  const webhookSecret = process.env.YOCO_WEBHOOK_SECRET;

  const rawBody = req.rawBody; // Ensure you capture raw body

  if (!verifyWebhookSignature(signatureHeader, rawBody, webhookSecret)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const { type, data } = req.body;

  if (type === 'payment.succeeded') {
    handlePaymentSuccess(data);
  } else if (type === 'payment.failed') {
    handlePaymentFailed(data);
  } else if (type === 'refund.succeeded') {
    handleRefundSuccess(data);
  }

  res.json({ status: 'received' });
});
```

**Important Notes:**
- Always use the raw request body for signature verification (no JSON parsing beforehand)
- Remove the `whsec_` prefix from the webhook secret before calculating the signature
- Use constant-time string comparison to prevent timing attacks
- Validate timestamp is within 3 minutes of current time to prevent replay attacks

## Online Checkout vs Popup Checkout

Yoco offers two main checkout experiences:

### Online Checkout (Hosted Checkout)
- Customer redirected to Yoco-hosted payment page
- Low integration effort
- Full control over Yoco's checkout UI
- Best for: Mobile users, simple integrations, cross-domain checkout
- Implementation: Use `POST /checkouts` and redirect to `redirectUrl`
- The checkout handles all card validation and PCI compliance

### Popup Checkout
- Payment form displayed as modal on your website
- Customers stay on your site during payment
- Requires JavaScript integration
- Best for: SPA applications, seamless UX, React/Vue projects
- Implementation: Use Yoco's JavaScript SDK for popup display
- More control over styling and UX flow

**Choose Online Checkout if:** You want minimal integration effort, need broad browser support, or don't have a frontend framework.

**Choose Popup Checkout if:** You're building a modern SPA, want users to stay on your site, or need to customize the checkout flow.

## Common Integration Patterns

### Pattern 1: E-commerce Checkout Flow
```
1. Customer proceeds to checkout
2. POST /checkouts → create checkout session
3. Redirect customer to checkout.redirectUrl
4. Customer enters card details on Yoco-hosted page
5. Customer redirected back to success/failure URL
6. GET /checkouts/{id} or GET /transactions/{id} → verify payment
7. Listen for webhook on payment.succeeded → fulfil order
8. Update order status and send confirmation email
```

### Pattern 2: Subscription Billing (Manual Renewal)
```
1. Store customer metadata (email, subscription ID)
2. On billing date, POST /checkouts with subscription metadata
3. Send checkout link to customer via email or SMS
4. Customer completes payment via Yoco checkout
5. Listen for payment.succeeded webhook
6. Update subscription status, extend access
7. For cancellations, pause future checkouts
8. For failed payments, retry within 3-7 days
```

### Pattern 3: Refund Processing
```
1. Customer requests refund
2. POST /refunds with transactionId and amount
3. Return refund ID to customer immediately
4. Listen for refund.succeeded webhook
5. Send refund confirmation email (typically 5-10 business days)
6. Update order status to refunded in your system
7. For partial refunds, track remaining balance in metadata
```

### Pattern 4: POS + Online Reconciliation
```
1. Use Yoco physical terminals for in-store payments
2. Use Checkout API for online payments
3. GET /transactions?createdAfter=yesterday → fetch all sales
4. Reconcile POS transactions (from terminal reports) with API transactions
5. Generate unified reporting dashboard
6. Track combined online + offline revenue per day
7. Settle funds from both channels to single bank account
```

## Error Handling

Yoco returns consistent error responses with HTTP status codes:

```json
{
  "code": "INVALID_REQUEST",
  "message": "The request body is invalid",
  "details": {
    "field": "amount",
    "issue": "Amount must be greater than 0"
  }
}
```

### HTTP Status Codes

| Status | Meaning | Action |
|--------|---------|--------|
| 400 | Bad Request | Check request format and validate all required fields |
| 401 | Unauthorized | Verify API key is correct and not expired |
| 404 | Not Found | Check that resource IDs are correct |
| 429 | Too Many Requests | Implement exponential backoff retry strategy |
| 500 | Server Error | Retry with exponential backoff; check status.yoco.com |

### Common Error Codes

| Code | Cause | Solution |
|------|-------|----------|
| `INVALID_REQUEST` | Malformed request body | Validate JSON structure and field types |
| `AUTHENTICATION_ERROR` | Invalid or missing API key | Verify sk_test_ or sk_live_ key in Authorization header |
| `RESOURCE_NOT_FOUND` | Checkout/transaction doesn't exist | Check that IDs are correct and from the same environment |
| `PAYMENT_FAILED` | Card declined or processing error | Show customer error details; suggest retry or different card |
| `REFUND_NOT_ALLOWED` | Can't refund this transaction | Verify transaction succeeded, is within 90 days, and not already refunded |

### Retry Strategy

```python
import time
import random

def api_call_with_retry(method, url, headers, data=None, max_retries=3):
    """Retry API calls with exponential backoff."""
    for attempt in range(max_retries):
        try:
            response = requests.request(method, url, headers=headers, json=data)

            # Don't retry on client errors (4xx except 429)
            if 400 <= response.status_code < 500 and response.status_code != 429:
                return response

            # Retry on server errors (5xx) and rate limits (429)
            if response.status_code >= 500 or response.status_code == 429:
                if attempt == max_retries - 1:
                    return response

                # Exponential backoff with jitter
                wait_time = (2 ** attempt) + random.uniform(0, 1)
                time.sleep(wait_time)
                continue

            return response

        except requests.RequestException as e:
            if attempt == max_retries - 1:
                raise
            wait_time = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait_time)
```

## Important Implementation Notes

1. **Amount Precision:** Always use integers for cents to avoid floating-point errors. R19.99 = 1999 cents. Never send decimal amounts.

2. **Status Verification:** Don't rely solely on client-side redirects. Always verify payment status via `GET /transactions/{id}` or webhook before fulfilling orders. Customers may close the browser before reaching the redirect URL.

3. **Webhook Validation:** Always verify webhook signatures using HMAC-SHA256 with constant-time comparison. This prevents spoofing and ensures webhooks originate from Yoco.

4. **Transaction Fees:** Yoco charges a percentage fee per transaction. Check your dashboard for exact rates. The `fee` field in responses shows the deducted amount; `net` shows your revenue.

5. **Idempotency:** Use `metadata.orderId` or similar to track orders and prevent duplicate charges if a checkout is completed multiple times.

6. **Refund Limits:** Refunds only available for succeeded transactions, typically within 90 days. Test refunds require live mode (test mode refunds not supported). Refunds take 5-10 business days to appear in customer's account.

7. **Sandbox vs Live:** Test mode keys (sk_test_) only work at https://api.yocosandbox.com. Live keys (sk_live_) only work at https://api.yoco.com. Test transactions do not create real charges.

8. **PCI Compliance:** Yoco's hosted checkout is PCI DSS Level 1 compliant. Never send raw card details to your server; always use Yoco's checkout or tokenization endpoints.

9. **Rate Limiting:** Yoco implements rate limiting. If you receive 429 status, wait before retrying. Implement exponential backoff with jitter.

10. **Environment Variables:** Store all secrets in environment variables:
    ```
    YOCO_API_KEY=sk_test_xxxxx  (or sk_live_xxxxx)
    YOCO_WEBHOOK_SECRET=whsec_xxxxx
    ```

## Fees and Settlement

- Yoco charges a percentage-based transaction fee (check your dashboard for your specific rate)
- Settlement typically occurs daily to your linked South African bank account
- The `fee` field in transaction responses shows the exact fee deducted
- The `net` field shows the amount deposited to your account
- International cards are accepted; funds settle in ZAR

## Currency Support

- **ZAR (South African Rand):** Amounts specified in cents (1 ZAR = 100 cents)
- International cards accepted; conversion handled by payment networks
- Only ZAR currency code supported via API

## Useful Links

- [Yoco Developer Hub](https://developer.yoco.com/)
- [API Integration Keys & Setup](https://developer.yoco.com/online/resources/integration-keys/)
- [Using the REST API](https://developer.yoco.com/docs/api/using-the-rest-api)
- [Create Checkout Reference](https://developer.yoco.com/checkout-api-reference/checkout/create-checkout)
- [Webhook Signature Verification](https://developer.yoco.com/online/api-reference/webhooks/verifying-events/)
- [Dashboard](https://dashboard.yoco.com)
- [Status Page](https://status.yoco.com)
- [Support](https://support.yoco.help)
