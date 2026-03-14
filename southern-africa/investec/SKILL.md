---
name: investec
description: "Integrate with Investec Open API for programmable banking in South Africa. Use this skill whenever the user wants to access Investec accounts programmatically, query transaction history, build banking apps, manage account information, initiate transfers, build fintech solutions on top of Investec, or work with Investec's Open API in any way. Also trigger when the user mentions 'Investec Open API', 'programmable banking', 'Investec integration', 'account API', 'transaction queries', or needs bank account data access."
---

# Investec Open API Integration Skill

Investec Open API enables programmable banking for businesses and private clients in South Africa. It provides secure, OAuth 2.0-based access to account information, transaction history, and the ability to initiate transfers programmatically. The API is PSD2 compliant and designed for fintech integrations, financial dashboards, and automated banking operations.

## When to use this skill

Use this skill when building:
- Financial dashboards and account management applications
- Transaction reconciliation and reporting tools
- Automated transfer and payment systems
- Cash flow forecasting and financial analytics
- Expense tracking and budgeting applications
- Any integration requiring secure access to Investec account data

The Open API is ideal for businesses that need programmatic access to real-time account balances, transaction history, and payment capabilities.

## Authentication

Investec Open API uses **OAuth 2.0 client credentials flow** for secure authentication. This is a server-to-server authentication pattern suitable for backend applications.

### Step 1: Obtain an Access Token

```
POST https://openapi.investec.com/identity/v2/oauth2/token
```

**Headers:**
```
Content-Type: application/x-www-form-urlencoded
```

**Body (form-encoded):**
```
grant_type=client_credentials
client_id=YOUR_CLIENT_ID
client_secret=YOUR_CLIENT_SECRET
scope=accounts transactions transfers
```

**Response (success):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "accounts transactions transfers"
}
```

**Important:**
- Access tokens expire in **1 hour (3600 seconds)**
- Cache the token and implement refresh logic before expiry
- Store `client_id` and `client_secret` in environment variables (never hardcode)
- Request only the scopes your application needs (principle of least privilege)

### Step 2: Use Token for API Requests

Include the access token in the `Authorization` header for all API requests:

```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

### Scopes

Request the following scopes based on your application's needs:

- `accounts` — List accounts and view account details
- `transactions` — Query transaction history and retrieve account statements
- `transfers` — Initiate transfers and payments

### Base URL

All endpoints use the base URL: `https://openapi.investec.com`

---

## Core API Reference

### Get OAuth Token

Obtain an access token to authenticate subsequent API requests.

**Endpoint:**
```
POST /identity/v2/oauth2/token
```

**Request:**
```bash
curl -X POST https://openapi.investec.com/identity/v2/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&scope=accounts%20transactions%20transfers"
```

