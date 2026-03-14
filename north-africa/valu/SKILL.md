---
name: valu
description: "Integrate with ValU BNPL (Buy Now Pay Later) for Egyptian installment payments. Use this skill when building checkout flows for Egypt, offering installment plans through payment aggregators like Paymob, PayTabs or OPay, implementing BNPL payment methods, handling installment eligibility checks, or managing customer financing options. Trigger on mentions of 'ValU', 'installments in Egypt', 'BNPL Egypt', 'interest-free payments', or Egyptian fintech solutions."
---

# ValU Integration Skill

ValU is Egypt's leading Buy Now Pay Later (BNPL) platform, enabling customers to purchase now and pay in installments interest-free. With millions of active users, ValU is essential for e-commerce platforms serving the Egyptian market. ValU does not offer a direct merchant API; instead, it integrates through payment aggregators and gateways like Paymob, PayTabs, OPay, and Geidea.

## When to use this skill

You're building an e-commerce store, digital services platform, or marketplace in Egypt and want to offer customers flexible payment options through installments. ValU provides the financing and approval infrastructure; you integrate through a payment aggregator that handles the technical integration. ValU covers customer verification, credit evaluation, fraud detection, and installment scheduling — you manage the checkout flow.

## Integration Overview

ValU is **not integrated directly** but through payment service providers (aggregators):

- **Paymob Accept** - Most popular in Egypt; integrates ValU as an alternative payment method (APM)
- **PayTabs** - Offers ValU as hosted payment page option
- **OPay Checkout** - Supports ValU installments with dedicated API endpoints
- **Geidea** - Provides ValU integration documentation

Choose your aggregator based on your existing payment infrastructure. Most Egyptian e-commerce platforms use Paymob or PayTabs.

## Integration Path: Via Paymob Accept

Paymob is the recommended integration path for most merchants. ValU is available as a BNPL payment method in Paymob's checkout.

### 1. Setup and Registration

1. Create a Paymob merchant account: https://paymob.com/
2. Enable ValU as a payment method (contact Paymob support if needed)
3. Get your Paymob API key and integration secrets
4. Store credentials in environment variables:
   ```
   PAYMOB_API_KEY=pk_live_xxxxx
   PAYMOB_SECRET=sk_live_xxxxx
   ```

### 2. Core Integration Flow

The typical BNPL checkout flow with Paymob:

```
1. Customer selects ValU at checkout
2. System calls Paymob API to get available installment plans
3. Customer selects installment plan
4. System initiates payment with Paymob
5. Customer completes KYC/OTP verification at ValU
6. Webhook confirms payment status
7. Fulfill order
```

### 3. Paymob Checkout Integration

**Initialize Payment Order:**

```bash
curl -X POST https://accept.paymob.com/api/ecommerce/orders \
  -H "Content-Type: application/json" \
  -d '{
    "auth_token": "YOUR_AUTH_TOKEN",
    "delivery_needed": false,
    "currency": "EGP",
    "amount_cents": 500000,
    "merchant_order_id": "ORD-2025-12345",
    "items": [
      {
        "name": "Laptop",
        "amount_cents": 300000,
        "quantity": 1,
        "description": "Electronics"
      },
      {
        "name": "Charger",
        "amount_cents": 200000,
        "quantity": 1,
        "description": "Accessories"
      }
    ],
    "description": "Order for electronics and accessories",
    "customer_phone": "+201234567890",
    "customer_email": "customer@email.com"
  }'
```

**Response:**
```json
{
  "id": 123456789,
  "pending": false,
  "amount_cents": 500000,
  "currency": "EGP",
  "merchant_order_id": "ORD-2025-12345",
  "status": "PENDING",
  "items": [...],
  "created_at": "2025-02-24T10:00:00Z"
}
```

**Get Authentication Token:**

```bash
curl -X POST https://accept.paymob.com/api/auth/tokens \
  -H "Content-Type: application/json" \
  -d '{
    "username": "YOUR_MERCHANT_API_KEY",
    "password": "YOUR_MERCHANT_PASSWORD"
  }'
```

