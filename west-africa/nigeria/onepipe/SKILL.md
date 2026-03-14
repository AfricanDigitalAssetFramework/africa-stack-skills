---
name: OnePipe
description: Nigerian open banking API aggregator and financial middleware platform. OnePipe wraps multiple bank and fintech provider APIs into a unified abstraction layer, enabling single-integration access to card charging, account debit, bill payments, airtime purchase, instant loans, and KYC lookup services.
triggers:
  - "OnePipe"
  - "Nigeria open banking"
  - "bank API aggregation"
  - "financial API middleware Nigeria"
  - "open banking aggregator"
  - "unified fintech API"
---

# OnePipe API Integration Guide

OnePipe is a Nigerian fintech infrastructure platform that acts as an API aggregator and middleware for multiple banks and payment providers. Instead of integrating with individual bank APIs (Polaris, SunTrust, Fidelity, Providus, Flutterwave, Paystack, etc.), developers integrate once with OnePipe and gain access to standardized financial services across all partner institutions.

The platform's core value proposition is provider abstraction: write integration code once and switch between underlying providers via simple configuration parameters without changing your implementation.

## When to Use This Skill

Use OnePipe when you need to:

- **Aggregate multiple bank APIs** into a single standardized interface
- **Accept bank transfers** as a payment method without managing multiple provider integrations
- **Perform account debit transactions** across multiple Nigerian banks
- **Look up customer identity information** (NIN/BVN) for KYC purposes
- **Retrieve bank statements** and account information
- **Process airtime and bill payments** through a unified API
- **Access instant loan services** across partner fintech providers
- **Issue instant business accounts** to customers
- **Switch between payment providers** without code changes
- **Build payment infrastructure** in Nigeria with minimal provider friction

This is particularly valuable for Nigerian fintech companies, payment platforms, lending platforms, and businesses that need multi-provider financial service access without managing separate integrations.

## Authentication

OnePipe uses API key-based authentication combined with request signing for transaction security.

### Authentication Headers

All API requests to OnePipe require the following headers:

```http
Authorization: Bearer YOUR_API_KEY
Signature: SIGNATURE_VALUE
Content-Type: application/json
```

### Getting API Credentials

OnePipe does not offer self-service API key generation. To integrate:

1. **Email integration@onepipe.io** with details about your use case
2. **Wait for OnePipe team response** - they will provide:
   - API key and secret
   - Postman collection configured for your environment
   - Integration instructions and sample requests
   - Sandbox environment details

### Request Signing

The `Signature` header uses HMAC-SHA256 signing. The signature is calculated over the request body:

```javascript
// Example Node.js signature generation
const crypto = require('crypto');

function generateSignature(requestBody, secret) {
  return crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(requestBody))
    .digest('hex');
}
```

### Authentication Example

```bash
curl -X POST https://api.onepipe.io/v2/transact \
  -H "Authorization: Bearer sk_test_abc123xyz" \
  -H "Signature: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6" \
  -H "Content-Type: application/json" \
  -d @request.json
```

## Core API Reference

### Base URL

```
https://api.onepipe.io/v2
```

### Primary Endpoint

All transactional requests go to a single endpoint with the operation type specified in the request body:

```http
POST /transact
```

### Request Structure

All OnePipe requests follow this structure:

```json
{
  "request_ref": "unique_request_identifier_12345",
  "request_type": "operation_type_here",
  "auth": {
    "type": "api_key",
    "secure": true,
    "auth_provider": "onepipe",
    "route_mode": "live"
  },
  "transaction": {
    "mock_mode": false,
    "transaction_ref": "txn_ref_unique_12345",
    "transaction_desc": "Brief description of transaction",
    "amount": 50000,
    "customer": {
      "customer_ref": "cust_abc123",
      "firstname": "John",
      "lastname": "Doe",
      "email": "john@example.com",
      "phone": "+2348012345678"
    },
    "meta": {
      "account_number": "0123456789",
      "bank_code": "050",
      "account_bank": "Polaris Bank"
    },
    "details": {
      "device": "WEB",
      "purpose": "General Purpose"
    }
  }
}
```

