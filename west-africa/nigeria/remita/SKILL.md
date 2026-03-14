---
name: remita
description: "Integrate with the Remita payment API for Nigerian billing, salary payments, collections, and transfers. Use this skill whenever the user wants to process bulk salary payments, handle billing payments, manage collections, initiate interbank transfers, generate payment reference numbers, or work with Remita's payment infrastructure. Also trigger when the user mentions 'Remita', 'Nigerian salary payments', 'bulk payroll', 'collections API', 'agency banking', 'bill payments Nigeria', or needs to process government or enterprise payments."
---

# Remita Integration Skill

Remita is Nigeria's leading enterprise payment infrastructure, operated by SystemSpecs. It powers government salary payments (processing payroll for all Nigerian federal government employees), tax collections, and large-scale billing across the country. It serves as the backbone for Treasury Single Account (TSA) collections in Nigeria, making it the go-to platform for government and enterprise payments.

## When to use this skill

You're building a payroll system, a billing platform, an agency banking solution, or a collections management tool that needs to process payments at scale in Nigeria. Remita handles salary bulk disbursements, interbank transfers, payment collections, bill payments, airtime/data vending, and payment reference number generation — essential for HR platforms, government integrations, fintech apps, and enterprise payment systems.

## Authentication

Remita uses JWT and OAuth 2.0 for API authentication. All API requests require an access token in the Authorization header along with your API key:

```
Authorization: Bearer {access_token}
Content-Type: application/json
API-Key: {your_api_key}
```

Obtain credentials from your Remita merchant dashboard. Store them in environment variables like `REMITA_API_KEY` and `REMITA_ACCESS_TOKEN`. Never hardcode credentials.

**Base URL:** `https://api.remita.net/api/v1`
**Sandbox URL:** `https://demo.remita.net/api/v1`

**Token Generation:**

```
POST /auth/token
```

```json
{
  "username": "your_merchant_username",
  "password": "your_merchant_password"
}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600
  }
}
```

## Core API Reference

### Initiate a Payment

Create a payment transaction and receive a redirect URL or reference number. The payment initiation endpoint creates a transaction and returns a checkout URL.

```
POST /payments/initiate
```

**Body:**
```json
{
  "merchantId": "your_merchant_id",
  "serviceTypeId": "your_service_type_id",
  "orderId": "ORD-2025-0001",
  "amount": 50000,
  "payerName": "Amina Okafor",
  "payerEmail": "amina@example.com",
  "payerPhone": "08012345678",
  "description": "Invoice payment for consulting services",
  "currency": "NGN",
  "responseurl": "https://yoursite.com/payment/callback"
}
```

**Important:** Amount is in **naira** (whole units). ₦500 = 500. The `orderId` must be unique per transaction. The `serviceTypeId` identifies the specific service/product being paid for and is configured on your Remita dashboard.

**Response:**
```json
{
  "status": "success",
  "data": {
    "rrr": "310007769514",
    "transactionId": "TXN_xxxxx",
    "authorizationUrl": "https://remita.net/pay/310007769514",
    "merchantId": "your_merchant_id",
    "amount": 50000
  }
}
```

The **RRR (Remita Retrieval Reference)** is Remita's unique transaction identifier. Redirect the customer to `authorizationUrl` or use the RRR for offline payment at any bank or Remita partner location.

### Check Payment Status

Verify the status of a payment using the RRR.

```
GET /payments/status/{rrr}
```

**Response (success):**
```json
{
  "status": "success",
  "data": {
    "rrr": "310007769514",
    "orderId": "ORD-2025-0001",
    "paymentStatus": "00",
    "paymentStatusDescription": "Successful",
    "amount": 50000,
    "currency": "NGN",
    "paymentDate": "2025-02-15T14:30:00Z",
    "transactionId": "TXN_xxxxx",
    "payerName": "Amina Okafor",
    "payerEmail": "amina@example.com"
  }
}
```

Always verify server-side after payment. `paymentStatus === "00"` means successful. Check `amount` matches your expected value.

### Initiate Single Interbank Transfer

Transfer funds from your Remita wallet to any Nigerian bank account.

```
POST /transfers/single
```

