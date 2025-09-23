
# 🎯 CEH v9 (2016) — Theory Q\&As + Quick Cheat Sheet

**Author:** Ivan Piskunov 
**What this is:** a tight, practitioner-style set of **original practice questions** (theory-only, not real exam items) with concise answers and “why this is right.” **Sourced from official standards and classic references** I used at the time; compiled and paraphrased for study. Use ethically and legally.

*Note:* CEH is breadth-first; focus on vocabulary, methods, and control rationales.

---

## 🧪 35 Practice Questions (with answers & reasoning)

1. **What are the classic phases of an ethical hack?**
   **Answer:** Reconnaissance → Scanning/Enumeration → Gaining Access → Maintaining Access → Covering Tracks → Reporting.
   **Why:** This mirrors EC-Council’s methodology emphasis on kill-chain-style flow; phases help you scope and justify techniques. ([EC-Council][1])

2. **Black-box vs. white-box vs. gray-box testing—what’s the practical difference?**
   **Answer:** Black-box: no internal knowledge; White-box: full knowledge (source/credentials); Gray-box: partial knowledge.
   **Why:** Knowledge level changes test depth, time, and risk; CEH expects you to tie this to scope/ROE per assessment standards. ([NIST Computer Security Resource Center][2])

3. **Passive vs. active recon—define and give one example each.**
   **Answer:** Passive: no direct touch (e.g., OSINT/DNS records). Active: direct interaction (e.g., banner grabbing).
   **Why:** Passive lowers detection risk; active yields richer fingerprints but increases exposure. ([NIST Computer Security Resource Center][2])

4. **SYN scan vs. Connect() scan—what’s the gist?**
   **Answer:** SYN (“half-open”) sends SYN and reads the reply; Connect() completes the TCP handshake.
   **Why:** SYN is stealthier (no full session), Connect() is noisier but simplest and most compatible.

5. **Banner grabbing serves what purpose?**
   **Answer:** Identifies service and (often) version via initial responses/strings.
   **Why:** Speeds vuln mapping and reduces false positives during enumeration.

6. **Common DNS abuse in recon—name one and a control.**
   **Answer:** AXFR zone transfer; mitigate by restricting transfers (TSIG, IP-based ACL), disable public recursion.
   **Why:** Prevents mass leakage of hostnames/internal structure.

7. **What’s the difference between vulnerability scanning and penetration testing?**
   **Answer:** Scanning finds potential weaknesses; pen-tests **validate exploitability** under rules of engagement.
   **Why:** CEH stresses that testing requires authorization, scoping, and a report with risk context. ([NIST Computer Security Resource Center][2])

8. **OSSTMM and PTES—what are they?**
   **Answer:** **OSSTMM** and **PTES** are community testing methodologies for structured security assessments.
   **Why:** They standardize phases (e.g., PTES seven phases) and reporting; useful to justify approach in the exam. ([isecom.org][3])

9. **What problem does time-sync solve in logging/IR?**
   **Answer:** Correlating events across systems.
   **Why:** Without NTP/Chrony, timelines break and evidence loses value in IR.

10. **IDS vs. IPS—what’s the operational difference?**
    **Answer:** IDS detects and alerts; IPS can block inline.
    **Why:** Signature-based is precise but brittle; anomaly-based sees novel activity but is noisier.

11. **Steganography—what is it and why would an attacker use it?**
    **Answer:** Hiding data inside benign carriers (e.g., images).
    **Why:** Covert exfiltration and C2 smuggling; defenders use statistical detection.

12. **XSS types and one mitigation.**
    **Answer:** Reflected, Stored, DOM-based; mitigate via **context-aware output encoding** and CSP.
    **Why:** Encoding breaks script injection paths at the sink.

13. **SQL Injection categories and the single best mitigation.**
    **Answer:** Union/error/blind; **parameterized queries (prepared statements)**.
    **Why:** Param binding separates code from data, defusing injection.

