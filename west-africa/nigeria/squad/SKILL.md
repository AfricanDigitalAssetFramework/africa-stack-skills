---
name: squad
description: "Integrate with the Squad payment gateway API (by Habaripay) for Nigerian payment transactions and payouts. Use this skill whenever the user wants to initialize transactions, process payments, create virtual accounts, handle payouts, manage transfers, or work with Squad's payment infrastructure. Also trigger when the user mentions 'Squad', 'Habaripay', 'Squad payments', 'payment gateway Nigeria', 'virtual account creation', 'direct debit', or needs embedded payment solutions."
---

# Squad Payment Gateway Integration Skill

Squad (by Habaripay) is a comprehensive payment gateway providing production-ready APIs for payment transaction initialization, virtual account creation, fund transfers, and direct debit capabilities. It enables developers to embed payment processing and payout functionality directly into their applications with reliable, well-documented APIs designed for the Nigerian financial ecosystem.

## When to use this skill

You're building a payment system, marketplace, subscription platform, or business tool that needs to process payments and send payouts in Nigeria. Squad handles:

- **Payment Processing**: Accept payments via card, bank transfer, USSD, and more through a single API
- **Virtual Accounts**: Create dedicated GTBank virtual account numbers for customers to receive funds
- **Payouts & Transfers**: Send money from your Squad balance to any Nigerian bank account
- **Direct Debit**: Set up recurring debits from customer accounts with mandate-based transactions
- **Transaction Reconciliation**: Comprehensive transaction history and verification endpoints

Perfect for e-commerce platforms, marketplaces, subscription services, seller payouts, employee payments, and enterprise payment systems operating in Nigeria.

## Authentication

Squad uses **Bearer Token authentication**. All API requests require an authorization header with your API secret key.

### Header Format

```
Authorization: Bearer your_secret_key
```

**Key Management:**
- API keys are generated in your Squad dashboard at https://dashboard.squadco.com
- Sandbox keys start with `sandbox_sk_` prefix
- Production keys start with `pk_` prefix
- Store keys in environment variables (e.g., `SQUAD_API_KEY`, `SQUAD_SECRET_KEY`)
- **Never hardcode credentials** in your source code

### Base URLs

| Environment | Base URL |
|-------------|----------|
| **Sandbox** | `https://sandbox-api-d.squadco.com` |
| **Production** | `https://api-d.squadco.com` |

### Example Request

```bash
curl -X POST https://sandbox-api-d.squadco.com/transaction/initiate \
  -H "Authorization: Bearer sandbox_sk_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f" \
  -H "Content-Type: application/json" \
  -d '{"amount": 50000, "email": "customer@example.com"}'
```

## Core API Reference

### Initiate a Transaction

Initialize a payment transaction and receive a checkout link for customers to complete payment.

**Endpoint:** `POST /transaction/initiate`

**Request Body:**
```json
{
  "amount": 50000,
  "email": "customer@example.com",
  "currency": "NGN",
  "initiate_type": "inline",
  "transaction_ref": "TXN-2025-001",
  "customer_name": "Amina Okafor",
  "customer_phone": "+234901234567",
  "payment_channels": ["card", "bank", "ussd"],
  "meta_data": {
    "order_id": "ORD-12345",
    "product": "Digital Service",
    "customer_tier": "premium"
  },
  "pass_charge": false
}
```

**Field Definitions:**
- `amount` (required, integer): Transaction amount in **kobo** (₦500 = 50000 kobo). Kobo is the smallest unit of Nigerian Naira.
- `email` (required, string): Customer email address for payment receipt and communication
- `currency` (required, string): Presently only supports `"NGN"` (Nigerian Naira)
- `initiate_type` (required, string): Payment initiation method; currently only `"inline"` is supported
- `transaction_ref` (required, string): Unique identifier for this transaction. Must be unique across all your transactions. Used for idempotency—duplicate refs within 24 hours return cached result.
- `customer_name` (optional, string): Full name of the customer
- `customer_phone` (optional, string): Customer phone number in international format
- `payment_channels` (optional, array): Array of payment methods to offer. Options: `["card", "bank", "ussd", "transfer"]`. If omitted, all channels are available.
- `meta_data` (optional, object): Custom key-value pairs for additional transaction context. Returned in webhook and verification responses.
- `pass_charge` (optional, boolean): If `true`, customer bears the transaction charge; if `false` (default), you bear the charge

**Response:**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "transaction_ref": "TXN-2025-001",
    "checkout_url": "https://checkout.squad.ng/94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "amount": 50000,
    "currency": "NGN",
    "status": "pending",
    "created_at": "2025-02-24T10:30:00Z"
  }
}
```

**Response Field Details:**
- `transaction_id`: Squad's unique identifier for the transaction
- `checkout_url`: Share this URL with the customer. When visited, displays the Squad payment modal.
- `status`: Initial status is always `"pending"` until payment is completed
- `created_at`: ISO 8601 timestamp of transaction creation

**Important Notes:**
- Share the `checkout_url` with your customer via email, SMS, or by embedding in your application
- The checkout link is valid for 24 hours
- This endpoint confirms request acceptance, **not payment completion**. Always wait for webhook confirmation or poll the verification endpoint before fulfilling orders
- Amount must be in kobo (smallest unit). Omitting trailing zeros is a common error. ₦500.00 = 50000 kobo.

---

### Verify Transaction Status

Check the status and details of a transaction using its reference.

**Endpoint:** `GET /transaction/verify/{reference}`

**Path Parameters:**
- `reference` (required, string): The transaction reference provided during initiation (e.g., `TXN-2025-001`)

**Response (Successful Payment):**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "transaction_ref": "TXN-2025-001",
    "amount": 50000,
    "currency": "NGN",
    "transaction_status": "processed",
    "payment_method": "card",
    "payment_type": "card payment",
    "customer_email": "customer@example.com",
    "customer_name": "Amina Okafor",
    "paid_at": "2025-02-24T10:35:00Z",
    "merchant_amount": 48500,
    "charge_amount": 1500,
    "meta_data": {
      "order_id": "ORD-12345"
    }
  }
}
```

