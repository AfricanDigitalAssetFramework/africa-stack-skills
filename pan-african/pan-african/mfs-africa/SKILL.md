---
name: onafriq-mfs-africa
description: "Integrate with the Onafriq (formerly MFS Africa) API to connect with 400M+ mobile wallets and 200M+ bank accounts across Africa. Use this skill whenever the user wants to send money to mobile money wallets, process collections, execute batch disbursements, or manage payouts across Africa. Trigger when the user mentions 'Onafriq', 'MFS Africa', 'mobile money hub', 'mobile wallet transfers', 'African payout', 'cross-border payments', or needs to reach customers on mobile money platforms across 40+ African countries."
---

# Onafriq (MFS Africa) Integration Skill

Onafriq, formerly known as MFS Africa, is Africa's largest digital payments hub connecting over 400 million mobile wallets and 200 million bank accounts across more than 40 African countries. It provides a unified API for money transfers, collections, and disbursements across diverse mobile money providers, banks, and cash agents on the continent.

## When to use this skill

Build fintech applications, salary disbursement systems, utility payment platforms, or any service that needs to reach customers via mobile money across Africa. Onafriq abstracts the complexity of multiple mobile money providers, banks, and regional payment schemes into a single integration. Perfect for:

- Salary and payroll disbursements to employee mobile wallets
- Marketplace and vendor payment systems
- Loan disbursement and repayment platforms
- Remittance and money transfer services
- Bill payment and utility collections
- Refunds and refund processing
- Withdrawal services in fintech applications

## Authentication

All Onafriq API requests require Token-based authentication using an API key:

```
Authorization: Token YOUR_API_KEY
```

Generate your API key in the Onafriq Dashboard under API Settings. Store the key securely in environment variables (`ONAFRIQ_API_KEY`) and never hardcode credentials.

**Base URLs:**
- Domestic transactions: `https://api.mfsafrica.com/api`
- Cross-border transactions: `https://mfsafrica.beyonicpartners.com/api`

Contact your account manager to confirm which base URL applies to your account. Both URLs support versioning via path (`/v1`, `/v2`) or headers.

## Core API Reference

### Payouts API: Create an Async Payment

Create a payment to a mobile money wallet, bank account, or cash agent.

```
POST /api/v1/create_payment
```

**Request Body:**
```json
{
  "account_reference": "REF_USER_123456",
  "transaction_reference": "TRX_2025_001",
  "customer": {
    "email": "user@example.com",
    "msisdn": "+254701234567"
  },
  "details": {
    "msisdn": "+254701234567",
    "country_code": "KE",
    "amount": 1000,
    "currency": "KES"
  },
  "service_type": "mobile_money_payout",
  "reason": "Vendor payment",
  "metadata": {
    "vendor_id": "VENDOR_123",
    "transaction_type": "payout"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payment_id": "MFS_PAY_xxxxx",
    "account_reference": "REF_USER_123456",
    "transaction_reference": "TRX_2025_001",
    "status": "pending",
    "destination_country": "KE",
    "destination_msisdn": "+254701234567",
    "amount": 1000,
    "currency": "KES",
    "service_type": "mobile_money_payout",
    "created_at": "2025-02-24T10:30:00Z"
  }
}
```

Results come asynchronously via webhook. Monitor `payment.status.changed` events for completion.

### Payouts API: Get Payment Status

Check the status of a previously initiated payment.

```
GET /api/v1/payments/{payment_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payment_id": "MFS_PAY_xxxxx",
    "account_reference": "REF_USER_123456",
    "transaction_reference": "TRX_2025_001",
    "status": "completed",
    "destination_country": "KE",
    "destination_msisdn": "+254701234567",
    "amount": 1000,
    "currency": "KES",
    "provider": "mpesa",
    "provider_reference": "MP_REF_12345",
    "created_at": "2025-02-24T10:30:00Z",
    "completed_at": "2025-02-24T10:30:45Z",
    "fees": 25,
    "failure_reason": null
  }
}
```

Status values: `pending`, `completed`, `failed`, `cancelled`.

### Collections API: Create Collection Request

Set up a payment collection request for customers to pay via mobile money.

```
POST /api/v1/collection_requests
```

