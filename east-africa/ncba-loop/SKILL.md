---
name: ncba-loop
description: "Integrate with NCBA Loop digital banking API for account management and payments across Kenya, Uganda, Tanzania, and Rwanda. Use this skill to check account balances, send M-Pesa payments, initiate bank transfers, query transaction history, and access open banking services through NCBA Group's secure OAuth 2.0 API."
---

# NCBA Loop Integration Skill

> ⚠️ **Access and documentation:** NCBA Loop documentation is not publicly accessible without a developer account. Apply via [loop.ncbagroup.com](https://loop.ncbagroup.com) or through NCBA's corporate banking team. Budget 2–4 weeks for activation.

> ⚠️ **Dual authentication required.** NCBA Loop uses both OAuth 2.0 Bearer tokens AND an API key (`x-api-key` header) on every request. Missing either will result in 401 errors. Some endpoints may additionally require the API key to be embedded in the request body — confirm with NCBA documentation during onboarding.

> ⚠️ **Domain uncertainty.** NCBA has undergone multiple rebrands (CBA + NIC → NCBA; Loop product variants). The base URLs documented here (`api.loop.ncbagroup.com`, `sandbox-api.loop.ncbagroup.com`) follow the expected pattern but should be confirmed against credentials provided during onboarding — do not assume these are stable.

NCBA Loop is the digital banking API platform from NCBA Group (National Commercial Bank of Africa), a leading pan-African banking institution. It enables developers to integrate account information retrieval, payments, transfers, and transaction history into their applications with enterprise-grade security and reliability. NCBA Loop provides open banking capabilities across Kenya, Uganda, Tanzania, and Rwanda.

## When to use this skill

Use NCBA Loop when building fintech applications, payment platforms, accounting tools, corporate payment systems, or any solution that requires integration with NCBA bank accounts. NCBA Loop is ideal for:

- **Personal Finance Apps**: Display account balances and transaction history in real-time dashboards
- **Payment Processing**: Accept payments via bank transfers, M-Pesa, or Pesalink
- **Enterprise Banking**: Integrate bulk payments, payroll processing, and transaction reconciliation
- **B2B Platforms**: Automate payment workflows between businesses
- **Financial Reporting**: Access transaction data for accounting and compliance purposes
- **Money Transfer Services**: Enable customers to send money domestically across East Africa

NCBA Loop's advantage is its regional coverage (Kenya, Uganda, Tanzania, Rwanda), OAuth 2.0 security, and support for multiple payment channels including Pesalink, M-Pesa, and bank-to-bank transfers.

## Authentication

NCBA Loop uses OAuth 2.0 with API Key authentication for a secure, token-based approach.

### Base URLs

```
Production: https://api.loop.ncbagroup.com
Sandbox:    https://sandbox-api.loop.ncbagroup.com
```

### Obtaining Credentials

1. Register for a developer account at the NCBA Loop Developer Portal
2. Create an application to obtain:
   - `client_id`: Your application identifier
   - `client_secret`: Your application secret (keep secure)
   - `api_key`: Your API key for request signing
3. Store all credentials in environment variables (never hardcode)

### Request an OAuth Token

Exchange your credentials for a Bearer token using the OAuth 2.0 Client Credentials flow:

```
POST /oauth/token
Host: api.loop.ncbagroup.com
Content-Type: application/x-www-form-urlencoded
```

**Body:**
```
grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET
```

**Response (200 OK):**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "accounts payments transactions"
}
```

### Required Headers for All Requests

```
Authorization: Bearer {access_token}
X-API-Key: YOUR_API_KEY
Content-Type: application/json
```

**Token Lifecycle:**
- Access tokens are valid for 3600 seconds (1 hour)
- Implement token caching with automatic refresh before expiration
- Handle 401 responses by requesting a new token
- Never hardcode credentials in code

### Example: Secure Token Management

```javascript
let cachedToken = null;
let tokenExpiry = null;