**Response (Pending Payment):**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "transaction_ref": "TXN-2025-001",
    "transaction_status": "pending",
    "amount": 50000,
    "currency": "NGN"
  }
}
```

**Transaction Status Values:**
- `pending`: Payment not yet completed; awaiting customer action
- `processed`: Payment successfully completed; funds received
- `failed`: Payment attempt failed; customer authorization declined or timeout occurred

**Key Fields:**
- `merchant_amount`: Amount you receive (after Squad's processing charge)
- `charge_amount`: Squad's transaction fee
- `payment_method`: Method used (card, bank_transfer, ussd, transfer)
- `paid_at`: Exact timestamp when payment cleared

**Important Notes:**
- Always verify transactions before fulfilling orders, especially for high-value transactions
- Poll this endpoint periodically if you don't receive webhooks (max every 30 seconds to avoid rate limits)
- `processed` status with `transaction_status === "processed"` indicates successful payment

---

### Create a Virtual Account

Generate a dedicated GTBank virtual account number for a customer to receive direct bank transfers. Funds deposited to this account appear in your Squad merchant balance.

**Endpoint:** `POST /virtual-account`

**Request Body (Individual Account):**
```json
{
  "customer_name": "Amina Okafor",
  "email": "amina@example.com",
  "phone": "+234901234567",
  "meta_data": {
    "customer_id": "CUST-001",
    "tier": "premium",
    "kyc_verified": true
  }
}
```

**Request Body (Business Account):**
```json
{
  "customer_name": "ABC Services Ltd",
  "email": "business@example.com",
  "phone": "+234901234567",
  "bvn": "12345678901",
  "business_type": "Limited Liability Company",
  "meta_data": {
    "business_id": "BIZ-001",
    "industry": "Technology"
  }
}
```

**Field Definitions:**
- `customer_name` (required, string): Name of individual or business
- `email` (required, string): Valid email address for account notifications
- `phone` (optional, string): Contact phone number
- `bvn` (optional, string): BVN (Bank Verification Number) for business accounts; required for KYC compliance
- `business_type` (optional, string): Type of business (Limited Liability Company, Sole Proprietorship, Partnership, etc.)
- `meta_data` (optional, object): Custom fields for your reference

**Response:**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "virtual_account_id": "VA_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "account_number": "1234567890",
    "bank_name": "Guaranty Trust Bank",
    "bank_code": "090288",
    "customer_name": "Amina Okafor",
    "customer_email": "amina@example.com",
    "account_status": "active",
    "created_at": "2025-02-24T10:45:00Z",
    "is_nuban": true
  }
}
```

**Response Fields:**
- `virtual_account_id`: Squad's unique identifier for this virtual account
- `account_number`: 10-digit NUBAN account number to share with customers for transfers
- `bank_code`: GTBank's CBN code (090288)
- `account_status`: `active` indicates the account is ready to receive funds
- `is_nuban`: `true` indicates this is a standard NUBAN (Nigerian Uniform Bank Account Number)

**Settlement Details:**
- All virtual accounts are GTBank accounts
- Funds transferred to the account number appear in your Squad merchant balance within minutes
- For instant settlement, both merchant and beneficiary accounts must be GTBank accounts
- Virtual accounts are permanent once created and cannot be deleted

**Important Notes:**
- Business accounts require BVN and KYC profiling approval before account creation (contact Squad support)
- Individual accounts can be created immediately without additional profiling
- Share the account number and bank code (`Guaranty Trust Bank - 090288`) with customers
- Virtual accounts are ideal for marketplace sellers, affiliate partners, and vendor payments
- Set up automatic sweeps in your dashboard to transfer virtual account balances to your main account

---

### Initiate a Payout (Transfer)

Send money from your Squad balance to any Nigerian bank account. Squad verifies account details before processing.

**Endpoint:** `POST /payout/transfer`

**Request Body:**
```json
{
  "amount": 100000,
  "currency": "NGN",
  "bank_code": "058",
  "account_number": "0123456789",
  "account_name": "Chinedu Adeyemi",
  "narration": "Payout for services rendered",
  "reference": "PAYOUT-2025-001",
  "remark": "Monthly commission payout"
}
```

**Field Definitions:**
- `amount` (required, integer): Payout amount in **kobo** (₦1,000 = 100000 kobo)
- `currency` (required, string): Must be `"NGN"` (Nigerian Naira)
- `bank_code` (required, string): 3-digit Nigerian bank code (e.g., `"058"` for GTBank, `"033"` for UBA, `"007"` for Access Bank)
- `account_number` (required, string): 10-digit NUBAN account number (must be validated using account lookup before payout)
- `account_name` (required, string): Beneficiary account name (as returned by account lookup)
- `narration` (optional, string): Description visible to the beneficiary in their bank statement
- `reference` (required, string): Your unique reference for this payout (for idempotency and tracking)
- `remark` (optional, string): Internal note for your records

