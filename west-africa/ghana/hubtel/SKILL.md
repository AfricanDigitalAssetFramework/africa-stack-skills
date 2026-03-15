---
name: Hubtel Payment Gateway
description: Ghana's leading payment gateway API supporting mobile money, cards, POS, QR codes, and SMS services for collecting and disbursing payments in GHS
triggers:
  - Hubtel
  - Ghana payments
  - Ghana mobile money
  - GHS checkout
  - MTN MoMo Ghana
  - Vodafone Cash Ghana
  - AirtelTigo Ghana
---

# Hubtel Payment Gateway

Hubtel is Ghana's leading payment gateway and platform, enabling instant payments processing and money transfers across all major mobile money networks (MTN Mobile Money, Vodafone Cash, AirtelTigo) alongside card payments, GHQR codes, and SMS services. The API allows merchants to collect payments, send money to customers, and manage transactions programmatically.

## When to Use This Skill

- **Mobile Money Collections**: Accepting payments from MTN MoMo, Vodafone Cash, or AirtelTigo customers in Ghana
- **Card Payments**: Processing debit and credit card transactions with GHS settlement
- **Disbursements**: Sending money to customers' mobile wallets or bank accounts
- **Payment Status Tracking**: Querying transaction status and reconciliation
- **SMS Notifications**: Sending transactional or promotional SMS messages to Ghanaian numbers
- **Bill Payments**: Enabling customers to pay utilities via your platform
- **eCommerce Integration**: Building online checkout flows for Ghanaian merchants
- **Subscription Payments**: Processing recurring payments and billing
- **B2B Transfers**: Facilitating money transfers between businesses and individuals

## Authentication

Hubtel uses **HTTP Basic Authentication** for all API requests. You must provide a Base64-encoded string of your ClientID and ClientSecret in the Authorization header.

### Getting API Credentials

