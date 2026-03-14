---
name: termii
description: "Integrate with the Termii messaging and OTP API for Nigerian SMS and one-time password delivery. Use this skill whenever the user wants to send SMS messages, send OTP codes, verify OTP tokens, build messaging systems, handle authentication, or work with Termii's SMS and OTP services. Also trigger when the user mentions 'Termii', 'SMS API Nigeria', 'OTP delivery', 'two-factor authentication', 'SMS authentication', or needs to send messages to Nigerian phone numbers."
---

# Termii Integration Skill

Termii is Nigeria's leading cross-channel messaging platform, providing RESTful APIs for SMS delivery, WhatsApp messaging, voice calls, and enterprise-grade OTP authentication. It delivers messages across all Nigerian network operators (MTN, Airtel, Glo, 9Mobile) with real-time status tracking and global reach.

## When to use this skill

Use Termii when building apps requiring SMS communication or user authentication in Nigeria or across Africa. Common use cases include:
- Two-factor authentication (2FA) and account verification
- One-time password (OTP) generation and validation
- Transactional alerts and notifications (payment confirmations, security alerts)
- Bulk promotional SMS campaigns
- WhatsApp template-based messaging
- User registration and phone number verification

Termii handles carrier routing complexity, manages DND (Do Not Disturb) regulations, and ensures reliable delivery across all networks.

## Authentication

Termii uses API Key authentication passed in request bodies (not headers):

```
API Key: your_api_key
Base URL: https://api.termii.com
```

Generate your API key in your Termii dashboard and store it in an environment variable `TERMII_API_KEY`. Never hardcode credentials in code.

## Core API Reference

### Send SMS Message

Send a text message to one or multiple phone numbers with flexible routing options.

```
POST /api/sms/send
```

**Body:**
```json
{
  "to": "+2348012345678",
  "from": "YourBrand",
  "sms": "Your verification code is 123456. Valid for 10 minutes.",
  "type": "plain",
  "api_key": "your_api_key",
  "channel": "generic"
}
```

**Key parameters:**
- `to`: Phone number with country code (+234 for Nigeria)
- `from`: Registered sender ID or "generic" (custom IDs require approval)
- `channel`: "generic" (promotional, best-effort delivery) or "dnd" (transactional, guaranteed)
- `type`: "plain" or "unicode" for special characters

**Critical routing choice:**
- **Generic route**: For promotional SMS. No DND compliance needed. MTN has delivery restrictions 8PM-8AM WAT. Non-guaranteed delivery.
- **DND route**: For transactional SMS (OTP, alerts, confirmations). Always delivered. MTN no time restrictions. Required for OTP — using generic route for OTP will eventually result in delivery failures or sender ID blocking.

**Message sizing:**
- 160 characters per SMS unit (standard ASCII)
- 70 characters per unit for messages with special characters
- Long messages automatically split across multiple units
- Each unit is charged separately

**Response:**
```json
{
  "message_id": "MSG_xxxxx",
  "message": "Message sent successfully",
  "balance": 95.50,
  "user": "user@example.com",
  "type": "plain"
}
```

Use `message_id` for tracking. `balance` shows remaining credits.

### Send Number Message (Alternative SMS Route)

Send SMS via direct number routing for direct carrier access.

```
POST /api/sms/number/send
```

**Body:**
```json
{
  "to": "+2348012345678",
  "sms": "Your message content",
  "api_key": "your_api_key"
}
```

Alternative to sender ID-based routing for direct number delivery.

### Send OTP Code

Generate and send a one-time password with configurable expiry, attempt limits, and message templates.

```
POST /api/sms/otp/send
```

**Body:**
```json
{
  "to": "+2348012345678",
  "from": "YourBrand",
  "api_key": "your_api_key",
  "message_type": "ALPHANUMERIC",
  "message_text": "Your verification code is {code}. Valid for 10 minutes.",
  "code_length": 6,
  "pin_attempts": 3,
  "pin_time_to_live": 600,
  "pin_placeholder": "{code}",
  "pin_type": "NUMERIC"
}
```

**Key parameters:**
- `message_text`: Custom message with `{code}` placeholder where OTP will be inserted
- `code_length`: Length of generated code (typically 4-8 digits)
- `pin_time_to_live`: Expiry time in seconds (600 = 10 minutes)
- `pin_attempts`: Maximum verification attempts before code expires (3-5 recommended)
- `pin_type`: "NUMERIC" for digits only, "ALPHANUMERIC" for mixed

**Response:**
```json
{
  "code": "ok",
  "message_id": "MSG_xxxxx",
  "pinId": "PIN_xxxxx",
  "to": "+2348012345678",
  "sms_status": "Message Sent",
  "message": "OTP sent successfully"
}
```

**Critical:** Store the `pinId` — you must use it to verify the OTP. A random code is automatically generated and sent to the customer.

### Verify OTP Code

Check if the code entered by a customer matches the sent OTP.

```
POST /api/sms/otp/verify
```

**Body:**
```json
{
  "pin_id": "PIN_xxxxx",
  "code": "123456",
  "api_key": "your_api_key"
}
```

**Response (success):**
```json
{
  "verified": true,
  "message": "OTP verified successfully",
  "msisdn": "+2348012345678",
  "pin_id": "PIN_xxxxx"
}
```

**Response (failure):**
```json
{
  "verified": false,
  "message": "OTP is incorrect or has expired",
  "pin_id": "PIN_xxxxx"
}
```

Once verified, the OTP is immediately invalidated and cannot be reused.