**Create Payment Intent with ValU:**

```bash
curl -X POST https://accept.paymob.com/api/ecommerce/integrations/123456/iframe \
  -H "Content-Type: application/json" \
  -d '{
    "auth_token": "AUTH_TOKEN_FROM_PREVIOUS_CALL",
    "amount_cents": 500000,
    "order_id": 123456789,
    "billing_data": {
      "first_name": "Ahmed",
      "last_name": "Hassan",
      "email": "customer@email.com",
      "phone_number": "+201234567890",
      "state": "Cairo",
      "country": "EG",
      "city": "Cairo",
      "postal_code": "11511",
      "street": "123 Main St"
    },
    "integrations": [INTEGRATION_ID_FOR_VALU]
  }'
```

**Paymob Flash Checkout (JavaScript):**

```javascript
// Mount ValU payment option in your checkout page
const paymobCheckout = new Paymob({
  publicKey: 'pk_live_xxxxx',
  onApprove: (data) => {
    console.log('Payment approved:', data);
    // Confirm order on server
  },
  onReject: (data) => {
    console.log('Payment rejected:', data);
  }
});

// Render ValU as payment method
paymobCheckout.renderPaymentMethod({
  method: 'valu',
  amount: 500000,
  currency: 'EGP',
  orderId: 'ORD-2025-12345',
  customerEmail: 'customer@email.com',
  containerSelector: '#payment-methods'
});
```

## Integration Path: Via PayTabs

PayTabs offers ValU through hosted payment pages (HPP):

**Step 1: Register with ValU via PayTabs**
Contact: customercare-eg@paytabs.com with your Merchant ID

**Step 2: Create Payment Request**

```bash
curl -X POST https://secure.paytabs.com/apiv2/create_pay_page \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'merchant_email=YOUR_EMAIL&merchant_password=YOUR_PASSWORD&currency_code=EGP&amount=500000&title=Electronics%20Order&cc_first_name=Ahmed&cc_last_name=Hassan&cc_phone_number=+201234567890&cc_email=customer@email.com&reference_no=ORD-2025-12345&return_url=https://yoursite.com/checkout/success&cms_with_version=ValU&cms_module_version=2.0'
```

**Step 3: Customer Flow at PayTabs**

1. Customer sees payment method selection screen
2. Selects "ValU" option
3. Enters phone number registered with ValU
4. Views available installment plans
5. Selects plan (if any down payment required, pays that separately)
6. Completes OTP verification
7. Redirected back to your `return_url`

## Integration Path: Via OPay

OPay provides a dedicated API for ValU installments:

**Get Installment Plans:**

```bash
curl -X POST https://api.opaycheckout.com/api/v1/payment/action/input-installment-month \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 5000,
    "currency": "EGP",
    "method": "VALU"
  }'
```

**Response:**
```json
{
  "success": true,
  "data": {
    "plans": [
      {
        "months": 3,
        "monthlyAmount": 1667,
        "totalAmount": 5000,
        "downPayment": 0
      },
      {
        "months": 6,
        "monthlyAmount": 834,
        "totalAmount": 5000,
        "downPayment": 0
      },
      {
        "months": 12,
        "monthlyAmount": 417,
        "totalAmount": 5000,
        "downPayment": 0
      }
    ]
  }
}
```

## Installment Plan Structure

ValU offers flexible installment options. Typical configurations:

```json
{
  "planId": "plan_xxxxx",
  "numberOfInstallments": 4,
  "downPayment": 0,
  "monthlyInstallment": 1250,
  "totalAmount": 5000,
  "currency": "EGP",
  "interestRate": 0,
  "firstDueDate": "2025-03-24",
  "frequency": "MONTHLY",
  "eligibilityRange": {
    "minAmount": 500,
    "maxAmount": 100000,
    "maxTenure": 60
  }
}
```