**Response (success - 200 OK):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkJhbmtpbmcgQXBwIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "accounts transactions transfers"
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "invalid_client",
  "error_description": "Client authentication failed"
}
```

---

### List Accounts

Retrieve all accounts accessible to the authenticated client.

**Endpoint:**
```
GET /za/pb/v1/accounts
```

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Response (success - 200 OK):**
```json
{
  "data": {
    "accounts": [
      {
        "accountId": "ACC-123456789",
        "accountNumber": "1234567890",
        "accountName": "Business Checking Account",
        "accountStatus": "Active",
        "currency": "ZAR",
        "accountType": "Transactional",
        "accountSubType": "Cheque",
        "referenceName": "Ops Account",
        "productName": "Business Bank Account",
        "kycCompliant": true
      },
      {
        "accountId": "ACC-987654321",
        "accountNumber": "0987654321",
        "accountName": "Savings Account",
        "accountStatus": "Active",
        "currency": "ZAR",
        "accountType": "Savings",
        "accountSubType": "Savings",
        "referenceName": "Emergency Fund",
        "productName": "Business Savings Account",
        "kycCompliant": true
      }
    ]
  }
}
```

**Key Fields:**
- `accountId` — Unique identifier for the account (use this in subsequent API calls)
- `accountNumber` — The traditional account number
- `accountStatus` — Current status (Active, Closed, Suspended, etc.)
- `kycCompliant` — Whether account meets Know Your Customer requirements

**Important:** Always use `accountId` in API calls, not the account number.

---

### Get Account Balance

Retrieve the current and available balances for an account.

**Endpoint:**
```
GET /za/pb/v1/accounts/{accountId}/balances
```

**Path Parameters:**
- `accountId` — The account ID from the accounts list

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Response (success - 200 OK):**
```json
{
  "data": {
    "accountId": "ACC-123456789",
    "balances": [
      {
        "type": "Available",
        "amount": "150000.00",
        "currency": "ZAR"
      },
      {
        "type": "Current",
        "amount": "155000.00",
        "currency": "ZAR"
      }
    ]
  }
}
```

**Balance Types:**
- **Available Balance** — Funds available for immediate use (excludes pending debits and holds)
- **Current Balance** — Total account balance including pending transactions

**Use Case:**
- Use **available balance** for cash flow decisions and transfer authorization
- Use **current balance** for accurate financial reporting and reconciliation

---

### Get Account Transactions

Retrieve transaction history for a specific account.

**Endpoint:**
```
GET /za/pb/v1/accounts/{accountId}/transactions
```

**Path Parameters:**
- `accountId` — The account ID from the accounts list

**Query Parameters:**
- `fromDate` **(required)** — Start date in ISO 8601 format (YYYY-MM-DD)
- `toDate` **(required)** — End date in ISO 8601 format (YYYY-MM-DD)
- `limit` (optional) — Records per page (default: 100, max: 500)
- `offset` (optional) — Records to skip for pagination (default: 0)

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Request Example:**
```bash
curl -X GET "https://openapi.investec.com/za/pb/v1/accounts/ACC-123456789/transactions?fromDate=2025-01-01&toDate=2025-02-28&limit=50" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Response (success - 200 OK):**
```json
{
  "data": {
    "accountId": "ACC-123456789",
    "transactions": [
      {
        "transactionId": "TXN-abc123def456",
        "accountNumber": "1234567890",
        "type": "DEBIT",
        "transactionType": "Payment",
        "status": "Posted",
        "transactionDate": "2025-02-24T10:30:00Z",
        "valueDate": "2025-02-24T10:30:00Z",
        "amount": "5000.00",
        "currency": "ZAR",
        "description": "Payment to ABC Corp",
        "reference": "INV-12345",
        "cardNumber": null,
        "postingDate": "2025-02-24T10:30:00Z",
        "runningBalance": "150000.00"
      },
      {
        "transactionId": "TXN-def789ghi012",
        "accountNumber": "1234567890",
        "type": "CREDIT",
        "transactionType": "Deposit",
        "status": "Posted",
        "transactionDate": "2025-02-23T14:15:00Z",
        "valueDate": "2025-02-23T14:15:00Z",
        "amount": "10000.00",
        "currency": "ZAR",
        "description": "Client payment received",
        "reference": "CLIENT-001",
        "cardNumber": null,
        "postingDate": "2025-02-23T14:15:00Z",
        "runningBalance": "155000.00"
      }
    ],
    "paging": {
      "offset": 0,
      "limit": 50,
      "total": 247
    }
  }
}
```

**Transaction Fields:**
- `transactionId` — Unique transaction identifier
- `type` — Transaction direction (DEBIT or CREDIT)
- `transactionType` — Type of transaction (Payment, Deposit, Withdrawal, Transfer, etc.)
- `status` — Transaction status (Posted, Pending, Failed, etc.)
- `amount` — Transaction amount
- `runningBalance` — Account balance after this transaction
- `reference` — Customer reference for the transaction

**Important Notes:**
- Always provide both `fromDate` and `toDate` (required parameters)
- Dates must be in ISO 8601 format (YYYY-MM-DD)
- Transactions are returned in chronological order
- Use `limit` and `offset` for pagination on large result sets
- Historical data is typically available for 12-24 months

---

### Initiate a Transfer

Initiate a bank transfer (or batch transfers) from your Investec account to one or more beneficiary accounts.

**Endpoint:**
```
POST /za/pb/v1/accounts/{accountId}/transfermultiple
```

**Path Parameters:**
- `accountId` — The account ID to transfer from

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request Body:**
```json
{
  "transferList": [
    {
      "lineAmount": "5000.00",
      "beneficiaryAccountId": "BEN-987654321",
      "reference": "INV-12345",
      "description": "Payment to ABC Corp"
    },
    {
      "lineAmount": "2500.00",
      "beneficiaryAccountId": "BEN-456789012",
      "reference": "INV-12346",
      "description": "Payment to XYZ Ltd"
    }
  ]
}
```

**Request Parameters:**
- `transferList` — Array of transfer objects
  - `lineAmount` — Amount to transfer (as string in currency format)
  - `beneficiaryAccountId` — Beneficiary account ID
  - `reference` — Your internal reference (max 40 characters, alphanumeric)
  - `description` — Transaction description visible to beneficiary (optional)

