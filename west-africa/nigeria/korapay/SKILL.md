---
name: korapay
description: "Integrate with the Korapay pan-African payment infrastructure API for payment processing, disbursements, mobile money transfers, card issuing, and KYC verification. Use this skill whenever the user wants to initialize charges, process payments, handle disbursements, verify transaction status, manage payment flows, work with mobile money, process remittance payouts, issue virtual cards, or perform identity verification. Also trigger when the user mentions 'Korapay', 'pan-African payments', 'charge initialization', 'Korapay disbursement', 'mobile money transfer', 'remittance', 'card issuing', 'KYC verification', or needs payment processing with multi-currency and fast disbursement capabilities."
---

# Korapay Integration Skill

Korapay is a comprehensive pan-African payment infrastructure platform providing APIs for charge initialization, transaction processing, mobile money transfers, payouts, remittance services, virtual card issuing, and identity verification across Nigeria, Ghana, Kenya, and other African markets. It enables developers to accept payments and send money with minimal friction, high reliability, and support for multiple African currencies.

## When to use this skill

You're building a payment solution that needs to accept charges, process disbursements, send remittances, issue cards, or verify identities — an e-commerce platform, marketplace, B2B payment tool, fintech app, or cross-border money transfer service. Korapay handles payment collection, fund transfers, mobile money payments, and identity verification across Africa with a modern REST API.

## Authentication

All Korapay API requests require a Bearer token in the Authorization header:

```
Authorization: Bearer your_secret_key
```

Keys are generated in your Korapay merchant dashboard. Store in an environment variable like `KORAPAY_SECRET_KEY`. Never hardcode tokens.

**Base URL:** `https://api.korapay.com/merchant`

**Official Documentation:**
- Developer Portal: https://developers.korapay.com
- Full API Docs: https://docs.korapay.com

## Core API Reference

### Initialize a Charge

Start a payment transaction and receive a checkout link to share with customers. Korapay handles card, mobile money, bank transfer, and other payment methods.

```
POST /api/v1/charges/initialize
```

**Body:**
```json
{
  "amount": 50000,
  "currency": "NGN",
  "reference": "CHG-2025-001",
  "email": "customer@example.com",
  "description": "Payment for digital services",
  "checkout": {
    "text": "Complete your purchase",
    "logo": "https://yoursite.com/logo.png"
  },
  "metadata": {
    "order_id": "ORD-12345",
    "customer_name": "Amina Okafor"
  }
}
```

**Important:** Amount is in the base unit of the currency (NGN, GHS, KES, etc.). Reference must be unique and is used for idempotency — sending the same reference twice returns the original charge.

**Response:**
```json
{
  "status": true,
  "message": "Charge created successfully",
  "data": {
    "reference": "CHG-2025-001",
    "charge_id": "CRG_xxxxx",
    "checkout_url": "https://checkout.korapay.com/xxxxx",
    "amount": 50000,
    "currency": "NGN",
    "status": "pending"
  }
}
```

Share `checkout_url` with the customer. After payment, they're redirected to your callback or you listen for webhooks.

### Get Charge Status

Retrieve the status and details of a charge.

```
GET /api/v1/charges/{reference}
```

**Path Parameters:**
- `reference`: The charge reference ID you provided

**Response:**
```json
{
  "status": true,
  "message": "Charge retrieved",
  "data": {
    "reference": "CHG-2025-001",
    "charge_id": "CRG_xxxxx",
    "amount": 50000,
    "currency": "NGN",
    "status": "successful",
    "paid_at": "2025-02-15T14:30:00Z",
    "customer_email": "customer@example.com",
    "payment_method": "card",
    "metadata": {
      "order_id": "ORD-12345"
    }
  }
}
```

Check `status` field: `pending`, `successful`, or `failed`. Always verify server-side — never trust client-side callbacks alone.

### Mobile Money Transfer

Process payments via mobile money (MTN, Airtel, Vodafone, Safaricom, etc.).

```
POST /api/v1/charges/mobile-money
```

