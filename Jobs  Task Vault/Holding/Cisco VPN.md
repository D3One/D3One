
# 🌐 Cisco Site-to-Site VPNs + Secure Config Cookbook

**Author:** I.P. (2012)
**Scope:** Internet-based IPSec between a headquarters router/firewall and **three** branch offices using Cisco gear typical of **2008–2012** (IOS 12.4T/15.0M, ASA 8.x–9.1). Focus on **strong crypto**, **admin access hardening**, and **avoiding IKEv1 Aggressive Mode** pitfalls.

---

## 🧭 Preamble

This is a **mini-guide** with opinionated, battle-tested settings. It leans on **Cisco official references** and classic Cisco Press texts from the period. I’ve annotated every block with *what it does* and *why it matters*. Where the 2012-ish platforms allow it, I also show **IKEv2/FlexVPN** as the safer, cleaner alternative. ([Cisco][1])

---

## 🗺️ Reference Topology & Assumptions

* **HQ (IOS router or ASA on the edge)**
  Public: `203.0.113.10` (outside) • LAN: `10.0.0.0/16` (core)
* **Branch-1**: Public `198.51.100.20` • LAN `10.10.1.0/24`
* **Branch-2**: Public `198.51.100.30` • LAN `10.20.1.0/24`
* **Branch-3**: Public `198.51.100.40` • LAN `10.30.1.0/24`
* Goal: **Three site-to-site IPSec tunnels**; policy-based (crypto map) or route-based (IPSec VTI). NAT-T, DPD/keepalives, and PFS enabled. ([Cisco][2])

---

## 🔒 Baseline device hardening (before you do VPN)

**Why:** Lock the box *first*, then bring up crypto.

```cisco
! --- AAA & local accounts with distinct privilege levels
aaa new-model
username netops privilege 15 secret 5 <HASHED_SECRET>   ! Full admin (type 5 MD5 secret in this era)
username helpdesk privilege 7 secret 5 <HASHED_SECRET>  ! Limited operators

! --- Use the secret, not plain 'password'; avoid type-7 where possible
enable secret <STRONG_SECRET>                            ! Prefer 'enable secret' over 'enable password'
service password-encryption                              ! Obfuscation only; still use 'secret' for real protection

! --- SSHv2 only, no Telnet
ip domain-name corp.example
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 4
 transport input ssh
 login local                                              ! Uses the local AAA database
 exec-timeout 10 0

! --- (Optional) Role-based CLI views for finer RBAC on IOS
parser view NOC
 secret 5 <HASHED_SECRET>
 commands exec include show
 commands exec include ping
! Assign a user to the view:
username nocuser secret 5 <HASHED_SECRET>
username nocuser view NOC
```

*Comments:*

* `enable secret` uses a one-way hash and supersedes `enable password`; `service password-encryption` only obfuscates legacy fields—keep it, but don’t rely on it. **SSHv2** is mandatory; disable Telnet. Role-based **parser views** give you human-readable RBAC on IOS (handy for tiered ops). ([Cisco][3])

---

## 🛡️ Design choices (quick hits)

* **IKEv1 Main Mode** for static peers; **disable Aggressive Mode** outright. For new builds (by 2012), prefer **IKEv2/FlexVPN** if your code supports it. ([Cisco][4])
* **Transforms:** AES-256 + SHA (era-correct; SHA-2 where supported), **DH Group 14** minimum; **PFS group14** for Phase 2. ([Cisco][5])
* **NAT-T:** keep at default or 20-second keepalives; only if NAT/PAT exists on the path. ([Cisco][6])
* **Liveness:** DPD/ISAKMP keepalives (`crypto isakmp keepalive`) to rekey/cleanup dead peers. ([Cisco][7])

---

## 🧩 IOS (crypto-map) — HQ with 3 branches (IKEv1 Main Mode, PSK)

