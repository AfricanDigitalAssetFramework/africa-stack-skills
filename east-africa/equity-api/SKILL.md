---
name: equity-jenga-api
description: Integrate with Equity Bank's Jenga API for payments, transfers, account queries, and financial transactions across East Africa. Use this skill for remittances, balance inquiries, transaction queries, PesaLink transfers, RTGS payments, and multi-country operations in Kenya, Uganda, Tanzania, Rwanda, DRC, and South Sudan.
---

# Equity Bank Jenga API Integration Skill

Equity Bank's Jenga API is a comprehensive fintech platform providing access to banking and payment services across East Africa. It enables developers to build payment applications, fintech solutions, and banking integrations with support for remittances, account management, real-time payments, and transaction history queries.

## When to use this skill

You're building a fintech application, payment platform, mobile wallet, accounting tool, or banking solution that needs to integrate with **Equity Bank Group** — East Africa's largest bank by asset base. Use Jenga API when you need to:

- Send domestic or international remittances
- Check account balances and retrieve transaction history
- Initiate payments via PesaLink, RTGS, or within Equity accounts
- Process inter-subsidiary transfers across six countries
- Query transaction details and payment status
- Build multi-country financial operations

Jenga API bridges Equity Bank services with modern applications across Kenya, Uganda, Tanzania, Rwanda, Democratic Republic of Congo, and South Sudan.

## Authentication

Jenga API uses Bearer token authentication with merchant credentials. Tokens are obtained via the merchant authentication endpoint and must be refreshed before expiry.

### Get Access Token

```
POST /authentication/api/v3/authenticate/merchant
Content-Type: application/json
Api-Key: {your_api_key}
```

**Body:**
```json
{
  "merchantCode": "your_merchant_code",
  "consumerSecret": "your_consumer_secret"
}
```

**Response:**
```json
{
  "accessToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "refreshToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "expiresIn": 3600,
  "issuedAt": "2025-02-24T10:00:00Z",
  "tokenType": "Bearer"
}
```

**Token Details:**
- `accessToken`: JWT token for API requests (expires in 1 hour typically)
- `refreshToken`: Token to refresh access without re-authenticating
- `expiresIn`: Token lifetime in seconds
- `tokenType`: Always "Bearer"

Cache the token and refresh before expiry. Implement 401 error handling to detect expired tokens.

### Base URLs

**UAT/Sandbox:**
```
https://uat.finserve.africa
```

**Production/Live:**
```
https://api.finserve.africa
```

All endpoints append to the base URL. Use sandbox credentials for development and testing.

### Required Headers

All API requests require these headers:

```
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

Store credentials in environment variables. Never hardcode merchant code, consumer secret, or API keys.

## Core API Reference

### Account Balance

Retrieve the current balance of an Equity Bank account.

```
GET /v3-apis/account-api/v3.0/accounts/{accountNumber}/balance
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Path Parameters:**
- `accountNumber`: The account number to query (e.g., "0640000000000001")

**Response:**
```json
{
  "data": {
    "accountNumber": "0640000000000001",
    "accountName": "John Doe",
    "currencyCode": "KES",
    "accountBalance": 50000.00,
    "availableBalance": 45000.00,
    "accountStatus": "ACTIVE",
    "ledgerBalance": 52000.00
  },
  "status": "success"
}
```

**Fields:**
- `accountBalance`: Total balance including pending transactions
- `availableBalance`: Usable balance (excludes holds)
- `ledgerBalance`: Ledger balance at the bank
- `accountStatus`: ACTIVE, DORMANT, CLOSED, etc.

### Account Details

Retrieve complete account information.

```
GET /v3-apis/account-api/v3.0/accounts/{accountNumber}
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Response:**
```json
{
  "data": {
    "accountNumber": "0640000000000001",
    "accountName": "John Doe",
    "accountType": "SAVINGS",
    "currencyCode": "KES",
    "accountStatus": "ACTIVE",
    "branch": "Nairobi Main Branch",
    "openedDate": "2020-01-15T00:00:00Z",
    "phoneNumber": "+254722000000",
    "email": "john.doe@example.com"
  },
  "status": "success"
}
```

### Query Transaction History

Retrieve transactions for an account within a date range.

```
GET /v3-apis/account-api/v3.0/accounts/{accountNumber}/transactions
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Query Parameters:**
- `startDate`: ISO 8601 format (e.g., "2025-01-01T00:00:00Z")
- `endDate`: ISO 8601 format (e.g., "2025-02-24T23:59:59Z")
- `pageNumber`: Page number (default: 1)
- `pageSize`: Records per page (default: 20, max: 100)