14. **CSRF—what’s the core idea and control?**
    **Answer:** Forcing a victim’s browser to perform authenticated actions; defend with **anti-CSRF tokens** and SameSite cookies.
    **Why:** Breaks the attacker’s ability to forge state-changing requests.

15. **Broken Auth vs. Broken Access Control—difference?**
    **Answer:** Auth: proving identity/session handling; Access Control: enforcing **who can do what** after login.
    **Why:** Distinct risks in OWASP; exam expects you to separate them. ([OWASP Foundation][4])

16. **Symmetric vs. asymmetric crypto—use-cases?**
    **Answer:** Symmetric (AES) for bulk speed; Asymmetric (RSA/ECC) for key exchange/signatures.
    **Why:** Hybrids use asymmetric to exchange a symmetric session key.

17. **Hash vs. HMAC—what’s added?**
    **Answer:** HMAC adds a secret key to a hash for integrity **and** authenticity.
    **Why:** Prevents attacker-forged digests.

18. **PKI: CRL vs. OCSP?**
    **Answer:** CRL = list of revoked certs; OCSP = real-time status check.
    **Why:** OCSP reduces latency/staleness; stapling improves privacy and performance.

19. **TLS Perfect Forward Secrecy—what enables it?**
    **Answer:** Ephemeral DH/ECDHE key exchange.
    **Why:** Past captured traffic stays safe even if server’s key is later compromised.

20. **Wireless: WPA2-PSK vs. WPA2-Enterprise.**
    **Answer:** PSK = shared secret; Enterprise = per-user creds via 802.1X (RADIUS/EAP).
    **Why:** Enterprise scales and isolates accounts; PSK is easier but less manageable.

21. **Social engineering flavors—name three.**
    **Answer:** Phishing (incl. spear/whaling), vishing, smishing, pretexting, baiting.
    **Why:** Targets human controls; train + simulate + MFA.

22. **Malware taxonomy—what’s a rootkit?**
    **Answer:** Stealth component hiding processes/files/keys and attacker presence.
    **Why:** Evades detection; kernel-mode is most potent.

23. **Risk math—define SLE, ARO, ALE.**
    **Answer:** SLE = **Single Loss Expectancy**; ARO = **Annualized Rate of Occurrence**; **ALE = SLE × ARO**.
    **Why:** Classic quantitative model for control cost-benefit.

24. **Access control models—DAC, MAC, RBAC, ABAC.**
    **Answer:** DAC = owner-decides; MAC = labels/clearances; RBAC = roles; ABAC = attributes/policy.
    **Why:** Know which fits compliance (e.g., MAC in high-assurance).

25. **Incident response lifecycle (NIST)—list the stages.**
    **Answer:** **Preparation → Detection/Analysis → Containment → Eradication → Recovery → Post-Incident (Lessons Learned).**
    **Why:** NIST’s canonical loop; CEH expects terminology accuracy. ([NIST Computer Security Resource Center][5])

26. **Data sanitization: clear vs. purge vs. destroy.**
    **Answer:** Clear = logical overwrite; Purge = more robust (e.g., crypto erase); Destroy = physical.
    **Why:** Choose level based on data sensitivity and media type. ([NIST Computer Security Resource Center][6])

27. **Email auth: SPF, DKIM, DMARC—what’s the trio do?**
    **Answer:** SPF = sending host allowed; DKIM = content signed; DMARC = policy tying SPF/DKIM to the From domain.
    **Why:** Reduces spoofing and improves deliverability/defense.

28. **SIEM value prop in CEH context?**
    **Answer:** Normalizes/correlates logs for detection and reporting.
    **Why:** Turns raw events into alerts + evidence across assets.

29. **MITRE ATT\&CK—what is it and why should a tester care?**
    **Answer:** A knowledge base of adversary **tactics/techniques** organized as a matrix.
    **Why:** Helps map findings to real-world behaviors and communicate impact. ([MITRE ATT\&CK][7])

