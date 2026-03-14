---
name: PaySwitch Ghana Payment Gateway
description: |
  PaySwitch is a wholly Ghanaian owned integrated payment solutions provider offering card hosting, POS, fraud prevention, payment gateway, and third-party processing (TPP) services. Integrate multiple payment methods including cards, mobile money, QR codes, and payment links for seamless transactions across C2C, C2B, B2C, and B2B flows.
triggers:
  - PaySwitch
  - Ghana payment gateway
  - Ghanaian payments
  - GHS mobile money
  - PaySwitch TPP
  - TheTeller
  - Card hosting Ghana
---

# PaySwitch Ghana Payment Gateway

## Introduction

PaySwitch is a FinTech specialist established in 2015 that provides integrated payment solutions for banks, non-banking financial institutions, telecommunications companies, and organizations across Ghana and West Africa. Their ecosystem is connected to telcos and banks through Ghana's national switch (GhIPSS - Ghana Interbank Payment and Settlement Systems Limited), enabling 360-degree transaction capabilities.

PaySwitch offers multiple convergent solutions including card issuance and hosting, third-party processing (TPP), fraud prevention and detection, service aggregation, customized payment solutions, and POS/MPOS capabilities. Their primary API product is **TheTeller**, a PCI DSS compliant payment platform designed for eCommerce integration.

## When to Use This Skill

Use PaySwitch/TheTeller when you need to:

- **Accept payments** in Ghana using Ghanaian payment methods
- **Integrate card payments** (Visa, Mastercard) from both local and international cards
- **Process mobile money** transactions through Ghanaian telcos
- **Generate payment links or QR codes** for quick-to-go "Scan, Pay and Go" experiences
- **Implement POS or mPOS solutions** for physical retail environments
- **Host and manage card data** securely with fraud prevention capabilities
- **Build B2B, B2C, C2B, or C2C payment flows** with sophisticated anti-fraud systems
- **Leverage third-party processing** for payment aggregation and distribution
- **Ensure regulatory compliance** with PCI DSS standards

## Authentication

### OAuth 2 Authentication (Recommended)

PaySwitch uses OAuth 2 for API authentication. Obtain a token by POSTing to the OAuth endpoint with your credentials.

**Endpoint:** `POST https://try.payswitch.net/oauth/token`

**Request:**
```json
{
  "email": "your-email@example.com",
  "password": "your-password",
  "grant_type": "password"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

**Using the Token:**

Set the Authorization header in all subsequent API requests:

```bash
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Legacy Header-Based Authentication (Alternative)

For legacy integrations, you can use X-User-Email and X-User-Token headers:

```bash
X-User-Email: your-email@example.com
X-User-Token: your-api-token
```

**Note:** When using Bearer token authentication, the X-User-Email and X-User-Token headers are no longer required.

## Core API Reference

### Base URLs

- **Test Environment:** `https://try.payswitch.net`
- **Live Environment:** `https://api.payswitch.net` (contact PaySwitch for access)

### TheTeller Transaction Processing

TheTeller is PaySwitch's main payment processing API for handling transactions and fund transfers.

**Endpoint:** `POST /v1.1/transaction/process`

**Headers:**
```
Content-Type: application/json
Authorization: Basic base64(api_username:api_key)
```

**Request Body Example:**
```json
{
  "merchant_id": "MCH123456",
  "transaction_id": "TXN20240224001",
  "processing_code": "000000",
  "amount": 10000,
  "currency": "GHS",
  "account_number": "1234567890",
  "account_issuer": "GCB",
  "description": "Payment for Order #12345",
  "customer_name": "John Doe",
  "customer_phone": "+233501234567",
  "customer_email": "john@example.com"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Transaction processed successfully",
  "transaction_reference": "TXN20240224001",
  "timestamp": "2024-02-24T10:30:00Z",
  "amount": 10000,
  "currency": "GHS",
  "merchant_id": "MCH123456"
}
```

### Payment Link Generation

Generate shareable payment links for customers to complete payments without requiring code integration.

**Endpoint:** `POST /api/v1/payment-link/generate`

**Request:**
```json
{
  "amount": 5000,
  "currency": "GHS",
  "description": "Payment for Invoice #789",
  "customer_name": "Jane Smith",
  "customer_email": "jane@example.com",
  "customer_phone": "+233501234567",
  "redirect_url": "https://yoursite.com/success",
  "metadata": {
    "order_id": "ORD123",
    "customer_ref": "CUST456"
  }
}
```

**Response:**
```json
{
  "status": "success",
  "payment_link": "https://pay.theteller.net/link/abc123def456",
  "link_id": "abc123def456",
  "expires_at": "2024-03-25T10:30:00Z",
  "short_url": "https://teller.link/abc123"
}
```

### QR Code Generation

Generate QR codes that customers can scan to initiate payments ("Scan, Pay and Go").

