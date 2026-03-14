---
name: africastalking
description: "Integrate with Africa's Talking API for SMS, USSD, Voice, Airtime, and Payments. Use this skill whenever the user wants to send SMS messages across Africa, build USSD applications, integrate voice calls, send airtime, process payments, or work with Africa's Talking APIs. Trigger when the user mentions 'Africa's Talking', 'SMS gateway', 'USSD menu', 'voice API', 'send airtime', 'mobile payments', or needs to communicate/transact with customers in Africa."
---

# Africa's Talking Integration Skill

Africa's Talking is the leading communication and payment API platform for Africa, enabling developers to send SMS, USSD, voice calls, airtime transfers, and process payments across 150+ African countries. It's the backbone for critical communications in fintech, healthcare, logistics, e-commerce, and enterprise solutions.

## When to use this skill

Use this skill when you're building solutions that need to reach customers across Africa through any of these channels:

- **SMS notifications**: Payment confirmations, OTP delivery, transaction alerts, appointment reminders, delivery updates
- **USSD menus**: Offline-first experiences for unbanked users, interactive services without data requirements
- **Voice IVR**: Customer support systems, automated notification delivery, polling and surveys
- **Airtime transfers**: Mobile top-up services, rewards programs, vendor payouts
- **Payments**: Mobile money checkout, B2C transfers (business to customer), B2B transfers (business to business)

Africa's Talking provides a unified API surface that handles carrier integration, routing, compliance, and local payment rail connectivity automatically.

## Authentication

All Africa's Talking API requests require two authentication components:

**API Key in Header:**
```
Authorization: Bearer {api_key}
```

**Username in Request Body:**
```
username=your_username
```

### Getting Credentials

1. Create an account at https://account.africastalking.com
2. Generate your API Key from the dashboard
3. Use your application username (typically your organization name or app identifier)
4. Store API Key in environment variable: `AFRICASTALKING_API_KEY`

**Important:** Never hardcode credentials. Use environment variables or secure credential management systems.

### Environments

- **Sandbox**: https://api.sandbox.africastalking.com (free testing environment)
- **Live/Production**: https://api.africastalking.com (billable operations)

---

## Core API Reference

### SMS - Send Messages

Send text messages to individual or multiple recipients. SMS is the most reliable channel for reaching users across Africa, with 160-character messages being the standard.

**Endpoint:**
```
POST /version1/messaging
```

**Headers:**
```
Authorization: Bearer {api_key}
Content-Type: application/x-www-form-urlencoded
```

**Request Body (Single Recipient):**
```
username=your_username
&to=%2B254712345678
&message=Hello+customer%2C+your+payment+was+successful
```

**Request Body (Multiple Recipients - Bulk SMS):**
```
username=your_username
&to=%2B254712345678%2C%2B256701234567%2C%2B255654321098
&message=Bulk+notification+message
```

**Response:**
```json
{
  "SMSMessageData": {
    "Message": "Sent to 1/1 recipients",
    "Recipients": [
      {
        "statusCode": 101,
        "number": "+254712345678",
        "status": "Success",
        "cost": "KES 0.80",
        "messageId": "ATXid_8b0f6d4c4c6c7d8e9f0a1b2c3d4e5f6g"
      }
    ]
  }
}
```

**Status Codes:**
- `101` - Success (message sent)
- `400` - Bad request (missing or invalid parameters)
- `401` - User does not exist
- `402` - User is suspended
- `403` - Insufficient balance
- `500` - Server error

**Important Notes:**
- Phone numbers must include country code (e.g., +254 for Kenya, +256 for Uganda)
- SMS limited to 160 characters; longer messages are split into multiple SMS
- Store `messageId` for delivery tracking and reconciliation
- Bulk SMS pricing is identical to single SMS

---

### USSD - Interactive Menu Sessions

Build interactive text-based menus for offline users. USSD (Unstructured Supplementary Service Data) works on all phones, even without data or internet.

**Key Concept: Callback-Based Flow**

