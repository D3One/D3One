

# 🔐 IKEv1 Aggressive Mode: Attack IPSec

**Author:** I.P. (c) 2017

---

## TL;DR of the linked piece (high-level):

The “Hacker”/Хакер write-up explains how **IKEv1 Aggressive Mode with PSK** exposes enough material in the handshake for a **passive capture → offline PSK cracking** workflow, and contrasts that with safer handshakes (Main Mode / IKEv2). Mitigations: kill Aggressive Mode, prefer IKEv2, use certs or high-entropy PSKs, and harden your gateways. ([Habr][1])

---

## 🎬 Cold-open (new): why you should care

If your VPN gateway still talks **IKEv1 Aggressive Mode + PSK**, you’re basically printing a “crack me later” postcard on the public internet. A lurker just needs one handshake capture and a GPU farm to take a swing at your shared secret while you sleep. That’s not “advanced threat”; that’s **script-kiddie speedrun**. The fix is not rocket science: **turn off Aggressive Mode**, move to **Main Mode** or better **IKEv2**, and stop treating PSKs like Wi-Fi passwords. RFCs and vendors have been waving this flag for a decade. ([IETF Datatracker][2])

---

## 🧠 Threat model in one breath

* **Protocol:** IKEv1, **Aggressive Mode**, **PSK auth**
* **What leaks:** hashes derived from PSK + nonces are observable **in the clear** during the fast 3-message setup
* **Attacker play:** **passive** sniff → **offline dictionary/brute-force** against the captured material → if PSK is weak or reused, **impersonation** is on the table
* **Safer alternatives:** IKEv1 **Main Mode** (protect peer IDs before auth), or **IKEv2** (no “modes,” tighter design), preferably with **certs** (RSA/ECDSA) rather than PSK. ([strongSwan][3])

> *Manager-speak translation:* Aggressive Mode externalizes enough info to make your VPN perimeter **password-equivalent**. Don’t be that org.

---

## 🧯 What **not** to do (and why)

* Don’t run **Aggressive Mode** on site-to-site. It’s there for edge cases (dynamic peers), not your backbone. **Ban it.**
* Don’t trust **short PSKs** (“Winter2025!”) or re-use them across tunnels. If PSK remains, make it **long, random, unique**.
* Don’t mix “legacy convenience” with “internet exposure.” Audit your gateways — you’ll be surprised how many still negotiate Aggressive.
  *Proof it’s a thing:* vendor advisories and scanners have warned on this since the 2000s; it’s still a Nessus plugin staple. ([Tenable®][4])

---

## 🛡️ Do this instead — hardening playbook (Cisco IOS/ASA, era-correct)

### 🚫 Kill Aggressive Mode globally

**Cisco IOS (classic):**

```cisco
! [2015-11-05 09:12:33] Policy hardening — no Aggressive Mode, ever.
crypto isakmp aggressive-mode disable
!
! Optional hygiene: keepalives to clean up dead SAs
crypto isakmp keepalive 10 periodic
```

*Why:* Blocks sending/accepting Aggressive Mode across all ISAKMP peers. Keepalives aid liveness. ([Cisco][5])

**Cisco ASA:**

```cisco
! [2015-11-05 09:13:41] Company standard — Aggressive Mode off.
crypto ikev1 am-disable
```

*Why:* Same idea, ASA syntax. One line, big win. ([Cisco][6])

---

### 🧱 If you must stay on IKEv1 → use **Main Mode** + strong suites

```cisco
! [2015-11-05 09:15:02] IKEv1 Phase 1, Main Mode, stronger knobs
crypto isakmp policy 10
 encr aes 256         ! modern for the era
 hash sha             ! SHA-1 at the time; use SHA-2 if your code supports it
 authentication pre-share
 group 14             ! DH group 14 (2048-bit MODP) was the sensible floor
 lifetime 86400
!
! Phase 2
crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha-hmac
 mode tunnel
!
! PFS to avoid key reuse across rekeys
crypto map CMAP 10 ipsec-isakmp
 set peer 198.51.100.20
 set transform-set TS-AES256
 set pfs group14
 match address ACL-HQ-BR1
```

*Why:* This is the “boring but correct” baseline: Main Mode, AES-256, DH14, PFS. It’s safe, predictable, and plays nice with 2012-ish code. ([NIST Computer Security Resource Center][7])

---

### ✅ Or skip the drama: move to **IKEv2** (preferably with certs)

```cisco
! [2015-11-05 09:18:09] IKEv2 skeleton (IOS)
crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha1
 group 14
!
crypto ikev2 policy IKEV2-POL
 proposal IKEV2-PROP
!
! Keyring & profile (use certs if possible; PSK shown for parity)
crypto ikev2 keyring KR-HQ
 peer BR1
  address 198.51.100.20
  pre-shared-key local HQ_SuperSecret_Long_Random
  pre-shared-key remote BR1_SuperSecret_Long_Random
!
crypto ikev2 profile PROF-HQ-BR1
 match identity remote address 198.51.100.20 255.255.255.255
 authentication remote pre-share
 authentication local  pre-share
 keyring local KR-HQ
!
! IPSec profile + VTI (route-based)
crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha-hmac
crypto ipsec profile IPSEC-HQ-BR1
 set transform-set TS-AES256
 set ikev2-profile PROF-HQ-BR1
!
interface Tunnel101
 description *** HQ<->BR1 over IKEv2 (2015 rollout) ***
 ip address 10.255.101.1 255.255.255.252
 tunnel source GigabitEthernet0/0
 tunnel destination 198.51.100.20
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC-HQ-BR1
```

*Why:* IKEv2 has no Aggressive/Main split and fixes many IKEv1 warts. VTI = clean routing; certs beat PSKs; same crypto suite story. ([IETF Datatracker][8])