**Endpoint:** `POST /api/v1/qr-code/generate`

**Request:**
```json
{
  "amount": 2500,
  "currency": "GHS",
  "merchant_name": "Your Store",
  "reference": "QR20240224001",
  "description": "Payment for items"
}
```

**Response:**
```json
{
  "status": "success",
  "qr_code_url": "https://cdn.theteller.net/qr/qr20240224001.png",
  "qr_data": "https://pay.theteller.net/qr/qr20240224001",
  "reference": "QR20240224001"
}
```

### Get Current User

Retrieve information about the authenticated user/merchant.

**Endpoint:** `GET /api/users/current`

**Response:**
```json
{
  "id": "usr_123456",
  "email": "merchant@example.com",
  "merchant_name": "Your Business",
  "merchant_id": "MCH123456",
  "status": "active",
  "created_at": "2023-01-15T00:00:00Z"
}
```

### Get Product Categories

Retrieve available payment product categories.

**Endpoint:** `GET /api/product_categories/main`

**Response:**
```json
{
  "status": "success",
  "categories": [
    {
      "id": "cat_001",
      "name": "Cards",
      "description": "Local and international card payments"
    },
    {
      "id": "cat_002",
      "name": "Mobile Money",
      "description": "Mobile money service providers"
    },
    {
      "id": "cat_003",
      "name": "Bank Transfer",
      "description": "Direct bank account transfers"
    }
  ]
}
```

## Webhooks

PaySwitch sends webhooks to notify your application of transaction events. Configure your webhook URL in the PaySwitch dashboard.

### Webhook Events

Common webhook events include:

- `transaction.completed` - Transaction successfully processed
- `transaction.failed` - Transaction failed
- `transaction.pending` - Transaction waiting for approval
- `refund.processed` - Refund has been processed
- `transfer.completed` - Fund transfer completed

### Webhook Structure

**Event Payload:**
```json
{
  "event_id": "evt_abc123def456",
  "event_type": "transaction.completed",
  "timestamp": "2024-02-24T10:30:00Z",
  "data": {
    "transaction_id": "TXN20240224001",
    "merchant_id": "MCH123456",
    "amount": 10000,
    "currency": "GHS",
    "status": "completed",
    "payment_method": "card",
    "customer_email": "john@example.com",
    "reference": "TXN20240224001"
  }
}
```

### Webhook Headers

PaySwitch includes security headers with each webhook:

```
X-PaySwitch-Signature: hmac-sha512-signature
X-PaySwitch-Event-ID: evt_abc123def456
X-PaySwitch-Timestamp: 1708774200
```

### Webhook Verification

Verify webhook authenticity using the signature header with HMAC-SHA512:

```python
import hmac
import hashlib

def verify_webhook(payload, signature, webhook_secret):
    """Verify PaySwitch webhook signature"""
    expected_signature = hmac.new(
        webhook_secret.encode(),
        payload.encode(),
        hashlib.sha512
    ).hexdigest()
    return hmac.compare_digest(expected_signature, signature)
```

### Webhook Retry Logic

PaySwitch expects HTTP 2XX responses for successful webhook delivery. If your endpoint does not return a 2XX status:

- PaySwitch will retry with increasing delays over the next 24 hours
- Your application may receive duplicate webhooks for the same event
- Use the `event_id` field to detect and handle duplicates
- Implement idempotent handlers for webhook processing

**Recommended Retry Detection:**
```python
# Store processed event_ids in database
processed_events = set()

def handle_webhook(event_payload):
    event_id = event_payload['event_id']
    if event_id in processed_events:
        return {"status": "ok"}  # Already processed

    # Process the event
    process_transaction(event_payload)
    processed_events.add(event_id)
    return {"status": "ok"}
```

## Common Integration Patterns

### Pattern 1: Basic Card Payment Integration

Integrate card payments for an eCommerce checkout:

```python
import requests
import base64
from datetime import datetime

class PaySwitchClient:
    def __init__(self, api_username, api_key, environment='test'):
        self.environment = environment
        self.base_url = "https://try.payswitch.net" if environment == 'test' else "https://api.payswitch.net"
        self.auth = base64.b64encode(f"{api_username}:{api_key}".encode()).decode()

    def process_payment(self, amount, currency, customer_email, order_id):
        """Process a card payment"""
        payload = {
            "merchant_id": "MCH123456",
            "transaction_id": f"TXN{datetime.now().strftime('%Y%m%d%H%M%S')}",
            "processing_code": "000000",
            "amount": int(amount * 100),  # Convert to smallest currency unit
            "currency": currency,
            "description": f"Payment for Order #{order_id}",
            "customer_email": customer_email,
            "redirect_url": "https://yoursite.com/payment-confirmation"
        }

        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Basic {self.auth}"
        }

        response = requests.post(
            f"{self.base_url}/v1.1/transaction/process",
            json=payload,
            headers=headers
        )
        return response.json()
```

