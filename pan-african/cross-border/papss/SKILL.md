---
title: PAPSS (Pan-African Payment and Settlement System)
description: Continental payment infrastructure enabling instant cross-border payments in local African currencies
aliases:
  - Pan-African Payment Settlement System
  - PAPSS API
  - Afreximbank PAPSS
triggers:
  - "PAPSS"
  - "Pan-African payments"
  - "cross-border Africa settlement"
  - "African currency exchange"
  - "intra-African payments"
  - "Afreximbank"
  - "PAPSSCARD"
  - "PACM"
  - "African Currency Marketplace"
  - "instant cross-border Africa"
difficulty: advanced
category: payment-infrastructure
status: production
updated: 2026-02-24
---

## What is PAPSS?

The **Pan-African Payment and Settlement System (PAPSS)** is a real-time gross settlement (RTGS) infrastructure for cross-border payments across Africa in local currencies. Developed and operated by the African Export-Import Bank (Afreximbank) in collaboration with the African Continental Free Trade Area (AfCFTA) Secretariat, PAPSS enables instant settlement of payments without requiring conversion to foreign currencies like USD or EUR.

PAPSS eliminates the need for correspondent banking corridors and foreign currency intermediaries, allowing companies across Africa to transact directly in their local currencies. As of early 2025, PAPSS connects 19 countries with 160+ commercial banks, and is expanding toward 40+ African nations.

## When to Use PAPSS

**Use PAPSS when you need to:**

- Send **instant cross-border payments** between African countries without foreign currency conversion
- Eliminate **correspondent bank delays** and reduce transaction costs for intra-African trade
- **Settle in local currencies** (e.g., Nigerian Naira directly to Kenyan Shilling) without USD/EUR intermediaries
- Access **continental card payments** via PAPSSCARD across participating African countries
- Facilitate **peer-to-peer currency exchange** at competitive rates via the African Currency Marketplace (PACM)
- Integrate **wholesale and retail payment flows** with a central African infrastructure rather than global networks
- Build **regulatory-compliant financial services** with support from African central banks

**PAPSS is NOT suitable for:**

- Non-African cross-border payments (outside the African continent)
- Countries not yet connected to PAPSS (see country list below)
- Direct consumer API access (PAPSS is infrastructure; access flows through banks/payment providers)
- Real-time exchange rate queries without broker relationships
- Transactions requiring immediate fiat stablecoin bridging across multiple blockchains

---

## Architecture and Infrastructure Model

PAPSS operates as a **continental financial market infrastructure (FMI)** connecting African central banks and commercial banks, not as a typical payment API with direct developer access.

### System Architecture

```
┌─────────────────────────────────────────────────────────┐
│          PAPSS Central Settlement Hub                    │
│         (Afreximbank-operated, cloud-based)              │
└──────────────────┬──────────────────────────────────────┘
                   │
    ┌──────────────┼──────────────────────┐
    │              │                      │
    ▼              ▼                      ▼
┌─────────────┐  ┌──────────────┐  ┌──────────────┐
│Central Bank │  │Central Bank  │  │ Central Bank │
│  (Nigeria)  │  │  (Kenya)     │  │   (Ghana)    │
└──────┬──────┘  └──────┬───────┘  └──────┬───────┘
       │                │                 │
       ▼                ▼                 ▼
    ┌────────────────────────────────────────┐
    │    RTGS (Real-Time Gross Settlement)   │
    │    systems of individual countries     │
    └────────────────────────────────────────┘
       │                │                 │
       ▼                ▼                 ▼
┌──────────────────────────────────────────────┐
│  Commercial Banks, Payment Switches, Fintechs │
│  (Direct & Indirect Participants)             │
└──────────────────────────────────────────────┘
```

### Key Architectural Components

**1. PAPSS Instant Payment System (PIP™)**
   - Core real-time payment processing engine
   - Supports wholesale and retail payment corridors
   - Hosted on secure, cloud-based infrastructure with 24/7 availability

**2. Central Bank RTGS Integration**
   - Each participating country's central bank maintains a RTGS system
   - PAPSS connects to these RTGS systems to enable pre-funding and settlement
   - Central banks validate and route payment instructions

**3. Settlement Agent (Afreximbank)**
   - Afreximbank acts as the settlement agent for all transactions
   - Issues daily net settlement instructions to central banks
   - Holds correspondent accounts with central banks for fund movements