**Request Body:**
```json
{
  "account_reference": "ACC_REF_001",
  "customer": {
    "email": "customer@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "msisdn": "+254701234567"
  },
  "collection_details": {
    "msisdn": "+254701234567",
    "country_code": "KE",
    "amount": 5000,
    "currency": "KES",
    "description": "Service subscription payment",
    "requested_by": "APP_VENDOR"
  },
  "metadata": {
    "invoice_id": "INV_2025_001",
    "subscription_plan": "premium"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "collection_request_id": "MFS_COL_xxxxx",
    "account_reference": "ACC_REF_001",
    "customer_msisdn": "+254701234567",
    "amount": 5000,
    "currency": "KES",
    "status": "pending",
    "created_at": "2025-02-24T10:30:00Z",
    "expires_at": "2025-02-26T10:30:00Z",
    "webhook_secret": "webhook_secret_xxxxx"
  }
}
```

Customer receives an in-app or USSD notification to complete payment. Status updates arrive via webhook (`collectionrequest.status.changed`).

### Collections API: Get Collection Status

```
GET /api/v1/collection_requests/{collection_request_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "collection_request_id": "MFS_COL_xxxxx",
    "account_reference": "ACC_REF_001",
    "customer_msisdn": "+254701234567",
    "amount": 5000,
    "currency": "KES",
    "status": "received",
    "created_at": "2025-02-24T10:30:00Z",
    "received_at": "2025-02-24T10:35:00Z",
    "received_amount": 5000,
    "provider": "mpesa"
  }
}
```

### Collections API: Received Collections

Get a list of collections received against a collection request.

```
GET /api/v1/collection_requests/{collection_request_id}/collections
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "collection_id": "MFS_COLL_xxxxx",
      "amount": 5000,
      "currency": "KES",
      "provider": "mpesa",
      "provider_reference": "MP_REF_54321",
      "received_at": "2025-02-24T10:35:00Z"
    }
  ]
}
```

### Transactions API: List Payments

```
GET /api/v1/payments?limit=50&offset=0&status=completed&created_after=2025-02-01&created_before=2025-02-28&country_code=KE
```

**Query Parameters:**
- `limit`: Max results (default 50, max 100)
- `offset`: Pagination offset
- `status`: Filter by status (pending/completed/failed)
- `created_after`: ISO 8601 start date
- `created_before`: ISO 8601 end date
- `country_code`: Filter by destination country
- `transaction_reference`: Filter by reference

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "payment_id": "MFS_PAY_xxxxx",
      "transaction_reference": "TRX_2025_001",
      "amount": 1000,
      "currency": "KES",
      "status": "completed",
      "created_at": "2025-02-24T10:30:00Z"
    }
  ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 150
  }
}
```

## Webhooks

Onafriq sends webhooks for payment and collection events. Configure your webhook endpoint in the Dashboard under API Settings.

### Webhook Signature Verification

Verify the HMAC signature to ensure webhooks are authentic:

```javascript
const crypto = require('crypto');
const signature = req.headers['x-onafriq-signature'];
const body = req.rawBody; // Use raw request body string, not parsed JSON
const expectedSignature = crypto
  .createHmac('sha256', YOUR_WEBHOOK_SECRET)
  .update(body)
  .digest('hex');