**Key Details:**
- Interest-free installments (ValU's business model relies on merchant fees/revenue share)
- Down payment may vary; some plans require 0% down payment
- Installments are typically monthly
- Maximum tenure: typically 60 months for high-value items
- Minimum amount: usually 500 EGP

## Eligibility Check

ValU evaluates customer eligibility based on:

- Phone number registered with ValU
- Credit history and past payment behavior
- Transaction amount
- Customer's approved credit limit with ValU

**Not all customers qualify for all plan amounts.** If a customer is ineligible, they'll see limited or no plans. You cannot directly check eligibility via API; customers discover eligibility during checkout.

## Webhooks and Callbacks

Payment aggregators send webhooks for transaction status updates:

**Paymob Webhook Example:**

```json
{
  "obj": {
    "id": 123456789,
    "pending": false,
    "amount_cents": 500000,
    "currency": "EGP",
    "merchant_order_id": "ORD-2025-12345",
    "status": "SUCCESS",
    "transaction": {
      "id": 987654321,
      "pending": false,
      "success": true,
      "amount_cents": 500000,
      "currency": "EGP",
      "gateway_integration_id": 123456,
      "method_integration_id": 123456,
      "data": {
        "message": "Approved",
        "phone": "+201234567890",
        "amount": "50.00"
      }
    }
  }
}
```

**Verify webhook signature:**

```javascript
const crypto = require('crypto');
const signature = crypto
  .createHmac('sha256', YOUR_WEBHOOK_SECRET)
  .update(JSON.stringify(webhookBody))
  .digest('hex');

if (signature === req.headers['x-hmac-sha256']) {
  // Valid webhook
}
```

**Listen for these events:**
- `success` - Payment and installment plan confirmed
- `failure` - Customer rejected or eligibility check failed
- `pending` - Transaction awaiting confirmation

## Common Integration Patterns

### E-commerce Checkout with ValU

```
1. Customer adds items to cart
2. At checkout, show available payment methods (include ValU)
3. IF customer selects ValU:
   a. Optionally fetch installment plans for display
   b. Redirect to aggregator checkout (Paymob/PayTabs/OPay)
   c. Customer completes KYC and plan selection
   d. Webhook confirms order status
4. Fulfill order when status = "success" or "approved"
```

### Display Installment Options Early

```
1. On product page, show price
2. Fetch installment plans from aggregator (optional)
3. Display "Pay EGP 1,250/month for 4 months" badge
4. Let customer add to cart and proceed to checkout
5. Real eligibility determined at checkout (depends on customer's ValU profile)
```

### Handle Delinquent Payments

```
1. Aggregator sends webhook when installment is due
2. Display reminder to customer in your app
3. Link customer to ValU app to complete payment
4. Receive webhook when payment is made
5. If customer defaults, aggregator notifies you
6. Escalate if needed (suspend account, restrict future BNPL, etc.)
```

## Error Handling

Common errors and how to handle them:

```json
{
  "error": true,
  "message": "Customer not eligible for ValU",
  "code": "VALU_INELIGIBLE"
}
```

**Common Errors:**

| Error Code | Meaning | Action |
|---|---|---|
| `VALU_INELIGIBLE` | Customer doesn't qualify for ValU | Offer alternative payment methods |
| `INSUFFICIENT_CREDIT` | Customer has hit credit limit | Reduce amount or suggest fewer installments |
| `INVALID_PHONE` | Phone not registered with ValU | Ask customer to register first |
| `OTP_FAILED` | Customer failed OTP verification | Allow retry |
| `TIMEOUT` | Checkout session expired | Create new checkout session |
| `AMOUNT_BELOW_MIN` | Amount < 500 EGP | Minimum is usually 500 EGP |
| `AMOUNT_EXCEEDS_MAX` | Amount exceeds customer's limit | Suggest different amount |

**HTTP Status Codes:**
- `200 OK` - Success
- `400 Bad Request` - Validation error
- `401 Unauthorized` - Invalid credentials
- `404 Not Found` - Order or plan not found
- `422 Unprocessable Entity` - Business logic error (ineligible, exceeded credit, etc.)
- `500 Server Error` - Retry with exponential backoff

## Amount Format

Amount units **depend on the aggregator**:

- **Paymob** — amounts are in **piasters** (`amount_cents`). 1 EGP = 100 piasters. EGP 50 = 5000.
- **OPay** — verify with OPay docs; examples show whole-unit amounts.
- **PayTabs** — verify with PayTabs docs; typically whole currency units.

Always check your chosen aggregator's documentation. Do not assume a single unit applies across all paths.

## Important Notes / Gotchas

**Direct API:**
- ValU does **not** offer a direct merchant API
- All integrations go through payment aggregators (Paymob, PayTabs, OPay, Geidea)
- You must use one of these aggregators; you cannot call ValU directly

**Authentication:**
- Each aggregator has its own authentication (API keys, OAuth, etc.)
- Never hardcode credentials; use environment variables
- Rotate API keys regularly

**Customer Eligibility:**
- Eligibility is determined **at checkout**, not before
- You cannot pre-check if a customer is eligible for ValU
- Some customers may have limited or no eligible plans
- Always provide fallback payment methods

**Installment Timing:**
- First installment (down payment) may be due immediately or at delivery
- Subsequent installments follow monthly schedule
- Some plans allow 0% down payment; others require a down payment
- Aggregator can configure this per merchant

**Minimum and Maximum Amounts:**
- Minimum amount: typically 500 EGP (varies by aggregator)
- Maximum amount: depends on customer's ValU credit limit (usually 100,000+ EGP)
- Not all amounts are available for all customers

**Interest-Free Claims:**
- ValU advertises interest-free installments
- There are no interest charges to the customer
- Merchants typically pay a revenue share or transaction fee to ValU (usually 2-5%)
- This is handled between merchant and aggregator/ValU

**Egyptian Market Specifics:**
- Phone numbers must include `+20` country code
- All amounts in Egyptian Pounds (EGP)
- KYC verification required for all ValU transactions
- OTP (One-Time Password) is standard for confirmation
- National ID verification may be required for new ValU users

**Aggregator Choice Matters:**
- Different aggregators have different APIs, webhook formats, and features
- Paymob is most feature-rich and widely used in Egypt
- PayTabs is good for traditional hosted payment page flows
- OPay has dedicated installment endpoints
- Choose based on your existing payment infrastructure

**Checkout Expiration:**
- Checkout sessions expire (typically 1 hour)
- If customer abandons checkout, create a new session
- Expired sessions cannot be reused

**Merchant Registration:**
- You (the merchant) must be registered with ValU through your chosen aggregator
- Aggregator handles the backend ValU merchant registration
- Contact aggregator support if ValU is not activated for your account

**No Recurring Payments:**
- ValU is for one-time purchases with installment payment
- Not suitable for subscriptions or recurring charges
- Each purchase is a new standalone ValU transaction

## Useful Links

- **Paymob Developer Portal:** https://developers.paymob.com/
- **Paymob Egypt Hub:** https://developers.paymob.com/hub/egypt
- **Paymob Flash Checkout GitHub:** https://github.com/PaymobAccept/paymob-js
- **PayTabs Support (ValU Integration):** https://support.paytabs.com/
- **OPay Documentation:** https://doc.opaycheckout.com/
- **Geidea ValU Docs:** https://docs.geidea.net/docs/valu
- **ValU Official Website:** https://www.valu.com.eg
- **Paymob Installments:** https://paymob.com/en/installments
- **ValU LinkedIn (News & Updates):** https://www.linkedin.com/company/valuegypt

## Recommended Aggregator Selection

| Aggregator | Best For | API Type | Support |
|---|---|---|---|
| **Paymob** | Most Egyptian merchants, feature-rich | REST + Webhooks | Excellent |
| **PayTabs** | Hosted page preference, in-store payments | Hosted page + API | Good |
| **OPay** | Dedicated installment flows | REST with installment endpoints | Good |
| **Geidea** | Enterprise merchants | REST + Advanced | Enterprise support |

Start with **Paymob** if you're new to Egyptian BNPL integration.
