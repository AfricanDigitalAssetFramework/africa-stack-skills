---
triggers:
  - "MTN MoMo"
  - "MTN Mobile Money"
  - "MTN MoMo API"
  - "mobile money collection"
  - "MTN disbursement"
  - "request to pay MTN"
  - "MTN payment API"
  - "MTN transfer money"

---

# MTN Mobile Money API (MTN MoMo)

## Overview

MTN Mobile Money API (MTN MoMo) is a pan-African mobile money platform by MTN Group that enables developers to integrate payment collection, disbursement, and remittance capabilities into digital applications. The API allows businesses to trigger real-time payments within any digital channel (web, mobile app) and disburse payments to consumers or other businesses.

**Official Developer Portal:** https://momodeveloper.mtn.com/

**Production Portal:** https://momoapi.mtn.com/

**Community Support:** https://momodevelopercommunity.mtn.com/

## When to Use

- **Payment Collection:** Collect payments from consumers/businesses via Request to Pay
- **Payroll & Disbursement:** Pay salaries, benefits, or refunds to multiple users
- **Remittance:** Send money across borders to supported MTN countries
- **Account Verification:** Validate customer identities before transactions
- **Balance Checks:** Query account balances for reconciliation
- **Pan-African Expansion:** Reach users in 12+ MTN-supported countries with a single integration

## Supported Countries

- Uganda (mtnuganda)
- Ghana (mtnghana)
- Ivory Coast (mtnivorycoast)
- Zambia (mtnzambia)
- Cameroon (mtncameroon)
- Benin (mtnbenin)
- Congo (mtncongo)
- Swaziland (mtnswaziland)
- Guinea Conakry (mtnguineaconakry)
- South Africa (mtnsouthafrica)
- Liberia (mtnliberia)
- Additional countries in the MTN footprint

## Authentication

### Step 1: Generate API User and API Key (Sandbox Only)

In the sandbox environment, you self-provision credentials via API:

```bash
POST https://sandbox.momodeveloper.mtn.com/v1_0/apiuser
Headers:
  X-Reference-Id: <YOUR-UUID-V4>
  Content-Type: application/json
  Ocp-Apim-Subscription-Key: <PRIMARY-SUBSCRIPTION-KEY>

Body:
{
  "providerCallbackHost": "https://yourdomain.com"
}
```

**Response:** 201 Created (successful)

**Note:** Generate a unique UUID v4 for X-Reference-Id using an online generator or your language's UUID library.

### Step 2: Get API Key

After creating the API User, retrieve the API Key:

```bash
GET https://sandbox.momodeveloper.mtn.com/v1_0/apiuser/<X-REFERENCE-ID>/apikey
Headers:
  Ocp-Apim-Subscription-Key: <PRIMARY-SUBSCRIPTION-KEY>
```

**Response:** Returns your `apiKey` (store securely)

### Step 3: Generate Access Token

Use Basic Authentication to obtain an OAuth 2.0 access token:

```bash
POST https://sandbox.momodeveloper.mtn.com/collection/token/
Headers:
  Authorization: Basic <BASE64-ENCODED-CREDENTIALS>
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>
  Content-Type: application/json
```

**Encoding Credentials:**
```
BASE64(<API-USER-ID>:<API-KEY>)
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

### Production Environment

In production:

1. Complete KYC (Know Your Customer) verification via the Partner Portal
2. Receive API credentials from MTN (not self-provisioned)
3. Use the production portal: https://momoapi.mtn.com/
4. Endpoints use `https://api.mtn.com` instead of sandbox URLs

## Base URLs

| Environment | Base URL |
|-------------|----------|
| Sandbox | `https://sandbox.momodeveloper.mtn.com` |
| Production | `https://api.mtn.com` (provided after KYC approval) |

## API Products

### 1. Collection

Receive payments from customers via Request to Pay.

#### Request to Pay

Initiate a payment request that the customer must authorize with their PIN:

```bash
POST /collection/v1_0/requesttopay
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Reference-Id: <UNIQUE-UUID-V4>
  X-Target-Environment: sandbox
  Content-Type: application/json
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>

Body:
{
  "amount": "100",
  "currency": "EUR",
  "externalId": "12345",
  "payer": {
    "partyIdType": "MSISDN",
    "partyId": "256772123456"
  },
  "payerMessage": "Please pay for your order",
  "payeeNote": "Payment for Order #12345"
}
```