**4. Messaging Standard: ISO 20022**
   - All payment instructions use ISO 20022 XML messaging standard
   - Enables rich payment data, complex instructions, and interoperability
   - Supports notifications at each transaction stage (validation, routing, settlement)

---

## Integration Model: Direct vs. Indirect Participants

PAPSS is **not directly accessible to developers or end-users**. Integration occurs through two participation models:

### Direct Participants

**Who:** Commercial banks, large payment service providers, and financial institutions with RTGS accounts in their country

**Integration Path:**
- Maintain a direct settlement account with the central bank of their operating country
- Connect directly to PAPSS and the country's RTGS system
- Submit payment instructions in ISO 20022 format
- Pre-fund their PAPSS account to cover outgoing payments
- Receive net settlement instructions from Afreximbank

**Technical Requirements:**
- ISO 20022 messaging capability
- RTGS account with home country's central bank
- Secure connectivity (typically SFTP, APIs, or banking networks like SWIFT)
- Compliance with PAPSS operational rules and AML/KYC standards

### Indirect Participants

**Who:** Smaller banks, payment service providers (fintechs, aggregators, remittance processors), and licensed financial services companies

**Integration Path:**
- Partner with a Direct Participant via a Sponsorship Agreement
- Issue funding instructions through their Direct Participant sponsor
- Direct Participant provides liquidity for their clearing account
- Do not maintain direct RTGS accounts

**Benefits:**
- Lower compliance and operational overhead
- Access to PAPSS without central bank settlement account
- Leverage an existing banking relationship

### For Developers/Startups

If you are a **fintech or payment startup**, your integration path is:

1. **Partner with a participating bank or payment service provider** that already connects to PAPSS
2. **Use their API or payment rails** to route transactions through PAPSS
3. **Examples:** Integration with Ecobank, Standard Bank, Zenith Bank, or other PAPSS-connected institutions
4. **You do not integrate directly with PAPSS**—you integrate with a bank that does

---

## Core Capabilities and Transaction Flows

### 1. Instant Payment System (IPS)

**Capability:** Real-time settlement of cross-border payments in local currencies

**Transaction Flow:**

```
1. ORIGINATOR INITIATES PAYMENT
   └─ Submits payment order to their bank/PSP in local currency

2. ORIGINATOR'S BANK VALIDATES
   └─ Checks account balance, KYC/AML compliance, PAPSS eligibility

3. TRANSMISSION TO PAPSS
   └─ Bank sends ISO 20022 payment instruction to PAPSS via central bank
   └─ Instruction includes amount in originating currency

4. PAPSS CENTRAL PROCESSING
   └─ PAPSS validates all transaction details
   └─ Confirms beneficiary bank is reachable on PAPSS
   └─ Initiates settlement sequence

5. FORWARD TO BENEFICIARY COUNTRY
   └─ Payment routed to beneficiary country's central bank
   └─ Central bank forwards to beneficiary's bank

6. LOCAL CURRENCY RECEIPT
   └─ Beneficiary's bank receives funds
   └─ Converts to beneficiary currency (if needed) at agreed rate
   └─ Credits beneficiary's account in local currency

7. SETTLEMENT (End-of-Day)
   └─ All transactions netted by currency pair
   └─ Afreximbank issues net settlement instructions
   └─ Central banks exchange hard currency to settle net positions
```

**Transaction Speed:** Typically completed within seconds (near real-time)

**Currencies:** Each country's local currency (Nigerian Naira, Kenyan Shilling, Ghanaian Cedi, etc.)

### 2. PAPSSCARD – Continental Card Scheme

**Capability:** Retail card payments across African borders in local currencies

**Features:**
- **Card Issuance:** Banks and payment providers issue PAPSSCARD debit/credit cards to customers
- **Cross-Border Acceptance:** Cards work at participating merchants across PAPSS-connected countries
- **Local Currency Settlement:** Transactions settled in merchant's local currency (no forex markup)
- **Real-Time Processing:** Authorizations and settlements processed within seconds
- **Lower Fees:** Reduced processing costs compared to international card networks (Visa, Mastercard)
- **EMV & Tokenization:** Fraud prevention via EMV standards and digital tokenization
- **Open API Architecture:** Banks integrate via PAPSSCARD APIs for card management, authorizations, and reconciliation

