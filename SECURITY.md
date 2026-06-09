# Security Policy

**Prism Ecosystem Protocol Specification**
*I. Smid-Woelders · Zwolle, Netherlands · 2026*

---

## Scope of this repository

This repository contains the **protocol specification** only.
No server-side implementation code is published here.
The working proof-of-concept runs at a separate, private repository.

Security issues in the specification (design flaws, architectural weaknesses,
incorrect claims about cryptographic properties) are in scope for this policy.

Security issues in the live proof-of-concept environment
(prismpass.globalsecurity.nu) are also in scope.

---

## Reporting a vulnerability

Please **do not** open a public GitHub issue for security vulnerabilities.
A public issue discloses the problem before it can be assessed or addressed.

Report security concerns by email:

**ietjesmid@gmail.com**

Include in your report:

- Which component or specification section is affected
- A description of the potential vulnerability
- If applicable: a proof-of-concept or reproduction steps
- Your preferred level of acknowledgement (named credit, anonymous, or none)

---

## Response

You will receive an acknowledgement within 5 business days.
Assessment and follow-up will happen as quickly as the nature of the issue allows.

This is an independent project without a dedicated security team.
Responses may take longer during periods when external review or funding processes are active.

---

## Out of scope

The following are out of scope for this policy:

- Vulnerabilities in the underlying open standards (WebAuthn, circom/snarkjs,
  Privacy Pass, NFC APIs). Please report those to the respective standards bodies.
- Theoretical attacks that require physical access to both the device and the physical NFC tag simultaneously. The threat model acknowledges this boundary explicitly.
- Issues in third-party dependencies not specific to the Prism Protocol architecture.

---

## Disclosure philosophy

Responsible disclosure is appreciated and respected.
The inventor will not pursue legal action against researchers who report in good faith
and allow reasonable time for assessment before public disclosure.

Coordinated disclosure timelines can be discussed on a case-by-case basis.

---

*Security Policy v1.0 · Prism Ecosystem · I. Smid-Woelders · June 2026*
