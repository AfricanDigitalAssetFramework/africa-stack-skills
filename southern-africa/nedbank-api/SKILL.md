---
name: Nedbank API Marketplace
description: South Africa's first Open Banking platform providing RESTful APIs for accounts, payments, instant EFT transfers, and digital wallet services using OAuth 2.0 authentication.
triggers:
  - Nedbank
  - Nedbank API
  - South Africa open banking
  - SA banking API
  - Nedbank instant EFT
  - Nedbank payments API
  - Nedbank open banking
  - Africa banking API
---

## Overview

Nedbank API Marketplace is South Africa's first Open Banking platform, making it the first bank in Africa to provide APIs meeting open banking standards and PSD2 compliance. The marketplace offers a comprehensive suite of RESTful APIs that enable third-party developers and fintech partners to integrate with Nedbank's banking services securely. Whether you need to retrieve account information, initiate payments, or offer instant EFT transfers, Nedbank's API Marketplace provides production-grade banking APIs with industry-standard OAuth 2.0 security.

## When to Use This Skill

- **Account Information Access**: Retrieve account balances, transaction history, and customer account details from Nedbank
- **Payment Initiation**: Send money to any bank account in South Africa or move funds between Nedbank accounts
- **Instant EFT Transfers**: Integrate Nedbank Direct EFT for real-time electronic fund transfers from Nedbank accounts
- **Digital Wallet Operations**: Open and manage Nedbank MobiMoney wallet accounts with real-time transaction capabilities
- **Customer Data Integration**: Access verified customer information for KYC/AML compliance
- **Loan Management**: Query and manage personal loan products and applications
- **Rewards Integration**: Retrieve and manage Nedbank Greenbacks or Amex Membership Rewards for customers
- **Open Data Access**: Use freely available Branch and Bank List APIs without authentication

## Authentication

All Nedbank APIs use **OAuth 2.0** as the primary authentication mechanism, providing secure, standardized authorization flows for developers.

### OAuth 2.0 Authorization Code Flow

The standard three-legged OAuth 2.0 flow is used for accessing customer-specific data:

```
1. Authorization Request
   User is redirected to Nedbank's authorization endpoint

2. User Grants Consent
   Customer authenticates and approves access scopes

3. Authorization Code Exchange
   Your application exchanges the code for access token

4. API Requests
   Use the access token to call protected APIs
```

### Token Endpoint

```
POST https://api.nedbank.co.za/apimarket/sandbox/nboauth/oauth20/token
```

### OAuth 2.0 Request Example

```bash
curl -X POST https://api.nedbank.co.za/apimarket/sandbox/nboauth/oauth20/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE_FROM_REDIRECT" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "redirect_uri=YOUR_REDIRECT_URI"
```

### Required OAuth 2.0 Scopes

You must subscribe to the **Authorization API** and **Subscription API** before integrating with any other APIs. Common scopes include:

- `accounts` - Access account information and balances
- `transactions` - Retrieve transaction history
- `payments` - Initiate payment instructions
- `customer` - Access customer profile information

### Response Headers

Secure API responses include:
- `x-jws-signature` - JWS signature for payload verification
- `x-fapi-interaction-id` - RFC4122 UID used as correlation identifier for request tracking

## Core API Reference

### Accounts API

Retrieve account information including balances, details, and transaction history.

**Base URL**: `https://api.nedbank.co.za/apimarket/sandbox/open-banking`

#### Get Accounts