### Common Request Types and Endpoints

#### 1. Open Account

Creates a new business account at a partner bank.

```json
{
  "request_ref": "req_open_acct_001",
  "request_type": "open_account",
  "auth": {
    "type": "api_key",
    "secure": true,
    "auth_provider": "onepipe",
    "route_mode": "live"
  },
  "transaction": {
    "transaction_ref": "txn_open_001",
    "transaction_desc": "Open business account",
    "customer": {
      "firstname": "Jane",
      "lastname": "Smith",
      "email": "jane@business.com",
      "phone": "+2349012345678"
    },
    "details": {
      "bank": "polaris"
    }
  }
}
```

**Response:**

```json
{
  "status": "SUCCESS",
  "code": "00",
  "message": "Account opened successfully",
  "data": {
    "account_number": "0123456789",
    "bank_code": "050",
    "bank_name": "Polaris Bank",
    "account_name": "Jane Smith",
    "account_reference": "acct_ref_12345"
  }
}
```

#### 2. Get Accounts

Retrieve all accounts associated with a customer.

```json
{
  "request_ref": "req_get_accts_001",
  "request_type": "get_accounts_min",
  "auth": {
    "type": "api_key",
    "secure": true,
    "auth_provider": "onepipe",
    "route_mode": "live"
  },
  "transaction": {
    "transaction_ref": "txn_get_001",
    "customer": {
      "customer_ref": "cust_12345"
    }
  }
}
```

**Response:**

```json
{
  "status": "SUCCESS",
  "code": "00",
  "message": "Accounts retrieved",
  "data": {
    "accounts": [
      {
        "account_number": "0123456789",
        "bank_code": "050",
        "bank_name": "Polaris Bank",
        "account_name": "Jane Smith",
        "account_type": "BUSINESS"
      }
    ]
  }
}
```

#### 3. Account Debit / Transfer

Debit funds from a customer's account.

```json
{
  "request_ref": "req_debit_001",
  "request_type": "account_debit",
  "auth": {
    "type": "api_key",
    "secure": true,
    "auth_provider": "onepipe",
    "route_mode": "live"
  },
  "transaction": {
    "transaction_ref": "txn_debit_001",
    "transaction_desc": "Payment for order #12345",
    "amount": 50000,
    "customer": {
      "customer_ref": "cust_12345",
      "firstname": "Jane",
      "lastname": "Smith"
    },
    "meta": {
      "account_number": "0123456789",
      "bank_code": "050"
    },
    "details": {
      "beneficiary_name": "Vendor Inc",
      "beneficiary_bank": "GTBank",
      "beneficiary_account": "9876543210"
    }
  }
}
```

**Response:**

```json
{
  "status": "SUCCESS",
  "code": "00",
  "message": "Transaction successful",
  "data": {
    "transaction_ref": "txn_debit_001",
    "session_id": "sess_abc123xyz",
    "amount": 50000,
    "timestamp": "2024-02-20T14:32:15Z",
    "reference_number": "NIP123456789"
  }
}
```

#### 4. Get Statement

Retrieve account statement/transaction history.

```json
{
  "request_ref": "req_stmt_001",
  "request_type": "get_statement",
  "auth": {
    "type": "api_key",
    "secure": true,
    "auth_provider": "onepipe",
    "route_mode": "live"
  },
  "transaction": {
    "transaction_ref": "txn_stmt_001",
    "customer": {
      "customer_ref": "cust_12345"
    },
    "meta": {
      "account_number": "0123456789",
      "bank_code": "050",
      "from_date": "2024-01-01",
      "to_date": "2024-02-29",
      "page": 1,
      "limit": 50
    }
  }
}
```

**Response:**

```json
{
  "status": "SUCCESS",
  "code": "00",
  "message": "Statement retrieved",
  "data": {
    "account_number": "0123456789",
    "statement": [
      {
        "date": "2024-02-20",
        "narration": "TRANSFER IN",
        "amount": 100000,
        "balance": 500000,
        "transaction_ref": "NIP987654321"
      }
    ]
  }
}
```

