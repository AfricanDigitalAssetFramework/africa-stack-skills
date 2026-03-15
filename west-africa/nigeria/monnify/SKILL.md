---
name: monnify
description: "Integrate with the Monnify payment API (by Moniepoint) for Nigerian collections and disbursements. Use this skill whenever the user wants to initialize transactions, process collections, handle disbursements, manage merchants, handle OTP authentication, or work with Monnify's collections and payout services. Also trigger when the user mentions 'Monnify', 'Moniepoint collections', 'payment collections Nigeria', 'merchant API', or needs to build payment collection and disbursement systems."
---

# Monnify Integration Skill

Monnify (by Moniepoint) is a comprehensive payments platform delivering payment collections, disbursements, invoicing, and settlement solutions for Nigeria. It provides enterprise-grade APIs for merchants to accept payments from customers and send money to bank accounts with high reliability, competitive rates, and full reconciliation capabilities.

## When to Use This Skill

You're building payment collection systems, billing platforms, invoicing systems, marketplace payout systems, or financial applications that require accepting payments and disbursing funds in Nigeria. Monnify handles merchant onboarding, payment collections, customer reserve accounts, bulk disbursements, split payments, recurring billing, and full reconciliation.

## API Environments

- **Sandbox:** https://sandbox.monnify.com (testing)
- **Live:** https://api.monnify.com (production)
- **Official Documentation:** developers.monnify.com
- **API Reference:** developers.monnify.com/api

## Authentication

Monnify uses OAuth 2.0 Bearer tokens for API requests. Obtain a token via your API credentials (API Key and Secret), generated in your Monnify dashboard.

**Step 1: Get Bearer Token**

```
POST https://api.monnify.com/api/v1/auth/login
Authorization: Basic base64(api_key:secret_key)
```

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Authentication successful",
  "responseCode": "0",
  "responseBody": {
    "accessToken": "your_bearer_token",
    "expiresIn": 1800
  }
}
```

Tokens expire in 30 minutes. Cache tokens and refresh before expiry.

**Step 2: Use Bearer Token**

Include the token in all subsequent requests:

```
Authorization: Bearer your_bearer_token
```

Store API Key and Secret in environment variables: `MONNIFY_API_KEY` and `MONNIFY_API_SECRET`. Never hardcode credentials.

## Core Endpoints

### Initiate Payment Transaction

Create a payment collection request. The customer completes payment via Monnify's hosted checkout.

```
POST /api/v1/merchant/transactions/init-transaction
Content-Type: application/json
Authorization: Bearer your_bearer_token
```

**Body:**
```json
{
  "amount": 50000,
  "customerName": "Amina Okafor",
  "customerEmail": "customer@example.com",
  "paymentReference": "PAY-2025-001",
  "paymentDescription": "Invoice payment for services",
  "currencyCode": "NGN",
  "contractCode": "YOUR_CONTRACT_CODE",
  "redirectUrl": "https://yoursite.com/payment/callback"
}
```

**Critical:** Amount is in **kobo** (1 NGN = 100 kobo). ₦500 = 50000 kobo.

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Transaction initiated",
  "responseCode": "0",
  "responseBody": {
    "transactionReference": "TXN_xxxxx",
    "paymentReference": "PAY-2025-001",
    "checkoutUrl": "https://checkout.monnify.com/pay/xxxxx",
    "accessCode": "xxxxx",
    "amount": 50000,
    "status": "PENDING"
  }
}
```

Redirect customers to `checkoutUrl`. After payment, they return to `redirectUrl`.

### Verify Transaction

Check payment status using the payment reference.