```http
GET /v3.1/accounts HTTP/1.1
Host: api.nedbank.co.za
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Response (200 OK)**:
```json
{
  "Data": {
    "Account": [
      {
        "AccountId": "50089234",
        "Currency": "ZAR",
        "Name": "Mr Kevin",
        "Subtype": "CurrentAccount",
        "Type": "Personal",
        "Account": [
          {
            "SchemeName": "SortCodeAccountNumber",
            "Identification": "40400140898283",
            "Name": "Mr Kevin",
            "SecondaryIdentification": "00002"
          }
        ]
      }
    ]
  },
  "Links": {
    "Self": "https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/accounts"
  },
  "Meta": {
    "TotalPages": 1
  }
}
```

#### Get Account Balances

```http
GET /v3.1/accounts/{AccountId}/balances HTTP/1.1
Host: api.nedbank.co.za
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Response (200 OK)**:
```json
{
  "Data": {
    "Balance": [
      {
        "Amount": {
          "Amount": "1230.00",
          "Currency": "ZAR"
        },
        "CreditDebitIndicator": "Credit",
        "Type": "Closing Available"
      }
    ]
  },
  "Links": {
    "Self": "https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/accounts/50089234/balances"
  },
  "Meta": {
    "TotalPages": 1
  }
}
```

#### Get Account Transactions

```http
GET /v3.1/accounts/{AccountId}/transactions HTTP/1.1
Host: api.nedbank.co.za
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Query Parameters**:
- `fromBookingDateTime` (optional) - ISO 8601 format: 2024-01-01T00:00:00Z
- `toBookingDateTime` (optional) - ISO 8601 format: 2024-12-31T23:59:59Z

**Response (200 OK)**:
```json
{
  "Data": {
    "Transaction": [
      {
        "AccountId": "50089234",
        "TransactionId": "TXN123456",
        "CreditDebitIndicator": "Credit",
        "Status": "Booked",
        "BookingDateTime": "2024-01-15T10:30:00Z",
        "ValueDateTime": "2024-01-15T10:30:00Z",
        "Amount": {
          "Amount": "150.50",
          "Currency": "ZAR"
        },
        "Description": "Salary Payment",
        "TransactionInformation": "Monthly Salary",
        "SupplementaryData": {
          "Reference": "SAL-2024-01"
        }
      }
    ]
  },
  "Links": {
    "Self": "https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/accounts/50089234/transactions"
  },
  "Meta": {
    "TotalPages": 1
  }
}
```

---

### Payments API

Initiate payments to any bank account or between Nedbank accounts.

#### Create Payment Consent

```http
POST /v3.1/pisp/domestic-payment-consents HTTP/1.1
Host: api.nedbank.co.za
Content-Type: application/json
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Request Body**:
```json
{
  "Data": {
    "Initiation": {
      "InstructionIdentification": "PAY-20240115-001",
      "EndToEndIdentification": "E2E-REF-123",
      "LocalInstrument": "UK.OBIE.SortCodeAccountNumber",
      "DebtorAccount": {
        "SchemeName": "SortCodeAccountNumber",
        "Identification": "40400140898283",
        "Name": "Mr Kevin",
        "SecondaryIdentification": "00002"
      },
      "CreditorAccount": {
        "SchemeName": "SortCodeAccountNumber",
        "Identification": "20000136000000",
        "Name": "Jane Doe"
      },
      "InstructedAmount": {
        "Amount": "500.00",
        "Currency": "ZAR"
      },
      "RemittanceInformation": {
        "Unstructured": "Invoice Payment #12345"
      }
    }
  },
  "Risk": {
    "PaymentContextCode": "ECommerce"
  }
}
```

**Response (201 Created)**:
```json
{
  "Data": {
    "ConsentId": "CONS-20240115-0001",
    "CreationDateTime": "2024-01-15T10:30:00Z",
    "Status": "AwaitingAuthorisation",
    "StatusUpdateDateTime": "2024-01-15T10:30:00Z"
  },
  "Links": {
    "Self": "https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/pisp/domestic-payment-consents/CONS-20240115-0001"
  },
  "Meta": {
    "TotalPages": 1
  }
}
```

#### Submit Payment

```http
POST /v3.1/pisp/domestic-payments HTTP/1.1
Host: api.nedbank.co.za
Content-Type: application/json
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
x-idempotency-key: {IDEMPOTENCY_KEY}
```