**Body:**
```json
{
  "amount": 100000,
  "transactionRef": "TRF-2025-0001",
  "destinationAccountNumber": "0123456789",
  "destinationBankCode": "058",
  "destinationAccountName": "Chinedu Adeyemi",
  "sourceAccountNumber": "your_remita_account",
  "narration": "Vendor payment for February"
}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "transactionRef": "TRF-2025-0001",
    "transactionId": "TXN_xxxxx",
    "status": "PROCESSING",
    "amount": 100000,
    "destinationBank": "GTBank"
  }
}
```

### Initiate Bulk Transfer (Payroll)

Send bulk payments to multiple recipients in a single request — ideal for salary disbursements.

```
POST /transfers/bulk
```

**Body:**
```json
{
  "batchRef": "PAYROLL-2025-FEB",
  "totalAmount": 5000000,
  "sourceAccount": "your_remita_account",
  "narration": "February 2025 Salary Disbursement",
  "transactions": [
    {
      "amount": 500000,
      "transactionRef": "SAL-EMP001-FEB",
      "destinationAccountNumber": "0123456789",
      "destinationBankCode": "033",
      "destinationAccountName": "Amina Okafor",
      "narration": "February 2025 Salary"
    },
    {
      "amount": 500000,
      "transactionRef": "SAL-EMP002-FEB",
      "destinationAccountNumber": "9876543210",
      "destinationBankCode": "044",
      "destinationAccountName": "Chinedu Adeyemi",
      "narration": "February 2025 Salary"
    }
  ]
}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "batchRef": "PAYROLL-2025-FEB",
    "batchId": "BATCH_xxxxx",
    "totalRecords": 2,
    "totalAmount": 5000000,
    "status": "PROCESSING"
  }
}
```

Processing is asynchronous. Use the `batchRef` to track status.

### Get Bulk Transfer Status

```
GET /transfers/bulk/{batchRef}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "batchRef": "PAYROLL-2025-FEB",
    "status": "COMPLETED",
    "totalRecords": 2,
    "successful": 2,
    "failed": 0,
    "transactions": [
      {
        "transactionRef": "SAL-EMP001-FEB",
        "amount": 500000,
        "status": "SUCCESSFUL",
        "destinationAccountName": "Amina Okafor"
      }
    ]
  }
}
```

### Generate Payment Reference Number (RRR)

Generate a reference number that customers can use to pay at any FawryPay-style retail location, bank branch, or POS terminal across Nigeria.

```
POST /payments/reference
```

**Body:**
```json
{
  "merchantId": "your_merchant_id",
  "serviceTypeId": "your_service_type_id",
  "orderId": "BILL-2025-0001",
  "amount": 25000,
  "payerName": "Fatima Ibrahim",
  "payerEmail": "fatima@example.com",
  "payerPhone": "08098765432",
  "description": "Utility bill payment"
}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "rrr": "290007654321",
    "orderId": "BILL-2025-0001",
    "amount": 25000
  }
}
```

The customer takes this RRR to any bank or Remita agent location to make payment. Status updates are sent via webhook.

### Bill Payment (VAS)

Purchase airtime, data, electricity, and other utilities through Remita's vending API.

```
POST /vas/purchase
```

**Body:**
```json
{
  "serviceId": "AIRTIME_MTN",
  "amount": 1000,
  "recipientPhone": "08012345678",
  "transactionRef": "VAS-2025-0001"
}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "transactionRef": "VAS-2025-0001",
    "serviceId": "AIRTIME_MTN",
    "amount": 1000,
    "status": "DELIVERED",
    "token": null
  }
}
```

For electricity payments, the response includes a `token` field containing the meter recharge token.

### Get Available Services (VAS)

```
GET /vas/services
```

Returns available bill payment categories and services: airtime, data, electricity, cable TV, internet, and more.

## Webhooks

Remita sends webhook notifications for payment and transfer events. Configure your webhook URL on the Remita dashboard.

**Webhook Payload:**
```json
{
  "event": "payment.successful",
  "data": {
    "rrr": "310007769514",
    "orderId": "ORD-2025-0001",
    "amount": 50000,
    "paymentDate": "2025-02-15T14:30:00Z",
    "paymentStatus": "00"
  }
}
```