```
GET /api/v1/merchant/transactions/query?paymentReference=PAY-2025-001
Authorization: Bearer your_bearer_token
```

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Retrieval successful",
  "responseCode": "0",
  "responseBody": {
    "transactionReference": "TXN_xxxxx",
    "paymentReference": "PAY-2025-001",
    "amount": 50000,
    "status": "PAID",
    "paidOn": "2025-02-15T14:30:00+01:00",
    "paymentMethod": "card",
    "customer": {
      "name": "Amina Okafor",
      "email": "customer@example.com"
    }
  }
}
```

Status values: `PENDING`, `PAID`, `FAILED`, `CANCELLED`. Always verify server-side after payment.

### Create Customer Reserved Account

Generate a permanent personalized account number for a customer to receive transfers directly.

```
POST /api/v2/bank-transfer/reserved-accounts
Content-Type: application/json
Authorization: Bearer your_bearer_token
```

**Body:**
```json
{
  "accountReference": "CUST-12345",
  "accountName": "Amina Okafor",
  "currencyCode": "NGN",
  "contractCode": "YOUR_CONTRACT_CODE",
  "customerEmail": "customer@example.com",
  "customerName": "Amina Okafor"
}
```

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Reserved account created",
  "responseCode": "0",
  "responseBody": {
    "accountReference": "CUST-12345",
    "accountNumber": "5900001234",
    "accountName": "Amina Okafor",
    "bankName": "Monnify",
    "currencyCode": "NGN",
    "status": "ACTIVE"
  }
}
```

Customer can transfer directly to this account number. Settlements appear in your merchant account.

### Initiate Single Disbursement

Send money to a single bank account.

```
POST /api/v2/disbursements/single
Content-Type: application/json
Authorization: Bearer your_bearer_token
```

**Body:**
```json
{
  "amount": 100000,
  "reference": "DIS-2025-001",
  "narration": "Payout for services rendered",
  "destinationAccountNumber": "0123456789",
  "destinationBankCode": "058",
  "destinationAccountName": "Chinedu Adeyemi",
  "currency": "NGN"
}
```

**Critical:** Amount in kobo. Bank code is 3-digit Nigerian bank code. Reference must be unique.

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Disbursement request processed",
  "responseCode": "0",
  "responseBody": {
    "reference": "DIS-2025-001",
    "status": "PENDING",
    "disbursementCode": "DIS_xxxxx",
    "amount": 100000,
    "currency": "NGN",
    "narration": "Payout for services rendered"
  }
}
```

Disbursement is queued for processing. Most complete within minutes.

### Bulk Disbursement

Send money to multiple accounts in a single batch.

```
POST /api/v1/disbursements/initiate-transfer-bulk
Content-Type: application/json
Authorization: Bearer your_bearer_token
```

**Body:**
```json
{
  "batchReference": "BULK-2025-001",
  "sourceAccountNumber": "YOUR_SETTLEMENT_ACCOUNT",
  "title": "February Payroll",
  "narration": "Monthly salary",
  "incomeSplitConfig": [],
  "disbursements": [
    {
      "amount": 150000,
      "reference": "DIS-001",
      "narration": "Salary - Employee 1",
      "destinationAccountNumber": "0123456789",
      "destinationBankCode": "058"
    },
    {
      "amount": 200000,
      "reference": "DIS-002",
      "narration": "Salary - Employee 2",
      "destinationAccountNumber": "9876543210",
      "destinationBankCode": "050"
    }
  ]
}
```

Returns batch reference for tracking all disbursements.

### Query Disbursement Status

Retrieve disbursement status using the reference.

```
GET /api/v1/disbursements/query?reference=DIS-2025-001
Authorization: Bearer your_bearer_token
```

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Retrieval successful",
  "responseCode": "0",
  "responseBody": {
    "reference": "DIS-2025-001",
    "disbursementCode": "DIS_xxxxx",
    "amount": 100000,
    "currency": "NGN",
    "status": "SUCCESSFUL",
    "completedOn": "2025-02-15T14:35:00+01:00",
    "destinationAccountNumber": "0123456789",
    "destinationBankCode": "058"
  }
}
```

Status values: `PENDING`, `SUCCESSFUL`, `FAILED`.

### Validate Bank Account

Verify a bank account before disbursing.