**Request Body**:
```json
{
  "Data": {
    "ConsentId": "CONS-20240115-0001",
    "Initiation": {
      "InstructionIdentification": "PAY-20240115-001",
      "EndToEndIdentification": "E2E-REF-123",
      "InstructedAmount": {
        "Amount": "500.00",
        "Currency": "ZAR"
      },
      "CreditorAccount": {
        "SchemeName": "SortCodeAccountNumber",
        "Identification": "20000136000000",
        "Name": "Jane Doe"
      },
      "RemittanceInformation": {
        "Unstructured": "Invoice Payment #12345"
      }
    }
  },
  "Risk": {
    "PaymentContextCode": "ECommerce"
  }
}
```

**Response (201 Created)**:
```json
{
  "Data": {
    "DomesticPaymentId": "DOM-PAY-20240115-001",
    "ConsentId": "CONS-20240115-0001",
    "CreationDateTime": "2024-01-15T10:35:00Z",
    "Status": "AcceptedSettlementInProcess",
    "StatusUpdateDateTime": "2024-01-15T10:35:00Z"
  },
  "Links": {
    "Self": "https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/pisp/domestic-payments/DOM-PAY-20240115-001"
  },
  "Meta": {
    "TotalPages": 1
  }
}
```

---

### Instant EFT API

Enable real-time electronic fund transfers directly from Nedbank accounts using Nedbank Direct EFT.

#### Initiate Instant EFT

```http
POST /v1/instant-eft/transfer HTTP/1.1
Host: api.nedbank.co.za
Content-Type: application/json
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
x-idempotency-key: {IDEMPOTENCY_KEY}
```

**Request Body**:
```json
{
  "DebtorAccount": {
    "AccountNumber": "1234567890",
    "AccountHolder": "John Smith"
  },
  "CreditorAccount": {
    "BankCode": "051001",
    "AccountNumber": "9876543210",
    "AccountHolder": "Jane Doe"
  },
  "TransactionAmount": {
    "Amount": "1500.00",
    "Currency": "ZAR"
  },
  "Reference": "INV-2024-001",
  "Description": "Payment for services rendered",
  "ChannelIndicator": "Mobile"
}
```

**Response (202 Accepted)**:
```json
{
  "TransactionId": "EFT-20240115-00001",
  "Status": "Processing",
  "CreatedDate": "2024-01-15T10:40:00Z",
  "Amount": {
    "Amount": "1500.00",
    "Currency": "ZAR"
  },
  "Reference": "INV-2024-001"
}
```

---

### Wallet API

Create and manage Nedbank MobiMoney wallet accounts with real-time payment capabilities.

#### Create Wallet

```http
POST /v2.3.0/wallets HTTP/1.1
Host: api.nedbank.co.za
Content-Type: application/json
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Request Body**:
```json
{
  "customerId": "CUST-2024-001",
  "walletType": "MobiMoney",
  "walletName": "My Nedbank Wallet",
  "currency": "ZAR",
  "metadata": {
    "region": "South Africa"
  }
}
```

**Response (201 Created)**:
```json
{
  "walletId": "WAL-20240115-001",
  "customerId": "CUST-2024-001",
  "walletType": "MobiMoney",
  "walletName": "My Nedbank Wallet",
  "status": "Active",
  "balance": {
    "available": "0.00",
    "currency": "ZAR"
  },
  "createdDate": "2024-01-15T10:45:00Z"
}
```

#### Get Wallet Details

```http
GET /v2.3.0/wallets/{walletId} HTTP/1.1
Host: api.nedbank.co.za
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Response (200 OK)**:
```json
{
  "walletId": "WAL-20240115-001",
  "customerId": "CUST-2024-001",
  "walletType": "MobiMoney",
  "walletName": "My Nedbank Wallet",
  "status": "Active",
  "balance": {
    "available": "2500.00",
    "currency": "ZAR"
  },
  "lastTransactionDate": "2024-01-15T11:30:00Z"
}
```

#### Wallet-to-Wallet Transfer

```http
POST /v2.3.0/wallets/{walletId}/transfer HTTP/1.1
Host: api.nedbank.co.za
Content-Type: application/json
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
x-idempotency-key: {IDEMPOTENCY_KEY}
```

