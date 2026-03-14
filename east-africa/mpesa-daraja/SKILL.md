---
name: mpesa-daraja
description: "Integrate with Safaricom's M-Pesa Daraja API for Kenyan mobile money payments. Use this skill whenever the user wants to accept M-Pesa payments, trigger STK push prompts, handle C2B or B2C transactions, send money to M-Pesa users, check account balances, or work with M-Pesa/Daraja in any way. Also trigger for 'M-Pesa', 'Daraja', 'Safaricom API', 'STK push', 'Lipa Na M-Pesa', 'KES mobile money', 'send money to M-Pesa', 'Kenyan mobile payments', or any mention of collecting or disbursing payments through M-Pesa in Kenya."
---

# M-Pesa Daraja Integration Skill

M-Pesa is East Africa's dominant mobile money platform with 50M+ active users in Kenya alone. The Daraja API is Safaricom's official gateway to M-Pesa — it lets you collect payments (STK Push, C2B), send money (B2C), and query transactions programmatically.

## When to use this skill

You're building something that needs to move money in Kenya via M-Pesa — collecting payments for goods/services, disbursing funds to users, building a marketplace with M-Pesa checkout, or integrating Lipa Na M-Pesa into an app. M-Pesa is how most Kenyans transact, so if your user base is in Kenya, you probably need this.

## Authentication

Daraja uses OAuth 2.0. You get an access token from a consumer key + consumer secret pair, then use the token for all subsequent requests.

### Get Access Token

```
GET /oauth/v1/generate?grant_type=client_credentials
Authorization: Basic {base64(consumer_key:consumer_secret)}
```

**Response:**
```json
{
  "access_token": "xxxxx",
  "expires_in": "3599"
}
```

Token expires in 1 hour. Cache it and refresh before expiry.

**Environment URLs:**
- Sandbox: `https://sandbox.safaricom.co.ke`
- Production: `https://api.safaricom.co.ke`

Store credentials in env vars: `MPESA_CONSUMER_KEY`, `MPESA_CONSUMER_SECRET`, `MPESA_PASSKEY`, `MPESA_SHORTCODE`.

## Core API Reference

### STK Push (Lipa Na M-Pesa Online)

The most common integration — triggers a payment prompt on the customer's phone. They enter their M-Pesa PIN to complete the payment.

```
POST /mpesa/stkpush/v1/processrequest
Authorization: Bearer {access_token}
```

```json
{
  "BusinessShortCode": "174379",
  "Password": "{base64(shortcode + passkey + timestamp)}",
  "Timestamp": "20250115143000",
  "TransactionType": "CustomerPayBillOnline",
  "Amount": 1000,
  "PartyA": "254712345678",
  "PartyB": "174379",
  "PhoneNumber": "254712345678",
  "CallBackURL": "https://yoursite.com/api/mpesa/callback",
  "AccountReference": "OrderRef123",
  "TransactionDesc": "Payment for Order #123"
}
```

**Password generation:**
```javascript
const timestamp = new Date().toISOString().replace(/[-T:.Z]/g, '').slice(0, 14);
const password = Buffer.from(`${shortcode}${passkey}${timestamp}`).toString('base64');
```

**Important:**
- `Amount` is in whole KES (not cents). KES 1000 = `1000`.
- Phone numbers must be in format `254XXXXXXXXX` (no + prefix, no leading 0).
- `TransactionType`: Use `CustomerPayBillOnline` for paybill, `CustomerBuyGoodsOnline` for till numbers.

**Response:**
```json
{
  "MerchantRequestID": "xxxxx",
  "CheckoutRequestID": "ws_CO_xxxxx",
  "ResponseCode": "0",
  "ResponseDescription": "Success. Request accepted for processing",
  "CustomerMessage": "Success. Request accepted for processing"
}
```

`ResponseCode: "0"` means the request was accepted — not that payment succeeded. You need to wait for the callback or query the status.

### Query STK Push Status

```
POST /mpesa/stkpushquery/v1/query
```

```json
{
  "BusinessShortCode": "174379",
  "Password": "{base64(shortcode + passkey + timestamp)}",
  "Timestamp": "20250115143000",
  "CheckoutRequestID": "ws_CO_xxxxx"
}
```

**Response (success):**
```json
{
  "ResponseCode": "0",
  "ResultCode": "0",
  "ResultDesc": "The service request is processed successfully.",
  "MerchantRequestID": "xxxxx",
  "CheckoutRequestID": "ws_CO_xxxxx"
}
```