```
GET /api/v1/disbursements/account/validate?accountNumber=0123456789&bankCode=058
Authorization: Bearer your_bearer_token
```

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Account resolved",
  "responseCode": "0",
  "responseBody": {
    "accountNumber": "0123456789",
    "accountName": "Chinedu Adeyemi",
    "bankCode": "058",
    "bankName": "Guaranty Trust Bank"
  }
}
```

Always validate before disbursing to prevent sending money to wrong recipients.

### List Transactions

Retrieve transaction history for reconciliation.

```
GET /api/v1/merchant/transactions/list?pageSize=50&pageNo=0&startDate=2025-01-01&endDate=2025-02-28
Authorization: Bearer your_bearer_token
```

**Query Parameters:**
- `pageSize`: Results per page (max 100)
- `pageNo`: Page number (0-indexed)
- `startDate`: ISO date format (YYYY-MM-DD)
- `endDate`: ISO date format (YYYY-MM-DD)

**Response:**
```json
{
  "requestSuccessful": true,
  "responseMessage": "Transaction list retrieved",
  "responseCode": "0",
  "responseBody": {
    "pageNo": 0,
    "pageSize": 50,
    "total": 25,
    "content": [
      {
        "transactionReference": "TXN_xxxxx",
        "paymentReference": "PAY-2025-001",
        "amount": 50000,
        "status": "PAID",
        "paidOn": "2025-02-15T14:30:00+01:00"
      }
    ]
  }
}
```

## Additional Features

**Customer Limit Profiles:** Set transaction limits for customer reserved accounts via limit profile APIs.

**BVN/NIN Linking:** Link Bank Verification Number or National Identification Number to maximize transaction limits.

**Invoicing:** Generate virtual account-linked invoices for customers to pay via transfer.

**Recurring Payments:** Set up recurring payment schedules for subscriptions.

**Direct Debits:** Authorize and collect recurring payments directly from customer accounts.

**Transaction Splitting:** Configure `incomeSplitConfig` to split payments with sub-accounts and affiliates.

**Refunds:** Process refunds for transactions via the refunds API.

**Bills Payment:** Unified API for airtime, data, electricity, and cable TV payments.

**Payment Links:** Generate shareable payment links for customers.

**Wallet Balance API:** Check merchant wallet and settlement account balances.

**Verification APIs:** Verify customer identities and account information.

**Offline Payments:** Accept offline payment references and complete settlement later.

## Webhooks

Monnify sends webhooks for transaction and disbursement events. Verify signatures:

```javascript
const crypto = require('crypto');
const hash = crypto
  .createHmac('sha512', webhook_secret)
  .update(JSON.stringify(req.body))
  .digest('hex');