async function getValidToken() {
  const now = Date.now();

  if (cachedToken && tokenExpiry && tokenExpiry > now + 300000) {
    return cachedToken;
  }

  const response = await fetch('https://api.loop.ncbagroup.com/oauth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: `grant_type=client_credentials&client_id=${process.env.NCBA_CLIENT_ID}&client_secret=${process.env.NCBA_CLIENT_SECRET}`
  });

  const data = await response.json();
  cachedToken = data.access_token;
  tokenExpiry = now + (data.expires_in * 1000);

  return cachedToken;
}
```

## Core API Reference

### Account Management

#### Get Account Balance

Retrieve the current and available balance for a specific account in real-time.

```
GET /api/v1/accounts/{accountId}/balance
Authorization: Bearer {access_token}
X-API-Key: {api_key}
```

**Path Parameters:**
- `accountId` (required): The unique account identifier

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "account_id": "ACC_1234567890",
    "account_number": "101234567890",
    "account_name": "John Doe",
    "balance": 150000.00,
    "available_balance": 145000.00,
    "currency": "KES",
    "last_updated": "2025-02-24T14:32:10Z",
    "account_type": "SAVINGS"
  }
}
```

**Key Fields:**
- `balance`: Total account balance including holds and pending transactions
- `available_balance`: Balance available for immediate use (excludes holds)
- `last_updated`: ISO 8601 timestamp of last balance refresh

#### Get Account Details

Retrieve comprehensive account information including account type, status, and customer ID.

```
GET /api/v1/accounts/{accountId}
Authorization: Bearer {access_token}
X-API-Key: {api_key}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "account_id": "ACC_1234567890",
    "account_number": "101234567890",
    "account_name": "John Doe",
    "account_type": "SAVINGS",
    "currency": "KES",
    "status": "ACTIVE",
    "created_date": "2023-01-15T00:00:00Z",
    "customer_id": "CUST_12345",
    "branch_code": "001",
    "iban": "KE89NCBA000000101234567890"
  }
}
```

#### List All Customer Accounts

Retrieve all accounts accessible by the authenticated customer.

```
GET /api/v1/accounts
Authorization: Bearer {access_token}
X-API-Key: {api_key}
```

**Query Parameters:**
- `page` (optional, default: 1): Page number for pagination
- `limit` (optional, default: 20, max: 100): Records per page

**Response (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "account_id": "ACC_1234567890",
      "account_number": "101234567890",
      "account_name": "John Doe",
      "account_type": "SAVINGS",
      "currency": "KES",
      "balance": 150000.00,
      "available_balance": 145000.00,
      "status": "ACTIVE"
    },
    {
      "account_id": "ACC_0987654321",
      "account_number": "101098765432",
      "account_name": "John Doe Checking",
      "account_type": "CHECKING",
      "currency": "KES",
      "balance": 500000.00,
      "available_balance": 495000.00,
      "status": "ACTIVE"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 2,
    "total_pages": 1
  }
}
```

### Payment Operations

#### Send M-Pesa Payment

Send money from an account to an M-Pesa mobile money number. Payments process within seconds.

```
POST /api/v1/payments/mpesa
Authorization: Bearer {access_token}
X-API-Key: {api_key}
Content-Type: application/json
```

**Request Body:**
```json
{
  "account_id": "ACC_1234567890",
  "phone_number": "+254712345678",
  "amount": 5000.00,
  "currency": "KES",
  "reference": "MPESA_TXN_001",
  "description": "Payment for goods",
  "idempotency_key": "unique-identifier-for-idempotency"
}
```

**Field Descriptions:**
- `account_id` (required): Source account identifier
- `phone_number` (required): Recipient's M-Pesa phone number with country code
- `amount` (required): Transaction amount in decimal format
- `currency` (required): ISO 4217 currency code (KES, UGX, TZS, RWF)
- `reference` (required): Unique transaction reference for idempotency (max 50 chars)
- `description` (optional): Transaction narrative (max 100 chars)
- `idempotency_key` (optional): For duplicate prevention across retries

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "transaction_id": "TXN_MPESA_20250224_001",
    "reference": "MPESA_TXN_001",
    "amount": 5000.00,
    "currency": "KES",
    "recipient": "+254712345678",
    "status": "PROCESSING",
    "initiated_at": "2025-02-24T14:33:15Z",
    "expires_at": "2025-02-24T14:43:15Z"
  }
}
```

**Status Values:**
- `PROCESSING`: Payment queued for transmission
- `COMPLETED`: Money sent successfully
- `FAILED`: Insufficient funds or invalid recipient
- `EXPIRED`: Payment not completed within 10 minutes

#### Initiate Bank Transfer

Send money to another bank account within the same country or across East Africa via Pesalink (Kenya).

```
POST /api/v1/transfers/bank
Authorization: Bearer {access_token}
X-API-Key: {api_key}
Content-Type: application/json
```

