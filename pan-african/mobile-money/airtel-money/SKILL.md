---
triggers:
  - "Airtel Money"
  - "Airtel Africa API"
  - "Airtel mobile money"
  - "Airtel Money collection"
  - "Airtel Money disbursement"
  - "Airtel Money API integration"
  - "Airtel remittance"
  - "Airtel payment gateway"
---

# Airtel Money API

Airtel Money is a pan-African mobile money API provided by Airtel Africa, enabling businesses to integrate digital payment collections and disbursements across 14+ African countries. The API uses OAuth 2.0 authentication and supports both sandbox and production environments.

## When to Use

- **Collections**: Accept payments from customers via Airtel Money wallets
- **Disbursements**: Send instant payments to recipients' Airtel Money wallets (refunds, payroll, commissions)
- **Pan-African Coverage**: Operate across multiple African markets with a single integration
- **Cross-border Remittances**: Enable money transfers across borders
- **Mobile Money Payments**: Accept payments from users without bank accounts
- **Developer Sandbox**: Test integrations in staging environment before going live

**Supported Countries**: Zambia, Malawi, Uganda, Tanzania, Kenya, Niger, Democratic Republic of Congo, Rwanda, Burkina Faso, Côte d'Ivoire, Gabon, and others across Africa

## Authentication

Airtel Money uses **OAuth 2.0 Client Credentials Grant** for server-to-server authentication.

### Getting Your Credentials

1. Register at [https://developers.airtel.africa/developer](https://developers.airtel.africa/developer)
2. Create an application in the developer portal
3. Add Collection APIs and Remittance APIs to your application
4. Retrieve your `client_id` and `client_secret`
5. For production access, submit KYC documents for approval

### OAuth 2.0 Token Request

**Endpoint**: `POST /auth/oauth2/token`

**Staging**: `https://openapiuat.airtel.africa/auth/oauth2/token`
**Production**: `https://openapi.airtel.africa/auth/oauth2/token`

**Request Body** (application/json):
```json
{
  "client_id": "your-client-id",
  "client_secret": "your-client-secret",
  "grant_type": "client_credentials"
}
```

**cURL Example**:
```bash
curl -X POST https://openapiuat.airtel.africa/auth/oauth2/token \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
    "client_id": "your-client-id",
    "client_secret": "your-client-secret",
    "grant_type": "client_credentials"
  }'
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### Using the Token

Include the access token in all subsequent API requests:
```bash
Authorization: Bearer {access_token}
```

### Required Headers (All Requests)

> ⚠️ **These headers are required on every API call** — missing them will result in rejected requests.

| Header | Description | Example |
|--------|-------------|---------|
| `Authorization` | Bearer token from OAuth flow | `Bearer eyJhbGci...` |
| `X-Country` | ISO 3166-1 alpha-2 country code | `KE`, `NG`, `ZM` |
| `X-Currency` | ISO 4217 currency code for that country | `KES`, `NGN`, `ZMW` |
| `Content-Type` | Always JSON | `application/json` |
| `Accept` | Always JSON | `application/json` |

**JavaScript Example** (Node.js with fetch):
```javascript
async function getAccessToken(clientId, clientSecret) {
  const response = await fetch('https://openapiuat.airtel.africa/auth/oauth2/token', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      client_id: clientId,
      client_secret: clientSecret,
      grant_type: 'client_credentials'
    })
  });

  const data = await response.json();
  return data.access_token;
}