if (hash === req.headers['monnify-signature']) {
  // Valid webhook — process event
}
```

Key events: `SUCCESSFUL_TRANSACTION`, `FAILED_TRANSACTION`, `DISBURSEMENT_SUCCESSFUL`, `DISBURSEMENT_FAILED`.

## Common Integration Patterns

### Payment Collection Flow
1. Call `POST /api/v1/merchant/transactions/init-transaction`
2. Share `checkoutUrl` with customer
3. Customer pays on Monnify checkout
4. Receive webhook or poll `GET /api/v1/merchant/transactions/query`
5. Confirm status is `PAID`
6. Deliver goods/services

### Marketplace Seller Payouts
1. Collect payments from buyers via payment collection
2. After confirming `PAID` status
3. Call `GET /api/v1/disbursements/account/validate` for seller account
4. Call `POST /api/v2/disbursements/single` to send seller's cut
5. Monitor `DISBURSEMENT_SUCCESSFUL` webhook
6. Send seller payout notification

### Batch Payroll Disbursement
1. Prepare employee list with bank accounts
2. For each employee, validate account first
3. Call `POST /api/v2/disbursements/single` for each payout
4. Store disbursement references
5. Poll `GET /api/v1/disbursements/query` or listen for webhooks
6. Generate reconciliation report once all complete

## Error Handling

Monnify returns structured error responses:

```json
{
  "requestSuccessful": false,
  "responseMessage": "Description of error",
  "responseCode": "xxx"
}
```

Common response codes:
- **0:** Success
- **01:** Invalid authentication credentials
- **02:** Invalid request format
- **03:** Transaction not found
- **04:** Account validation failed
- **99:** Server error — retry with exponential backoff

Check `requestSuccessful` flag first, then examine `responseCode` for error details.

## Reserved Accounts (Virtual Accounts for Business Banking)

Reserved accounts are dedicated bank account numbers assigned to a specific customer — any payment into that account is automatically matched to the customer. Widely used in Nigeria for business banking integrations (e.g. every user gets their own bank account number for deposits).

### Create a Reserved Account
```
POST /api/v1/bank-transfer/reserved-accounts
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "accountReference": "unique_ref_per_customer",
  "accountName": "Adaobi Okafor",
  "currencyCode": "NGN",
  "contractCode": "YOUR_CONTRACT_CODE",
  "customerEmail": "adaobi@example.com",
  "customerName": "Adaobi Okafor",
  "bvn": "22222222222",
  "getAllAvailableBanks": false,
  "preferredBanks": ["035"]
}
```
`contractCode` is obtained from your Monnify dashboard. `preferredBanks` accepts bank codes — e.g. Wema Bank (035) is commonly used for virtual accounts.

### Get Reserved Account Details
```
GET /api/v1/bank-transfer/reserved-accounts/{accountReference}
Authorization: Bearer {access_token}
```
Returns the assigned account number(s) and bank(s). Use this to show the customer their dedicated account number.

### List Transactions on a Reserved Account
```
GET /api/v1/bank-transfer/reserved-accounts/transactions?accountReference={ref}&page=0&size=20
Authorization: Bearer {access_token}
```

Monnify sends a webhook event (`SUCCESSFUL_TRANSACTION`) when funds are received into a reserved account. Always use webhook + server-side verification rather than polling.

---

## Split Payment (Marketplace / Sub-Merchant)

Split payments allow you to automatically distribute incoming payments between multiple accounts — useful for marketplaces where a platform fee is deducted before the seller receives their share.

### Create a Payment with Splits
```
POST /api/v1/merchant/transactions/init-transaction
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "amount": 100000,
  "customerName": "Buyer Name",
  "customerEmail": "buyer@example.com",
  "paymentReference": "unique_ref_" + timestamp,
  "paymentDescription": "Order payment",
  "currencyCode": "NGN",
  "contractCode": "YOUR_CONTRACT_CODE",
  "redirectUrl": "https://yourapp.com/payment-complete",
  "paymentMethods": ["CARD", "ACCOUNT_TRANSFER"],
  "incomeSplitConfig": [
    {
      "subAccountCode": "MFY_SUB_xxxxx",   // sub-account code from dashboard
      "feePercentage": 10,                  // seller gets 10%
      "splitPercentage": 90,               // or use splitAmount instead
      "feeBearer": false
    }
  ]
}
```
Sub-account codes are generated in the Monnify dashboard for each merchant/seller. The platform retains the remainder after splits.

## Best Practices

- **Kobo Units:** All amounts in kobo. ₦500 = 50000 kobo. Critical for correct payment processing.
- **Token Management:** Bearer tokens expire in 30 minutes. Cache and refresh before expiry.
- **Bank Codes:** Use valid 3-digit Nigerian bank codes only.
- **Account Validation:** Always validate destination accounts before disbursing.
- **Unique References:** Use unique references per transaction. Duplicates may be rejected.
- **Webhook Verification:** Always verify signatures — never process unverified webhooks.
- **Idempotency:** Use consistent references to handle retries safely.
- **Rate Limiting:** Implement backoff strategies for API rate limits.
- **Reconciliation:** Maintain transaction logs for audit and reconciliation.

## Useful Resources

- **API Documentation:** https://developers.monnify.com
- **API Reference:** https://developers.monnify.com/api
- **Dashboard:** https://dashboard.monnify.com
- **Sandbox Testing:** Available in sandbox environment
- **SDKs & Plugins:** Official SDKs and plugins available for various platforms