**Nigerian Bank Codes (Common):**
| Bank Name | Code |
|-----------|------|
| Guaranty Trust Bank (GTBank) | 058 |
| United Bank for Africa (UBA) | 033 |
| Zenith Bank | 050 |
| First Bank | 011 |
| Access Bank | 044 |
| FCMB | 070 |
| Ecobank | 050 |
| Stanbic IBTC | 221 |

For a complete list, refer to the [CBN Bank Codes documentation](https://docs.prembly.com/docs/cbn-bank-codes).

**Response (Initiated):**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "reference": "PAYOUT-2025-001",
    "amount": 100000,
    "currency": "NGN",
    "bank_code": "058",
    "account_number": "0123456789",
    "account_name": "Chinedu Adeyemi",
    "status": "initiated",
    "initiated_at": "2025-02-24T11:00:00Z",
    "narration": "Payout for services rendered"
  }
}
```

**Payout Status Values:**
- `initiated`: Payout request accepted and queued for processing
- `processing`: Transfer is being processed by the banking system
- `success`: Funds successfully transferred to beneficiary account
- `failed`: Transfer failed (insufficient balance, invalid account, bank error, etc.)
- `reversed`: Transfer was reversed (unusual; contact Squad support if this occurs)

**Important Notes:**
- Always validate the account number using the account lookup endpoint before initiating payout
- Minimum payout amount is typically ₦100 (10000 kobo)
- Check your available balance before initiating payouts; insufficient balance causes failure
- Most payouts complete within 2-5 minutes; some may take up to 30 minutes during peak hours
- Use unique references to prevent duplicate payouts (same reference within 24 hours returns cached result)
- Payout limit varies by merchant tier; check your dashboard for daily/monthly limits
- Failed payouts are automatically retried; no action needed

---

### Get Payout Status

Retrieve the current status of a payout transfer using its reference.

**Endpoint:** `GET /payout/transfer/status/{reference}`

**Path Parameters:**
- `reference` (required, string): The payout reference provided during transfer initiation (e.g., `PAYOUT-2025-001`)

**Response (Successful):**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "reference": "PAYOUT-2025-001",
    "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "amount": 100000,
    "currency": "NGN",
    "status": "success",
    "bank_code": "058",
    "account_number": "0123456789",
    "account_name": "Chinedu Adeyemi",
    "recipient_bank": "Guaranty Trust Bank",
    "completed_at": "2025-02-24T11:02:30Z"
  }
}
```

**Response (Failed):**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "reference": "PAYOUT-2025-001",
    "status": "failed",
    "failure_reason": "Invalid account number",
    "failed_at": "2025-02-24T11:01:00Z"
  }
}
```

**Failure Reasons (Common):**
- `Invalid account number`: Account doesn't exist or is malformed
- `Insufficient balance`: Your Squad balance is insufficient
- `Account closed`: Beneficiary account is closed
- `Daily limit exceeded`: You've reached your daily payout limit
- `Bank error`: Temporary banking system issue (will retry automatically)

**Important Notes:**
- Poll this endpoint to monitor payout progress, or rely on webhook notifications
- Check every 30-60 seconds if awaiting completion (avoid rate limiting)
- A payout reference not found may indicate the payout hasn't been initiated yet

---

### Verify Bank Account

Lookup and validate a bank account before initiating a payout. This prevents sending money to incorrect accounts.

**Endpoint:** `POST /payout/account/lookup`

**Request Body:**
```json
{
  "bank_code": "058",
  "account_number": "0123456789"
}
```

**Field Definitions:**
- `bank_code` (required, string): 3-digit CBN bank code
- `account_number` (required, string): 10-digit NUBAN account number

**Response (Valid Account):**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "bank_code": "058",
    "account_number": "0123456789",
    "account_name": "Chinedu Adeyemi",
    "account_type": "Individual",
    "verified": true
  }
}
```

**Response (Invalid Account):**
```json
{
  "status": 400,
  "success": false,
  "message": "Account number not found"
}
```

**Account Type Values:**
- `Individual`: Personal bank account
- `Business`: Business/Corporate bank account
- `Government`: Government account
- `NGO`: Non-profit organization account

**Important Notes:**
- **Always verify accounts before payouts** to prevent sending money to wrong recipients
- Account verification is free and does not affect your quota
- Verification should complete within 2-3 seconds
- If verification fails, do not proceed with payout; re-check the bank code and account number
- Use the verified `account_name` in your payout request to ensure accuracy

---

### List Transactions

Retrieve your transaction history for reconciliation, reporting, and audit purposes.

**Endpoint:** `GET /transaction/list`

**Query Parameters:**
- `page` (optional, integer): Page number (starts at 1, defaults to 1)
- `per_page` (optional, integer): Results per page (max 100, defaults to 50)
- `from` (optional, string): Start date in YYYY-MM-DD format (inclusive)
- `to` (optional, string): End date in YYYY-MM-DD format (inclusive)
- `status` (optional, string): Filter by status (`pending`, `processed`, `failed`)
- `type` (optional, string): Filter by transaction type (`payment`, `payout`, `virtual_account`)

**Example Request:**
```
GET /transaction/list?page=1&per_page=25&from=2025-02-01&to=2025-02-28&status=processed
```