> ℹ️ **Self-service signup now available.** Hubtel has introduced self-service merchant registration at [https://unity.hubtel.com](https://unity.hubtel.com). You can create an account and generate API keys without needing to email support, though full activation for live transactions still requires business verification.

1. Register at [https://unity.hubtel.com](https://unity.hubtel.com) or contact [support@hubtel.com](mailto:support@hubtel.com) for enterprise onboarding
2. Log in to your merchant dashboard at [https://unity.hubtel.com](https://unity.hubtel.com)
3. Navigate to API accounts section and select "Request For New API Keys"
4. Choose "HTTP Rest API" as the API type
5. Provide a description for your application
6. Save and your **ClientID**, **ClientSecret**, and **Account Number** will be displayed

> ⚠️ **API versioning note:** Hubtel has been reorganising their API ecosystem. The primary collection/disbursement API currently documents at `payproxyapi.hubtel.com`, while newer endpoints may appear under `api.hubtel.com/v2/`. If you encounter 404s on the paths documented here, check the [Hubtel developer portal](https://developers.hubtel.com) for the current base URLs — they have been updated without redirect in past migrations.

### Authentication Header Format

```bash
# Step 1: Create Base64 string
# base64_encode("YOUR_CLIENT_ID:YOUR_CLIENT_SECRET")

# Step 2: Add to request header
Authorization: Basic {base64_encoded_credentials}
```

### Example Implementation

```javascript
// Node.js example
const clientId = "your_client_id";
const clientSecret = "your_client_secret";
const credentials = Buffer.from(`${clientId}:${clientSecret}`).toString('base64');

const headers = {
  "Authorization": `Basic ${credentials}`,
  "Content-Type": "application/json"
};
```

```python
# Python example
import base64
import requests

client_id = "your_client_id"
client_secret = "your_client_secret"
credentials = base64.b64encode(f"{client_id}:{client_secret}".encode()).decode()

headers = {
    "Authorization": f"Basic {credentials}",
    "Content-Type": "application/json"
}
```

```php
// PHP example
$clientId = "your_client_id";
$clientSecret = "your_client_secret";
$credentials = base64_encode("{$clientId}:{$clientSecret}");

$headers = [
    "Authorization: Basic {$credentials}",
    "Content-Type: application/json"
];
```

## Core API Reference

### Base URL

```
https://payproxyapi.hubtel.com
```

### Online Checkout - Receive Money

Create a checkout invoice to collect payments from customers via mobile money, cards, or GHQR.

**Endpoint:**
```
POST https://payproxyapi.hubtel.com/items/initiate
```

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `InvoiceId` | string | Yes | Unique invoice identifier from your system |
| `TotalAmount` | decimal | Yes | Amount in GHS (e.g., 50.00) |
| `Description` | string | Yes | Transaction description visible to customer |
| `CustomerName` | string | Yes | Name of the customer |
| `CustomerEmail` | string | No | Customer's email address |
| `CustomerMsisdn` | string | No | Customer's phone number (e.g., 233XXXXXXXXX) |
| `PrimaryCallbackUrl` | string | Yes | Webhook URL for payment notifications |
| `SecondaryCallbackUrl` | string | No | Fallback webhook URL |
| `ReturnUrl` | string | Yes | URL to redirect customer after payment |
| `CancellationUrl` | string | No | URL to redirect if customer cancels |
| `Logo` | string | No | Merchant logo URL |

**Request Example:**

```json
{
  "InvoiceId": "INV-2024-001",
  "TotalAmount": 150.00,
  "Description": "Payment for Order #12345",
  "CustomerName": "Kwaku Asante",
  "CustomerEmail": "kwaku@example.com",
  "CustomerMsisdn": "233241234567",
  "PrimaryCallbackUrl": "https://yoursite.com/webhook/hubtel",
  "SecondaryCallbackUrl": "https://yoursite.com/webhook/hubtel-backup",
  "ReturnUrl": "https://yoursite.com/order-confirmation",
  "CancellationUrl": "https://yoursite.com/order-cancelled"
}
```

**Response Example (Success):**

```json
{
  "ResponseCode": "00",
  "ResponseMessage": "Success",
  "Data": {
    "CheckoutUrl": "https://checkout.hubtel.com/pay/yB4zHgK9mP",
    "CheckoutId": "yB4zHgK9mP",
    "InvoiceId": "INV-2024-001",
    "Amount": 150.00,
    "TransactionId": "TRX-2024-123456"
  }
}
```

### Receive Money via Direct API

Request payment directly from a customer without redirecting to Hubtel checkout page.

**Endpoint:**
```
POST https://payproxyapi.hubtel.com/receive/initiate
```

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `CustomerName` | string | Yes | Customer's full name |
| `CustomerMsisdn` | string | Yes | Customer's phone number (e.g., 233XXXXXXXXX) |
| `CustomerEmail` | string | No | Customer's email address |
| `Amount` | decimal | Yes | Amount in GHS |
| `Description` | string | Yes | Payment description |
| `Channel` | string | No | Payment channel (e.g., 'mtn-gh', 'vodafone-gh', 'airtel-gh') |
| `ClientReference` | string | Yes | Your unique transaction reference (max 32 chars) |
| `PrimaryCallbackUrl` | string | Yes | Webhook for payment updates |
| `SecondaryCallbackUrl` | string | No | Fallback webhook URL |

**Request Example:**

```json
{
  "CustomerName": "Abena Mensah",
  "CustomerMsisdn": "233541234567",
  "CustomerEmail": "abena@example.com",
  "Amount": 75.50,
  "Description": "Service charge for account upgrade",
  "Channel": "mtn-gh",
  "ClientReference": "REF-20240215-001",
  "PrimaryCallbackUrl": "https://yoursite.com/webhook/payment",
  "SecondaryCallbackUrl": "https://yoursite.com/webhook/payment-backup"
}
```

**Response Example (Success):**

```json
{
  "ResponseCode": "00",
  "ResponseMessage": "Success",
  "Data": {
    "TransactionId": "TRX-2024-654321",
    "ClientReference": "REF-20240215-001",
    "Amount": 75.50,
    "Status": "Pending",
    "Message": "Payment prompt sent to customer"
  }
}
```

### Send Money (Disbursement)

Send money from your Hubtel merchant account to a customer's mobile wallet.

**Endpoint:**
```
POST https://payproxyapi.hubtel.com/send/money
```

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `RecipientName` | string | Yes | Recipient's full name |
| `RecipientMsisdn` | string | Yes | Recipient's phone number |
| `Amount` | decimal | Yes | Amount in GHS to send |
| `Description` | string | Yes | Disbursement reason |
| `Channel` | string | No | Network channel (e.g., 'mtn-gh', 'vodafone-gh') |
| `ClientReference` | string | Yes | Your unique reference for tracking |

**Request Example:**

```json
{
  "RecipientName": "Ama Osei",
  "RecipientMsisdn": "233551234567",
  "Amount": 200.00,
  "Description": "Refund for order cancellation",
  "Channel": "mtn-gh",
  "ClientReference": "REFUND-20240215-123"
}
```

**Response Example:**

```json
{
  "ResponseCode": "00",
  "ResponseMessage": "Success",
  "Data": {
    "TransactionId": "TRX-2024-999123",
    "ClientReference": "REFUND-20240215-123",
    "Amount": 200.00,
    "Status": "Processing"
  }
}
```

### Check Transaction Status

Query the status of a transaction that was initiated earlier.

**Endpoint:**
```
GET https://payproxyapi.hubtel.com/transaction/{transactionId}
```

or

```
POST https://payproxyapi.hubtel.com/transaction/status
```

**Request Parameters (POST):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `TransactionId` | string | Conditional | Hubtel transaction ID |
| `ClientReference` | string | Conditional | Your client reference |
| `InvoiceId` | string | Conditional | Checkout invoice ID |

**Response Example:**

```json
{
  "ResponseCode": "00",
  "ResponseMessage": "Success",
  "Data": {
    "TransactionId": "TRX-2024-123456",
    "ClientReference": "REF-20240215-001",
    "Amount": 75.50,
    "Status": "Completed",
    "PaymentMethod": "MTN Mobile Money",
    "Timestamp": "2024-02-15T14:30:45Z",
    "Description": "Service charge for account upgrade"
  }
}
```

### SMS Quick Send

Send SMS messages to Ghanaian numbers.

**Endpoint:**
```
GET/POST https://api.hubtel.com/v1/messages/send
```

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `From` | string | Yes | Sender ID (alphanumeric, max 11 chars) |
| `To` | string | Yes | Recipient phone number |
| `Content` | string | Yes | Message content (max 160 chars per SMS) |
| `ClientId` | string | Yes | Your API ClientId |
| `ClientSecret` | string | Yes | Your API ClientSecret |
| `RegisteredDelivery` | boolean | No | Request delivery confirmation |

**Request Example:**

```
https://api.hubtel.com/v1/messages/send?From=YourBiz&To=233541234567&Content=Hello%20Kwaku%20your%20payment%20of%20GHS%2050%20was%20successful&ClientId=YOUR_CLIENT_ID&ClientSecret=YOUR_CLIENT_SECRET&RegisteredDelivery=true
```

**Response Example:**

```json
{
  "Code": "00",
  "Message": "Success",
  "Data": {
    "MessageId": "MSG-2024-001",
    "Status": "Sent",
    "RecipientNumber": "233541234567"
  }
}
```

## Webhooks / Callbacks

Hubtel sends real-time payment status notifications to your callback URL after transactions complete.

### Callback Request Format

Hubtel will POST a JSON payload to your `PrimaryCallbackUrl` when a transaction status changes.

**Webhook Endpoint (Your Server):**
```
POST https://yoursite.com/webhook/hubtel
```

**Request Headers:**
```
Content-Type: application/json
```

**Webhook Payload Example:**

```json
{
  "TransactionId": "TRX-2024-123456",
  "ClientReference": "REF-20240215-001",
  "InvoiceId": "INV-2024-001",
  "Amount": 150.00,
  "Status": "Completed",
  "PaymentMethod": "MTN Mobile Money",
  "CustomerName": "Kwaku Asante",
  "CustomerMsisdn": "233241234567",
  "Description": "Payment for Order #12345",
  "Timestamp": "2024-02-15T14:30:45Z",
  "ResponseCode": "00",
  "ResponseMessage": "Transaction Successful"
}
```

### Webhook Response Requirements

Your webhook endpoint **MUST respond with HTTP 200** within the specified timeout period to confirm receipt:

```json
{
  "success": true,
  "message": "Webhook received and processed"
}
```

### Webhook Status Values

| Status | Meaning |
|--------|---------|
| `Completed` | Transaction successful, customer paid |
| `Pending` | Payment in progress, awaiting customer response |
| `Cancelled` | Customer cancelled payment |
| `Failed` | Transaction failed (insufficient funds, wrong PIN, etc.) |
| `Timeout` | Customer did not respond within time limit |

### Webhook Retry Policy

- Hubtel attempts delivery to your `PrimaryCallbackUrl`
- If primary fails or times out, retries are sent to `SecondaryCallbackUrl` if provided
- Implement idempotency by checking `TransactionId` or `ClientReference` to prevent duplicate processing
- Always log webhook payloads for audit and reconciliation purposes

## Common Integration Patterns

### Pattern 1: Mobile Money Payment Collection

Collect payment from a customer's MTN Mobile Money wallet:

```javascript
// Step 1: Initiate receive money request
const requestData = {
  CustomerName: "John Doe",
  CustomerMsisdn: "233541234567",
  Amount: 50.00,
  Description: "Product purchase",
  Channel: "mtn-gh",
  ClientReference: "ORDER-123",
  PrimaryCallbackUrl: "https://yoursite.com/webhook/payment"
};

// Step 2: Send to Hubtel API
const response = await fetch('https://payproxyapi.hubtel.com/receive/initiate', {
  method: 'POST',
  headers: {
    'Authorization': `Basic ${credentials}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(requestData)
});

// Step 3: Handle response
const result = await response.json();
if (result.ResponseCode === '00') {
  console.log('Payment initiated, customer will receive USSD prompt');
} else {
  console.error('Failed to initiate payment:', result.ResponseMessage);
}

// Step 4: Handle webhook callback
app.post('/webhook/payment', (req, res) => {
  const { TransactionId, Status, Amount, ClientReference } = req.body;

  if (Status === 'Completed') {
    // Update order as paid
    updateOrderStatus(ClientReference, 'paid');
  } else if (Status === 'Failed') {
    // Notify customer of failed payment
    notifyCustomer(ClientReference, 'payment_failed');
  }

  res.json({ success: true });
});
```

### Pattern 2: Online Checkout with Redirect

Create a hosted checkout page for customers:

```javascript
// Step 1: Generate checkout URL
const checkoutData = {
  InvoiceId: "INV-2024-" + Date.now(),
  TotalAmount: 150.00,
  Description: "Complete your purchase",
  CustomerName: "Jane Smith",
  CustomerEmail: "jane@example.com",
  CustomerMsisdn: "233251234567",
  ReturnUrl: "https://yoursite.com/payment-success",
  CancellationUrl: "https://yoursite.com/payment-cancelled",
  PrimaryCallbackUrl: "https://yoursite.com/webhook/checkout"
};

const response = await fetch('https://payproxyapi.hubtel.com/items/initiate', {
  method: 'POST',
  headers: {
    'Authorization': `Basic ${credentials}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(checkoutData)
});

const result = await response.json();

// Step 2: Redirect customer to Hubtel checkout
if (result.ResponseCode === '00') {
  window.location.href = result.Data.CheckoutUrl;
}

// Step 3: Handle return from Hubtel
app.get('/payment-success', (req, res) => {
  // Customer was redirected here, verify with status check
  res.render('success', { message: 'Payment successful!' });
});
```

### Pattern 3: Money Disbursement / Refund

Send money back to a customer:

```javascript
// Send refund after order cancellation
const disbursement = {
  RecipientName: "John Doe",
  RecipientMsisdn: "233541234567",
  Amount: 50.00,
  Description: "Refund for cancelled order #123",
  Channel: "mtn-gh",
  ClientReference: "REFUND-123-" + Date.now()
};

const response = await fetch('https://payproxyapi.hubtel.com/send/money', {
  method: 'POST',
  headers: {
    'Authorization': `Basic ${credentials}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(disbursement)
});

const result = await response.json();

if (result.ResponseCode === '00') {
  console.log('Refund initiated:', result.Data.TransactionId);
  // Update order record
  markOrderAsRefunded(disbursement.ClientReference);
}
```

### Pattern 4: Transaction Status Verification

Check payment status (critical after 5 minutes):

```javascript
// Called if you don't receive webhook after 5 minutes
async function verifyTransactionStatus(transactionId, clientReference) {
  const response = await fetch(
    `https://payproxyapi.hubtel.com/transaction/${transactionId}`,
    {
      method: 'GET',
      headers: {
        'Authorization': `Basic ${credentials}`
      }
    }
  );

  const result = await response.json();

  if (result.ResponseCode === '00') {
    const status = result.Data.Status;

    if (status === 'Completed') {
      updateOrderStatus(clientReference, 'paid');
    } else if (status === 'Failed') {
      updateOrderStatus(clientReference, 'payment_failed');
    } else if (status === 'Pending') {
      // Still processing, check again later
      scheduleRetry(transactionId);
    }
  }

  return result;
}
```

## USSD Product

Hubtel's USSD product enables you to build interactive USSD menus accessible by any mobile user in Ghana via shortcodes (e.g. `*714#`). Works without internet on any handset — widely used for mobile money flows, service menus, and payment prompts.

### USSD Webhook Payload (Hubtel → your server)
```json
{
  "Mobile": "233241234567",
  "SessionId": "ATUid_abc123",
  "ServiceCode": "*714*100#",
  "Type": "Initiation",
  "Message": "",
  "Operator": "MTN",
  "Sequence": 1
}
```
- `Type`: `Initiation` (first menu), `Response` (customer replied), `Release` (session ended), `Timeout`
- `Message`: what the customer typed in response to your previous menu

### USSD Response Format (your server → Hubtel)
```json
{
  "Type": "Response",
  "Message": "Welcome to MyService\n1. Check Balance\n2. Pay Bill\n3. Exit"
}
```
To end the session: `"Type": "Release"`. Customer sees the final message and session closes.

> Contact Hubtel to be assigned a USSD shortcode. Shared shortcodes (sub-menus of `*714#`) are available for quick onboarding; dedicated shortcodes require licensing from Ghana's NCA.

## Error Handling

### Common Response Codes

| Code | Message | Meaning | Action |
|------|---------|---------|--------|
| `00` | Success | Transaction successful or initiated | Check webhook for final status |
| `01` | Insufficient Balance | Customer has insufficient funds | Retry or notify customer to top up |
| `02` | Transaction Declined | Payment declined by network | Try different payment method |
| `03` | Invalid Parameters | Missing or invalid request data | Check request format and retry |
| `04` | User Not Found | Invalid phone number or customer | Verify phone number format |
| `05` | Transaction Timeout | Customer didn't respond in time | Retry transaction |
| `06` | Network Error | Hubtel system error | Implement exponential backoff retry |

### HTTP Status Codes

| Status | Meaning |
|--------|---------|
| `200` | Request successful, check ResponseCode in body |
| `400` | Bad request (invalid parameters) |
| `401` | Unauthorized (invalid credentials) |
| `403` | Forbidden (account not authorized for endpoint) |
| `429` | Rate limited (exceeded request quota) |
| `500` | Server error (retry with exponential backoff) |

### Error Response Example

```json
{
  "ResponseCode": "03",
  "ResponseMessage": "Invalid Parameters",
  "Data": {
    "Error": "CustomerMsisdn is required",
    "Field": "CustomerMsisdn"
  }
}
```

### Error Handling Best Practices

```javascript
async function makeHubtelRequest(endpoint, data) {
  const maxRetries = 3;
  let retryCount = 0;

  while (retryCount < maxRetries) {
    try {
      const response = await fetch(endpoint, {
        method: 'POST',
        headers: {
          'Authorization': `Basic ${credentials}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data),
        timeout: 15000 // 15 second timeout
      });

      const result = await response.json();

      // Handle specific error codes
      if (result.ResponseCode === '06') {
        // Network error - retry with exponential backoff
        await delay(Math.pow(2, retryCount) * 1000);
        retryCount++;
        continue;
      } else if (result.ResponseCode === '01' || result.ResponseCode === '02') {
        // Insufficient balance or declined - don't retry
        throw new Error(`Payment failed: ${result.ResponseMessage}`);
      } else if (result.ResponseCode === '00') {
        return result;
      }

      break;

    } catch (error) {
      if (retryCount < maxRetries - 1) {
        retryCount++;
        await delay(Math.pow(2, retryCount) * 1000);
      } else {
        throw error;
      }
    }
  }
}
```

## Important Notes / Gotchas

### 1. Phone Number Format is Critical
- Always include country code: `233` (not `0`) for Ghana
- Example: `233541234567` (not `0541234567`)
- Hubtel will reject incorrectly formatted numbers
- Validate format before sending to API

### 2. Five-Minute Rule for Status Checks
- **You must check transaction status if webhook isn't received within 5 minutes**
- Webhooks are not 100% guaranteed delivery
- Always implement a timeout-based status verification mechanism
- Use `ClientReference` for idempotency and tracking

### 3. ClientReference Must Be Unique
- Each transaction should have a unique `ClientReference`
- Use this for idempotency - if you retry with same reference, Hubtel won't double-charge
- Keep references between 1-32 characters
- Examples: `ORDER-12345`, `REF-20240215-001`, `REFUND-ABC123`

### 4. Currency is Always GHS
- All amounts are in Ghana cedis (GHS)
- Decimal format: `150.00` (not `15000` for pesewas)
- No currency symbol in API requests
- Hubtel quotes fees in GHS separately

### 5. Webhook Idempotency is Your Responsibility
- Webhooks may be delivered multiple times
- Always check if a transaction was already processed before updating database
- Use `TransactionId` or `ClientReference` to deduplicate
- Database unique constraints recommended

### 6. Authenticate via Headers, Not URL Parameters
- Use `Authorization: Basic` header format for production
- Never pass credentials in URL parameters for sensitive endpoints
- URL parameters only for SMS API (Quick Send)
- Store credentials securely (environment variables, secrets manager)

### 7. Different Networks Have Different Speeds
- MTN MoMo: Usually completes within 30-60 seconds
- Vodafone Cash: 45-90 seconds
- AirtelTigo: 60-120 seconds
- Always give users 2-3 minutes before prompting retry
- Network congestion may cause delays

### 8. Test Credentials vs. Live Credentials
- Hubtel provides separate test and production API credentials
- Test credentials work against sandbox environment
- Transactions in test environment are simulated
- Switch to live credentials only when ready for production
- Never hardcode credentials - use environment variables

### 9. Callback URL Must Be Publicly Accessible
- Webhook endpoint cannot be localhost or private IP
- HTTPS is required for production
- Implement timeout handling (respond within 30 seconds)
- Log all webhook payloads for compliance and debugging
- Use services like ngrok or RequestBin for testing

### 10. Amount Precision Matters
- Use decimal type, not integers, for amounts
- Examples: `150.00`, not `15000` or `150`
- Some payment methods may not support certain decimal precisions
- Document minimum transaction amount with customers (typically GHS 0.10)

### 11. Rate Limiting Applies
- Hubtel typically limits to 5 requests per minute per merchant
- Implement request queuing for bulk operations
- Add exponential backoff for 429 responses
- Contact Hubtel support for higher rate limits if needed

### 12. Test in Sandbox First
- Always test complete integration before going live
- Use test phone numbers provided by Hubtel
- Verify webhook delivery and error handling
- Test network failures and timeouts
- Don't skip testing on staging environment

## Useful Links

- **Official Hubtel Developer Portal**: [https://developers.hubtel.com](https://developers.hubtel.com)
- **API Documentation**: [https://businessdocs-developers.hubtel.com](https://businessdocs-developers.hubtel.com)
- **Merchant Dashboard**: [https://unity.hubtel.com/merchantaccount/dashboard](https://unity.hubtel.com/merchantaccount/dashboard)
- **Online Checkout Integration**: [https://explore.hubtel.com/integrate-online-checkout/](https://explore.hubtel.com/integrate-online-checkout/)
- **Support Email**: [support@hubtel.com](mailto:support@hubtel.com)
- **GitHub - OVAC PHP SDK**: [https://github.com/ovac/hubtel-payment](https://github.com/ovac/hubtel-payment)
- **GitHub - Hubtel Web Checkout SDK**: [https://github.com/hubtel/hubtel-web-merchant-checkout-sdk](https://github.com/hubtel/hubtel-web-merchant-checkout-sdk)
- **Node.js SMS SDK**: [https://github.com/KwabenBerko/hubtel-sms](https://github.com/KwabenBerko/hubtel-sms)
- **Integration Tutorials**: [https://news.hubtel.com](https://news.hubtel.com)