**Card Use Cases:**
- Consumer cross-border purchases
- Travel payments within Africa
- B2B supplier payments via corporate cards
- Merchant acquiring in multiple African countries

### 3. PAPSS African Currency Marketplace (PACM)

**Capability:** Peer-to-peer currency exchange for African currencies without forex conversion delays

**Technology Stack:**
- **Blockchain:** Built on Interstellar's permissioned blockchain infrastructure (Bantu blockchain)
- **Architecture:** Enterprise-grade, blockchain-agnostic with institutional security and near-instant settlement
- **Settlement:** Direct settlement in African currencies (no USD/EUR intermediaries)

**How It Works:**

```
1. MARKET PARTICIPANT (e.g., Kenya Airways) EARNS IN ONE CURRENCY
   └─ Kenya Airways receives payment in Nigerian Naira (NGN)

2. PARTICIPANT CONVERTS VIA PACM
   └─ Posts buy/sell orders for currency pairs on PACM
   └─ Example: Sell 1,000,000 NGN → Buy 2,500,000 KES

3. MATCHING & CLEARING
   └─ PACM matches counterparty orders on blockchain
   └─ Smart contracts manage escrow and settlement

4. INSTANT SETTLEMENT
   └─ Both parties settle in home currencies
   └─ NGN transferred to seller, KES transferred to buyer
   └─ Transaction completes in near real-time
```

**Pilot Results (2025):**
- 80+ African corporates participated
- 12+ currency pairs traded
- Zero foreign currency conversions (all local African currencies)

**Supported Currency Pairs:**
- NGN ↔ KES (Nigeria ↔ Kenya)
- NGN ↔ GHS (Nigeria ↔ Ghana)
- ZMW ↔ ZWL (Zambia ↔ Zimbabwe)
- And expanding pairs across 19+ connected countries

### 4. Pre-Funding Mechanism

**Capability:** Account management for intra-day liquidity on PAPSS

**Flow:**
- Direct Participants maintain pre-funded accounts on PAPSS
- Account balance checked before payment processing
- Insufficient funds = transaction rejected or queued
- Daily settlement of net positions at end-of-day
- Remaining balance carried to next business day

---

## Settlement and Clearing Model

### Real-Time Gross Settlement (RTGS)

PAPSS uses a **Real-Time Gross Settlement** approach for individual transactions:

- **Real-time:** Each payment is settled immediately upon validation (not batched)
- **Gross:** Transactions are settled individually, not netted during the day
- **No delays:** No correspondent banking hops or clearinghouse waits

### Multilateral Net Settlement (End-of-Day)

Daily settlement operates via multilateral netting:

```
END-OF-DAY SETTLEMENT PROCESS (Pre-midnight):

1. AGGREGATION
   ├─ PAPSS aggregates all transactions by currency pair
   ├─ Example: Total NGN→KES = +1M, Total KES→NGN = -800K
   └─ Net position = 200K NGN to Kenya

2. NETTING BY CURRENCY
   ├─ Each currency's net position calculated across all corridors
   ├─ If Nigeria = +2M net, Kenya = -1.5M net
   └─ Afreximbank issues settlement instructions for net amounts

3. SETTLEMENT INSTRUCTIONS
   ├─ Afreximbank debits Nigeria's account by 2M (USD equivalent)
   ├─ Afreximbank credits Kenya's account by 1.5M (USD equivalent)
   └─ Remaining 500K netted against other countries

4. HARD CURRENCY MOVEMENT
   ├─ Central banks exchange USD/hard currency for net positions
   ├─ No individual transaction requires forex conversion
   └─ Only net country-to-country flows use hard currency

5. RESET
   └─ All accounts reset to zero for next business day
```

### Settlement Agent Role (Afreximbank)

- Maintains correspondent accounts with all participating central banks
- Issues daily net settlement instructions to central banks
- Ensures sufficient hard currency liquidity for settlement
- Guarantees final settlement regardless of individual bank failures
- Acts as a trusted third party in the settlement process

### No Day-End Liquidity Issues

- Transactions settled real-time throughout the day
- Net settlement at end-of-day only involves net amounts
- Individual participants don't carry intraday settlement risk
- Central banks guarantee settlement finality

---

## Supported Countries and Currency Coverage

### Participating Countries (19+ as of Feb 2025)