**Request Body**:
```json
{
  "recipientWalletId": "WAL-20240115-002",
  "amount": {
    "amount": "250.00",
    "currency": "ZAR"
  },
  "reference": "PEER-PAYMENT-001",
  "description": "Shared lunch expense"
}
```

**Response (202 Accepted)**:
```json
{
  "transactionId": "TXN-20240115-001",
  "fromWalletId": "WAL-20240115-001",
  "toWalletId": "WAL-20240115-002",
  "amount": {
    "amount": "250.00",
    "currency": "ZAR"
  },
  "status": "Pending",
  "createdDate": "2024-01-15T11:35:00Z"
}
```

---

### Customer Information API

Access verified customer profile data for KYC/AML compliance.

```http
GET /v1/customer/{customerId} HTTP/1.1
Host: api.nedbank.co.za
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Response (200 OK)**:
```json
{
  "CustomerId": "CUST-2024-001",
  "FirstName": "John",
  "MiddleName": "Michael",
  "LastName": "Smith",
  "Email": "john.smith@example.com",
  "MobileNumber": "+27711234567",
  "DateOfBirth": "1985-03-15",
  "Nationality": "ZA",
  "DocumentType": "NationalId",
  "DocumentNumber": "8503151234081",
  "ResidentialAddress": {
    "AddressLine1": "123 Main Street",
    "AddressLine2": "Johannesburg",
    "City": "Johannesburg",
    "PostalCode": "2000",
    "Country": "ZA"
  },
  "VerificationStatus": "Verified",
  "VerificationDate": "2024-01-10T09:00:00Z"
}
```

---

### Personal Loans API

Query and manage personal loan applications.

```http
GET /v1/personal-loans HTTP/1.1
Host: api.nedbank.co.za
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Response (200 OK)**:
```json
{
  "Data": {
    "PersonalLoan": [
      {
        "PersonalLoanId": "LOAN-20240101-001",
        "PLStatus": "Active",
        "PLOfferId": "OFFER-12345",
        "LoanAmount": {
          "Amount": "50000.00",
          "Currency": "ZAR"
        },
        "InterestRate": 10.5,
        "LoanTerm": 60,
        "MonthlyInstallment": {
          "Amount": "943.56",
          "Currency": "ZAR"
        },
        "StartDate": "2024-01-01",
        "MaturityDate": "2028-12-31",
        "RemainingBalance": {
          "Amount": "48500.00",
          "Currency": "ZAR"
        }
      }
    ]
  },
  "Links": {
    "Self": "https://api.nedbank.co.za/apimarket/sandbox/personal-loans"
  },
  "Meta": {
    "TotalPages": 1
  }
}
```

---

### Rewards API

Retrieve customer rewards points (Greenbacks or Amex Membership Rewards).

```http
GET /v1/rewards/{customerId} HTTP/1.1
Host: api.nedbank.co.za
Authorization: Bearer {access_token}
x-fapi-interaction-id: {UUID}
```

**Response (200 OK)**:
```json
{
  "CustomerId": "CUST-2024-001",
  "RewardProgram": "Greenbacks",
  "TotalPoints": 15750,
  "AvailablePoints": 15750,
  "PendingPoints": 0,
  "ExpiringPoints": 250,
  "ExpiryDate": "2024-12-31T23:59:59Z",
  "LastTransactionDate": "2024-01-10T14:20:00Z",
  "LastTransactionAmount": 150.00,
  "PointsEarned": 15
}
```

---

### Open Data APIs (No Authentication Required)

Free APIs providing public banking information without OAuth authentication.

#### Branch List API

```http
GET /v1/branches HTTP/1.1
Host: api.nedbank.co.za
```

