---
name: CinetPay Payment Integration
description: Francophone West African payment aggregator supporting 40+ payment methods including Orange Money, MTN MoMo, Moov Money across 9+ countries with mobile money, cards, and e-commerce plugins.
triggers:
  - CinetPay
  - Francophone Africa payments
  - West Africa payment gateway
  - Orange Money API
  - MTN MoMo Côte d'Ivoire
  - FCFA payments
  - XOF payments
  - CFA franc checkout
  - Mobile money aggregator
  - African fintech
---

# CinetPay Payment Integration

CinetPay is a payment aggregator platform serving Francophone West Africa, enabling merchants to accept payments through 40+ payment methods including mobile money wallets (Orange Money, MTN MoMo, Moov Money, Flooz, T-Money), bank cards (Visa, MasterCard, Maestro, American Express, Union Pay), and e-wallets across 9+ countries. Founded to solve fragmented payment infrastructure, CinetPay acts as a hub consolidating multiple payment channels into a single integration point.

## When to Use This Skill

Use CinetPay for:
- **Mobile Money Payments**: Accepting Orange Money, MTN MoMo, Moov Money, Flooz, T-Money across West Africa
- **Multi-Country Expansion**: Serving Côte d'Ivoire, Benin, Togo, Mali, Congo, Guinea, Niger, Senegal, Cameroon, Burkina Faso
- **Unified Payment Interface**: Single integration to support 40+ payment methods across regions
- **E-commerce Platforms**: Pre-built plugins for WooCommerce, PrestaShop, WHMCS, Moodle
- **FCFA/XOF Transactions**: Native support for West African CFA franc currency
- **Merchant Settlements**: Direct mobile money transfers to merchant accounts
- **Subscription & Billing**: Card-based recurring payments for SaaS/subscriptions

Do **not** use CinetPay if:
- Your customers are in English-speaking Africa (Flutterwave or Paystack may be better)
- You require payment support outside West/Central Africa
- Your primary use case requires real-time payout verification (CinetPay has settlement delays)

## Authentication

### Getting Credentials

1. **Create Account**: Sign up at https://cinetpay.com/
2. **Access Merchant Panel**: Log in at https://app.cinetpay.com/marchand
3. **Retrieve API Key**: Found on Integration page
4. **Create Service**: Subscribe to a service to obtain your `site_id`
5. **Store Securely**: Use environment variables; never commit credentials

### Credentials Structure

- **`apikey`**: Unique 40-character API key (works for test and production)
- **`site_id`**: Numeric ID per service (obtained after subscription)
- Both credentials are required for all API calls

### Environment Setup

```bash
# .env file (test or production environment)
CINETPAY_API_KEY=your_40_character_api_key_here
CINETPAY_SITE_ID=your_numeric_site_id_here
```

### Example: Initial Setup (Python)

```python
import os
from datetime import datetime
import uuid

# Load credentials from environment
api_key = os.getenv('CINETPAY_API_KEY')
site_id = os.getenv('CINETPAY_SITE_ID')

# Verify credentials before first transaction
if not api_key or not site_id:
    raise ValueError("Missing CINETPAY_API_KEY or CINETPAY_SITE_ID environment variables")

print(f"✓ CinetPay authenticated for site_id: {site_id}")
```

## Core API Reference

### Payment Initialization

**Endpoint**: `POST https://api-checkout.cinetpay.com/v2/payment`

**Purpose**: Generate payment link for checkout

**Required Parameters**:

| Parameter | Type | Length | Description |
|-----------|------|--------|-------------|
| `apikey` | string | 40 | Your API key |
| `site_id` | numeric | - | Your site ID |
| `transaction_id` | string | - | Your unique transaction reference (alphanumeric, no special chars) |
| `amount` | numeric | - | Amount in smallest currency unit (e.g., 1000 = 1000 XOF) |
| `currency` | string | 3 | XOF (West African CFA franc) |
| `description` | string | 255 | Payment purpose/item description |
| `notify_url` | URL | - | Your webhook endpoint (must be publicly accessible, POST) |
| `return_url` | URL | - | Customer redirect after payment |

**Customer Details** (required):