**Response:**
```json
{
  "data": [
    {
      "transactionId": "TXN_123456789",
      "transactionDate": "2025-02-24T10:31:00Z",
      "transactionType": "DEBIT",
      "transactionAmount": 5000.00,
      "transactionCurrency": "KES",
      "transactionNarration": "Remittance to John Smith",
      "transactionReference": "TXN-12345",
      "transactionStatus": "COMPLETED",
      "balanceAfterTransaction": 45000.00,
      "counterpartyName": "John Smith",
      "counterpartyAccountNumber": "0640000000000002"
    }
  ],
  "pageNumber": 1,
  "pageSize": 20,
  "totalPages": 5,
  "totalRecords": 95,
  "status": "success"
}
```

### Query Transaction Details

Get detailed information about a specific transaction.

```
GET /v3-apis/transaction-api/v3.0/transactions/{transactionReference}
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Response:**
```json
{
  "data": {
    "transactionReference": "TXN-12345",
    "transactionId": "TXN_123456789",
    "transactionDate": "2025-02-24T10:31:00Z",
    "transactionType": "DEBIT",
    "transactionStatus": "SUCCESS",
    "amount": 5000.00,
    "currency": "KES",
    "description": "Remittance to John Smith",
    "sourceAccount": "0640000000000001",
    "destinationAccount": "0640000000000002",
    "destinationName": "John Smith",
    "paymentMethod": "EFT",
    "completionDate": "2025-02-24T10:32:00Z"
  },
  "status": "success"
}
```

**Status Values:** SUCCESS, PENDING, FAILED

### Within Equity Transfer (EFT)

Send money between Equity Bank accounts (domestic within same country).

```
POST /v3-apis/transaction-api/v3.0/remittance/intrabank
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Body:**
```json
{
  "source": {
    "accountNumber": "0640000000000001"
  },
  "destination": {
    "name": "John Smith",
    "accountNumber": "0640000000000002",
    "countryCode": "KE"
  },
  "transfer": {
    "type": "EFT",
    "amount": 5000.00,
    "currencyCode": "KES",
    "reference": "TXN-12345",
    "date": "2025-02-24T00:00:00Z",
    "description": "Payment for invoice #12345"
  }
}
```

**Response:**
```json
{
  "data": {
    "transactionId": "TXN_123456789",
    "transactionReference": "TXN-12345",
    "transactionStatus": "PROCESSING",
    "amount": 5000.00,
    "currency": "KES",
    "completionEstimate": "2025-02-24T10:35:00Z"
  },
  "status": "success"
}
```

Processing time: Usually 1-5 minutes for intra-bank transfers.

### Inter-Subsidiary Transfer

Send money between Equity Bank accounts in different countries.

```
POST /v3-apis/transaction-api/v3.0/remittance/intersubsidiary
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Body:**
```json
{
  "source": {
    "accountNumber": "0640000000000001",
    "countryCode": "KE"
  },
  "destination": {
    "name": "Jane Kamau",
    "accountNumber": "3010012345678",
    "countryCode": "UG"
  },
  "transfer": {
    "type": "EFT",
    "amount": 10000.00,
    "currencyCode": "KES",
    "reference": "INT-12345",
    "date": "2025-02-24T00:00:00Z",
    "description": "Cross-border remittance"
  }
}
```

**Response:**
```json
{
  "data": {
    "transactionId": "TXN_987654321",
    "transactionReference": "INT-12345",
    "transactionStatus": "PROCESSING",
    "amount": 10000.00,
    "sourceCurrency": "KES",
    "destinationCurrency": "UGX",
    "exchangeRate": 27.45,
    "destinationAmount": 274500.00,
    "completionEstimate": "2025-02-25T12:00:00Z"
  },
  "status": "success"
}
```

**Supported Country Pairs:**
- Kenya (KE) ↔ Uganda (UG)
- Kenya (KE) ↔ Tanzania (TZ)
- Kenya (KE) ↔ Rwanda (RW)
- Kenya (KE) ↔ DRC (CD)
- Kenya (KE) ↔ South Sudan (SS)
- Cross-country transfers with automatic forex conversion

Processing time: 1-2 business days.

### PesaLink Bank Account Transfer

Send money via PesaLink to any participating bank account (Kenya only).

```
POST /v3-apis/transaction-api/v3.0/remittance/pesalinkacc
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Body:**
```json
{
  "source": {
    "accountNumber": "0640000000000001"
  },
  "destination": {
    "name": "Alice Johnson",
    "accountNumber": "1234567890",
    "bankCode": "01"
  },
  "transfer": {
    "type": "PESALINK",
    "amount": 2500.00,
    "currencyCode": "KES",
    "reference": "PESA-12345",
    "date": "2025-02-24T00:00:00Z",
    "description": "PesaLink transfer to bank account"
  }
}
```