### Pattern 2: Generate Payment Link for Invoice

Create a shareable payment link for invoice collection:

```python
def generate_invoice_payment_link(client, invoice_data):
    """Generate a payment link for an invoice"""
    payload = {
        "amount": invoice_data['total_amount'],
        "currency": "GHS",
        "description": f"Invoice #{invoice_data['invoice_number']}",
        "customer_name": invoice_data['customer_name'],
        "customer_email": invoice_data['customer_email'],
        "redirect_url": f"https://yoursite.com/invoice/{invoice_data['invoice_id']}/paid",
        "metadata": {
            "invoice_id": invoice_data['invoice_id'],
            "customer_ref": invoice_data['customer_id']
        }
    }

    # Use OAuth bearer token
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {oauth_token}"
    }

    response = requests.post(
        f"{client.base_url}/api/v1/payment-link/generate",
        json=payload,
        headers=headers
    )

    result = response.json()
    if result['status'] == 'success':
        # Send payment link to customer
        send_email_to_customer(
            invoice_data['customer_email'],
            f"Payment Link: {result['payment_link']}"
        )
    return result
```

### Pattern 3: POS/In-Store QR Code Payments

Enable "Scan, Pay and Go" checkout with QR codes:

```python
def create_pos_transaction_qr(client, transaction_amount, merchant_reference):
    """Create a QR code for in-store payment"""
    payload = {
        "amount": transaction_amount,
        "currency": "GHS",
        "merchant_name": "Your Store Name",
        "reference": merchant_reference,
        "description": "In-store purchase"
    }

    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {oauth_token}"
    }

    response = requests.post(
        f"{client.base_url}/api/v1/qr-code/generate",
        json=payload,
        headers=headers
    )

    result = response.json()
    # Display QR code URL on POS terminal
    print(f"QR Code: {result['qr_code_url']}")
    return result
```

### Pattern 4: Handle Webhook Events

Process incoming webhook events securely:

```python
from flask import Flask, request, jsonify
import json

app = Flask(__name__)

# Store your webhook secret from PaySwitch dashboard
WEBHOOK_SECRET = "your_webhook_secret_key"

@app.route('/webhooks/payswitch', methods=['POST'])
def handle_payswitch_webhook():
    """Handle incoming PaySwitch webhooks"""
    # Verify signature
    payload = request.get_data(as_text=True)
    signature = request.headers.get('X-PaySwitch-Signature')

    if not verify_webhook_signature(payload, signature, WEBHOOK_SECRET):
        return {"error": "Invalid signature"}, 401

    event_data = request.get_json()
    event_id = event_data['event_id']

    # Check for duplicate processing
    if is_event_processed(event_id):
        return {"status": "ok"}, 200

    # Handle different event types
    if event_data['event_type'] == 'transaction.completed':
        process_successful_payment(event_data['data'])
    elif event_data['event_type'] == 'transaction.failed':
        process_failed_payment(event_data['data'])

    # Mark event as processed
    mark_event_processed(event_id)

    return {"status": "ok"}, 200
```

## Error Handling

PaySwitch API uses standard HTTP status codes and provides descriptive error messages.

### HTTP Status Codes

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorized | Invalid/missing authentication |
| 403 | Forbidden | Access denied to resource |
| 404 | Not Found | Resource does not exist |
| 409 | Conflict | Duplicate transaction or conflict |
| 422 | Unprocessable Entity | Validation error in payload |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Server Error | Internal server error |
| 503 | Service Unavailable | Service temporarily down |

### Error Response Format

```json
{
  "status": "error",
  "code": "INVALID_AMOUNT",
  "message": "Amount must be greater than 0",
  "details": {
    "field": "amount",
    "value": -100,
    "constraint": "min_value:1"
  },
  "timestamp": "2024-02-24T10:30:00Z"
}
```

### Common Error Codes

- `INVALID_AMOUNT` - Amount is zero or negative
- `INVALID_CURRENCY` - Currency code not supported
- `INSUFFICIENT_BALANCE` - Merchant account balance insufficient
- `DUPLICATE_TRANSACTION` - Transaction ID already exists
- `INVALID_MERCHANT_ID` - Merchant ID not valid
- `AUTH_FAILED` - Authentication failed or token expired
- `RATE_LIMIT_EXCEEDED` - Too many requests in time window
- `INVALID_WEBHOOK_URL` - Webhook URL format invalid
- `PAYMENT_METHOD_NOT_SUPPORTED` - Selected payment method unavailable
- `NETWORK_ERROR` - Unable to reach payment processor

### Error Handling Best Practices