```cisco
! ===== Phase 1 (IKEv1) =====
crypto isakmp aggressive-mode disable                    ! Kill Aggressive Mode globally (defense-in-depth)
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 14
 lifetime 86400

! Optional: detect dead peers
crypto isakmp keepalive 10 periodic

! Pre-shared keys per peer (static IPs)
crypto isakmp key BR1_SuperSecret address 198.51.100.20
crypto isakmp key BR2_SuperSecret address 198.51.100.30
crypto isakmp key BR3_SuperSecret address 198.51.100.40

! NAT-T if you have NAT along the path (often auto-negotiated)
crypto isakmp nat-traversal 20

! ===== Phase 2 (IPSec) =====
crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha-hmac
 mode tunnel
!
! Stronger PFS for rekeys
! (group14 is a good 2012-era floor)
!
ip access-list extended ACL-HQ-BR1
 permit ip 10.0.0.0 0.0.255.255 10.10.1.0 0.0.0.255
ip access-list extended ACL-HQ-BR2
 permit ip 10.0.0.0 0.0.255.255 10.20.1.0 0.0.0.255
ip access-list extended ACL-HQ-BR3
 permit ip 10.0.0.0 0.0.255.255 10.30.1.0 0.0.0.255

crypto map CMAP 10 ipsec-isakmp
 set peer 198.51.100.20
 set transform-set TS-AES256
 set pfs group14
 match address ACL-HQ-BR1

crypto map CMAP 20 ipsec-isakmp
 set peer 198.51.100.30
 set transform-set TS-AES256
 set pfs group14
 match address ACL-HQ-BR2

crypto map CMAP 30 ipsec-isakmp
 set peer 198.51.100.40
 set transform-set TS-AES256
 set pfs group14
 match address ACL-HQ-BR3

! Apply to WAN interface
interface GigabitEthernet0/0
 description *** Internet-facing ***
 crypto map CMAP
```

**Why these lines matter:**

* `crypto isakmp aggressive-mode disable` ensures the box won’t *initiate or accept* IKEv1 Aggressive Mode.
* Policies define **encryption**, **hash**, **auth**, **DH group**, **lifetime** — the classic IKEv1 Phase 1 knobs.
* `set pfs group14` forces **Perfect Forward Secrecy** for Phase 2 rekeys.
* ACLs define “interesting” traffic per branch.
* NAT-T and keepalives harden tunnel *reliability* through NAT devices and flaky circuits. ([Cisco][4])

> **Branch side (IOS)** looks symmetrical: copy the above, flip source/destination subnets and the `set peer` to HQ (`203.0.113.10`). See Cisco’s policy-based site-to-site (IKEv1) examples for syntax parity and verification commands. ([Cisco][2])

---

## 🧰 IOS (route-based) — one branch as IPSec VTI (cleaner routing)

```cisco
! Phase 1 (same as above) + Phase 2 via an IPsec profile
crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha-hmac
 mode tunnel
!
crypto ipsec profile PR-HQ-BR1
 set transform-set TS-AES256
 set pfs group14

interface Tunnel10
 description *** HQ <-> Branch-1 (IPSec VTI) ***
 ip address 10.255.1.1 255.255.255.252
 tunnel source GigabitEthernet0/0
 tunnel destination 198.51.100.20
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile PR-HQ-BR1

! Now route instead of ACL matching:
ip route 10.10.1.0 255.255.255.0 10.255.1.2
```

*Why:* **VTI** simplifies life (no crypto-map ACL hair). You route over a **routable tunnel interface**, which plays nicer with dynamic routing if you need it later. Era-correct feature with IOS 12.4T+ and widely referenced in Cisco’s VTI whitepaper. ([Cisco][8])

---

## 🧱 ASA one-liner sample (one branch, IKEv1 Main Mode)

```cisco
! Kill Aggressive Mode (global)
crypto ikev1 am-disable

! IKEv1 policy
crypto ikev1 policy 10
 authentication pre-share
 encryption aes-256
 hash sha
 group 14
 lifetime 86400

! Phase 2 proposal
crypto ipsec ikev1 transform-set TS-AES256 esp-aes-256 esp-sha-hmac

! Define tunnel group (peer) and PSK
tunnel-group 198.51.100.20 type ipsec-l2l
tunnel-group 198.51.100.20 ipsec-attributes
 pre-shared-key BR1_SuperSecret

! Interesting traffic
access-list ACL-HQ-BR1 extended permit ip 10.0.0.0 255.255.0.0 10.10.1.0 255.255.255.0

! Crypto map and apply on 'outside'
crypto map OUTSIDE-MAP 10 match address ACL-HQ-BR1
crypto map OUTSIDE-MAP 10 set peer 198.51.100.20
crypto map OUTSIDE-MAP 10 set ikev1 transform-set TS-AES256
crypto map OUTSIDE-MAP interface outside
crypto ikev1 enable outside
```

*Why:* ASA syntax mirrors IOS ideas (policy, transform, peer, crypto map). `am-disable` is the key security hardening to bury Aggressive Mode. ([Cisco][9])

---

## 🚨 IPSec Aggressive Mode (IKEv1): what’s wrong & how to fix it