#### 5. NIN/BVN Lookup

Look up customer identity information for KYC.

```json
{
  "request_ref": "req_nin_001",
  "request_type": "lookup_nin_mid",
  "auth": {
    "type": "api_key",
    "secure": true,
    "auth_provider": "onepipe",
    "route_mode": "live"
  },
  "transaction": {
    "transaction_ref": "txn_nin_001",
    "customer": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "meta": {
      "nin": "12345678901",
      "bvn": "22191810283"
    }
  }
}
```

**Response:**

```json
{
  "status": "SUCCESS",
  "code": "00",
  "message": "Identity verification successful",
  "data": {
    "firstname": "John",
    "lastname": "Doe",
    "middle_name": "Ade",
    "date_of_birth": "1990-05-15",
    "phone": "+2348012345678",
    "nin": "12345678901",
    "photo": "base64_encoded_image_data",
    "verification_status": "verified"
  }
}
```

#### 6. Airtime Purchase

Buy airtime for mobile numbers.

```json
{
  "request_ref": "req_airtime_001",
  "request_type": "buy_airtime",
  "auth": {
    "type": "api_key",
    "secure": true,
    "auth_provider": "onepipe",
    "route_mode": "live"
  },
  "transaction": {
    "transaction_ref": "txn_airtime_001",
    "amount": 1000,
    "customer": {
      "customer_ref": "cust_12345"
    },
    "meta": {
      "phone_number": "+2348012345678",
      "provider": "MTN"
    }
  }
}
```

**Response:**

```json
{
  "status": "SUCCESS",
  "code": "00",
  "message": "Airtime purchase successful",
  "data": {
    "transaction_ref": "txn_airtime_001",
    "phone_number": "+2348012345678",
    "amount": 1000,
    "provider": "MTN",
    "timestamp": "2024-02-20T14:35:22Z"
  }
}
```

#### 7. Query Transaction Status

Check the status of a previously submitted transaction.

```http
GET /transact/query?request_ref=req_debit_001&transaction_ref=txn_debit_001
Authorization: Bearer YOUR_API_KEY
Signature: SIGNATURE_VALUE
```

**Response:**

```json
{
  "status": "SUCCESS",
  "code": "00",
  "message": "Transaction found",
  "data": {
    "transaction_ref": "txn_debit_001",
    "status": "SUCCESS",
    "amount": 50000,
    "timestamp": "2024-02-20T14:32:15Z",
    "response_code": "00",
    "response_message": "Transaction successful"
  }
}
```

## Webhooks

OnePipe uses webhooks to notify your application of transaction events and updates. OnePipe transforms provider-specific webhooks into a standardized payload format.

### Webhook Registration

To receive webhooks:

1. **Provide webhook URL** to OnePipe integration team
2. **Ensure endpoint is publicly accessible** and returns HTTP 200-299 within 10 seconds
3. **Implement signature verification** to validate webhook authenticity

### Webhook Payload Structure

```json
{
  "event_type": "transaction.completed",
  "webhook_id": "whk_abc123xyz",
  "timestamp": "2024-02-20T14:32:15Z",
  "data": {
    "transaction_ref": "txn_debit_001",
    "request_ref": "req_debit_001",
    "status": "SUCCESS",
    "code": "00",
    "message": "Transaction completed",
    "amount": 50000,
    "currency": "NGN",
    "customer_ref": "cust_12345",
    "account_number": "0123456789",
    "bank_code": "050",
    "meta": {
      "provider": "polaris",
      "provider_reference": "NIP123456789"
    }
  }
}
```

### Webhook Events

- **transaction.completed** - Transaction processed successfully
- **transaction.failed** - Transaction failed or was rejected
- **account.created** - New account successfully opened
- **transfer.received** - Funds received on monitored account
- **statement.updated** - New statement data available

### Webhook Verification

