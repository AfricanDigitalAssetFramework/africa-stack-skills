---
name: ikhokha
description: "Integrate with iKhokha's mobile POS payment solution for South Africa. Use this skill whenever the user wants to accept card payments via iKhokha, integrate POS capabilities, process mobile payments, manage transaction history, handle refunds, reconcile settlements, build on iKhokha's platform, or work with iKhokha's API and SDKs. Also trigger when the user mentions 'iKhokha', 'mobile POS South Africa', 'iKhokha payments', 'card payments mobile', 'iK Pay', 'iK Pay API', or needs POS payment integration."
---

# iKhokha Integration Skill

iKhokha is South Africa's leading mobile Point of Sale (POS) solution enabling merchants to accept card payments on smartphones and tablets. iKhokha operates as both a hardware-integrated POS system (via Bluetooth-connected card readers) and provides a REST-based iK Pay API for payment link generation and online payment processing. The company serves thousands of South African businesses from micro-merchants to large retailers.

## When to Use This Skill

You're building or integrating with:
- **Mobile POS applications** requiring card acceptance on Android/iOS devices
- **Hardware-integrated payment systems** using iKhokha's Mover, Flyer, or Shaker card machines via Bluetooth
- **Online payment gateways** needing to generate and manage payment links
- **E-commerce integrations** for WooCommerce, Shopify, or custom platforms in South Africa
- **Custom merchant dashboards** requiring transaction reporting, settlement tracking, or refund management
- **Field sales or delivery apps** needing mobile-first card acceptance

iKhokha is optimized for the South African market, supporting ZAR transactions, local payment methods (Instant EFT, digital wallets), and PCI DSS compliance.

## Authentication

### API Key & Application ID

iKhokha uses a hybrid authentication model:

**For REST API (iK Pay API):**
- **IK-AppID:** Your application/merchant ID
- **AppSecret:** Your application secret key
- **ik-sign:** HMAC-SHA256 signature of the request payload

**Authentication Headers:**
```
IK-AppID: YOUR_APP_ID
ik-sign: HMAC_SHA256_SIGNATURE
Content-Type: application/json
```

**Signature Computation:**
The `ik-sign` header is generated using HMAC-SHA256 with your AppSecret as the key and the request body (JSON-serialized) as the message. Store credentials in environment variables:

```bash
export IK_APPID=your_app_id
export IK_APPSECRET=your_app_secret
export IK_ENTITYID=your_entity_id
```

### Credential Management

- Generate API credentials from your iKhokha Merchant Dashboard → Payments → iK Pay API
- Each merchant account gets a unique AppID, AppSecret, and EntityID
- For multi-location merchants, you may manage locations separately within a single account or use location identifiers in API calls
- Never commit credentials to version control; use environment variables or secure secret management

**Base URL (development):** `https://dev.ikhokha.com/api`

<!-- TODO: confirm production base URL — developer.ikhokha.com or another host; the dev URL above is for testing only -->

## Core API Reference

### Important Note on iKhokha's Architecture

**iKhokha operates in two distinct modes:**

1. **Hardware POS Mode (Primary):** Android SDK integrated with Bluetooth card readers (iK Mover, iK Flyer, iK Shaker). This is the flagship product and requires the iKhokha Android app or custom SDK integration.

2. **REST API Mode (iK Pay API):** Limited REST endpoints for payment link generation and online payments. This is secondary and primarily serves e-commerce, invoicing, and remote payment scenarios.

**Most production deployments use the hardware + SDK approach**, not the REST API alone. If you need real-time, high-volume transaction processing with immediate card acceptance, use the Android SDK with Bluetooth card readers.

### Payment Link Creation (iK Pay API)

Generate a payment link that customers can click to pay online.

```
POST /paymentLink
```

**Headers:**
```
IK-AppID: YOUR_APP_ID
ik-sign: CALCULATED_HMAC_SHA256
Content-Type: application/json
```

**Request Body:**
```json
{
  "amount": 19999,
  "currency": "ZAR",
  "externalTransactionID": "ORDER-12345",
  "requesterUrl": "https://yoursite.com",
  "description": "Payment for Order 12345",
  "entityID": "YOUR_ENTITY_ID",
  "callbackUrl": "https://yoursite.com/ikhokha-callback",
  "successUrl": "https://yoursite.com/success",
  "failureUrl": "https://yoursite.com/failure",
  "cancelUrl": "https://yoursite.com/cancel"
}
```