**The problem:** In **Aggressive Mode** with **PSK authentication**, the exchange leaks enough to allow **offline dictionary/brute-force** guessing of the preshared key. Attackers don’t have to be inline; they can capture and crack later. This has been “well-known” for years and is documented in later analyses as well. **Main Mode** avoids this exposure by protecting identities before auth; **IKEv2** removes the mode distinction entirely and is generally tighter. **Bottom line:** never use Aggressive Mode for site-to-site with PSKs. ([NVD][10])

**Mitigations (pick all that apply):**

* Use **IKEv1 Main Mode** only (static peers), and **disable Aggressive Mode** globally.

  * IOS: `crypto isakmp aggressive-mode disable`
  * ASA: `crypto ikev1 am-disable` ([Cisco][4])
* Prefer **IKEv2** (supported on many 2012-era images: IOS 15.2MT, ASA 8.4/9.x) with **certs** or long PSKs. ([Cisco][11])
* If you *must* support dynamic peers, avoid PSK+Aggressive; use **certificates** and pin identities.

### Mini-example: IOS IKEv2 / FlexVPN (2012)

```cisco
crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha1
 group 14

crypto ikev2 policy IKEV2-POL
 proposal IKEV2-PROP

crypto ikev2 keyring KR-HQ
 peer BR1
  address 198.51.100.20
  pre-shared-key local HQ_SuperSecret
  pre-shared-key remote BR1_SuperSecret

crypto ikev2 profile PROF-HQ-BR1
 match identity remote address 198.51.100.20 255.255.255.255
 authentication remote pre-share
 authentication local  pre-share
 keyring local KR-HQ

crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha-hmac
 mode tunnel
crypto ipsec profile IPSEC-HQ-BR1
 set transform-set TS-AES256
 set ikev2-profile PROF-HQ-BR1

interface Tunnel101
 ip address 10.255.101.1 255.255.255.252
 tunnel source GigabitEthernet0/0
 tunnel destination 198.51.100.20
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC-HQ-BR1
```

*Why:* **IKEv2 + VTI** is cleaner, more deterministic, and ditches Main/Aggressive ambiguity. The cited 2012 FlexVPN module documents this exact style. ([Cisco][11])

---

## 🧪 Verification & Ops quickies

```cisco
! Phase 1 / Phase 2 status (IOS)
show crypto isakmp sa
show crypto ipsec sa
debug crypto isakmp
debug crypto ipsec

! ASA equivalents
show crypto ikev1 sa
show crypto ipsec sa
debug crypto ikev1
debug crypto ipsec
```

Add **DPD** (`crypto isakmp keepalive ...`) to clear stale SAs and **syslog** the tunnel events for your SIEM. ([Cisco][7])

---

## ✅ Security checklist (period-correct)

* **Kill IKEv1 Aggressive Mode.** Main Mode or IKEv2 only. ([Cisco][4])
* **AES-256 + SHA (or SHA-2 where supported), DH-14+, PFS on Phase 2.** ([Cisco][5])
* **Strong PSKs** per peer (or, better, **certificates**); rotate on staff/vendor changes.
* **DPD/keepalives** on, sensible lifetimes (e.g., 24h / 3600s rekeys for Phase 2 as needed). ([Cisco][7])
* **Admin plane locked down:** AAA, unique **privilege levels** or **parser views**, **SSHv2 only**, no Telnet, `enable secret`, logging to syslog. ([Cisco][12])
* **NAT-T** only if necessary; otherwise keep native ESP. ([Cisco][6])
* **Prefer VTIs** (route-based) for scale/clarity; it’s era-valid and easier to operate. ([Cisco][8])

---

## 📚 Official docs & solid references (for 2008–2012 gear)

* **Disable IKEv1 Aggressive Mode (IOS/ASA):** IOS `crypto isakmp aggressive-mode disable`; ASA `crypto ikev1 am-disable`. ([Cisco][4])
* **IKEv1 site-to-site on IOS/ASA (policy-based):** config walkthroughs and verification. ([Cisco][2])
* **IPSec VTI / Route-based VPNs (IOS):** whitepaper and implementation notes. ([Cisco][8])
* **NAT-T keepalives & behavior:** command references and notes. ([Cisco][13])
* **DPD / ISAKMP keepalive (IOS):** on-demand vs periodic. ([Cisco][7])
* **IKEv2 / FlexVPN (2012 module):** design + config (IOS 15.2MT). ([Cisco][11])
* **Transform-sets, PFS, and crypto maps (IOS):** command references. ([Cisco][5])
* **Aggressive-mode PSK risk (dictionary offline):** NVD summary; academic deep-dive. ([NVD][10])
* **Admin plane hardening:** SSHv2 setup; AAA basics; `enable secret` vs `password`. ([Cisco][14])

