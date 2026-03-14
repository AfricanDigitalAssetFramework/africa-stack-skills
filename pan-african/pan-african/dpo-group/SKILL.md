---
name: dpo-group
description: "Integrate with DPO Group (now Network International) XML API for pan-African payment processing. Use when building payment solutions accepting cards, mobile money, or bank transfers across Africa, implementing token-based recurring billing, processing secure transactions across multiple African countries, or handling multi-currency payments. Trigger on: 'DPO Group', 'Network International', 'DPO Pay', '3G Direct Pay', 'pan-African payment', 'African payment gateway', 'secure payment token', 'mobile money Africa', 'recurring payments Africa'."
---

# DPO Group Integration Skill

DPO Group, now rebranded as Network International, is a leading pan-African payment gateway enabling businesses to securely accept payments via credit cards, debit cards, mobile money (M-Pesa, MTN Mobile Money, Airtel Money), USSD, bank transfers, and digital wallets across 21+ African countries. The API uses XML-based request/response format for secure token-based payment processing.

## When to use this skill

You're building a payment solution that needs to accept diverse payment methods across Africa: an e-commerce platform processing multiple payment types, a service provider collecting subscriptions with recurring billing, a business expanding to multiple African markets with different payment preferences, or a platform handling tokenized payments with compliance requirements. DPO Group abstracts away the complexity of diverse payment methods, regulatory requirements, and settlement mechanisms across different African countries.

## Authentication

DPO Group uses Company Token authentication. Your merchant account provides a unique `CompanyToken` (a UUID format identifier) that authenticates all API requests.

**Base URL (V6):** `https://secure.3gdirectpay.com/API/v6/`

> Note: V7 of the API is available (see Useful Links). Verify with DPO/Network International whether V7 changes any endpoint paths or request formats before upgrading.

**Authentication Method:**
Include your CompanyToken in the XML request body. Store the token in an environment variable like `DPO_COMPANY_TOKEN` and never hardcode credentials.

Example token format: `57466282-EBD7-4ED5-B699-8659330A6996`

**Service Type:**
When setting up your account, DPO assigns Service Type codes for each product/service category. Common examples:
- Service Type "101": Card payments
- Service Type "102": Mobile money
- Service Type "103": Bank transfers

Include the appropriate ServiceType in your Services section when making API requests.

## Core API Reference

**Important:** DPO Group uses XML format for all API requests and responses, not JSON. Ensure proper XML parsing in your implementation.

### Create Payment Token (createToken V6)

Tokenize a payment transaction to securely initiate payment processing. This is the primary entry point for the DPO payment flow.

**Endpoint:** `POST /createToken`