```python
def make_api_request(endpoint, payload):
    """Make API request with comprehensive error handling"""
    try:
        response = requests.post(endpoint, json=payload, timeout=30)
        response.raise_for_status()
        return response.json()

    except requests.exceptions.Timeout:
        # Handle timeout - retry with exponential backoff
        return retry_with_backoff(endpoint, payload)

    except requests.exceptions.HTTPError as e:
        error_data = e.response.json()
        if e.response.status_code == 401:
            # Refresh authentication token
            refresh_auth_token()
            return make_api_request(endpoint, payload)  # Retry
        elif e.response.status_code == 429:
            # Rate limited - wait and retry
            time.sleep(60)
            return make_api_request(endpoint, payload)
        else:
            log_error(error_data)
            raise

    except Exception as e:
        log_error(str(e))
        raise
```

## Important Notes / Gotchas

### 1. Amount Format and Currency Units

PaySwitch APIs handle amounts in the **smallest currency unit** (pesewas for GHS). Always multiply amounts by 100:
- Display amount: 100 GHS → API amount: 10000
- Always store amounts as integers to avoid floating-point precision issues

### 2. Transaction ID Uniqueness

Each `transaction_id` must be **globally unique across all time**. Reusing transaction IDs will result in `DUPLICATE_TRANSACTION` errors. Recommended practice:
```python
transaction_id = f"TXN{datetime.utcnow().strftime('%Y%m%d%H%M%S')}{random_suffix}"
```

### 3. Authentication Token Expiration

OAuth tokens have a limited lifetime (typically 3600 seconds). Implement token refresh logic:
- Store token expiration time
- Refresh proactively before expiration
- Catch 401 Unauthorized responses and refresh on-demand
- Never hardcode tokens in code

### 4. Webhook Signature Verification is Critical

Always verify webhook signatures to prevent spoofed events. Use HMAC-SHA512 with your webhook secret. Unverified webhooks could represent fraudulent or replay attacks.

### 5. Mobile Money Integration Complexity

Mobile money integration requires:
- Telco-specific routing rules
- USSD fallback options for low-bandwidth scenarios
- Longer timeout windows (telcos are slower than card networks)
- Handle network timeouts gracefully with transaction status checking

### 6. PCI Compliance Requirements

- Never store full card details; PaySwitch is PCI DSS certified
- Use payment links or hosted forms for card data collection
- Transmit card data only over HTTPS with proper TLS versions
- Implement proper access controls for API credentials

### 7. Test vs. Live Environment Separation

- Test environment: `https://try.payswitch.net`
- Live environment: `https://api.payswitch.net`
- Use separate API credentials for each environment
- Test all scenarios thoroughly before going live
- Credentials cannot be mixed between environments

### 8. Webhook Delivery is Not Guaranteed

- Webhooks can be delayed (up to 24 hours in retry window)
- Webhooks can be delivered multiple times (use event_id for deduplication)
- Always implement periodic reconciliation checks
- Poll transaction status endpoint as backup verification
- Do not rely solely on webhooks for critical operations

### 9. Rate Limiting

PaySwitch implements rate limiting per merchant account:
- Monitor 429 responses
- Implement exponential backoff for retries
- Space out bulk operations
- Contact PaySwitch for higher limits if needed

### 10. Supported Payment Methods Vary by Region

- Cards: Visa, Mastercard (local and international)
- Mobile Money: Available through Ghanaian telcos
- Bank Transfer: Limited to Ghanaian banks
- Geographic and telco availability may change
- Test all payment methods in test environment

### 11. Merchant Account Status Impact

Your PaySwitch merchant account status affects API availability:
- `active` - Full API access
- `suspended` - Limited/no API access
- `closed` - No API access
- Check merchant status regularly
- Contact PaySwitch support if status changes unexpectedly

### 12. Timezone Handling

- All timestamps in API responses are UTC
- Store timestamps as UTC in your system
- Convert to local timezone for display only
- Be consistent with timezone handling in webhooks

## Useful Links

- **PaySwitch Official Website:** https://www.payswitch.com.gh/
- **PaySwitch API Documentation:** https://docs.payswitch.net/
- **TheTeller API Documentation:** https://theteller.net/documentation
- **TheTeller Platform:** https://www.theteller.net/
- **MarcoPolis - PaySwitch Overview:** https://marcopolis.net/inside-payswitch-ghana-building-a-trusted-api-driven-payment-ecosystem-for-banks-smes-and-merchants.htm
- **GhIPSS (Ghana Interbank Payment and Settlement Systems):** Official national payment switch
- **Ghana Web - PaySwitch News:** https://www.ghanaweb.com/

---

**Note:** This documentation is based on available research as of February 2024. PaySwitch API specifications may change. Always consult the official documentation at https://docs.payswitch.net/ for the most current information. Contact PaySwitch developer support at ps_dev@payswitch.com.gh for clarifications on API behavior and integration specifics.