**Response (Success - responseCode 00):**
```json
{
  "responseCode": "00",
  "message": "Payment link created successfully",
  "paylinkUrl": "https://ikhokha.com/pay/abc123def456xyz",
  "paylinkID": "link_abc123def456xyz",
  "externalTransactionID": "ORDER-12345",
  "amount": 19999,
  "currency": "ZAR",
  "expiry": "2025-02-24T23:59:59Z"
}
```

**Response (Error):**
```json
{
  "responseCode": "01",
  "message": "Invalid request parameters",
  "externalTransactionID": "ORDER-12345"
}
```

**Important:** Amount is in **cents** (smallest currency unit). R199.99 = 19999 cents.

### Query Payment Link Status

Retrieve the current status of a payment link.

```
GET /paymentLink/{paylinkID}
```

**Response (Pending):**
```json
{
  "responseCode": "00",
  "paylinkID": "link_abc123def456xyz",
  "externalTransactionID": "ORDER-12345",
  "amount": 19999,
  "currency": "ZAR",
  "status": "pending",
  "createdAt": "2025-02-23T10:30:00Z",
  "expiryAt": "2025-02-24T23:59:59Z"
}
```

**Response (Paid):**
```json
{
  "responseCode": "00",
  "paylinkID": "link_abc123def456xyz",
  "externalTransactionID": "ORDER-12345",
  "amount": 19999,
  "currency": "ZAR",
  "status": "paid",
  "transactionID": "txn_abc123def456",
  "paymentMethod": "card",
  "cardBrand": "VISA",
  "cardLastFour": "1111",
  "paidAt": "2025-02-23T15:45:00Z"
}
```

**Status Values:** `pending`, `paid`, `expired`, `cancelled`

### Android SDK Integration (Hardware POS)

**For mobile app developers**, iKhokha provides an Android SDK for direct card machine integration:

```java
// Initialize SDK
iKhokhaPaymentSDK sdk = new iKhokhaPaymentSDK(
    context,
    appId,
    appSecret,
    entityId
);

// Connect to card reader via Bluetooth
sdk.connectCardReader("iK_MOVER_001");

// Process payment
PaymentRequest request = new PaymentRequest.Builder()
    .amount(19999)  // In cents
    .currency("ZAR")
    .externalTransactionId("ORDER-12345")
    .description("Product XYZ Purchase")
    .build();

sdk.processPayment(request, new PaymentCallback() {
    @Override
    public void onSuccess(PaymentResult result) {
        String transactionId = result.getTransactionId();
        String cardBrand = result.getCardBrand();
        // Handle successful payment
    }

    @Override
    public void onFailure(PaymentError error) {
        String errorCode = error.getCode();
        String errorMessage = error.getMessage();
        // Handle declined payment
    }
});
```

**Supported Card Machines:** iK Mover, iK Flyer, iK Shaker (all Bluetooth-enabled, PCI-PTS certified, EMV Level 1 & 2 compliant)

**Supported Payment Methods:**
- Credit/Debit cards (Visa, Mastercard, American Express)
- Instant EFT
- Google Pay
- Apple Pay
- Samsung Pay

## Webhooks

iKhokha sends webhook notifications for payment and link events. Configure your webhook endpoint in the Merchant Dashboard.

**Webhook Signature Verification:**

Webhooks include authentication headers:
```
IK-AppID: YOUR_APP_ID
ik-sign: HMAC_SHA256_SIGNATURE
Content-Type: application/json
```

Verify the `ik-sign` header using HMAC-SHA256 with your AppSecret and the raw request body.

**Webhook Events:**

```json
{
  "event": "paymentLink.paid",
  "timestamp": "2025-02-23T15:45:00Z",
  "data": {
    "paylinkID": "link_abc123def456xyz",
    "externalTransactionID": "ORDER-12345",
    "transactionID": "txn_abc123def456",
    "amount": 19999,
    "currency": "ZAR",
    "paymentMethod": "card",
    "cardBrand": "VISA",
    "cardLastFour": "1111",
    "paidAt": "2025-02-23T15:45:00Z"
  }
}
```

