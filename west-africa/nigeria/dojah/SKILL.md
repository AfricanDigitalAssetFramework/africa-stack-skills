---
name: dojah
description: "Integrate with the Dojah KYC and identity verification API for African financial services. Use this skill whenever the user needs to verify BVN (Bank Verification Number), validate NIN (National Identification Number), perform liveness checks, conduct KYC/AML compliance, verify driver's licenses, government IDs, passports, or NUBAN accounts. Dojah supports biometric verification, fraud detection, and identity checks for Nigeria and other African countries. Trigger when the user mentions 'Dojah', 'BVN verification', 'NIN verification', 'KYC checks Nigeria', 'liveness detection', 'identity verification', 'NUBAN', 'passport lookup', 'phone number verification', or needs compliance services."
---

# Dojah Integration Skill

Dojah is an API-first infrastructure platform providing comprehensive KYC (Know Your Customer) and AML (Anti-Money Laundering) compliance for African financial services. It connects to government databases and financial institutions across Nigeria and other African countries, enabling identity verification, fraud detection, and regulatory compliance.

## When to use this skill

You're building fintech applications, digital marketplaces, payment platforms, or any service requiring customer identity verification in Africa. Dojah provides seamless identity verification against government databases (BVN, NIN, VNIN), financial institution records (NUBAN), and travel documents (passports). Essential for regulatory compliance, fraud prevention, and KYC onboarding workflows.

## Overview

Dojah's KYC suite includes:
- **Identity Verification**: BVN, NIN, VNIN, passports, driver's licenses
- **Financial Records**: NUBAN account validation
- **Biometric Verification**: Liveness checks and selfie-based verification
- **Document Verification**: Government ID scanning and validation
- **Phone Number Lookup**: Telecom provider verification
- **Fraud Detection**: Anti-fraud checks and verification intelligence
- **Multi-Country Support**: Nigeria, Kenya (KRA PIN), and other African nations
- **Address Verification**: Residential address validation

## Authentication

Dojah uses header-based authentication:

```
Authorization: Bearer your_api_key
AppId: your_app_id
```

API credentials are generated in your Dojah dashboard. Store in environment variables (`DOJAH_API_KEY`, `DOJAH_APP_ID`). Never hardcode credentials in source code.

**Base URL:** `https://api.dojah.io`

**Environments:**
- **Sandbox**: For testing with test data
- **Live**: For production verification against real government databases

**Sandbox Test BVN:** `22222222222`

## Core API Reference

### BVN Verification (Bank Verification Number)

Validate customer identity using their BVN, which is linked to Nigerian bank accounts.

#### BVN Basic Lookup
```
GET /api/v1/kyc/bvn?bvn=22222222222
```

**Parameters:**
- `bvn` (required): 11-digit Bank Verification Number

**Response:**
```json
{
  "entity": {
    "bvn": "22222222222",
    "first_name": "JOHN",
    "last_name": "DOE",
    "middle_name": "OLUWATOYIN",
    "phone_number1": "08012345678",
    "gender": "Male",
    "date_of_birth": "1990-05-15",
    "email": "john.doe@example.com",
    "registration_date": "2015-03-10",
    "nationality": "Nigeria",
    "state_of_residence": "Lagos"
  }
}
```

Note: response wraps in `entity`, not `data`. No `matched` field is returned for basic lookup.

#### BVN Validate (Name + DOB Match)
```
GET /api/v1/kyc/bvn/advance?bvn=22222222222&first_name=John&last_name=Doe&dob=1990-05-15
```

**Parameters:**
- `bvn` (required): 11-digit Bank Verification Number
- `first_name` (required): First name to match
- `last_name` (required): Last name to match
- `dob` (required): Date of birth in YYYY-MM-DD format

**Response:**
```json
{
  "entity": {
    "bvn": { "value": "22222222222", "status": true },
    "first_name": { "confidence_value": 100, "status": true },
    "date_of_birth": { "status": true }
  }
}
```

Use this endpoint (not the basic lookup) when you need to confirm a name matches the BVN on record.

#### BVN Full Lookup
```
GET /api/v1/kyc/bvn/full?bvn=22222222222
```

Returns comprehensive BVN data including banking relationships and verification history.

### NIN Verification (National Identification Number)

Validate identity using Nigeria's national ID system.

#### Basic NIN Lookup
```
GET /api/v1/kyc/nin?nin=12345678901
```

**Parameters:**
- `nin` (required): 11-digit National Identification Number

**Response:**
```json
{
  "entity": {
    "nin": "12345678901",
    "first_name": "Amina",
    "last_name": "Okafor",
    "middle_name": "Chioma",
    "phone": "08087654321",
    "gender": "f",
    "birthdate": "1990-05-15",
    "state": "Lagos",
    "lga": "Ikoyi",
    "nationality": "Nigeria",
    "marital_status": "Single",
    "profession": "Software Engineer"
  }
}
```

Note: response wraps in `entity`, not `data`. No `matched` field is returned; use `/api/v1/kyc/nin/verify` if you need name-match confirmation.

#### VNIN (Virtual NIN)
```
GET /api/v1/kyc/vnin?vnin=virtual_nin_value
```

Validates virtual NIN for digital-first verification.

### Biometric Verification

#### NIN with Selfie
```
POST /api/v1/kyc/nin/selfie
```