30. **Evasion basics an IDS should catch but often doesn’t?**
    **Answer:** Fragmentation, TTL games, encoding/padding, protocol quirks.
    **Why:** Defensive tuning must reassemble/normalize traffic.

31. **Windows network exposure—why disable LM and require SMB signing where possible?**
    **Answer:** LM is weak; unsigned SMB enables MITM tampering.
    **Why:** Hardens legacy auth and integrity.

32. **Physical security in CEH scope—one example control.**
    **Answer:** Tamper-evident seals on racks/ports.
    **Why:** Prevents covert device placement and supports forensics.

33. **Cloud shared-responsibility—who secures what?**
    **Answer:** Provider secures the platform **of** the service; customer secures data/configuration **in** the service.
    **Why:** Misconfigs (IAM, storage ACLs) are customer-side risk.

34. **Web session security: what does SameSite cookie do?**
    **Answer:** Restricts cross-site sending of cookies.
    **Why:** Helps mitigate CSRF and some cross-site leakage.

35. **Scope & Rules of Engagement (RoE)—why are they legally critical?**
    **Answer:** Define authorization, timing, systems, methods, and stop-conditions.
    **Why:** Protects both parties; aligns with NIST testing guidance and PTES pre-engagement phase. ([NIST Computer Security Resource Center][2])

---

## 🧷 CEH v9 Quick Cheat Sheet (exam-friendly)

**Frameworks & Flows**

* **Hacking phases:** Recon → Scan/Enum → Gain → Maintain → Cover → Report.
* **PTES phases:** Pre-engagement, Intelligence Gathering, Threat Modeling, Vuln Analysis, Exploitation, Post-Exploitation, Reporting. ([pentest-standard.org][8])
* **NIST IR:** Prep → Detect/Analyze → Contain → Eradicate → Recover → Lessons Learned. ([NIST Computer Security Resource Center][5])
* **ATT\&CK:** Tactics columns (Recon → Resource Dev → Initial Access … Impact). Map findings to TTPs. ([MITRE ATT\&CK][7])

**Crypto & Identity**

* **Symmetric vs. Asymmetric:** bulk vs. exchange/sign; enable **PFS** with E(DHE).
* **Hash vs. HMAC:** integrity vs. integrity+authenticity.
* **PKI checks:** CRL vs. OCSP (stapling).

**Web App Risks**

* **OWASP Top 10 (2013→2017):** Injection, Broken Auth, Sensitive Data Exposure, XXE, Broken Access Control, Misconfig, XSS, Insecure Deserialization, Components with Known Vulns, Insufficient Logging & Monitoring. ([OWASP Foundation][4])

**Risk Math**

* **SLE = Asset Value × Exposure Factor**, **ALE = SLE × ARO** (remember units/year).

**Network Hygiene**

* **Default-deny inbound**, minimize services, log centrally, time-sync.
* **Email trio:** SPF + DKIM + DMARC policy alignment.

**Data Lifecycle**

* **Clear → Purge → Destroy** (per **NIST SP 800-88r1** media & data sensitivity). ([NIST Computer Security Resource Center][6])

---

## 📚 Official Docs & Legit Study Aids

* **EC-Council CEH (official program page)** — exam overview/objectives. ([EC-Council][1])
* **NIST SP 800-115** — *Technical Guide to Information Security Testing & Assessment* (testing methods, planning, reporting). ([NIST Computer Security Resource Center][2])
* **NIST SP 800-61 r2 / r3** — *Computer Security Incident Handling Guide* (r2 historic; now superseded by r3). ([NIST Computer Security Resource Center][5])
* **NIST SP 800-88 r1** — *Guidelines for Media Sanitization* (clear/purge/destroy). ([NIST Computer Security Resource Center][6])
* **OWASP Top 10 (2017 & earlier 2013)** — risks, mitigations, testing references. ([OWASP Foundation][4])
* **PTES (Penetration Testing Execution Standard)** — official site/phases & technical guidelines. ([pentest-standard.org][8])
* **OSSTMM 3** — peer-reviewed methodology for operational security testing. ([isecom.org][3])
* **MITRE ATT\&CK** — tactics/techniques matrix for mapping behaviors. ([MITRE ATT\&CK][7])