**Response (200 OK)**:
```json
{
  "Data": {
    "Branch": [
      {
        "BranchId": "BRANCH-JNB-001",
        "BranchCode": "051001",
        "BranchName": "Johannesburg Main",
        "Address": "100 Grayston Drive, Johannesburg, 2146",
        "City": "Johannesburg",
        "Province": "Gauteng",
        "PostalCode": "2146",
        "Phone": "+27113214567",
        "OperatingHours": {
          "Monday": "08:00-16:00",
          "Tuesday": "08:00-16:00",
          "Wednesday": "08:00-16:00",
          "Thursday": "08:00-16:00",
          "Friday": "08:00-16:00",
          "Saturday": "09:00-13:00"
        }
      }
    ]
  }
}
```

#### Bank List API

```http
GET /v1/banks HTTP/1.1
Host: api.nedbank.co.za
```

**Response (200 OK)**:
```json
{
  "Data": {
    "Bank": [
      {
        "BankId": "BANK-ZA-001",
        "BankCode": "051001",
        "BankName": "Nedbank Limited",
        "Country": "ZA",
        "Established": 1888
      }
    ]
  }
}
```

## Webhooks and Notifications

Nedbank APIs support event-driven integrations through webhooks for asynchronous notifications. Configure webhook endpoints in your developer portal to receive real-time notifications about:

- **Payment Status Updates** - When initiated payments change status (Pending → Booked → Failed)
- **Transaction Notifications** - New transactions posted to accounts
- **Wallet Events** - Wallet balance changes and transfer completions
- **Loan Updates** - Changes to loan status or payment due dates

### Webhook Event Structure

```json
{
  "eventId": "EVT-20240115-001",
  "eventType": "payment.status.updated",
  "timestamp": "2024-01-15T10:50:00Z",
  "resourceId": "DOM-PAY-20240115-001",
  "data": {
    "status": "AcceptedSettlementInProcess",
    "previousStatus": "AwaitingAuthorisation",
    "amount": {
      "amount": "500.00",
      "currency": "ZAR"
    }
  }
}
```

### Webhook Registration

Register webhooks in the Nedbank API Marketplace developer portal:
1. Navigate to your Application Settings
2. Select Webhooks configuration
3. Provide your HTTPS endpoint URL
4. Select events you want to subscribe to
5. Ensure your endpoint responds with `200 OK` within 10 seconds

## Common Integration Patterns

### Pattern 1: Check Account Balance

A simple read-only integration to display customer account information:

```javascript
async function getAccountBalance(accountId, accessToken) {
  const response = await fetch(
    `https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/accounts/${accountId}/balances`,
    {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'x-fapi-interaction-id': generateUUID()
      }
    }
  );

  const data = await response.json();
  return data.Data.Balance[0].Amount;
}
```

### Pattern 2: Initiate Payment with Consent

A complete payment flow with user authorization:

```javascript
// Step 1: Create payment consent
async function createPaymentConsent(paymentDetails, accessToken) {
  const response = await fetch(
    'https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/pisp/domestic-payment-consents',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
        'x-fapi-interaction-id': generateUUID()
      },
      body: JSON.stringify({
        Data: {
          Initiation: {
            InstructionIdentification: paymentDetails.reference,
            InstructedAmount: {
              Amount: paymentDetails.amount,
              Currency: 'ZAR'
            },
            CreditorAccount: {
              SchemeName: 'SortCodeAccountNumber',
              Identification: paymentDetails.recipientAccount
            }
          }
        }
      })
    }
  );
  return await response.json();
}

// Step 2: Submit payment after user authorization
async function submitPayment(consentId, accessToken, idempotencyKey) {
  const response = await fetch(
    'https://api.nedbank.co.za/apimarket/sandbox/open-banking/v3.1/pisp/domestic-payments',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
        'x-fapi-interaction-id': generateUUID(),
        'x-idempotency-key': idempotencyKey
      },
      body: JSON.stringify({
        Data: { ConsentId: consentId }
      })
    }
  );
  return await response.json();
}
```

### Pattern 3: Real-Time Wallet Transfer

Peer-to-peer wallet payments:

```javascript
async function transferBetweenWallets(sourceWalletId, targetWalletId, amount, accessToken) {
  const response = await fetch(
    `https://api.nedbank.co.za/apimarket/sandbox/v2.3.0/wallets/${sourceWalletId}/transfer`,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
        'x-fapi-interaction-id': generateUUID(),
        'x-idempotency-key': generateUUID()
      },
      body: JSON.stringify({
        recipientWalletId: targetWalletId,
        amount: {
          amount: amount.toString(),
          currency: 'ZAR'
        },
        reference: generateReference()
      })
    }
  );
  return await response.json();
}
```

### Pattern 4: Instant EFT from App

Initiate real-time bank transfers:

```javascript
async function initiateInstantEFT(debtorAccount, creditorAccount, amount, accessToken) {
  const response = await fetch(
    'https://api.nedbank.co.za/apimarket/sandbox/v1/instant-eft/transfer',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
        'x-fapi-interaction-id': generateUUID(),
        'x-idempotency-key': generateUUID()
      },
      body: JSON.stringify({
        DebtorAccount: debtorAccount,
        CreditorAccount: creditorAccount,
        TransactionAmount: {
          Amount: amount.toString(),
          Currency: 'ZAR'
        },
        ChannelIndicator: 'Mobile'
      })
    }
  );
  return await response.json();
}
```

## Error Handling

Nedbank APIs use standard HTTP status codes with detailed error responses. Always implement proper error handling in production:

### Success Responses

- **200 OK** - Successful GET, PUT, or POST request with response body
- **201 Created** - Successful POST request creating a resource
- **202 Accepted** - Request accepted for asynchronous processing (payments)
- **204 No Content** - Successful request with no response body

### Client Error Responses

- **400 Bad Request** - Malformed request syntax or invalid parameters
- **401 Unauthorized** - Missing or invalid OAuth 2.0 access token
- **403 Forbidden** - Valid token but insufficient permissions for resource
- **404 Not Found** - Requested resource does not exist
- **422 Unprocessable Entity** - Request validation failed (invalid amount, account, etc.)
- **429 Too Many Requests** - Rate limit exceeded

### Server Error Responses

- **500 Internal Server Error** - Unexpected server error
- **502 Bad Gateway** - API gateway error
- **503 Service Unavailable** - API temporarily unavailable
- **504 Gateway Timeout** - Request timeout

### Error Response Example

```json
{
  "Errors": [
    {
      "ErrorCode": "UK.OBIE.Field.Invalid",
      "Message": "Invalid account number",
      "Path": "Data.Initiation.CreditorAccount.Identification",
      "Url": "https://openbanking.atlassian.net/wiki/display/DZ/Data+Validation+Errors"
    }
  ]
}
```

### Error Handling Best Practices

```javascript
async function callNebankAPI(url, options) {
  const maxRetries = 3;
  let retryCount = 0;

  while (retryCount < maxRetries) {
    try {
      const response = await fetch(url, options);

      if (response.ok) {
        return await response.json();
      }

      if (response.status === 429) {
        // Rate limited - wait and retry
        const retryAfter = response.headers.get('Retry-After') || 60;
        await sleep(parseInt(retryAfter) * 1000);
        retryCount++;
        continue;
      }

      if (response.status >= 500) {
        // Server error - retry with backoff
        await sleep(Math.pow(2, retryCount) * 1000);
        retryCount++;
        continue;
      }

      // Client error - don't retry
      const errorData = await response.json();
      throw new Error(`API Error (${response.status}): ${JSON.stringify(errorData.Errors)}`);

    } catch (error) {
      if (retryCount === maxRetries - 1) throw error;
      retryCount++;
    }
  }
}
```

## Important Notes and Gotchas

1. **Partnership Registration Required** - You must register as a Nedbank API partner before accessing production APIs. Use the "Register your interest" button on https://apim.nedbank.co.za/ and a sales consultant will contact you with onboarding details.

2. **Sandbox vs Production** - Always test thoroughly in the sandbox environment (`https://api.nedbank.co.za/apimarket/sandbox/`) before moving to production. Sandbox and production use separate OAuth credentials and have isolated data.