**Request Body:**
```json
{
  "account_id": "ACC_1234567890",
  "beneficiary": {
    "account_number": "0987654321",
    "account_name": "Jane Smith",
    "bank_code": "011",
    "country": "KE"
  },
  "amount": 25000.00,
  "currency": "KES",
  "reference": "BANK_TXN_001",
  "description": "Invoice payment",
  "idempotency_key": "unique-identifier-for-idempotency",
  "charge_bearer": "SENDER"
}
```

**Field Descriptions:**
- `account_id` (required): Source account identifier
- `beneficiary.account_number` (required): Recipient's account number
- `beneficiary.account_name` (required): Recipient's full name
- `beneficiary.bank_code` (required): Swift code or bank identifier
  - Kenya NCBA: "011"
  - Equity: "031"
  - KCB: "001"
  - See bank codes table below
- `beneficiary.country` (required): Beneficiary country code (KE, UG, TZ, RW)
- `amount` (required): Transfer amount in decimal format
- `currency` (required): ISO 4217 currency code
- `reference` (required): Unique transaction reference (max 50 chars)
- `description` (optional): Transfer narration (max 100 chars)
- `charge_bearer` (required): "SENDER" (charges deducted from sender) or "BENEFICIARY"

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "transaction_id": "TXN_BANK_20250224_001",
    "reference": "BANK_TXN_001",
    "amount": 25000.00,
    "currency": "KES",
    "beneficiary": {
      "account_number": "0987654321",
      "account_name": "Jane Smith",
      "bank_code": "011"
    },
    "status": "PENDING",
    "initiated_at": "2025-02-24T14:34:20Z",
    "expected_completion": "2025-02-25T14:34:20Z"
  }
}
```

**Processing Times:**
- Same-bank transfers: 1-5 minutes
- Pesalink (Kenya): Real-time, 24/7/365
- EFT (Kenya): 1-2 business hours
- Cross-border (East Africa): 1-2 business days

**Supported Bank Codes (Kenya):**
```
011 - NCBA
001 - KCB
031 - Equity
022 - Barclays
046 - Diamond Trust
082 - Family Bank
091 - Transnational Bank
105 - Co-operative Bank
106 - Pride Microfinance
113 - Bank of Africa
114 - Invest & Mortgages
102 - Jamii Bora Bank
108 - Kenyatta University Co-op
109 - Mwalimu National SACCO
110 - Nairobi City SACCO
```
> ⚠️ **Bank codes need verification before production use.** The codes in the table above are based on common Kenyan sort codes but have not been independently confirmed against NCBA Loop's own bank registry. Kenyan sort codes can differ from CBK institution codes. Before going live, resolve bank codes dynamically via the NCBA Loop bank lookup endpoint (check their developer documentation) or confirm the exact codes with NCBA Loop support. Using an incorrect bank code will result in failed or misdirected transfers.

### Transaction History

#### Get Transaction History

Retrieve a customer's transaction history with flexible filtering and pagination.

```
GET /api/v1/transactions
Authorization: Bearer {access_token}
X-API-Key: {api_key}
```

**Query Parameters:**
- `account_id` (required): The account to retrieve transactions for
- `from_date` (optional): Start date in ISO 8601 format (YYYY-MM-DD)
- `to_date` (optional): End date in ISO 8601 format (YYYY-MM-DD)
- `transaction_type` (optional): Filter by type (DEBIT, CREDIT, all)
- `page` (optional, default: 1): Page number for pagination
- `limit` (optional, default: 50, max: 100): Records per page
- `sort` (optional): "asc" or "desc" (default: "desc")

**Example Request:**
```
GET /api/v1/transactions?account_id=ACC_1234567890&from_date=2025-02-01&to_date=2025-02-24&limit=50&page=1
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "transaction_id": "TXN_MPESA_20250224_001",
      "date": "2025-02-24T14:33:15Z",
      "type": "DEBIT",
      "amount": 5000.00,
      "currency": "KES",
      "description": "M-Pesa transfer",
      "reference": "MPESA_TXN_001",
      "beneficiary": "+254712345678",
      "balance_after": 145000.00,
      "status": "COMPLETED",
      "channel": "MPESA"
    },
    {
      "transaction_id": "TXN_DEPOSIT_20250224_002",
      "date": "2025-02-24T10:15:30Z",
      "type": "CREDIT",
      "amount": 50000.00,
      "currency": "KES",
      "description": "Salary deposit",
      "reference": "SAL_FEB_2025",
      "balance_after": 150000.00,
      "status": "COMPLETED",
      "channel": "BANK_TRANSFER"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 127,
    "total_pages": 3
  }
}
```

**Transaction Type Mapping:**
- `DEBIT`: Money sent out
- `CREDIT`: Money received
- `TRANSFER`: Account-to-account transfers
- `PAYMENT`: Merchant or bill payments
- `REVERSAL`: Transaction reversal or refund

#### Get Transaction Status

Check the real-time status of a specific transaction.

```
GET /api/v1/transactions/{transactionId}
Authorization: Bearer {access_token}
X-API-Key: {api_key}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "transaction_id": "TXN_MPESA_20250224_001",
    "reference": "MPESA_TXN_001",
    "amount": 5000.00,
    "currency": "KES",
    "status": "COMPLETED",
    "initiated_at": "2025-02-24T14:33:15Z",
    "completed_at": "2025-02-24T14:33:45Z",
    "recipient": "+254712345678",
    "description": "Payment for goods"
  }
}
```

## Webhooks

NCBA Loop sends real-time webhooks for transaction events. Configure your webhook endpoint in the dashboard to receive notifications.

### Webhook Security

Every webhook includes an `X-Loop-Signature` header. Verify this signature to ensure the webhook originated from NCBA Loop:

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(body, signature, secret) {
  const hash = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(body))
    .digest('hex');

  return hash === signature;
}

// In your webhook handler:
app.post('/webhook', (req, res) => {
  const signature = req.headers['x-loop-signature'];

  if (!verifyWebhookSignature(req.body, signature, process.env.NCBA_WEBHOOK_SECRET)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook
  res.status(200).json({ received: true });
});
```