**Response:**
```json
{
  "data": {
    "transactionId": "TXN_456789123",
    "transactionReference": "PESA-12345",
    "transactionStatus": "PROCESSING",
    "amount": 2500.00,
    "currency": "KES",
    "destinationBank": "Kenya Commercial Bank",
    "completionEstimate": "2025-02-24T11:00:00Z"
  },
  "status": "success"
}
```

**Supported Banks:** All KBA (Kenya Bankers Association) member banks plus mobile money providers.

Processing time: Usually within 1-2 hours.

### PesaLink Mobile Transfer

Send money via PesaLink to mobile wallets (Kenya only).

```
POST /v3-apis/transaction-api/v3.0/remittance/pesalinkmobile
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Body:**
```json
{
  "source": {
    "accountNumber": "0640000000000001"
  },
  "destination": {
    "name": "Mark Kipchoge",
    "mobileNumber": "+254722123456",
    "mobileProvider": "SAFARICOM"
  },
  "transfer": {
    "type": "PESALINK_MOBILE",
    "amount": 1500.00,
    "currencyCode": "KES",
    "reference": "PESAM-12345",
    "date": "2025-02-24T00:00:00Z",
    "description": "Mobile money transfer"
  }
}
```

**Response:**
```json
{
  "data": {
    "transactionId": "TXN_654321789",
    "transactionReference": "PESAM-12345",
    "transactionStatus": "PROCESSING",
    "amount": 1500.00,
    "currency": "KES",
    "recipientMobile": "+254722123456",
    "recipientProvider": "SAFARICOM",
    "completionEstimate": "2025-02-24T10:45:00Z"
  },
  "status": "success"
}
```

**Supported Providers:** Safaricom, Airtel, Telecom Plus, Equity Bank Direct.

Processing time: Instant to 5 minutes.

### RTGS Payment

Send money via RTGS (Real-Time Gross Settlement) for high-value transfers.

```
POST /v3-apis/transaction-api/v3.0/remittance/rtgs
Authorization: Bearer {accessToken}
Content-Type: application/json
Api-Key: {your_api_key}
```

**Body:**
```json
{
  "source": {
    "accountNumber": "0640000000000001"
  },
  "destination": {
    "name": "Corporate Recipient",
    "accountNumber": "9876543210",
    "bankCode": "03",
    "swiftCode": "EQBLKENA"
  },
  "transfer": {
    "type": "RTGS",
    "amount": 500000.00,
    "currencyCode": "KES",
    "reference": "RTGS-12345",
    "date": "2025-02-24T00:00:00Z",
    "description": "RTGS corporate payment"
  }
}
```

**Response:**
```json
{
  "data": {
    "transactionId": "TXN_789456123",
    "transactionReference": "RTGS-12345",
    "transactionStatus": "PROCESSING",
    "amount": 500000.00,
    "currency": "KES",
    "processingTime": "Real-time (5-10 minutes)",
    "destinationBank": "Cooperative Bank of Kenya",
    "completionEstimate": "2025-02-24T10:40:00Z"
  },
  "status": "success"
}
```

**Requirements:**
- Minimum amount: Usually KES 40,000 or currency equivalent
- Operating hours: Monday-Friday, 7:00 AM - 6:00 PM EAT
- Charges apply (consult Equity for current rates)

Processing time: 5-30 minutes during business hours.

## Webhooks

Equity Bank can send webhooks for transaction status updates. Configure your webhook URL in the Jenga merchant dashboard.

### Webhook URL Configuration

1. Login to Jenga dashboard (https://jengahq.io)
2. Navigate to Settings → API Integration
3. Enter your webhook URL (must be HTTPS)
4. Specify which events to receive (transaction.completed, transaction.failed, etc.)

### Webhook Headers

All webhooks include authentication headers:

```
X-Signature: {HMAC-SHA256 signature}
X-Timestamp: 2025-02-24T10:31:00Z
Content-Type: application/json
```

### Verify Webhook Signature

```javascript
const crypto = require('crypto');
const webhookSecret = 'your_webhook_secret';

