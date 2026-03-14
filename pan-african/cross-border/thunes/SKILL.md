---
title: Thunes Cross-Border Payment Infrastructure
triggers:
  - Thunes
  - cross-border payments Africa
  - international remittance API
  - pan-African transfers
  - mobile money cross-border
  - Thunes Money Transfer API
  - global payouts
---

# Thunes Cross-Border Payment Infrastructure

Thunes is a global cross-border payment infrastructure platform that enables financial institutions, fintech companies, and money transfer operators to send and receive payments across 130+ countries. With deep African coverage through 50+ intra-Africa corridors, Thunes connects to over 200 million mobile wallets, 4 billion bank accounts, and multiple alternative payment rails including cash pickup services.

## When to Use Thunes

- **International Remittances**: Building a remittance product for diaspora sending money home
- **Cross-Border Payouts**: Paying out earnings, salaries, or gig work to recipients globally
- **Intra-Africa Transfers**: Moving money between African countries via mobile money, bank accounts, or cash pickup
- **Multi-Corridor Expansion**: Your product needs to scale across multiple countries with a single API integration
- **Mobile Money Networks**: Accessing M-Pesa, MTN Mobile Money, Orange Money, and other digital wallets
- **Settlement Services**: Providing settlement infrastructure for fintech platforms, PSPs, or money transfer operators
- **Real-Time Transparency**: Customers need instant confirmation and tracking of international transactions

## Authentication

Thunes uses **HTTP Basic Authentication** with API credentials (API_KEY and API_SECRET).

### Getting Credentials

API credentials are generated through the Thunes Portal after completing the initial compliance review. Your account will receive:
- `API_KEY`: Your unique API identifier
- `API_SECRET`: Your authentication secret

**Important**: API credentials must be generated via the Thunes Portal's Developer Module. Self-service API access is available after account activation, but initial account setup requires a partnership agreement review.

### Base URLs

| Environment | URL |
|-------------|-----|
| **Sandbox** | `https://api-sandbox.thunes.com` |
| **Production** | `https://api.thunes.com` |

Replace `{API_ENDPOINT}` in all examples below with the appropriate URL for your environment.

### Basic Auth Header

All API requests must include an Authorization header with base64-encoded credentials:

```
Authorization: Basic base64(API_KEY:API_SECRET)
```

### cURL Example

```bash
curl -X GET https://{API_ENDPOINT}/ping \
  -H "Authorization: Basic $(echo -n 'YOUR_API_KEY:YOUR_API_SECRET' | base64)" \
  -H "Content-Type: application/json"
```

### Node.js Example

```javascript
const https = require('https');

const apiKey = process.env.THUNES_API_KEY;
const apiSecret = process.env.THUNES_API_SECRET;
const credentials = Buffer.from(`${apiKey}:${apiSecret}`).toString('base64');

const options = {
  hostname: process.env.THUNES_API_ENDPOINT,
  port: 443,
  path: '/money-transfer/v2/quotes',
  method: 'POST',
  headers: {
    'Authorization': `Basic ${credentials}`,
    'Content-Type': 'application/json'
  }
};

const req = https.request(options, (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => console.log(JSON.parse(data)));
});

req.write(JSON.stringify(requestBody));
req.end();
```

### Python Example

```python
import requests
import os
from base64 import b64encode

api_key = os.getenv('THUNES_API_KEY')
api_secret = os.getenv('THUNES_API_SECRET')
api_endpoint = os.getenv('THUNES_API_ENDPOINT')

credentials = b64encode(f'{api_key}:{api_secret}'.encode()).decode()

headers = {
    'Authorization': f'Basic {credentials}',
    'Content-Type': 'application/json'
}

response = requests.post(
    f'https://{api_endpoint}/money-transfer/v2/quotes',
    headers=headers,
    json=request_body
)
```

## Core API Reference

### Environments

Thunes provides two environments:
- **Pre-Production (Sandbox)**: For testing and development
- **Production**: For live transactions

**Note**: Specific endpoint URLs are provided upon account creation. Endpoints are accessible via:
- HTTPS (TLS 1.2 or later required)
- IPSec VPN (optional, for high-security integrations)