// Use the token
const token = await getAccessToken(process.env.AIRTEL_CLIENT_ID, process.env.AIRTEL_CLIENT_SECRET);
const headers = {
  'Authorization': `Bearer ${token}`,
  'Content-Type': 'application/json'
};
```

**Token Caching Best Practice**: Cache tokens and reuse until expiration to reduce authentication overhead.

## Core API Reference

### 1. Collection API (Payment Request)

Initiate a payment request from a customer. The customer will be prompted to enter their Airtel Money PIN to authorize the transaction.

**Endpoint**: `POST /merchant/v2/payments/{country_code}`

**Request Body**:
```json
{
  "reference": "ORDER-12345",
  "subscriber": {
    "country": "KE",
    "currency": "KES",
    "msisdn": "254700000000"
  },
  "transaction": {
    "amount": "100",
    "country": "KE",
    "currency": "KES",
    "id": "TXN-12345",
    "type": "MerchantPayment"
  }
}
```

**Important:** Do **not** include a `pin` field in the request body. The customer's PIN is entered by the customer themselves in the Airtel Money app or USSD flow — it is never sent by the merchant. Including a merchant-provided PIN in the request is a security design error.
```

**Response (Success - 200)**:
```json
{
  "data": {
    "transaction": {
      "id": "TXN-12345",
      "status": "AwaitingCustomerPayment",
      "message": "Payment request initiated"
    }
  }
}
```

**Response (Pending - 202)**:
```json
{
  "data": {
    "transaction": {
      "id": "TXN-12345",
      "status": "Pending",
      "statusCode": "02",
      "message": "Request is pending"
    }
  }
}
```

### 2. Disbursement API (Money Transfer Out)

Send money directly to a recipient's Airtel Money wallet.

**Endpoint**: `POST /merchant/v2/payouts/`

**Request Body**:
```json
{
  "reference": "PAYOUT-67890",
  "subscriber": {
    "country": "KE",
    "currency": "KES",
    "msisdn": "254700000001"
  },
  "transaction": {
    "amount": "500",
    "country": "KE",
    "currency": "KES",
    "id": "PAYOUT-67890",
    "type": "MerchantPayment"
  }
}
```

**Response**:
```json
{
  "data": {
    "transaction": {
      "id": "PAYOUT-67890",
      "status": "Successful",
      "statusCode": "00",
      "message": "Payout processed successfully",
      "transactionId": "AIRTEL-67890"
    }
  }
}
```

### 3. Account Balance Inquiry

Check the available balance in your Airtel Money merchant wallet.

**Endpoint**: `GET /merchant/v2/wallets/balance/`

**Request Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Response**:
```json
{
  "data": {
    "wallet": {
      "msisdn": "your-wallet-number",
      "balance": "50000.00",
      "currency": "KES",
      "status": "Active"
    }
  }
}
```

### 4. KYC (Know Your Customer) API

Verify customer information and complete KYC requirements for transactions.

**Endpoint**: `POST /merchant/v2/kyc/`

**Request Body**:
```json
{
  "subscriber": {
    "country": "KE",
    "currency": "KES",
    "msisdn": "254700000000"
  },
  "identification": {
    "type": "NationalID",
    "number": "ID123456"
  }
}
```

**Response**:
```json
{
  "data": {
    "kyc": {
      "status": "Verified",
      "msisdn": "254700000000",
      "kycLevel": 2,
      "maxDaily": "1000000",
      "maxTransaction": "500000",
      "currency": "KES"
    }
  }
}
```

### 5. Transaction Query/Status

Query the status of a previously initiated transaction.

**Endpoint**: `GET /merchant/v2/transactions/{transactionId}`

**Response**:
```json
{
  "data": {
    "transaction": {
      "id": "TXN-12345",
      "reference": "ORDER-12345",
      "status": "Successful",
      "statusCode": "00",
      "amount": "100.00",
      "currency": "KES",
      "msisdn": "254700000000",
      "timestamp": "2026-02-24T10:30:00Z"
    }
  }
}
```

## Callbacks (Webhooks)

Airtel Money sends real-time notifications to your callback URL when transaction status changes. Configure your callback URL in the developer portal.

### Callback Payload Structure

**Collection Callback**:
```json
{
  "phone": "254700000000",
  "reference": "ORDER-12345",
  "transactionId": "TXN-12345",
  "amount": "100",
  "currency": "KES",
  "status": "SUCCESS",
  "statusCode": "00",
  "narrative": "Payment for goods",
  "timestamp": "2026-02-24T10:30:00Z"
}
```