Unlike polling, Africa's Talking USSD operates on a **callback model**:
1. User dials a USSD code (e.g., *123#)
2. Africa's Talking receives the request and sends an HTTP POST to YOUR webhook endpoint
3. Your server processes the request and responds with menu text
4. Africa's Talking delivers the response to the user's phone
5. User's response is posted back to your webhook

Your webhook must return a response starting with either:
- `CON` - Continue the session (show another menu)
- `END` - End the session (final response)

**Registering Your USSD Endpoint:**

Set your webhook URL in the Africa's Talking dashboard under USSD Settings. Africa's Talking will POST to this URL when users interact with your USSD service.

**Webhook Request (Incoming from Africa's Talking):**
```
POST https://yourserver.com/ussd-webhook
Content-Type: application/x-www-form-urlencoded

sessionId=SESSION_12345
&serviceCode=%2A123%23
&phoneNumber=%2B254712345678
&text=1
```

**Webhook Request Parameters:**
- `sessionId` - Unique identifier for this USSD session
- `serviceCode` - The USSD code user dialed (URL-encoded)
- `phoneNumber` - User's phone number with country code
- `text` - User's input (empty on first request, contains menu selection on subsequent requests)

**Your Webhook Response (Plain Text):**
```
CON Welcome to our service
1. Check Balance
2. Transfer Money
3. Exit
```

Or to end the session:
```
END Your balance is KES 500.00
Thank you for using our service.
```

**USSD Session Flow Example:**
```
1. User dials *123#
2. Your webhook receives: sessionId=ABC, text="" (empty)
3. You return: CON Menu with options
4. User presses "1"
5. Your webhook receives: sessionId=ABC, text="1"
6. You process selection and return: CON Next menu OR END Final response
```

**Important Notes:**
- USSD responses limited to 182 characters
- Session persists through multiple back-and-forths using `sessionId`
- Always include phone number validation
- Response formatting is critical - responses not starting with CON/END are rejected
- No API key required for webhook - Africa's Talking knows it's your endpoint

---

### Voice - Initiate Phone Calls

Make outbound voice calls with IVR (Interactive Voice Response) capabilities using XML-based responses.

Voice uses a **separate subdomain** from the main API:
- **Live:** `https://voice.africastalking.com/call`
- **Sandbox:** `https://voice.sandbox.africastalking.com/call`

**Endpoint:**
```
POST https://voice.africastalking.com/call
```

**Headers:**
```
Authorization: Bearer {api_key}
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
username=your_username
&phoneNumber=%2B254712345678
&callerId=%2B254712345679
```

**Request Parameters:**
- `phoneNumber` - Customer's phone number (URL-encoded, with country code)
- `callerId` - Number to display on caller ID (your business number or shortcode)

**Response:**
```json
{
  "data": {
    "phoneNumber": "+254712345678",
    "status": "Queued",
    "callSessionId": "CALL_SESSION_123456"
  }
}
```

**IVR Callback Flow:**

When the call connects, Africa's Talking calls YOUR callback URL with call details and expects an XML response describing the IVR flow.

**Your IVR Endpoint - Callback URL Request (from Africa's Talking):**
```
POST https://yourserver.com/voice-callback
Content-Type: application/x-www-form-urlencoded

callSessionId=CALL_SESSION_123456
&phoneNumber=%2B254712345678
&callerId=%2B254712345679
&direction=Inbound
&dialStartTime=2025-02-20T10:30:00Z
```

**Your IVR Endpoint - XML Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Say>Welcome to our service. Press 1 for support, press 2 for billing.</Say>
  <GetDigits timeout="10" finishOnKey="#" callbackUrl="https://yourserver.com/handle-digits">
    <Say>Enter your choice followed by the hash key.</Say>
  </GetDigits>
</Response>
```

**XML Response Elements:**
- `<Say>` - Text for the system to read to the caller
- `<GetDigits>` - Prompt for user input (DTMF tones)
  - `timeout` - Seconds to wait for input
  - `finishOnKey` - Character that ends input (usually #)
  - `callbackUrl` - Your endpoint to receive the digits
- `<Dial>` - Transfer the call to another number
- `<Record>` - Record the caller's voice

**Digit Callback Request (User's DTMF Input):**
```
POST https://yourserver.com/handle-digits
Content-Type: application/x-www-form-urlencoded

callSessionId=CALL_SESSION_123456
&phoneNumber=%2B254712345678
&dtmfDigits=1
```

**Response with Next Action:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Say>You selected support. Transferring to our team.</Say>
  <Dial phoneNumber="+254711111111" callerId="+254712345679"/>
</Response>
```

**Important Notes:**
- Calls are queued and may not connect immediately
- IVR XML callbacks are synchronous - respond within 30 seconds
- Always validate `callSessionId` for security
- Use reasonable timeouts and finish keys for better UX
- Recording requires explicit consent in some jurisdictions

---

### Airtime - Send Mobile Top-up

Transfer airtime/mobile credit to customer phone numbers. Deducted from your account balance.

**Endpoint:**
```
POST /version1/airtime/send
```

**Headers:**
```
Authorization: Bearer {api_key}
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
username=your_username
&recipients=%5B%7B%22phoneNumber%22%3A%22%2B254712345678%22%2C%22amount%22%3A%22KES100%22%7D%2C%7B%22phoneNumber%22%3A%22%2B256701234567%22%2C%22amount%22%3A%22UGX5000%22%7D%5D
```

(URL-decoded for clarity):
```
recipients=[{"phoneNumber":"+254712345678","amount":"KES100"},{"phoneNumber":"+256701234567","amount":"UGX5000"}]
```

**Response:**
```json
{
  "data": {
    "totalAmount": "KES 100.00",
    "totalDiscount": "KES 0.00",
    "totalSent": 2,
    "recipients": [
      {
        "phoneNumber": "+254712345678",
        "amount": "KES 100.00",
        "status": "Success",
        "discount": "KES 0.00",
        "requestId": "AIRTIME_REQ_001"
      },
      {
        "phoneNumber": "+256701234567",
        "amount": "UGX 5000.00",
        "status": "Success",
        "discount": "KES 0.00",
        "requestId": "AIRTIME_REQ_002"
      }
    ]
  }
}
```

**Important Notes:**
- Amount format: `{CURRENCY_CODE}{NUMERIC_VALUE}` (e.g., "KES100", "UGX5000")
- Discounts vary by carrier and country
- Amounts are deducted from your account balance immediately on success
- Failed transfers show in recipients array with error status
- Some carriers charge fees that reduce the received amount
- Store `requestId` for transaction reconciliation

---

### Payments - Mobile Money Checkout

Process payments directly from customer mobile wallets (M-Pesa, MTN Money, Airtel Money, etc.).

**Endpoint:**
```
POST /mobile/checkout/request
```

**Base URL:**
- Live: `https://payments.africastalking.com`
- Sandbox: `https://payments.sandbox.africastalking.com`

**Headers:**
```
Authorization: Bearer {api_key}
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
username=your_username
&phoneNumber=%2B254712345678
&amount=100
&currencyCode=KES
&productName=Premium+Subscription
&productDescription=Monthly+premium+access
&callbackUrl=https://yourserver.com/payment-callback
```

**Response:**
```json
{
  "data": {
    "checkoutRequestId": "ATPid_45e9a6326d6b8fe3088d17eb5c1a0e79",
    "status": "PendingConfirmation"
  }
}
```

**Payment Flow:**
1. Customer receives payment prompt on their phone (auto-popup in M-Pesa, etc.)
2. Customer enters their PIN to confirm
3. Africa's Talking sends webhook to your `callbackUrl` with result
4. You update order status and notify customer

**Webhook - Payment Callback (from Africa's Talking):**
```json
{
  "checkoutRequestId": "ATPid_45e9a6326d6b8fe3088d17eb5c1a0e79",
  "status": "Success",
  "phoneNumber": "+254712345678",
  "amount": 100,
  "currencyCode": "KES",
  "description": "Premium Subscription",
  "metadata": {
    "order_id": "ORDER_123"
  }
}
```

**Response Status Values:**
- `PendingConfirmation` - Awaiting customer PIN entry
- `Success` - Payment completed successfully
- `Failed` - Payment failed

---

### Payments - B2C Transfer (Business-to-Customer)

Send money from your business account to customer accounts (payouts, refunds, rewards).

**Endpoint:**
```
POST /mobile/b2c/request
```

**Base URL:**
- Live: `https://payments.africastalking.com`
- Sandbox: `https://payments.sandbox.africastalking.com`

**Headers:**
```
Authorization: Bearer {api_key}
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
username=your_username
&recipients=%5B%7B%22phoneNumber%22%3A%22%2B254712345678%22%2C%22amount%22%3A%22500%22%2C%22reason%22%3A%22Refund%22%7D%5D
&currencyCode=KES
&callbackUrl=https://yourserver.com/b2c-callback
```

(URL-decoded):
```
recipients=[{"phoneNumber":"+254712345678","amount":"500","reason":"Refund"}]
&currencyCode=KES
```

**Response:**
```json
{
  "data": {
    "numQueued": 1,
    "entries": [
      {
        "transactionId": "ATPid_45e9a6326d6b8fe3088d17eb5c1a0e79",
        "phoneNumber": "+254712345678",
        "status": "Queued"
      }
    ]
  }
}
```

**Webhook - B2C Callback (from Africa's Talking):**
```json
{
  "transactionId": "ATPid_45e9a6326d6b8fe3088d17eb5c1a0e79",
  "phoneNumber": "+254712345678",
  "amount": 500,
  "status": "Success",
  "description": "Refund",
  "failureReason": null,
  "metadata": {
    "order_id": "ORDER_123"
  }
}
```

---

### Payments - B2B Transfer (Business-to-Business)

Send money between business accounts (vendor payments, partner settlements).

**Endpoint:**
```
POST /mobile/b2b/request
```

**Base URL:**
- Live: `https://payments.africastalking.com`
- Sandbox: `https://payments.sandbox.africastalking.com`

**Headers:**
```
Authorization: Bearer {api_key}
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
username=your_username
&recipients=%5B%7B%22phoneNumber%22%3A%22%2B254712345678%22%2C%22amount%22%3A%221000%22%2C%22reason%22%3A%22Vendor+Payment%22%7D%5D
&currencyCode=KES
&callbackUrl=https://yourserver.com/b2b-callback
```

**Response:**
```json
{
  "data": {
    "numQueued": 1,
    "entries": [
      {
        "transactionId": "ATPid_45e9a6326d6b8fe3088d17eb5c1a0e79",
        "phoneNumber": "+254712345678",
        "status": "Queued"
      }
    ]
  }
}
```

**Webhook - B2B Callback (from Africa's Talking):**
```json
{
  "transactionId": "ATPid_45e9a6326d6b8fe3088d17eb5c1a0e79",
  "phoneNumber": "+254712345678",
  "amount": 1000,
  "status": "Success",
  "description": "Vendor Payment",
  "failureReason": null
}
```

---

### Account - Fetch Balance

Check your account balance and available credit.

**Endpoint:**
```
GET /version1/user?username=YOUR_USERNAME
```

**Headers:**
```
Authorization: Bearer {api_key}
```

The `username` query parameter is required.

**Response:**
```json
{
  "data": {
    "username": "your_username",
    "balance": "KES 50,000.00",
    "uniqueName": "your_company"
  }
}
```

---

## Webhooks & Callbacks

Africa's Talking sends webhooks for delivery confirmations, payment status, and voice interactions. All webhooks include a signature for verification.

**Webhook Signature Verification:**

Every webhook includes an `X-Signature` header with an HMAC-SHA256 signature. Verify it:

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(body, signature, apiKey) {
  const hash = crypto
    .createHmac('sha256', apiKey)
    .update(body)
    .digest('base64');
  return hash === signature;
}

// In your Express handler:
app.post('/webhook', (req, res) => {
  const signature = req.headers['x-signature'];
  const body = req.body; // Raw request body as string

  if (!verifyWebhookSignature(body, signature, process.env.AFRICASTALKING_API_KEY)) {
    return res.status(401).send('Unauthorized');
  }

  // Process webhook safely
  res.send('OK');
});
```

### Common Webhook Types

**SMS Delivery Report:**
```json
{
  "id": "ATXid_message_123",
  "phoneNumber": "+254712345678",
  "status": "Success",
  "retryCount": 0,
  "provider": "Safaricom",
  "timestamp": "2025-02-20T10:31:00Z"
}
```

**Incoming SMS (from customer):**
```json
{
  "id": "ATXid_in_123",
  "from": "+254712345678",
  "to": "20880",
  "text": "HELLO",
  "linkId": "short_code_session_id",
  "date": "2025-02-20T10:30:00Z"
}
```

**Voice Call Event:**
```json
{
  "callSessionId": "CALL_SESSION_123",
  "caller": "+254712345678",
  "recipient": "+254712345679",
  "direction": "Outbound",
  "dtmfDigits": "1",
  "recordingUrl": "https://media.africastalking.com/recording.mp3",
  "timestamp": "2025-02-20T10:31:00Z"
}
```

---

## Common Integration Patterns

### SMS Notification Workflow

```
1. Customer action triggered (payment, order, etc.)
2. POST /version1/messaging → send SMS to customer
3. Store messageId in database for tracking
4. Receive webhook with delivery status
5. Update database and notify support if delivery fails
6. Log successful delivery for compliance/audit
```

### USSD Menu Application

```
1. Register webhook URL in dashboard: https://yourserver.com/ussd
2. User dials *123# or sends USSD request
3. Africa's Talking POSTs to your webhook with sessionId, text=""
4. Your code determines menu based on session state
5. Return CON response with menu options
6. User selects option (e.g., "1")
7. Africa's Talking POSTs again with text="1"
8. Process selection and return CON (next menu) or END (final response)
9. Session ends when you return END
```

### Voice IVR System

```
1. POST /version1/voice/call with phoneNumber and callerId
2. Call is queued
3. When call connects, Africa's Talking calls YOUR callback URL
4. Your callback returns XML describing the IVR flow
5. User hears prompt and can press keys (DTMF)
6. Their input is POSTed back to your callback URL
7. Return next XML action (prompt, transfer, record, etc.)
8. Repeat until call ends or you return no response
```

### Airtime Top-up Feature

```
1. Customer requests airtime amount
2. Validate amount and phone number
3. Check account balance
4. POST /version1/airtime/send with recipients array
5. Check response for success/failure
6. Log transaction with requestId
7. Notify customer of success or failure
8. Update customer's transaction history/receipt
```

### Mobile Money Payment Flow

```
1. Customer initiates purchase
2. POST /mobile/checkout/request with amount, currency, callback URL
3. Receive checkoutRequestId
4. Customer receives prompt on phone to confirm payment
5. Customer enters PIN
6. Africa's Talking POSTs to your callbackUrl with status
7. If Success: fulfill order, send confirmation SMS
8. If Failed: show error, allow retry
9. Log transaction for reconciliation
```

---

## Error Handling

Africa's Talking returns status codes and error messages. Always handle failures gracefully:

**HTTP Status Codes:**
- `200` - Request accepted (check response body for details)
- `201` - Created successfully
- `400` - Bad request (invalid parameters)
- `401` - Unauthorized (invalid API key or username)
- `403` - Forbidden (insufficient balance)
- `500` - Server error (retry with exponential backoff)

**Response Status Codes (in JSON):**
- `101` - Success
- `400` - Bad request
- `401` - User does not exist
- `402` - User suspended
- `403` - Insufficient balance
- `500` - Internal server error

**Retry Strategy:**
```javascript
async function sendSmsWithRetry(phoneNumber, message, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await postToAT('/version1/messaging', {
        to: phoneNumber,
        message: message
      });

      if (response.statusCode === 101) {
        return { success: true, messageId: response.messageId };
      } else if (response.statusCode === 403) {
        throw new Error('Insufficient balance'); // Don't retry
      }
    } catch (error) {
      if (attempt < maxRetries) {
        // Exponential backoff: 2s, 4s, 8s
        await delay(Math.pow(2, attempt) * 1000);
      } else {
        return { success: false, error: error.message };
      }
    }
  }
}
```

---

## Supported Countries & Currencies

Africa's Talking operates across 150+ African countries. Key markets include:

| Country | Currency | Carriers | Features |
|---------|----------|----------|----------|
| Kenya | KES | Safaricom, Airtel, Equity | SMS, USSD, Voice, Airtime, M-Pesa |
| Uganda | UGX | MTN, Airtel, Stanbic | SMS, USSD, Voice, Airtime, Mobile Money |
| Tanzania | TZS | Vodacom, Tigo, Airtel | SMS, USSD, Voice, Airtime |
| Nigeria | NGN | MTN, Airtel, Glo, 9mobile | SMS, USSD, Voice, Airtime |
| Rwanda | RWF | Airtel, MTN | SMS, USSD, Voice, Airtime |
| Ghana | GHS | Vodafone, MTN, Airtel | SMS, USSD, Voice, Airtime |
| Zambia | ZMW | Airtel, Zain, MTN | SMS, USSD, Voice, Airtime |
| Zimbabwe | ZWL | Econet, NetOne, Telecel | SMS, USSD, Voice, Airtime |

For a complete list and real-time country/currency support, check your dashboard or contact support.

---

## Important Notes & Gotchas

### Critical Points

1. **USSD is Callback-Based, NOT Polling** - Africa's Talking POSTs to YOUR webhook. You don't poll. Your webhook must respond with CON or END.

2. **Phone Number Format** - Always use international format with country code (e.g., +254712345678). Local formats (0712345678) will fail.

3. **Character Limits:**
   - SMS: 160 characters (longer messages split automatically)
   - USSD: 182 characters per response
   - API responses are not affected by these limits

4. **Webhook Timeout** - Respond to callbacks within 30 seconds. Timeout = failure and Africa's Talking may retry.

5. **Signature Expiry** - Don't implement custom TTL logic on signatures. Africa's Talking's signature is valid for the request's lifetime.

6. **Test First** - Always test with sandbox credentials before going live:
   - Use `sandbox.africastalking.com` endpoints
   - Use sandbox credentials from your account
   - No real charges in sandbox

7. **Rate Limiting** - Africa's Talking has rate limits per account. Implement exponential backoff for retries, not linear retry loops.

8. **Sandbox Phone Numbers** - In sandbox, use any phone number format. In production, all numbers must be valid with country codes.

9. **Currency Codes** - Always use 3-letter currency codes (KES, UGX, NGN, etc.). Amount should be string: "KES100" not "100".

10. **Webhook Security** - Always verify X-Signature headers. Never trust webhook data without verification.

### Common Pitfalls

- Hardcoding API keys → Use environment variables
- Not handling USSD as callbacks → Your code must listen for POST requests
- Sending SMS without country codes → Messages fail silently
- Missing username in request body → Authentication fails
- Not responding to USSD with CON/END → Message delivery fails
- Implementing custom webhook TTL → Breaks signature verification
- Forgetting to URL-encode phone numbers → Request malformed
- Not storing messageId/transactionId → Can't track delivery/payments
- Testing with live credentials → Charges to account
- Assuming synchronous responses → Most operations are async

---

## API Endpoints Summary

| Service | Live URL | Sandbox URL |
|---------|----------|-------------|
| SMS | `https://api.africastalking.com/version1/messaging` | `https://api.sandbox.africastalking.com/version1/messaging` |
| USSD (Webhook) | Your callback URL | Your callback URL |
| Voice Call | `https://voice.africastalking.com/call` | `https://voice.sandbox.africastalking.com/call` |
| Airtime | `https://api.africastalking.com/version1/airtime/send` | `https://api.sandbox.africastalking.com/version1/airtime/send` |
| Account Balance | `https://api.africastalking.com/version1/user` | `https://api.sandbox.africastalking.com/version1/user` |
| Mobile Checkout | `https://payments.africastalking.com/mobile/checkout/request` | `https://payments.sandbox.africastalking.com/mobile/checkout/request` |
| B2C Transfer | `https://payments.africastalking.com/mobile/b2c/request` | `https://payments.sandbox.africastalking.com/mobile/b2c/request` |
| B2B Transfer | `https://payments.africastalking.com/mobile/b2b/request` | `https://payments.sandbox.africastalking.com/mobile/b2b/request` |

---

## Useful Links & Resources

- **Official Documentation**: https://developers.africastalking.com/
- **Dashboard & Account Management**: https://account.africastalking.com
- **Help Center**: https://help.africastalking.com
- **API Endpoints Reference**: https://help.africastalking.com/en/articles/2232953-what-are-the-africa-s-talking-api-endpoints
- **Sandbox Getting Started**: https://help.africastalking.com/en/articles/1170660-how-do-i-get-started-on-the-africa-s-talking-sandbox
- **Authentication Docs**: https://developers.africastalking.com/docs/authentication
- **SMS API Docs**: https://developers.africastalking.com/docs/sms/overview
- **USSD API Docs**: https://developers.africastalking.com/docs/ussd/overview
- **Voice API Docs**: https://africastalking.com/voice
- **Payments Docs**: https://developers.africastalking.com/docs/payments/mobile_c2b/checkout
- **GitHub SDKs**: https://github.com/AfricasTalkingLtd
- **Node.js SDK**: https://www.npmjs.com/package/africastalking
