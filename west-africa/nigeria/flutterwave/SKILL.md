---
name: flutterwave
description: "Integrate with Flutterwave's payment API for pan-African commerce. Use this skill whenever the user wants to accept payments across multiple African countries, process mobile money payments, handle card charges, create payment links, issue virtual cards, send payouts/transfers, or work with Flutterwave (Rave) in any way. Also trigger for 'Flutterwave', 'Rave', 'African mobile money', 'pan-African payments', 'multi-currency Africa checkout', or when the user needs to support payments in Nigeria, Ghana, Kenya, South Africa, Tanzania, Uganda, and 30+ other African markets simultaneously."
---

# Flutterwave Integration Skill

Flutterwave is Africa's leading pan-African payment infrastructure, supporting 150+ currencies and multiple payment methods across 30+ African countries. It handles card payments, mobile money (M-Pesa, MTN MoMo, Airtel Money), bank transfers, USSD, and more — all through a single API.

## When to use this skill

You're building something that needs to accept or send payments across Africa — not just one country. A marketplace connecting buyers and sellers across borders, a SaaS serving multiple African markets, a remittance product, or anything where multi-country coverage matters. Flutterwave is the "one API for all of Africa" play.

## Authentication

All requests require your secret key as a Bearer token:

```
Authorization: Bearer FLWSECK_TEST-xxxxx
```

Keys: `FLWSECK_TEST-` for sandbox, `FLWSECK-` for production. Store in `FLUTTERWAVE_SECRET_KEY` env var.

**Base URL:** `https://api.flutterwave.com/v3`

> ⚠️ **v3 vs v4:** This skill documents the **v3 API**, which is fully functional and widely used. Flutterwave launched **v4** in 2024 with OAuth 2.0 authentication (`https://idp.flutterwave.com/realms/flutterwave/protocol/openid-connect/token`) and updated sandbox URLs (`https://developersandbox-api.flutterwave.com`). If you are starting a new integration and your dashboard offers v4 credentials, prefer v4. The v3 `FLWSECK_` secret key auth described here will continue to work for existing integrations.

## Core API Reference

### Standard Payment (Redirect)

The simplest way to collect payments — create a payment link and redirect the customer.

```
POST /payments
```

```json
{
  "tx_ref": "txn_unique_ref_123",
  "amount": 5000,
  "currency": "NGN",
  "redirect_url": "https://yoursite.com/payment/callback",
  "customer": {
    "email": "customer@email.com",
    "name": "Amina Okafor",
    "phonenumber": "+2348012345678"
  },
  "customizations": {
    "title": "My Store",
    "description": "Payment for Order #123",
    "logo": "https://yoursite.com/logo.png"
  },
  "payment_options": "card, mobilemoney, ussd, banktransfer",
  "meta": {
    "order_id": "ORD-123"
  }
}
```

**Important:** Amounts are in the **major currency unit** (not kobo/pesewas). ₦5000 = `5000`, not `500000`.

**Response:**
```json
{
  "status": "success",
  "message": "Hosted Link",
  "data": {
    "link": "https://checkout.flutterwave.com/v3/hosted/pay/xxxxx"
  }
}
```

Redirect customer to `data.link`. After payment, they return to `redirect_url?tx_ref=txn_unique_ref_123&transaction_id=12345&status=successful`.

### Initiate Mobile Money Charge

For direct mobile money integration (M-Pesa, MTN MoMo, etc.):

```
POST /charges?type=mobile_money_kenya
```

```json
{
  "tx_ref": "mm_ref_123",
  "amount": 1000,
  "currency": "KES",
  "email": "customer@email.com",
  "phone_number": "254712345678",
  "network": "Safaricom"
}
```

Types by country and supported networks:
- **Kenya**: `mobile_money_kenya` (M-Pesa via Safaricom)
- **Ghana**: `mobile_money_ghana` (MTN, Vodafone, Airtel)
- **Uganda**: `mobile_money_uganda` (MTN, Airtel)
- **Rwanda**: `mobile_money_rwanda` (MTN, Airtel)
- **Tanzania**: `mobile_money_tanzania` (Airtel, Tigo, Halopesa)
- **Côte d'Ivoire**: `mobile_money_franco` (MTN, Orange, Wave)
- **Senegal**: `mobile_money_franco` (Orange, Free Money, Wave)
- **Zambia**: `mobile_money_franco` (Airtel, MTN, Zamtel)

**Response includes a `redirect` URL or requires OTP validation depending on the provider.**

### Initiate Card Charge

For direct card integration (PCI DSS compliant environments only):

```
POST /charges?type=card
```