**Disbursement Callback**:
```json
{
  "phone": "254700000001",
  "reference": "PAYOUT-67890",
  "transactionId": "AIRTEL-67890",
  "amount": "500",
  "currency": "KES",
  "status": "SUCCESS",
  "statusCode": "00",
  "narrative": "Payout processed",
  "timestamp": "2026-02-24T10:30:00Z"
}
```

### Callback Best Practices

1. **Verify Signature**: Validate callback authenticity using provided signature headers (HMAC)
2. **Idempotency**: Handle duplicate callbacks by checking transaction reference
3. **Timeout**: Respond with HTTP 200 within 10-30 seconds
4. **Retry Logic**: Airtel may retry failed callbacks; implement idempotent processing
5. **Logging**: Log all callbacks for reconciliation and debugging

**Node.js Example - Callback Handler**:
```javascript
const express = require('express');
const app = express();

app.post('/webhook/airtel-callback', express.json(), (req, res) => {
  const callback = req.body;

  // Verify signature (implementation depends on Airtel's signature method)
  if (!verifySignature(callback, req.headers['x-signature'])) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Check if transaction already processed (idempotency)
  if (isTransactionProcessed(callback.reference)) {
    return res.status(200).json({ status: 'already_processed' });
  }

  // Process callback
  processTransaction(callback);

  // Store in database
  logCallbackTransaction(callback);

  // Respond immediately
  res.status(200).json({ status: 'received' });
});

function verifySignature(payload, signature) {
  // Implement HMAC verification using your secret key
  const crypto = require('crypto');
  const hash = crypto
    .createHmac('sha256', process.env.AIRTEL_SECRET)
    .update(JSON.stringify(payload))
    .digest('hex');
  return hash === signature;
}
```

## Common Integration Patterns

### Pattern 1: Simple Payment Collection

```javascript
async function collectPayment(msisdn, amount, reference) {
  const token = await getAccessToken();

  const response = await fetch('https://openapi.airtel.africa/merchant/v2/payments/${countryCode}', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      reference,
      subscriber: {
        country: 'KE',
        currency: 'KES',
        msisdn
      },
      transaction: {
        amount: amount.toString(),
        country: 'KE',
        currency: 'KES',
        id: reference,
        type: 'MerchantPayment'
      },
      // Note: do NOT include a pin field — customer enters PIN in their Airtel Money app/USSD
    })
  });

  return await response.json();
}

// Usage
const result = await collectPayment('254700000000', 100, 'ORDER-001');
```

### Pattern 2: Bulk Disbursement

```javascript
async function bulkDisburse(recipients) {
  const token = await getAccessToken();
  const results = [];

  for (const recipient of recipients) {
    try {
      const response = await fetch('https://openapi.airtel.africa/merchant/v2/payouts/', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          reference: recipient.reference,
          subscriber: {
            country: 'KE',
            currency: 'KES',
            msisdn: recipient.msisdn
          },
          transaction: {
            amount: recipient.amount.toString(),
            country: 'KE',
            currency: 'KES',
            id: recipient.reference,
            type: 'MerchantPayment'
          }
        })
      });

      results.push({
        reference: recipient.reference,
        status: response.status,
        data: await response.json()
      });

      // Add delay to respect rate limits
      await new Promise(resolve => setTimeout(resolve, 500));
    } catch (error) {
      results.push({
        reference: recipient.reference,
        error: error.message
      });
    }
  }

  return results;
}
```

### Pattern 3: Transaction Status Polling