**Response (success - 200 OK):**
```json
{
  "data": [
    {
      "lineNumber": 1,
      "lineAmount": "5000.00",
      "beneficiaryAccountId": "BEN-987654321",
      "reference": "INV-12345",
      "status": "Success",
      "message": "Transfer initiated successfully"
    },
    {
      "lineNumber": 2,
      "lineAmount": "2500.00",
      "beneficiaryAccountId": "BEN-456789012",
      "reference": "INV-12346",
      "status": "Success",
      "message": "Transfer initiated successfully"
    }
  ]
}
```

**Error Response (400 Bad Request - Insufficient Funds):**
```json
{
  "error": {
    "code": "INSUFFICIENT_FUNDS",
    "message": "Insufficient funds in account for transfer",
    "details": "Available balance: 2500.00, Requested: 5000.00"
  }
}
```

**Important Notes:**
- Transfers typically process within **minutes** during business hours
- Amounts must be positive numbers in ZAR (South African Rand)
- You can batch multiple transfers in a single request
- Beneficiary account IDs must be pre-registered or valid Investec accounts
- Keep references for tracking and reconciliation
- Transfer status should be verified by checking transaction history

---

## Card Programmable Banking

Investec offers programmable banking for Visa cards linked to your account. This allows you to write custom JavaScript code that executes during card transactions.

### Card Transaction Rules

Deploy custom rules to your Investec Visa card to:
- **Approve or decline specific transactions** based on custom logic
- **Limit spend** per merchant, category, or time period
- **Control usage** by restricting transaction types (ATM, online, physical)
- **Track expenses** programmatically in real-time
- **Respond to transactions** before they post to the account

### How It Works

1. Write JavaScript code that defines your transaction rules
2. Deploy to your card via the Investec Developer Portal
3. The code executes automatically on each transaction (including ATM withdrawals)
4. Your rules determine if the transaction is approved or declined
5. Results are logged for audit and analysis

### Example Use Cases

- **Automated expense control** — Decline unauthorized merchants
- **Category-based budgeting** — Limit spending by category
- **Geographic restrictions** — Only allow transactions in specific countries
- **Real-time alerts** — Flag unusual transactions
- **Integration with accounting systems** — Categorize transactions automatically