```json
{
  "tx_ref": "card_ref_123",
  "amount": 5000,
  "currency": "NGN",
  "email": "customer@email.com",
  "card_number": "4187427415564246",
  "cvv": "828",
  "expiry_month": "09",
  "expiry_year": "32",
  "fullname": "Amina Okafor",
  "enckey": "your_encryption_key"
}
```

Card data must be encrypted with AES-256 using your encryption key. Retrieve your encryption key from your Flutterwave dashboard API settings. Response may require `pin`, `otp`, `redirect` (3DS), or `avs_noauth` — handle each `meta.authorization.mode` accordingly.

### Verify a Transaction

Always verify server-side:

```
GET /transactions/{transaction_id}/verify
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "id": 12345,
    "tx_ref": "txn_unique_ref_123",
    "flw_ref": "FLW-MOCK-xxxxx",
    "amount": 5000,
    "currency": "NGN",
    "charged_amount": 5000,
    "status": "successful",
    "payment_type": "card",
    "customer": {
      "email": "customer@email.com",
      "name": "Amina Okafor"
    }
  }
}
```

Verify: `data.status === "successful"`, `data.amount` and `data.currency` match your expected values.

### List Transactions

```
GET /transactions?from=2025-01-01&to=2025-01-31&page=1&status=successful
```

Query params: `from`, `to` (YYYY-MM-DD), `page`, `status`, `tx_ref`, `customer_email`, `currency`.

### Create a Payment Link

For no-code payment pages:

```
POST /payment-plans
```

```json
{
  "amount": 10000,
  "name": "Annual Subscription",
  "interval": "monthly",
  "currency": "NGN"
}
```

Intervals: `daily`, `weekly`, `monthly`, `quarterly`, `yearly`.

### Create a Transfer (Payout)

Send money to bank accounts or mobile money wallets:

```
POST /transfers
```

```json
{
  "account_bank": "044",
  "account_number": "0690000040",
  "amount": 5000,
  "currency": "NGN",
  "narration": "Seller payout",
  "reference": "transfer_ref_123",
  "debit_currency": "NGN"
}
```

For mobile money payouts:
```json
{
  "account_bank": "MPS",
  "account_number": "254712345678",
  "amount": 1000,
  "currency": "KES",
  "narration": "Driver payout",
  "reference": "mm_payout_123",
  "beneficiary_name": "John Kamau"
}
```

### List Banks

```
GET /banks/{country_code}
```