```javascript
async function pollTransactionStatus(transactionId, maxAttempts = 10) {
  const token = await getAccessToken();
  let attempts = 0;

  while (attempts < maxAttempts) {
    try {
      const response = await fetch(
        `https://openapi.airtel.africa/merchant/v2/transactions/${transactionId}`,
        {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
          }
        }
      );

      const data = await response.json();
      const status = data.data.transaction.status;

      if (status === 'Successful' || status === 'Failed') {
        return data;
      }

      attempts++;
      await new Promise(resolve => setTimeout(resolve, 3000)); // Wait 3 seconds
    } catch (error) {
      console.error('Poll error:', error);
      attempts++;
    }
  }

  throw new Error('Transaction status unknown after polling');
}
```

### Pattern 4: Token Caching with Expiration

```javascript
class AirtelMoneyClient {
  constructor(clientId, clientSecret) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.token = null;
    this.tokenExpiry = null;
  }

  async getToken() {
    // Return cached token if still valid
    if (this.token && this.tokenExpiry > Date.now()) {
      return this.token;
    }

    const response = await fetch('https://openapi.airtel.africa/auth/oauth2/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        client_id: this.clientId,
        client_secret: this.clientSecret,
        grant_type: 'client_credentials'
      })
    });

    const data = await response.json();
    this.token = data.access_token;
    // Cache for 55 minutes (token expires in 60 minutes)
    this.tokenExpiry = Date.now() + (data.expires_in * 1000) - 300000;
    return this.token;
  }

  async makeRequest(endpoint, method = 'POST', body = null) {
    const token = await this.getToken();
    const options = {
      method,
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    };

    if (body) {
      options.body = JSON.stringify(body);
    }

    return fetch(`https://openapi.airtel.africa${endpoint}`, options);
  }
}

// Usage
const client = new AirtelMoneyClient(process.env.CLIENT_ID, process.env.CLIENT_SECRET);
const response = await client.makeRequest('/merchant/v2/payments/', 'POST', paymentPayload);
```

## Error Handling

### Common Response Codes

| Code | Status | Meaning | Action |
|------|--------|---------|--------|
| 00 | SUCCESS | Transaction successful | Proceed |
| 01 | INVALID_REQUEST | Malformed request | Review request format |
| 02 | PENDING | Transaction pending | Poll status or wait for callback |
| 03 | FAILED | Transaction failed | Check error details, retry if applicable |
| 04 | SYSTEM_ERROR | System error at provider | Retry with exponential backoff |
| 05 | NOT_FOUND | Transaction/reference not found | Verify reference ID |
| 06 | DUPLICATE | Duplicate request | Check existing transaction status |
| 09 | INSUFFICIENT_FUNDS | Insufficient balance | Top up merchant account |
| 429 | TOO_MANY_REQUESTS | Rate limit exceeded | Implement exponential backoff |

### HTTP Status Codes

- **200 OK**: Successful request
- **202 Accepted**: Request accepted but pending (most common for collections)
- **400 Bad Request**: Invalid request format
- **401 Unauthorized**: Invalid/expired token
- **403 Forbidden**: No permission for operation
- **404 Not Found**: Resource not found
- **429 Too Many Requests**: Rate limit exceeded
- **500 Internal Server Error**: Server error
- **503 Service Unavailable**: Service temporarily unavailable

### Retry Strategy

```javascript
async function makeRequestWithRetry(fn, maxRetries = 3) {
  let lastError;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fn();

      // Don't retry on 4xx errors (except 429)
      if (response.status >= 400 && response.status < 500 && response.status !== 429) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      if (response.ok || response.status === 202) {
        return response;
      }
    } catch (error) {
      lastError = error;

      // Exponential backoff: 1s, 2s, 4s
      const delay = Math.pow(2, attempt) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}

