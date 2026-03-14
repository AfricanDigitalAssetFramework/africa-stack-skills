---
name: FedaPay
description: Francophone West African payment gateway for small merchants and digital entrepreneurs. Supports mobile money (MTN, Moov) and card payments in XOF/XAF across Benin, Côte d'Ivoire, Senegal, and more.
triggers:
  - FedaPay
  - Benin payment gateway
  - Francophone Africa payments
  - XOF payments
  - Small merchant Africa
  - West African payment API
  - MTN mobile money payment
  - Moov payment gateway
---

# FedaPay Payment Gateway

FedaPay is an all-in-one payment aggregator designed for West African businesses, entrepreneurs, and NGOs. It integrates bank card payments and mobile money services, enabling secure transactions across Francophone Africa with a focus on small merchants and digital entrepreneurs.

**Supported Countries:** Benin, Burkina Faso, Côte d'Ivoire, Mali, Niger, Senegal, Togo, and Guinea
**Primary Currency:** XOF (West African CFA Franc), with XAF support in select regions
**Payment Methods:** Mobile money (MTN, Moov, Togocel), bank cards (Visa, Mastercard)

## When to Use This Skill

- Building payment infrastructure for e-commerce platforms in West Africa
- Integrating mobile money payments for small businesses and digital entrepreneurs
- Processing payments in XOF currency for Francophone African markets
- Implementing customer management and payout automation
- Handling recurring billing or subscription payments
- Processing payouts to merchant bank accounts or mobile money wallets
- Need webhook-based real-time payment notifications

## Authentication

FedaPay uses **Bearer token authentication** with API keys stored in your FedaPay dashboard.

### Getting Your API Keys