**Event Types:**
- `paymentLink.created` — Payment link generated
- `paymentLink.paid` — Link payment successful
- `paymentLink.failed` — Link payment declined
- `paymentLink.expired` — Link expired without payment

Return HTTP 200 OK to acknowledge receipt.

## Common Integration Patterns

### E-commerce Checkout Flow

1. Customer adds items to cart and proceeds to checkout
2. Your backend creates a payment link: `POST /api/paymentLink`
3. Retrieve the `paylinkUrl` from response
4. Redirect customer to `paylinkUrl` (iKhokha's hosted payment page)
5. Customer enters card details and completes payment
6. Receive webhook notification (`paymentLink.paid`)
7. Update order status in your system
8. Send customer confirmation email

### WooCommerce / Shopify Plugin Integration

iKhokha provides pre-built plugins for major platforms:

- **WooCommerce:** Install iKhokha Payment Gateway Plugin from WordPress.org
- **Shopify:** Download iKhokha app from Shopify App Store

These plugins handle authentication and payment link creation automatically.

### Field Sales / Delivery App (Hardware + SDK)

1. Sales representative opens your custom Android app
2. App connects to Bluetooth card reader (iK Mover)
3. Rep enters sale amount and customer details
4. App calls `sdk.processPayment(request)` with card reader
5. Customer inserts/taps card on device
6. App receives payment result (success/decline)
7. Generate and print receipt on mobile printer
8. Upload transaction to your backend
9. Sync all transactions for end-of-day reconciliation

### Transaction Reporting & Settlement Tracking

1. End of day (EOD), query Merchant Dashboard or use iKhokha's reporting API
2. Retrieve settlement details: total transactions, amount, fees
3. Reconcile against your POS records
4. Identify failed/reversed transactions
5. Investigate discrepancies
6. Prepare for daily settlement payout (typically 1-2 business days to bank account)

### Handling Payment Link Expiry

- Payment links remain valid until payment is successfully made
- Expired links cannot be reused; generate new links for retry
- Set reasonable expiry times in your payment flow (24-48 hours typical)
- After expiry, offer customer option to generate new link
- Mark expired orders in your system accordingly

## Error Handling

### API Response Codes

All iKhokha API responses include a `responseCode` field:

```json
{
  "responseCode": "00",
  "message": "Success",
  "data": { ... }
}
```

**Common Response Codes:**

| Code | Meaning | Action |
|------|---------|--------|
| 00 | Success | Proceed normally |
| 01 | Invalid request | Check request parameters (amount, currency, dates) |
| 02 | Authentication failed | Verify AppID, AppSecret, ik-sign signature |
| 03 | Insufficient permissions | Check entity ID and account permissions |
| 04 | Resource not found | Verify payment link ID or transaction ID exists |
| 05 | Card declined | Customer should try different card or contact bank |
| 12 | Invalid card | Card number/expiry invalid; ask customer to verify |
| 51 | Insufficient funds | Card has insufficient balance; try different card |
| 91 | Network error | Retry request with exponential backoff |
| 99 | System error | Contact iKhokha support |

### HTTP Status Codes

| Status | Meaning |
|--------|---------|
| 200 OK | Request successful |
| 400 Bad Request | Invalid parameters or malformed JSON |
| 401 Unauthorized | Invalid credentials or signature |
| 403 Forbidden | Account/permission issue |
| 404 Not Found | Resource doesn't exist |
| 429 Too Many Requests | Rate limited; implement backoff |
| 500+ Server Error | iKhokha API error; retry with backoff |

### Handling Failures

**Payment Link Creation Fails:**
- Validate request body (required fields, correct types)
- Verify signature computation (AppSecret correct?)
- Check account status (active merchant? sufficient balance?)
- Retry with exponential backoff for 500+ errors

**Payment Link Marked as Paid But Not Confirmed:**
- Query payment link status: `GET /api/paymentLink/{paylinkID}`
- Compare with your order database
- For discrepancies, contact iKhokha support with transaction ID

**Hardware Payment Processing Fails (SDK):**
- Check Bluetooth connection to card reader
- Verify card reader is powered and paired
- Retry payment (typically soft decline, customer can retry)
- For persistent failures, fall back to payment link alternative

## Important Implementation Notes

### Security & Compliance

1. **PCI DSS Compliance:** iKhokha handles card data; minimize your exposure by using payment links or SDK-managed card processing, not manual card entry.

2. **Signature Verification:** Always compute and verify HMAC-SHA256 signatures. Never trust unsigned API responses. Use your AppSecret as the HMAC key.

3. **Webhook Validation:** Always verify incoming webhooks using the `ik-sign` header before processing. Prevent webhook spoofing attacks.

4. **Credentials Management:** Never commit API credentials to Git. Use environment variables, AWS Secrets Manager, or secure vaults.

5. **HTTPS Only:** All API communication must use HTTPS (TLS 1.2+). Reject HTTP requests.

### Development Best Practices

1. **Idempotency:** Use `externalTransactionID` to ensure idempotent requests. iKhokha can deduplicate based on this field if you retry.

2. **Amount Precision:** Always use **integers for cents** (ZAR). Never use floating-point arithmetic. R199.99 = 19999 cents.

3. **Timeout Handling:** Network timeouts are possible. Implement retry logic with exponential backoff (e.g., retry after 1s, 2s, 4s, 8s...).

4. **Status Polling:** For critical payments, poll the payment link status every few seconds rather than relying solely on webhooks.

5. **Testing:** iKhokha test environment uses same API endpoint with test credentials. Test card numbers:
   - Success: `4111111111111111`
   - Decline: `4000000000000002`
   - 3D Secure: `4012888888881881`

### Hardware Integration Notes

1. **Bluetooth Pairing:** Ensure card reader is powered, in pairing mode, and within range before app startup.

2. **Android Versions:** iK Tap on Phone (NFC-based card acceptance) requires Android 9.0+. Hardware readers support Android 5.0+.

3. **Network Dependency:** Card processing may work offline (chip reading) but settlement requires internet connectivity.

4. **Receipt Printing:** Integrate with compatible Bluetooth or USB receipt printers for customer receipts.

5. **Merchant Fees:** iKhokha charges per transaction (2% Instant EFT, 2.85% local cards, 3.25% international cards). Fees vary; check Merchant Dashboard for your rates.

## Settlement & Payouts

- **Settlement Frequency:** Daily (typically EOD South African time)
- **Payout Timing:** 1-2 business days to your linked bank account
- **Settlement Calculation:** Total transactions minus iKhokha fees and chargebacks
- **Minimum Payout:** Some tiers have minimum amounts (check your agreement)
- **Bank Details:** Configure payout bank account in Merchant Dashboard → Settings

## Currencies & Localization

- **Currency:** ZAR (South African Rand)
- **Locale:** South Africa (payment methods optimized for SA: Instant EFT, FNB, Standard Bank, Capitec, etc.)
- **Mobile:** Android (primary), iOS (via iK Pay API / payment links)

## Useful Links

- **Official Website:** https://www.ikhokha.com/
- **iK Pay API Documentation:** https://developer.ikhokha.com/
- **Merchant Dashboard:** https://merchant.ikhokha.com/
- **Help Centre:** https://help.ikhokha.com/
- **Status Page:** https://status.ikhokha.com/
- **API Examples (GitHub):** https://github.com/ikhokha/ik-pay-api-examples
- **Blog & Guides:** https://www.ikhokha.com/blog/
- **Support Email:** support@ikhokha.com
- **Postman API Collection:** https://www.postman.com/restless-star-159788/workspace/ikhokha/overview

## Key Differentiators

- **Mobile-first:** Designed for small business owners, field sales, delivery agents
- **Hardware-included:** Provides certified card readers (not just API)
- **Local:** Supports South African payment methods and banking partners
- **Simple onboarding:** Minimal KYC, fast merchant activation
- **Developer friendly:** Postman collection, code examples, active support

---

**Last Updated:** February 2025
**Research Sources:** [iKhokha Official Site](https://www.ikhokha.com/), [Developer Portal](https://developer.ikhokha.com/), [Help Centre](https://help.ikhokha.com/), [GitHub Examples](https://github.com/ikhokha/)