| Country | Central Bank | Currency | Status |
|---------|-------------|----------|--------|
| Nigeria | Central Bank of Nigeria | NGN | Active |
| Ghana | Bank of Ghana | GHS | Active |
| Kenya | Central Bank of Kenya | KES | Active |
| Zambia | Bank of Zambia | ZMW | Active |
| Zimbabwe | Reserve Bank of Zimbabwe | ZWL | Active |
| Sierra Leone | Bank of Sierra Leone | SLL | Active |
| Liberia | Central Bank of Liberia | LRD | Active |
| Gambia | Central Bank of The Gambia | GMD | Active |
| Guinea | Central Bank of Guinea | GNF | Active |
| Rwanda | National Bank of Rwanda | RWF | Active |
| Djibouti | Central Bank of Djibouti | DJF | Active |
| Malawi | Reserve Bank of Malawi | MWK | Active |
| Algeria | Bank of Algeria | DZD | Active (joined 2025) |
| (7+ more) | In expansion phase | Various | Pipeline |

**Expected Expansion:** PAPSS aims to connect 40+ African countries by 2026-2027

### Currency Support Model

**Local Currency Transactions:**
- All payments sent and received in local/home currency
- No mandatory forex conversion at payment initiation
- Rates determined by market mechanisms on PAPSS or through PACM
- Central banks manage daily net settlement in hard currency

**Currency Pairs:**
- All bilateral pairs between connected countries are supported
- NGN-KES, NGN-GHS, KES-ZMW, etc.
- Rates set by market (PACM) or bilateral central bank arrangements

### Commercial Bank Coverage

- **160+ commercial banks** currently connected to PAPSS
- Includes major pan-African banks: Ecobank, Standard Bank, Zenith Bank, GTBank, Absa Bank, etc.
- Smaller regional banks integrated through sponsorship arrangements

---

## Important Notes and Gotchas

### 1. **PAPSS is Infrastructure, Not an API**
PAPSS is a financial market infrastructure operated by Afreximbank and connected at the central bank level. There is **no public REST API** for direct developer access. If you need to use PAPSS, you must do so through a participating bank or licensed payment service provider.

### 2. **No Direct Developer Integration**
Unlike payment gateways (Paystack, Flutterwave), you cannot sign up for PAPSS developer credentials or request API keys. You must partner with a bank or PSP that already connects to PAPSS.

### 3. **Bank/PSP Integration Required**
Your integration path: **Your App → Bank/PSP API → PAPSS Network**. Your payment provider (e.g., a fintech partner connected to Ecobank) exposes APIs that route transactions through PAPSS on your behalf.

### 4. **Central Bank Governance**
Each country's central bank controls its PAPSS node, validates transactions, and enforces regulations. Regulatory changes in any country can affect PAPSS transaction flows (e.g., KYC escalations, transaction limits, currency controls).

### 5. **Pre-Funding Requirement**
Direct participants must pre-fund their PAPSS accounts to send payments. If you're an indirect participant using a bank, the bank manages pre-funding. Insufficient account balance results in payment failure or queueing.

### 6. **Clearing and Settlement Delays**
While payments are processed in real-time, final settlement occurs at end-of-day via central bank hard currency movements. Beneficiary funds are typically available within seconds, but regulatory holds or country-specific compliance checks can extend this to hours.

### 7. **ISO 20022 Messaging**
All PAPSS communications use ISO 20022 XML format. If you're integrating via a bank API, the bank abstracts this. Direct participants must implement ISO 20022 message generation and parsing.

### 8. **AML/KYC Compliance is Strict**
PAPSS enforces enhanced KYC, sanctions screening, and beneficial ownership verification. Transaction initiation requires confirming both originator and beneficiary identities to central bank standards. Transactions failing compliance are rejected immediately.

### 9. **Multi-Currency Settlement Complexity**
On PAPSS, you send NGN and receive KES, but the exchange rate depends on:
- Market rate at time of submission (if using PACM)
- Bilateral central bank rates (if using IPS)
- Beneficiary bank's markup (if any)

The originating bank cannot guarantee the exact received amount before submission—only after central bank confirmation.

### 10. **Limited Transaction Limits**
Each country's central bank may impose transaction size limits for retail vs. wholesale payments, daily caps per account, or sector-specific limits. Check with your bank partner for their limits.