**Response:**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "page": 1,
    "per_page": 25,
    "total": 156,
    "total_pages": 7,
    "transactions": [
      {
        "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
        "transaction_ref": "TXN-2025-001",
        "type": "payment",
        "amount": 50000,
        "currency": "NGN",
        "status": "processed",
        "customer_email": "customer@example.com",
        "payment_method": "card",
        "created_at": "2025-02-24T10:30:00Z",
        "completed_at": "2025-02-24T10:35:00Z"
      },
      {
        "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
        "transaction_ref": "PAYOUT-2025-001",
        "type": "payout",
        "amount": 100000,
        "currency": "NGN",
        "status": "success",
        "created_at": "2025-02-24T11:00:00Z",
        "completed_at": "2025-02-24T11:02:30Z"
      }
    ]
  }
}
```

**Response Fields:**
- `page`: Current page number
- `per_page`: Results returned per page
- `total`: Total transaction count matching filters
- `total_pages`: Number of pages available
- `transactions`: Array of transaction objects

**Important Notes:**
- Results are sorted by creation date in descending order (newest first)
- Maximum date range is typically 90 days; request longer ranges in multiple queries
- Use pagination for large result sets to avoid timeout
- This endpoint is useful for daily reconciliation and accounting purposes

---

### Create a Direct Debit Mandate

Set up recurring debits from a customer's bank account with their authorization. Mandates enable subscription billing and automated payments.

**Endpoint:** `POST /transaction/mandate/create`

**Request Body:**
```json
{
  "customer_email": "customer@example.com",
  "customer_name": "Amina Okafor",
  "bank_code": "058",
  "account_number": "0123456789",
  "account_name": "Amina Okafor",
  "amount": 50000,
  "start_date": "2025-03-01",
  "mandate_ref": "MANDATE-2025-001",
  "meta_data": {
    "subscription_id": "SUB-12345",
    "plan": "premium_monthly"
  }
}
```

**Field Definitions:**
- `customer_email` (required, string): Customer's email for mandate notifications
- `customer_name` (required, string): Full name of customer
- `bank_code` (required, string): 3-digit CBN bank code of customer's account
- `account_number` (required, string): 10-digit NUBAN account number
- `account_name` (required, string): Account name (must match bank records)
- `amount` (required, integer): Amount per debit in kobo
- `start_date` (optional, string): Mandate activation date (YYYY-MM-DD format)
- `mandate_ref` (required, string): Your unique mandate reference

**Response:**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "mandate_id": "MAND_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "mandate_ref": "MANDATE-2025-001",
    "customer_email": "customer@example.com",
    "bank_code": "058",
    "account_number": "0123456789",
    "amount": 50000,
    "status": "pending_activation",
    "created_at": "2025-02-24T12:00:00Z"
  }
}
```

**Mandate Status Values:**
- `pending_activation`: Awaiting customer activation (via email/SMS link)
- `active`: Mandate is active and ready for debits
- `suspended`: Temporarily paused
- `revoked`: Cancelled by customer or merchant

**Important Notes:**
- Customers must activate the mandate via an authorization link sent to their email
- Activation typically takes 24-48 hours
- Only active mandates can be used for debiting
- Customers can revoke mandates at any time
- Use mandates for subscription renewals and automated billing

---

### Debit with Mandate

Debit a customer's account using an active mandate.

**Endpoint:** `POST /transaction/mandate/debit`

**Request Body:**
```json
{
  "mandate_ref": "MANDATE-2025-001",
  "amount": 50000,
  "transaction_ref": "DEBIT-2025-001",
  "narration": "Monthly subscription payment"
}
```

**Field Definitions:**
- `mandate_ref` (required, string): Reference of the activated mandate
- `amount` (required, integer): Amount to debit in kobo (must not exceed mandate limit)
- `transaction_ref` (required, string): Unique reference for this debit transaction
- `narration` (optional, string): Description for the customer's statement

**Response:**
```json
{
  "status": 200,
  "success": true,
  "data": {
    "debit_id": "DEBIT_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "transaction_ref": "DEBIT-2025-001",
    "mandate_ref": "MANDATE-2025-001",
    "amount": 50000,
    "status": "initiated",
    "initiated_at": "2025-02-24T12:15:00Z"
  }
}
```

**Important Notes:**
- The mandate must be in `active` status for debits to succeed
- Amount debited cannot exceed the mandate authorization limit
- Debit confirmations are sent via webhook
- Poll the transaction verification endpoint to check debit status

---

## Webhooks

Squad sends real-time webhooks for transaction and payout events. This enables you to react immediately to payment status changes without polling.

### Webhook Configuration

1. Log in to your Squad dashboard at https://dashboard.squadco.com
2. Navigate to **Settings > API & Webhooks**
3. Enter your webhook endpoint URL (e.g., `https://yourdomain.com/webhooks/squad`)
4. Your endpoint must:
   - Accept POST requests
   - Respond with HTTP 200 within 30 seconds
   - Verify webhook signatures for security
   - Handle duplicate events (use transaction reference for deduplication)

### Webhook Events

Squad sends the following webhook events:

| Event | Trigger | Payload Contains |
|-------|---------|------------------|
| `charge_successful` | Payment completed successfully | Transaction ID, reference, amount, payment method |
| `charge_failed` | Payment attempt failed | Reason for failure, transaction reference |
| `payout_successful` | Payout transfer completed | Payout reference, amount, beneficiary details |
| `payout_failed` | Payout transfer failed | Failure reason, payout reference |
| `virtual_account.funded` | Money deposited to virtual account | Account ID, amount, sender, deposit reference |
| `mandate.created` | New mandate created | Mandate ID, customer email, status |
| `mandate.activated` | Mandate activated by customer | Mandate ID, activation timestamp |
| `mandate.revoked` | Mandate cancelled | Mandate ID, reason |

### Webhook Payload Format

**Example: `charge_successful` Event**
```json
{
  "event": "charge_successful",
  "data": {
    "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "transaction_ref": "TXN-2025-001",
    "amount": 50000,
    "currency": "NGN",
    "status": "processed",
    "customer_email": "customer@example.com",
    "customer_name": "Amina Okafor",
    "payment_method": "card",
    "paid_at": "2025-02-24T10:35:00Z",
    "meta_data": {
      "order_id": "ORD-12345"
    }
  },
  "timestamp": "2025-02-24T10:35:05Z"
}
```

**Example: `payout_successful` Event**
```json
{
  "event": "payout_successful",
  "data": {
    "reference": "PAYOUT-2025-001",
    "transaction_id": "TXN_94f2b798466408ef4d19e848ee1a4d1a3e93f104046f",
    "amount": 100000,
    "currency": "NGN",
    "bank_code": "058",
    "account_number": "0123456789",
    "account_name": "Chinedu Adeyemi",
    "status": "success",
    "completed_at": "2025-02-24T11:02:30Z"
  },
  "timestamp": "2025-02-24T11:02:35Z"
}
```

### Signature Verification

Squad signs all webhooks using HMAC-SHA512. Verify the signature in the `x-squad-signature` header to ensure authenticity.

**Node.js/Express Example:**
```javascript
const crypto = require('crypto');

app.post('/webhooks/squad', (req, res) => {
  const signature = req.headers['x-squad-signature'];
  const secret = process.env.SQUAD_WEBHOOK_SECRET;

  // Create HMAC-SHA512 hash of the request body
  const hash = crypto
    .createHmac('sha512', secret)
    .update(JSON.stringify(req.body), 'utf8')
    .digest('hex');

  // Compare signatures
  if (hash !== signature) {
    console.error('Invalid webhook signature');
    return res.status(403).json({ error: 'Unauthorized' });
  }

  // Signature is valid; process the webhook
  const event = req.body.event;
  const data = req.body.data;

  if (event === 'charge_successful') {
    // Handle successful payment
    console.log(`Payment confirmed: ${data.transaction_ref}`);
    // Update database, fulfill order, etc.
  } else if (event === 'payout_successful') {
    // Handle successful payout
    console.log(`Payout completed: ${data.reference}`);
  }

  res.json({ status: 'received' });
});
```

**Python/Flask Example:**
```python
from flask import request
import hmac
import hashlib
import json

@app.route('/webhooks/squad', methods=['POST'])
def squad_webhook():
    signature = request.headers.get('x-squad-signature')
    secret = os.environ.get('SQUAD_WEBHOOK_SECRET')

    # Create HMAC-SHA512 hash
    computed_hash = hmac.new(
        secret.encode(),
        request.data,
        hashlib.sha512
    ).hexdigest()

    # Verify signature
    if not hmac.compare_digest(computed_hash, signature):
        return {'error': 'Unauthorized'}, 403

    # Process webhook
    payload = request.json
    event = payload.get('event')
    data = payload.get('data')

    if event == 'charge_successful':
        handle_payment_success(data)
    elif event == 'payout_successful':
        handle_payout_success(data)

    return {'status': 'received'}, 200
```

### Webhook Best Practices

1. **Verify signatures** before processing any webhook
2. **Use transaction references for deduplication** to handle duplicate webhook deliveries
3. **Respond quickly** with HTTP 200 and process asynchronously
4. **Log all webhooks** for debugging and audit trails
5. **Implement exponential backoff** if your endpoint is temporarily unavailable (Squad will retry)
6. **Handle idempotency** — same event delivered multiple times should have the same result
7. **Never trust webhook timing** — always verify via the API before taking critical actions

---

## Common Integration Patterns

### Pattern 1: E-commerce Checkout

Accept payments from online store customers.

```
1. Customer adds items to cart and clicks checkout
2. POST /transaction/initiate with order details
3. Receive checkout_url; redirect customer or send via email
4. Customer completes payment on Squad checkout page
5. Receive webhook: charge_successful
6. Verify transaction via GET /transaction/verify/{ref}
7. Confirm status is "processed"
8. Fulfill order and send confirmation email
```

**Code Example (Node.js):**
```javascript
const axios = require('axios');

async function initiatePayment(order) {
  const response = await axios.post(
    'https://api-d.squadco.com/transaction/initiate',
    {
      amount: order.total_amount_kobo,
      email: order.customer_email,
      currency: 'NGN',
      initiate_type: 'inline',
      transaction_ref: `ORD-${order.id}`,
      customer_name: order.customer_name,
      meta_data: {
        order_id: order.id,
        items: order.items.length,
        shipping_address: order.address
      },
      pass_charge: false
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.SQUAD_API_KEY}`
      }
    }
  );

  return response.data.data.checkout_url;
}