**Body:**
```json
{
  "amount": 50000,
  "currency": "NGN",
  "reference": "MM-2025-001",
  "phone_number": "2348012345678",
  "email": "customer@example.com",
  "description": "Mobile money payment"
}
```

**Response:** Returns charge reference and payment initiation status. Customer confirms on their mobile device.

### Initialize a Disbursement

Send money to a bank account (fast transfer). Requires positive account balance.

```
POST /api/v1/transactions/disburse
```

**Body:**
```json
{
  "reference": "DIS-2025-001",
  "amount": 100000,
  "currency": "NGN",
  "recipient": {
    "bank_account": "0123456789",
    "bank_code": "058",
    "account_name": "Chinedu Adeyemi"
  },
  "narration": "Payout for contract work",
  "metadata": {
    "worker_id": "WRK-001"
  }
}
```

**Important:** Amount is in the base currency unit. Bank code is the 3-digit code (058 = GTB). Reference must be unique — duplicates are rejected or return cached result.

**Response:**
```json
{
  "status": true,
  "message": "Disbursement initiated",
  "data": {
    "reference": "DIS-2025-001",
    "transaction_id": "TXN_xxxxx",
    "amount": 100000,
    "currency": "NGN",
    "recipient_bank_code": "058",
    "recipient_account": "0123456789",
    "status": "processing"
  }
}
```

### Remittance Payout

Send international remittances and cross-border payouts using the dedicated remittance API.

```
POST /api/v1/transactions/disburse/remittance
```

**Body:**
```json
{
  "reference": "REM-2025-001",
  "amount": 100000,
  "currency": "NGN",
  "recipient": {
    "bank_account": "0123456789",
    "bank_code": "058",
    "country_code": "NG"
  },
  "narration": "Family remittance"
}
```

Remittance payouts support cross-border transfers with optimized processing.

### Get Transaction Status

Check the status of a disbursement or remittance transaction.

```
GET /api/v1/transactions/{reference}
```

**Response:**
```json
{
  "status": true,
  "message": "Transaction retrieved",
  "data": {
    "reference": "DIS-2025-001",
    "transaction_id": "TXN_xxxxx",
    "type": "disbursement",
    "amount": 100000,
    "currency": "NGN",
    "status": "successful",
    "completed_at": "2025-02-15T14:35:00Z",
    "recipient_account": "0123456789"
  }
}
```

Possible statuses: `processing`, `successful`, `failed`. Most disbursements complete within 1-5 minutes.

### Payout History

Retrieve payout transaction history and reconciliation data.

```
GET /api/v1/transactions?limit=50&offset=0&status=successful&from=2025-01-01&to=2025-02-28
```

**Query Parameters:**
- `limit`: Max results (default 50, max 100)
- `offset`: Pagination offset
- `status`: Filter by `successful`, `failed`, or `processing`
- `from`: Start date (ISO format)
- `to`: End date (ISO format)

### List Banks API

Retrieve valid bank codes for disbursements.

```
GET /api/v1/banks
```

Returns a list of all banks with their codes and names for the specified country.

### List Mobile Money Operators API

Retrieve valid mobile money operator codes.

```
GET /api/v1/mobile-money-operators
```

Returns a list of all MMO providers (MTN, Airtel, Vodafone, Safaricom, etc.) with their codes.

### Verify Bank Account

Validate a bank account before disbursing to confirm the account name.

```
GET /api/v1/transactions/verify-account?bank_code=058&account_number=0123456789
```

**Query Parameters:**
- `bank_code`: 3-digit bank code
- `account_number`: 10-digit NUBAN account number

**Response:**
```json
{
  "status": true,
  "message": "Account verified",
  "data": {
    "bank_code": "058",
    "account_number": "0123456789",
    "account_name": "Chinedu Adeyemi",
    "valid": true
  }
}
```

Always verify before disbursing to prevent sending money to wrong accounts.

### Card Issuing

Issue virtual payment cards for customers (requires Card Issuing integration).