```json
{
  "customer": {
    "name": "John",
    "surname": "Doe",
    "email": "john@example.com",
    "phone_number": "+225XXXXXXXXXX",
    "address": "123 Street",
    "city": "Abidjan",
    "state": "Lagunes",
    "country": "CI",
    "zip_code": "01"
  }
}
```

**Optional Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `channels` | string | Payment methods: "ALL", "MOBILE", "CARD" |
| `metadata` | object | Custom key-value pairs for your reference |
| `lang` | string | "FR" or "EN" (payment interface language) |
| `invoice_data` | object | Invoice items array (for receipts) |

**Full Example Request**:

```python
import requests
import json
from datetime import datetime
import uuid

def initialize_payment(amount_xof, customer_email, description, return_url):
    """Initialize a payment request with CinetPay"""

    payload = {
        "apikey": "your_api_key_here",
        "site_id": "your_site_id_here",
        "transaction_id": f"TXN-{datetime.now().strftime('%Y%m%d%H%M%S')}-{str(uuid.uuid4())[:8]}",
        "amount": amount_xof,  # e.g., 5000 for 5000 XOF
        "currency": "XOF",
        "description": description,  # e.g., "Purchase of Premium Subscription"
        "notify_url": "https://yourdomain.com/webhooks/cinetpay",
        "return_url": return_url,
        "channels": "ALL",  # ALL, MOBILE, CARD
        "lang": "FR",  # FR or EN
        "customer": {
            "name": "John",
            "surname": "Doe",
            "email": customer_email,
            "phone_number": "+225XXXXXXXXXX",
            "address": "123 Street",
            "city": "Abidjan",
            "state": "Lagunes",
            "country": "CI",
            "zip_code": "01"
        },
        "metadata": {
            "order_id": "ORD-12345",
            "customer_id": "CUST-789",
            "plan": "pro"
        }
    }

    headers = {"Content-Type": "application/json"}

    try:
        response = requests.post(
            "https://api-checkout.cinetpay.com/v2/payment",
            json=payload,
            headers=headers,
            timeout=10
        )
        response.raise_for_status()
        result = response.json()

        if result.get("code") == "00":
            payment_token = result.get("data", {}).get("token")
            payment_url = result.get("data", {}).get("payment_url")
            print(f"✓ Payment initialized. Token: {payment_token}")
            print(f"✓ Redirect customer to: {payment_url}")
            return {
                "success": True,
                "token": payment_token,
                "url": payment_url,
                "transaction_id": payload["transaction_id"]
            }
        else:
            print(f"✗ Initialization failed: {result.get('message')}")
            return {"success": False, "error": result}

    except requests.exceptions.RequestException as e:
        print(f"✗ API error: {str(e)}")
        return {"success": False, "error": str(e)}

# Usage
result = initialize_payment(
    amount_xof=5000,
    customer_email="customer@example.com",
    description="Premium subscription - Monthly",
    return_url="https://yourdomain.com/payment/success"
)
```

**Success Response** (code: "00"):

```json
{
  "code": "00",
  "message": "SUCCES",
  "description": "Operation successful",
  "data": {
    "token": "abcdef123456",
    "payment_url": "https://checkout.cinetpay.com/?token=abcdef123456"
  }
}
```

**Error Response Example** (code: "609"):

```json
{
  "code": "609",
  "message": "AUTH_NOT_FOUND",
  "description": "apikey does not belong to any merchant"
}
```

---

### Transaction Verification

**Endpoint**: `POST https://api-checkout.cinetpay.com/v2/payment/check`