**Required Fields:**
- `amount`: Transaction amount (string)
- `currency`: Currency code (e.g., EUR, GHS, UGX)
- `externalId`: Your internal reference ID
- `payer.partyIdType`: MSISDN (phone) or EMAIL
- `payer.partyId`: Customer phone (format: 256xxxxxxxxx) or email
- `payerMessage`: Message shown to payer
- `payeeNote`: Note stored in payee's transaction history

**Response:** 202 Accepted — empty body. The response carries no JSON. Use the `X-Reference-Id` value you sent in the request header to subsequently query status.

#### Get Request Status

Check the status of a Request to Pay:

```bash
GET /collection/v1_0/requesttopay/<REFERENCE-ID>
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Target-Environment: sandbox
  Content-Type: application/json
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>
```

**Response:**
```json
{
  "amount": "100",
  "currency": "EUR",
  "financialTransactionId": "1633100230",
  "externalId": "12345",
  "payer": {
    "partyIdType": "MSISDN",
    "partyId": "256772123456"
  },
  "status": "SUCCESSFUL",
  "reason": {
    "code": "SUCCESS",
    "message": "Transaction successful"
  }
}
```

**Possible Status Values:**
- `PENDING` - Awaiting customer approval
- `SUCCESSFUL` - Payment completed
- `FAILED` - Payment rejected
- `EXPIRED` - Request timed out

---

### 2. Disbursement

Send money to customers (salaries, refunds, benefits):

```bash
POST /disbursement/v1_0/transfer
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Reference-Id: <UNIQUE-UUID-V4>
  X-Target-Environment: sandbox
  Content-Type: application/json
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>

Body:
{
  "amount": "100",
  "currency": "EUR",
  "externalId": "54321",
  "payee": {
    "partyIdType": "MSISDN",
    "partyId": "256772987654"
  },
  "payerMessage": "Your salary payment",
  "payeeNote": "Salary for January"
}
```

**Response:** 202 Accepted — empty body. Use the `X-Reference-Id` you sent to query status.

#### Get Disbursement Status

```bash
GET /disbursement/v1_0/transfer/<REFERENCE-ID>
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Target-Environment: sandbox
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>
```

---

### 3. Remittance

International money transfers between MTN markets:

```bash
POST /remittance/v1_0/transfer
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Reference-Id: <UNIQUE-UUID-V4>
  X-Target-Environment: sandbox
  Content-Type: application/json
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>

Body:
{
  "amount": "50",
  "currency": "EUR",
  "externalId": "99999",
  "payee": {
    "partyIdType": "MSISDN",
    "partyId": "233555123456"  // Ghana number example
  },
  "payerMessage": "Family support",
  "payeeNote": "Funds from abroad"
}
```

#### Validate Account Holder

Check if a customer is active and eligible:

```bash
GET /remittance/v1_0/accountholder/validate/<PARTY-ID-TYPE>/<PARTY-ID>
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Target-Environment: sandbox
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>
```

**Response:**
```json
{
  "result": true  // true if account holder is active
}
```

#### Get Basic User Info

Retrieve limited customer information (name, age) for remittance/sanctions screening:

```bash
GET /remittance/v1_0/basicuserinfo/<PARTY-ID-TYPE>/<PARTY-ID>
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Target-Environment: sandbox
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>
```

**Response:**
```json
{
  "name": "John Doe",
  "givenName": "John",
  "familyName": "Doe",
  "dateOfBirth": "1985-01-15"
}
```

---

### 4. Account Operations

#### Get Account Balance

Check your MoMo wallet balance (all products):

```bash
GET /collection/v1_0/account/balance
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Target-Environment: sandbox
  Content-Type: application/json
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>
```

**Response:**
```json
{
  "availableBalance": "1000.00",
  "currency": "EUR"
}
```

---

## Callbacks / Webhooks

MTN MoMo sends asynchronous callbacks when transactions complete. The callback URL host must match the `providerCallbackHost` configured when you created the API User.

### Callback Request Format