**Request (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>createToken</Request>
  <Transaction>
    <PaymentAmount>5000</PaymentAmount>
    <PaymentCurrency>USD</PaymentCurrency>
    <CompanyRef>your_company_ref_001</CompanyRef>
    <RedirectURL>https://yoursite.com/payment/callback</RedirectURL>
    <BackURL>https://yoursite.com/checkout</BackURL>
    <CompanyRefUnique>0</CompanyRefUnique>
    <PTL>5</PTL>
    <PTLType>minutes</PTLType>
    <customerFirstName>Amina</customerFirstName>
    <customerLastName>Okafor</customerLastName>
    <customerEmail>amina@example.com</customerEmail>
    <customerPhone>+234801234567</customerPhone>
    <customerAddress>123 Main Street</customerAddress>
    <customerCity>Lagos</customerCity>
    <customerCountry>NG</customerCountry>
  </Transaction>
  <Services>
    <Service>
      <ServiceType>101</ServiceType>
      <ServiceDescription>Product purchase</ServiceDescription>
      <ServiceDate>2025/02/24</ServiceDate>
    </Service>
  </Services>
</API3G>
```

**Response (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>createToken</Request>
  <Result>000</Result>
  <ResultExplanation>Transaction created</ResultExplanation>
  <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  <TransRef>20250224001</TransRef>
  <Eci>0</Eci>
</API3G>
```

**Key Fields:**
- `PaymentAmount`: Transaction amount — **DPO uses major currency units**, not cents/minor units. Send `50.00` for 50 USD, not `5000`. ⚠️ The original note saying "in cents" is incorrect. Confirm with DPO's `verifyToken` response to validate the amount interpretation in your integration before going live.- `PaymentCurrency`: ISO 4217 code (USD, GHS, KES, NGN, ZAR, etc.)
- `CompanyRef`: Your internal transaction reference
- `RedirectURL`: Customer redirected here after payment
- `BackURL`: Fallback URL if customer cancels
- `PTL`: Payment Timeout Limit in minutes (default 5)
- `TransToken`: Use this to redirect customer to payment page

**Redirect Customer to Payment Page:**
```
https://secure.3gdirectpay.com/payv2.php?ID={TransToken}
```

Customer enters payment details on DPO's PCI-compliant hosted page, then is redirected back to your `RedirectURL` with transaction results.

### Verify Token Status (verifyToken V6)

Check the payment status of a created token server-side to confirm successful payment.

**Endpoint:** `POST /verifyToken`

**Request (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>verifyToken</Request>
  <Transaction>
    <PaymentAmount>5000</PaymentAmount>
    <PaymentCurrency>USD</PaymentCurrency>
    <CompanyRef>your_company_ref_001</CompanyRef>
    <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  </Transaction>
</API3G>
```

**Response (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>verifyToken</Request>
  <Result>000</Result>
  <ResultExplanation>Transaction Paid</ResultExplanation>
  <TransactionApproved>1</TransactionApproved>
  <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  <TransRef>20250224001</TransRef>
  <PaymentAmount>5000</PaymentAmount>
  <PaymentCurrency>USD</PaymentCurrency>
  <CardType>Visa</CardType>
  <CardNumber>****5647</CardNumber>
  <Eci>0</Eci>
</API3G>
```

**Key Fields:**
- `Result`: "000" = payment successful, other codes indicate failure
- `TransactionApproved`: "1" = approved, "0" = declined
- `CardType`: Type of card used (Visa, Mastercard, etc.)
- `CardNumber`: Last 4 digits (masked for security)

Always verify token server-side after customer returns from payment page. Do not rely solely on client-side callbacks.

### Charge Token Credit Card (chargeTokenCreditCard V6)

Process a credit card payment using previously tokenized card details for recurring billing or direct charging.

**Endpoint:** `POST /chargeTokenCreditCard`

**Request (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>chargeTokenCreditCard</Request>
  <Transaction>
    <PaymentAmount>5000</PaymentAmount>
    <PaymentCurrency>USD</PaymentCurrency>
    <CompanyRef>your_company_ref_002</CompanyRef>
    <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
    <CardCVV>123</CardCVV>
  </Transaction>
  <Services>
    <Service>
      <ServiceType>101</ServiceType>
      <ServiceDescription>Monthly subscription</ServiceDescription>
      <ServiceDate>2025/02/24</ServiceDate>
    </Service>
  </Services>
</API3G>
```

**Response (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>chargeTokenCreditCard</Request>
  <Result>000</Result>
  <ResultExplanation>Transaction Charged</ResultExplanation>
  <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  <TransRef>20250224002</TransRef>
  <PaymentAmount>5000</PaymentAmount>
  <PaymentCurrency>USD</PaymentCurrency>
  <Eci>0</Eci>
</API3G>
```

**Important PCI Considerations:**
- Do not store full card numbers — use tokens provided by DPO
- Do not transmit raw card data directly — DPO's hosted payment page handles this
- Ensure all connections use HTTPS/TLS
- Maintain PCI DSS compliance for your integration

### Charge Token Mobile (chargeTokenMobile V6)

Process mobile money payments (M-Pesa, MTN Money, Airtel Money, etc.) using a tokenized transaction.

**Endpoint:** `POST /chargeTokenMobile`

**Request (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>chargeTokenMobile</Request>
  <Transaction>
    <PaymentAmount>500</PaymentAmount>
    <PaymentCurrency>KES</PaymentCurrency>
    <CompanyRef>your_company_ref_003</CompanyRef>
    <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
    <MobileNumber>254701234567</MobileNumber>
  </Transaction>
  <Services>
    <Service>
      <ServiceType>102</ServiceType>
      <ServiceDescription>Mobile money payment</ServiceDescription>
      <ServiceDate>2025/02/24</ServiceDate>
    </Service>
  </Services>
</API3G>
```

**Response (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>chargeTokenMobile</Request>
  <Result>000</Result>
  <ResultExplanation>Transaction Charged</ResultExplanation>
  <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  <TransRef>20250224003</TransRef>
  <MobileStatus>pending_confirmation</MobileStatus>
  <PaymentAmount>500</PaymentAmount>
  <PaymentCurrency>KES</PaymentCurrency>
</API3G>
```

**Mobile Money Considerations:**
- Customer receives USSD/SMS prompt to confirm payment
- `MobileStatus: pending_confirmation` indicates awaiting customer action
- Monitor webhooks for status updates
- Timeout varies by mobile operator (typically 2-5 minutes)

### Refund Token

Issue a refund for a previously processed and settled transaction.

**Endpoint:** `POST /refundToken`

**Request (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>refundToken</Request>
  <Transaction>
    <PaymentAmount>2500</PaymentAmount>
    <PaymentCurrency>USD</PaymentCurrency>
    <CompanyRef>refund_ref_001</CompanyRef>
    <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  </Transaction>
</API3G>
```

**Response (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>refundToken</Request>
  <Result>000</Result>
  <ResultExplanation>Transaction Refunded</ResultExplanation>
  <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  <TransRef>refund_ref_001</TransRef>
</API3G>
```

**Refund Processing:**
- Partial refunds supported (specify amount less than original)
- Full refunds supported (use original amount)
- Refunds process within 1-3 business days
- Refund status tracked via verifyToken
- Some payment methods may have longer refund windows

### Charge Token Bank Transfer (chargeTokenBankTransfer V6)

Process bank transfer payments for direct account-to-account transfers.

**Endpoint:** `POST /chargeTokenBankTransfer`

**Request (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>chargeTokenBankTransfer</Request>
  <Transaction>
    <PaymentAmount>10000</PaymentAmount>
    <PaymentCurrency>NGN</PaymentCurrency>
    <CompanyRef>bank_transfer_001</CompanyRef>
    <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  </Transaction>
</API3G>
```

**Response (XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<API3G>
  <CompanyToken>57466282-EBD7-4ED5-B699-8659330A6996</CompanyToken>
  <Request>chargeTokenBankTransfer</Request>
  <Result>000</Result>
  <ResultExplanation>Transaction Charged</ResultExplanation>
  <TransToken>D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B</TransToken>
  <TransRef>bank_transfer_001</TransRef>
</API3G>
```

**Bank Transfer Considerations:**
- Slower settlement (typically 2-5 business days)
- Lower transaction fees for large amounts
- Suitable for B2B and large e-commerce transactions
- Monitor for bank-specific confirmations

## Webhooks

DPO Group sends redirect responses to your `RedirectURL` and `BackURL` after payment completion. Instead of traditional webhooks, DPO uses HTTP redirects with query parameters appended to your URLs.

**Redirect Parameters (Query String):**
```
RedirectURL?TransToken={TransToken}&Reference={CompanyRef}&Result={Result}&ResultExplanation={ResultExplanation}&ECI={ECI}
```

**Example:**
```
https://yoursite.com/payment/callback?TransToken=D3435F63-3E7F-4A50-B6F4-4F8E9C2D5A1B&Reference=your_company_ref_001&Result=000&ResultExplanation=Transaction%20Paid
```

**Verification Steps:**
1. Customer returns from DPO payment page
2. Extract `TransToken` from URL parameters
3. Immediately call `verifyToken` API server-side
4. Confirm `Result=000` and `TransactionApproved=1`
5. Only after server-side verification, fulfill the order

**Important:** Never trust client-side redirect parameters alone. Always verify with verifyToken API from your backend to prevent fraud.

**BackURL Behavior:**
Customer is redirected to `BackURL` if they click "Back to Merchant" or payment times out. This is not a success indication — verify the actual payment status via `verifyToken` before processing.

## Common Integration Patterns

### Card Payment Flow
1. User initiates checkout on your site
2. Call `createToken` with transaction details
3. Receive `TransToken` in response
4. Redirect customer to: `https://secure.3gdirectpay.com/payv2.php?ID={TransToken}`
5. Customer enters card details on DPO's secure payment page
6. Customer automatically redirected to your `RedirectURL`
7. Extract `TransToken` from redirect URL
8. Call `verifyToken` server-side to confirm payment
9. Upon verification, fulfill order and send confirmation

### Recurring Billing
1. Initial payment: follow card payment flow above
2. DPO tokenizes the card during first transaction
3. For future recurring charges:
   - Call `chargeTokenCreditCard` with the stored `TransToken`
   - Include `CardCVV` for security (customer provides once during signup)
4. Monitor results for declined charges
5. Implement retry logic with exponential backoff for failed charges
6. Send customer receipts and payment confirmations
7. Track failed payments and notify customer

### Mobile Money (M-Pesa, MTN, Airtel)
1. Call `createToken` with transaction details
2. Redirect customer to DPO payment page
3. Customer selects mobile money option
4. Customer enters phone number
5. Receives USSD/SMS prompt on their phone
6. Customer confirms payment on their phone
7. DPO redirects back to your `RedirectURL`
8. Call `verifyToken` to confirm payment
9. Monitor `MobileStatus` for confirmation states:
   - `pending_confirmation`: Awaiting customer action
   - `completed`: Successfully charged
   - `failed`: Customer declined or timeout

### Partial Refunds
1. After successful `chargeTokenCreditCard` or card payment
2. Customer requests refund of partial amount
3. Call `refundToken` with amount less than original
4. DPO processes partial refund
5. Credit appears in customer account within 1-3 business days

### Bank Transfer for B2B
1. Call `createToken` for larger B2B transaction
2. Redirect to DPO payment page
3. Payer selects bank transfer option
4. Receives bank account details to transfer funds
5. Payer completes transfer from their bank
6. DPO confirms receipt (may take 2-5 business days)
7. Call `verifyToken` to check settlement status

## Error Handling

DPO Group returns consistent XML responses with Result and ResultExplanation fields. Always check the Result code before processing.

**Standard Response Structure:**
```xml
<API3G>
  <Result>000</Result>
  <ResultExplanation>Description of result</ResultExplanation>
  <TransToken>...</TransToken>
  <TransRef>...</TransRef>
</API3G>
```

**Common Result Codes:**

| Code | Meaning | Action |
|------|---------|--------|
| 000 | Success | Transaction approved, proceed with fulfillment |
| 001 | Unknown error | Log details, contact DPO support |
| 002 | Invalid CompanyToken | Verify token is correct and active |
| 003 | Invalid Request type | Check Request field spelling |
| 005 | Invalid PaymentAmount | Verify amount is positive integer in cents |
| 006 | Invalid PaymentCurrency | Use valid ISO 4217 codes (USD, KES, NGN, etc.) |
| 007 | Invalid CompanyRef | Ensure CompanyRef is provided and unique |
| 008 | Invalid TransToken | Token may have expired (check PTL timeout) |
| 009 | Transaction already processed | Duplicate transaction detected, use unique CompanyRef |
| 012 | Token not found | TransToken doesn't exist or has expired |
| 027 | Service Type not configured | Configure ServiceType in merchant account |
| 100 | Card declined | Customer's bank declined the transaction |
| 101 | Insufficient funds | Customer doesn't have enough balance |
| 102 | Invalid card | Card number failed validation checks |
| 103 | Expired card | Card expiry date has passed |
| 200 | Transaction already paid | Token was already charged successfully |
| 999 | Generic error | Check ResultExplanation for details, retry with backoff |

**Error Handling Best Practices:**
```
- Log all error responses with timestamp and CompanyRef
- Implement exponential backoff retry for Result=999
- Never retry on Result=100, 101, 102, 103 (customer-side issues)
- Contact DPO support for Result=002, 003, 027 (configuration issues)
- For production, monitor error rates and set up alerts
- Provide clear error messages to customers without exposing internal codes
```

## Important Notes & Gotchas

### XML Format Requirement
- DPO Group API uses **XML only**, not JSON
- Ensure proper XML encoding and declarations
- Use robust XML parsers to avoid injection attacks
- Escape special characters: `<`, `>`, `&`, `"`, `'`
- Validate XML schema before submitting requests

### Payment Timeout Limit (PTL)
- Set in `createToken` request (default 5 minutes)
- After timeout, token expires and cannot be used
- If customer delays payment, they must start checkout again
- Adjust PTL based on your payment flow requirements
- Minimum: 1 minute, Maximum: typically 120 minutes

### Transaction References
- `CompanyRef`: Your unique transaction ID (must be unique per createToken)
- `TransRef`: DPO's transaction ID (returned in response)
- Store both for reconciliation and support
- Use for idempotency: same CompanyRef = same transaction

### Network International Rebranding
- DPO Group officially rebranded to Network International in 2024-2025
- Rebranding rolled out across Uganda, Kenya, Zambia, Namibia, and other African markets
- API endpoints and functionality remain unchanged
- merchant accounts and credentials remain valid
- Widget/UI branding may reference "Network International" instead of "DPO Group"

### Supported Payment Methods
- **Cards**: Visa, Mastercard, American Express, Diners Club
- **Mobile Money**: M-Pesa (Kenya), MTN Mobile Money, Airtel Money, Vodafone Cash, Tigo Pesa
- **Bank Transfers**: Direct bank account transfers
- **USSD**: Unstructured Supplementary Service Data (Africa)
- **Digital Wallets**: PayPal and regional mobile wallets

### Supported Currencies & Countries
- **Countries**: 21+ African countries including Nigeria, Kenya, Tanzania, Uganda, South Africa, Ghana, Rwanda, Zambia, Namibia, Botswana, Mauritius, Zimbabwe, Malawi, Côte d'Ivoire
- **Currencies**: NGN, GHS, KES, TZS, UGX, RWF, ZAR, USD, ZMW, NAD, BWP, MUR, ZWL, and others
- Check with DPO for current list of supported countries and currencies

### Transaction Limits
- **Minimum**: Varies by country and payment method (typically $1 USD equivalent)
- **Maximum**: Varies by country regulations and card/account limits
- Check your merchant dashboard for current limits
- High-value transactions may require manual approval

### Security Considerations
- Always transmit over HTTPS/TLS
- Never log full card numbers or CVV
- Never hardcode CompanyToken in code
- Use environment variables for credentials
- Validate all redirect URLs to prevent open redirect attacks
- Implement proper CORS headers if API called from browser
- Rotate credentials periodically

### Testing & Sandbox
- DPO provides test CompanyToken and test payment methods
- Use test cards and test phone numbers in sandbox
- Ask DPO support for test credentials and documentation
- Integration can be validated using test merchant account

### Rate Limiting
- DPO implements rate limiting (typically 100+ requests/minute per merchant)
- If you receive Result=429, implement exponential backoff
- Contact DPO support for higher rate limits if needed

### PCI Compliance
- Using DPO's hosted payment page (`payv2.php`) helps meet PCI requirements
- You do NOT need PCI Level 1 if card details never touch your servers
- Keep integration endpoints secure and behind authentication
- Regular security audits recommended

## Useful Links

- **Official Documentation**: [https://docs.dpopay.com/api/index.html](https://docs.dpopay.com/api/index.html)
- **DPO Group Website**: [https://dpogroup.com/](https://dpogroup.com/)
- **Network International**: [https://www.network.ae/](https://www.network.ae/)
- **Payment Methods Guide**: [https://dpogroup.com/payment-methods/](https://dpogroup.com/payment-methods/)
- **Integration Resources**: [https://dpogroup.com/integration/](https://dpogroup.com/integration/)
- **Merchant Dashboard**: [https://dashboard.3gdirectpay.com/](https://dashboard.3gdirectpay.com/)
- **Support & FAQs**: [https://dpogroup.com/faq/](https://dpogroup.com/faq/)
- **GitHub SDK Examples**: [https://github.com/DPO-Group](https://github.com/DPO-Group)
- **createToken API Docs**: [https://directpayonline.atlassian.net/wiki/spaces/API/pages/36110341/createToken+V6](https://directpayonline.atlassian.net/wiki/spaces/API/pages/36110341/createToken+V6)
- **Response Codes**: [https://dpogroup.com/response-codes/](https://dpogroup.com/response-codes/)

---

**Last Updated**: February 2025
**DPO Group/Network International Status**: Actively maintained and rebranding to Network International brand
**API Version**: V6 (Current), V7 (Latest)