### Send WhatsApp Template Message

Send templated messages via WhatsApp.

```
POST /api/send/template
```

**Body:**
```json
{
  "to": "+2348012345678",
  "template_id": "your_template_id",
  "api_key": "your_api_key",
  "data": {
    "customer_name": "John"
  }
}
```

Templates must be pre-approved by Termii and WhatsApp.

### Resend OTP

Resend an OTP to the same customer if they didn't receive the initial code.

```
POST /api/sms/otp/resend
```

**Body:**
```json
{
  "pin_id": "PIN_xxxxx",
  "api_key": "your_api_key"
}
```

Generates a new code and invalidates the previous one.

### Get Message Status

Query the delivery status of a sent message.

```
GET /api/sms/message/query?api_key=your_api_key&message_id=MSG_xxxxx
```

**Response:**
```json
{
  "message_id": "MSG_xxxxx",
  "to": "+2348012345678",
  "sms_status": "Delivered",
  "sent_time": "2026-02-24T14:30:00Z",
  "delivery_time": "2026-02-24T14:31:00Z",
  "status_code": "1"
}
```

Status codes: `1` = Delivered, `0` = Pending, `-1` = Failed.

### Get Account Balance

Check your account balance in credits.

```
GET /api/get/balance?api_key=your_api_key
```

**Response:**
```json
{
  "balance": 250.50,
  "currency": "NGN",
  "user": "user@example.com",
  "status": "success"
}
```

Monitor balance regularly to avoid mid-transaction failures.

## Webhook Verification

Termii sends webhooks for delivery confirmation. Always verify signatures:

```javascript
const crypto = require('crypto');
const hash = crypto
  .createHmac('sha256', webhook_secret)
  .update(JSON.stringify(req.body))
  .digest('hex');

if (hash === req.headers['x-termii-signature']) {
  // Webhook is authentic, process it
}
```

Key events: `sms.delivered`, `sms.failed`, `otp.verified`, `otp.failed`.

## Common Integration Patterns

### Two-factor authentication (2FA)
1. User logs in with credentials
2. Call `POST /api/sms/otp/send` to customer's phone
3. Store `pinId` in session (user's browser or server)
4. User enters 6-digit code on verification screen
5. Call `POST /api/sms/otp/verify` with `pinId` and code
6. If `verified: true`, allow login; if false, show error and allow retry (max 3 attempts)

### Phone number verification (registration)
1. During signup, collect phone number
2. Call `POST /api/sms/otp/send` immediately
3. Show verification screen with countdown timer
4. User enters code, call `POST /api/sms/otp/verify`
5. Mark phone as verified in user profile and proceed

### OTP resend flow
1. Send initial OTP via `POST /api/sms/otp/send`
2. If user doesn't receive: call `POST /api/sms/otp/resend`
3. New code generated, previous code expires
4. User enters new code and verifies via `POST /api/sms/otp/verify`

### Transactional alerts
1. Critical event occurs (payment, security alert)
2. Call `POST /api/sms/send` with `channel: "dnd"` (guaranteed delivery)
3. Include relevant details (amount, timestamp, action required)
4. Store message ID and delivery status for audit trail

## Error Handling

Termii returns error responses with HTTP status codes and descriptive messages:

```json
{
  "code": "error_code",
  "message": "Description of error",
  "status": "failed"
}
```

**HTTP Status codes:**
- `400`: Validation error (invalid phone format, missing required fields)
- `401`: Invalid or missing API key
- `403`: Insufficient account balance
- `429`: Rate limited — implement exponential backoff
- `500`: Server error — retry with exponential backoff

**Common error codes:**
- `InvalidPhoneNumber`: Phone number format incorrect or missing country code
- `InsufficientBalance`: Account balance insufficient for message
- `OTPExpired`: OTP code has expired; customer must request new code
- `InvalidOTP`: Code entered is incorrect; allow retry if attempts remain
- `MaxAttemptsExceeded`: Customer exceeded maximum verification attempts

## Critical Implementation Notes

**Route selection for OTP:** Always use `channel: "dnd"` for OTP endpoints. Generic route lacks guaranteed delivery, will eventually fail, and risks sender ID blocking during high-volume periods.

**Phone number formatting:** Always include country code (+234 for Nigeria). Termii rejects improperly formatted numbers.

**Sender ID requirements:** Custom sender IDs must be registered in your Termii dashboard. Unregistered IDs are rejected. Use "generic" as fallback.

**OTP expiry timing:** Set `pin_time_to_live` to 10 minutes (600 seconds) for optimal balance between security and usability. Too short causes frustration; too long increases compromise risk.

**Attempt limits:** Set `pin_attempts` to 3-5. Lower limits may frustrate legitimate users; higher limits enable brute-force attacks.

**Message pagination:** Messages exceeding 160 characters (70 with special chars) split automatically. Each unit consumes one credit. Plan message length accordingly.

**Delivery SLA:** SMS typically deliver within seconds, but can take up to minutes during peak network hours. Design flows with expected delays in mind.

**Rate limiting:** Implement at least 30-second delays between OTP sends to the same number. Rapid repeated sends trigger Termii rate limits and risk account suspension.

**Balance monitoring:** Monitor account balance before initiating OTP flows. Depletion mid-authentication breaks user experience.

## Useful Links

- Official Documentation: https://developers.termii.com
- Dashboard: https://dashboard.termii.com
- API Base URL: https://api.termii.com
- Sender ID Management: https://dashboard.termii.com/sender-ids