Verify webhook signatures to ensure authenticity:

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const calculatedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');

  return calculatedSignature === signature;
}
```

## Common Integration Patterns

### Pattern 1: Pay with Transfer

Allow customers to pay you via bank transfer to a dedicated account:

```javascript
// 1. Open customer-specific account
const openAccountRequest = {
  request_ref: `req_${Date.now()}`,
  request_type: "open_account",
  // ... auth details ...
  transaction: {
    transaction_ref: `txn_${Date.now()}`,
    customer: customerData
  }
};

// 2. Receive webhook when funds arrive
// Webhook with event_type: "transfer.received"

// 3. Query statement to confirm
const statementRequest = {
  request_ref: `req_stmt_${Date.now()}`,
  request_type: "get_statement",
  // ... auth details ...
};

// 4. Match received amounts to orders and reconcile
```

### Pattern 2: Multi-Provider Fallback

Switch providers if one fails:

```javascript
async function debitAccount(transaction, primaryProvider = 'polaris') {
  const providers = ['polaris', 'suntrust', 'fidelity'];

  for (const provider of providers) {
    try {
      const request = {
        ...transaction,
        auth: {
          ...transaction.auth,
          route_mode: 'live',
          provider_preference: provider
        }
      };

      const response = await callOnePipe(request);
      if (response.status === 'SUCCESS') {
        return response;
      }
    } catch (error) {
      console.log(`Provider ${provider} failed, trying next...`);
      continue;
    }
  }

  throw new Error('All providers failed');
}
```

### Pattern 3: Verify Customer Before Transaction

Always perform KYC before accepting payments:

```javascript
async function verifyAndDebit(nin, amount) {
  // Step 1: Verify identity
  const verification = await onepipe.lookupNIN(nin);
  if (verification.status !== 'SUCCESS') {
    throw new Error('Verification failed');
  }

  // Step 2: Get customer accounts
  const accounts = await onepipe.getAccounts(verification.data.customer_ref);

  // Step 3: Perform debit
  return await onepipe.debitAccount({
    customer_ref: verification.data.customer_ref,
    account_number: accounts.data.accounts[0].account_number,
    amount: amount
  });
}
```

### Pattern 4: Idempotent Requests

Always use unique `request_ref` and `transaction_ref` for idempotency:

```javascript
function generateRequestId(userId, action, timestamp) {
  // Ensure same transaction produces same IDs
  return `${userId}_${action}_${timestamp}`;
}