**Books (solid for v9 era):**

* *Certified Ethical Hacker (CEH) v9 Cert Guide* — Michael Gregg, Pearson/Sybex. ([Pearson IT Certification][9])
* *CEH v9: Practice Tests* — Ric Messier, Wiley. ([Wiley][10])
* *Gray Hat Hacking (5th ed.)* — Harper/Regalado et al., McGraw-Hill. ([McGraw Hill][11])
* *The Web Application Hacker’s Handbook (2nd ed.)* — Stuttard & Pinto, Wiley. ([Wiley][12])
* *Metasploit: The Penetration Tester’s Guide* — Kennedy et al., No Starch. ([Amazon][13])

---

## ✍️ Attribution & Notes

* **Compiled by:** *Deputy Head of IT Security (Bank, Top-400)* — **author of this study set**.
* **Sources:** vendor standards and public references above; wording and sequences are my own.
* **Ethics & legality:** obtain written authorization, scope, and RoE before any testing (PTES/NIST). ([NIST Computer Security Resource Center][2])

[1]: https://www.eccouncil.org/train-certify/certified-ethical-hacker-ceh/?utm_source=chatgpt.com "CEH Certification | Ethical Hacking Training & Course"
[2]: https://csrc.nist.gov/pubs/sp/800/115/final?utm_source=chatgpt.com "Technical Guide to Information Security Testing and Assessment"
[3]: https://www.isecom.org/OSSTMM.3.pdf?utm_source=chatgpt.com "OSSTMM 3 – The Open Source Security Testing ..."
[4]: https://owasp.org/www-project-top-ten/2017/?utm_source=chatgpt.com "OWASP Top Ten 2017 | Table of Contents"
[5]: https://csrc.nist.gov/pubs/sp/800/61/r2/final?utm_source=chatgpt.com "SP 800-61 Rev. 2, Computer Security Incident Handling Guide"
[6]: https://csrc.nist.gov/pubs/sp/800/88/r1/final?utm_source=chatgpt.com "SP 800-88 Rev. 1, Guidelines for Media Sanitization | CSRC"
[7]: https://attack.mitre.org/matrices/?utm_source=chatgpt.com "Enterprise Matrix"
[8]: https://www.pentest-standard.org/index.php/Main_Page?utm_source=chatgpt.com "The Penetration Testing Execution Standard"
[9]: https://www.pearsonitcertification.com/store/certified-ethical-hacker-ceh-version-9-cert-guide-premium-9780134680804?utm_source=chatgpt.com "Certified Ethical Hacker (CEH) Version 9 Cert Guide ..."
[10]: https://www.wiley.com/en-us/CEH%2Bv9%3A%2BCertified%2BEthical%2BHacker%2BVersion%2B9%2BPractice%2BTests-p-x000952162?utm_source=chatgpt.com "CEH v9: Certified Ethical Hacker Version 9 Practice Tests"
[11]: https://www.mheducation.com/highered/mhp/product/gray-hat-hacking-ethical-hacker-s-handbook-fifth-edition.html?utm_source=chatgpt.com "Gray Hat Hacking: The Ethical Hacker's Handbook, Fifth ..."
[12]: https://www.wiley.com/en-us/The%2BWeb%2BApplication%2BHacker%27s%2BHandbook%3A%2BFinding%2Band%2BExploiting%2BSecurity%2BFlaws%2C%2B2nd%2BEdition-p-9781118026472?utm_source=chatgpt.com "The Web Application Hacker's Handbook: Finding and ..."
[13]: https://www.amazon.com/Metasploit-Penetration-Testers-David-Kennedy/dp/159327288X?utm_source=chatgpt.com "Metasploit: The Penetration Tester's Guide"
