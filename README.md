# Africa Stack Skills

AI agent skills for African payment APIs. One SKILL.md per API — each one teaches an AI agent the endpoints, auth patterns, error codes, and usage examples for that provider.

Built by [ADAF — African Digital Asset Foundation](https://github.com/AfricanDigitalAssetFramework).

---

## What is a SKILL.md?

Each `SKILL.md` is a structured instruction file for AI agents (Claude, GPT, Gemini, and compatible runtimes). Drop it into your agent context and it gains the ability to call that API — correctly, with proper auth, real endpoints, and working code examples.

---

## Coverage — 54 APIs across Africa

### 🌍 West Africa

#### Nigeria (12 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🇳🇬 Paystack | Bearer | Bridge | [SKILL.md](west-africa/nigeria/paystack/SKILL.md) |
| 🇳🇬 Flutterwave | Bearer | Bridge | [SKILL.md](west-africa/nigeria/flutterwave/SKILL.md) |
| 🇳🇬 Remita | Bearer | On-ramp | [SKILL.md](west-africa/nigeria/remita/SKILL.md) |
| 🇳🇬 Kuda Bank | Bearer | Bridge | [SKILL.md](west-africa/nigeria/kuda/SKILL.md) |
| 🇳🇬 KoraPay | Bearer | Bridge | [SKILL.md](west-africa/nigeria/korapay/SKILL.md) |
| 🇳🇬 Monnify | Bearer | On-ramp | [SKILL.md](west-africa/nigeria/monnify/SKILL.md) |
| 🇳🇬 Squad | Bearer | Bridge | [SKILL.md](west-africa/nigeria/squad/SKILL.md) |
| 🇳🇬 Interswitch | Bearer | Bridge | [SKILL.md](west-africa/nigeria/interswitch/SKILL.md) |
| 🇳🇬 Seerbit | Bearer | Bridge | [SKILL.md](west-africa/nigeria/seerbit/SKILL.md) |
| 🇳🇬 OnePipe | Bearer | Bridge | [SKILL.md](west-africa/nigeria/onepipe/SKILL.md) |
| 🇳🇬 Termii | Bearer | Messaging | [SKILL.md](west-africa/nigeria/termii/SKILL.md) |
| 🇳🇬 Dojah | Bearer | KYC | [SKILL.md](west-africa/nigeria/dojah/SKILL.md) |

#### Ghana (2 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🇬🇭 Hubtel | Basic | Bridge | [SKILL.md](west-africa/ghana/hubtel/SKILL.md) |
| 🇬🇭 PaySwitch | Custom | Bridge | [SKILL.md](west-africa/ghana/payswitch/SKILL.md) |

---

### 🌍 East Africa

#### Kenya (10 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🇰🇪 M-Pesa Daraja | OAuth2 | Bridge | [SKILL.md](east-africa/mpesa-daraja/SKILL.md) |
| 🇰🇪 PesaPal | Bearer | Bridge | [SKILL.md](east-africa/pesapal/SKILL.md) |
| 🇰🇪 Africa's Talking | Bearer | Bridge | [SKILL.md](east-africa/africastalking/SKILL.md) |
| 🇰🇪 Kopo Kopo | Bearer | Bridge | [SKILL.md](east-africa/kopokopo/SKILL.md) |
| 🇰🇪 IntaSend | Bearer | Bridge | [SKILL.md](east-africa/intasend/SKILL.md) |
| 🇰🇪 Cellulant | Bearer | Bridge | [SKILL.md](east-africa/cellulant/SKILL.md) |
| 🇰🇪 NCBA Loop | Bearer | Bridge | [SKILL.md](east-africa/ncba-loop/SKILL.md) |
| 🇰🇪 Equity Bank API | Bearer | Bridge | [SKILL.md](east-africa/equity-api/SKILL.md) |
| 🇰🇪 JamboPay | Bearer | On-ramp | [SKILL.md](east-africa/jambopay/SKILL.md) |
| 🇰🇪 iPay | HMAC-SHA256 | Bridge | [SKILL.md](east-africa/ipay/SKILL.md) |

---

### 🌍 Southern Africa

#### South Africa (10 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🇿🇦 Stitch | OAuth2 | Bridge | [SKILL.md](southern-africa/stitch/SKILL.md) |
| 🇿🇦 Ozow | HMAC-SHA256 | On-ramp | [SKILL.md](southern-africa/ozow/SKILL.md) |
| 🇿🇦 Peach Payments | Bearer | Bridge | [SKILL.md](southern-africa/peach-payments/SKILL.md) |
| 🇿🇦 Yoco | Bearer | On-ramp | [SKILL.md](southern-africa/yoco/SKILL.md) |
| 🇿🇦 PayFast | HMAC-SHA256 | Bridge | [SKILL.md](southern-africa/payfast/SKILL.md) |
| 🇿🇦 Investec | OAuth2 | Bridge | [SKILL.md](southern-africa/investec/SKILL.md) |
| 🇿🇦 SnapScan | Basic | On-ramp | [SKILL.md](southern-africa/snapscan/SKILL.md) |
| 🇿🇦 iKhokha | HMAC-SHA256 | On-ramp | [SKILL.md](southern-africa/ikhokha/SKILL.md) |
| 🇿🇦 Nedbank API | Bearer | Bridge | [SKILL.md](southern-africa/nedbank-api/SKILL.md) |
| 🇿🇦 PayU SA | Bearer | Bridge | [SKILL.md](southern-africa/payu-sa/SKILL.md) |

---

### 🌍 North Africa

#### Egypt (7 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🇪🇬 Fawry | HMAC-SHA256 | Bridge | [SKILL.md](north-africa/fawry/SKILL.md) |
| 🇪🇬 Paymob | Bearer | Bridge | [SKILL.md](north-africa/paymob/SKILL.md) |
| 🇪🇬 Kashier | Bearer | On-ramp | [SKILL.md](north-africa/kashier/SKILL.md) |
| 🇪🇬 Cowpay | HMAC-SHA256 | On-ramp | [SKILL.md](north-africa/cowpay/SKILL.md) |
| 🇪🇬 Vodafone Cash | Bearer | Bridge | [SKILL.md](north-africa/vodafone-cash/SKILL.md) |
| 🇪🇬 ValU | Bearer | On-ramp | [SKILL.md](north-africa/valu/SKILL.md) |
| 🇪🇬 Tap Payments | Bearer | Bridge | [SKILL.md](north-africa/tap-payments/SKILL.md) |

---

### 🌐 Pan-African

#### Pan-African (5 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🌍 Mono | Custom | Bridge | [SKILL.md](pan-african/pan-african/mono/SKILL.md) |
| 🌍 Chipper Cash | Bearer | Bridge | [SKILL.md](pan-african/pan-african/chipper-cash/SKILL.md) |
| 🌍 Onafriq (MFS Africa) | Bearer | Bridge | [SKILL.md](pan-african/pan-african/mfs-africa/SKILL.md) |
| 🌍 DPO Group | Custom | Bridge | [SKILL.md](pan-african/pan-african/dpo-group/SKILL.md) |
| 🌍 PawaPay | Bearer | Bridge | [SKILL.md](pan-african/pan-african/pawapay/SKILL.md) |

#### Mobile Money (2 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🌍 MTN MoMo | OAuth2 | Bridge | [SKILL.md](pan-african/mobile-money/mtn-momo/SKILL.md) |
| 🌍 Airtel Money | OAuth2 | Bridge | [SKILL.md](pan-african/mobile-money/airtel-money/SKILL.md) |

#### Cross-Border (2 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🌍 Thunes | Basic | Bridge | [SKILL.md](pan-african/cross-border/thunes/SKILL.md) |
| 🌍 PAPSS | Custom | Bridge | [SKILL.md](pan-african/cross-border/papss/SKILL.md) |

#### UEMOA / Francophone West Africa (4 APIs)

| API | Auth | Capability | Skill |
|-----|------|------------|-------|
| 🇨🇮 CinetPay | Bearer | Bridge | [SKILL.md](pan-african/uemoa/cinetpay/SKILL.md) |
| 🇧🇯 KKiaPay | Bearer | Bridge | [SKILL.md](pan-african/uemoa/kkiapay/SKILL.md) |
| 🇧🇯 FedaPay | Bearer | Bridge | [SKILL.md](pan-african/uemoa/fedapay/SKILL.md) |
| 🇸🇳 Wave | Bearer | Bridge | [SKILL.md](pan-african/uemoa/wave/SKILL.md) |

---

## Auth Patterns

| Pattern | APIs |
|---------|------|
| Bearer token | 31 APIs |
| OAuth2 | 10 APIs |
| HMAC-SHA256 | 6 APIs |
| Basic auth | 3 APIs |
| Custom | 4 APIs |

---

## Contributing

Found an outdated endpoint? Missing an API? Open an issue or PR.

- **Bug:** endpoint, auth, or response shape is wrong
- **New API:** open an issue with the API docs link — we'll add it

---

## License

MIT — African Digital Asset Foundation