```bash
POST https://yourdomain.com/momo-callback
Headers:
  Content-Type: application/json

Body:
{
  "externalId": "12345",
  "amount": "100",
  "transactionStatus": "SUCCESSFUL",
  "financialTransactionId": "1633100230",
  "currency": "EUR",
  "payee": {
    "partyIdType": "MSISDN",
    "partyId": "256772123456"
  }
}
```

### Important Callback Details

- **Asynchronous:** Callbacks happen after the transaction completes, not immediately
- **Retry Logic:** MTN retries callbacks if your endpoint returns non-2xx status
- **Host Validation:** The callback Host header must match the provisioned callback host (INVALID_CALLBACK_URL_HOST error if mismatch)
- **Idempotency:** Use `externalId` to deduplicate callbacks; the same callback may be sent multiple times
- **Hostname Only:** Use fully qualified domain names, not IP addresses for callbacks

### Webhook Testing in Sandbox

MTN MoMo provides endpoints to simulate callback payloads. You can test what the callback data will look like when users input their MoMo PIN in the sandbox environment.

---

## Common Integration Patterns

### Pattern 1: Request to Pay + Polling

1. Call Request to Pay endpoint
2. Receive `financialTransactionId`
3. Poll the status endpoint every 5-10 seconds
4. Continue until status is SUCCESSFUL or FAILED
5. Max polling time: 2-3 minutes

```python
import time
import requests

response = requests.post(
    f"{BASE_URL}/collection/v1_0/requesttopay",
    headers=headers,
    json=payload
)
financial_tx_id = response.json()["financialTransactionId"]

# Poll for completion
for _ in range(20):
    time.sleep(5)
    status_response = requests.get(
        f"{BASE_URL}/collection/v1_0/requesttopay/{reference_id}",
        headers=headers
    )
    status = status_response.json()["status"]
    if status in ["SUCCESSFUL", "FAILED"]:
        break
```

### Pattern 2: Disbursement (Fire and Forget)

1. Call Transfer endpoint
2. Store `financialTransactionId`
3. Wait for webhook callback
4. Update transaction status in your database

```python
response = requests.post(
    f"{BASE_URL}/disbursement/v1_0/transfer",
    headers=headers,
    json=payload
)
financial_tx_id = response.json()["financialTransactionId"]

# Store for reconciliation
db.store_transaction({
    "external_id": payload["externalId"],
    "financial_tx_id": financial_tx_id,
    "status": "PENDING",
    "type": "DISBURSEMENT"
})
```

### Pattern 3: Batch Disbursements

Process multiple payments in a loop:

```python
users = [
    {"phone": "256772123456", "amount": "100"},
    {"phone": "256772654321", "amount": "150"},
    {"phone": "256772999999", "amount": "200"}
]

for user in users:
    payload = {
        "amount": user["amount"],
        "currency": "UGX",
        "externalId": f"batch_pay_{user['phone']}_{timestamp}",
        "payee": {
            "partyIdType": "MSISDN",
            "partyId": user["phone"]
        },
        "payerMessage": "Monthly stipend",
        "payeeNote": "Your monthly payment"
    }
    requests.post(f"{BASE_URL}/disbursement/v1_0/transfer", headers=headers, json=payload)
    time.sleep(1)  # Rate limiting
```

---

## Error Handling

### Common HTTP Status Codes

| Code | Meaning | Example |
|------|---------|---------|
| 201 | Created (API User created successfully) | Sandbox user provisioning |
| 202 | Accepted (Transaction initiated) | Request to Pay, Transfer accepted |
| 400 | Bad Request | Invalid JSON, missing required fields |
| 401 | Unauthorized | Invalid subscription key, malformed auth header |
| 404 | Not Found | Reference ID doesn't exist |
| 409 | Conflict | Duplicate Reference ID (X-Reference-Id used twice) |
| 500 | Internal Server Error | Generic server error, often insufficient funds |

### Error Response Format

```json
{
  "errorCode": "INVALID_CALLBACK_URL_HOST",
  "errorDescription": "Callback host mismatch"
}
```

### Common Error Codes