if (signature === expectedSignature) {
  // Valid webhook — process the event
} else {
  // Invalid signature — reject
  return res.status(401).send('Unauthorized');
}
```

### Webhook Events

**Payment Events:**
- `payment.status.changed` - Payment status changed (pending → completed/failed)
- `payment.completed` - Payment successfully delivered
- `payment.failed` - Payment failed

**Collection Events:**
- `collection.received` - Collection received from customer
- `collectionrequest.status.changed` - Collection request status changed (pending → received/expired/cancelled)
- `collectionrequest.expired` - Collection request expired without payment

**Example Payment Webhook Payload:**
```json
{
  "event": "payment.status.changed",
  "timestamp": "2025-02-24T10:30:45Z",
  "data": {
    "payment_id": "MFS_PAY_xxxxx",
    "transaction_reference": "TRX_2025_001",
    "account_reference": "REF_USER_123456",
    "status": "completed",
    "amount": 1000,
    "currency": "KES",
    "destination_msisdn": "+254701234567",
    "destination_country": "KE",
    "provider": "mpesa",
    "provider_reference": "MP_REF_12345",
    "completed_at": "2025-02-24T10:30:45Z",
    "fees": 25
  }
}
```

**Example Collection Webhook Payload:**
```json
{
  "event": "collection.received",
  "timestamp": "2025-02-24T10:35:00Z",
  "data": {
    "collection_id": "MFS_COLL_xxxxx",
    "collection_request_id": "MFS_COL_xxxxx",
    "account_reference": "ACC_REF_001",
    "customer_msisdn": "+254701234567",
    "amount": 5000,
    "currency": "KES",
    "provider": "mpesa",
    "provider_reference": "MP_REF_54321",
    "received_at": "2025-02-24T10:35:00Z"
  }
}
```

### Webhook Best Practices

- Respond with 200 OK immediately after receiving the webhook
- Process the event asynchronously (queue/job system)
- Implement exponential backoff for retries
- Store webhook logs for debugging
- Verify signature on every webhook

## Common Integration Patterns

### Simple Payout Flow

1. Collect recipient phone number and amount from your application
2. Validate phone format includes country code (e.g., +254701234567)
3. `POST /api/v1/create_payment` with recipient details
4. Store the returned `payment_id` in your database
5. Listen for `payment.status.changed` webhook
6. Update your database with final status (completed/failed)
7. Notify user or trigger fulfillment

### Salary Disbursement (Batch Payroll)

1. Prepare employee list with phone numbers, amounts, and employee IDs
2. Create a collection or batch identifier (e.g., "PAYROLL_FEB_2025")
3. For each employee: `POST /api/v1/create_payment` with salary details
4. Store all `payment_id` values and associate with batch
5. Listen for `payment.status.changed` webhooks
6. Track completion status per employee
7. Generate payroll report showing success/failure counts and fees

### Collections for Utilities/Services

1. Customer initiates payment in your app (bill, subscription, deposit)
2. `POST /api/v1/collection_requests` with customer details
3. Onafriq sends in-app/USSD notification to customer
4. Customer authorizes payment in their mobile money app
5. Listen for `collection.received` webhook
6. Verify amount matches and fulfill service (activate, credit account)
7. Send confirmation SMS/email to customer

### Marketplace Seller Payouts

1. Aggregate seller earnings from transactions
2. Group by seller and currency
3. `POST /api/v1/create_payment` for each seller payout
4. Track payment status via webhooks
5. Notify seller of payout completion
6. Update seller wallet/ledger

## Error Handling

Onafriq returns consistent error responses:

```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Your wallet balance is insufficient for this transaction",
    "type": "validation_error"
  }
}
```

### Common HTTP Status Codes

| Code | Error | Action |
|------|-------|--------|
| 400 | Bad Request | Validation error (invalid phone, unsupported country, invalid amount). Check request format. |
| 401 | Unauthorized | Invalid or missing API key. Verify token in Authorization header. |
| 403 | Forbidden | Permission denied or account not enabled for this operation. Contact support. |
| 404 | Not Found | Resource not found (payment ID, collection request ID). Check ID format. |
| 422 | Unprocessable Entity | Validation failed (e.g., phone number format invalid, amount outside limits). |
| 429 | Too Many Requests | Rate limited. Implement exponential backoff (1s, 2s, 4s, 8s...). |
| 500 | Server Error | Service unavailable. Retry with exponential backoff. |
| 502 | Bad Gateway | Provider unavailable. Retry with exponential backoff. |
| 503 | Service Unavailable | Temporary outage. Retry with exponential backoff. |

### Common Error Codes

- `INSUFFICIENT_BALANCE` - Wallet balance too low for transaction
- `INVALID_MSISDN` - Phone number format invalid or unsupported
- `UNSUPPORTED_COUNTRY` - Country code not supported
- `UNSUPPORTED_PROVIDER` - Mobile money provider not available in country
- `INVALID_AMOUNT` - Amount outside provider limits
- `DUPLICATE_TRANSACTION` - Duplicate reference already processed
- `PAYMENT_FAILED` - Provider rejected payment
- `INVALID_API_KEY` - API key invalid or expired
- `RATE_LIMIT_EXCEEDED` - Too many requests

## Important Notes / Gotchas

### Rebrand: MFS Africa → Onafriq

MFS Africa rebranded to **Onafriq** in November 2023. The company name changed but the API endpoints, base URLs, and core functionality remain the same. Documentation and dashboards now reference "Onafriq" but some legacy references to "MFS Africa" may still exist. Both names refer to the same service.

### Supported Mobile Money Providers

Onafriq connects to 500M+ mobile money wallets across Africa. Major providers include:

- **M-Pesa** (Kenya, Tanzania, Uganda, DR Congo)
- **MTN Mobile Money** (Ghana, Uganda, Cameroon, Rwanda)
- **Airtel Money** (Kenya, Tanzania, Uganda, Rwanda, Malawi)
- **Vodacom M-Pesa** (Tanzania, DR Congo)
- **Equity Bank** (Kenya, Rwanda)
- **Bank transfers** (40+ banks across 20+ countries)
- **Cash agents** (Network of 300k+ agents)
- **Credit unions** and other providers

Coverage continues to expand. Check your account dashboard for the latest provider availability in your target countries.

### Supported Countries

Onafriq operates in 40+ African countries including: Kenya, Uganda, Tanzania, Rwanda, Burundi, Ghana, Nigeria, Cameroon, Ivory Coast, Senegal, Togo, South Africa, Zambia, Zimbabwe, Malawi, Botswana, Namibia, Lesotho, Eswatini, Mauritius, Seychelles, Sudan, Egypt, and more. Check your dashboard for current coverage.

### Phone Number Format

Always format phone numbers with country code and no leading zero:
- Correct: `+254701234567` (Kenya)
- Correct: `+256701234567` (Uganda)
- Incorrect: `+254 0701234567` (leading zero)
- Incorrect: `0701234567` (missing country code)

### Transaction Limits

Limits vary by country, provider, and customer tier:
- Minimum: Usually $0.50 USD equivalent or local currency minimum
- Maximum: Varies widely (can range from $100 to $10,000+ depending on provider and regulation)
- Daily/monthly limits may apply per phone number
- Check your account dashboard for specific limits per provider/country

### Fees

Onafriq charges fees per transaction. Fee structure depends on:
- Provider (M-Pesa, MTN, etc.)
- Destination country
- Amount
- Transaction type (mobile money, bank transfer, cash agent)

Fees are deducted from your wallet balance and returned in all API responses. Always display fees to end users and account for them in pricing calculations.

### Wallet Management

- Monitor your wallet balance before bulk disbursements
- Funds deducted immediately when transactions are created
- Failed transactions typically credit back within 24-48 hours
- Request wallet top-up/settlement from your account manager

### Account Setup Requirements

- Verify business identity and documentation
- Set up webhook endpoint in Dashboard
- Generate API key (keep secure, rotate periodically)
- Enable countries and providers as needed
- Contact Onafriq for cross-border or special capabilities

## Useful Links

- **Developer Portal:** https://developer.mfsafrica.com/
- **API Documentation:** https://developer.mfsafrica.com/docs
- **API Overview:** https://developer.mfsafrica.com/docs/api-overview
- **General API Information:** https://developer.mfsafrica.com/docs/combined-api-information
- **Collections API Docs:** https://developer.mfsafrica.com/docs/collections-api
- **Payouts Overview:** https://developer.mfsafrica.com/docs/overview-4
- **Async Payments API:** https://developer.mfsafrica.com/docs/async-payments-api
- **Webhooks Setup:** https://developer.mfsafrica.com/docs/webhooks
- **API Endpoints Reference:** https://developer.mfsafrica.com/docs/api-endpoints
- **Dashboard:** https://dashboard.mfsafrica.com
- **Status Page:** https://status.mfsafrica.com
- **Support:** https://support.mfsafrica.com

## Sources

This skill was updated with verified information from:
- [MFS Africa Developer Hub](https://developer.mfsafrica.com/)
- [API Overview](https://developer.mfsafrica.com/docs/api-overview)
- [General API Information](https://developer.mfsafrica.com/docs/combined-api-information)
- [Onafriq Official Site](https://onafriq.com/)
- [MFS Africa Rebrand Announcement](https://onafriq.com/press/article/mfs-africa-announces-rebrand-to-onafriq)