### 11. **Business Days and Cut-off Times**
PAPSS operates during African central banks' business hours (typically 08:00–17:00 local time). Off-hours transactions are queued until the next business day. Weekend and public holiday transactions are delayed.

### 12. **No Chargebacks or Reversals**
PAPSS transactions, once settled, are final. There is no automatic chargeback mechanism like international card networks. Disputes are resolved via bilateral agreement between banks and central banks (lengthy process).

### 13. **Blockchain in PACM is Permissioned**
The African Currency Marketplace (PACM) uses Interstellar's permissioned blockchain, not a public chain. You cannot directly access blockchain nodes or smart contracts—only through PAPSS or authorized brokers.

### 14. **Cost Structure**
PAPSS itself does not charge end-users. Costs are borne by participating banks and payment service providers. Banks pass on processing fees to you (typically lower than international corridors but not zero).

---

## Key Integration Considerations

### For Banks Connecting to PAPSS
- Implement ISO 20022 message generation and parsing
- Integrate with country's central bank RTGS system
- Establish settlement account with central bank
- Implement fraud detection and AML/KYC screening systems
- Maintain continuous connectivity with PAPSS Hub (24/7)
- Implement pre-funding liquidity management
- Reconcile end-of-day settlement instructions

### For Fintechs/Startups Using PAPSS
- Partner with a PAPSS-connected bank (as indirect participant)
- Use the bank's API to submit cross-border payments
- Manage customer KYC to bank standards
- Accept that final settlement is end-of-day; real-time notification only
- Design for retry logic (pre-funding failures, compliance rejections)
- Monitor your bank partner's PAPSS connectivity and availability

### For Payment Switches and Aggregators
- Integrate with multiple PAPSS-connected banks for redundancy
- Route transactions to the lowest-cost corridor (based on fees and rates)
- Implement currency conversion logic at pre-payment stage
- Cache PAPSS country/currency availability for routing decisions

---

## Useful Links

**Official PAPSS Resources:**
- [PAPSS Official Website](https://papss.com/)
- [PAPSS About Us](https://papss.com/about-us/)
- [PAPSS How It Works](https://papss.com/how-it-works/)
- [PAPSS Get Connected](https://papss.com/get-connected/)
- [PAPSSCARD](https://papss.com/papsscard/)

**Afreximbank Resources:**
- [African Export-Import Bank (Afreximbank)](https://www.afreximbank.com/)
- [PAPSSCARD Launch Announcement](https://www.afreximbank.com/africa-launches-first-pan-african-card-scheme-papsscard/)
- [PAPSS + Interstellar PACM Launch](https://www.afreximbank.com/papss-and-interstellar-unveil-african-currency-marketplace-eliminating-5-billion-trade-bottleneck/)
- [Afreximbank Careers (PAPSS Roles)](https://www.afreximbank.com/careers/)

**Research & News:**
- [AZA Finance: PAPSS Overview](https://azafinance.com/papss-what-it-is-benefits-and-how-it-works/)
- [The Cable: PAPSS Connects 19 Countries](https://www.thecable.ng/papss-now-connects-19-countries-facilitates-cross-border-payments-in-seconds-says-ceo/)
- [Wikipedia: Pan-African Payment and Settlement System](https://en.wikipedia.org/wiki/Pan-African_Payment_and_Settlement_System)
- [World Bank FASTT: PAPSS Profile](https://fastpayments.worldbank.org/node/373)
- [MEF: PAPSS Revolution of Cross-Border Payments](https://mobileecosystemforum.com/2025/02/19/pan-african-payment-and-settlement-system-papss-the-revolution-of-cross-border-payments-in-africa/)

**Technical Standards:**
- [ISO 20022 Standard](https://www.iso20022.org/about-iso-20022)
- [SWIFT ISO 20022 Resources](https://www.swift.com/standards/iso-20022/iso-20022-financial-institutions-focus-payments-instructions)

**Related Platforms:**
- [Interstellar: PAPSS Technology Partner](https://www.interstellar.co/)
- [Bank of Ghana PAPSS Info](https://boaghana.com/personal-banking/pan-african-payment-and-settlement-system-papss/)
- [Access Bank PAPSS Services](https://www.accessbankplc.com/personal/money-transfer/papss)

---

**Last Updated:** February 24, 2026
**Skill Status:** Production-Ready
**Difficulty:** Advanced (infrastructure-level integration)