// Webhook handler
app.post('/webhooks/squad', async (req, res) => {
  const { event, data } = req.body;

  if (event === 'charge_successful') {
    const orderId = data.meta_data.order_id;

    // Update order status
    await db.order.update(
      { id: orderId },
      { status: 'paid', paid_at: new Date() }
    );

    // Trigger fulfillment
    await fulfillOrder(orderId);

    // Send confirmation
    await sendConfirmationEmail(data.customer_email, orderId);
  }

  res.json({ status: 'received' });
});
```

---

### Pattern 2: Marketplace Seller Payouts

Process payments to multiple sellers based on their sales.

```
1. Multiple buyers make purchases (each initiates payment)
2. Receive charge_successful webhooks for each payment
3. After order fulfillment (e.g., 7 days):
   a. GET /payout/verify to validate seller's bank account
   b. Calculate seller's commission (total - platform fee)
   c. POST /payout/transfer to send commission
   d. Receive payout_successful webhook
4. Log payout in seller dashboard
5. Send payout confirmation to seller
```

**Code Example (Python):**
```python
async def process_seller_payout(seller_id, orders):
    seller = await db.seller.get(seller_id)

    # Calculate total sales
    total_sales = sum(o.amount for o in orders)

    # Platform takes 10% commission
    commission = int(total_sales * 0.10)
    payout_amount = total_sales - commission

    # Verify seller's bank account
    verify_response = requests.get(
        f'{SQUAD_BASE_URL}/payout/account/lookup',
        params={
            'bank_code': seller.bank_code,
            'account_number': seller.account_number
        },
        headers={'Authorization': f'Bearer {SQUAD_API_KEY}'}
    )

    if not verify_response.json()['success']:
        raise Exception('Account verification failed')

    # Initiate payout
    payout_response = requests.post(
        f'{SQUAD_BASE_URL}/payout/transfer',
        json={
            'amount': payout_amount,
            'currency': 'NGN',
            'bank_code': seller.bank_code,
            'account_number': seller.account_number,
            'account_name': verify_response.json()['data']['account_name'],
            'reference': f'PAYOUT-{seller_id}-{int(time.time())}',
            'narration': f'Sales commission - {len(orders)} orders'
        },
        headers={'Authorization': f'Bearer {SQUAD_API_KEY}'}
    )

    payout = payout_response.json()['data']

    # Store payout record
    await db.payout.create({
        seller_id: seller_id,
        amount: payout_amount,
        reference: payout['reference'],
        status: 'initiated'
    })

    return payout
```

---

### Pattern 3: Dedicated Vendor Virtual Accounts

Create unique virtual accounts for vendors to receive payments from customers.

```
1. Vendor signs up on your platform
2. POST /virtual-account to create dedicated GTBank account
3. Share account number with vendor
4. Customers transfer money directly to vendor's account
5. Funds appear in your merchant balance immediately
6. GET /transaction/list to track virtual account deposits
7. Optional: Auto-sweep to vendor's own bank account
```

**Code Example:**
```javascript
async function createVendorAccount(vendor) {
  // Create virtual account
  const response = await axios.post(
    'https://api-d.squadco.com/virtual-account',
    {
      customer_name: vendor.business_name,
      email: vendor.email,
      phone: vendor.phone,
      meta_data: {
        vendor_id: vendor.id,
        onboarding_date: new Date().toISOString()
      }
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.SQUAD_API_KEY}`
      }
    }
  );

  const account = response.data.data;

  // Store account details
  await db.vendor.update(
    { id: vendor.id },
    {
      virtual_account_id: account.virtual_account_id,
      virtual_account_number: account.account_number,
      virtual_account_bank: 'Guaranty Trust Bank',
      virtual_account_code: account.bank_code,
      account_status: 'active'
    }
  );

  // Send account details to vendor
  await sendEmailWithAccountDetails(vendor.email, account);

  return account;
}

// Monitor deposits to virtual accounts
async function syncVirtualAccountTransactions() {
  const vendors = await db.vendor.findAll({
    where: { virtual_account_id: { $ne: null } }
  });

  for (const vendor of vendors) {
    const txnResponse = await axios.get(
      'https://api-d.squadco.com/virtual-account/customer/transactions/' +
      vendor.virtual_account_id,
      {
        headers: {
          Authorization: `Bearer ${process.env.SQUAD_API_KEY}`
        }
      }
    );

    const transactions = txnResponse.data.data;

    // Process each transaction
    for (const txn of transactions) {
      if (!await db.virtualAccountDeposit.findOne({
        where: { transaction_id: txn.transaction_id }
      })) {
        // New deposit
        await db.virtualAccountDeposit.create({
          vendor_id: vendor.id,
          transaction_id: txn.transaction_id,
          amount: txn.amount,
          deposited_at: txn.created_at
        });

        // Credit vendor's balance
        await creditVendorBalance(vendor.id, txn.amount);
      }
    }
  }
}
```

---

### Pattern 4: Subscription Billing with Mandates

Set up recurring subscription payments using direct debits.

```
1. Customer subscribes to plan (e.g., monthly software service)
2. GET /transaction/mandate/banklists to show supported banks
3. Customer provides bank account details
4. POST /transaction/mandate/create to create mandate
5. Customer receives activation email from Squad
6. Customer clicks activation link and authorizes mandate
7. Mandate status changes to "active"
8. Monthly: POST /transaction/mandate/debit to charge subscription
9. Receive webhook confirmation of debit
10. Update subscription status and renew service
```

**Code Example:**
```javascript
// Step 1: Create mandate
async function createSubscriptionMandate(customer, plan) {
  const mandateResponse = await axios.post(
    'https://api-d.squadco.com/transaction/mandate/create',
    {
      customer_email: customer.email,
      customer_name: customer.name,
      bank_code: customer.bank_code,
      account_number: customer.account_number,
      account_name: customer.account_name,
      amount: plan.monthly_amount_kobo,
      mandate_ref: `MANDATE-${customer.id}-${plan.id}`,
      start_date: new Date().toISOString().split('T')[0],
      meta_data: {
        customer_id: customer.id,
        plan_id: plan.id,
        plan_name: plan.name
      }
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.SQUAD_API_KEY}`
      }
    }
  );

  const mandate = mandateResponse.data.data;

  // Store mandate
  await db.subscription.create({
    customer_id: customer.id,
    plan_id: plan.id,
    mandate_id: mandate.mandate_id,
    mandate_ref: mandate.mandate_ref,
    status: 'pending_activation',
    created_at: new Date()
  });

  return mandate;
}