3. **ZAR Currency Default** - All Nedbank APIs in South Africa operate in ZAR (South African Rand). Specify currency explicitly in requests: `"Currency": "ZAR"`. Amounts must be formatted as strings with two decimal places: `"Amount": "1500.00"`.

4. **OAuth Token Expiration** - Access tokens have limited lifetimes (typically 1 hour). Implement token refresh logic to obtain new tokens when expired using the `refresh_token` grant type:
   ```
   grant_type=refresh_token&refresh_token=YOUR_REFRESH_TOKEN
   ```

5. **Idempotency Keys Required for Payments** - Always include a unique `x-idempotency-key` header when initiating payments or transfers to prevent duplicate submissions if requests are retried. Use a UUID or hash of the transaction details.

6. **Subscription Required for Each API** - You must explicitly subscribe to each API you want to use in the developer portal. You cannot use APIs without an active subscription. This includes the Authorization API, which is required before using any other APIs.

7. **Rate Limits Applied Per App** - Nedbank reserves the right to enforce rate limits on API calls. The platform tracks usage per application, and limits can be enforced by number of requests, frequency, currency amounts, data volume, or other metrics. Monitor your API usage in the developer portal dashboard.

8. **Account Identification Schemes** - Different APIs support different account identification schemes. For South African accounts, use `"SchemeName": "SortCodeAccountNumber"` or `"SchemeName": "IBAN"` for international accounts. Reference the specific API documentation for supported schemes.

9. **User Authorization Required for Account Data** - Accessing customer account information and transaction history requires explicit user consent through the OAuth authorization flow. Users must authenticate and approve specific data scopes before your app can access their accounts.

10. **Webhook Endpoint Requirements** - Webhook endpoints must be publicly accessible HTTPS URLs that respond with `200 OK` within 10 seconds. Implement exponential backoff if your endpoint is temporarily unavailable. Nedbank will retry failed webhook deliveries up to 5 times.

11. **PSD2 Compliance** - Nedbank's API Marketplace is PSD2-compliant for applicable APIs. This means specific transaction limits may apply, and Strong Customer Authentication (SCA) may be required for sensitive operations. Consult the PSD2 section of the documentation for your jurisdiction.

12. **Data Retention and Privacy** - Customer data accessed through Nedbank APIs is governed by South African data protection laws and Nedbank's privacy policy. Only request and retain necessary data, obtain explicit consent for data usage, and securely store credentials and access tokens.

## Useful Links

- **API Marketplace Portal**: https://apim.nedbank.co.za/
- **Alternative Africa Portal**: https://apim.nedbank.africa/
- **Getting Started Guide**: https://apim.nedbank.co.za/static/docs/start
- **Account Registration**: https://apim.nedbank.co.za/static/docs/register
- **Documentation Hub**: https://apim.nedbank.co.za/static/docs
- **Available APIs Catalog**: https://apim.nedbank.co.za/static/products
- **Open Banking Documentation**: https://apim.nedbank.co.za/static/docs/ob
- **Open Data APIs**: https://apim.nedbank.co.za/static/opendata
- **Payments API Documentation**: https://apim.nedbank.co.za/static/payment
- **Accounts API Documentation**: https://apim.nedbank.co.za/static/account
- **Wallet API Documentation**: https://apim.nedbank.co.za/static/wallet
- **Payments Test Cases & Error Codes**: https://apim.nedbank.co.za/static/docs/payments-testcases
- **API Marketplace FAQs**: https://apim.nedbank.africa/faq.html
- **Nedbank Blog - API Updates**: https://personal.nedbank.co.za/learn/blog/
- **Developer Support**: Available through 24/7 helpdesk and online chat on the API Marketplace

---

**Last Updated**: February 2026

**Compatibility**: OAuth 2.0, REST APIs, JSON request/response format

**Supported Regions**: South Africa, Namibia, Lesotho, eSwatini (for EFT services)

**Sandbox Base URL**: `https://api.nedbank.co.za/apimarket/sandbox/`

**Portal URL**: `https://apim.nedbank.co.za/`