**Books (era-appropriate):**

* *CCNP Security VPN 642-648 Official Cert Guide (2nd ed., 2012),* Hooper, Cisco Press. ([Cisco Press][15])
* *Cisco ASA: All-in-One Firewall, IPS, and VPN Services (3rd ed., 2014),* Frahim/Santos/Ossipov. ([Cisco Press][16])
* *IPSec VPN Design,* Hanz/Pezeshki-Esfahani (Cisco Press). ([Cisco Press][17])

---

### Attribution

Compiled by **I.P.**. This write-up is an original synthesis; configs reflect Cisco syntax of the time and are grounded in the vendor docs and books cited above.

[1]: https://www.cisco.com/en/US/docs/ios-xml/ios/sec_conn_ikevpn/configuration/15-2mt/sec-key-exch-ipsec.html?utm_source=chatgpt.com "Configuring Internet Key Exchange for IPsec VPNs [Support]"
[2]: https://www.cisco.com/c/en/us/support/docs/routers/1700-series-modular-access-routers/71462-rtr-l2l-ipsec-split.html?utm_source=chatgpt.com "Configure a LAN-to-LAN IPsec Tunnel Between Two Routers"
[3]: https://www.cisco.com/c/en/us/support/docs/security-vpn/remote-authentication-dial-user-service-radius/107614-64.html?utm_source=chatgpt.com "Understand Cisco IOS Password Encryption"
[4]: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/security/a1/sec-a1-cr-book/sec-cr-c4.html?utm_source=chatgpt.com "Cisco IOS Security Command Reference: Commands A to C"
[5]: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/security/a1/sec-a1-cr-book/sec-cr-c3.html?utm_source=chatgpt.com "Cisco IOS Security Command Reference: Commands A to C"
[6]: https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/sec-vpn/b-security-vpn/m_sec-ipsec-nat-transp-0.html?utm_source=chatgpt.com "Security and VPN Configuration Guide, Cisco IOS XE 17.x"
[7]: https://www.cisco.com/en/US/docs/ios-xml/ios/sec_conn_dplane/configuration/15-1s/sec-ipsec-dead-peer.html?utm_source=chatgpt.com "IPsec Dead Peer Detection Periodic Message Option"
[8]: https://www.cisco.com/en/US/technologies/tk583/tk372/technologies_white_paper0900aecd8029d629.html?utm_source=chatgpt.com "Configuring a Virtual Tunnel Interface with IP Security"
[9]: https://www.cisco.com/c/en/us/td/docs/security/asa/asa916/configuration/vpn/asa-916-vpn-config/vpn-ike.html?utm_source=chatgpt.com "Cisco ASA Series VPN CLI Configuration Guide, 9.16"
[10]: https://nvd.nist.gov/vuln/detail/CVE-2018-5389?utm_source=chatgpt.com "CVE-2018-5389 Detail - NVD"
[11]: https://www.cisco.com/en/US/docs/ios-xml/ios/sec_conn_ike2vpn/configuration/15-2mt/sec-cfg-ikev2-flex.html?utm_source=chatgpt.com "Configuring Internet Key Exchange Version 2 (IKEv2) and ..."
[12]: https://www.cisco.com/c/en/us/td/docs/ios/sec_user_services/configuration/guide/12_4/sec_securing_user_services_12-4_book/sec_role_base_cli.html?utm_source=chatgpt.com "Securing User Services, Release 12.4 - Role-Based CLI ..."
[13]: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_dplane/configuration/xe-16-12/sec-ipsec-data-plane-xe-16-12-book/sec-ipsec-nat-transp.pdf?utm_source=chatgpt.com "IPsec NAT Transparency"
[14]: https://www.cisco.com/c/en/us/support/docs/security-vpn/secure-shell-ssh/4145-ssh.html?utm_source=chatgpt.com "Configure SSH on Routers and Switches"
[15]: https://www.ciscopress.com/store/ccnp-security-vpn-642-648-official-cert-guide-9781587204470?utm_source=chatgpt.com "CCNP Security VPN 642-648 Official Cert Guide, 2nd Edition"
[16]: https://www.ciscopress.com/store/cisco-asa-all-in-one-next-generation-firewall-ips-and-9780132954402?utm_source=chatgpt.com "Cisco ASA: All-in-one Next-Generation Firewall, IPS, and ..."
[17]: https://www.ciscopress.com/store/ipsec-vpn-design-9780133433562?utm_source=chatgpt.com "IPSec VPN Design"