Verify NIN combined with biometric selfie validation for enhanced security.

#### BVN with Selfie
```
POST /api/v1/kyc/bvn/selfie
```

Combine BVN verification with selfie-based liveness detection.

#### Liveness Check
```
POST /api/v1/kyc/liveness
```

**Body:**
```json
{
  "image": "base64_encoded_image_data",
  "type": "selfie"
}
```

**Image Requirements:**
- Base64-encoded JPEG or PNG
- Clear frontal face photo
- Well-lit (natural or bright lighting, no heavy shadows)
- Face fully visible (no glasses, hats, or obscured features)
- Recent photo (not scanned from documents)
- File size under 5MB

**Response:**
```json
{
  "status": "success",
  "data": {
    "liveness_check_passed": true,
    "confidence_score": 98.5,
    "message": "Face is live and present"
  }
}
```

Confidence score ranges 0-100. Require >95 for high-security operations.

### NUBAN Verification

Verify Nigerian bank accounts using NUBAN (Nigeria Uniform Bank Account Number).

```
GET /api/v1/kyc/nuban?nuban=1234567890&bank_code=044
```

Returns account holder information linked to the bank account.

### Passport Verification

```
GET /api/v1/kyc/passport?passport_number=A12345678
```

Validates international passports with biographical data extraction.

### Phone Number Lookup

```
GET /api/v1/kyc/phone_number?phone_number=+2348012345678
```

Verifies phone number ownership and telecom provider information.

### Driver's License Verification

```
GET /api/v1/kyc/driver_license?license_number=ABC123456&first_name=Bolatito&last_name=Bankole
```

Validates driver's license and returns license holder information.

### Easy Lookup (No-Code Option)

Simple verification without complex parameters:

```
GET /api/v1/kyc/easy?identifier=phone_number_or_nin_or_bvn
```

## Common Integration Patterns

### New User Onboarding
1. Collect BVN or NIN from user during signup
2. Call BVN Basic Lookup (`GET /api/v1/kyc/bvn`) or NIN Lookup (`GET /api/v1/kyc/nin`)
3. Compare returned `entity` fields (name, DOB) against what the user provided
4. For name-match confirmation use BVN Validate (`GET /api/v1/kyc/bvn/advance`)
5. Request selfie for biometric verification → `POST /api/v1/kyc/liveness`
6. Store verification status and timestamp in database

### Fintech App Verification
1. User submits NIN during KYC flow
2. `GET /api/v1/kyc/nin?nin=...` validates identity; response in `entity` field
3. Compare returned name/DOB with signup data
4. Request selfie if data matches
5. `POST /api/v1/kyc/liveness` performs biometric check
6. Mark customer as verified if liveness score >95%

### Marketplace Seller Onboarding
1. Seller provides BVN or NIN
2. `GET /api/v1/kyc/bvn/full` or `/nin` retrieves comprehensive data
3. Cross-validate documents for consistency
4. Request liveness check for higher-risk sellers
5. Set transaction limits based on verification level

### High-Value Transaction Verification
1. On suspicious or high-value transactions
2. Request identity re-verification via liveness check
3. Compare current data with stored verification records
4. Block transaction if verification fails
5. Log failed verification attempts for compliance

## Error Handling

Dojah returns structured error responses:

```json
{
  "status": "error",
  "message": "Description of error",
  "code": "ERROR_CODE"
}
```

**Common HTTP Status Codes:**
- **400**: Validation error (invalid format, missing required fields)
- **401**: Unauthorized (invalid API key or App ID)
- **404**: Record not found (NIN/BVN doesn't exist in database)
- **429**: Rate limit exceeded — implement exponential backoff
- **500**: Server error — retry with exponential backoff

**Best Practices:**
- Implement retry logic for 5xx errors
- Cache verification results to avoid rate limiting
- Validate input format before API submission
- Store error logs for audit and compliance

## Important Notes and Best Practices

- **NIN/BVN Format**: Always 11 digits. Validate format before API submission.
- **Name Matching**: First and last names must match government records exactly. Nicknames or shortened names cause mismatches.
- **Middle Name**: Include middle name when available for higher accuracy.
- **Liveness Quality**: Image must be clear, well-lit, recent. Confidence threshold should be >95% for security-critical operations.
- **Rate Limiting**: Dojah enforces rate limits. Cache results and avoid repeated checks for same user.
- **Data Privacy**: Identity data and biometric images are sensitive. Never log unencrypted or store images without encryption.
- **Compliance**: Dojah verification supports regulatory compliance for financial services (CBN, FIRS requirements).
- **Match Verification**: Basic BVN/NIN lookups return the raw profile in `entity` (no `matched` field). Use `GET /api/v1/kyc/bvn/advance` (requires first_name, last_name, dob) for field-level match confirmation with per-field `status` booleans.
- **Sandbox Testing**: Use sandbox environment and test BVN `22222222222` for development.
- **Multi-Country**: Dojah supports Kenya (KRA PIN) and other African markets beyond Nigeria.

## Resources

- **API Documentation**: https://api-docs.dojah.io and https://docs.dojah.io
- **Sandbox Base URL**: Available in dashboard
- **Test Data**: Sandbox credentials provided in dashboard
- **Support**: Dojah support team for technical issues
