---
layout: post
title: "Engineering Notes: Device Attestation, Passkeys and Step-Up Auth for a Mobile Wallet"
date: 2026-10-31 00:00:00 +0000
categories: ["Security", "Mobile"]
tags: ["security", "passkeys", "mobile", "authentication"]
description: "Engineering notes on ewallet app development security: device attestation, passkeys, step-up authentication, key management and operator controls."
image: "https://cdn.sanity.io/images/563mnkns/production/ad26034b245b76f40d4dcd9afa1714186a4bca50-1600x1055.jpg"
author: "TechCirkle"
---

![Person holding a smartphone that shows a mobile wallet app with a fingerprint authentication icon](https://cdn.sanity.io/images/563mnkns/production/ad26034b245b76f40d4dcd9afa1714186a4bca50-1600x1055.jpg)

These are working notes from our eWallet app development projects on the security controls that sponsor banks and regulators ask about before a wallet holds a single customer dollar or pound. They are organised the way we review them internally: the client, authentication, risky actions, data protection, operators and testing. They are not exhaustive, but each item has come up in a real due-diligence conversation.

## 1. Trust the device only as far as you can verify it

**Goal:** the backend should know it is talking to a genuine, unmodified app on a real device.

- Use **Play Integrity** on Android and **App Attest** on iOS.
- Request attestation at registration and on high-risk events, not just once at install.
- Bind the attestation result to the session and to the device key, so a valid token cannot be replayed from another device.
- Treat attestation failure as a risk signal that feeds the fraud engine, rather than a hard block in every case. Older or unusual devices exist; emulators and tampered builds should score high.
- Keep the client free of secrets. The only keys on the device should be device-bound keys generated in the secure enclave or hardware-backed keystore.

**Implementation note:** even in a Flutter or React Native app, write the attestation, key generation and biometric modules natively. These are the parts where cross-platform abstractions most often leak.

## 2. Replace SMS codes with passkeys or device-bound keys

**Goal:** remove the shared secrets that phishing and SIM-swap exploit.

- Prefer **passkeys** (WebAuthn / FIDO2) where the market and user base allow.
- Alternatively, generate an asymmetric keypair on the device during onboarding, register the public key server-side and sign challenges with a biometric-gated private key.
- Keep SMS one-time codes only as a fallback, and treat fallback logins as lower assurance in risk scoring. SIM-swap is still a leading takeover vector.
- Plan device change carefully. "New phone" is the moment attackers target, so re-binding a device should require strong verification and should temporarily lower limits.

## 3. Step-up authentication on risky actions

**Goal:** a valid session is not enough for every action.

Require fresh, strong authentication for:

- Adding a new payee.
- Transfers above a configured threshold.
- Changing phone number or email.
- Logging in on a new device.
- Changing security settings or recovery methods.

```
action request -> risk engine score
  low    -> proceed
  medium -> step-up (passkey / biometric)
  high   -> step-up + hold for review
```

Keep thresholds in configuration owned by risk and compliance. In markets with strong customer authentication rules, map these flows explicitly to the regulatory requirements so you can show the mapping during audits.

![A man's hand making a digital payment on a mobile phone](https://cdn.sanity.io/images/563mnkns/production/397309b553926e9da7661b53f3532fca4849bf4f-1600x1068.jpg)

## 4. Transport and API protection

- Terminate mutual TLS or use certificate pinning at the gateway / BFF layer.
- Rate-limit per device, per user and per IP, with tighter limits on authentication and payee endpoints.
- Sign sensitive requests with the device key so the server can verify the request came from the bound device.

## 5. Data protection and key management

- Encrypt data at rest with keys held in a managed **HSM or KMS**.
- Apply **field-level encryption** to identity documents and other sensitive personal data, so a database read alone exposes nothing useful.
- Rotate secrets on a schedule and on staff changes.
- Minimise PCI DSS v4.0 scope by never handling raw card numbers. Use the processor's vault and network tokens so internal systems see only tokens.
- Isolate the ledger database with the strictest network and access controls in the platform.

## 6. Operator controls

Insider risk is a big part of what due-diligence teams probe.

- Least-privilege roles for support, compliance and engineering staff.
- **Maker-checker** on every manual ledger adjustment: one person proposes, another approves.
- **Immutable audit logs** of every operator action, including reads of sensitive data.
- No direct production database access for routine work. If staff need to fix something, the operations console must provide a controlled way to do it.

## 7. AI components and permissions

If the wallet includes an AI support assistant or analyst copilot:

- Ground it in the user's own data through read-only tools by default.
- Never give it tools that move money or close compliance cases.
- Require step-up authentication for any user-initiated action it helps prepare.
- Log every tool call with inputs, outputs and model version.

These constraints are the core of how we approach [LLM integration](https://techcirkle.com/llm-integration) in regulated products.

## 8. Testing and disclosure

- External penetration test before launch and after every major release.
- Mobile-specific testing: tampered builds, rooted and jailbroken devices, hooking frameworks and replayed attestation tokens.
- A bug bounty programme once real users are on the platform.
- Track and close findings with owners and deadlines, since partners will ask for evidence.

## Further reading

These notes cover one slice of the system. The full TechCirkle guide to [eWallet app development](https://techcirkle.com/blog/ewallet-app-development) covers the ledger, architecture, compliance in the US, UK and UAE, and cost. Our [mobile app development](https://techcirkle.com/development/mobile-app-development) team builds these controls into wallets from the first sprint.

## Frequently Asked Questions

### What is device attestation and why does a wallet need it?

It is a platform check, such as Play Integrity or App Attest, that tells the backend whether it is talking to a genuine, unmodified app on a real device. It helps block tampered apps and emulators.

### Are passkeys better than SMS one-time codes for wallets?

Generally yes. Passkeys resist phishing and are not exposed to SIM-swap attacks, which remain a common takeover route for SMS-based logins.

### Which actions should trigger step-up authentication?

New payees, large transfers, phone or email changes, new device logins and changes to security or recovery settings.

### How can a wallet reduce PCI DSS scope?

By never touching raw card numbers and relying on the processor's vaulting and network tokens, so internal systems only handle tokens.

### What is maker-checker and why does it matter?

It is a two-person rule where one staff member proposes a sensitive change, such as a ledger adjustment, and another approves it. It reduces insider risk and supports audits.

### Should cross-platform wallet apps use native security modules?

Yes. Attestation, key generation, biometrics and NFC are best implemented natively, even when the rest of the app is cross-platform.