`ResultCode: "0"` = payment successful. `ResultCode: "1032"` = user cancelled. `ResultCode: "1037"` = timeout (user didn't respond).

### C2B (Customer to Business) Registration

Register URLs to receive payment notifications when customers pay your paybill/till. This is critical for receiving callbacks when customers send money to your business account.

```
POST /mpesa/c2b/v1/registerurl
```

```json
{
  "ShortCode": "600000",
  "ResponseType": "Completed",
  "ConfirmationURL": "https://yoursite.com/api/mpesa/confirm",
  "ValidationURL": "https://yoursite.com/api/mpesa/validate"
}
```

**Important:**
- Both URLs must be **HTTPS** (not HTTP — M-Pesa will reject).
- Do not use words like "mpesa", "m-pesa", "safaricom" in your URLs.
- Do not use localhost or ngrok URLs in production (they will be blocked).
- `ValidationURL` is called before the transaction completes — return `{"ResultCode": 0}` to accept or `{"ResultCode": 1}` to reject.
- `ConfirmationURL` receives the final payment confirmation after the transaction is complete.

### C2B Validation Callback (Incoming Request)

M-Pesa POSTs to your validation URL before completing the transaction:

```json
{
  "TransactionType": "Pay Bill Online",
  "TransID": "LIB221011CD6D",
  "TransTime": "20221011095810",
  "TransAmount": 1000,
  "BusinessShortCode": "600000",
  "BillRefNumber": "OrderRef123",
  "InvoiceNumber": "",
  "OrgAccountBalance": "100000.00",
  "ThirdPartyTransID": "",
  "MSISDN": "254712345678",
  "FirstName": "John",
  "MiddleName": "",
  "LastName": "Doe"
}
```

**Your response:**
```json
{
  "ResultCode": 0,
  "ResultDesc": "Accepted"
}
```

Return `ResultCode: 0` to accept, `ResultCode: 1` to reject the payment.

### C2B Confirmation Callback (Incoming Request)

M-Pesa POSTs to your confirmation URL after the transaction is complete:

```json
{
  "TransactionType": "Pay Bill Online",
  "TransID": "LIB221011CD6D",
  "TransTime": "20221011095810",
  "TransAmount": 1000,
  "BusinessShortCode": "600000",
  "BillRefNumber": "OrderRef123",
  "InvoiceNumber": "",
  "OrgAccountBalance": "100000.00",
  "ThirdPartyTransID": "",
  "MSISDN": "254712345678",
  "FirstName": "John",
  "MiddleName": "",
  "LastName": "Doe"
}
```

**Always respond with HTTP 200** to M-Pesa callbacks, even if you encounter processing errors — otherwise M-Pesa will keep retrying.

### B2C (Business to Customer) Payment

Send money from your M-Pesa business account to a customer's phone:

```
POST /mpesa/b2c/v1/paymentrequest
```

```json
{
  "InitiatorName": "testapi",
  "SecurityCredential": "{encrypted_password}",
  "CommandID": "BusinessPayment",
  "Amount": 500,
  "PartyA": "600000",
  "PartyB": "254712345678",
  "Remarks": "Salary payment",
  "QueueTimeOutURL": "https://yoursite.com/api/mpesa/timeout",
  "ResultURL": "https://yoursite.com/api/mpesa/b2c/result",
  "Occasion": "January salary"
}
```

**CommandID options:**
- `BusinessPayment` — normal B2C payment
- `SalaryPayment` — salary disbursement
- `PromotionPayment` — promotional payment

**SecurityCredential:** Your initiator password encrypted with Safaricom's RSA public key certificate. Download the cert from the Daraja portal and encrypt:

```javascript
const fs = require('fs');
const crypto = require('crypto');
const cert = fs.readFileSync('ProductionCertificate.cer');
const encrypted = crypto.publicEncrypt(
  { key: cert, padding: crypto.constants.RSA_PKCS1_PADDING },
  Buffer.from(password)
);
const securityCredential = encrypted.toString('base64');
```

**Response:**
```json
{
  "ConversationID": "AG_20250115_XXXXX",
  "OriginatorConversationID": "XXXXX",
  "ResponseCode": "0",
  "ResponseDescription": "Accept the service request successfully."
}
```

### B2C Result Callback (Incoming Request)

M-Pesa POSTs to your ResultURL after processing the B2C payment:

```json
{
  "Result": {
    "ResultType": 0,
    "ResultCode": 0,
    "ResultDesc": "The service request is processed successfully.",
    "OriginatorConversationID": "XXXXX",
    "ConversationID": "AG_20250115_XXXXX",
    "TransactionID": "OEI2AK4Q16",
    "ReferenceData": {
      "ReferenceItem": {
        "Key": "QueueTimeoutURL",
        "Value": "https://yoursite.com/api/mpesa/timeout"
      }
    },
    "ResultParameters": {
      "ResultParameter": [
        {
          "Key": "TransactionAmount",
          "Value": 500
        },
        {
          "Key": "TransactionReceipt",
          "Value": "OEI2AK4Q16"
        },
        {
          "Key": "ReceiverPartyPublicName",
          "Value": "254712345678"
        },
        {
          "Key": "TransactionCompletedDateTime",
          "Value": "20250115143025"
        },
        {
          "Key": "OriginatorAccountBalance",
          "Value": "100000.00"
        }
      ]
    }
  }
}
```

Check `Result.ResultCode: 0` for success.

### Transaction Status Query

```
POST /mpesa/transactionstatus/v1/query
```

```json
{
  "Initiator": "testapi",
  "SecurityCredential": "{encrypted}",
  "CommandID": "TransactionStatusQuery",
  "TransactionID": "OEI2AK4Q16",
  "PartyA": "600000",
  "IdentifierType": "4",
  "ResultURL": "https://yoursite.com/api/mpesa/status/result",
  "QueueTimeOutURL": "https://yoursite.com/api/mpesa/timeout",
  "Remarks": "Check status",
  "Occasion": "Status check"
}
```

**IdentifierType:**
- `1` = MSISDN (phone number)
- `2` = Organization short code
- `4` = Organization short code (for business accounts)

### Account Balance Query

```
POST /mpesa/accountbalance/v1/query
```

```json
{
  "Initiator": "testapi",
  "SecurityCredential": "{encrypted}",
  "CommandID": "AccountBalance",
  "PartyA": "600000",
  "IdentifierType": "4",
  "Remarks": "Balance check",
  "QueueTimeOutURL": "https://yoursite.com/api/mpesa/timeout",
  "ResultURL": "https://yoursite.com/api/mpesa/balance/result"
}
```

### Reversal

Reverse a completed M-Pesa transaction:

```
POST /mpesa/reversal/v1/request
```

```json
{
  "Initiator": "testapi",
  "SecurityCredential": "{encrypted}",
  "CommandID": "TransactionReversal",
  "TransactionID": "OEI2AK4Q16",
  "Amount": 500,
  "ReceiverParty": "600000",
  "ReceiverIdentifierType": "11",
  "ResultURL": "https://yoursite.com/api/mpesa/reversal/result",
  "QueueTimeOutURL": "https://yoursite.com/api/mpesa/timeout",
  "Remarks": "Wrong amount",
  "Occasion": "Reversal"
}
```

## STK Push Callback Handling

When an STK Push transaction completes, M-Pesa POSTs to your CallbackURL:

```json
{
  "Body": {
    "stkCallback": {
      "MerchantRequestID": "xxxxx",
      "CheckoutRequestID": "ws_CO_xxxxx",
      "ResultCode": 0,
      "ResultDesc": "The service request is processed successfully.",
      "CallbackMetadata": {
        "Item": [
          { "Name": "Amount", "Value": 1000 },
          { "Name": "MpesaReceiptNumber", "Value": "OEI2AK4Q16" },
          { "Name": "TransactionDate", "Value": 20250115143025 },
          { "Name": "PhoneNumber", "Value": 254712345678 }
        ]
      }
    }
  }
}
```

**Key ResultCode values:**
- `0` — Success, money received
- `1` — Not enough balance on user's M-Pesa account
- `1032` — User cancelled the transaction
- `1037` — Transaction timeout (user didn't respond within ~60 seconds)
- `2001` — Wrong PIN entered by user

Always respond with HTTP 200 to callbacks, even if you encounter errors — otherwise M-Pesa will keep retrying (up to ~3 times over 30 minutes).

## Important Notes / Gotchas

### Token Expiry
- Access tokens expire in 3600 seconds (1 hour). Cache tokens and implement automatic refresh.
- In production, implement token caching with TTL to avoid hitting the auth endpoint on every request.

### Callback URLs Must Be HTTPS
- M-Pesa will only POST to HTTPS endpoints. HTTP callbacks are rejected.
- Endpoints must have valid SSL/TLS certificates (self-signed won't work in production).

### Shortcode Types
- **Paybill** (e.g., 600000): Businesses receive money from customers. Use `CustomerPayBillOnline` for STK Push.
- **Till** (e.g., 174379): Retail businesses. Use `CustomerBuyGoodsOnline` for STK Push.
- Know your shortcode type before integration.

### Sandbox vs Production
- **Sandbox URL:** `https://sandbox.safaricom.co.ke` (uses test credentials)
- **Production URL:** `https://api.safaricom.co.ke` (uses live credentials)
- Get separate credentials for sandbox and production from the Daraja portal.
- Sandbox token and production token cannot be mixed.

### Security Credentials (B2C)
- Security credentials are encrypted with Safaricom's RSA public key certificate.
- Download the correct certificate from Daraja portal (sandbox cert ≠ production cert).
- Use RSA PKCS#1 padding (not OAEP).
- Re-encrypt credentials; never reuse an encrypted value.

### Phone Number Format
- Always use format `254XXXXXXXXX` (no `+`, no leading `0`).
- `254` is Kenya's country code. All numbers must start with it.
- Invalid format is a common source of errors.

### Callback Retry Logic
- M-Pesa retries callbacks ~3 times over ~30 minutes if you don't respond with HTTP 200.
- Always return HTTP 200 immediately, even if processing fails.
- Process callback data asynchronously (queue, background job, etc.).

### Validation URL vs Confirmation URL (C2B)
- **ValidationURL**: Called before transaction (can reject it here).
- **ConfirmationURL**: Called after transaction completes (for logging/confirmation).
- Both are called by M-Pesa — don't use just one.

### Amount Precision
- Amounts are **whole KES integers** (no decimal places).
- Minimum amount for STK Push is typically KES 1.
- Maximum varies by account type (usually KES 70,000 for consumer accounts).

### ResponseCode vs ResultCode
- **ResponseCode**: Returned in sync response to your API request. `"0"` = accepted for processing.
- **ResultCode**: Returned in async callback after M-Pesa processes the request. `0` = success.
- Don't confuse them — ResponseCode doesn't indicate transaction success.

### Sandbox Callback Limitations
- Sandbox callback delivery is unreliable (often 40% success rate).
- Use status query APIs instead of relying on callbacks in sandbox testing.
- Callbacks work reliably in production.

### No ngrok or Request Inspection Tools
- Don't use ngrok, requestbin, or similar public URL testers in production.
- Safaricom blocks requests from these services for security.
- Use a proper production server with a public domain.

## Common Integration Patterns

### E-commerce checkout with STK Push
1. Customer clicks "Pay with M-Pesa"
2. `POST /mpesa/stkpush/v1/processrequest` with their phone number
3. Customer sees M-Pesa prompt → enters PIN
4. Your callback URL receives payment confirmation
5. Verify `ResultCode === 0` → fulfill order

### Marketplace disbursements
1. Collect payments via STK Push (C2B)
2. Calculate platform commission
3. `POST /mpesa/b2c/v1/paymentrequest` to pay each seller
4. Track via result callbacks
5. Handle `ResultCode: 500.001.1001` (insufficient funds) gracefully

### Subscription service
1. First payment via STK Push
2. Store customer phone number
3. On renewal: trigger another STK Push
4. Handle `ResultCode: 1032` (cancel) → update subscription status
5. Handle `ResultCode: 1037` (timeout) → retry or notify user

### Receive payments with C2B
1. Register validation + confirmation URLs: `POST /mpesa/c2b/v1/registerurl`
2. Customer sends money to your paybill/till
3. M-Pesa calls your validation URL (accept/reject here if needed)
4. M-Pesa calls your confirmation URL (log payment, update ledger)
5. Always respond HTTP 200

## Error Handling

M-Pesa error responses vary by endpoint but follow this pattern:

```json
{
  "requestId": "xxxxx",
  "errorCode": "400.002.02",
  "errorMessage": "Bad Request - Invalid Amount"
}
```

Common ResultCode error values in callbacks:
- **0** — Success
- **1** — Insufficient balance on user's M-Pesa account
- **2001** — Wrong PIN entered by user
- **1032** — Request cancelled by user
- **1037** — Request timeout (user didn't respond)
- **500.001.1001** — Insufficient funds in business account (B2C)
- **201** — Unable to lock subscriber (M-Pesa user account)
- **202** — Subscriber not found in the system

Sync response error codes:
- **400.002.02** — Bad Request - Invalid Amount
- **400.002.03** — Bad Request - Invalid Payment Type
- **401.002.01** — Unauthorized - Invalid API Request

When you receive an error:
- Check phone number format (`254...` required)
- Verify amount is positive integer (whole KES)
- Confirm shortcode and transaction type match
- Validate callback URL is HTTPS and accessible
- Check credentials and tokens are current

## Sandbox Testing

Daraja sandbox credentials are available at https://developer.safaricom.co.ke/test_credentials

Test phone numbers: Use any `254XXXXXXXXX` number in sandbox. The sandbox simulates successful payments by default for testing.

**Sandbox limitations:**
- Callback delivery is unreliable (test with status queries instead)
- Use production environment to verify callbacks work
- Credentials expire periodically — refresh them on the portal

## Useful Links

- Daraja Portal: https://developer.safaricom.co.ke
- API Docs: https://developer.safaricom.co.ke/Documentation
- Sandbox Credentials: https://developer.safaricom.co.ke/test_credentials
- GitHub M-Pesa Libraries: https://github.com/safaricom