| Error Code | Description | Solution |
|------------|-------------|----------|
| `RESOURCE_ALREADY_EXIST` | Duplicate Reference ID | Use a unique UUID v4 for each request |
| `INVALID_CALLBACK_URL_HOST` | Callback URL host doesn't match provisioned host | Ensure `providerCallbackHost` matches callback URL domain |
| `ACCESS_DENIED` | Invalid subscription key or authorization | Verify subscription key and token are correct |
| `INVALID_CURRENCY` | Unsupported currency for the country | Use country-specific currency (EUR, GHS, UGX, XAF, etc.) |
| `INSUFFICIENT_BALANCE` | Your MoMo wallet lacks funds | Top-up your account or use sandbox test funds |
| `INVALID_PARTY_ID` | Phone number format error | Use E.164 format: 256xxxxxxxxx |
| `INTERNAL_PROCESSING_ERROR` | Generic server error | Usually insufficient funds in subscriber's wallet |
| `INVALID_REFERENCE_ID` | Reference ID format invalid | Use UUID v4 format |

### Error Response Enrichment

Newer versions of the API provide more detailed error reasons, particularly for INTERNAL_PROCESSING_ERROR. Monitor the MoMo Developer Community for updates on error message improvements.

---

## Important Notes & Gotchas

### 1. **Unique X-Reference-Id for Every Request**
Each POST request (Request to Pay, Transfer) MUST have a unique UUID v4. Reusing an X-Reference-Id returns HTTP 409 CONFLICT. Generate fresh UUIDs for each transaction.

```bash
X-Reference-Id: 550e8400-e29b-41d4-a716-446655440000  # Unique every time
```

### 2. **X-Target-Environment Header Required**
Include this header in all requests:
```bash
X-Target-Environment: sandbox  # or "production"
```
Omitting it causes 500 errors.

### 3. **Callback Host Must Exactly Match**
The callback host (domain name) must match the `providerCallbackHost` you provided when creating the API User. Using a different subdomain or IP address causes `INVALID_CALLBACK_URL_HOST` errors. Always use fully qualified domain names (FQDN), not IP addresses.

### 4. **API Tokens Expire**
Access tokens expire in 3600 seconds (1 hour). Implement token refresh logic:
```python
if token_expires_at < time.time() + 300:  # Refresh if < 5 min left
    token = get_new_token()
```

### 5. **Sandbox Credentials Are Self-Provisioned**
In sandbox, you create your own API User and API Key. In production, these are issued by MTN after KYC approval. Do not hardcode sandbox credentials into production code.

### 6. **Idempotency via External ID**
Use `externalId` as your idempotency key. If a request fails and you retry, use the same `externalId` to prevent duplicate transactions. Combine with timestamp to ensure uniqueness:
```json
{
  "externalId": "12345_20250224_1633100230"
}
```

### 7. **Reference ID vs. External ID**
- **X-Reference-Id (Header):** Identifies the API request itself; must be unique per API call
- **externalId (Body):** Your internal transaction reference; used for idempotency and reconciliation

### 8. **Polling Timeout Strategy**
When polling for transaction status, don't poll indefinitely. Implement a timeout (2-3 minutes) and assume the transaction failed if status is still PENDING.

```python
polling_count = 0
max_polls = 36  # 3 minutes at 5-second intervals
while polling_count < max_polls:
    status = check_status()
    if status != "PENDING":
        break
    polling_count += 1
    time.sleep(5)
else:
    # Timeout reached
    handle_timeout()
```

### 9. **Phone Number Format (E.164)**
Phone numbers must use E.164 format with country code, no spaces or special characters:
- Uganda: `256772123456` (not +256 772 123 456)
- Ghana: `233555123456`
- Ivory Coast: `225071234567`

### 10. **Currency Varies by Country**
Each country uses specific currencies:
- Uganda: UGX
- Ghana: GHS
- Ivory Coast: XOF
- Congo: XAF
- South Africa: ZAR

Always verify the correct currency for the target country/phone number.