**Purpose**: Verify payment status (always call this from your webhook, don't rely on client-side callbacks)

**Required Parameters**:

```json
{
  "apikey": "your_api_key",
  "site_id": "your_site_id",
  "transaction_id": "TXN-20250224143022-a1b2c3d4"
}
```

**Verification Example**:

```python
def verify_transaction(transaction_id):
    """Verify a transaction status with CinetPay"""

    payload = {
        "apikey": "your_api_key",
        "site_id": "your_site_id",
        "transaction_id": transaction_id
    }

    headers = {"Content-Type": "application/json"}

    try:
        response = requests.post(
            "https://api-checkout.cinetpay.com/v2/payment/check",
            json=payload,
            headers=headers,
            timeout=10
        )
        response.raise_for_status()
        result = response.json()

        if result.get("code") == "00":
            transaction_data = result.get("data", {})
            status = transaction_data.get("status")

            # Status values: VALIDATED, REFUSED, CANCELLED, etc.
            if status == "VALIDATED":
                print(f"✓ Payment successful for {transaction_data.get('amount')} {transaction_data.get('currency')}")
                print(f"  Method: {transaction_data.get('payment_method')}")
                print(f"  Date: {transaction_data.get('payment_date')}")
            else:
                print(f"✗ Payment status: {status}")

            return {
                "verified": True,
                "status": status,
                "amount": transaction_data.get("amount"),
                "currency": transaction_data.get("currency"),
                "payment_method": transaction_data.get("payment_method"),
                "payment_date": transaction_data.get("payment_date"),
                "reference": transaction_data.get("cpm_trans_id")  # CinetPay's internal ID
            }
        else:
            print(f"✗ Verification failed: {result.get('message')}")
            return {"verified": False, "error": result}

    except requests.exceptions.RequestException as e:
        print(f"✗ API error: {str(e)}")
        return {"verified": False, "error": str(e)}

# Usage after receiving webhook notification
transaction_status = verify_transaction("TXN-20250224143022-a1b2c3d4")
```

**Verification Response**:

```json
{
  "code": "00",
  "message": "SUCCES",
  "data": {
    "cpm_trans_id": "123456789",
    "cpm_currency": "XOF",
    "cpm_amount": "5000",
    "cpm_order_id": "ORDER-12345",
    "cpm_custom": "metadata_here",
    "cpm_trans_date": "2025-02-24 14:30:22",
    "cpm_trans_status": "VALIDATED",
    "payment_method": "ORANGE_MONEY_CI",
    "cpm_phone_prefill": "+225XXXXXXXXXX",
    "cpm_customer_name": "John Doe",
    "cpm_customer_email": "john@example.com",
    "fund_availability_date": "2025-02-25"
  }
}
```

---

## Webhooks / Notification Handling

CinetPay delivers transaction notifications to your `notify_url` when payment status changes.

### Webhook Payload Structure

**Method**: `POST`
**Authentication**: HMAC token in `X-TOKEN` header
**Payload Parameter**: `cpm_trans_id` (transaction ID)

```python
# Your webhook endpoint (Flask example)
from flask import Flask, request
import hmac
import hashlib
import requests
import json

app = Flask(__name__)

@app.route('/webhooks/cinetpay', methods=['POST', 'GET'])
def cinetpay_webhook():
    """Handle CinetPay webhook notifications"""

    # Method 1: Extract transaction ID from request
    transaction_id = request.values.get('cpm_trans_id')

    if not transaction_id:
        print("✗ No transaction ID in webhook")
        return {"status": "error"}, 400

    # Method 2: Verify HMAC token (optional but recommended)
    hmac_token = request.headers.get('X-TOKEN', '')
    if hmac_token:
        # Verify token using your API key as secret
        # (Token verification depends on CinetPay's exact implementation)
        print(f"✓ HMAC Token received: {hmac_token}")

    # Method 3: Always verify transaction status with CinetPay API
    # This is CRITICAL—don't trust the webhook data alone
    verification = verify_transaction_with_cinetpay(transaction_id)

    if verification.get("verified") and verification.get("status") == "VALIDATED":
        # Update your database
        update_order_status(transaction_id, "paid")
        return {"status": "success"}, 200
    else:
        # Log failed payment
        update_order_status(transaction_id, "failed")
        return {"status": "failed"}, 200  # Return 200 anyway so CinetPay knows we received it

def verify_transaction_with_cinetpay(transaction_id):
    """Verify the transaction directly with CinetPay API"""
    payload = {
        "apikey": os.getenv('CINETPAY_API_KEY'),
        "site_id": os.getenv('CINETPAY_SITE_ID'),
        "transaction_id": transaction_id
    }

    response = requests.post(
        "https://api-checkout.cinetpay.com/v2/payment/check",
        json=payload,
        headers={"Content-Type": "application/json"},
        timeout=10
    )

    result = response.json()
    if result.get("code") == "00":
        data = result.get("data", {})
        return {
            "verified": True,
            "status": data.get("cpm_trans_status"),
            "amount": data.get("cpm_amount"),
            "currency": data.get("cpm_currency")
        }
    return {"verified": False}

def update_order_status(transaction_id, status):
    """Update order status in your database"""
    # Your database update logic here
    print(f"✓ Order {transaction_id} status updated to: {status}")
```

### Webhook Requirements

- **Must accept POST and GET** requests
- **Must be publicly accessible** (https recommended)
- **Must respond with 200 OK** even if processing fails (so CinetPay stops retrying)
- **Must call verification API** — webhook data is for notification only, not for confirmation
- **CinetPay may retry** the notification multiple times; implement idempotency
- **Security**: Validate HMAC token in `X-TOKEN` header if present

---

## Common Integration Patterns

### Pattern 1: Mobile Money Payment (Mobile-First)

```python
# Scenario: User on mobile app buying digital content

def create_mobile_money_payment(user_email, amount_xof, country_code='CI'):
    """Create payment optimized for mobile money"""

    payload = {
        "apikey": os.getenv('CINETPAY_API_KEY'),
        "site_id": os.getenv('CINETPAY_SITE_ID'),
        "transaction_id": generate_transaction_id(),
        "amount": amount_xof,
        "currency": "XOF",
        "description": "Digital content purchase",
        "channels": "MOBILE",  # Show only mobile money methods
        "lang": "FR",  # Default to French for West Africa
        "notify_url": "https://yourdomain.com/webhooks/cinetpay",
        "return_url": "https://yourdomain.com/payment/success",
        "customer": {
            "name": "Customer",
            "surname": "Name",
            "email": user_email,
            "phone_number": "+225XXXXXXXXXX",
            "country": country_code
        }
    }

    response = requests.post(
        "https://api-checkout.cinetpay.com/v2/payment",
        json=payload,
        headers={"Content-Type": "application/json"}
    )

    result = response.json()
    if result.get("code") == "00":
        return {
            "payment_url": result["data"]["payment_url"],
            "token": result["data"]["token"],
            "transaction_id": payload["transaction_id"]
        }
    raise Exception(f"Payment init failed: {result.get('message')}")
```

### Pattern 2: Card Payment (E-commerce)

```python
# Scenario: Online store accepting international/local cards

def create_card_payment(customer_email, amount_xof, order_id):
    """Create payment optimized for card transactions"""

    payload = {
        "apikey": os.getenv('CINETPAY_API_KEY'),
        "site_id": os.getenv('CINETPAY_SITE_ID'),
        "transaction_id": f"CARD-{order_id}",
        "amount": amount_xof,
        "currency": "XOF",
        "description": f"Order {order_id}",
        "channels": "CARD",  # Show only card methods
        "lang": "EN",  # English for international merchants
        "notify_url": "https://yourdomain.com/webhooks/cinetpay",
        "return_url": f"https://yourdomain.com/orders/{order_id}/confirm",
        "customer": {
            "name": "John",
            "surname": "Doe",
            "email": customer_email,
            "phone_number": "+225XXXXXXXXXX",
            "address": "123 Street",
            "city": "Abidjan",
            "country": "CI"
        },
        "invoice_data": [
            {"name": "Product A", "quantity": "1", "unit_price": amount_xof}
        ]
    }

    response = requests.post(
        "https://api-checkout.cinetpay.com/v2/payment",
        json=payload,
        headers={"Content-Type": "application/json"}
    )

    result = response.json()
    return result
```

### Pattern 3: WooCommerce Integration

**Installation**:
1. Download CinetPay plugin from WordPress Admin → Plugins → Add New
2. Search "CinetPay" and install
3. Activate plugin
4. Go to WooCommerce → Settings → Payments
5. Configure with your API key and Site ID
6. Enable and save

**Features**:
- Automatic order status updates on payment confirmation
- Webhook auto-configuration
- Support for all 40+ payment methods
- Bilingual (FR/EN) payment interface

**Troubleshooting WooCommerce**:
- Verify notify_url is publicly accessible (WooCommerce generates this automatically)
- Check `wp-config.php` allows outbound POST requests
- Enable debug logging: `define('WP_DEBUG_LOG', true);`

### Pattern 4: PrestaShop Integration

**Installation**:
1. Download from PrestaShop Addons or `Module Manager`
2. Upload to `/modules/cinetpay/`
3. Install via PrestaShop Admin
4. Configure payment method with API key and Site ID
5. Test in PrestaShop sandbox

**Features**:
- Automatic order creation and tracking
- Multi-currency support (XOF, local currencies)
- Customer account linking
- Invoice generation on confirmation

---

## Error Handling

### Common Error Codes

| Code | Message | Description | Solution |
|------|---------|-------------|----------|
| `00` | SUCCES | Transaction successful | - |
| `201` | CREATED | Transaction created (pending) | Wait for webhook notification |
| `600` | PAYMENT_FAILED | Payment processing failed | Verify customer has sufficient balance; retry |
| `602` | INSUFFICIENT_BALANCE | Customer balance too low | Customer must top up wallet |
| `608` | MINIMUM_REQUIRED_FIELDS | Missing required parameter | Check all mandatory fields in request |
| `609` | AUTH_NOT_FOUND | Invalid API key | Verify API key from merchant panel |
| `613` | ERROR_SITE_ID_NOTVALID | Invalid site_id | Verify site_id from merchant panel |
| `627` | TRANSACTION_CANCEL | User cancelled payment | Inform customer; allow retry |
| `701` | INVALID_CREDENTIALS | Login credentials wrong | Check API key and site_id |
| `706` | INVALID_TOKEN | Payment token invalid/expired | Reinitialize payment |
| `723` | NOT_FOUND | Transaction not found | Verify transaction_id is correct |

### Error Handling Best Practices

```python
def safe_api_call(endpoint, payload, max_retries=3):
    """Safely call CinetPay API with retry logic"""

    import time

    for attempt in range(max_retries):
        try:
            response = requests.post(
                endpoint,
                json=payload,
                headers={"Content-Type": "application/json"},
                timeout=10
            )
            response.raise_for_status()

            result = response.json()

            # Check CinetPay error code
            if result.get("code") == "00":
                return {"success": True, "data": result}

            # Retryable errors
            elif result.get("code") in ["201", "627"]:
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
                return {"success": False, "retryable": True, "error": result}

            # Non-retryable errors
            else:
                return {"success": False, "retryable": False, "error": result}

        except requests.exceptions.Timeout:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)
                continue
            return {"success": False, "retryable": True, "error": "Timeout after retries"}

        except requests.exceptions.RequestException as e:
            return {"success": False, "retryable": False, "error": str(e)}

    return {"success": False, "retryable": False, "error": "Max retries exceeded"}
```

---

## Important Notes & Gotchas

### 1. **FCFA Currency Handling is Critical**
CinetPay operates exclusively in **West African CFA Franc (XOF)** or local currencies. If your business operates in USD/EUR, you **must convert at prevailing exchange rates before sending**. CinetPay will not accept or convert currencies dynamically. Example: `amount = usd_price * 650` (approximate rate; use live rates in production).

### 2. **Transaction IDs Must Be Unique Globally**
Every payment request requires a unique `transaction_id`. Reusing transaction IDs will either be rejected or cause duplicate payments. Use format: `TXN-{timestamp}-{uuid}` or `TXN-{increment}-{date}`. Store transaction IDs in your database immediately after generating them.

### 3. **Webhook is Notification Only, Not Confirmation**
CinetPay's webhook tells you payment status changed, **but you must always call the verification API** (`/v2/payment/check`) to confirm actual payment status. Man-in-the-middle attacks or network spoofing could fake webhook data. Never trust webhook data alone for fulfilling orders.

### 4. **Customer Phone Number Format is Strict**
Phone numbers must include country code (e.g., `+225XXXXXXXXXX` for Côte d'Ivoire). Without the leading `+` and country code, mobile money payments may fail silently. Validate and normalize phone numbers before sending: `+{country_code}{number}`.

### 5. **Language Parameter Affects Mobile Money Provider Display**
Setting `"lang": "FR"` shows payment interface in French and may prioritize French-language providers (Orange Money, Moov). Setting `"lang": "EN"` shows English and may prioritize English-supporting methods. For Francophone markets, use `"FR"` for better UX.

### 6. **Settlement Delays Are Standard (24-48 Hours)**
CinetPay settles funds to your merchant account within 24-48 hours, not immediately. If a customer contacts support 1 hour after payment saying "I haven't received my item," you must still mark the order as pending until CinetPay confirms settlement. Track settlement status in your merchant dashboard separately from payment verification.

### 7. **Country Codes Must Match Payment Method Available**
Not all payment methods are available in all countries. Example:
- Orange Money is available in: CI, SN, ML, GN
- MTN MoMo is available in: CM, CI, BJ, TG, BF
- Moov Money is available in: TG, BJ, CM, SN

If you send `country: "SN"` with `channels: "ALL"`, only Senegal-available methods appear. Verify country code matches available methods or use `channels: "ALL"` and let CinetPay filter.

### 8. **Return URL Redirect is Not Reliable for Payment Confirmation**
Customers may close the browser after payment before being redirected. **Never rely on return_url callbacks to confirm payment**. Only use `notify_url` (server-to-server webhook) and the verification API. The return_url is purely for user experience.

### 9. **Rate Limiting: 100 Requests Per Minute Per API Key**
CinetPay applies rate limits. If you exceed ~100 requests/minute, subsequent requests will be throttled. Batch verification requests and implement exponential backoff. Cache verification results when possible.

### 10. **Metadata and Custom Fields Are Silently Truncated**
Fields like `invoice_data`, `metadata`, and `description` have length limits (typically 255 chars). Longer values are silently truncated without error. Always validate field lengths before sending or risk losing important information.

### 11. **Service ID (site_id) Must Match Service Type**
Each `site_id` is tied to a specific service (e.g., "Payments", "Transfers", "Subscriptions"). You cannot use a `site_id` from the "Transfers" service for payment processing. Verify your `site_id` matches your integration use case in the merchant dashboard.

### 12. **Seamless Integration vs. Checkout Differs Significantly**
CinetPay offers two integration modes:
- **Standard (Checkout)**: Redirect to CinetPay-hosted page (this guide)
- **Seamless (SDK)**: Embedded payment form in your site (requires JavaScript SDK)

Seamless is faster but requires frontend SDK. Checkout is easier but requires redirect. Choose based on UX requirements, not interchangeably.

### 13. **Test vs. Production Environment**
Same API key works for **both test and production**. To test:
1. Use test credentials (API key + site_id configured for test)
2. Test notifications by simulating webhook calls
3. Verify with test transactions before going live
4. Production uses same endpoints; no URL changes needed

### 14. **HMAC Token Verification Implementation May Vary**
CinetPay includes an `X-TOKEN` header in webhook requests for HMAC verification. However, HMAC implementation details vary by documentation version. Always verify transactions directly with the API regardless of token validity.

### 15. **Payment Methods Depend on Customer's Mobile Operator**
Even if `channels: "MOBILE"` is set, customer's actual payment method options depend on their SIM card operator. A customer with Vodafone (no partnership) might not see any mobile money option despite requesting mobile channel. Design your UX to handle empty payment method lists gracefully.

---

## Useful Links

- **Official Documentation**: https://docs.cinetpay.com/
- **Payment Initialization Guide**: https://docs.cinetpay.com/api/1.0-en/checkout/initialisation
- **Transaction Verification**: https://docs.cinetpay.com/api/1.0-en/checkout/verification
- **Webhook/Notification Setup**: https://docs.cinetpay.com/api/1.0-en/checkout/notification
- **HMAC Token Verification**: https://docs.cinetpay.com/api/1.0-en/checkout/hmac
- **WooCommerce Integration**: https://docs.cinetpay.com/api/1.0-en/modules/woocommerce
- **PrestaShop Integration**: https://docs.cinetpay.com/api/1.0-en/modules/prestashop
- **PHP SDK**: https://github.com/cinetpay/cinetpay-php-legacy
- **JavaScript SDK**: https://github.com/cinetpay/seamlessIntegration
- **Merchant Dashboard**: https://app.cinetpay.com/marchand
- **Main Website**: https://cinetpay.com/
- **GitHub Organization**: https://github.com/cinetpay