Country codes: `NG` (Nigeria), `GH` (Ghana), `KE` (Kenya), `ZA` (South Africa), `TZ` (Tanzania), `UG` (Uganda), `RW` (Rwanda), `SN` (Senegal), `CM` (Cameroon), `CI` (Côte d'Ivoire).

### Create Virtual Card

```
POST /virtual-cards
```

```json
{
  "currency": "USD",
  "amount": 5000,
  "billing_name": "Amina Okafor",
  "billing_address": "123 Main St",
  "billing_city": "Lagos",
  "billing_state": "Lagos",
  "billing_postal_code": "100001",
  "billing_country": "NG",
  "first_name": "Amina",
  "last_name": "Okafor",
  "date_of_birth": "1990/01/15",
  "email": "amina@email.com",
  "phone": "+2348012345678",
  "title": "Mr",
  "gender": "F"
}
```

## Webhooks

Set your webhook URL in the Flutterwave dashboard. Verify with the secret hash:

The `verif-hash` header Flutterwave sends is the literal secret hash value configured in your dashboard. Compare it directly — it is **not** an HMAC digest.

```javascript
const secretHash = process.env.FLW_SECRET_HASH;
const signature = req.headers['verif-hash'];

if (!signature || signature !== secretHash) {
  return res.status(401).send('Unauthorized');
}

// Process webhook
res.status(200).send('OK');
```

**Critical:** Return HTTP 200 status code to acknowledge receipt. Any other response code (including 3xx redirects) is treated as a failure.

Key events: `charge.completed`, `transfer.completed`, `payment_plan.created`, `charge.updated`.

## Error Handling

Standard error response format:

```json
{
  "status": "error",
  "message": "What went wrong",
  "data": null
}
```

Common HTTP status codes and meanings:
- **400**: Validation error — check `message` for details
- **401**: Unauthorized — invalid or missing API key
- **403**: Forbidden — insufficient permissions
- **404**: Resource not found
- **429**: Rate limited — retry after delay
- **500**: Server error on Flutterwave's end
- **503**: Service unavailable or timeout

Response codes in transaction responses:
- `0`, `00`, `02`: Success
- Any other value: Failure — check `responseMessage` for reason

Implement exponential backoff for 429/503 errors and always retry failed requests with unique transaction reference IDs.

## Important Notes / Gotchas

### Card Charge Encryption
- Card charges **require AES-256 encryption** of card data before sending to the API
- The encryption key is different from your secret API key — retrieve it from dashboard Settings > API
- Never log or store unencrypted card data
- Always handle `3DS` redirect mode for card transactions (not all cards authorize immediately)
- PCI DSS compliance is your responsibility; use Flutterwave's hosted payment page when possible to avoid PCI burden

### FLW_SECRET_KEY vs Secret Hash
- **`FLWSECK_` key**: Your API authentication secret for making requests to the API (goes in Authorization header)
- **`FLW_SECRET_HASH`**: Your webhook verification hash configured in the dashboard. Flutterwave sends this exact value in the `verif-hash` header with every webhook. Compare directly — no HMAC computation required.
- These are two separate keys set in your Flutterwave dashboard

### Country-Specific Payment Methods
- Not all payment methods are available in all countries
- Mobile money requires the exact country code and correct `mobile_money_*` charge type
- Some countries like Nigeria support USSD (dial `*` codes), others don't
- Always call `GET /banks/{country_code}` first to get valid bank codes before initiating transfers

### Virtual Card Limitations
- Virtual cards are issued in USD or other specified currencies
- Country/region restrictions apply based on issuing country
- Cards have transaction limits per card and per month
- Virtual cards are for online transactions; in-store use may be restricted
- Flutterwave deprecated the Barter virtual card service — use the current Card Issuing API

### Amount and Currency Quirks
- Amounts are in **major units** (not minor units like cents), unlike Paystack
- ₦5000 NGN = `5000` (not 500000), $10 USD = `10` (not 1000)
- Each country has supported currencies; verify before making requests
- Some currencies like NGN and GHS are local-only; international transfers may auto-convert
- Settlement happens in your configured business currency, not the transaction currency

### Mobile Money Specifics
- M-Pesa (Kenya) requires phone number with 254 prefix (international format)
- MTN MoMo transactions may require customer confirmation via prompt on their phone
- OTP prompts vary by provider and country — plan UX for potential delays
- Some mobile money networks settle within 24 hours; plan cash flow accordingly
- Mobile money refunds can take 3-5 business days

### Rate Limiting and Retries
- Flutterwave has rate limits; implement exponential backoff for 429 responses
- Use unique `tx_ref` values for each transaction attempt
- Don't retry immediately; use 1s → 2s → 4s delays
- A failed transaction verify doesn't mean the payment failed — always check webhook events

### Webhook Reliability
- Webhooks are not guaranteed to arrive once; implement idempotency checks on your side
- Process webhooks asynchronously to avoid timeouts
- Always verify the `verif-hash` header signature before trusting webhook data
- Test webhook handling thoroughly in sandbox before going live

### Testing
- Use sandbox key `FLWSECK_TEST-` for all test transactions
- Test card: `4187427415564246`, CVV: `828`, any future expiry
- Mobile money testing requires test phone numbers per country (check docs)
- Some payment methods don't work in sandbox — use virtual cards for testing online store scenarios

## Common Integration Patterns

### Multi-country checkout
1. `POST /payments` with the customer's currency (NGN, KES, GHS, etc.)
2. Flutterwave auto-detects available payment methods for that currency
3. Customer pays with their preferred method
4. `GET /transactions/{id}/verify` server-side
5. Settlement happens in your configured currency

### Cross-border payouts
1. Collect payments in any currency
2. `GET /banks/{country}` to get bank codes for target country
3. `POST /transfers` to send to seller's bank or mobile money
4. Track with webhooks or `GET /transfers/{id}`

### Subscription billing
1. Create plan with `POST /payment-plans`
2. Pass `payment_plan` ID when initializing payment
3. Flutterwave charges the customer on schedule
4. Handle failures via webhooks

## Supported Countries & Currencies

Major markets: Nigeria (NGN), Ghana (GHS), Kenya (KES), South Africa (ZAR), Tanzania (TZS), Uganda (UGX), Rwanda (RWF), Cameroon (XAF), Côte d'Ivoire (XOF), Senegal (XOF), Egypt (EGP), and 20+ more.

Flutterwave is supported in Nigeria, Cameroon, Côte d'Ivoire, Egypt, Ghana, Kenya, Rwanda, Senegal, South Africa, Uganda, Tanzania, The United Kingdom, America, and Europe.

## Useful Links

- API Docs: https://developer.flutterwave.com/docs
- Webhooks: https://developer.flutterwave.com/docs/webhooks
- Encryption: https://developer.flutterwave.com/docs/encryption
- Mobile Money: https://developer.flutterwave.com/docs/mobile-money
- Error Codes: https://developer.flutterwave.com/docs/common-errors
- Dashboard: https://dashboard.flutterwave.com
- Test Cards: https://developer.flutterwave.com/docs/testing-helpers/
