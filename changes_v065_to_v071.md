# Secure Crypto P2M EUDI Use Case — Changes from v0.6.5 to v0.7.1

_13 March 2026 · For APTITUDE WP6 / TS12 WG review_

---

## Purpose

This document summarises the technical changes introduced in v0.7.1 relative to the v0.6.5 baseline. It is intended for TS12 WG participants and APTITUDE WP6 contributors who have already read v0.6.5 and want to understand what changed, why, and where the new requirements come from.

The primary goal of the changes is to make the specification **fully grounded in TS12 v1.0.1** and OID4VP, removing fields we had introduced that duplicate mechanisms already defined by those standards, and adding explicit requirements where the v0.6.5 spec was silent.

---

## Summary of Changes

| Area | v0.6.5 | v0.7.1 |
|------|----------------|--------|
| Credential type URI | `https://talao.co/vct/crypto` | `urn:eudi:sca:crypto:1` — issuer-independent URN |
| `blockchain_logo` claim | Present in signed credential | Removed — belongs in `vct` metadata, not in signed claims |
| Revocation | Not specified | **Mandatory** StatusList2021 + `revocation_id` |
| `from_address` in payload | Absent | **Mandatory** — binds originator for TFR & dynamic linking |
| Amount encoding | `"amount": 10` (bare number) | `amount_display` + decimal string in smallest unit (float-safe) |
| Authorization Request query language | Presentation Definition | **DCQL** (`dcql_query`) per ARF-adopted standard |
| Credential issuance flow | Not specified | Full OID4VCI flow with challenge-response address binding |
| Dynamic Linking requirements | Referenced loosely | **DL-1–DL-5** explicit requirements anchored in TS12 |
| Replay protection | Not specified | **RP-1–RP-3** grounded in TS12 KB-JWT `jti` & `nonce` |
| KB-JWT required claims | Not mentioned | `jti`, `amr`, `response_mode` (all REQUIRED by TS12 §3.6) |
| P2P mutual authentication | Not specified | Section 16: receiver (Q)EAA verification for high-value P2P |
| Split-wallet model | Not addressed | Section 16a: two-session OID4VP flow for non-custodial government wallets |
| On-chain consent anchoring | Mentioned (`transaction_id` in calldata) | Section 17: specified per implementation option (native, ERC-20 event, `transferFrom`, ERC-3009) |
| Credential lifecycle & revocation | Not specified | Section 18: 6–12 month expiry, revocation triggers, re-issuance flow |
| Open WG issues | None raised | **7 open issues** for TS12 WG (OPEN-1–7) |
| TypeScript definitions | None | Annex B: full TS types + `encodeCalldata()`, `computeConsentHash()` |

---

## 1. Credential Schema

### 1.1 Credential type URI

v0.6.5 uses `"vct": "https://talao.co/vct/crypto"` — a URL bound to the issuer's domain. This creates issuer coupling: any other Attestation Provider wanting to issue the same credential type would need to either host the issuer's metadata or use a different `vct`, breaking interoperability.

v0.7.1 changes this to `"vct": "urn:eudi:sca:crypto:1"` — a URN that is issuer-independent and follows the same pattern as the PID (`urn:eudi:pid:de:1`). Any QTSP can issue the credential without DNS dependency.

### 1.2 `blockchain_logo` removed from signed claims

The `blockchain_logo` field was present in the signed credential payload in v0.6.5. This is incorrect: display metadata (logos, labels) belongs in the `vct` type metadata document or in the wallet's CAIP-2 chain registry, not in the signed attestation. Including it in signed claims means every logo URL change requires credential re-issuance.

v0.7.1 removes the field from the signed payload entirely.

### 1.3 Revocation — now mandatory

v0.6.5 does not specify a revocation mechanism. v0.7.1 adds **mandatory StatusList2021** support, with a `revocation_id` claim in the credential. Section 18 specifies a 6–12 month credential validity period, revocation triggers (address compromise, account closure, regulatory hold), and re-issuance flow.

---

## 2. Transaction Data Model

### 2.1 `from_address` — new mandatory field

v0.6.5 does not include the payer's sending address in the transaction data payload. This creates two problems:

- The gateway cannot verify that the blockchain transaction was sent from the address proven in the (Q)EAA — the core security property of the architecture.
- TFR (Transfer of Funds Regulation) requires originator identification. Without `from_address` in the signed payload, there is no cryptographic link between the payer's identity and the originating address.

v0.7.1 adds `from_address` as **MANDATORY**. The gateway MUST verify that it matches `account_address` in the payer's (Q)EAA. This field is included in the transaction data hash bound into the KB-JWT, satisfying dynamic linking requirement DL-3.

### 2.2 Amount encoding

v0.6.5 encodes amount as a bare JSON number: `"amount": 10`. JSON numbers use IEEE 754 floating point, which loses precision for large token values (e.g., USDC with 6 decimals at high amounts).

v0.7.1 uses a decimal string in the smallest unit (e.g., `"amount": "10000000"` for 10 USDC) plus an `amount_display` string (e.g., `"10.00 USDC"`) for wallet rendering. This is consistent with how ISO 20022 handles amounts and avoids float rounding.

### 2.3 Authorization Request: Presentation Definition → DCQL

v0.6.5 uses a Presentation Definition for the OID4VP Authorization Request. The ARF (and TS12's referenced OID4VP profile) has since adopted **DCQL (Digital Credentials Query Language)** as the standard query format.

v0.7.1 replaces the Presentation Definition with a `dcql_query` object specifying minimum necessary claims from each credential type. This also enables cleaner binding: the `transaction_data[].credential_ids` array references the DCQL credential `id` to bind the dynamic linking hash specifically to the SCA credential's KB-JWT, not the PID.

---

## 3. Protocol Flows

### 3.1 Credential issuance flow — new (Section 5a)

v0.6.5 says nothing about how the Proof of Crypto Account Ownership credential is issued. v0.7.1 adds a full OID4VCI issuance flow including:

- PID-based identity verification at issuance time
- Challenge-response for address control: the issuer sends a nonce, the wallet signs it with the blockchain key (EVM: `personal_sign`; Tezos: Micheline encoding; ERC-1271 for smart contract wallets)
- Nonce replay protection at the issuer

This section addresses the root of trust: without specifying how address ownership is proven at issuance, the credential's core security claim is unverifiable.

### 3.2 Dynamic Linking — explicit requirements (Section 14)

v0.6.5 references TS12 for dynamic linking without specifying how it applies to crypto transfers. v0.7.1 adds **five explicit requirements (DL-1–DL-5)** derived from TS12 §3.3 and PSD2-RTS Article 5:

- **DL-1** — WYSIWYS: wallet MUST display `amount_display`, `asset.symbol`, `payee.account_address`, `from_address`, and `caip2_chain_id` before consent
- **DL-2** — Transaction data hash MUST be bound in the KB-JWT (`transaction_data_hashes`)
- **DL-3** — `from_address` MUST be present in the hash and match the (Q)EAA `account_address`
- **DL-4** — `caip2_chain_id` MUST match between the payload, the (Q)EAA, and the eventual on-chain transaction
- **DL-5** — KB-JWT `amr` MUST contain ≥2 authentication categories (REQUIRED by TS12 §3.6)

### 3.3 Replay protection — grounded in TS12 (Section 15)

v0.6.5 has no replay protection. An earlier draft of v0.7.x introduced `request_nonce` and `consent_timestamp` as transaction data fields — but these are redundant with mechanisms TS12 already mandates.

v0.7.1 removes both fields and grounds replay protection entirely in TS12 §3.6:

- **`jti`** (REQUIRED by TS12) — the PSD2 Authentication Code. A fresh cryptographically random value, unique per presentation. The gateway MUST treat it as single-use (RP-1).
- **`nonce`** (OID4VP) — session binding. The gateway's `nonce` in the Authorization Request MUST match the `nonce` in the KB-JWT (RP-2).
- **`iat`** (MAY per TS12) — the gateway SHOULD reject responses with KB-JWT `iat` older than 5 minutes (RP-3, RECOMMENDED not MUST).

This approach introduces no new payload fields and is fully conformant with TS12.

### 3.4 Required KB-JWT claims — now explicit

TS12 §3.6 specifies three additional REQUIRED KB-JWT claims beyond what base OID4VP mandates. v0.6.5 does not mention them; v0.7.1 makes them explicit throughout:

- **`jti`** — REQUIRED, PSD2 Authentication Code
- **`amr`** — REQUIRED, authentication methods references (drives DL-5)
- **`response_mode`** — REQUIRED, value of the `response_mode` from the Authorization Request

---

## 4. New Sections Not Present in v0.6.5

### 4.1 P2P Mutual Authentication (Section 16)

v0.6.5 covers only P2M (person-to-merchant). For **peer-to-peer transfers** above the MiCA/TFR €1,000 threshold, v0.7.1 adds an optional extension requiring the receiver to present their own Proof of Crypto Account Ownership (Q)EAA before the sender proceeds. This enables bidirectional TFR originator + beneficiary identification without a merchant gateway.

### 4.2 Split-Wallet Model (Section 16a)

Government wallets such as France Identité refuse to host third-party credentials (including crypto (Q)EAAs). v0.7.1 addresses this with a **two-session OID4VP flow**: the gateway first requests the PID from the government wallet, then initiates a separate SCA request to a second wallet holding the crypto (Q)EAA. The two sessions are correlated by `transaction_id`.

This is weaker than cryptographic co-presentation (an open issue: OPEN-6) but is compliant with Art. 5f(2), which does not require single-wallet co-presentation.

### 4.3 On-Chain Consent Anchoring — specified per implementation option (Section 17)

v0.6.5 briefly mentions that `transaction_id` could be embedded in calldata but does not specify how. v0.7.1 defines the encoding per implementation option:

- **Native transfers:** `tx.data = 0x455544495f504159 ("EUDI_PAY") || transaction_id_bytes`
- **ERC-20 event-based:** filter `Transfer` events by `(from, to, amount)` within a time window after KB-JWT `iat` — no calldata change needed
- **`transferFrom` / `permit`:** gateway contract emits `EUDIPayment(bytes32 transactionId, ...)` event (potential ERC — OPEN-5)
- **ERC-3009:** `transaction_id` maps to the authorization `nonce` via `keccak256`

An optional **consent hash** (SHA-256 of `presentation_response_bytes || kb_jwt_jti || kb_jwt_iat`) is defined for high-assurance anchoring where a smart contract stores a commitment to the off-chain consent.

### 4.4 Credential Lifecycle and Revocation (Section 18)

Specifies 6–12 month credential validity, revocation triggers (address compromise, account closure, regulatory hold), and OID4VCI re-issuance flow.

### 4.5 Open WG Issues (Section 19)

v0.7.1 surfaces **7 open issues** for the TS12 WG:

| Issue | Topic |
|-------|-------|
| OPEN-1 | mDoc `DeviceAuthentication` profiling for crypto SCA |
| OPEN-2 | Issuer qualification path — QTSP vs. non-qualified EAA (PoC uses non-qualified) |
| OPEN-3 | MiCA/TFR threshold handling and `required_verification_level` field |
| OPEN-4 | ERC-1271 smart contract wallet address binding at issuance |
| OPEN-5 | Standardisation of `EUDIPayment` contract event as a potential ERC |
| OPEN-6 | Cryptographic binding across wallets in split-wallet model |
| OPEN-7 | Gas cost disclosure in wallet UI before consent |

---

## 5. What Was Kept from v0.6.5

The following elements from the v0.6.5 spec are preserved without change:

- **CAIP-2 chain IDs** (`"caip2_chain_id": "eip155:1"`) — chain-agnostic, covers non-EVM chains including Tezos
- `cnf.jwk` binding model — device key as holder binding in the SD-JWT VC
- `payee` object structure — `name`, `id`, `logo`, `website`, `account_address`
- `asset.decimals` field
- `transaction_id` concept for off-chain / on-chain reconciliation
- Annex A — four payment gateway implementation options (event-based, `transferFrom`, ERC-2612 permit, ERC-3009)
- Two-QR-code UX flow (EUDI Wallet first, crypto wallet second)
- Architecture principle: no custodial intermediary, gateway does not hold funds or sign transactions
- Regulatory framing — eIDAS 2.0, MiCA, TFR, GDPR, PSD2/3
- Digital Euro readiness section

---

_Prepared for APTITUDE WP6 · March 2026_
