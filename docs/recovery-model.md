# PrismPass — Recovery Model

**Invention Disclosure v19 · Priority date 25 April 2026**  
**Zenodo DOI:** [10.5281/zenodo.20029291](https://doi.org/10.5281/zenodo.20029291)  
**Author:** I. Smid-Woelders, Zwolle, Netherlands

---

## The fundamental tension

PrismPass stores no personal data on the server. The server has nothing to recover from. This is the privacy guarantee — and it is also the reason recovery requires a deliberate design.

When a device is lost, the Secure Enclave key is gone with it. That is intentional: the key is the security. Recovery works through two parallel mechanisms that reinforce each other, both of which keep the server out of the role of custodian.

---

## The blackbox

The blackbox is the encrypted container that holds everything belonging to the user and nothing else. Not a product — an architectural primitive.

### Three layers

| Layer | Contents | Storage | Recoverable? |
|-------|----------|---------|--------------|
| Layer 1 | Private key (Secure Enclave) | OS hardware, not exportable | No. Always regenerate. |
| Layer 2 | Personal data: preferences, session memory, purchase history, photos, documents | Local by default. Backup as encrypted blob. | Yes, via recovery vault. |
| Layer 3 | Credential links: which services know this device | Partly local, partly as revocable token at the service | Yes, via reissuance or recovery vault. |

Layer 1 is the key to the blackbox, not the blackbox itself. Layer 2 is the core. Layer 3 is the access card to the outside world.

### How the blob is constructed

The blackbox is an AES-GCM-256 encrypted file. The key for this file does not exist anywhere as a whole — it is derived via WebAuthn PRF at the moment of use.

```
[header]
  version         — protocol version number
  blob-id         — unique identifier (random, not traceable to owner)
  iv              — initialisation vector for AES-GCM (12 bytes, random per session)
  created_at      — creation timestamp (stored locally, not sent to server)

[encrypted payload]
  JSON or binary data — everything in layers 2 and 3

[authenticator tag]
  AES-GCM integrity tag — tamper detection
```

Key derivation:

| Step | Mechanism | Requirement |
|------|-----------|-------------|
| 1 | WebAuthn PRF extension on the registered device | Biometrics on device |
| 2 | HKDF of the PRF output with blob-id as salt | No extra action |
| 3 | Resulting AES-256 key — in memory, never on disk | Disappears when session closes |

**What the server never sees:**

| Server receives | Server never sees |
|-----------------|-------------------|
| Encrypted blob (ciphertext) | Key or PRF output |
| Blob-id (randomly generated) | Name or identity of owner |
| AES-GCM authenticator tag | Contents of the blob |
| ZKP proof of blob ownership (optional) | Link between blob and person |

---

## Recovery mechanism A: recovery vault

At registration, the user generates a recovery vault: an encrypted package containing the blackbox key, encrypted with a derivative of the recovery PIN. The package is worthless without the PIN.

| Step | Action | Who is involved |
|------|--------|-----------------|
| 1 | User retrieves recovery vault (USB, NAS, own cloud, or PrismAir relay) | User |
| 2 | Enters recovery PIN on new device | User |
| 3 | Enters biometrics on new device | User + OS |
| 4 | Blackbox key is derived and blackbox loaded | Device (local) |
| 5 | New device registered; old device deregistered | Device + server (no personal data involved) |

The server is not involved at any step as owner or custodian of data.

**Design note on Argon2id vs HKDF:** the recovery PIN derivation uses Argon2id for password-based key derivation (slow by design, resists brute force) rather than HKDF (fast, suitable for high-entropy keys). The choice between these has implications for PIN length requirements and hardware cost on low-end devices. This is a documented open design point for production implementation.

---

## Recovery mechanism B: PrismAir as second copy

If the user has set up PrismAir, the encrypted blob is already on the relay. After device loss, only the anchor is lost. The anchor is recoverable via the recovery vault. Once restored, the blob on the relay is immediately accessible.

**Recommended backup strategy:**

- Primary copy: PrismAir relay (encrypted, identity-free)
- Backup copy: own cloud or USB (same encrypted blob)
- Anchor recovery: via recovery vault with recovery PIN

### Storage options for the blob

| Option | Description | Autonomy | Recommended for |
|--------|-------------|----------|-----------------|
| USB / local disk | Maximum control, maximum risk of losing the carrier | High | Extra copy alongside other options |
| Own NAS | Good for technically capable users; blob is encrypted | High | Advanced users |
| Own cloud (iCloud, Nextcloud) | Lowest threshold; cloud sees only ciphertext | Medium | Broad user group |
| PrismAir relay | PrismPass-native; server does not know whose it is; identity-free | High | Preferred in PrismPass context |

---

## Scope limiting: the shadow blob

The recovery vault gives full access to the owner's blackbox. For partial access — a next of kin, a caregiver, an executor — the shadow blob mechanism applies.

The shadow blob is a separately encrypted package containing only selected data. The owner determines scope at the time of creating the partial token.

```
At creation of partial token:
  1. Owner selects scope: which data, which period, which conditions
  2. System generates shadow blob: only selected data, encrypted with
     partial token key (HMAC of own commitment + scope hash)
  3. Server receives: partial token + encrypted shadow blob
  4. Server sees: two blind data points. No names, no contents.

At use by recipient:
  1. Recipient presents partial token
  2. Server checks: does this request fall within the scope definition?
  3. If yes: shadow blob is released
  4. The rest of the blackbox: inaccessible to the recipient
```

**Which layers fall within scope of a partial token:**

| Layer | In scope by default? | Can owner configure? | Notes |
|-------|---------------------|---------------------|-------|
| Layer 1 (private key) | Never | No | Structurally never leaves the Secure Enclave |
| Layer 2 (personal data) | Partially, owner's choice | Yes | Owner selects which data goes into the shadow blob |
| Layer 3 (credential links) | No, unless explicitly added | Yes, deliberately | Credentials give access to services; requires conscious choice |

**Practical example:** owner configures "photos from 2010 to 2025 and the will." Recipient sees exactly that data and nothing else. The server knows: a partial token was used, within scope. Not who. Not what was inside.

---

## Trusted-person mode

The partial token architecture supports a caregiver or family member helping with onboarding or recovery during the owner's lifetime.

- Owner configures scope: "my daughter may help me with the recovery process."
- This is not a backdoor: the owner determines the scope and receives an alert upon use.
- The architecture does not change. Only the permitted actions within the scope.

**Scope boundary:** the trusted-person mode is designed for person-owned products (PrismPass, PrismShop). It does not apply to organisation-issued credentials (PrismGate, PrismID-work). A dismissed employee must not be able to restore their credential.

---

## Explicit tradeoff

PrismPass maximises privacy at the cost of recoverability. This is a deliberate design choice.

**Forgotten PIN without a recovery vault = no recovery.** The responsibility lies entirely with the user. This is consistent with the philosophy that the server must not be a custodian of identity data, even in the role of recovery service.

Users who want a softer fallback can use the trusted-person mode, which keeps the responsibility distributed between the owner and a person of their own choosing — not a central service.

---

*Full Invention Disclosure: [Zenodo DOI 10.5281/zenodo.20029291](https://doi.org/10.5281/zenodo.20029291)*