```
POST /api/v1/cards/issue
```

Enables virtual card creation for payouts, subscriptions, and marketplace payments.

### Identity Verification API

Perform KYC/KYB verification and identity checks.

```
POST /api/v1/identity/verify
```

Supports identity verification, liveness checks, and business verification across African markets.

## Supported Currencies

Korapay supports multiple African currencies:
- NGN (Nigerian Naira)
- GHS (Ghanaian Cedi)
- KES (Kenyan Shilling)
- And other African currencies based on market expansion

## Webhooks

Korapay sends webhooks for charge and disbursement events. Verify and listen:

```javascript
const crypto = require('crypto');
const hash = crypto
  .createHmac('sha256', webhook_secret)
  .update(JSON.stringify(req.body))
  .digest('hex');

if (hash === req.headers['x-korapay-signature']) {
  // Valid webhook
}
```

Key events: `charge.success`, `charge.failed`, `disbursement.success`, `disbursement.failed`, `transaction.completed`.

## Common Integration Patterns

### Payment collection flow
1. `POST /api/v1/charges/initialize` when customer is ready to pay
2. Share `checkout_url` with customer
3. Customer completes payment on Korapay checkout
4. Listen for `charge.success` webhook or `GET /charges/{reference}` to verify
5. Fulfill order after confirming payment

### Marketplace seller payouts
1. Collect payments via charges
2. After receiving `charge.success` webhook
3. `GET /api/v1/transactions/verify-account` to validate seller bank account
4. `POST /api/v1/transactions/disburse` to send seller's cut
5. Track all disbursements via `GET /api/v1/transactions/{reference}`
6. Reconcile daily via `GET /api/v1/transactions` with date filters

### Mobile money payment
1. Collect customer phone number and amount
2. `POST /api/v1/charges/mobile-money` to initiate transfer
3. Customer confirms on their device
4. Listen for webhook confirmation
5. Verify via `GET /api/v1/charges/{reference}`

### Batch disbursement (invoice payouts)
1. Collect approved invoices with recipient details
2. For each invoice, `GET /api/v1/transactions/verify-account` (cache results)
3. `POST /api/v1/transactions/disburse` for each approved recipient
4. Store all references in your database
5. Poll `GET /api/v1/transactions` with status filter
6. Send confirmation emails once status is `successful`

## Error Handling

Korapay returns structured error responses:

```json
{
  "status": false,
  "message": "Description of error",
  "code": "ERROR_CODE"
}
```

Common errors:
- **400**: Validation error (invalid bank code, bad amount, missing fields)
- **401**: Invalid or missing Bearer token
- **404**: Resource not found (reference doesn't exist)
- **429**: Rate limited — back off and retry
- **500**: Server error — retry with exponential backoff

For disbursements, failed status might indicate invalid account, insufficient balance, or network issues.

## Important Notes and Gotchas

- **Currency units:** Amounts are in base units of the currency (NGN, GHS, KES).
- **Bank codes:** Use correct 3-digit bank codes. Invalid codes cause disbursement failures.
- **Account numbers:** Must be exactly 10 digits (NUBAN format) for Nigerian accounts.
- **Idempotent references:** Use unique references per transaction. Same reference returns cached result within 24 hours.
- **Account verification:** Always verify destination accounts before disbursing to prevent failed transfers.
- **Processing time:** Disbursements usually complete in 1-5 minutes but can take longer during off-hours.
- **Webhook timing:** Webhooks may arrive after API response. Poll if webhook is critical for your flow.
- **Metadata:** Use metadata for custom fields — they're returned in transaction details for reconciliation.
- **PCI DSS:** For direct card processing, ensure PCI DSS Level 1 certification.
- **Balance:** Verify your merchant account has sufficient balance before initiating payouts.

## Useful Links

- API Documentation: https://docs.korapay.com
- Developer Portal: https://developers.korapay.com
- Dashboard: https://dashboard.korapay.com
- Sandbox Testing: Available in test environment with test credentials