### Webhook Events

#### Transaction Completed

Sent when a payment or transfer completes successfully.

```json
{
  "event": "transaction.completed",
  "timestamp": "2025-02-24T14:33:45Z",
  "data": {
    "transaction_id": "TXN_MPESA_20250224_001",
    "account_id": "ACC_1234567890",
    "reference": "MPESA_TXN_001",
    "amount": 5000.00,
    "currency": "KES",
    "type": "MPESA_TRANSFER",
    "recipient": "+254712345678",
    "status": "COMPLETED",
    "description": "Payment for goods"
  }
}
```

#### Transaction Failed

Sent when a payment or transfer fails.

```json
{
  "event": "transaction.failed",
  "timestamp": "2025-02-24T14:35:10Z",
  "data": {
    "transaction_id": "TXN_BANK_20250224_002",
    "account_id": "ACC_1234567890",
    "reference": "BANK_TXN_002",
    "amount": 10000.00,
    "currency": "KES",
    "status": "FAILED",
    "error_code": "INSUFFICIENT_BALANCE",
    "error_message": "Account balance is insufficient for this transaction",
    "initiated_at": "2025-02-24T14:34:50Z"
  }
}
```

#### Payment Confirmed

Sent when a payment gateway confirms transaction initiation.

```json
{
  "event": "payment.confirmed",
  "timestamp": "2025-02-24T14:33:20Z",
  "data": {
    "transaction_id": "TXN_MPESA_20250224_001",
    "reference": "MPESA_TXN_001",
    "amount": 5000.00,
    "type": "MPESA",
    "recipient": "+254712345678",
    "status": "CONFIRMED"
  }
}
```

#### Account Updated

Sent when account balance or status changes.

```json
{
  "event": "account.updated",
  "timestamp": "2025-02-24T14:36:00Z",
  "data": {
    "account_id": "ACC_1234567890",
    "account_number": "101234567890",
    "balance": 140000.00,
    "available_balance": 135000.00,
    "status": "ACTIVE"
  }
}
```

### Webhook Retry Policy

NCBA Loop retries failed webhooks with exponential backoff:
- Attempt 1: Immediate
- Attempt 2: 5 seconds
- Attempt 3: 30 seconds
- Attempt 4: 5 minutes
- Attempt 5: 30 minutes

Return HTTP 2xx status to acknowledge receipt. Return any other status to trigger a retry.

## Common Integration Patterns

### Pattern 1: Real-Time Balance Dashboard

Display account balances and transaction history in a web dashboard.