// Always query first to check if already processed
async function idempotentDebit(userId, amount, timestamp) {
  const requestRef = generateRequestId(userId, 'debit', timestamp);

  // Check if already processed
  const existing = await onepipe.queryTransaction(requestRef);
  if (existing.status === 'SUCCESS') {
    return existing.data;
  }

  // Process new transaction
  return await onepipe.debitAccount({
    request_ref: requestRef,
    amount: amount,
    customer_ref: userId
  });
}
```

## Error Handling

### Standard Response Codes

| Code | Message | Meaning |
|------|---------|---------|
| `00` | SUCCESS | Transaction processed successfully |
| `01` | PENDING | Transaction still processing |
| `02` | FAILED | Transaction failed |
| `03` | INVALID_REQUEST | Request format or validation error |
| `04` | UNAUTHORIZED | Authentication failed |
| `05` | INSUFFICIENT_FUNDS | Account has insufficient balance |
| `06` | ACCOUNT_NOT_FOUND | Account number invalid or not found |
| `07` | DUPLICATE_TRANSACTION | Duplicate transaction detected |
| `08` | TIMEOUT | Request timeout |
| `09` | SERVICE_UNAVAILABLE | Provider service unavailable |
| `10` | INVALID_AMOUNT | Amount is invalid or too large |

### Error Response Example

```json
{
  "status": "FAILED",
  "code": "05",
  "message": "Insufficient funds in account",
  "data": {
    "request_ref": "req_debit_001",
    "transaction_ref": "txn_debit_001",
    "error_details": {
      "account_balance": 30000,
      "requested_amount": 50000,
      "required_balance": 50000
    }
  }
}
```

### Recommended Error Handling Logic

```javascript
async function handleOnePipeError(error, transaction) {
  const errorCode = error.code;

  switch (errorCode) {
    case '01': // PENDING
      // Poll for status after 5 seconds
      return await retryAfterDelay(transaction, 5000);

    case '05': // INSUFFICIENT_FUNDS
      // Inform user and suggest alternative payment method
      return { retry: false, reason: 'Insufficient funds' };

    case '07': // DUPLICATE_TRANSACTION
      // Return success - already processed
      return { retry: false, already_processed: true };

    case '09': // SERVICE_UNAVAILABLE
      // Retry with exponential backoff
      return await retryWithBackoff(transaction, 3);

    default:
      // Log and notify support for investigation
      logger.error('OnePipe error', { error, transaction });
      throw error;
  }
}
```

## Important Notes / Gotchas

1. **No Self-Service Onboarding**: Unlike most APIs, OnePipe requires manual approval and setup. Email integration@onepipe.io and wait for their team to contact you with credentials, Postman collection, and documentation. There is no automated signup.

2. **Limited Public Documentation**: OnePipe's full API documentation (v2.docs.onepipe.io) is not directly accessible. The integration team provides complete documentation via Postman collection and direct communication. Rely on the Postman collection they provide rather than assuming endpoint availability.

3. **Signature Validation Required**: Every request requires both an API key AND a request signature (HMAC-SHA256 of the request body). Missing or incorrect signatures will cause authentication failures. The integration team will provide signing guidelines specific to your SDK/language.

4. **Request/Transaction IDs Are Critical**: Always generate unique `request_ref` and `transaction_ref` values. OnePipe uses these for idempotency and duplicate detection. Reusing IDs will cause "DUPLICATE_TRANSACTION" errors even if the previous request timed out.

5. **Provider Routing Must Be Configured**: While OnePipe abstracts multiple providers (Polaris, SunTrust, Fidelity, Providus, Flutterwave, Paystack), your integration must specify which provider to route to. This is usually handled via environment configuration provided by the integration team rather than per-request parameters.

6. **Webhooks May Be Delayed**: Webhook delivery is not guaranteed to be immediate. Transaction status should be verified via polling the `/transact/query` endpoint rather than relying solely on webhooks. Implement both webhook handling AND periodic polling for critical operations.

7. **Account Opening Has Prerequisites**: The `open_account` endpoint requires customer identity verification (NIN/BVN lookup) before account creation in most cases. Verify identity first, then create accounts. Some banks may have additional KYC requirements that delay account activation.

8. **Bank Codes Are Required for Transfers**: When specifying destination accounts for transfers, always include the correct NIBSS bank code (e.g., "050" for Polaris). Invalid bank codes will cause transaction failures. Maintain a mapping of bank names to official NIBSS codes.

9. **Mock Mode for Testing**: During integration, requests can include `mock_mode: true` in the transaction object to use the test Postman environment. The integration team provides a mock base URL for non-live testing. Never use mock_mode in production.

10. **Amount Values Are in Smallest Currency Units**: Amounts are specified in kobo (smallest NGN unit), not naira. An amount of 50000 means 500 NGN. Always multiply user-facing amounts by 100 before sending to the API.

## Useful Links

- [OnePipe Official Website](https://www.onepipe.io/)
- [OnePipe v2 Documentation](https://v2.docs.onepipe.io/)
- [OnePipe v1 Documentation](https://v1.docs.onepipe.io/)
- [OnePipe on Postman](https://www.postman.com/onepipe/workspace/onepipe-s-public-workspace/collection/6358444-bc6887b7-b4bc-4790-8051-48635f1238b9)
- [OnePipe GitHub Examples](https://github.com/9trocode/Onepipe-api)
- [OnePipe Products Overview](https://onepipe.com/products/)
- [OnePipe Payments](https://onepipe.com/payments/)
- [OnePipe for Banks](https://onepipe.com/banks/)
- [Open Banking Nigeria Initiative](https://openbanking.ng/)
- **Integration Email**: integration@onepipe.io

---

*Last Updated: February 2026*