// Step 2: Process monthly renewals
async function processMonthlyRenewals() {
  const activeSubscriptions = await db.subscription.findAll({
    where: { status: 'active', last_billed_at: { $lt: Date.now() - 30*24*60*60*1000 } }
  });

  for (const sub of activeSubscriptions) {
    const plan = await db.plan.get(sub.plan_id);

    try {
      const debitResponse = await axios.post(
        'https://api-d.squadco.com/transaction/mandate/debit',
        {
          mandate_ref: sub.mandate_ref,
          amount: plan.monthly_amount_kobo,
          transaction_ref: `DEBIT-${sub.id}-${Date.now()}`,
          narration: `${plan.name} subscription renewal`
        },
        {
          headers: {
            Authorization: `Bearer ${process.env.SQUAD_API_KEY}`
          }
        }
      );

      // Update subscription
      await db.subscription.update(
        { id: sub.id },
        { last_billed_at: new Date() }
      );

      console.log(`Subscription renewal initiated: ${sub.id}`);
    } catch (error) {
      console.error(`Subscription renewal failed: ${sub.id}`, error.message);
      await notifyCustomerOfFailedRenewal(sub.customer_id);
    }
  }
}

// Webhook handler for mandate debits
app.post('/webhooks/squad', async (req, res) => {
  const { event, data } = req.body;

  if (event === 'charge_successful' && data.meta_data?.type === 'mandate_debit') {
    const subscription = await db.subscription.findOne({
      where: { mandate_ref: data.meta_data.mandate_ref }
    });

    if (subscription) {
      // Update subscription expiry
      await db.subscription.update(
        { id: subscription.id },
        { expires_at: new Date(Date.now() + 30*24*60*60*1000) }
      );

      await sendRenewalConfirmation(subscription.customer_id);
    }
  }

  res.json({ status: 'received' });
});
```

---

## Error Handling

Squad returns structured error responses. Handle errors gracefully and implement appropriate retry logic.

### Error Response Format

**HTTP 4xx Error (Client Error):**
```json
{
  "status": 400,
  "success": false,
  "message": "Invalid bank code provided"
}
```

**HTTP 5xx Error (Server Error):**
```json
{
  "status": 500,
  "success": false,
  "message": "Internal server error. Please try again later."
}
```

### HTTP Status Codes

| Code | Error Type | Meaning | Action |
|------|-----------|---------|--------|
| **200** | Success | Request successful | Process normally |
| **400** | Bad Request | Invalid parameters, missing fields | Fix request and retry immediately |
| **401** | Unauthorized | Invalid or missing API key | Check Bearer token in headers |
| **404** | Not Found | Resource doesn't exist | Verify transaction/account reference |
| **429** | Rate Limited | Exceeded API rate limit | Implement exponential backoff |
| **500** | Server Error | Squad service issue | Retry with exponential backoff |
| **502** | Bad Gateway | Temporary connectivity issue | Retry with exponential backoff |
| **503** | Service Unavailable | Squad maintenance or overload | Retry with exponential backoff |

### Common Error Messages

**Payment Initiation:**
- `"amount is required"` — Missing amount field
- `"amount must be greater than 0"` — Amount is zero or negative
- `"Invalid email format"` — Customer email is invalid
- `"Duplicate transaction reference"` — Reference already processed in last 24 hours

**Payout:**
- `"Insufficient balance"` — Your Squad merchant balance is too low
- `"Invalid bank code"` — Bank code doesn't exist or is malformed
- `"Invalid account number"` — Account number format is wrong (must be 10 digits)
- `"Account not found"` — Account doesn't exist at specified bank
- `"Daily limit exceeded"` — You've reached your daily payout limit

**Virtual Account:**
- `"KYC not approved"` — Your business KYC is not verified for account creation
- `"Account creation limit reached"` — You've created the maximum allowed accounts

**Mandate/Direct Debit:**
- `"Mandate not found"` — Mandate reference doesn't exist
- `"Mandate not active"` — Mandate hasn't been activated by customer yet
- `"Amount exceeds mandate limit"` — Debit amount exceeds authorized limit

### Retry Strategy

Implement exponential backoff for retryable errors (429, 5xx):

```javascript
async function apiRequestWithRetry(fn, maxRetries = 3) {
  let lastError;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      // Don't retry 4xx errors except 429
      if (error.response?.status >= 400 && error.response?.status < 500 && error.response?.status !== 429) {
        throw error;
      }

      // Calculate backoff: 1s, 2s, 4s
      const delayMs = Math.pow(2, attempt) * 1000;
      console.log(`Attempt ${attempt + 1} failed. Retrying in ${delayMs}ms...`);

      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
  }

  throw lastError;
}