// Usage
const response = await makeRequestWithRetry(async () => {
  const token = await getAccessToken();
  return fetch('https://openapi.airtel.africa/merchant/v2/payments/${countryCode}', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${token}` },
    body: JSON.stringify(payload)
  });
});
```

## Important Notes / Gotchas

### 1. **Country and Currency Code Format**
Ensure you use correct 2-letter ISO country codes (KE, TZ, UG, ZM) and 3-letter currency codes (KES, TZS, UGX, ZMW). Mismatches will cause transaction failures.

### 2. **MSISDN Phone Number Format**
Phone numbers must include country code without the '+' sign. Example: 254700000000 for Kenya, not +254700000000 or 0700000000. Validate format before submitting.

### 3. **Transaction Reference Uniqueness**
Transaction references must be unique. Duplicate references within a short time window may be treated as retries. Implement proper idempotency keys to handle accidental duplicates.

### 4. **PIN Requirement Varies by Configuration**
The PIN field may be optional or required depending on your merchant configuration. Test in sandbox to verify your specific setup. Some countries/merchants don't require customer PIN for collections.

### 5. **Sandbox vs Production Authentication**
Use `openapiuat.airtel.africa` for testing and `openapi.airtel.africa` for production. Credentials are environment-specific. Test thoroughly before switching environments.

### 6. **KYC Limits on Transactions**
Each customer account has KYC level limits determining maximum transaction amounts and daily ceilings. Always query KYC status before large transactions to avoid failures.

### 7. **Callback Delivery is Not Guaranteed**
Always poll transaction status for critical operations. Callbacks provide real-time updates but may be delayed or lost. Implement polling with exponential backoff as fallback.

### 8. **Token Expiration Handling**
Access tokens expire (typically after 60 minutes). Always cache and refresh tokens before expiry. A fresh token request on 401 response can prevent cascading failures.

### 9. **Rate Limiting**
Airtel Money APIs have rate limits. Implement request queuing, exponential backoff, and respect Retry-After headers. Bulk operations should include delays (500ms-1s) between requests.

### 10. **Error Response Format Inconsistency**
Error response structures may vary slightly across endpoints and countries. Parse errors defensively; check for status in multiple locations (statusCode, status, code fields).

### 11. **Cross-Border Transaction Requirements**
Cross-border disbursements may require additional KYC verification. Verify recipient country limits and compliance requirements before processing international transactions.

### 12. **Currency Conversion and Rates**
The API doesn't automatically convert between currencies. Ensure both subscriber and transaction currencies match. Exchange rates are not provided by the API; source rates externally.

### 13. **Wallet Balance Not Real-Time**
Balance inquiry responses may be cached. Don't rely on them for real-time checks immediately after transactions. Allow a short delay before subsequent balance checks.

### 14. **Production Onboarding Timeline**
Production access requires KYC document submission and manual approval by Airtel. Allow 5-10 business days for approval. Start the process early in your development cycle.

### 15. **Testing Phone Numbers**
Use specific test phone numbers in sandbox. Standard production numbers may not work in UAT. Check the developer portal for designated test numbers per country.

## Useful Links

- **Official Developer Portal**: [https://developers.airtel.africa/](https://developers.airtel.africa/)
- **API Documentation**: [https://developers.airtel.africa/docs](https://developers.airtel.africa/docs)
- **Sandbox Environment**: [https://openapiuat.airtel.africa](https://openapiuat.airtel.africa)
- **Production Environment**: [https://openapi.airtel.africa](https://openapi.airtel.africa)
- **Medium Integration Guide**: [How to integrate Airtel Money API for payment collections and remittances](https://medium.com/@muhanzi/how-to-integrate-airtel-money-api-for-payment-collections-and-remittances-67d2202cd488)
- **PHP SDK**: [https://github.com/osenco/airtel](https://github.com/osenco/airtel)
- **Node.js SDK**: [https://github.com/Chizzoz/airtel-money-expressjs](https://github.com/Chizzoz/airtel-money-expressjs)
- **ClickPesa Integration Guide**: [Airtel Money API Integration Guide](https://clickpesa.com/payment-gateway/payment-and-payout-methods/airtel-money-api-integration-guide/)
- **Airtel Money & UNCDF Initiative**: [Airtel Money and UNCDF launch Open APIs](https://www.uncdf.org/article/8241/airtel-money-and-uncdf-launch-open-apis-to-support-innovators-accelerate-digital-financial-inclusion)

---

**Last Updated**: February 2026
**Status**: Production Ready
**Supported Regions**: 14+ African Countries