### 11. **Insufficient Balance Returns Generic Error**
When your MoMo wallet (the API User's account) lacks funds, you receive `INTERNAL_PROCESSING_ERROR`. This is the most common error in production. Maintain wallet balance monitoring:

```python
balance = get_account_balance()
if balance < transaction_amount:
    raise InsufficientFundsError()
```

### 12. **Subscription Key Placement Varies by Endpoint**
All endpoints require the `Ocp-Apim-Subscription-Key` header. Do not confuse this with the API Key (used in Basic Auth). The subscription key is provided when you subscribe to a product in the developer portal.

```bash
Headers:
  Authorization: Basic <BASE64(API-USER:API-KEY)>          # For token endpoint
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>           # For all endpoints
  X-Reference-Id: <UUID>                                  # For POST requests
```

### 13. **Date/Time Handling**
No explicit timestamp is required in request bodies, but always use server time (UTC) for logging and reconciliation. Store transaction timestamps for audit trails.

### 14. **Rate Limiting**
No official rate limit is documented, but implement backoff logic for production:
- Add 1-2 second delays between batch requests
- Retry failed requests with exponential backoff (1s, 2s, 4s, 8s)

### 15. **Webhook Deduplication**
MTN may retry webhook callbacks. Deduplicate using `externalId`:
```python
if Transaction.objects.filter(external_id=callback["externalId"]).exists():
    # Duplicate callback, update status if needed
    pass
else:
    # New transaction, create it
    Transaction.create(external_id=callback["externalId"], ...)
```

---

## SDK & Libraries

Official and community SDKs for MTN MoMo:

- **JavaScript/Node.js:** [mtn-momo](https://www.npmjs.com/package/mtn-momo), [mtn-pay](https://sopherapps.github.io/mtn-pay-js/)
- **PHP:** [mtn-momo-api-php](https://github.com/patricpoba/mtn-momo-api-php), [laravel-mtn-momo](https://github.com/bmatvou/laravel-mtn-momo)
- **Python:** [mtnmomoapi](https://pypi.org/project/mtnmomoapi/), [mtnmomo](https://pypi.org/project/mtnmomo/)
- **Elixir:** [ExMtnMomo](https://hexdocs.pm/ex_mtn_momo/)
- **Ruby:** [momoapi-ruby](https://github.com/sparkplug/momoapi-ruby)
- **Android:** Official Android SDK available on Developer Portal

---

## Testing in Sandbox

### Test Credentials (Sandbox)

Use these test phone numbers in sandbox to simulate different scenarios:

- **Successful Payment:** `256772123456` (will auto-approve)
- **Failed Payment:** `256772654321` (will auto-reject)
- **Pending Transaction:** `256772999999` (will timeout)

(Actual test numbers may vary; check the sandbox documentation for current test credentials.)

### Simulating Callbacks

Use the sandbox callback simulation endpoint to test your webhook without waiting for real transactions:

```bash
POST https://sandbox.momodeveloper.mtn.com/collection/v1_0/requesttopay/<REFERENCE-ID>/notify
Headers:
  Authorization: Bearer <ACCESS-TOKEN>
  X-Target-Environment: sandbox
  Ocp-Apim-Subscription-Key: <SUBSCRIPTION-KEY>
```

---

## Useful Links

- **Official Developer Portal:** https://momodeveloper.mtn.com/
- **Production Portal:** https://momoapi.mtn.com/
- **API Documentation:** https://momodeveloper.mtn.com/api-documentation
- **Sandbox Documentation:** https://www.postman.com/momoapis/momo-open-apis/documentation/0qcufs3/momo-open-apis-sandbox
- **Developer Community & Support:** https://momodevelopercommunity.mtn.com/
- **Common Error Codes:** https://momodeveloper.mtn.com/api-documentation/common-error
- **Getting Started Guide:** https://momodeveloper.mtn.com/api-documentation/getting-started
- **GitHub Examples:** https://github.com/ogwok/mtn-momo

---

## Summary

MTN MoMo is a production-ready mobile money API for pan-African payment collection, disbursement, and remittance. Key integration steps:

1. **Sandbox:** Self-provision API User + API Key
2. **Production:** Complete KYC, receive credentials from MTN
3. **Authentication:** Generate access token using Basic Auth
4. **API Calls:** Use Bearer token + X-Reference-Id + X-Target-Environment headers
5. **Callbacks:** Implement webhooks for async transaction notifications
6. **Error Handling:** Handle 409 conflicts, 500 insufficient balance, callback retries

Always verify phone number formats, currency codes, and callback host configuration before going to production.