---

## 🕵️‍♀️ Blue-team ops: what to **monitor** and **alert** on (no hack-steps)

* **Handshake mode:** Alert if any tunnel negotiates **Aggressive Mode** (syntax varies per platform/SIEM parser).
* **Policy drift:** flag DH group < 14, 3DES/MD5, missing PFS.
* **PSK hygiene:** changes outside CAB windows, multiple peers reusing the same PSK, or suspiciously short secrets.
* **Exposure surface:** internet-wide scanners & vuln feeds constantly flag Aggressive Mode; treat detections like a fire drill. ([Tenable®][4])

---

## 🧩 FAQ speed-round (manager edition)

* **“Is Main Mode enough?”**
  It’s far better than Aggressive (IDs protected). But if you can, **use IKEv2** and **certs** — that’s the long-term path. ([IETF Datatracker][8])

* **“Why does Aggressive exist then?”**
  Historical reasons (fast setup, dynamic peers). The trade-off is **security**. Most vendors let you **disable it** globally — do that. ([Cisco][5])

* **“Any official guidance?”**
  Yes: **NIST SP 800-77** (IPsec guide) + vendor docs + RFCs. The chorus is consistent: **avoid Aggressive Mode with PSK**. ([NIST Publications][9])

---

## 🧰 Verification (safe commands)

```cisco
! [2015-11-05 09:25:30] Status checks (IOS)
show crypto isakmp sa
show crypto ipsec sa
!
! [2015-11-05 09:26:02] ASA equivalents
show crypto ikev1 sa
show crypto ipsec sa
```

*Tip:* pipe to filters in your NOC tooling to catch any **AGGRESSIVE** negotiations or weak suites when they slip back in during change storms.

---

## 🎤 New closing: the “no-regrets” checklist

* **Ban Aggressive Mode** organization-wide. (IOS: `crypto isakmp aggressive-mode disable`; ASA: `crypto ikev1 am-disable`.)
* **Prefer IKEv2 + certificates.** If PSK remains, go **long & random**, per-tunnel, with rotation.
* **Raise the floor:** AES-256, DH-14+, **PFS on Phase 2**, ditch MD5/3DES.
* **Instrument it:** SIEM rules for handshake mode, suite strength, PSK reuse, and config drift.
* **Document it:** security standard + CAB guardrails so Aggressive Mode can’t “accidentally” come back. ([Cisco][5])

---

## 📚 References (English & Russian)

**Standards & guidance**

* **RFC 2409** — *The Internet Key Exchange (IKE)* (v1, Aggressive/Main). ([IETF Datatracker][2])
* **RFC 7296** — *IKEv2* (Internet Standard; replaces RFC 5996). ([IETF Datatracker][8])
* **NIST SP 800-77 r1** — *Guide to IPsec VPNs* (modern edition; core principles unchanged). ([NIST Publications][9])

**Vendor docs (how to disable Aggressive Mode)**

* Cisco IOS: `crypto isakmp aggressive-mode disable`. ([Cisco][5])
* Cisco ASA: `crypto ikev1 am-disable`. ([Cisco][6])

**Risk background & notes**

* strongSwan Security Recommendations — why IKEv1 Aggressive Mode + PSK is fundamentally flawed. ([strongSwan][3])
* CERT/CC VU#857035 (and CVE-2018-5389 context) — PSK dictionary attack considerations. ([kb.cert.org][10])
* Tenable/Nessus plugin 62694 — detection of IKEv1 Aggressive Mode with PSK. ([Tenable®][4])

**Russian**

* Журнал «Хакер» / Хабр: «Анатомия IPsec. Проверяем на прочность легендарный протокол шифрования» (обсуждает IKE и режимы, 2015). *(Used as a thematic source; not reproduced.)* ([Habr][1])

**Books (period-correct)**

* *CCNP Security VPN Official Cert Guide* (Cisco Press, 2012).
* *Cisco ASA: All-in-One Firewall, IPS, and VPN Services* (3rd ed., 2014).

---

## Attribution
[1]: https://habr.com/ru/companies/xakep/articles/256659/?utm_source=chatgpt.com "Анатомия IPsec. Проверяем на прочность легендарный ..."
[2]: https://datatracker.ietf.org/doc/html/rfc2409?utm_source=chatgpt.com "RFC 2409 - The Internet Key Exchange (IKE)"
[3]: https://docs.strongswan.org/docs/latest/howtos/securityRecommendations.html?utm_source=chatgpt.com "Security Recommendations"
[4]: https://www.tenable.com/plugins/nessus/62694?utm_source=chatgpt.com "Internet Key Exchange (IKE) Aggressive Mode with Pre-Shared ..."
[5]: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/security/a1/sec-a1-cr-book/sec-cr-c4.html?utm_source=chatgpt.com "Cisco IOS Security Command Reference: Commands A to C"
[6]: https://www.cisco.com/c/en/us/td/docs/security/asa/asa918/configuration/vpn/asa-918-vpn-config/vpn-ike.html?utm_source=chatgpt.com "CLI Book 3: Cisco Secure Firewall ASA Series VPN ..."
[7]: https://csrc.nist.gov/pubs/sp/800/77/r1/final?utm_source=chatgpt.com "SP 800-77 Rev. 1, Guide to IPsec VPNs | CSRC"
[8]: https://datatracker.ietf.org/doc/html/rfc7296?utm_source=chatgpt.com "RFC 7296 - Internet Key Exchange Protocol Version 2 ..."
[9]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-77r1.pdf?utm_source=chatgpt.com "Guide to IPsec VPNs - NIST Technical Series Publications"
[10]: https://www.kb.cert.org/vuls/id/857035?utm_source=chatgpt.com "VU#857035 - IKEv1 Main Mode vulnerable to brute force attacks"