For detailed documentation on card rule deployment and JavaScript API, visit the [Investec Developer Portal](https://developer.investec.com/programmable-banking/).

---

## Common Integration Patterns

### Pattern 1: Financial Dashboard

Display real-time account balances and recent transactions:

1. `POST /identity/v2/oauth2/token` — Authenticate and get access token
2. `GET /za/pb/v1/accounts` — List all accounts
3. `GET /za/pb/v1/accounts/{id}/balances` — Fetch current and available balances
4. `GET /za/pb/v1/accounts/{id}/transactions?fromDate=YYYY-MM-DD&toDate=YYYY-MM-DD` — Retrieve recent transactions
5. Refresh data every 5-10 minutes using the same token

**Implementation (JavaScript):**
```javascript
async function refreshDashboard() {
  const token = await getOAuthToken();
  const accounts = await getAccounts(token);

  for (const account of accounts) {
    const balance = await getBalance(token, account.accountId);
    const txns = await getTransactions(token, account.accountId, today, today);

    displayAccountCard(account, balance, txns);
  }
}

// Run every 5 minutes
setInterval(refreshDashboard, 5 * 60 * 1000);
```

---

### Pattern 2: Automated Reconciliation

Match bank transactions against internal records (invoices, orders):

1. `GET /za/pb/v1/accounts/{id}/transactions` — Fetch transactions for date range
2. Compare against your internal ledger (invoices, purchase orders)
3. Identify matched transactions and flag discrepancies
4. Store reconciliation results for audit trail
5. Run nightly or weekly via scheduled job (cron, Lambda, etc.)

**Implementation (JavaScript):**
```javascript
async function reconcileTransactions(accountId, startDate, endDate) {
  const token = await getOAuthToken();
  const bankTxns = await getTransactions(token, accountId, startDate, endDate);
  const internalRecords = await getInternalRecords(startDate, endDate);

  const results = {
    matched: [],
    unmatched: [],
    discrepancies: []
  };

  for (const btxn of bankTxns) {
    const match = internalRecords.find(
      r => r.amount === btxn.amount &&
           r.date === btxn.transactionDate
    );

    if (match) {
      results.matched.push({ bank: btxn, internal: match });
    } else {
      results.unmatched.push(btxn);
    }
  }

  return results;
}

// Run nightly at 2 AM
schedule.scheduleJob('0 2 * * *', () => {
  reconcileTransactions(accountId, yesterday, yesterday);
});
```

---

### Pattern 3: Automated Transfer Initiation

Trigger transfers programmatically based on business logic:

1. Determine transfer amount and beneficiary
2. `POST /za/pb/v1/accounts/{id}/transfermultiple` — Initiate transfer
3. Store transfer reference for tracking
4. Poll `GET /za/pb/v1/accounts/{id}/transactions` — Confirm transfer posted
5. Send notifications to relevant stakeholders

**Implementation (JavaScript):**
```javascript
async function processPayments(accountId, payments) {
  const token = await getOAuthToken();

  // Initiate transfers
  const transferList = payments.map(p => ({
    lineAmount: p.amount.toString(),
    beneficiaryAccountId: p.beneficiaryId,
    reference: p.invoiceNumber,
    description: p.description
  }));

  const result = await initiateTransfer(token, accountId, transferList);

  // Store references for tracking
  for (const transfer of result.data) {
    await storeTransferRecord(transfer);
  }

  // Poll for confirmation (after 30 seconds)
  setTimeout(async () => {
    const txns = await getTransactions(token, accountId, today, today);
    const confirmed = txns.data.transactions.filter(
      t => t.reference && t.type === 'DEBIT'
    );

    notifyPaymentConfirmation(confirmed);
  }, 30000);
}
```

---

### Pattern 4: Cash Flow Forecasting

Analyze historical transactions to predict future cash positions:

1. `GET /za/pb/v1/accounts/{id}/transactions` — Fetch 6-12 months of history
2. Analyze transaction patterns (recurring payments, seasonal variations)
3. Build predictive model using amount, frequency, and timing
4. Generate cash flow forecasts
5. Alert on low balance predictions

**Implementation (JavaScript):**
```javascript
async function forecastCashFlow(accountId, months = 6) {
  const token = await getOAuthToken();
  const endDate = new Date();
  const startDate = new Date();
  startDate.setMonth(startDate.getMonth() - months);

  const txns = await getTransactions(
    token,
    accountId,
    startDate.toISOString().split('T')[0],
    endDate.toISOString().split('T')[0]
  );

  // Group by category and analyze patterns
  const patterns = analyzeTransactionPatterns(txns.data.transactions);

  // Generate forecast
  const forecast = generateForecast(patterns, months);

  // Alert if low balance predicted
  for (const [date, predictedBalance] of Object.entries(forecast)) {
    if (predictedBalance < 10000) { // Alert threshold
      alertLowBalance(date, predictedBalance);
    }
  }

  return forecast;
}
```

---

## Error Handling

Investec API returns standard HTTP status codes with error details in the response body.

### Common Error Responses

**400 Bad Request — Invalid Parameters**
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request is invalid",
    "details": "Missing required parameter: fromDate"
  }
}
```

**401 Unauthorized — Invalid Token**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or expired access token",
    "details": "Token has expired. Request a new token."
  }
}
```

**403 Forbidden — Insufficient Permissions**
```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions",
    "details": "You do not have permission to access this account"
  }
}
```

**404 Not Found**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Resource not found",
    "details": "Account ACC-999999999 does not exist"
  }
}
```

**429 Too Many Requests — Rate Limited**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "details": "Rate limit exceeded. Please retry after 60 seconds."
  }
}
```

**500 Server Error**
```json
{
  "error": {
    "code": "SERVER_ERROR",
    "message": "Internal server error",
    "details": "An unexpected error occurred. Please retry."
  }
}
```

### HTTP Status Codes Summary

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | Success | Process response normally |
| 400 | Bad Request | Check request parameters and format |
| 401 | Unauthorized | Token expired or invalid — request new token |
| 403 | Forbidden | Insufficient permissions — check account access |
| 404 | Not Found | Resource doesn't exist — verify IDs and paths |
| 429 | Rate Limited | Wait before retrying (exponential backoff) |
| 500 | Server Error | Retry with exponential backoff |

### Error Handling Pattern