**Verify webhook signature:**
```javascript
const crypto = require('crypto');
const hash = crypto.createHmac('sha256', process.env.REMITA_WEBHOOK_SECRET)
  .update(JSON.stringify(req.body))
  .digest('hex');

if (hash === req.headers['x-remita-signature']) {
  // Valid webhook — process the event
  const { event, data } = req.body;
  if (event === 'payment.successful' && data.paymentStatus === '00') {
    // Fulfill order
  }
}
```

Key events: `payment.successful`, `payment.failed`, `payment.cancelled`, `transfer.successful`, `transfer.failed`, `bulk.completed`, `bulk.partial`.

## Common Integration Patterns

### Enterprise payment collection
1. Configure service type on Remita dashboard
2. `POST /payments/initiate` with customer and service details
3. Receive RRR — customer can pay online, at bank, or via POS
4. `GET /payments/status/{rrr}` to verify, or listen for `payment.successful` webhook
5. Reconcile collections in your system

### Bulk payroll disbursement
1. Prepare employee list with verified bank details (10-digit NUBAN accounts)
2. `POST /transfers/bulk` with all employees and a unique `batchRef`
3. Poll `GET /transfers/bulk/{batchRef}` for completion
4. Listen for `bulk.completed` webhook
5. Handle `failed` transactions individually — retry or escalate
6. Generate payroll reports for HR

### Agency banking / bill payments
1. `GET /vas/services` to list available services
2. Collect customer details and service selection
3. `POST /vas/purchase` to execute the transaction
4. Confirm delivery status in response
5. Store transaction ref for reconciliation

### Offline payment with reference number
1. `POST /payments/reference` to generate RRR
2. Display RRR to customer (print receipt, send SMS)
3. Customer pays at any Nigerian bank or Remita agent
4. Webhook notifies your system when payment is confirmed
5. `GET /payments/status/{rrr}` for manual verification

## Error Handling

Remita returns structured error responses:

```json
{
  "status": "error",
  "message": "Invalid merchant ID",
  "code": "AUTH_001"
}
```

Common errors:
- **400**: Validation error — missing required fields, invalid bank codes, malformed request
- **401**: Authentication failure — invalid or expired token, missing API key
- **403**: Forbidden — insufficient permissions for the requested operation
- **404**: Resource not found — invalid RRR, unknown batch reference
- **409**: Conflict — duplicate `orderId` or `transactionRef`
- **429**: Rate limited — implement exponential backoff (start at 1s, max 60s)
- **500**: Server error — retry with backoff, escalate if persistent

For bulk operations, always check individual transaction statuses even if the batch succeeds. Some transactions in a batch may fail while others succeed.

## Important Notes and Gotchas

- **RRR is king:** The Remita Retrieval Reference is the primary identifier for all payment transactions. Store it alongside your internal order ID.
- **Bank codes:** Use standard Nigerian bank codes (033 = First Bank, 044 = Access Bank, 058 = GTBank, 057 = Zenith, 011 = First Bank). Use the bank lookup endpoint to get current codes.
- **Account numbers:** Must be exactly 10 digits for Nigerian NUBAN accounts. Validate before submission.
- **Treasury Single Account (TSA):** Remita is the official platform for Nigerian government TSA collections. If you're building government payment integrations, Remita is mandatory.
- **Bulk processing:** Batches are processed asynchronously. Individual transactions may settle at different times. Never assume all-or-nothing completion.
- **Idempotency:** Use unique `orderId` and `transactionRef` values. Duplicate submissions are rejected with a 409 error.
- **Amount format:** Amounts are in whole naira (not kobo). ₦5,000 = 5000.
- **Settlement timing:** Collections typically settle T+1 (next business day). Transfers settle same-day for most Nigerian banks.
- **Sandbox vs Production:** Always test in sandbox (`demo.remita.net`) before going live. Sandbox uses different credentials.

## Useful Links

- API Documentation: https://api.remita.net
- Developer Center: https://remita.net/developers
- Sandbox Environment: https://demo.remita.net
- Merchant Dashboard: https://login.remita.net
- Interbank Transfer Guide: https://blog.remita.net/mastering-the-remita-interbank-transfer-api-a-comprehensive-guide/
- GitHub SDKs: https://github.com/RemitaNet