function verifyWebhook(payload, signature) {
  const hash = crypto
    .createHmac('sha256', webhookSecret)
    .update(JSON.stringify(payload))
    .digest('hex');
  return hash === signature;
}
```

### Transaction Completed Webhook

```json
{
  "event": "transaction.completed",
  "data": {
    "transactionId": "TXN_123456789",
    "transactionReference": "TXN-12345",
    "transactionStatus": "SUCCESS",
    "amount": 5000.00,
    "currency": "KES",
    "sourceAccount": "0640000000000001",
    "destinationAccount": "0640000000000002",
    "transactionDate": "2025-02-24T10:31:00Z",
    "completionDate": "2025-02-24T10:32:00Z",
    "description": "Remittance"
  },
  "timestamp": "2025-02-24T10:32:01Z"
}
```

### Transaction Failed Webhook

```json
{
  "event": "transaction.failed",
  "data": {
    "transactionId": "TXN_123456789",
    "transactionReference": "TXN-12345",
    "transactionStatus": "FAILED",
    "amount": 5000.00,
    "currency": "KES",
    "sourceAccount": "0640000000000001",
    "destinationAccount": "0640000000000002",
    "failureReason": "Insufficient funds",
    "failureCode": "INSUFFICIENT_FUNDS",
    "transactionDate": "2025-02-24T10:31:00Z",
    "failureDate": "2025-02-24T10:31:15Z"
  },
  "timestamp": "2025-02-24T10:31:16Z"
}
```

**Important:** Always verify transaction status by calling the API. Webhooks are informational only; API queries are the source of truth.

## Common Integration Patterns

### Payment Processing Flow

1. **Authenticate**: `POST /authentication/api/v3/authenticate/merchant` → Obtain Bearer token
2. **Check Balance**: `GET /v3-apis/account-api/v3.0/accounts/{accountNumber}/balance` → Verify sufficient funds
3. **Initiate Transfer**: `POST /v3-apis/transaction-api/v3.0/remittance/{type}` → Send money (intrabank, intersubsidiary, pesalink, rtgs)
4. **Verify Status**: `GET /v3-apis/transaction-api/v3.0/transactions/{reference}` → Confirm completion
5. **Log Transaction**: Store transaction ID and status in your database
6. **Handle Webhook**: Receive optional webhook notification

### Account Reconciliation

1. **Authenticate**: `POST /authentication/api/v3/authenticate/merchant`
2. **Fetch Transactions**: `GET /v3-apis/account-api/v3.0/accounts/{accountNumber}/transactions?startDate={date1}&endDate={date2}`
3. **Compare Records**: Match API transactions against internal ledger
4. **Flag Discrepancies**: Identify missing or mismatched transactions
5. **Investigate**: Use `GET /v3-apis/transaction-api/v3.0/transactions/{reference}` for disputed transactions
6. **Update System**: Sync balances and transaction status

### Balance Monitoring

1. **Cache Balance**: Store last-known balance and timestamp
2. **Periodic Check**: Call `GET /v3-apis/account-api/v3.0/accounts/{accountNumber}/balance` every 5-15 minutes
3. **Alert on Changes**: Notify user of significant balance movements
4. **Update Dashboard**: Display real-time balance on your UI
5. **Log History**: Track balance over time for analytics

### Multi-Country Operations

1. **Identify Destination**: Determine recipient's country code (KE, UG, TZ, RW, CD, SS)
2. **Same Country**: Use intrabank transfer (`/remittance/intrabank`)
3. **Different Country**: Use inter-subsidiary transfer (`/remittance/intersubsidiary`)
4. **Handle Forex**: API automatically converts currency; store exchange rate and converted amount
5. **Track Status**: Monitor for cross-border delays (1-2 business days typical)

## Error Handling

Jenga API returns consistent error responses. Always check the `status` field in responses.

### Success Response

```json
{
  "data": { /* response data */ },
  "status": "success"
}
```

### Error Response

```json
{
  "errors": [
    {
      "code": "INVALID_ACCOUNT",
      "message": "Account not found or inactive"
    }
  ],
  "status": "error"
}
```

### HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process response data |
| 400 | Bad Request | Check request format, missing fields, invalid data types |
| 401 | Unauthorized | Token expired or invalid; re-authenticate |
| 403 | Forbidden | Insufficient permissions, account inactive, or insufficient funds |
| 404 | Not Found | Account or transaction not found |
| 429 | Rate Limited | Exceeded request limit; implement exponential backoff |
| 500 | Server Error | Temporary issue; retry with exponential backoff |
| 503 | Service Unavailable | Maintenance; retry after delay |

### Common Error Codes

| Code | Meaning | Resolution |
|------|---------|-----------|
| INVALID_ACCOUNT | Account doesn't exist or is inactive | Verify account number and status |
| INSUFFICIENT_FUNDS | Source account lacks funds | Check balance before transfer |
| INVALID_TOKEN | Token expired or malformed | Re-authenticate to get new token |
| INVALID_AMOUNT | Amount is invalid (zero, negative, exceeds limit) | Verify amount is positive and within limits |
| INVALID_REFERENCE | Reference already exists | Use unique reference for each transaction |
| INVALID_CURRENCY | Currency not supported for this transfer | Use supported currency (KES, USD, EUR, etc.) |
| INVALID_DESTINATION | Recipient account doesn't exist | Verify destination account number |
| DUPLICATE_TRANSACTION | Transaction already processed | Check API response; money may have been sent |
| PROCESSING_ERROR | Backend processing failed | Retry; contact support if persists |
| NETWORK_ERROR | Connection issue | Retry with exponential backoff |

### Retry Strategy

Implement exponential backoff for transient errors (5xx, 429):

```javascript
async function requestWithRetry(fn, maxRetries = 3) {
  let lastError;
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      if (error.status === 429 || error.status >= 500) {
        const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error; // Don't retry 4xx errors
      }
    }
  }
  throw lastError;
}
```

## Supported Countries and Currencies

### Coverage

Equity Bank operates in six East African countries:

| Country | Country Code | Local Currency |
|---------|--------------|-----------------|
| Kenya | KE | KES (Kenyan Shilling) |
| Uganda | UG | UGX (Ugandan Shilling) |
| Tanzania | TZ | TZS (Tanzanian Shilling) |
| Rwanda | RW | RWF (Rwandan Franc) |
| Democratic Republic of Congo | CD | CDF (Congolese Franc) |
| South Sudan | SS | SSP (South Sudanese Pound) |

### Supported Currencies

The Jenga API supports the following currencies for transactions:

- **Local Currencies**: KES, UGX, TZS, RWF, CDF, SSP
- **International Currencies**: USD (US Dollar), EUR (Euro), GBP (British Pound), ZAR (South African Rand)

Forex conversion automatically applied for cross-currency transfers. Exchange rates provided in transaction responses.

## Important Notes and Gotchas

### Authentication & Tokens

- **Token Expiry**: Access tokens expire after 1 hour. Implement refresh logic before expiry.
- **Token Caching**: Cache tokens to reduce authentication calls and improve performance.
- **401 Handling**: Automatically re-authenticate if you receive a 401 Unauthorized response.
- **Consumer Secret**: Keep consumer secret confidential; treat like a password.
- **Api-Key Header**: Required on ALL requests; don't forget it.

### Transactions

- **Idempotency**: Always use unique transaction references. If a request times out, use the same reference to check status rather than retrying the POST.
- **Processing Time**: Transfers vary by type:
  - Intra-bank (EFT): 1-5 minutes
  - Inter-subsidiary: 1-2 business days
  - PesaLink: 1-2 hours
  - RTGS: 5-30 minutes (during business hours)
- **Amounts**: Amounts are decimal format (5000.00 for 5,000 KES). Verify precision when handling currency conversions.
- **No Refunds**: Transactions cannot be reversed via API. Contact Equity support for disputed transactions.
- **Business Hours**: RTGS transfers only process Monday-Friday, 7:00 AM - 6:00 PM EAT.

### Account Operations

- **Account Numbers**: Format and length vary by country. Validate using destination country requirements.
- **Account Status**: Check account is ACTIVE before sending funds; DORMANT or CLOSED accounts will fail.
- **Available vs. Total Balance**: Use `availableBalance` to check if funds are available now; `accountBalance` includes pending transactions.
- **Currency**: Account currency determines which transfers are allowed.

### Error Handling

- **Retries**: Retry only 5xx and 429 errors. 4xx errors indicate client errors that won't succeed on retry.
- **Webhook Verification**: Always verify webhook signatures before processing. Don't trust webhook data alone for transaction confirmation.
- **Idempotency Keys**: Jenga doesn't support idempotency keys; use unique references instead.
- **Rate Limits**: Typically 1000 requests/hour per merchant. Monitor usage and implement caching.

### Integration Best Practices

- **Test Thoroughly**: Use sandbox credentials extensively before production.
- **Log Everything**: Store transaction IDs, amounts, statuses, timestamps for audit trails.
- **Handle Webhooks Asynchronously**: Process webhooks in background to avoid timeouts.
- **Monitor Balances**: Periodically verify account balances match your records.
- **Use HTTPS**: All connections must be HTTPS; never transmit credentials over HTTP.
- **Environment Variables**: Store all credentials in environment variables or secure vaults.
- **Error Messages**: Don't expose Equity API error details to end users; log internally and show generic messages.

### Known Limitations

- **Daily Limits**: Account may have daily transaction limits; consult Equity for your account specifics.
- **Country Restrictions**: Some transfer types are restricted by country (e.g., PesaLink only in Kenya).
- **Bank Holidays**: Transfers may be delayed on regional bank holidays.
- **International Transfers**: Cross-border transfers may take longer and incur additional fees.
- **Mobile Providers**: PesaLink mobile transfers only work with registered mobile wallets.

## Rate Limiting

- **Limit**: 1000 requests per hour per merchant
- **Response Header**: X-RateLimit-Remaining indicates requests left
- **When Limited**: Receive 429 Too Many Requests status code
- **Retry**: Implement exponential backoff; wait 60+ seconds before retry
- **Optimization**: Cache account balances, batch queries, and reuse tokens to minimize requests

## Supported Transfer Types

| Type | Use Case | Processing Time | Countries |
|------|----------|-----------------|-----------|
| EFT (Intrabank) | Transfer between Equity accounts in same country | 1-5 min | All 6 |
| Inter-Subsidiary | Transfer between Equity accounts in different countries | 1-2 days | All 6 |
| PesaLink Account | Transfer to other bank accounts | 1-2 hours | Kenya only |
| PesaLink Mobile | Transfer to mobile wallets | Instant - 5 min | Kenya only |
| RTGS | High-value, real-time settlement transfers | 5-30 min | Kenya, Uganda, Tanzania |

## Useful Links

- **Official Jenga API Documentation**: https://developer.jengahq.io/
- **API Explorer**: https://developer.jengahq.io/api-explorer/
- **Developer Dashboard**: https://jengahq.io
- **Getting Started Guide**: https://developer.jengahq.io/guides/get-started/developer-quickstart
- **Error Responses**: https://developer.jengahq.io/guides/jenga-api/errors/error-responses
- **Support Portal**: https://support.jengahq.io
- **Account Services API**: https://www.jengaapi.io/accountaapi.php
- **Payment Methods**: https://developer.jengahq.io/guides/jenga-api/supported-payment-methods
- **Sandbox Base URL**: https://uat.finserve.africa
- **Production Base URL**: https://api.finserve.africa

## Version Information

- **API Version**: v3.0
- **Authentication Version**: v3
- **Documentation Updated**: February 2025
- **Last Verified**: February 24, 2025

---

For the most current endpoint details, error codes, and feature updates, consult the official Jenga API documentation at https://developer.jengahq.io.