```javascript
async function apiCall(method, endpoint, token, data = null) {
  try {
    const response = await fetch(`https://openapi.investec.com${endpoint}`, {
      method,
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: data ? JSON.stringify(data) : undefined
    });

    if (response.status === 401) {
      // Token expired — refresh and retry
      const newToken = await getOAuthToken();
      return apiCall(method, endpoint, newToken, data);
    }

    if (response.status === 429) {
      // Rate limited — exponential backoff
      const retryAfter = response.headers.get('Retry-After') || 60;
      await sleep(parseInt(retryAfter) * 1000);
      return apiCall(method, endpoint, token, data);
    }

    if (!response.ok) {
      const error = await response.json();
      throw new Error(`API Error: ${error.error.code} - ${error.error.message}`);
    }

    return await response.json();
  } catch (error) {
    console.error('API call failed:', error);
    // Implement retry logic with exponential backoff
    throw error;
  }
}
```

---

## Important Implementation Notes

### Token Management
- **Token Expiration:** Access tokens expire in **1 hour**. Cache tokens and implement refresh logic before expiry
- **Token Storage:** Never hardcode tokens. Regenerate before expiry or implement token refresh endpoints
- **Scope Control:** Request only the scopes your application needs (principle of least privilege)

### Date Handling
- **Required Parameters:** `fromDate` and `toDate` are required for transaction queries
- **ISO 8601 Format:** Use YYYY-MM-DD format for all date parameters
- **Historical Availability:** Transaction history is typically available for 12-24 months
- **Time Zones:** All timestamps use UTC (Zulu time with 'Z' suffix)

### Pagination
- **Default Limit:** 100 records per request
- **Maximum Limit:** 500 records per request
- **Offset:** Use `offset` parameter to skip records (e.g., offset=100 for second page)
- **Large Datasets:** Always use pagination for queries that may return many results

### Account Identification
- **Use accountId:** Always use `accountId` in API calls, not the account number
- **Unique Identifiers:** `accountId` is the unique identifier for an account
- **Account Number:** For display purposes only, not for API operations

### Rate Limiting
- **Limits:** Investec enforces rate limits (specific limits per endpoint vary)
- **Backoff Strategy:** Implement exponential backoff for retries
- **Headers:** Check response headers for rate limit information
- **Polling:** Recommended polling interval is 15-30 minutes for transaction monitoring
- **Frequency:** More frequent polling may hit rate limits

### Balance Types
- **Available Balance:** Use for authorization checks and cash flow decisions
- **Current Balance:** Use for accurate reporting and reconciliation
- **Pending Transactions:** Available balance excludes pending debits

### Transfer Operations
- **Processing Time:** Transfers typically process within minutes during business hours
- **Beneficiary Accounts:** Must be pre-registered or valid Investec accounts
- **Batch Processing:** Use `transfermultiple` for batch operations (multiple transfers in one request)
- **References:** Always provide reference codes for tracking and reconciliation
- **Verification:** Confirm transfers by querying transaction history after processing

### Security Best Practices
- **Credentials:** Store `client_id` and `client_secret` in environment variables
- **HTTPS Only:** All API communications use HTTPS (TLS 1.2+)
- **Token Security:** Treat access tokens like passwords — never expose in logs or URLs
- **PSD2 Compliance:** API implements Strong Customer Authentication (SCA) standards

---

## Sandbox vs. Production

Investec provides separate sandbox and production environments:

### Sandbox (Testing)
- **URL:** `https://openapisandbox.investec.com`
- **Purpose:** Development, testing, and integration validation
- **Credentials:** Use separate test client credentials
- **No Real Transactions:** All operations are simulated
- **Data:** Test with dummy accounts and transactions

### Production
- **URL:** `https://openapi.investec.com`
- **Purpose:** Live banking operations
- **Credentials:** Use production client credentials
- **Real Transactions:** Actual account access and money movement
- **Compliance:** Subject to all regulatory and security requirements

**Migration Steps:**
1. Complete testing and validation in sandbox
2. Register for production credentials at [developer.investec.com](https://developer.investec.com/)
3. Update API URLs and credentials
4. Test with limited amounts first
5. Deploy with monitoring and alerts

---

## Useful Links

- **Developer Portal:** https://developer.investec.com/
- **Programmable Banking:** https://developer.investec.com/programmable-banking/
- **Community Wiki:** https://investec.gitbook.io/programmable-banking-community-wiki/
- **Postman Collections:** https://www.postman.com/investec-open-api/programmable-banking/
- **GitHub Community:** https://github.com/Investec-Developer-Community/
- **Status Page:** https://status.investec.com/
- **Dashboard/Credentials:** https://login.investec.co.za/
