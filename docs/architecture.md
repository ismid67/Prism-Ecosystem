# PrismPass — Architecture

**Invention Disclosure v19 · Priority date 25 April 2026**  
**Zenodo DOI:** [10.5281/zenodo.20029291](https://doi.org/10.5281/zenodo.20029291)  
**Author:** I. Smid-Woelders, Zwolle, Netherlands

---

## Core principle

PrismPass inverts the authentication model. Instead of a server knowing the user, the user's device proves to the server that the entity is valid — without ever revealing who they are. The server receives one thing only: a cryptographic yes or no.

No username. No password. No biometric data leaving the device. No login history stored centrally. No linkage between sessions of the same user.

**The key design claim: what does not exist cannot be stolen.**

---

## The authentication triangle

PrismPass combines three factors. All three are owned by the user. None leaves the device.

| Factor | What it is | Ownership |
|--------|-----------|-----------|
| Biometrics | Fingerprint or face via WebAuthn / FIDO2 | User. Never leaves the device. |
| Device binding | Device seed in Secure Enclave / TEE | User. Hardware-protected. |
| Physical presence | Time-bound NFC nonce via passive tag (card, ring, sticker). Valid max 500 ms. | User. Proves physical possession. |

An ephemeral key is generated per session from the combination of all three factors. The server stores only a public commitment: no name, no email address, no biometric data. After each session, the ephemeral key is discarded.

The NFC factor proves physical presence by design. A passive NFC tag has a practical read range of 4 to 10 cm. Combined with a 500 ms nonce window, relay attacks are structurally made difficult by design — a deliberate security choice analogous to the logic of a bank card.

---

## What the server never sees

| Never received by server | Received by server |
|--------------------------|-------------------|
| Name, email, password, biometric data | Valid or invalid (ZKP proof) |
| Login history or session patterns | Encrypted blobs (contents unreadable) |
| Linkage between sessions of the same user | Session token (temporary, not persistent) |
| Location or movement data | Timestamp of session (no identity data) |
| That two devices belong to the same owner | Two separate commitment hashes — linkage exists only locally on the devices |
| Contents of the user's blackbox | Encrypted blob ciphertext, unreadable without PRF key derivation on user's device |

---

## Ecosystem components

### Core protocol

These components are inseparably tied to the triangle architecture. Splitting them off would lose the security guarantee.

| Component | Function | Status |
|-----------|----------|--------|
| PrismPass | Authentication. The triangle itself. | PoC proven 25 April 2026 |
| PrismID | ZKP attribute proof. Confirms criteria without revealing identity. | Working demo + credential architecture |
| PrismShield / Centroid | Behavioural baseline. Detects coercion and involuntary action. | Hypothesis + PPG demo. Requires independent validation. |
| PrismAir | Identity-free storage. Encrypted blobs, server does not know the owner. | PoC proven 9 May 2026 |
| PrismGate | Physical access control without central identity storage. | PoC proven 24 May 2026 |
| PrismPin | Recovery protocol. Restores access after device loss without server knowing identity. | PoC proven 1 June 2026 |
| PrismWipe | Secure identity dissolution. Removes triangle without server knowing who is leaving. | Concept, no PoC yet |
| PrismAdd | Anonymous interest layer. Relevance without profile. Adoption incentive for commercial parties. | Working demo |
| Blackbox | Three-layer encrypted container. AES-GCM-256 with WebAuthn PRF key derivation. Shadow blob for scope limiting. | Architecture documented June 2026 |

### Standalone building blocks

These primitives were designed within the ecosystem but do not require the triangle to operate. Third parties can implement them independently.

| Building block | What it enables |
|----------------|----------------|
| PrismHash | Findable without a platform account. A single HTML meta-tag. No registration, no platform dependency. |
| AI Verification Layer | Fairer claim processing. Document never leaves the browser. Applicable to any organisation with a dispute process. |
| De Twee Schuifjes | Opt-in AI training per conversation. Anonymous via Privacy Pass token. Two independent choices per conversation. |

### Demonstrators

Applications that show what the ecosystem makes possible. Not core components; intended to make the architecture tangible for partners and reviewers.

| Demonstrator | What it demonstrates |
|--------------|---------------------|
| PrismShop | Anonymous e-commerce. No customer profile. Delivery address deleted after shipping. |
| PrismGate (demo mode) | ZKP-based credential issuance for organisations. Shows PrismID in a practical scenario. |

---

## Technical standards

| Layer | Current implementation | Post-quantum migration path |
|-------|----------------------|----------------------------|
| Authentication | WebAuthn Level 3 / FIDO2 | Unchanged — hardware standard |
| Key exchange | ECDH (P-384) | ML-KEM-768 (NIST FIPS 203) |
| Signatures | ECDSA (P-384) | ML-DSA-65 (NIST FIPS 204) |
| Symmetric encryption | AES-256-GCM | Unchanged — already quantum-resistant |
| Zero-Knowledge Proofs | circom/snarkjs Groth16 / BN128 | STARK or post-quantum Bulletproofs |
| Anonymous tokens | Privacy Pass RFC 9576/9578 | Unchanged |

> Post-quantum migration (ML-KEM-768, ML-DSA-65) is a documented architectural design decision for forward compatibility. It is not a currently implemented feature. The working implementation uses ECDH, ECDSA, AES-256-GCM, and Groth16.

---

## Live infrastructure — PoC status

All core components have been demonstrated on live servers.

| Component | Live endpoint |
|-----------|--------------|
| PrismPass | prismpass.globalsecurity.nu |
| PrismID | prismid.globalsecurity.nu |
| PrismGate | prismgate.globalsecurity.nu |
| PrismAir | prismair.globalsecurity.nu |
| PrismShop | prismshop.globalsecurity.nu |
| PrismEco (ecosystem demo) | prismeco.globalsecurity.nu |
| NFC tap relay | tap.globalsecurity.nu |

Implementation: Node.js with SimpleWebAuthn v13, circom/snarkjs (Groth16, BN128), @cloudflare/privacypass-ts, WebAuthn PRF extension, HKDF, and AES-GCM-256.

---

## Positioning

The novelty of PrismPass lies in the combination, not in individual components. WebAuthn exists. ZKP exists. Privacy Pass exists. NFC exists. The claim is that the specific architecture — a three-factor ephemeral key system with a no-storage server model, a behavioural baseline layer, a contested-signal mechanism, and a coherent privacy-by-design philosophy — has not been combined in this way before.

| System | Difference with PrismPass |
|--------|--------------------------|
| Passkeys / FIDO2 | PrismPass adds: NFC physical presence, zero server storage, ZKP attribute proof, behavioural baseline. |
| Yivi (iDIN-based) | PrismPass proves you are valid without ever recording who you are. |
| EUDI Wallet / eIDAS 2.0 | Complementary, not competing. The wallet manages credentials; PrismPass manages the authentication layer. |
| DIDs / Verifiable Credentials | PrismPass implements these principles in a working PoC with a coherent UX and adoption model. |

---

*Full Invention Disclosure: [Zenodo DOI 10.5281/zenodo.20029291](https://doi.org/10.5281/zenodo.20029291)*
