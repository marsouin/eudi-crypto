# Secure Crypto Payments with the EUDI Wallet

Last update: March 2026

This repository provides a **reference architecture and specification** describing how the **European Digital Identity Wallet (EUDI Wallet)** can enable **secure and compliant crypto-asset payments** while preserving the decentralized nature of blockchain settlement.

The primary scenario described is a **person-to-merchant (P2M) crypto payment**, where a natural person pays a merchant using a **self-custodial crypto wallet**, while the **EUDI Wallet provides the trusted identity, authentication, and consent layer**.

The architecture demonstrates how **identity-verified crypto payments** can be implemented in alignment with European regulatory frameworks including **eIDAS 2.0, MiCA, GDPR, and ARF TS12 Strong Customer Authentication**.

## Specification Documents

| Document | Description |
|----------|-------------|
| **[Secure_Crypto_P2M_EUDI_Use_Case_v0.7.1.md](Secure_Crypto_P2M_EUDI_Use_Case_v0.7.1.md)** | Current specification |
| **[Secure_Crypto_P2M_EUDI_Use_Case.md](Secure_Crypto_P2M_EUDI_Use_Case.md)** | v0.6.5 baseline |
| **[changes_v065_to_v071.md](changes_v065_to_v071.md)** | Changelog — what changed from v0.6.5 to v0.7.1 and why |

---

# Overview

The growth of **digital assets, stablecoins, and decentralized finance** is transforming digital commerce. However, large-scale adoption in Europe requires solutions that combine:

- regulatory compliance
- strong authentication
- privacy-preserving identity verification
- interoperability with the **European Digital Identity framework**

This project illustrates how **EUDI Wallets and self-custodial crypto wallets can work together** to enable **secure, identity-verified blockchain payments**.

In this architecture:

- the **EUDI Wallet** performs identity authentication and transaction consent
- the **crypto wallet** performs the blockchain transaction
- a **payment gateway** orchestrates the payment interaction
- the **blockchain acts purely as the settlement layer**

No intermediary holds funds or executes blockchain transactions on behalf of the user.

---

# Architecture Principles

The architecture separates **identity verification, payment authorization, and transaction execution** into independent layers.

## Identity and Authentication

The **EUDI Wallet** acts as the **identity and consent layer**.

It allows the payer to:

- authenticate using **PID or equivalent identity credentials**
- present **Proof of Crypto Account Ownership credentials**
- provide **explicit transaction consent**
- sign authorization using **advanced electronic signatures**

Authentication follows the **Strong Customer Authentication (SCA)** framework defined in **EUDI ARF TS12**.

## Payment Execution

The blockchain transaction is executed **directly by the payer** using their **self-custodial crypto wallet**.

This ensures:

- full control of funds by the user
- no custody by intermediaries
- decentralized blockchain settlement

The **payment gateway** monitors the blockchain and confirms settlement once the transaction is included on-chain.

## Architecture & Trust Model

The following diagram illustrates the high-level architecture and trust relationships between the payer, the merchant, the payment gateway, the EUDI identity system, and the blockchain settlement layer.

Identity authentication and transaction consent are handled off-chain through the **EUDI Wallet**, while the **blockchain acts purely as the decentralized settlement layer**.  
The **payment gateway orchestrates the interaction** between the parties without holding funds or executing blockchain transactions.

![EUDI Crypto Payment Architecture](architecture.png)

---

# Roles and Actors

## Payer (Natural Person)

The payer:

- holds an **EUDI Wallet**
- holds a **self-custodial crypto wallet**
- possesses identity credentials such as **PID**
- holds a **Proof of Crypto Account Ownership credential**

The payer authenticates with the EUDI Wallet and executes the payment using their crypto wallet.

## Merchant (Legal Person)

The merchant:

- provides goods or services
- holds a **receiving crypto wallet address**
- may disclose **verifiable credentials proving legal identity**

Merchant identity information can be displayed to the payer during the authorization process.

## Payment Gateway

The payment gateway acts as the **orchestration and verification layer**.

Its responsibilities include:

- generating structured payment requests
- acting as a **Relying Party (RP)** in OpenID4VP authentication
- verifying identity attestations returned by the EUDI Wallet
- monitoring the blockchain to confirm payment settlement

The gateway **does not hold funds** and **does not execute blockchain transactions**.

## Blockchain Network

The blockchain serves purely as the **decentralized settlement layer** for the payment.

Supported networks may include:

- Ethereum and other EVM chains
- Tezos
- other compatible distributed ledger technologies

---

# Strong Customer Authentication (TS12)

Strong Customer Authentication is implemented through a **Proof of Crypto Account Ownership credential**, issued as an **Electronic Attestation of Attributes (EAA)** — or a **Qualified EAA (QEAA)** when issued by a Qualified Trust Service Provider.

This credential proves that the user controls a specific blockchain address without exposing private keys.

During the **OpenID4VP authentication flow**, the payer presents:

- an identity credential (PID or equivalent)
- the **crypto account ownership credential**

The EUDI Wallet binds these credentials to the **transaction context** and returns a **verifiable presentation** confirming the authorization.

---

# Trust Model

Trust in the system is established through three complementary components:

### Wallet → Payment Gateway

The EUDI Wallet verifies that the gateway is an **authorized relying party** within the EUDI trust framework.

### Merchant → Wallet Attestations

The merchant relies on:

- verifiable identity credentials
- proof of crypto account ownership
- user-generated signatures

### Blockchain Settlement

The blockchain provides a **publicly verifiable settlement layer** confirming that the payment has been executed.

---

# Regulatory Alignment

The architecture aligns with major European regulatory frameworks.

### eIDAS 2.0

- EUDI Wallet authentication
- Qualified Electronic Attestations of Attributes
- advanced electronic signatures

### MiCA

- compliant merchant acceptance of crypto-assets
- identity-assured crypto payments

### TFR (Travel Rule)

- selective disclosure of originator information
- compliance support when required

### GDPR

- privacy-by-design architecture
- no personal data written on-chain
- selective disclosure mechanisms

### PSD2 / PSD3 Principles

Although PSD2 does not regulate peer-to-peer crypto transfers, the architecture follows key principles:

- Strong Customer Authentication
- explicit user consent
- dynamic linking of transaction parameters

---

# Future Extensions

The architecture may be extended to additional scenarios, including:

- **person-to-person crypto payments**
- **merchant-to-merchant payments**
- **legal person wallets**
- **Digital Euro payment integration**

Additional blockchain networks and wallet implementations may also be supported.