### Discovery Endpoints

Before creating transactions, retrieve available services, countries, and payout methods:

#### List Services

```
GET /money-transfer/v2/services
```

Returns available services like "MobileWallet", "BankAccount", "CashPickup", "Card".

#### List Countries

```
GET /money-transfer/v2/countries
```

Retrieve all supported countries and their codes.

#### List Payers (Payout Methods)

```
GET /money-transfer/v2/countries/{countryCode}/services/{serviceCode}/payers
```

Get available payout methods for a specific country/service combination, including:
- Required fields
- Minimum/maximum limits
- Supported currencies and precision

**Payer Example Response**:

```json
{
  "payers": [
    {
      "payerId": "MM_TZ_MPESA_BANK",
      "service": "BankAccount",
      "country": "TZ",
      "requiredFields": [
        "fullName",
        "accountNumber",
        "bankCode"
      ],
      "limits": {
        "minAmount": 10,
        "maxAmount": 50000
      },
      "currency": "TZS",
      "currencyPrecision": 2
    }
  ]
}
```

### Create Quote

Get a quote for a transaction (exchange rate, fees, recipient amount):

```
POST /money-transfer/v2/quotes
```

**Request Body**:

```json
{
  "sendingCountry": "US",
  "receivingCountry": "KE",
  "sendingCurrency": "USD",
  "receivingCurrency": "KES",
  "paymentMethod": "BankDebit",
  "payoutMethod": "MobileWallet",
  "amount": 100.00,
  "payerId": "MM_KE_MPESA"
}
```

**Response**:

```json
{
  "quoteId": "QT_1234567890",
  "sendingAmount": 100.00,
  "sendingCurrency": "USD",
  "receivingAmount": 12450.00,
  "receivingCurrency": "KES",
  "exchangeRate": 124.50,
  "totalFees": 2.50,
  "payoutMethod": "MobileWallet",
  "payerId": "MM_KE_MPESA",
  "expiresAt": "2026-02-24T12:30:00Z"
}
```

### Create Transaction

Initiate a money transfer:

```
POST /money-transfer/v2/transactions
```

**Request Body**:

```json
{
  "quoteId": "QT_1234567890",
  "senderName": "John Doe",
  "senderEmail": "john@example.com",
  "senderPhone": "+1234567890",
  "senderAddress": "123 Main St",
  "recipientName": "Jane Smith",
  "recipientPhone": "+254712345678",
  "recipientEmail": "jane@example.com",
  "payerId": "MM_KE_MPESA",
  "payerFields": {
    "mobileNumber": "254712345678"
  },
  "externalReference": "REF_12345",
  "callbackUrl": "https://yourapp.com/webhooks/thunes"
}
```

**Response**:

```json
{
  "transactionId": "TXN_1234567890",
  "status": "PENDING",
  "sendingAmount": 100.00,
  "receivingAmount": 12450.00,
  "exchangeRate": 124.50,
  "fees": 2.50,
  "createdAt": "2026-02-24T10:15:00Z",
  "externalReference": "REF_12345"
}
```

### Get Transaction Status

Retrieve transaction status (alternative to webhooks):

```
GET /money-transfer/v2/transactions/{transactionId}
```

**Response**:

```json
{
  "transactionId": "TXN_1234567890",
  "status": "COMPLETED",
  "sendingAmount": 100.00,
  "receivingAmount": 12450.00,
  "createdAt": "2026-02-24T10:15:00Z",
  "completedAt": "2026-02-24T10:18:30Z",
  "externalReference": "REF_12345",
  "payoutReference": "PAYOUT_KE_12345"
}
```

### Cancel Transaction

Cancel a pending transaction:

```
POST /money-transfer/v2/transactions/{transactionId}/cancel
```

Works only if transaction status is "PENDING". Once completed or failed, cancellation is not possible.

## Webhooks and Callbacks

Thunes sends real-time notifications to your callback URL whenever transaction status changes.

### Webhook Implementation

Provide a `callbackUrl` when creating a transaction. Thunes will POST transaction updates to this endpoint:

```
POST https://yourapp.com/webhooks/thunes
```

### Webhook Payload

```json
{
  "transactionId": "TXN_1234567890",
  "status": "COMPLETED",
  "sendingAmount": 100.00,
  "receivingAmount": 12450.00,
  "exchangeRate": 124.50,
  "fees": 2.50,
  "createdAt": "2026-02-24T10:15:00Z",
  "completedAt": "2026-02-24T10:18:30Z",
  "externalReference": "REF_12345",
  "payoutReference": "PAYOUT_KE_12345"
}
```

### Webhook Response Requirements

Your endpoint must:
- Accept HTTP POST requests
- Return HTTP 2XX status code (200, 201, 202, etc.) within the timeout
- Process the payload and persist it to your database
- Not depend on request ordering (webhooks may arrive out of order)

### Webhook Retry Logic

If your endpoint doesn't return HTTP 2XX:
- Thunes retries multiple times
- Implement idempotent processing (use `transactionId` as unique key)
- After retries are exhausted, poll the transaction status API

### Node.js Webhook Handler Example

```javascript
const express = require('express');
const app = express();

app.post('/webhooks/thunes', express.json(), async (req, res) => {
  try {
    const { transactionId, status, externalReference } = req.body;

    // Idempotent: Check if already processed
    const existing = await db.webhooks.findOne({ transactionId });
    if (existing) {
      return res.status(200).json({ message: 'Already processed' });
    }

    // Process webhook
    console.log(`Transaction ${transactionId} status: ${status}`);

    // Update your database
    await db.transactions.updateOne(
      { externalReference },
      {
        status: status,
        thunes_id: transactionId,
        updated_at: new Date()
      }
    );

    // Save webhook log
    await db.webhooks.insertOne({
      transactionId,
      status,
      receivedAt: new Date(),
      payload: req.body
    });

    // Always return 2XX to acknowledge receipt
    res.status(200).json({ status: 'received' });
  } catch (error) {
    console.error('Webhook error:', error);
    res.status(200).json({ status: 'received', error: error.message });
  }
});

app.listen(3000);
```

## Common Integration Patterns

### Pattern 1: Simple Remittance Flow

A user sends money from the US to Kenya:

```
1. User enters: $100 USD to Jane's M-Pesa (254712345678)
2. GET /countries/US/services/BankDebit/payers → list available US payment methods
3. GET /countries/KE/services/MobileWallet/payers → find MM_KE_MPESA payer
4. POST /quotes → get rate and fees ($100 USD = 12,450 KES)
5. Display to user: "Jane will receive 12,450 KES (fee: $2.50)"
6. User confirms and funds the transaction
7. POST /transactions → create transaction with callbackUrl
8. Webhook arrives when completed → update UI
```

### Pattern 2: Intra-Africa Bank Transfer

Sending from Nigeria to Ghana bank account:

```json
{
  "sendingCountry": "NG",
  "receivingCountry": "GH",
  "sendingCurrency": "NGN",
  "receivingCurrency": "GHS",
  "payoutMethod": "BankAccount",
  "payerId": "BA_GH_BANK",
  "amount": 50000.00,
  "senderName": "Kofi Mensah",
  "recipientName": "Ama Osei",
  "payerFields": {
    "accountNumber": "1234567890",
    "bankCode": "GHS_GTBANK"
  }
}
```

### Pattern 3: Cash Pickup

Enabling cash pickup at physical locations:

```json
{
  "receivingCountry": "UG",
  "payoutMethod": "CashPickup",
  "payerId": "CP_UG_MONEYGRAM",
  "amount": 500.00,
  "recipientName": "Akankwasa Jane",
  "payerFields": {
    "moneygram_mtcn": "AUTO_GENERATED"
  }
}
```

### Pattern 4: Polling for Status (No Webhooks)

If webhooks aren't available:

```javascript
async function pollTransactionStatus(transactionId) {
  const maxRetries = 30; // 5 minutes with 10-second intervals
  let retries = 0;

  while (retries < maxRetries) {
    const response = await fetch(
      `https://{API_ENDPOINT}/money-transfer/v2/transactions/${transactionId}`,
      {
        headers: { 'Authorization': `Basic ${credentials}` }
      }
    );

    const transaction = await response.json();

    if (['COMPLETED', 'FAILED', 'CANCELLED'].includes(transaction.status)) {
      return transaction;
    }

    retries++;
    await new Promise(resolve => setTimeout(resolve, 10000)); // Wait 10s
  }

  throw new Error('Transaction status check timeout');
}
```

## Error Handling

### HTTP Status Codes

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | OK | Request successful |
| 400 | Bad Request | Validate request body (missing/invalid fields) |
| 401 | Unauthorized | Check API credentials |
| 403 | Forbidden | IP address not whitelisted or insufficient permissions |
| 404 | Not Found | Resource not found (e.g., transaction ID doesn't exist) |
| 409 | Conflict | Transaction already processed or state conflict |
| 422 | Unprocessable Entity | Validation error (e.g., amount exceeds limit) |
| 429 | Too Many Requests | Rate limit exceeded, implement exponential backoff |
| 500 | Server Error | Thunes backend issue, retry with backoff |
| 503 | Service Unavailable | Maintenance, retry later |

### Error Response Format

```json
{
  "errors": [
    {
      "code": "1000401",
      "message": "Unauthorized"
    },
    {
      "code": "1000422",
      "message": "Amount exceeds maximum limit for this corridor"
    }
  ]
}
```

### Common Error Codes

| Code | Scenario | Resolution |
|------|----------|-----------|
| `1000401` | Invalid API credentials | Verify API_KEY and API_SECRET |
| `1000403` | IP not whitelisted | Whitelist your server IP in Developer Module |
| `1000422` | Amount out of range | Check `limits` from payer endpoints |
| `1000422` | Missing required field | Ensure all `requiredFields` from payer are included |
| `1000404` | Quote/Transaction not found | Verify ID and environment (sandbox vs production) |
| `1000409` | Transaction already completed | Cannot modify completed transactions |
| `1010001` | Corridor not available | Service may be down, retry after delay |

### Retry Strategy

```javascript
async function apiCallWithRetry(url, options, maxRetries = 3) {
  let lastError;

  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);

      if (response.ok) return response;

      if ([429, 500, 503].includes(response.status)) {
        const delay = Math.pow(2, i) * 1000; // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }

      const error = await response.json();
      throw new Error(`API Error: ${error.errors[0]?.message}`);
    } catch (error) {
      lastError = error;
      if (i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError;
}
```

## Important Notes and Gotchas

### 1. Partnership Model: Not Open API

Thunes does not offer self-service public API access. You must establish a partnership agreement with Thunes before:
- Accessing production credentials
- Using the API at scale
- Accessing specific African corridors
- Using their branding and integrations

**Action Required**: Contact Thunes sales or partnerships team to discuss your use case. Initial setup includes a compliance review.

### 2. Endpoint URLs Not Publicly Listed

The base URL and specific endpoint paths are **provided upon account creation**. They are not publicly documented and vary by:
- Environment (sandbox vs production)
- Your account tier
- Network connectivity method (HTTPS vs VPN)

**Action Required**: Store the provided endpoint in environment variables, not in code.

### 3. Compliance and KYC Requirements

Thunes has strict compliance requirements:
- Account creation requires business verification
- Transaction limits depend on your compliance tier
- Certain countries have corridor-specific compliance rules
- Regular compliance reviews may affect access

**Action Required**: Plan for compliance timelines (can take weeks to months). Keep compliance documentation current.

### 4. Corridor-Specific Payout Methods

Not all payout methods are available in all corridors:
- M-Pesa available in Kenya, Tanzania, Uganda, DRC
- MTN Mobile Money available in Nigeria, Ghana, Cameroon, Ivory Coast
- Bank transfers have different requirements by country (bank codes, account formats)
- Cash pickup only available in selected countries

**Action Required**: Always check payer endpoints before assuming a method exists. Display only available methods to users.

### 5. Currency and Exchange Rate Precision

Exchange rates fluctuate and quotes expire:
- Quotes are valid for a limited time (typically minutes)
- Must create transaction before quote expiration
- Currency precision varies (some 0 decimals, others 2-3 decimals)
- FX spreads and fees are corridor-specific

**Action Required**: Store quote expiry time, refresh quotes regularly, handle partial fills or rejections.

### 6. Field Requirements Vary by Payer

Each payer has different required fields. Example:
- M-Pesa: `mobileNumber` only
- Bank Account: `accountNumber`, `bankCode`, `accountHolderName`
- Cash Pickup: `recipientName`, `agentLocation`

**Action Required**: Call discovery endpoints to get current `requiredFields` for each payer. Don't hardcode field lists.

### 7. Real-Time Processing Is Best-Effort

Thunes aims for instant processing but guarantees vary:
- Most corridors complete in seconds to minutes
- Some corridors may take 1-2 hours
- Night/weekend processing may be delayed
- Bank holidays affect timelines

**Action Required**: Set user expectations clearly. Implement status polling as fallback. Don't guarantee instant delivery in marketing.

### 8. IP Whitelisting Required for Security

The API requires whitelisting your server's IP address(es):
- Required for both sandbox and production
- Set up in Developer Module → Environments
- Changes require updating IP whitelist

**Action Required**: Identify all server IPs (including load balancers) and register them. Update when IPs change.

### 9. Webhook Delivery Guarantee Is Best-Effort

While webhooks are recommended:
- Not guaranteed to arrive exactly once
- May arrive out of order
- May be delayed by hours in edge cases
- Polling remains the authoritative status source

**Action Required**: Use `transactionId` as idempotency key. Implement idempotent webhook handling. Regularly poll for status of old transactions.

### 10. Rate Limits and Throttling

Thunes implements rate limiting:
- Limits vary by account tier
- Discovery endpoints have different limits than transaction endpoints
- Hitting limits returns HTTP 429

**Action Required**: Implement exponential backoff. Cache discovery endpoint responses (countries, services, payers) and refresh periodically, not on every request.

### 11. Payer Limits May Be Lower Than Corridor Limits

Each payer has min/max limits:
- Same corridor may have different limits for different payers
- Limits change based on compliance tier and region
- Amount validation is required before creating quotes

**Action Required**: Check payer `limits` from discovery. Display limits to users. Validate before API call.

### 12. Testing in Sandbox Before Production

Sandbox environment:
- Uses test credentials
- Has limited corridors (main Africa corridors available)
- Transactions complete quickly
- Test data is cleared periodically

**Action Required**: Thoroughly test sandbox integration before requesting production credentials. Test both success and failure scenarios.

## Testing Checklist

- [ ] List all countries, services, and payers in sandbox
- [ ] Get a quote for a popular corridor (e.g., USD to KES)
- [ ] Create a transaction with sandbox credentials
- [ ] Implement webhook handler and test payload delivery
- [ ] Test error cases: invalid amounts, missing fields, invalid payer
- [ ] Test transaction cancellation on pending transactions
- [ ] Verify polling and webhook both work
- [ ] Test retry logic with rate limit simulation
- [ ] Validate receipt of different transaction statuses (PENDING, COMPLETED, FAILED)

## Useful Links

- **Official Documentation**: https://docs.thunes.com/
- **Money Transfer API v2**: https://docs.thunes.com/money-transfer/v2/
- **Money Transfer API v1**: https://docs.thunes.com/money-transfer/v1/
- **Developer Module Guide**: https://thunes.zendesk.com/hc/en-us/articles/17271405592477-Guide-to-Developer-Module
- **Accept API (Payments)**: https://docs.thunes.com/accept/v1/
- **Thunes Homepage**: https://www.thunes.com/
- **Cross-Border Payments Info**: https://www.thunes.com/cross-border-payments/
- **Sample Integration (GitHub)**: https://github.com/fghpdf/thunes_sample
- **API Tracker**: https://apitracker.io/a/thunes

---

**Last Updated**: February 2026
**API Version**: Money Transfer API v2
**Status**: Production Ready (Partnership Required)
