# PrismPass — Threat Model

**Invention Disclosure v19 · Priority date 25 April 2026**  
**Zenodo DOI:** [10.5281/zenodo.20029291](https://doi.org/10.5281/zenodo.20029291)  
**Author:** I. Smid-Woelders, Zwolle, Netherlands

> This is a condensed summary. The full threat model is documented in the Invention Disclosure on Zenodo (Appendix H).

---

## Core defence

The primary defence is architectural: there is nothing to steal on the server. Most attacks are neutralised before they begin because no personal data is stored centrally.

**Claim discipline throughout this document:** claims are described as "structurally made difficult by design," not as impossible. Absolute language is used only where a standard guarantees it. Three categories are distinguished:

- **Proven in PoC** — demonstrated on a live server with documented evidence
- **Follows from open standard** — behaviour guaranteed by a published specification (W3C, IETF, NIST)
- **Architectural design choice** — reduces risk by design; not yet independently validated

---

## 1. Attack surface overview

| Layer | Attack type | PrismPass position |
|-------|-------------|-------------------|
| Server | Data exfiltration / breach | Structurally resolved. Server stores no personal data. A database breach yields nothing usable. |
| Network | Man-in-the-middle / relay attack | Covered via nonce. Time-bound nonce (500 ms) makes relay attacks structurally difficult by design — same principle as TOTP / bank card logic. |
| Device | Compromised endpoint (malware / rooted phone) | Explicit scope boundary. Applies to every authentication protocol in the world. See section 2. |
| Biometrics | Spoofing (fake fingerprint / deepfake) | Hardware-dependent. PrismPass relies on Secure Enclave. Improves with hardware generations. |
| Cryptography | Quantum computer breaks RSA/ECDSA | Modularity: architecture provides a post-quantum migration path per layer. See architecture.md. |
| Implementation | ZKP library bug or backdoor | PoC risk; production requires audited library. circom/snarkjs is replaceable. |
| Recovery flow | Social engineering on recovery mechanism | Resolved by the triangle itself. No physical office required. The triangle is the safety net. |
| Availability | DDoS — making the chain unreachable | Does not affect confidentiality or integrity. Stateless design enables horizontal scaling. Active nonce state is an open design point. See section 5. |

---

## 2. Explicit scope boundary: compromised device

If the device itself is compromised — malware, rooted phone, OS-level keylogger — no authentication protocol in the world provides protection. This applies equally to Apple, Google, Microsoft, Signal, and PrismPass.

PrismPass takes the following explicit position:

- PrismPass protects against attacks on the server and the network.
- PrismPass protects against surveillance and data collection by the service provider.
- A compromised device falls outside scope — consistent with WebAuthn Level 3, FIDO2, and Apple Passkeys.

---

## 3. Security assumptions

PrismPass is secure under the following assumptions. Where an assumption does not hold, the protection is correspondingly limited.

| Assumption | Notes |
|------------|-------|
| Secure Enclave / TEE is intact | At hardware-level compromise, no protocol provides protection. |
| WebAuthn implementation is correct | W3C standard. PrismPass assumes the platform implementation (Apple, Google, Microsoft) is correct and uncompromised. |
| No OS-level compromise | Explicit scope boundary. Consistent with the industry norm. |
| Issuer acts with integrity | PrismID assumes the issuing party does not link credential issuance to an external identity profile. Mitigation: multiple issuers, no monopoly, jurisdiction choice. |
| ZKP circuit is correctly implemented | The circom/snarkjs Groth16 implementation has not been externally audited for the PoC. Production deployment requires a formal circuit audit. |

---

## 4. Formal privacy goals

| Privacy goal | What it means | Status in PrismPass |
|--------------|--------------|---------------------|
| Unlinkability | Two sessions of the same user are not linkable to each other. | Guaranteed via ephemeral keys and ZKP per session. |
| Minimal server knowledge | The server receives only what is necessary for verification. | Guaranteed: server receives only valid/invalid and credential ID. |
| Verifier blindness | The verifier does not know who the user is, only that the proof is valid. | Guaranteed for PrismPass and PrismGate. PrismID: verifier sees selected attributes, never full identity. |
| Issuer separation | The issuer does not know where and when a credential is used. | Guaranteed: verifier and issuer do not communicate. Issuer knows identity at issuance — explicit acknowledged assumption. |
| Local credential storage | Credentials and keys do not leave the user's device. | Guaranteed via WebAuthn Secure Enclave and local storage. |
| Attribute minimisation | Only the minimally required attribute is disclosed per context. | Guaranteed via ZKP circuits: verifier receives yes/no, never the underlying value. |

---

## 5. Availability and DDoS — open design point

Confidentiality and integrity are addressed above. Availability is a separate property.

An attacker who cannot steal data can still attempt to make the chain unreachable by flooding it. This does not affect the privacy model but does affect usability.

**The structural advantage:** there is no central personal data store that must be synchronised between server copies. Both the relay and the verification layer are therefore horizontally scalable. More copies means more capacity, without the copies needing to maintain a shared truth. This is the property a classical database server lacks.

| Layer | Availability profile | Notes |
|-------|---------------------|-------|
| Relay (tap) | Trivially replicable | Stateless. No database, no sessions. Identical copies behind geo-distribution or anycast. A request without a valid active nonce is cheaply rejected. |
| Verification server | Replicable, heavier per request | Load is cryptographic verification (computation), not lookup in central storage. Computation scales horizontally. A capacity and cost question, not an architectural dead end. |
| Active nonce | Open design point | A nonce must be issued and then invalidated; otherwise replay is possible. This is short-lived shared state that must be consistent across server copies. This is the genuine bottleneck under load. |

**Claim discipline:** PrismPass does not claim "DDoS-resistant" or "no single point of failure." The defensible formulation is: the stateless design makes horizontal scaling of both the relay and the verification layer possible, because there is no central personal data store requiring synchronisation. The remaining shared state (the active nonce) is named as an open design point for the availability layer.

---

## 6. PrismGate: physical access control

Physical access systems are normally strongly centralised. Systems such as HID, ASSA ABLOY, and Nedap link each access event to a person in a central database — name, role, timestamp, and location are logged per event.

PrismGate breaks this pattern. The verifier receives only valid or invalid. Name and role are architecturally invisible to the reader. The server stores no personal data — only a credential ID and an expiry date.

The combination of verifier blindness, minimal server knowledge, and ZKP-based credential verification without central identity registration is, to the best of current knowledge, not previously implemented in a working system for physical access control. PoC demonstrated 24 May 2026.

---

## 7. Architectural tradeoffs

Privacy systems always involve tradeoffs. PrismPass maximises privacy at the cost of recoverability. This is a deliberate design choice, not an omission.

| Tradeoff | What PrismPass chooses | What that costs |
|----------|----------------------|----------------|
| Privacy vs recoverability | Maximum privacy: no central storage, no external recovery party. | Forgotten PIN = no recovery without the recovery vault. Responsibility lies entirely with the user. |
| Anonymity vs Sybil resistance | Anonymity: no uniqueness requirement per person. | No cryptographic guarantee that one person holds only one identity. |
| Decentralised vs scalability | Compute on the device: no central processing server. | ZKP generation on weak hardware is slow. Scalability depends on device generation. |
| Unlinkability vs revocation | Sessions are not linkable, even by the server. | Fully unlinkable revocation is an open cryptographic question. |

---

*Full Invention Disclosure: [Zenodo DOI 10.5281/zenodo.20029291](https://doi.org/10.5281/zenodo.20029291)*