// Usage
const result = await apiRequestWithRetry(async () => {
  return axios.post(`${SQUAD_BASE_URL}/payout/transfer`, payoutData, {
    headers: { Authorization: `Bearer ${SQUAD_API_KEY}` }
  });
});
```

---

## Important Notes and Gotchas

### Amount Units (Critical)

**All amounts are in kobo, not Naira.**

- 1 NGN = 100 kobo
- ₦500 = 50,000 kobo
- ₦1 = 100 kobo
- ₦0.01 = 1 kobo

This is the most common integration error. Always multiply Nigerian Naira by 100 before sending to Squad APIs.

### Unique References

Use globally unique transaction references to ensure idempotency:

- Duplicate references within 24 hours return the cached result
- Use UUID, timestamp-based, or incrementing references: `TXN-${Date.now()}-${uuid()}`
- References should be immutable once used
- Prevents accidental duplicate charges or payouts

### Bank Codes

Always use valid 3-digit CBN bank codes:

- **058** - Guaranty Trust Bank (GTBank)
- **033** - United Bank for Africa (UBA)
- **050** - Zenith Bank
- **011** - First Bank
- **044** - Access Bank
- **070** - FCMB
- **051** - Ecobank
- **221** - Stanbic IBTC Bank

Invalid codes cause payout failures. Use the complete bank code list from [CBN documentation](https://docs.prembly.com/docs/cbn-bank-codes).

### Account Validation

**Always validate recipient accounts before payouts:**

```javascript
// Never skip this step
const verifyResponse = await axios.get(
  'https://api-d.squadco.com/payout/account/lookup?bank_code=058&account_number=0123456789',
  { headers: { Authorization: `Bearer ${SQUAD_API_KEY}` } }
);

if (!verifyResponse.data.success) {
  throw new Error('Account validation failed');
}

// Use the verified account name in your payout
const accountName = verifyResponse.data.data.account_name;
```

### Virtual Accounts are Permanent

- Once created, virtual accounts cannot be deleted
- They remain active indefinitely
- Funds deposited are instantly transferred to your merchant balance (no sweep required)
- Each virtual account is linked to a single customer

### Webhook Signature Verification

Always verify webhook signatures using HMAC-SHA512:

- Never process unverified webhooks
- Use the `x-squad-signature` header
- Signature is computed over the entire request body as JSON string
- Mismatched signatures indicate tampering or test requests

### Payment Confirmation

The API response to `POST /transaction/initiate` indicates **request acceptance, not payment completion**:

- Always wait for webhook or poll verification endpoint before fulfilling orders
- Share `checkout_url` with customer
- Monitor for `charge_successful` webhook
- Verify transaction status is `"processed"` before shipping/delivering

### Sandbox vs Production

- **Sandbox URL**: `https://sandbox-api-d.squadco.com` — Use for testing
- **Production URL**: `https://api-d.squadco.com` — Use for real transactions
- Use corresponding API keys (sandbox keys start with `sandbox_sk_`)
- Never send production requests to sandbox environment

### Rate Limits

Squad enforces API rate limits to protect infrastructure:

- Typical limit: ~100-1000 requests per minute (varies by plan)
- Rate limit exceeded returns HTTP 429
- Response includes `Retry-After` header indicating wait time
- Implement exponential backoff for 429 responses

### Virtual Account Settlement

For instant settlement when using virtual accounts:

- Both merchant and beneficiary accounts must be GTBank accounts
- Funds appear in your balance within minutes
- Set up automatic sweeps to transfer virtual account balances to your main account
- Contact Squad support to configure sweep settings

### Payout Limits

Check your dashboard for:

- Daily payout limit (varies by merchant tier)
- Monthly payout limit
- Per-transaction maximum
- Minimum payout amount (typically ₦100 = 10,000 kobo)

Large payouts may be queued or require manual approval.

### Transaction History Retention

- Transaction history is retained for 12 months
- Use date range filters to query older transactions
- For long-term reporting, export and archive transaction lists regularly

---

## Useful Links

| Resource | URL |
|----------|-----|
| Official Documentation (Current) | https://docs.squadco.com |
| Deprecated GitBook Docs | https://squadinc.gitbook.io/squad-api-documentation |
| Dashboard | https://dashboard.squadco.com |
| Sandbox Playground | https://sandbox-api-d.squadco.com |
| Nigeria Bank Codes (CBN) | https://docs.prembly.com/docs/cbn-bank-codes |
| Squad Developers | https://squadco.com/developers/ |
| Support & Help | https://support.squadco.com |
| Status Page | https://status.squadco.com |

---

## Summary

Squad provides a comprehensive, production-ready payment gateway for Nigerian businesses:

- **Payments**: Accept payments via multiple channels (card, bank, USSD)
- **Virtual Accounts**: Create dedicated customer accounts for receiving transfers
- **Payouts**: Send money to any Nigerian bank account with account validation
- **Direct Debit**: Set up recurring charges with customer authorization
- **Real-time Webhooks**: Instant notifications of payment status changes
- **Reconciliation**: Complete transaction history with filtering and search

With proper error handling, webhook verification, and idempotency practices, Squad enables reliable, scalable payment processing for Nigerian fintech and e-commerce applications.