1. Log in to your FedaPay Dashboard (https://dashboard.fedapay.com)
2. Navigate to your account settings
3. Retrieve both **Test API Key** (for sandbox) and **Live API Key** (for production)
4. Keep API keys secure—never commit them to version control

### API Key Authentication Header

All requests require the `Authorization` header with Bearer token format:

```http
Authorization: Bearer sk_test_YOUR_SECRET_KEY
```

### Setting API Key in Code

#### PHP SDK
```php
<?php
\FedaPay\Fedapay::setApiKey('sk_test_YOUR_SECRET_KEY');
\FedaPay\Fedapay::setEnvironment('sandbox'); // or 'production'
```

#### Node.js
```javascript
const FedaPay = require('fedapay');
FedaPay.setApiKey('sk_test_YOUR_SECRET_KEY');
FedaPay.setEnvironment('sandbox');
```

#### cURL
```bash
curl -H "Authorization: Bearer sk_test_YOUR_SECRET_KEY" \
     https://sandbox-api.fedapay.com/v1/transactions
```

### Environment Selection

- **Sandbox:** `https://sandbox-api.fedapay.com/v1` (testing, no real charges)
- **Production:** `https://api.fedapay.com/v1` (live transactions)

Always test thoroughly in sandbox before switching to production.

## Core API Reference

### Base URLs

```
Sandbox:    https://sandbox-api.fedapay.com/v1
Production: https://api.fedapay.com/v1
```

### Transaction Creation

Create a new transaction to initiate a payment.

**Endpoint:** `POST /transactions`

**Request:**
```json
{
  "description": "Order #12345 - Purchase",
  "amount": 10000,
  "currency": {
    "iso": "XOF"
  },
  "callback_url": "https://yoursite.com/payment-callback",
  "mode": "mtn_open",
  "customer": {
    "id": 1
  }
}
```

**Key Parameters:**
- `description` (string): Payment description shown to customer
- `amount` (integer): Amount in XOF francs (whole units). XOF centimes are not used in practice; treat every unit as one franc. `5000` = 5,000 XOF.
- `currency.iso` (string): Currency code (XOF, XAF)
- `callback_url` (string): URL to redirect after payment completion
- `mode` (string): Payment method (see Payment Methods section)
- `customer` (object): Customer ID or inline customer details
  - `id` (integer): Existing customer ID, or create inline:
  - `firstname` (string): Customer first name
  - `lastname` (string): Customer last name
  - `email` (string): Customer email
  - `phone_number` (string): Customer phone number

**Successful Response (201 Created):**
```json
{
  "id": 12345,
  "status": "pending",
  "description": "Order #12345 - Purchase",
  "amount": 10000,
  "currency": {
    "iso": "XOF",
    "name": "CFA Franc"
  },
  "mode": "mtn_open",
  "reference": "TX-20240115-ABC123",
  "customer": {
    "id": 1,
    "firstname": "John",
    "lastname": "Doe",
    "email": "john@example.com"
  },
  "created_at": "2024-01-15T10:30:00Z"
}
```

### Retrieve Transaction

Get details about a specific transaction.

**Endpoint:** `GET /transactions/{id}`

**Response:**
```json
{
  "id": 12345,
  "status": "approved",
  "description": "Order #12345 - Purchase",
  "amount": 10000,
  "currency": {
    "iso": "XOF"
  },
  "mode": "mtn_open",
  "reference": "TX-20240115-ABC123",
  "approved_at": "2024-01-15T10:35:00Z",
  "customer": {
    "id": 1,
    "firstname": "John",
    "lastname": "Doe"
  }
}
```

### List Transactions

Retrieve paginated list of transactions.

**Endpoint:** `GET /transactions`

**Query Parameters:**
- `limit` (integer): Max results per page (default: 20, max: 100)
- `page` (integer): Page number (default: 1)
- `sort` (string): Sort field (e.g., `-created_at` for descending)

**Response:**
```json
{
  "data": [
    {
      "id": 12345,
      "status": "approved",
      "amount": 10000,
      "currency": { "iso": "XOF" }
    }
  ],
  "meta": {
    "total": 150,
    "current_page": 1,
    "per_page": 20,
    "last_page": 8
  }
}
```

### Customer Management

#### Create Customer

**Endpoint:** `POST /customers`

```json
{
  "firstname": "John",
  "lastname": "Doe",
  "email": "john@example.com",
  "phone_number": "+22960000000",
  "metadata": {
    "merchant_id": "12345"
  }
}
```

**Response:**
```json
{
  "id": 1,
  "firstname": "John",
  "lastname": "Doe",
  "email": "john@example.com",
  "phone_number": "+22960000000",
  "created_at": "2024-01-15T10:00:00Z"
}
```

#### Retrieve Customer

**Endpoint:** `GET /customers/{id}`

## Supported Payment Methods

FedaPay supports the following payment modes (varies by country):

| Mode | Description | Countries |
|------|-------------|-----------|
| `mtn_open` | MTN Mobile Money | Benin |
| `moov` | Moov Mobile Money | Benin, Togo |
| `mtn_ci` | MTN Mobile Money | Côte d'Ivoire |
| `moov_tg` | Moov Mobile Money | Togo |
| `sbin` | SBIC Bank | Benin |
| `togocel` | Togocel Mobile Money | Togo |
| `airtel_ne` | Airtel Mobile Money | Niger |
| `free_sn` | Free Money | Senegal |
| `mtn_open_gn` | MTN Mobile Money | Guinea |

**No-Redirect Modes:** Direct payment possible with MTN Benin, Moov Benin, Moov Togo, and MTN Côte d'Ivoire without redirecting to FedaPay's payment page.

## Webhooks

FedaPay sends real-time notifications to your application when transaction events occur.

### Setting Up Webhooks

1. Log in to FedaPay Dashboard
2. Navigate to **Settings → Webhooks**
3. Click **New Webhook**
4. Enter your webhook URL (must be publicly accessible HTTPS)
5. Select events to subscribe to
6. Save

### Webhook URL Example

```
https://yoursite.com/webhooks/fedapay
```

### Webhook Events

FedaPay sends POST requests with the following event types:

| Event | Trigger |
|-------|---------|
| `transaction.approved` | Payment successfully completed |
| `transaction.declined` | Payment failed or was rejected |
| `transaction.canceled` | Customer canceled the payment |
| `transaction.transferred` | Funds transferred to merchant account |
| `transaction.updated` | Transaction status changed |

### Webhook Payload Structure

```json
{
  "id": "evt_123456",
  "type": "transaction.approved",
  "created_at": "2024-01-15T10:35:00Z",
  "data": {
    "id": 12345,
    "status": "approved",
    "description": "Order #12345",
    "amount": 10000,
    "currency": { "iso": "XOF" },
    "reference": "TX-20240115-ABC123",
    "customer": {
      "id": 1,
      "firstname": "John",
      "lastname": "Doe",
      "email": "john@example.com"
    }
  }
}
```

### Webhook Handling Best Practices

1. **Verify webhook authenticity** using the signature header (if provided)
2. **Respond quickly** with HTTP 200 within 5 seconds
3. **Process asynchronously** - acknowledge immediately, then process in background
4. **Handle retries** - FedaPay may retry failed deliveries
5. **Store webhook data** for audit trails
6. **Never trust callback_url redirects** - verify with transaction retrieval

### Sample Webhook Handler (Node.js)

```javascript
const express = require('express');
const app = express();
app.use(express.json());

app.post('/webhooks/fedapay', (req, res) => {
  const event = req.body;

  // Respond immediately
  res.status(200).json({ received: true });

  // Process asynchronously
  setImmediate(() => {
    switch (event.type) {
      case 'transaction.approved':
        handlePaymentApproved(event.data);
        break;
      case 'transaction.declined':
        handlePaymentDeclined(event.data);
        break;
      default:
        console.log('Unhandled event:', event.type);
    }
  });
});

function handlePaymentApproved(transaction) {
  console.log('Payment approved:', transaction.id);
  // Update order status, send confirmation email, etc.
}

function handlePaymentDeclined(transaction) {
  console.log('Payment declined:', transaction.id);
  // Notify customer, offer retry option, etc.
}

app.listen(3000);
```

## Common Integration Patterns

### 1. Simple Payment Flow

```php
<?php
// 1. Create transaction
\FedaPay\Fedapay::setApiKey('sk_test_YOUR_SECRET_KEY');
\FedaPay\Fedapay::setEnvironment('sandbox');

$transaction = \FedaPay\Transaction::create([
    'description' => 'Order #1001',
    'amount' => 5000, // 5000 XOF
    'currency' => ['iso' => 'XOF'],
    'callback_url' => 'https://yoursite.com/order-confirmation',
    'mode' => 'mtn_open',
    'customer' => [
        'firstname' => 'Jean',
        'lastname' => 'Dupont',
        'email' => 'jean@example.com',
        'phone_number' => '+22960000000'
    ]
]);

// 2. Redirect customer to payment
$paymentUrl = "https://sandbox-process.fedapay.com/transaction/{$transaction->id}";
header("Location: $paymentUrl");
?>
```

### 2. Webhook-Based Order Fulfillment

```javascript
// Verify transaction status via webhook
app.post('/webhooks/fedapay', async (req, res) => {
  const event = req.body;

  res.status(200).json({ received: true });

  if (event.type === 'transaction.approved') {
    const txId = event.data.id;
    const orderId = event.data.reference;

    // Update database
    await Order.updateOne(
      { fedapay_transaction_id: txId },
      { status: 'paid', paid_at: new Date() }
    );

    // Trigger fulfillment
    await fulfillOrder(orderId);
  }
});
```

### 3. Recurring Payments / Subscriptions

Create individual transactions for each billing cycle:

```php
// Monthly subscription
$transaction = \FedaPay\Transaction::create([
    'description' => 'Monthly Subscription - Jan 2024',
    'amount' => 50000, // Monthly fee in XOF
    'currency' => ['iso' => 'XOF'],
    'mode' => 'mtn_open',
    'customer' => ['id' => $customerId]
]);
```

### 4. Split Payments / Commission Calculation

```php
// Calculate platform fee and merchant payout
$totalAmount = 100000; // XOF
$platformFeePercent = 0.05; // 5%
$platformFee = $totalAmount * $platformFeePercent;
$merchantPayout = $totalAmount - $platformFee;

// Create transaction
$transaction = \FedaPay\Transaction::create([
    'amount' => $totalAmount,
    'description' => 'Sale - Payout: ' . $merchantPayout . ' XOF'
]);

// Store commission in database
Commission::create([
    'transaction_id' => $transaction->id,
    'platform_fee' => $platformFee,
    'merchant_payout' => $merchantPayout
]);
```

## Error Handling

FedaPay returns standard HTTP status codes with error details in the response body.

### Common HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK - Request successful |
| 201 | Created - Resource created |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Invalid/missing API key |
| 404 | Not Found - Resource doesn't exist |
| 422 | Unprocessable Entity - Validation error |
| 500 | Internal Server Error - FedaPay error |

### Error Response Format

```json
{
  "status": false,
  "message": "Validation error",
  "errors": {
    "amount": ["Amount is required"],
    "currency": ["Invalid currency code"]
  }
}
```

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| "Invalid API key" | Wrong or expired key | Verify key in dashboard, regenerate if needed |
| "Currency not supported" | Invalid currency code | Use XOF for most countries, check country support |
| "Invalid payment mode" | Mode not available in country | Verify available modes for customer's location |
| "Customer not found" | Invalid customer ID | Create customer first or use inline customer data |
| "Amount too low" | Minimum transaction not met | Check minimum transaction limits (typically 100 XOF) |
| "Duplicate reference" | Reference already used | Use unique transaction references |

### Error Handling Example

```php
<?php
try {
    $transaction = \FedaPay\Transaction::create([
        'description' => 'Order #1001',
        'amount' => 5000,
        'currency' => ['iso' => 'XOF'],
        'mode' => 'mtn_open',
        'customer' => ['id' => 1]
    ]);

    echo "Transaction created: " . $transaction->id;
} catch (\FedaPay\Exception\ApiException $e) {
    // FedaPay API error
    echo "API Error: " . $e->getMessage();
    error_log($e);
} catch (\Exception $e) {
    // Other errors
    echo "Error: " . $e->getMessage();
}
?>
```

## Important Notes & Gotchas

1. **API Keys Are Environment-Specific**: Test and live API keys are completely separate. Using a live key in sandbox mode (or vice versa) will fail. Always verify you're using the correct key for the environment.

2. **Minimum Transaction Amount**: FedaPay enforces minimum transaction amounts (typically 100 XOF). Amounts below the minimum will be rejected. Check with FedaPay support if you need lower limits.

3. **Currency Code Must Match Account**: Your FedaPay account is created for a specific country/currency. Attempting to use a different currency (e.g., XAF if your account is XOF) will fail. Verify your account currency in the dashboard.

4. **Phone Number Format**: Phone numbers must include country code (e.g., +22960000000 for Benin). FedaPay may validate phone formats and reject improperly formatted numbers.

5. **Webhook URLs Must Be HTTPS**: FedaPay will reject HTTP webhook URLs. Your webhook endpoint must use HTTPS and be publicly accessible. Self-signed certificates may not work in production.

6. **Webhook Delivery Is Not Guaranteed**: FedaPay attempts webhook delivery but cannot guarantee delivery. Always verify transaction status by querying the API for critical operations. Don't solely rely on webhooks for order fulfillment.

7. **Transaction Status Can Only Move Forward**: Once a transaction reaches `approved` or `declined`, it cannot be changed. To issue a refund, use the payout system, not transaction reversal.

8. **No-Redirect Mode Has Limitations**: Direct payment without redirecting users only works for specific modes (MTN Benin, Moov Benin, MTN CI, Moov Togo). Other modes require redirecting customers to FedaPay's payment page.

9. **Callback URL Redirect Failures**: If customer redirection fails after payment (network issues, slow page load), the payment still completes. Rely on webhooks or API queries to verify payment status, not redirect URL arrival.

10. **Rate Limiting and Throttling**: FedaPay may rate-limit API requests during high-traffic periods. Implement exponential backoff and retry logic for transient failures. Check rate limit headers in response.

11. **Sandbox vs. Production Data Isolation**: Sandbox and production environments are completely isolated. Transactions created in sandbox do not appear in production, and vice versa. Test comprehensively before going live.

12. **Customer ID vs. Inline Customer Creation**: When creating a transaction, you can either reference an existing customer by ID or provide inline customer details. Mixing both (providing both `customer.id` and inline `firstname`/`lastname`) may cause unexpected behavior.

## Useful Links

- **Official FedaPay Documentation:** https://docs.fedapay.com/api-reference/introduction
- **FedaPay Dashboard:** https://dashboard.fedapay.com
- **PHP SDK GitHub:** https://github.com/fedapay/fedapay-php
- **Node.js SDK (npm):** https://www.npmjs.com/package/fedapay
- **Community & Support:** https://support.fedapay.com
- **Payment Methods Guide:** https://docs.fedapay.com/payment-methods/en/payment-methods-en
- **Webhook Documentation:** https://docs.fedapay.com/integration-api/en/webhooks-en
- **Authentication Guide:** https://docs.fedapay.com/integration-api/en/authentication-en
- **Supported Countries & Currencies:** Contact FedaPay support for current coverage

---

**Last Updated:** February 2026
**API Version:** v1
**Status:** Production-Ready