```javascript
async function updateDashboard() {
  const token = await getValidToken();

  // Fetch all accounts
  const accountsRes = await fetch('https://api.loop.ncbagroup.com/api/v1/accounts', {
    headers: {
      'Authorization': `Bearer ${token}`,
      'X-API-Key': process.env.NCBA_API_KEY
    }
  });

  const accounts = await accountsRes.json();

  // Fetch balance for each account
  const balances = await Promise.all(
    accounts.data.map(async (account) => {
      const res = await fetch(
        `https://api.loop.ncbagroup.com/api/v1/accounts/${account.account_id}/balance`,
        { headers: { 'Authorization': `Bearer ${token}`, 'X-API-Key': process.env.NCBA_API_KEY } }
      );
      return res.json();
    })
  );

  // Refresh dashboard every 5 minutes
  setTimeout(updateDashboard, 5 * 60 * 1000);
}
```

### Pattern 2: Payment Processing Workflow

1. Customer selects account and enters payment details
2. Initiate payment with M-Pesa or bank transfer
3. Return transaction ID to customer
4. Listen for webhook confirmation
5. Update UI with payment status

```javascript
async function processPayment(accountId, recipient, amount, type) {
  const token = await getValidToken();

  const payload = type === 'mpesa'
    ? {
        account_id: accountId,
        phone_number: recipient,
        amount: amount,
        currency: 'KES',
        reference: `PAY_${Date.now()}`,
        idempotency_key: `PAY_${accountId}_${Date.now()}`
      }
    : {
        account_id: accountId,
        beneficiary: { account_number: recipient.accountNumber, account_name: recipient.name, bank_code: recipient.bankCode, country: 'KE' },
        amount: amount,
        currency: 'KES',
        reference: `PAY_${Date.now()}`,
        idempotency_key: `PAY_${accountId}_${Date.now()}`
      };

  const endpoint = type === 'mpesa'
    ? '/api/v1/payments/mpesa'
    : '/api/v1/transfers/bank';

  const res = await fetch(`https://api.loop.ncbagroup.com${endpoint}`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'X-API-Key': process.env.NCBA_API_KEY,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(payload)
  });

  return res.json();
}
```

### Pattern 3: Transaction Reconciliation

Periodically sync NCBA transactions with your internal records to identify discrepancies.

```javascript
async function reconcileTransactions(accountId, internalTransactions) {
  const token = await getValidToken();

  // Fetch NCBA transactions from last 30 days
  const today = new Date();
  const thirtyDaysAgo = new Date(today.getTime() - 30 * 24 * 60 * 60 * 1000);

  const res = await fetch(
    `https://api.loop.ncbagroup.com/api/v1/transactions?account_id=${accountId}&from_date=${thirtyDaysAgo.toISOString().split('T')[0]}&to_date=${today.toISOString().split('T')[0]}`,
    { headers: { 'Authorization': `Bearer ${token}`, 'X-API-Key': process.env.NCBA_API_KEY } }
  );

  const ncbaTransactions = await res.json();

  // Compare and flag discrepancies
  const discrepancies = [];

  for (const ncbaTxn of ncbaTransactions.data) {
    const match = internalTransactions.find(t =>
      t.amount === ncbaTxn.amount &&
      t.date === ncbaTxn.date &&
      t.reference === ncbaTxn.reference
    );

    if (!match) {
      discrepancies.push({
        ncbaTransaction: ncbaTxn,
        status: 'UNMATCHED',
        action: 'VERIFY_REQUIRED'
      });
    }
  }

  return { matched: ncbaTransactions.data.length - discrepancies.length, discrepancies };
}
```

### Pattern 4: Webhook Event Handler

```javascript
app.post('/ncba-webhook', (req, res) => {
  const signature = req.headers['x-loop-signature'];

  // Verify signature
  if (!verifyWebhookSignature(req.body, signature, process.env.NCBA_WEBHOOK_SECRET)) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const { event, data } = req.body;

  switch (event) {
    case 'transaction.completed':
      handleTransactionCompleted(data);
      break;
    case 'transaction.failed':
      handleTransactionFailed(data);
      break;
    case 'payment.confirmed':
      handlePaymentConfirmed(data);
      break;
    case 'account.updated':
      handleAccountUpdated(data);
      break;
  }

  res.status(200).json({ received: true });
});
```

## Error Handling

NCBA Loop returns error responses with a consistent structure:

```json
{
  "status": "error",
  "code": "ERROR_CODE",
  "message": "Human-readable error description",
  "details": {
    "field": "Specific field that caused the error (if applicable)"
  }
}
```

### HTTP Status Codes and Error Codes

| Status | Error Code | Meaning | Action |
|--------|-----------|---------|--------|
| 400 | INVALID_REQUEST | Missing or malformed fields | Validate request payload |
| 400 | INVALID_AMOUNT | Amount is invalid or zero | Use positive decimal amounts |
| 400 | INVALID_PHONE | Phone number format incorrect | Ensure country code included (+254...) |
| 400 | INVALID_ACCOUNT | Account number format incorrect | Verify account number format |
| 401 | UNAUTHORIZED | Invalid or expired token | Request a new token |
| 401 | INVALID_API_KEY | API key is invalid or missing | Verify API key in X-API-Key header |
| 403 | INSUFFICIENT_BALANCE | Account balance is insufficient | Ensure sufficient funds available |
| 403 | ACCOUNT_RESTRICTED | Account is frozen or restricted | Contact NCBA support |
| 404 | ACCOUNT_NOT_FOUND | Account doesn't exist | Verify account ID is correct |
| 404 | TRANSACTION_NOT_FOUND | Transaction ID doesn't exist | Check transaction ID |
| 409 | DUPLICATE_REQUEST | Idempotency key already processed | Use unique idempotency key |
| 422 | VALIDATION_FAILED | Business logic validation failed | Review error details |
| 429 | RATE_LIMITED | Rate limit exceeded | Implement exponential backoff |
| 500 | SERVER_ERROR | NCBA API server error | Retry with exponential backoff |
| 503 | SERVICE_UNAVAILABLE | Service temporarily unavailable | Retry after 60 seconds |

### Error Response Examples

**Insufficient Balance:**
```json
{
  "status": "error",
  "code": "INSUFFICIENT_BALANCE",
  "message": "Account balance is insufficient for this transaction",
  "details": {
    "required": 10000.00,
    "available": 5000.00,
    "currency": "KES"
  }
}
```

**Invalid Phone Number:**
```json
{
  "status": "error",
  "code": "INVALID_PHONE",
  "message": "Phone number must include country code",
  "details": {
    "provided": "712345678",
    "format": "+254712345678"
  }
}
```

**Rate Limited:**
```json
{
  "status": "error",
  "code": "RATE_LIMITED",
  "message": "Rate limit exceeded. Maximum 1000 requests per hour",
  "details": {
    "retry_after": 60,
    "current_requests": 1005,
    "limit": 1000
  }
}
```

### Recommended Error Handling Strategy

```javascript
async function makeRequest(endpoint, options = {}, retries = 3) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      const response = await fetch(endpoint, options);

      if (response.ok) {
        return await response.json();
      }

      const error = await response.json();

      // Retry on server errors and rate limiting
      if ([429, 500, 503].includes(response.status) && attempt < retries) {
        const backoff = Math.pow(2, attempt - 1) * 1000; // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, backoff));
        continue;
      }

      // Don't retry on client errors
      throw new Error(`${error.code}: ${error.message}`);
    } catch (error) {
      if (attempt === retries) throw error;
    }
  }
}
```

## Important Notes and Gotchas

### Security Best Practices

1. **Never hardcode credentials**: Store client_id, client_secret, and api_key in environment variables
2. **Always verify webhooks**: Check X-Loop-Signature header to ensure webhooks are from NCBA
3. **Use HTTPS only**: All API communication must use encrypted connections
4. **Rotate API keys regularly**: Implement key rotation every 90 days
5. **Implement idempotency**: Use idempotency_key to prevent duplicate transactions on retries
6. **Rate limiting**: Implement exponential backoff starting at 1 second for 429 responses

### Data Format and Validation

1. **Amounts**: Always use decimal format (e.g., 1000.00 for 1000 KES, not 100000 cents)
2. **Phone numbers**: Must include country code (e.g., +254712345678, not 0712345678)
3. **Account IDs**: Use the account_id from API responses, not the account_number
4. **References**: Keep references unique and under 50 characters for tracking
5. **Currency codes**: Use ISO 4217 format (KES, UGX, TZS, RWF)
6. **Dates**: Always use ISO 8601 format (YYYY-MM-DDTHH:mm:ssZ)

### Processing Times

1. **M-Pesa transfers**: 5-30 seconds for processing
2. **Pesalink (Kenya)**: Real-time settlement, 24/7/365
3. **EFT transfers (Kenya)**: 1-2 hours during business hours
4. **Cross-border transfers**: 1-2 business days
5. **Account balance**: Updated within 1 minute of transaction
6. **Transaction history**: Available within 2-3 minutes of completion

### Regional Considerations

- **Kenya (KES)**: Pesalink, M-Pesa, EFT, RTGS supported
- **Uganda (UGX)**: Bank transfers, mobile money via partners
- **Tanzania (TZS)**: Bank transfers, mobile money via partners
- **Rwanda (RWF)**: Bank transfers supported
- Pesalink minimum: KES 50 (or currency equivalent)
- Pesalink maximum: KES 999,999 (or currency equivalent)
- Some channels may not be available in all countries

### Common Pitfalls to Avoid

1. **Not caching tokens**: Requesting new tokens on every API call causes rate limit issues
2. **Missing idempotency keys**: Can cause duplicate transactions if retries occur
3. **Not handling 401s**: Expired tokens aren't automatically refreshed
4. **Insufficient error handling**: Not implementing exponential backoff for retries
5. **Hardcoded base URLs**: Using production URL in development environment
6. **Not validating webhooks**: Processing webhooks from unknown sources
7. **Assuming instant completion**: Transactions have various processing times
8. **Not handling partial failures**: Retrying entire operations instead of atomic units

### Transaction Reference Best Practices

1. Keep references globally unique and idempotent
2. Use format: `PREFIX_TIMESTAMP_SEQUENCE` (e.g., `PAY_1708859595_001`)
3. Store references in your database for reconciliation
4. Don't reuse references for different transactions
5. Query transaction history by reference for confirmation

```javascript
function generateTransactionReference(prefix = 'TXN') {
  return `${prefix}_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
}
```

### Token Management

```javascript
class NCBATokenManager {
  constructor(clientId, clientSecret) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.token = null;
    this.expiresAt = null;
  }

  async getToken() {
    if (this.token && this.expiresAt > Date.now() + 300000) {
      return this.token;
    }

    const response = await fetch('https://api.loop.ncbagroup.com/oauth/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: `grant_type=client_credentials&client_id=${this.clientId}&client_secret=${this.clientSecret}`
    });

    const data = await response.json();
    this.token = data.access_token;
    this.expiresAt = Date.now() + (data.expires_in * 1000);

    return this.token;
  }

  async refreshIfNeeded() {
    if (!this.token || this.expiresAt <= Date.now() + 300000) {
      return await this.getToken();
    }
    return this.token;
  }
}
```

## Rate Limiting and Quotas

- **Rate Limit**: 1000 requests per hour per API key
- **Burst Limit**: 100 requests per minute
- **Transaction Limit**: No daily limit on transaction count
- **Amount Limits** (per transaction):
  - M-Pesa: KES 500,000 maximum
  - Bank transfer: KES 5,000,000 maximum (varies by account type)
  - Pesalink: KES 50 - KES 999,999 per transaction

**Rate Limit Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1708863000
```

## Supported Currencies and Countries

| Country | Currency | Code | Channels |
|---------|----------|------|----------|
| Kenya | Kenyan Shilling | KES | M-Pesa, Pesalink, EFT, RTGS, Bank Transfer |
| Uganda | Ugandan Shilling | UGX | Bank Transfer, Mobile Money |
| Tanzania | Tanzanian Shilling | TZS | Bank Transfer, Mobile Money |
| Rwanda | Rwandan Franc | RWF | Bank Transfer |

## Useful Links

- [NCBA Loop Developer Portal](https://developer.loop.ncbagroup.com)
- [NCBA Group Official Website](https://www.ncbagroup.com)
- [NCBA Kenya](https://ke.ncbagroup.com)
- [NCBA Tanzania](https://ncbagroup.co.tz)
- [API Solutions - NCBA](https://ke.ncbagroup.com/payment-solution/api-solutions/)
- [Payment Solutions - NCBA Group](https://ke.ncbagroup.com/for-corporates/payment-solutions/)
- [API Integration Documentation](https://ke.ncbagroup.com/payment-solution/instant-payment-notification-push-service-api-integration/)
- [NCBA Support Portal](https://support.ncbagroup.com)
- [Open Banking Standards](https://standards.openbanking.org.uk)
- [NCBA Loop GitHub Resources](https://github.com/akikadigital/laravel-ncba)

---

**Last Updated**: 2025-02-24
**API Version**: v1
**Status**: Production Ready
